---
name: implement-slice
description: Implement a bounded delegated slice of an approved architecture; report evidence and escalate design contradictions to the coordinator.
---

# Implement a slice

Own the assigned slice, including requested corrections. The coordinator owns architecture and user agreement. Architecture artifacts are read-only for workers: propose amendments in the assigned result report, and let the coordinator update the authoritative design after the appropriate approval. This applies even to minor corrections and already-approved amendments. Work directly rather than delegating further.

## 1. Establish the contract

Read the assignment, approved architecture, applicable project instructions, and relevant source. Identify owned files, public seams, acceptance criteria, exclusions, dependencies, and existing validation commands. Inspect the worktree and preserve pre-existing and other workers' changes.

Ask about missing consequential decisions, conflicting requirements, or overlapping ownership before editing affected code: as a request to the answerer the assignment names (the architect), or in the report's Questions section when it names none.

**Exit:** the assigned behavior and ownership are explicit and dependencies are available or reported blocked.

## 2. Implement the minimum sufficient behavior

Follow approved responsibilities, contracts, and placement. Choose local implementation details independently. Reuse existing mechanisms and keep changes within the slice.

Add supporting machinery only for a concrete current requirement, contract, or accepted architecture rule. Keep natural failure behavior where sufficient. Speculative validation, fallback paths, compatibility layers, retries, extension points, and incidental cleanup are outside scope. Preserve required security and contract protections; surface any missing design decision about them.

Before adding a defensive check or reporting a defect, trace the scenario through actual callers, input provenance, and reachable lifecycle states. Distinguish demonstrated failures from hypothetical misuse. Locate the owner of the violated invariant: if the cause is broken ownership or data flow, propose correcting that layer rather than masking the symptom in consumers. Keep checks that enforce legitimate boundary contracts distinct from compensating patches.

When implementation reveals a design problem, put the evidence, affected assumption, impact, and smallest proposed remedy to the same answerer, or in the report. Pause affected work pending a decision; independent work may continue. An architectural workaround needs agreement rather than silent adoption.

Tests, fixtures, snapshots, test helpers, and test configuration remain unchanged. Report obsolete-test conflicts for the later testing phase; preserve checks rather than weakening them. Staging, commits, and pushes require separate explicit authorization.

**Exit:** each criterion has an implementation location or an explicit blocker, with no unapproved contract or ownership changes.

## 3. Verify and report

Run applicable existing checks from the assignment and repository scripts. Inspect the final diff for scope and unintended changes. Distinguish observed results from assumptions; record unrun checks and failures without claiming an unverified baseline.

Write the assigned durable report, ending with the end-marker line the assignment gives, containing:

- criteria mapped to implementation locations and status;
- changed files and public seams;
- checks, outcomes, and limitations;
- deviations and their approval status;
- discoveries, blockers, and questions for the coordinator.

Provide a continuation handle when the harness supports it. Keep the report sufficient for another worker to resume if live context is unavailable.

**Exit:** the coordinator has the result and evidence needed for direct inspection. Completion of the slice is not acceptance of the feature.

## 4. Respond to reconciliation

Explain disputed decisions using requirements and source evidence. Correct implementation errors in the owned slice and update the report. If a requested correction conflicts with observed behavior or the approved design, explain why and ask for a decision rather than blindly complying or silently refusing.

Retain valid work; implement the agreed correction at the smallest appropriate scope. Return fresh validation evidence for changed behavior.
