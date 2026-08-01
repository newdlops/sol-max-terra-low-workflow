# Operational protocol

## Contents

- [Operating balance](#operating-balance)
- [State and handoff rules](#state-and-handoff-rules)
- [Planner contract](#planner-contract)
- [Terra implementation](#terra-implementation)
- [Validation lock and failure handling](#validation-lock-and-failure-handling)
- [Decision protocol](#decision-protocol)
- [Completion gates](#completion-gates)
- [User completion report](#user-completion-report)

## Operating balance

Optimize for delivered behavior, not verification volume.

- Keep implementation at least 80% of implementation-plus-verification active effort and verification at most 20%. Count product/test edits and corrective fixes as implementation. Count preflight inspection, test execution, manual checking, diff review, decision evidence gathering, and final Sol review as verification. Keep planning concise; do not relabel verification as planning to evade the budget.
- Treat 80:20 as a ceiling on process overhead, not a reason to manufacture implementation. Select evidence by risk and changed behavior.
- Prefer one targeted check that directly exercises the acceptance criterion over several indirect checks. Batch related checks after a coherent edit instead of validating every small patch.
- Do not rerun a passing check unless relevant code changed, do not run both overlapping checks without a distinct risk, and do not inspect unaffected callers or repository areas for reassurance.
- Use validation to confirm accepted behavior, never to discover new requirements or adjacent work. Check output is evidence about the locked change; it is not authority to expand inspection, implementation, or validation.
- If credible mandatory verification would exceed 20%, first narrow the verification surface, stage the delivery, or reduce scope without weakening the accepted behavior. If none is safe, return `USER_DECISION_REQUIRED` and ask permission to exceed the ratio.

## State and handoff rules

Keep the current plan version, current step, commands/results, material decisions, and a lightweight effort ledger in the orchestration conversation. The ledger uses coarse units, not token or wall-clock accounting:

```markdown
IMPLEMENTATION_UNITS: <edits, tests authored, corrective fixes>
VERIFICATION_UNITS: <preflight, checks run, reviews>
BALANCE: <implementation>:<verification>
```

Start Terra only after a complete `## PLAN`. Save the exact planner artifact under `./.plan/<work-id>-<plan-version>.md`; never overwrite an earlier version. Transport-only newline normalization is allowed. Keep amended plans only when a material Sol decision changes the chosen approach; local executor adaptations do not require plan-file churn.

Use `gpt-5.6-sol` at `max` reasoning for planner, material decisions, and required final review. Use `gpt-5.6-terra` at `low` reasoning for execution. Spawn without inherited task context and explicitly prohibit Sol from writing product code. Do not run a role whose decision could invalidate concurrent Terra edits.

## Planner contract

Derive `SCOPE_LOCK` from the user request and approved decisions before inspection. Inspect only enough code, tests, configuration, and direct dependencies to choose one viable approach. Return `USER_DECISION_REQUIRED` only when ambiguity changes the requested outcome, public behavior, risk posture, or authorized scope.

Produce an implementation-ready but compact plan. Do not create exhaustive repository inventories, narrate obvious mechanics, prove each line independently, or prescribe all local naming and formatting. Resolve material semantics; leave reversible local mechanics to Terra.

Return exactly:

```markdown
## PLAN

### PLAN_VERSION
<version and work identifier>

### RISK_LEVEL
<LOW | NORMAL | HIGH>

### GOAL_AND_SCOPE
<user-visible outcome, SCOPE_LOCK, allowed files/areas, and meaningful non-goals>

### ACCEPTANCE_CRITERIA
- AC-1 — <condition → observable result>
- INV-1 — <behavior that must remain unchanged>

### IMPLEMENTATION_STEPS
1. **<ID>** — `<file/area>`: <behavioral change and important control flow>
   - Maps to: <AC-*/INV-*>
   - Test change, if needed: <case and assertion>

### MATERIAL_RISKS_AND_ESCALATION
- <only semantic, safety, irreversible, public-contract, or scope conditions requiring Sol/user decision; NONE if absent>

### MINIMUM_VALIDATION
- V-1 — `<command or manual scenario>` — Evidence target: <one AC-* or material risk>; Stop: <decisive result that ends this item>

### EFFORT_BUDGET
- Implementation units: <estimate, at least 80% of combined units>
- Verification units: <estimate, at most 20% of combined units>
```

Classify HIGH for migrations, authorization/security, money/inventory, concurrency/transactions, destructive operations, operational data, or consequential public-contract changes. Tests are part of implementation when they encode new or changed behavior; do not plan tests solely to increase verification volume. Every validation item must prove exactly one acceptance criterion or material risk and end at its stated stop condition. Reject a validation item that is exploratory, duplicates another item's evidence, has no stop condition, or scans beyond `SCOPE_LOCK` merely for reassurance. Reject only plans with an unresolved material semantic choice, unsafe scope, no implementable path, or a budget below 80:20.

## Terra implementation

Read the whole plan. Perform a brief preflight limited to:

1. Confirm the target repository and `SCOPE_LOCK`.
2. Check that planned edit areas and commands broadly exist.
3. Protect overlapping user changes.
4. Identify only blockers that could invalidate the chosen approach.

Do not verify every symbol, assertion mapping, caller, or command before beginning when those facts can be resolved safely during implementation.

Implement coherent steps before running their targeted checks. Use existing patterns and make the smallest scope-locked diff. Terra Low may choose local names, import placement, equivalent test-fixture form, and uniquely determined mechanical in-scope adaptations. It may repair its own syntax, type, import, format, and test failures and may adjust stale file/symbol anchors when the intended behavior and approach remain clear. It must follow an existing repository pattern rather than inventing a new abstraction or semantic approach.

Escalate only when at least one condition holds:

1. The requested outcome or chosen approach would materially change.
2. Work must cross `SCOPE_LOCK`, add a dependency, alter a public contract/data semantic, or perform an irreversible/high-risk operation not approved by the plan.
3. Request, plan, code, and tests materially conflict and repository evidence does not identify one safe answer.
4. Two focused implementation/fix attempts fail for the same unexplained reason.
5. Passing requires weakening a relevant test or safety control.

Do not escalate for a renamed private symbol, moved file with an unambiguous replacement, local pattern choice, mechanical failure, formatting, or a single understandable test failure. Continue unaffected implementation while a decision concerns only a separable step; stop all edits only when the decision could invalidate the broader approach.

Update the effort ledger after each coherent implementation block and its validation. If verification approaches 20%, stop at the highest-value remaining locked evidence. Ask the user before exceeding the ceiling; do not add checks to compensate for an inconclusive result.

## Validation lock and failure handling

Freeze `MINIMUM_VALIDATION` as `VALIDATION_LOCK` when the plan is handed to Terra. It is a closed allowlist, not a starting point for discovery. Run only its items, and stop each item at its declared condition. A check result cannot create a requirement, implementation step, audit, adjacent fix, new check category, or broader completion topic.

For a failed locked item, perform one focused classification using only the check output, the changed hunks, and an exact file or symbol named by that output:

1. If the failure is directly caused by the current change and one repair is uniquely determined inside `SCOPE_LOCK`, make that repair and rerun the same item once.
2. If the failure is pre-existing, unrelated, or outside `SCOPE_LOCK`, record the item as failed with minimal attribution. Do not investigate, fix, scan, or validate the adjacent area.
3. If attribution remains indeterminate, or the item still fails after its one repair and rerun, stop the item and report it as failed. Do not branch into another diagnostic or check. Request a user decision only when the unresolved fact makes the locked acceptance criteria unsafe or impossible; otherwise report the remaining risk.

Treat out-of-scope observations as quarantined. Do not promote them into the plan, a Sol decision, a replan, final-review feedback, or the user-facing topic. If one proves the locked work unsafe or impossible, return `USER_DECISION_REQUIRED` with only the observed fact and precise decision needed. Even when an explicitly required repository-wide command emits many failures, classify only failures directly tied to changed hunks or the locked evidence target.

A material in-scope Sol decision may replace an affected validation item with an equal-or-narrower item for the same evidence target. It must not append checks cumulatively. Only an explicit user-approved scope change may broaden `VALIDATION_LOCK`.

## Decision protocol

Send only evidence needed for the material decision:

```markdown
## DECISION_REQUEST

### PLAN_REFERENCE
<version and step>

### MATERIAL_BLOCKER
<observed fact, why it changes semantics/scope/safety, and one precise question>

### OPTIONS
<genuine options with short tradeoffs, or NONE>

### RELEVANT_EVIDENCE
<minimal source/diff/test evidence>
```

Sol returns:

```markdown
## DECISION
<chosen approach>

## EXACT_CHANGES
<only behavior needed to unblock Terra>

## VALIDATION
<smallest affected check>

## PLAN_UPDATE
<CONTINUE | AMEND | REPLAN_REMAINDER | USER_DECISION_REQUIRED>
```

Keep `SCOPE_LOCK`. For `AMEND` or `REPLAN_REMAINDER`, save the complete new plan version and hand it to Terra. Use `CONTINUE` for localized decisions; do not replan merely to refresh anchors, document an implementation detail, or pursue a validation observation. After a decision, replace the affected locked validation item when necessary and run only that item; do not append validation.

## Completion gates

Terra must:

1. Run only the closed `VALIDATION_LOCK` items that directly cover changed behavior and material risks, stopping each at its declared condition.
2. Inspect the final changed hunks for scope, accidental debug code, and obvious regressions.
3. Report each locked item as passed, failed, unavailable, or skipped, and never represent an unchecked condition as passing.
4. Confirm the final effort ledger is at least 80:20. If not, obtain explicit user approval before more verification or reduce redundant verification work.

No validation category is mandatory by name. Compile/typecheck, unit, integration, lint, manual, performance, data, and authorization checks are selected only when they add distinct evidence for this change. A full suite is reserved for repository convention, broad cross-cutting change, release gating, or explicit user request.

Require final read-only Sol review only for HIGH risk, an unresolved skipped safety-critical check, a material decision that changed public/data/auth semantics, or explicit user request. An ordinary amendment, replan, local decision, or validation observation does not by itself trigger review. Give the reviewer the current plan, final diff, and concise evidence. It returns `PASS`, `CHANGES_REQUIRED` mapped to a locked acceptance criterion and changed hunk, or `REPLAN_REQUIRED` only when the locked approach is independently nonviable. Omit out-of-scope recommendations. Apply repairs through Terra and rerun only the affected locked item once.

## User completion report

```markdown
## STATUS
COMPLETED | COMPLETED_WITH_MANUAL_CHECKS | BLOCKED | USER_DECISION_REQUIRED

## SUMMARY
<user-facing result>

## CHANGED_FILES
- `<file>` — <key change>

## VALIDATION
- `<locked item>` — <PASSED | FAILED | UNAVAILABLE | SKIPPED; result and why it was sufficient or limiting>

## EFFORT_BALANCE
<implementation units>:<verification units> — <at least 80:20, or explicit user-approved exception>

## SOL_DECISIONS
<material decision summary and plan version, or 없음>

## REMAINING_RISKS
<unverified material risks that affect the locked outcome, or 없음; omit incidental out-of-scope observations>
```
