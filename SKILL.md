---
name: sol-max-terra-low-workflow
description: Orchestrate repository coding tasks with a read-only Sol Max planner, a minimal-change Terra Low executor, and fail-closed Sol Max decisions. Use when a user asks for this Sol Max plan → Terra Low implementation workflow, needs high-confidence implementation delegation, or wants planning, execution, validation, and final review separated by model role.
---

# Sol Max → Terra Low workflow

Use this skill to complete a repository coding request through bounded model roles. Read [the protocol](references/protocol.md) in full before beginning any task work; it defines the report formats, escalation rules, state transitions, and validation gates.

Run one active implementation path at a time: planner → executor → (zero or more decision/replan turns) → validation → optional final review. Do not run Terra concurrently with a planner, decision, or reviewer that could change its instructions.

## Required orchestration

1. Create `sol_max_planner` with `model: gpt-5.6-sol`, `reasoning_effort: max`, and no inherited task context. Instruct it to inspect the repository read-only and return the exact `## PLAN` protocol artifact. Do not let it modify product files. The plan has no length limit: make it as detailed as necessary for Terra Low to execute safely without further judgment requests, including exact file/symbol locations, control flow, edge cases, caller effects, existing patterns, and validation expectations. After requesting the plan, continue waiting for the planner until it returns a complete `## PLAN`; do not time out, proceed, or start Terra based on an incomplete, delayed, or absent plan. Save the complete, unabridged artifact as a Markdown file in the target repository's `./.plan/` directory before handing it to Terra.
2. Send the complete, unabridged plan to `terra_low_executor`, created with `model: gpt-5.6-terra`, `reasoning_effort: low`, and no inherited task context. It must perform the specified preflight checks, implement only the approved steps, and run their validations.
3. On every protocol escalation condition, stop implementation before any further edit. Create or reuse `sol_max_decision` with `model: gpt-5.6-sol`, `reasoning_effort: max`; provide the plan reference plus the exact `## DECISION_REQUEST` and only decision-relevant source evidence, diff, and test output. Relay its unabridged decision to the executor.
4. Have the executor complete the final validation and diff review. If the protocol requires final Sol review, give a read-only Sol reviewer the complete plan, final diff, and validation evidence; apply any exact `CHANGES_REQUIRED` instructions through Terra and revalidate.
5. Respond to the user using only the `## STATUS` completion-report format from the protocol. Never represent unchecked validations as passing.

## Role boundaries

- Sol agents analyze, plan, decide, and review; they do not edit product code by default. Explicitly prohibit mutation commands in their task prompt.
- Terra edits, tests, and verifies only the current approved plan or exact Sol decision. It must fail closed when a meaningful choice, mismatch, or unplanned scope appears; the planner must make the plan sufficiently detailed to avoid such requests wherever the repository evidence permits.
- Preserve user changes, minimize the diff, and treat the user request, approved user decisions, latest Sol decision, plan, then repository conventions as the precedence order.

## Execution notes

- Treat each PLAN/DECISION artifact as immutable until superseded by a Sol decision. Label amendments with a new plan version, save every complete initial, amended, or remainder PLAN as a separate Markdown file in the target repository's `./.plan/` directory, and pass the full current plan to Terra.
- Reuse a live Sol decision agent for follow-ups when possible; do not re-explore the whole repository for an isolated decision.
- Ask Sol to replan the remaining work after two decision requests in one implementation step or when the protocol's replan signals occur. Do not give Terra a stale plan after that request.
- Use the smallest appropriate specialist set only when the underlying task requires it. For user-visible UI, follow the repository and global UI-workflow instructions in addition to this protocol.
