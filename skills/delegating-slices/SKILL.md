---
name: delegating-slices
description: "Coordinate a delegated phase: cut approved work into slices, dispatch brief authors, workers and reviewers, gate each slice. Use when a coordinator brief (`implementation-brief.md`, `tests-brief.md`) hands you a phase, or when a skill hands work to a coordinator."
---

# Delegating slices

The workflow for the parent of a delegated phase: implementation in `build`, test authoring in `tests`. Herdr mechanics (tabs, `agent start`/`prompt`/`wait`, fan-out panes, waiting scripts) and the role map of harness, model and thinking live in the `multi-agent-delegate` skill.

## Roles

The **coordinator** is thin: it plans, dispatches, gates, commits, and talks to the user. Every other job goes to a role that starts fresh, so the coordinator's context holds verdicts and decisions, not source.

- **Coordinator**: a Claude delegate in its own tab (`multi-agent-delegate`, kind `claude`). Writes no code and no briefs; its edits are the execution plan, `progress.md`, and commits.
- **Brief author**: a one-shot Agent subagent per slice, no `skills:`. It reads the plan row, the decisions, and the source the slice touches, and writes `briefs/<slice>.md`.
- **Worker**: implements one slice from its brief and validates it; `skills: implement-slice`, or `skills: tests` for a test slice.
- **Reviewer**: a one-shot Agent subagent per slice, no `skills:`. It checks the diff against the brief and returns a verdict.
- **Architect**: the delegate for design questions, named by the user or dispatched before the hand-off; the `architect` skill covers dispatching, asking, and acting as it.

Take each role's harness, model and thinking from the `multi-agent-delegate` role map; the user's choice for this phase overrides it.

## Coordinator

The dispatching session hands the phase to the coordinator. Its brief is a file in the task directory (`implementation-brief.md`, `tests-brief.md`) holding the accepted decisions, the sources to read, this workflow, boundaries, and the final report it owes. The brief states what is decided; how the work is executed is the coordinator's execution plan. The coordinator works with the user in its tab and reports once, at the end.

## Execution plan

Read the brief and its sources, then present the user your **execution plan** before editing any file:

- the cut: each slice's owned files, dependencies, and parallel or serial order;
- each worker's model, with the reason for any step-up from the role map default, and any skills added to its role's own;
- the phase-specific items its skill adds.

Revise the plan with the user. **Exit:** the user has explicitly approved the plan. Answered questions or silence are not approval.

## Slices

Partition the approved work into slices with explicit ownership and dependencies. Parallelize independent slices with minimal overlap; serialize dependent work. Cut each slice small enough to review in one sitting. Shared seams (contracts, stubs) and composition wiring are slices of their own.

Keep concurrent writers in disjoint files. Give each worker its own git worktree when workers would otherwise collide on shared outputs (a build directory, a test host); symlink dependency and test-host caches (such as `node_modules`) from the main checkout instead of reinstalling. The coordinator applies each worker's diff to the working branch.

Save each assignment, result, and open question in the task directory so continuation does not depend on a live session.

## Briefs

Workers get one common brief (shared rules, conventions, sources to read) plus a slice brief from the brief author: owned files, the seams it may use, the issue and decision lines that bear on the slice (quoted, so the worker need not reread the whole issue), expected observable behavior, completion criteria, exclusions, and validation commands. Skim each slice brief before dispatch.

The common brief carries:

- the write boundary, stated positively: the files the worker owns; a needed change anywhere else goes in the report instead;
- reading economy: excerpts with `rg` and `sed -n`, not `cat` over many files;
- validation: the full check suite, run in the foreground, so the worker's turn ends only when the result exists;
- questions: the architect's name and pane, to be asked directly through the `architect` skill, with the absolute paths of the task directory and the `multi-agent-delegate` scripts; a question the architect cannot settle goes in the report's Questions section with evidence and a proposed answer; no interactive prompts;
- the report: `reports/<slice>-<round>.md` is the worker's reply file for that round (each correction round gets the next one and names the previous report as input), opening with a summary of at most 20 lines (status, checks, deviations, questions), details below, and ending with the end marker (`multi-agent-delegate`).

## Questions to the architect

Workers ask the architect directly; the coordinator reads the resulting decisions in `progress.md` and the worker's report.

**When the user declines an architect**, workers end their turn with the questions in the report. The coordinator answers what the approved design and the recorded decisions already settle, writes the answer to `answers/<slice>-q<n>.md`, and points the worker at it. A question that would change a decision goes to the user, and its answer is recorded in `progress.md`.

## Review gate

For each finished slice, dispatch the reviewer with: the slice brief, the common brief, the architecture documents, `progress.md`, the worker's report, and the diff command. The reviewer returns a verdict in this shape:

- each completion criterion → `file:line`, or what is missing;
- `git diff` outside the owned files: empty, or each change listed;
- assertions unchanged or explicitly allowed; production code not bent to suit tests; fakes keep the real contract;
- violations of the repository's standards and the user's recorded preferences;
- workarounds and over-engineering, by the `final-review` skill's Evidence bar: each with `file:line`, or none;
- verdict: accept, or a numbered list of corrections.

Spot-check one `file:line` claim from the verdict. Send corrections within the approved design to the worker and review again until the verdict is accept. A proposed preparatory refactoring or other design amendment goes through the architect or the user first; update the plan and briefs when it is approved. Trust the worker's reported checks; the reviewer reads code, it does not rerun them.

**Shadow review.** Until the user switches a phase to reviewer-only, also review each diff yourself as before, and record in `progress.md` what each review found. A reviewer that misses something you caught needs a missing source added to its prompt, not a retry.

**Exit:** every slice is accepted through the gate or explicitly blocked, and integrated into the working branch.
