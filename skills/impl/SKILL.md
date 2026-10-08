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

A preparatory refactoring is not incidental: when the ticket fits the code only through workarounds (`final-review`, Evidence bar), propose the smallest refactoring that makes it fit, and start it after the user agrees.

Use SOLID as design judgement, not a compliance checklist. YAGNI wins over speculative SOLID, while accepted architecture and existing public contracts still apply. Dependency inversion belongs at real external boundaries; a single internal implementation does not automatically need an interface.

Treat roughly 100–150 lines as a cognitive review threshold, not a file-size limit. When crossing it, check whether the file contains a coherent named responsibility that should move. Extract only when doing so improves locality and comprehension. Do not split a cohesive file into tiny files merely to satisfy a number, and do not refactor pre-existing large files without a ticket need.

**Exit:** every planned file and abstraction has a current, nameable reason to exist.

## 3. Implement production behavior

Implement directly through the identified seams. Keep the change local to the ticket. Run typechecking and focused existing checks during development when they provide useful feedback.

Do not add or modify tests, fixtures, snapshots, test helpers, or test configuration. Do not weaken checks to make them pass. If the intended behavior makes an existing test obsolete, report the conflict for the later testing phase instead of rewriting the test here.

For every acceptance criterion, retain an implementation location or an explicit unresolved blocker.

**Exit:** every criterion is implemented or explicitly blocked; no test artifact changed.

## 4. Validate and review once

Run the `final-review` skill over the implementation; the ticket is its requirements source.

**Exit:** validation evidence and acceptance-criteria coverage are recorded, one review round is complete, and every finding is triaged.

## 5. Hand off for human review

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
