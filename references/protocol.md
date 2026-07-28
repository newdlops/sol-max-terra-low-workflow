# Operational protocol

## State and handoff rules

Keep the current plan version, current step ID, executed commands/results, and every Sol decision in the orchestration conversation. Start Terra only after the planner's `## PLAN` is complete. After the planning request, wait continuously for the planner to complete that artifact; a delayed response, polling timeout, or temporary lack of output is not permission to proceed, substitute a plan, or end the workflow. Before the Terra handoff, create the target repository's `./.plan/` directory if needed and save the planner's exact, complete `## PLAN` artifact as a Markdown file there. The orchestrator must not summarize, paraphrase, shorten, trim examples or evidence, collapse steps, merge sections, or rewrite the planner artifact. The saved file and the Terra handoff must contain the same substantive planner-authored artifact; transport-only newline normalization is allowed. Use a descriptive, unique filename containing its work identifier and `PLAN_VERSION` (for example, `./.plan/<work-id>-<plan-version>.md`); never overwrite an earlier plan version. During an escalation, Terra must stop editing; it may inspect the named evidence and collect the requested validation output only. Resume it only with a `CONTINUE` decision, an unabridged amended plan, or an unabridged remainder plan.

Use this model policy exactly: planner, decision-maker, and final reviewer use `gpt-5.6-sol` at `max` reasoning; executor uses `gpt-5.6-terra` at `low` reasoning. Spawn these roles without inherited task context and give each role only the task-local material it needs. Sol role prompts must explicitly prohibit writes to product code. Do not reuse a final review as a planning or decision turn.

## Planner contract

Before substantive planning inspection, the Sol planner must derive and record `SCOPE_LOCK` from the user request and approved user decisions. If material ambiguity would change the concern or outcome, return `USER_DECISION_REQUIRED`. Inspect only code, callers, tests, configuration, and data structures directly required to implement or validate that lock. Inspection does not authorize modification: record adjacent discoveries as `OUT_OF_SCOPE_FINDINGS` with `NO ACTION`. Do not guess names, APIs, or framework behavior, and do not modify product code.

Apply this plan-completeness standard:

- Plan detail has no workflow maximum, no minimum, and no line, word, token, section, step, example, or evidence target. Plan length is not a completeness signal. Include only detail directly required to implement and validate `SCOPE_LOCK`.
- Define completeness as depth inside the lock: required happy, failure, empty, and boundary behavior; direct dependencies and callers; tests; validation; and behavior that must remain unchanged.
- Every planned changed file, symbol, implementation step, test, and validation must map to an acceptance criterion or preservation invariant. Repository proximity, convention, or an adjacent defect is not authorization.
- For each step, provide only the edit map and control-flow detail needed for deterministic in-lock execution. Include pseudocode only when the required implementation would otherwise remain ambiguous.
- Name only directly affected callers, configuration, schemas, and generated artifacts required by the locked outcome or a preservation invariant. Do not enumerate unrelated repository inventory or global no-change conclusions.
- Do not add opportunistic features, UX expansion, cleanup, refactoring, abstractions, documentation, configuration, schemas, generated artifacts, or unrelated tests merely because they were discovered.
- Leave Terra only local, nonsemantic mechanics listed in `ALLOWED_EXECUTOR_DISCRETION`. If an in-lock approach, branch, semantic, direct caller, required test, assertion, or validation remains undecided, the plan is incomplete.
- Resolve missing facts only inside `SCOPE_LOCK`. Record adjacent discoveries in `OUT_OF_SCOPE_FINDINGS` with `NO ACTION`. A new concern, outcome, or optional adjacent task requires `USER_DECISION_REQUIRED`; do not expand the plan.

Return one chosen, executable solution in exactly this structure:

