---
name: tests
description: "Plan and author tests after the user has reviewed an implementation."
---

# Behavior Tests

Lock down approved ticket behavior with the fewest durable tests that provide meaningful confidence. Optimize for protection against real regressions, not test count or coverage percentage.

Run this after the production implementation has received human review. It may follow `/impl` in the same session.

Hand steps 1–4 to a coordinator as described in the `delegating-slices` skill; the brief is `tests-brief.md` in the task directory (create both when missing), sent with `skills: delegating-slices, tests`. The brief carries the user's review decisions, the requirements (without a ticket, the behavior the user agreed in the conversation), and the sources to read; the coordinator chooses the scenarios. Its execution plan is the scenario list of step 2 plus who writes the tests and how the work is cut. When you are that coordinator, run steps 1–4 yourself. A test-slice worker runs step 3 and the checks of step 4 on its slice, and returns its report to the coordinator.

## 1. Ground the testing phase

Read the ticket, comments, linked spec (without a ticket, the agreed behavior in `spec.md` or the brief), implementation diff, and the user's review decisions. Read repository instructions, `CONTEXT.md`, relevant ADRs, architecture documents, existing test conventions, and executable scripts. Reuse the current session's accepted decisions, but verify requirements against their primary sources.

Identify:

- the observable behavior that was approved;
- the public seams through which callers or users experience it;
- material regression risks;
- existing tests that already provide relevant confidence;
- unresolved implementation defects or requirement ambiguity.

Stop and ask when production behavior is not yet accepted or the intended outcome is ambiguous. Testing must not silently decide product behavior.

**Exit:** the approved behavior, stable seams, existing coverage, and material risks are explicit.

## 2. Propose tests before writing them

Invoke the `testing-scenarios` skill and apply its recommendation rules and after-implementation branch. Reconcile any architecture-stage plan with the accepted implementation and present the changes. Present the scenario list as part of the execution plan and obtain explicit approval of both before writing tests; prior approval of the architecture or production implementation does not authorize test authoring.

**Exit:** the user has approved a bounded scenario list and the execution plan.

## 3. Write durable tests

Write only the approved scenarios.

When the plan splits authoring into slices, cut them by test file, one worker per file, each in its own worktree: the extension suite rebuilds `dist/`, so parallel runs collide. The one infrastructure change a worker may need is registering a new extension test file in the `esbuild.mjs` entry-point list. In the review gate, also check each test against this step: it maps to an approved scenario, observes a public seam, and its fakes honor the real contract.

- Test observable outcomes through public interfaces.
- Use real internal components where practical.
- Fake or mock external systems, nondeterminism, time, filesystem, process, network, or editor boundaries only when needed for control and repeatability.
- Derive expected outcomes from the ticket, an agreed example, or another independent source of truth—not by repeating the production algorithm.
- Use the domain vocabulary from `CONTEXT.md` in test names and fixtures.
- Keep setup focused on the behavior being demonstrated.
- Prefer one coherent scenario over several tests that merely assert intermediate implementation steps.

Do not test private methods, internal abstractions, collaborator call counts, or incidental ordering unless that ordering is itself part of the public contract. Do not add an abstraction solely to make an internal detail mockable. A refactor that creates a genuinely needed public seam requires separate user approval.

If a test exposes a production defect or contradicts the approved behavior, stop and report it. Do not silently alter production code or weaken the test.

A durable test should normally require change only when the observable requirement changes, not when the implementation is refactored.

**Exit:** every added test corresponds to an approved scenario and observes behavior through a stable seam.

## 4. Validate and hand off

Run each focused test while developing, then run the applicable typecheck, lint, formatting check, full normal suite, and required integration/extension suite using repository scripts. Distinguish introduced failures, pre-existing failures, and environmental blockers.

Report:

- tests added and the requirements or risks they protect;
- the seam and test level used for each;
- optional scenarios left out;
- commands and outcomes;
- production defects or limitations discovered;
- any reason the tests may still be sensitive to implementation changes.

Do not stage, commit, push, or close the ticket unless the user explicitly requests it.
