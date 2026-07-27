# Operational protocol

## State and handoff rules

Keep the current plan version, current step ID, executed commands/results, and every Sol decision in the orchestration conversation. Start Terra only after the planner's `## PLAN` is complete. During an escalation, Terra must stop editing; it may inspect the named evidence and collect the requested validation output only. Resume it only with a `CONTINUE` decision, an unabridged amended plan, or an unabridged remainder plan.

Use this model policy exactly: planner, decision-maker, and final reviewer use `gpt-5.6-sol` at `max` reasoning; executor uses `gpt-5.6-terra` at `low` reasoning. Spawn these roles without inherited task context and give each role only the task-local material it needs. Sol role prompts must explicitly prohibit writes to product code. Do not reuse a final review as a planning or decision turn.

## Planner contract

Before implementation, the Sol planner must actually inspect relevant code, callers, tests, configuration, and data structures. It must not guess names, APIs, or framework behavior, and it must not modify product code. Return one chosen, executable solution in exactly this structure:

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
- Input/condition → expected result
- Failure condition → must fail
- Existing behavior that remains unchanged

### SCOPE
<exact files and symbols, including tests>

### NON_GOALS
<excluded scope>

### INVARIANTS
<compatibility, transaction, authorization, integrity, and no-unrelated-change requirements>

### IMPLEMENTATION_STEPS
1. **<ID>** — `<file>` / `<symbol>`
   - Current behavior: ...
   - Change: ...
   - Existing pattern/reference: ...
   - Done when: ...
   - Validate: `<command>`

### ALLOWED_EXECUTOR_DISCRETION
<only local naming, private helper name, imports, formatting, nonsemantic test-fixture form, and mechanical repair of Terra's own syntax/type/import errors using an identical repository pattern>

### MANDATORY_ESCALATION
<task-specific conditions>

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
<verified assumptions, or an explicit user/extra-investigation blocker>
```

Classify LOW only for local, mechanical changes with nearly no external/data-semantic effect; NORMAL for standard multi-file product work following existing structure; HIGH for integrity, migration, transaction/concurrency, auth, money/quantity/inventory/state transition, public contract, security, or operational-data work. Resolve material assumptions before handing off to Terra.

## Terra preflight and implementation

Read the whole plan. Before editing, confirm every named file and symbol exists, its claimed current behavior and plan preconditions match the current branch, the worktree has no user changes that would be overwritten, and planned test commands exist. Any meaningful mismatch means stop and request Sol decision.

Perform steps in order. After each, run its smallest stated validation. Make the smallest diff; follow existing patterns; do not add layers, dependencies, abstractions, refactors, hardcoded bypasses, behavior changes, or weakened/deleted tests outside the approved scope. Do not hide errors. Directly fix only obvious, uniquely determined syntax/import/format/simple type errors introduced by Terra's immediately previous patch and supported by a clear existing pattern.

Escalate immediately if any of these are true:

1. A named file/type/function/field is missing or semantically different, or current behavior differs from plan.
2. Work needs an unplanned file/layer, assumption, abstraction, dependency, or meaningful performance decision.
3. There are multiple meaningful approaches, conflicts between request/plan/code/tests, or uncertain test-failure cause.
4. Public APIs, schemas, database, events/messages/serialization, data semantics, transactions/locking/retry/idempotency, money/quantity/inventory/state, security/auth/privacy, or existing callers' exceptional behavior are affected.
5. Framework behavior must be guessed, validation cannot prove completion, normal flow passes but error/rollback/caller impact is unclear, or the same step fails twice.
6. Passing tests would require removing or weakening a test or validation.

When unsure, escalate. After two Sol decision requests in the same step, or if scope/constraints/failures keep expanding, ask Sol to replan all remaining work instead of continuing point decisions.

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

Sol must investigate the provided facts and return a single executable decision:

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

For `AMEND` or `REPLAN_REMAINDER`, assign a new `PLAN_VERSION` and include the complete amended/remainder plan unabridged. Terra must apply it without reinterpretation. `CONTINUE` may add only the exact changes and validations stated in the decision; it does not authorize unrelated plan changes.

## Completion gates

Terra must run the planned compile/type check, relevant unit and integration tests, lint/static analysis, tests for changed caller paths, and complete diff review. Also confirm only planned files changed; every acceptance criterion and invariant holds; and no debug output, TODO, temporary workaround, or bypass remains. Report tests that cannot run, with the reason.

Require a read-only final Sol review for HIGH risk; amended/replanned work; decisions changing data semantics or external behavior; public/DB/event/auth changes; transaction/concurrency/money/quantity/inventory/state work; skipped validation; inadequate existing tests; or an explicit user request. Give the reviewer the current full plan, every plan amendment, `git diff` (or equivalent), and concise command/result evidence. The reviewer returns exactly `PASS`, `CHANGES_REQUIRED` with exact Terra instructions, or `REPLAN_REQUIRED`. Apply `CHANGES_REQUIRED` through Terra and rerun applicable validation; return to planning for `REPLAN_REQUIRED`.

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
