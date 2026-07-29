---
name: sol-max-terra-low-workflow
description: Orchestrate repository coding tasks with a read-only Sol Max planner, a minimal-change Terra Low executor, risk-based validation, and Sol decisions only for material blockers. Use when a user asks for this Sol Max plan → Terra Low implementation workflow, needs bounded implementation delegation, or wants implementation to remain at least 60% of implementation-plus-verification effort.
---

# Sol Max → Terra Low workflow

Use this skill to complete a repository coding request through bounded model roles. Read [the protocol](references/protocol.md) in full before beginning any task work; it defines the report formats, escalation rules, state transitions, and validation gates.

Run one active implementation path at a time: concise planner → implementation-led executor → proportionate validation → optional final review. Do not run Terra concurrently with a planner, decision, or reviewer that could change its instructions.

## Delivery balance

- Keep implementation work at least 60% and verification at most 40% of their combined active effort. Implementation includes product/test changes and fixes needed to deliver the accepted behavior. Verification includes preflight inspection, test execution, diff review, and Sol review; planning is kept concise and is not used to evade this budget.
- Choose the smallest validation set that gives credible evidence for the risk and changed behavior. Do not run every available check, repeat unchanged checks, or add review ceremonies merely because the protocol supports them.
- If required verification would exceed the 40% ceiling, first narrow or stage validation without weakening safety. If that is impossible, stop and ask the user before exceeding the ceiling.

## Required orchestration

1. Create `sol_max_planner` with `model: gpt-5.6-sol`, `reasoning_effort: max`, and no inherited task context. Instruct it to derive `SCOPE_LOCK`, inspect only facts needed to choose one implementation path, and return the concise `## PLAN` artifact from the protocol. It must not modify product files. Require acceptance criteria, edit steps, material risks, and the minimum sufficient validation set; do not require exhaustive repository inventory, per-hunk proofs, or ceremonial completeness sections. Wait for the complete plan, save it unchanged under `./.plan/`, and hand off that same artifact.
2. Send that same exact, complete planner artifact to `terra_low_executor`, created with `model: gpt-5.6-terra`, `reasoning_effort: low`, and no inherited task context. It must perform the specified preflight checks, implement only the approved steps, and run their validations.
3. Escalate only a material semantic, scope, safety, or irreversible decision that Terra cannot resolve from the plan and existing repository patterns. Let Terra correct local plan drift, mechanical issues, and uniquely determined in-scope adaptations without a Sol turn. For a true escalation, stop the affected edit, create or reuse `sol_max_decision` with `model: gpt-5.6-sol`, `reasoning_effort: max`, and relay its unabridged decision.
4. Have the executor run the smallest risk-appropriate validation set and review changed hunks. Use final Sol review only when the protocol requires it; apply mapped `CHANGES_REQUIRED` through Terra and rerun only affected checks.
5. Respond to the user using only the `## STATUS` completion-report format from the protocol. Never represent unchecked validations as passing.

## Role boundaries

- Sol agents analyze, plan, decide, and review; they do not edit product code by default. Explicitly prohibit mutation commands in their task prompt.
- Sol planning, decisions, replans, final review, and Terra execution may narrow, reorder, or deepen work only within `SCOPE_LOCK`; none may broaden it. Terra may make reversible, local, pattern-backed implementation choices inside the chosen approach. It may not change product semantics or scope. A broader concern or outcome requires `USER_DECISION_REQUIRED`.
- Preserve user changes, minimize the diff, and treat the user request, approved user decisions, latest Sol decision, plan, then repository conventions as the precedence order.

## Execution notes

- Treat each PLAN/DECISION artifact as immutable until superseded by a Sol decision. Label amendments with a new plan version, save every complete initial, amended, or remainder PLAN as a separate Markdown file in the target repository's `./.plan/` directory, and pass the full current plan to Terra.
- Reuse a live Sol decision agent for follow-ups when possible; do not re-explore the whole repository for an isolated decision.
- Replan only when the chosen approach is no longer viable, not merely because a local detail changed or a check failed once. Replanning may narrow, reorder, or deepen the current `SCOPE_LOCK`; broadening it requires `USER_DECISION_REQUIRED`.
- Use the smallest appropriate specialist set only when the underlying task requires it. For user-visible UI, follow the repository and global UI-workflow instructions in addition to this protocol.