```markdown
## PLAN

### PLAN_VERSION
<version and work identifier>

### RISK_LEVEL
<LOW | NORMAL | HIGH>

### GOAL
<user-visible result>

### CURRENT_BEHAVIOR
<facts grounded in file paths and symbols>

### ACCEPTANCE_CRITERIA
- AC-1 — Input/condition → expected result
- AC-2 — Failure condition → must fail
- INV-1 — Existing behavior that remains unchanged

### SCOPE
<SCOPE_LOCK: the user-authorized concern, outcome, exact files/symbols allowed to change, and direct-dependency boundary>

### NON_GOALS
<excluded concerns and outcomes; OUT_OF_SCOPE_FINDINGS: <finding> — NO ACTION, or NONE>

### INVARIANTS
- INV-1 — <compatibility, transaction, authorization, integrity, or no-unrelated-change requirement>

### IMPLEMENTATION_STEPS
1. **<ID>** — `<file>` / `<symbol>`
   - Acceptance/invariant mapping: <AC-* and/or INV-* IDs>
   - Current behavior: <exact relevant branches/data flow and source locations>
   - Edit map: <declarations/blocks/branches to add, replace, move, or remove, with exact anchors and ordering>
   - Change: <exact edits, signatures/types/fields, data transformations, side effects, error handling, and behavior to preserve>
   - Ordered control flow: <implementation-ready branch sequence or pseudocode, including success, empty, and failure paths>
   - Existing pattern/reference: <file and symbol to follow, and which aspects to copy>
   - Direct dependency impact: <only directly affected callers, configuration, schemas, or generated artifacts required by the mapping; otherwise NONE>
   - Test specification: <mapped test file and case, setup/fixtures/mocks, action, exact assertions, and unchanged directly affected tests>
   - Edge cases: <inputs, failures, empty states, boundary conditions, and expected result>
   - Done when: <observable completion conditions mapped to AC-* and INV-* IDs>
   - Validate: `<command>` → <expected success condition and AC-*/INV-* IDs proved>

### ALLOWED_EXECUTOR_DISCRETION
<an exhaustive list limited to local naming, private helper names, import ordering, formatting, nonsemantic test-fixture form, and mechanical repair of Terra's own syntax/type/import errors using an identical repository pattern; write NONE if no discretion is needed>

### MANDATORY_ESCALATION
<task-specific conditions, including USER_DECISION_REQUIRED before broadening SCOPE_LOCK>

### VALIDATION_PLAN
- Compile/type check: ...
- Unit: ...
- Integration: ...
- Lint/static analysis: ...
- Manual business scenario: ...
- Performance/data/authorization: ...

### ROLLBACK_OR_RECOVERY
<required for HIGH risk; otherwise N/A>

### OPEN_ASSUMPTIONS
<verified assumptions inside SCOPE_LOCK, or an explicit user/extra-investigation blocker>

### PLAN_COMPLETENESS_SELF_CHECK
- Remaining meaningful executor decisions inside SCOPE_LOCK: NONE
- Planned changes or tests without an AC-*/INV-* mapping: NONE
- Unspecified direct-caller, error, edge-case, or test behavior inside SCOPE_LOCK: NONE
- OUT_OF_SCOPE_FINDINGS converted into work: ZERO
- Unauthorized scope expansion: ZERO
- Evidence that each acceptance criterion maps to implementation and validation: <step/test mapping>
```

Classify LOW only for local, mechanical changes with nearly no external/data-semantic effect; NORMAL for standard multi-file product work following existing structure; HIGH for integrity, migration, transaction/concurrency, auth, money/quantity/inventory/state transition, public contract, security, or operational-data work. Resolve material assumptions inside `SCOPE_LOCK` before handing off to Terra. Investigate only facts directly required for in-lock implementation or validation. Reject a plan with unmapped work, unresolved in-lock executor judgment, or adjacent-scope additions; deepen it only inside the lock. If completion requires a broader concern or outcome, return `USER_DECISION_REQUIRED`.

## Terra preflight and implementation

Read the whole plan. Before editing, confirm `SCOPE_LOCK` is explicit; every named change and test has an acceptance-criterion or preservation-invariant mapping; every named file and symbol exists; current behavior and plan preconditions match the branch; the worktree has no user changes that would be overwritten; and planned test commands exist. Any meaningful mismatch or out-of-lock instruction means stop and request Sol decision.

Perform steps in order. After each, run its smallest mapped validation. Make the smallest diff and follow existing patterns only inside `SCOPE_LOCK`; a pattern is not authorization to change adjacent code. Do not act on `OUT_OF_SCOPE_FINDINGS`, add unmapped work, or hide errors. Directly fix only obvious, uniquely determined syntax/import/format/simple type errors introduced by Terra's immediately previous patch and supported by a clear in-lock pattern.

Escalate immediately if any of these are true:

1. A named file/type/function/field is missing or semantically different, or current behavior differs from plan.
2. Work needs an unplanned or unmapped file, layer, assumption, abstraction, dependency, test, or validation, or would cross `SCOPE_LOCK`.
3. There are multiple meaningful approaches, conflicts between request/plan/code/tests, or uncertain test-failure cause.
4. Public APIs, schemas, database, events/messages/serialization, data semantics, transactions/locking/retry/idempotency, money/quantity/inventory/state, security/auth/privacy, or existing callers' exceptional behavior are affected.
5. Framework behavior must be guessed, validation cannot prove completion, normal flow passes but error/rollback/caller impact is unclear, or the same step fails twice.
6. Passing tests would require removing or weakening a test or validation.

