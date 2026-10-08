---
name: final-review
description: "Review a completed implementation before human review, or a working-tree change before commit."
---

# Final review

One validation and one advisory review round over a finished change, before the user reviews it. The round is *final*: its findings go to the user triaged, and further edits or another round wait for the user's direction. This skill authorizes validation and review; staging, commits, and test authoring belong to later phases (`tests`).

## 1. Find the record

Find the task directory by the repository's artifact convention, otherwise `docs/design/<task>/`; create it when there is none. A finished round for this change set in its `final-review/` is the answer: report it rather than run another.

Find the requirements in this order: the ticket and its linked spec, the task's `architecture.md`, decisions already recorded in the task directory. When none holds them, write `spec.md` from what the user agreed in the conversation: behavior, exclusions, and what is still unknown. Show it to the user and launch reviewers after their ok. Requirements come from the agreement, never from the diff under review: a spec read off the implementation makes the Spec axis check the code against itself.

**Exit:** the requirements have a durable source the user has seen, and the task directory exists.

## 2. Validate

Discover commands from repository scripts. Run applicable typecheck, lint, formatting checks, focused existing tests, the normal test suite, and required integration/extension checks. Use background execution for long-running checks when supported.

Classify failures with evidence as introduced, pre-existing, or environment-blocked. A test that conflicts with intentionally changed behavior is an outstanding test conflict, not automatically a pre-existing failure. Inside an implementation phase, resolve implementation defects before review; on a review-only request, report them and ask before repairing. Record every required unrun check and its reason.

Keep tests, fixtures, snapshots, helpers, and test configuration unchanged; preserve checks rather than weakening them. Build an acceptance table linking each criterion to its implementation location and status.

**Exit:** coverage and validation evidence are recorded, with all failures and gaps accounted for.

## 3. Review once

Capture one baseline and the complete intended change set: staged, unstaged, and untracked files. Separate pre-existing changes. Keep that scope stable during review; every reviewer inspects the same snapshot. A commit is not a prerequisite: the `code-review` skill compares a fixed point to `HEAD`, so use it only when that range already holds the whole change.

Run one fresh-context delegate per axis (`multi-agent-delegate`, role: axis reviewer, no `skills:`) and give each the evidence bar below:

- **Standards:** project rules, applicable architecture, and maintainability.
- **Spec:** missing, incorrect, or extra behavior against the requirements.

Verify availability before launch; surface unavailable routes rather than substituting them.

### Intent review

After the two axes, run one more fresh-context delegate (role: intent reviewer; `skills: architect` when the architect is reachable) that checks the change against the design's *intent*, not its letter. The intent lives in `architecture.md`; without one, in the requirements from step 1 together with the repository's architecture documents and ADRs, and the reviewer says no task design was recorded. Its prompt carries:

- the snapshot, the intent sources, and the two axis reports, so it does not redo them;
- the focus: each block in the right layer, block boundaries and public entries, composition, over-engineering the change introduced (facades, wrappers, needless narrow types, defensive checks), workarounds where the structure should have changed (Evidence bar), and leftovers (dead code, stale names, docs describing the old structure);
- when the phase's architect is reachable: consult it through the `architect` skill's Ask branch, answers in `final-review/answers-<n>.md`. First ask what it considers most important to check and which decisions it regards as risky or provisional; then review the whole change; then put every finding it is unsure of to the architect before reporting;
- the report at `final-review/intent.md`: each finding with `file:line`, classification, severity, recommended fix, and the architect answers that shaped the verdicts.

The architect holds what the documents leave implicit, so its pass catches what a cold reviewer files as taste or misses: a feature that is a facade around one module call, composition bookkeeping that grew without need. Without a reachable architect the reviewer works from the documents alone, and its report says the consultation did not happen.

### Triage

Apply the evidence bar personally to every finding before recommending action. Preserve its axis and classify it:

- **confirmed defect**: a demonstrated violation of the requirements, an existing contract, or an accepted project rule;
- **risk**: a plausible concern without enough evidence to call it a defect;
- **optional improvement**: useful but unnecessary for this change;
- **scope expansion**: behavior or cleanup the requirements did not ask for;
- **false positive**: contradicted by the source or requirements.

Assign `blocking`, `important`, or `minor` severity. Review advice is evidence, not a new requirement. Write the baseline, scope, reports, and triaged findings to `final-review/round.md`.

**Exit:** one round is complete, every finding is triaged and recorded in `final-review/round.md`, and the code is as the reviewers saw it.

Return the acceptance table, validation outcomes and gaps, and the triaged findings to the skill that called you (`build`, `impl`). Run on its own, report them to the user and finish with exactly one state: **implementation ready for human review** or **blocked/incomplete**; a known defect or missing required verification means the latter.

## Evidence bar

Reference for reviewers, triage, slice reviews and reconciliation of delegated work (`build`, `delegating-slices`), and implementers deciding how a change fits.

### Reachability before recommendation

Trace the actual entry point, callers, input/state provenance, lifecycle or event ordering, and existing guards to the claimed observable failure. Cite concrete source locations and realistic triggering conditions. A reproduction is useful; a complete source-backed execution trace can also establish reachability without authoring tests.

A hypothetical caller, impossible state, contract-breaking mock, or imagined future consumer is not evidence of a present defect. At real external boundaries, consider inputs allowed by the actual trust model rather than assuming all inputs are well-behaved. If reachability is unresolved, state exactly what evidence is missing and recommend investigation, not a defensive patch. If existing invariants exclude the scenario, dismiss it with that evidence. Recommend a fix only for a demonstrated reachable failure or a demonstrated violation of an applicable requirement or architecture rule.

### Root cause before remedy

For each confirmed problem, trace which owner should enforce the invariant and where invalid state or ordering becomes possible. Check responsibility allocation, competing sources of truth, lifecycle ownership, and dependency direction before proposing a local guard, retry, or fallback.

Prefer the smallest correction at the responsible layer that removes the cause. If a modest ownership or decomposition change eliminates the bad state, recommend that instead of masking it at consumers. A local check is appropriate when it enforces a legitimate boundary invariant, not merely because it suppresses a symptom. Explain the causal diagnosis and why the proposed layer owns the fix; escalate architectural amendments through the design-approval process. This diagnosis does not authorize post-review edits or unrelated redesign.

### Workarounds

A **workaround** fits a change around a structure that should have changed. Its markers:

- a flag or mode parameter that switches behavior;
- a special case keyed on a caller or a particular value;
- a near-copy of existing code;
- a swallowed error;
- a sleep or retry standing in for ordering;
- a consumer-side check standing in for the owner's invariant.

A marker is a question, not a verdict: trace it by the two rules above. A marker with a legitimate reason, such as a real configuration point or a boundary contract, stays; name the reason. When the trace shows a structure that does not fit the change, recommend the smallest **preparatory refactoring** that removes the cause (make the change easy, then make the easy change), with the requirement it serves and why a local correction falls short. It is a design amendment: it waits for the design owner's approval.
