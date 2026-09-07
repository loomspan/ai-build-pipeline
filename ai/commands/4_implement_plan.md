---
description: Implement an approved plan and verify it safely
---

# Implement Plan

Implement the supplied plan and testing plan completely while preserving unrelated developer work.

## Shared Protocols

- Follow `ai/commands/shared/automation-protocol.md` for execution mode, escalation, and the Step Report.
- Preserve any active `ai/thoughts/design-lens.md` guardrails identified by the plan.

## Getting Started

1. Read the ticket, research, implementation plan, and testing plan completely.
2. Inspect `git status` and the relevant diff before editing. Existing changes belong to the developer unless the artifacts clearly identify them as this ticket's work.
3. Reread every production and test file named by the plan plus directly affected callers.
4. Create a working checklist from the plan and begin when the intended behavior is clear.

If no plan path is provided, ask for one. Do not infer that an arbitrary plan in `ai/thoughts/plans/` is the intended input.

## Implementation Rules

- Follow the plan's outcome and decisions while adapting routine details to the current code.
- Implement one coherent phase at a time, including its tests and documentation/configuration changes.
- Prefer existing repository conventions in the affected area.
- Keep unrelated modernization and cleanup out of scope.
- Use the repository's standard build and test tools.
- Use `apply_patch` or another reviewable edit mechanism; do not overwrite unrelated changes.

Do not perform live or destructive operations as routine verification. Use the safe boundaries, test environments, and operational constraints established by the plan and repository.

## Plan Mismatches

Routine adaptation includes a renamed helper, a moved line, or an additional caller clearly covered by the plan. Record material adaptations in `DECISIONS` and continue.

A genuine mismatch exists when:

- the plan's assumption is false;
- observable behavior or scope must change;
- an affected consumer, dependency, or operational constraint needs treatment the plan did not address;
- the implementation changes the risk profile materially; or
- safe verification cannot establish a required acceptance criterion.

For a genuine mismatch, stop only the affected work and finish independent safe work. State `Expected`, `Found`, and `Why it matters`, then propose the smallest plan correction. Ask the developer in standalone mode or return `STATUS: needs-developer` in pipeline mode. Never hide the mismatch behind a fallback or broaden scope silently.

## Verification Loop

For each phase:

1. Add or update the planned test first when the testing plan calls for a red test.
2. Confirm the intended pre-fix failure when practical and record the exact command/result.
3. Implement the change.
4. Run the focused test, then the broadest safe relevant suite.
5. Inspect the diff for correctness, secret or data leakage, accidental live-service use, and unrelated edits.
6. Check completed boxes in the implementation and testing plans only after the work and verification are actually complete.

Do not claim a test passed if it was not run. When a test is blocked by a pre-existing failure or missing environment dependency, capture the exact failure, determine whether a narrower safe test still proves the change, and report residual risk accurately.

Do not return `STATUS: complete` while verification is failing because of the ticket-scoped change or while a required acceptance criterion lacks sufficient executable evidence. Fix the failure, return `needs-developer` when resolving it requires a material decision, or return `failed` when an unrecoverable tool or environment failure prevents completion.

## Completion

Before reporting completion:

- map every acceptance criterion to code and test evidence;
- confirm all required code, tests, configuration, documentation, and supporting artifacts changed coherently;
- verify no secrets or sensitive data entered the diff or logs;
- confirm routine tests did not perform unintended live or destructive operations; and
- review the full ticket-scoped diff, not only the last phase.

End with the Step Report. Under `VERIFICATION`, list exact commands actually run with PASS/FAIL/NOT RUN. Under `OPTIONAL_DEVELOPER_CHECKS`, carry forward nonblocking manual or configured-environment observations from the plan. The independent step-5 review, not implementation confidence, determines final completion.

## Resuming Work

When plan checkboxes are already complete, verify the corresponding code exists and continue from the first incomplete item. Re-run prior verification only when the current diff, failure, or dependency makes it necessary. Never use destructive Git commands to recreate a clean state.
