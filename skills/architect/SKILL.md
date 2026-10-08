---
name: architect
description: "Consult an architect about unresolved design decisions during implementation or review; check an approved design before delegation; answer a request as the architect."
---

# Architect

The **architect** is the delegate that holds a phase's design decisions: a pi agent in its own tab (role map in `multi-agent-delegate`: architect) that lives for the whole phase. It reads only files, so what it knows lives in the record (`architecture.md`, `progress.md`, the ADRs, the ticket), not in any session. Envelope, pointer, and waiting are `multi-agent-delegate`'s; this skill decides who asks what, and what an answer authorizes.

Every answer either explains the accepted design or proposes an amendment, and says which. An amendment takes effect once the design's owner (the design session, or the user when there is none) has the user's approval and records it in `architecture.md`.

## Dispatch and readiness check

Before handing an approved design to delegates, make sure the phase has an architect. When the user has not named one and has not declined one, dispatch it: lifecycle `ongoing`, `skills: architect`, its brief only the record files. Write its name and pane into `progress.md`, so any session of the phase finds it.

Its first request is a **readiness check**: the architect reads the files cold and lists every question an implementer would hit that they leave open. You, the design session, answer each from the design discussion and write the answer into `architecture.md`, so it lives in the record rather than in either session's context. Bring the user only the questions that would change a decision or were never discussed. When the user has declined an architect, run the same check yourself on the files and note in `progress.md` that the independent check was declined.

**Exit:** the architect (or your own check) finds no open questions, and `architecture.md` holds every answer.

## Ask the architect

Ask when a decision the work needs is missing, requirements conflict, ownership overlaps, or the implementation contradicts the design. Find the phase's architect in your brief or `progress.md`. When there is none, a delegated worker puts the question in its report for the coordinator; a session working with the user takes it to the user and offers an architect in one line. Dispatch one before a hand-off (above) or when the user asks for or accepts a consultation. Without a task directory, create one by the repository's artifact convention (otherwise `docs/design/<task>/`) and write the agreed context into the request: the architect reads only files.

Write each question with its evidence and a proposed answer, batched: in your report's Questions section when you have one, otherwise in the request. Send a request in the `multi-agent-delegate` envelope to `<task dir>/requests/<name>-q<n>.md`, reply-to `<task dir>/answers/<name>-q<n>.md` (or the paths your phase names), with the task "answer the questions in `<path>` and add each decision to `progress.md`". Send the pointer and wait as `multi-agent-delegate`'s `tabs.md` (step 6) describes for your harness. Use absolute paths throughout: askers may run in other worktrees.

The named sender and answer file keep concurrent questions apart: pi folds a prompt that arrives mid-turn into the running turn, so one turn may answer several askers in any order. The answer file's end marker is the signal that your question is answered.

Pause the work an answer affects; independent work continues. A question the architect cannot settle goes to the user through the session that talks with them (a worker's report reaches its coordinator), and the user's answer is recorded in `progress.md`.

## Act as the architect

When a request names you the architect:

- Keep your context for decisions: get code evidence from read-only research subagents (role map: research, no `skills:`) rather than reading source yourself.
- Answer each question in the reply file, marked as explanation or proposed amendment, with the evidence behind it.
- Record each decision in `progress.md`. `architecture.md` belongs to the design's owner.
- For a question the record cannot settle, say what would settle it; it goes back to the user through the asker's phase.
