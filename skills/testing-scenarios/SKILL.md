---
name: testing-scenarios
description: "Choose which test scenarios to plan: critical now, optional, deliberately excluded. Use when planning verification during architecture design, or when a skill asks for a verification plan."
---

# Choosing test scenarios

Shared planning reference for architecture design and the separately authorized test-authoring phase. Applying this reference plans tests; it does not authorize writing them.

Ground recommendations in requirements, stable public seams, material regression risks, and existing coverage. During design, label proposed seams and unresolved assumptions; after implementation, verify them against the actual code and human-review decisions.

## Recommendation

Present three groups:

### Critical now

Include only scenarios whose failure would materially violate a requirement or create a likely regression:

- the primary user or caller path;
- an important domain invariant;
- a consequential failure mode;
- a fixed regression;
- lifecycle, ordering, concurrency, or ownership behavior required by the task;
- an integration boundary where failure is a concrete risk.

### Optional

Separate lower-risk edge cases and additional confidence that may be useful but is unnecessary now. Compatibility variations belong here only when grounded in supported behavior, not hypothetical consumers.

### Deliberately excluded

Identify implementation details, private helpers, trivial data plumbing, incidental collaborator calls, third-party behavior, and scenarios already adequately proved elsewhere that do not warrant new tests.

For every proposed scenario, name:

- observable behavior;
- requirement or risk protected;
- stable public seam;
- test level: behavioral, integration, adapter integration, or extension/end-to-end;
- why existing coverage is insufficient.

Use realistic inputs and reachable states/event orderings. A contract-breaking mock or imagined future caller does not establish a regression risk. Mark uncertain reachability as a research question rather than padding the critical set.

Prefer the narrowest stable public seam that proves the behavior and a few complete behavioral/integration scenarios over implementation-coupled unit tests. Recommending no new tests is valid when justified.

## During architecture design

Walk through the proposed critical scenarios with the user to check that the contracts expose meaningful outcomes and responsibilities are clear. If a scenario reveals broken ownership or data flow, revisit the design rather than adding test-only abstractions or compensating guards.

Record the preliminary recommendation with the architecture, linking existing coverage where relevant. Resolve consequential behavioral ambiguity before architecture approval. Approval of this plan or production implementation is not permission to author tests.

## After implementation and human review

Reconcile the preliminary recommendation, if present, with actual seams, existing coverage, discoveries, and accepted behavior. Explain additions, removals, or changed priorities rather than replacing the plan silently.

Ask the user to approve the critical set and select optional scenarios before writing tests. Completion means an explicitly approved, bounded scenario list; test authoring then follows the `tests` skill.
