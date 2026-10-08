---
name: impl
description: "Implement one ticket for human review, without writing tests or committing."
disable-model-invocation: true
---

# Implementation

Implement one ticket as the smallest complete production change that satisfies its current requirements. Stop after one advisory review and hand the implementation to the user. Tests are authored only in a later, explicitly requested phase.

## 1. Ground and bound

Read the full ticket, its comments, linked spec, and dependency state using `docs/agents/issue-tracker.md`. Read the repository instructions, `CONTEXT.md`, relevant ADRs, and architecture documents required for the affected area. Follow `docs/agents/domain.md` when it exists; missing domain documents are not blockers and do not need scaffolding.

Inspect the relevant source, public interfaces, existing tests, and executable scripts. Record the repository status before editing and distinguish pre-existing changes from this ticket's changes. Preserve unrelated work. If ownership overlaps or the intended diff is ambiguous, stop and ask whether to continue in place or isolate the work.

Write down:

- the acceptance criteria;
- explicit exclusions and non-goals;
- affected public seams and architecture constraints;
- unresolved blockers or consequential product/API ambiguities;
- the validation commands discovered from repository scripts.

Resolve conflicting requirements, open blockers, and consequential ambiguity before coding. Do not implement blocker tickets or invent requirements to unblock yourself.

**Exit:** the ticket scope, exclusions, seams, blockers, and validation commands are explicit.

## 2. Choose the laziest complete design

Prefer the smallest concrete implementation that completely satisfies the current ticket. Reuse an existing mechanism before introducing another one. Every planned change must serve an acceptance criterion, an existing contract, or an accepted architecture rule.

Apply **YAGNI** aggressively:

- no speculative extension points, configuration, or future-proofing;
- no abstraction for a hypothetical consumer;
- no unnecessary interfaces, factories, services, managers, helpers, or generic utilities;
- no incidental cleanup or unrelated refactoring;
- record worthwhile future work in the handoff instead of implementing it.

Use SOLID as design judgement, not a compliance checklist. YAGNI wins over speculative SOLID, while accepted architecture and existing public contracts still apply. Dependency inversion belongs at real external boundaries; a single internal implementation does not automatically need an interface.

Treat roughly 100–150 lines as a cognitive review threshold, not a file-size limit. When crossing it, check whether the file contains a coherent named responsibility that should move. Extract only when doing so improves locality and comprehension. Do not split a cohesive file into tiny files merely to satisfy a number, and do not refactor pre-existing large files without a ticket need.

**Exit:** every planned file and abstraction has a current, nameable reason to exist.

## 3. Implement production behavior

Implement directly through the identified seams. Keep the change local to the ticket. Run typechecking and focused existing checks during development when they provide useful feedback.

Do not add or modify tests, fixtures, snapshots, test helpers, or test configuration. Do not weaken checks to make them pass. If the intended behavior makes an existing test obsolete, report the conflict for the later testing phase instead of rewriting the test here.

For every acceptance criterion, retain an implementation location or an explicit unresolved blocker.

**Exit:** every criterion is implemented or explicitly blocked; no test artifact changed.

## 4. Validate the implementation

Use repository scripts as the command source of truth. Run the applicable typecheck, lint, formatting check, focused existing tests, full normal test suite, and required integration/extension suite. Long-running checks may run in the background.

Classify failures honestly as:

- introduced by this implementation;
- pre-existing;
- blocked by the environment.

Resolve implementation-caused failures before review. Record unrun or blocked required checks as outstanding; do not claim readiness when a known implementation defect remains.

Create an acceptance-criteria table with each requirement, its implementation location, and status.

**Exit:** validation evidence and acceptance-criteria coverage are recorded.

## 5. Review once and triage

Run exactly one advisory review round over one captured change set. Review both axes:

- **Standards:** repository rules, accepted architecture, and maintainability;
- **Spec:** missing, incorrect, or extra behavior relative to the ticket.

Review is evidence, not authority. Validate every finding against the source, ticket, and project rules before recommending action. Preserve which review axis produced it. Classify each finding as:

- **confirmed defect** — demonstrated violation of the ticket, an existing contract, or an accepted project rule;
- **risk** — plausible concern without enough evidence to call it a defect;
- **optional improvement** — useful but unnecessary for this ticket;
- **scope expansion** — behavior or cleanup not requested by the ticket;
- **false positive** — contradicted by the source or requirements.

Assign `blocking`, `important`, or `minor` severity. A review comment never becomes a new acceptance criterion. Do not apply review findings automatically, and do not start a review → fix → re-review loop. Further edits or another review require explicit user direction.

### Working-tree review contract

Do not commit merely to make review tooling work. By default, review the implementation in the working tree, including relevant staged changes, unstaged changes, and untracked files. Both review axes must inspect the same captured scope and baseline.

The existing `/code-review` flow compares a fixed point to `HEAD`; invoke it unchanged only when that range already contains the complete intended implementation. Otherwise perform the same two-axis review directly against the captured working-tree change set.

**Exit:** one review round is complete and every finding is triaged; no post-review edits were made.

## 6. Hand off for human review

Report:

- the implementation summary;
- the acceptance-criteria table;
- changed public interfaces or behavior;
- validation commands and outcomes;
- triaged review findings and recommended decisions;
- deferred improvements, existing-test conflicts, and remaining risks;
- a short manual-review checklist;
- a testing recommendation grouped into **critical now**, **optional**, and **deliberately excluded** scenarios.

For each recommended test, name the observable behavior, the risk it protects against, the stable public seam, and the appropriate test level. Prefer a few durable behavioral or integration scenarios over many implementation-coupled unit tests. It is valid to recommend that no new tests are worthwhile.

End with exactly one state: **implementation ready for human review** or **blocked/incomplete**. After the user's review, offer `/tests` for the separately approved testing phase.

Do not write tests, stage files, commit, push, close the ticket, or declare the ticket complete. Those actions belong to later user-authorized phases.
