---
name: build
description: Design a feature together, delegate implementation, and reconcile the result before human review.
disable-model-invocation: true
argument-hint: "Ticket, specification, or feature"
---

# Build

Lead the design with the user; delegate implementation of the agreed solution. Architecture is a revisable agreement, not proof that the designer is right. Keep the core process understandable before adding supporting machinery.

## 1. Ground the task

Read the request and applicable repository instructions. For a ticket, follow the repository's issue-tracker instructions and read its requirements, comments, linked specifications, and dependencies. Load domain and architecture documentation required for the affected area. Inspect relevant source, public seams, existing tests, prototypes, and executable scripts.

Record acceptance criteria, exclusions, consequential ambiguities, and existing worktree changes. Preserve unrelated work; resolve overlapping ownership with the user before editing. For a request without a ticket, establish the same bounded requirements together rather than inventing a ticket prerequisite.

Create a local task directory using the repository's artifact convention; otherwise propose `docs/design/<task>/`. Maintain `architecture.md`, `progress.md`, and research reports there as work proceeds. Link existing authoritative material rather than copying it.

**Exit:** scope and blockers are explicit, existing work is distinguished, and the task's durable record exists.

## 2. Design together, top down

Product clarification establishes what to build; architectural discussion establishes how it will work. After clarifying requirements, present a concrete recommended design to the user before asking to approve or finalize the architecture. Offer alternatives only where a consequential trade-off exists, with a recommendation and reasons.

### Present the proposal

Walk the user through these views in manageable sections, starting with the high-level picture and then detailing its parts:

- **Modules and responsibilities:** a compact diagram of the proposed boundaries and dependency directions; name what each module owns and what remains outside it.
- **Domain model:** the existing and proposed entities, value objects, or other domain concepts actually needed, their relationships, significant state, and authoritative owners. Distinguish domain concepts from transport and UI representations; a concept does not automatically require a new class or file.
- **Data flow:** trace a concrete primary scenario from entry point through calls/events and state changes to the visible result. Name who reads, writes, and publishes data. Include relevant lifecycle and failure paths justified by the requirements.
- **Public seams:** describe the responsibilities and shape of the contracts connecting the parts, sufficiently to expose consequential decisions without writing function bodies.
- **Expected file structure:** show an approximate annotated directory/file tree, marking existing, changed, and new locations and their responsibilities. Reuse existing structure; explain when no new entity or file is needed.

Make these views visible in the conversation, not only in a saved document. Scale their depth to the task, but account for each view. Recursively detail parts that still contain architectural or product decisions. Invite questions and revise the proposal with the user before seeking approval.

### Plan verification with the design

Invoke the `testing-scenarios` skill and apply its architecture-design branch. Discuss the preliminary critical, optional, and excluded scenarios with the user alongside the proposed design. Use them to check observable contracts and ownership before implementation. Record the plan in the architecture artifact; this step plans verification without authoring test artifacts.

Maintain `architecture.md` as a draft during this discussion, with open questions and approval status explicit. Saving a draft preserves progress; it does not finalize the design. Agreement on product requirements or permission to write the document does not authorize implementation.

Explain decisions and trade-offs to the user in manageable sections. Resolve uncertainty through discussion rather than silent assumptions. When a decision needs pressure-testing, use a bounded grilling discussion focused on that decision; reach the available grilling skill when useful.

Research unknown facts before relying on them:

- Use `research` for primary-source documentation, APIs, and specifications.
- Use `source-audit` for implementation evidence from a concrete external repository at a pinned revision.
- Inspect relevant local prototypes for prior experiments; distinguish demonstrated behavior from production suitability.

Coordinate research from the parent rather than passing a delegating workflow wholesale to a child. Take research workers' model from the `multi-agent-delegate` role map. Save evidence, limitations, and unresolved questions in the task directory, with links from the design.

### Minimum sufficient design

Every mechanism must serve a current requirement, existing contract, or applicable architecture rule. Reuse existing mechanisms. Keep natural failures when an additional check adds no required behavior. Introduce validation, compatibility, retries, fallbacks, and abstractions only for a concrete present need; explain required security and contract protections during design. Defer hypothetical cases and incidental cleanup.

