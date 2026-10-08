---
name: multi-agent-delegate
description: "Delegating work to another agent. Read it before spawning any subagent or launching an agent in a Herdr tab; when the user wants a non-Anthropic model (GPT/Codex, Gemini, …) or names a pi model (luna, sol, astra; «через луну», «через gpt»); when the user wants an agent in a separate tab («в отдельной табе»); and when following up with or closing a delegate."
---

# Delegating to agents

A **delegate** is an agent doing work for you: a subagent, or an agent running in its own Herdr tab. Every request to it carries an envelope and gets exactly one reply, written to a file.

## Choose the delegate

Take the harness, model and thinking for the role from [models.md](models.md); the user's words override it.

- **Subagent** (Agent tool): Claude work whose result you only need back, such as research, a brief, or a one-pass review. It costs no tab and returns by itself.
- **pi tab**: any non-Anthropic model.
- **Claude tab**: a Claude session the user wants to watch or talk to directly.

For a tab, follow [tabs.md](tabs.md) from here on: it covers the lifecycle, launch, sending, waiting, and closing.

## Write the request

The delegate starts cold: it knows only what the request says. Make it self-contained:

- the **envelope**: `from: <your name> (<harness>, Herdr pane <$HERDR_PANE_ID>)`, `reply-to: <absolute reply path>`, and `skills: <skills to run>`: the role's own skill where the dispatching skill names one, plus any skill the task needs that you choose (`tdd`, a repository skill);
- the goal and a checkable done criterion;
- the context it cannot find by looking: relevant paths, decisions already made, what the user wants;
- boundaries: which files it may change, and that it must not commit, push, or touch files outside the task;
- questions: asked in writing, in the reply's Questions section or as a request of their own to the answerer the phase names (the architect). Herdr cannot see an interactive dialog, and a prompt typed into one silently picks its default option;
- the reply: "Write your complete reply as Markdown to `<reply-to>`: a summary of at most 20 lines first (verdict or status, changed files, open questions), details below. Its last line must be exactly `<!-- end of reply -->`. Then end your turn with a one-line final message." A subagent's final message is its return value instead: "Return at most five lines: the reply path and the verdict."

The **end marker** is the only sign that a reply is complete. Herdr's `idle`/`done` and Claude's idle notice fire while a delegate's background job still runs, and a file's first write may not be its last.

Every request gets a fresh reply path, so a marker found there always belongs to that request; a follow-up names the previous reply as input. Put request and reply files in the task directory when one exists (`requests/<name>-<n>.md`, `replies/<name>-<n>.md`, or the paths the phase names), otherwise in your scratchpad, always as absolute paths. A tab delegate gets the request as a file plus a one-line pointer to it; a subagent gets it as its prompt.

## Wait for the reply

- Subagent: its result arrives by itself, as a tool result or a completion notification.
- Tab delegate: `scripts/wait-report`, run as [tabs.md](tabs.md) describes. It is the one wait. `herdr agent prompt --wait`, `herdr agent wait`, and loops of your own end on lifecycle states, before the reply is final.

Read the reply's summary (`head -n 20 <reply>`), more only where you need it, and relay what matters to the user.