When unsure, escalate. After two Sol decision requests in the same step, ask Sol to replan the remaining work inside the same `SCOPE_LOCK` instead of continuing point decisions. A replan may narrow, reorder, or deepen in-lock work; broadening requires `USER_DECISION_REQUIRED`.

## Decision request and response

Terra must send only relevant evidence in this exact format:

```markdown
## DECISION_REQUEST

### PLAN_REFERENCE
- PLAN_VERSION: ...
- Current implementation step: ...

### INTENDED_CHANGE
...

### OBSERVED_FACTS
- `<file>:<symbol/location>` — current behavior ...

### MISMATCH_OR_DECISION
...

### OPTIONS
<include only genuine options, each with short tradeoff and scope>

### CURRENT_DIFF
<only relevant changed fragments/summary>

### VALIDATION_EVIDENCE
`<command>`: <key result>

### REQUIRED_DECISION
<one precise question>
```

Sol must decide within the current `SCOPE_LOCK`. It may narrow, reorder, or deepen mapped work, but it must return `USER_DECISION_REQUIRED` instead of authorizing a new concern, outcome, or optional adjacent task. Sol then returns a single executable decision:

```markdown
## DECISION
<chosen approach>

## REASON
<short evidence-based reason>

## EXACT_CHANGES
- `<file>` / `<symbol>`: <exact behavior, branches, data flow, error handling, and code to preserve>

## INVARIANTS
- ...

## VALIDATION
- `<command>` → <success condition>

## PLAN_UPDATE
<CONTINUE | AMEND | REPLAN_REMAINDER | USER_DECISION_REQUIRED>
```

For `AMEND` or `REPLAN_REMAINDER`, assign a new `PLAN_VERSION` and include the complete amended/remainder plan unabridged. Preserve `SCOPE_LOCK`; amendments may narrow, reorder, or deepen mapped work but may not broaden the concern or outcome. Before Terra resumes, save that complete plan as a new Markdown file in `./.plan/` using the same work identifier and its new plan version; preserve all earlier plan files. Terra must apply it without reinterpretation. `CONTINUE` may add only the exact mapped changes and validations stated in the decision; it does not authorize unrelated plan changes. `USER_DECISION_REQUIRED` authorizes no edit.

## Completion gates

Terra must run the planned compile/type check, relevant unit and integration tests, lint/static analysis, directly affected caller-path tests, and complete diff review. Confirm every changed hunk and test maps to an acceptance criterion or preservation invariant; only planned files changed; `OUT_OF_SCOPE_FINDINGS` received no action; every acceptance criterion and invariant holds; and no debug output, TODO, temporary workaround, or bypass remains. Report tests that cannot run, with the reason.

Require a read-only final Sol review for HIGH risk; amended/replanned work; decisions changing data semantics or external behavior; public/DB/event/auth changes; transaction/concurrency/money/quantity/inventory/state work; skipped validation; inadequate existing tests; or an explicit user request. Give the reviewer the current full plan, every plan amendment, `git diff` (or equivalent), and concise command/result evidence. The reviewer returns exactly `PASS`, `CHANGES_REQUIRED` with exact Terra instructions, or `REPLAN_REQUIRED`. `CHANGES_REQUIRED` may contain only repairs mapped inside `SCOPE_LOCK`; adjacent findings remain `NO ACTION`. Apply valid instructions through Terra and rerun applicable validation; return to scope-locked planning for `REPLAN_REQUIRED`.

You may omit final Sol review only for LOW/NORMAL work with no Sol decision during implementation, an unchanged plan, all mandatory checks passing, and no external-contract or data-semantic change.

## User completion report

```markdown
## STATUS
COMPLETED | COMPLETED_WITH_MANUAL_CHECKS | BLOCKED | USER_DECISION_REQUIRED

## SUMMARY
<user-facing result>

## CHANGED_FILES
- `<file>` — <key change>

## VALIDATION
- `<command>` — <result>

## SOL_DECISIONS
<decision summary and applied plan version, or 없음>

## MANUAL_CHECKS
<business scenarios, or 없음>

## REMAINING_RISKS
<unverified risks, or 없음>
```