Design invariants into ownership: identify the authoritative owner of each mutable state and lifecycle transition, and how consumers receive consistent state. Prefer contracts and data flow that prevent invalid states over compensating checks scattered across consumers. Evaluate concrete failure paths in the actual system rather than inventing hypothetical consumers or unsupported scenarios.

Stop detailing when an implementer can proceed without making architectural or product decisions. Leave local function implementation choices to the implementer. The design must still be specific enough to check the result against it.

**Exit:** the user has seen and discussed the concrete proposal covering all five views above and its preliminary verification plan, consequential questions are resolved, and the artifact reflects the resulting design and plan. Ask for explicit approval of that design and authorization to implement; proceed only when both are given. Silence, lack of objections, product answers, or a completed document is not approval.

## 3. Delegate bounded slices

### Readiness check by the architect

Before handing off, make sure the phase has an architect (see the `delegating-slices` skill, Roles). When the user has not named one and has not declined one, dispatch it yourself: kind, model and thinking from the `multi-agent-delegate` role map, lifecycle `ongoing`, its brief only files (the approved architecture, `progress.md`, the ADRs, the ticket).

Its first task is a **readiness check**: read the files cold and list every question an implementer would hit that they leave open. Answer each from the design discussion and write the answer into `architecture.md`, so it lives in the record rather than in either session's context. Bring the user only the questions that would change a decision or were never discussed. **Exit:** the architect reports no open questions, and `architecture.md` holds every answer.

### Hand off to the coordinator

Hand the rest of step 3 and steps 4–5 to a coordinator as described in the `delegating-slices` skill; the brief is `implementation-brief.md`. The brief carries the approved design and requirements and names the architect; the coordinator's execution plan decides who implements and how the work is cut. Implementation adds:

- When the plan spans several owner areas, fan out **contract-first**: the first slice writes every shared seam as code (capability interfaces, extended contracts, and the minimal stubs that keep existing code compiling). Later slices depend only on those accepted contracts and run in parallel in the same worktree; the common brief says that type errors in other workers' files are expected.
- Wire composition (manifest, top-level composition) as a final slice after the others are reconciled.
- Workers use the companion `implement-slice` skill. Their brief names the approved architecture version and the relevant requirement references.
- Architecture artifacts are parent-only: workers may read them and propose changes in their reports, but must not edit them, even for minor or approved amendments. The parent records accepted changes after the appropriate approval.

**Exit:** each slice has an owner, bounded contract, durable handoff, and completed or explicitly blocked result.

## 4. Reconcile in the parent

Check the result against requirements and the approved architecture through the review gate of the `delegating-slices` skill. Account for every criterion and changed public seam.

For a discrepancy, ask the implementer why it exists and examine the evidence before deciding. Trace any claimed failure through real callers and reachable states; locate the responsible invariant owner before choosing a remedy. Correct broken ownership or data flow at its source rather than masking symptoms with consumer-side checks. Apply the reachability and root-cause evidence bar in [completion.md](completion.md) during reconciliation as well as final review:

- **Implementation error:** return a concrete correction to the worker, then inspect the result.
- **Minor justified deviation:** accept and update the architecture when behavior, public contracts, and module boundaries remain unchanged.
- **Architectural amendment:** discuss affected decisions with the user, update the design, and obtain approval before continuing affected implementation.
- **Foundational contradiction:** pause affected work and reassess the architecture with fresh eyes, potentially in a new session. Preserve valid work rather than automatically starting over.
- **Uncertain cause:** gather evidence before choosing a remedy.

Judge divergence by its impact on responsibilities, contracts, data flow, and dependent work—not line count. Both designer and implementer can be wrong. Record disproven assumptions separately from accepted decisions and proposed changes.

For a session transition, write a compact handoff linking the architecture, research, current changes, checks, worker reports/continuation handles, contradictions, open questions, and approval state. Record how to resume; a new session must be able to distinguish accepted design from unfinished proposals.

This reconciliation loop can repeat. It is distinct from the single final advisory review.

**Exit:** the parent has checked every criterion and architectural seam; deviations are resolved or explicitly blocked; the architecture reflects the accepted result.

## 5. Validate, review once, and hand off

Follow [completion.md](completion.md) for existing checks, one independent two-axis advisory review, triage, and the final report.
