# Completion

Use after parent reconciliation. Test authoring is a separate, explicitly authorized phase.

## Validate

Discover commands from repository scripts. Run applicable typecheck, lint, formatting checks, focused existing tests, the normal test suite, and required integration/extension checks. Use background execution for long-running checks when supported.

Classify failures with evidence as introduced, pre-existing, or environment-blocked. A test that conflicts with intentionally changed behavior is an outstanding test conflict, not automatically a pre-existing failure. Resolve implementation defects before final review. Record every required unrun check and its reason.

Keep tests, fixtures, snapshots, helpers, and test configuration unchanged during this phase; preserve checks rather than weakening them. Build an acceptance table linking each criterion to implementation evidence and status.

**Exit:** coverage and validation evidence are recorded, with all failures and gaps accounted for.

## Review once

Capture one baseline and complete intended change set, including relevant staged, unstaged, and untracked files. Separate pre-existing changes. Keep that scope stable during review; both reviewers inspect the same snapshot. A commit is not a prerequisite.

Run one independent advisory round using fresh-context subagents:

- **Standards:** project rules, accepted architecture, and maintainability.
- **Spec:** missing, incorrect, or extra behavior against requirements.

Take their model from the `multi-agent-delegate` role map (read-only review).

Verify availability before launch; surface unavailable routes rather than substituting them.

Use an existing review skill only if it supports the intended captured scope. A fixed-point-to-HEAD review does not cover uncommitted implementation.

Give reviewers the evidence bar below and apply it personally to every finding before recommending action. Preserve its axis and classify it as confirmed defect, risk, optional improvement, scope expansion, or false positive; assign blocking, important, or minor severity. Review advice is evidence, not a new requirement.

### Reachability before recommendation

Trace the actual entry point, callers, input/state provenance, lifecycle or event ordering, and existing guards to the claimed observable failure. Cite concrete source locations and realistic triggering conditions. A reproduction is useful; a complete source-backed execution trace can also establish reachability without authoring tests.

A hypothetical caller, impossible state, contract-breaking mock, or imagined future consumer is not evidence of a present defect. At real external boundaries, consider inputs allowed by the actual trust model rather than assuming all inputs are well-behaved. If reachability is unresolved, state exactly what evidence is missing and recommend investigation, not a defensive patch. If existing invariants exclude the scenario, dismiss it with that evidence. Recommend a fix only for a demonstrated reachable failure or a demonstrated violation of an applicable requirement or architecture rule.

### Root cause before remedy

For each confirmed problem, trace which owner should enforce the invariant and where invalid state or ordering becomes possible. Check responsibility allocation, competing sources of truth, lifecycle ownership, and dependency direction before proposing a local guard, retry, or fallback.

Prefer the smallest correction at the responsible layer that removes the cause. If a modest ownership or decomposition change eliminates the bad state, recommend that instead of masking it at consumers. A local check is appropriate when it enforces a legitimate boundary invariant, not merely because it suppresses a symptom. Explain the causal diagnosis and why the proposed layer owns the fix; escalate architectural amendments through the design-approval process. This diagnosis does not authorize post-review edits or unrelated redesign.

### Architecture-intent review with the architect

After the two axes, run one more fresh-context reviewer on a strong model (role map: slice reviewer) that checks the change against the architecture's *intent*, not its letter, together with the phase's architect. Its prompt carries:

- the snapshot and the two axis reports, so it does not redo them;
- the focus: each block in the right layer, block boundaries and public entries, composition, over-engineering the change introduced (facades, wrappers, needless narrow types, defensive checks), and leftovers (dead code, stale names, docs describing the old structure);
- how to reach the architect: named sender in every message, longer answers appended to `answers/final-architecture.md`, few batched questions;
- the order of work: first ask the architect what it considers most important to check and which decisions it regards as risky or provisional; then review the whole change; then put every finding it is unsure of to the architect before reporting;
- the report at `final-review/architecture.md`: each finding with `file:line`, classification (confirmed defect / architect-confirmed / suggestion / not a defect after consultation), severity, recommended fix, and the architect answers that shaped the verdicts.

The architect holds what the documents leave implicit, so this pass catches what a cold reviewer files as taste or misses: a feature that is a facade around one module call, composition bookkeeping that grew without need.

**Exit:** one review round is complete, every finding is triaged, and no post-review edits were made. Further edits or reviews require user direction.

## Hand off

Report:

- implementation summary and changed public behavior/contracts;
- acceptance table and validation commands/outcomes;
- triaged review findings and recommended decisions;
- remaining risks, deferred work, and existing-test conflicts;
- links to architecture, research, and continuation records;
- a short manual-review checklist;
- an updated verification plan using the `testing-scenarios` skill: reconcile the architecture-stage recommendation against actual implementation and coverage, explaining any changes. Keep test-authoring approval for the later testing phase.

Explain the core process so the user can choose scenarios and necessary protections in that later phase.

Keep staging, commits, pushes, ticket closure, and test authoring for later user-authorized phases. Offer the separate testing workflow after human review. Finish with exactly one state: **implementation ready for human review** or **blocked/incomplete**. Known implementation defects or missing required verification prevent a readiness claim.
