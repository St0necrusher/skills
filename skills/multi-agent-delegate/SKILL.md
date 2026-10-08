---
name: multi-agent-delegate
description: "Delegate a task to an agent running in its own Herdr tab: a pi agent for non-Anthropic models (OpenAI/Codex/GPT, Gemini, …), or a Claude session the user can watch and talk to. Use when the user asks for a subagent, research, implementation, or review on such a model or names a pi model (luna, sol, astra; «через луну», «через gpt»), asks for an agent in a separate tab («в отдельной табе»), and when following up with or closing an already delegated agent. Requires HERDR_ENV=1."
---

# Delegating to agents in Herdr tabs

A **delegate** is an agent running in its own Herdr tab. You dispatch work to it, keep working while it runs, and get notified when it settles. Every delegate has a **kind** and a **lifecycle**.

If `test "${HERDR_ENV:-}" = 1` fails, tell the user you are not inside Herdr and stop. For any Herdr command beyond the ones below, invoke the `herdr` skill.

## Kind

- `pi`: any non-Anthropic model. Driven through Herdr: `herdr agent prompt` in, background `wait-report` on its answer file for completion, the answer file for the result.
- `claude`: a Claude Code session the user wants to watch or talk to directly. Driven through cross-session messaging: `SendMessage` in, `notify_when_idle` for completion, and the delegate sends its result back as a message.

A Claude task the user did not ask to see in a tab goes to the built-in Agent tool instead: it returns its result directly and costs no tab.

## Lifecycle

Decide at dispatch and state it to the user in one line together with the delegate's name, kind, model, and tab.

- `one-shot`: the result is a report the delegate hands back once: research, a question, a summary, a one-pass review. Close the tab after you have the result.
- `ongoing`: you will likely come back to the same session: implementation you will review and send fixes for, iterative refactoring, a review whose findings the delegate should then apply. Keep the tab open after each round; close it when the user accepts the work or says the delegate is no longer needed.

The user's wording overrides the default («разово», «потом поправим», «закрой после»). When the kind of task leaves it unclear, pick `ongoing` and ask at the end whether to close it: reopening a closed session costs the delegate's whole context, while an idle tab costs nothing.

Re-evaluate after each result: a `one-shot` result that reveals needed follow-up becomes `ongoing` (tell the user); an `ongoing` delegate whose work is accepted gets closed.

## Dispatch

1. **Resolve harness, model and thinking** from the role map in [models.md](models.md). A short name the user gives (luna, sol, astra, …) resolves against `enabledModels` in `~/.pi/agent/settings.json`, newest version first. Ask only if the name matches nothing or several equally recent models.
2. **Name it.** One name serves as agent name, tab label, and (for `claude`) session name: `[a-z][a-z0-9_-]{0,31}`, model plus role: `luna-review`, `claude-impl`. Check `herdr agent list` for a clash.
3. **Open the tab** without stealing focus, in the current directory unless the task needs another:
   ```bash
   herdr tab create --workspace "$HERDR_WORKSPACE_ID" --cwd "$PWD" --label <name> --no-focus
   ```
   Take the pane from `.result.root_pane.pane_id`.

   A **fan-out** (several workers launched at once on parts of one task, such as parallel implementation slices) shares one tab instead: label the tab by the task, open it for the first worker, and give each further worker a split of an existing worker pane (`herdr pane split <pane> --direction right|down --no-focus`), taking its pane id from the result. Lay out every pane before sending any brief: moving a pane later breaks its running `agent prompt --wait` (`agent_not_running`) while the agent keeps working. Any other delegate keeps its own tab.
4. **Start the agent.** A fresh tab's shell needs a moment: on `agent_pane_busy`, sleep 1 s and retry (up to ~15 tries).
   ```bash
   herdr agent start <name> --kind pi --pane <pane> --timeout 60000 -- --model <provider/id> --thinking <level>
   herdr agent start <name> --kind claude --pane <pane> --timeout 60000 -- --name <name> --model <model> --effort <level>
   ```
   For `claude`, confirm with `ListAgents` that the session appears under `<name>`; its first line also gives your own session name, the reply address.
5. **Write the brief.** The delegate starts cold: it knows only what the brief says. Make it self-contained:
   - who dispatched it: your session name and Herdr pane (`$HERDR_PANE_ID`);
   - the goal and a checkable done criterion;
   - the context it cannot find by looking: relevant paths, decisions already made, what the user wants;
   - boundaries: which files it may change, and that it must not commit, push, or touch files outside the task;
   - for `pi`: put every question in the final message instead of an interactive prompt. Herdr skips screen detection for pi, so a pi question dialog shows as `working`, `herdr agent wait` never returns `blocked`, and the delegate stalls silently;
   - the output: `pi`: "Write the complete result as Markdown to `<answer path>`, then end with a final message of at most five lines: the path and the verdict." Pick the answer path in the task directory or your scratchpad; `herdr agent read` cuts long answers, and reading a file costs only what you choose to read. `claude`: "When done, send the complete result with SendMessage to `<your session name>`, then stop." Research/review: findings; implementation: changed files and what is still open.

   For a brief longer than a screen, write it to a file in your scratchpad and send "Read <path> and do the task described there."
6. **Send it.**
   - `pi`: send the brief, then wait on the answer file with Bash `run_in_background: true`:
     ```bash
     herdr agent prompt <name> "<brief>"
     <skill-dir>/scripts/wait-report <name> <answer path> [timeout-min]
     ```
     `wait-report` is the one wait for pi. It returns only after the file changed and the agent stayed settled for 20 s, which covers the two cases that fool `--wait` and hand-written loops: a turn that ends while the delegate's background job still runs, and a delegate that rewrites its file after the first write. `<skill-dir>` is this skill's base directory; pass its absolute path to any delegate that runs the script itself.
   - `claude`: first arm the **blocked trap** with Bash `run_in_background: true`, then `SendMessage` to `<name>` with the brief and `notify_when_idle: true`. The idle notice never fires while the delegate sits on a permission prompt; the trap does:
     ```bash
     herdr agent wait <name> --until blocked --timeout 1800000
     ```

   Tell the user the one-line dispatch summary, then keep working on anything independent or end the turn: the notification wakes you.

While an `ongoing` delegate implements, leave the files it owns alone; review after it settles.

## On completion

### `pi`

Branch on the script's last line:

- `REPORT_READY`: the file is final; read it and relay what matters to the user.
- `NO_REPORT`: the delegate settled for 5 minutes without writing the file. Read the screen: `herdr agent read <name> --source recent-unwrapped --lines 60`. Resend the brief only after the read shows it never arrived.
- `TIMEOUT`: read the screen. If it is still `working`, re-arm the script in the background.

### `claude`

Two events arrive, usually in this order: the delegate's result as a cross-session message, then the idle notice. The blocked trap fires before them whenever the delegate stops on a permission prompt or question.

- Blocked trap returns `blocked`: read the prompt with `herdr agent read <name> --source visible`, show the user what the delegate asks and its options, and act only on the user's decision: they answer in the delegate's tab, or you send their choice with `herdr agent send-keys <name> <keys>`. Then re-arm the trap. Herdr's state is the truth about prompts: the delegate's own report may not mention a prompt answered by keystroke.
- Blocked trap returns `timeout` while the delegate is still `working`: re-arm it.
- Result message: relay what matters to the user.
- Idle notice with no result message: the delegate stopped without reporting. Read its screen with `herdr agent read <name> --source recent-unwrapped --lines 200`.
- `[Cross-session delivery notice]` saying the message is held: the sessions run in different permission classes. Tell the user to approve it in the delegate's tab.

Then apply the lifecycle: close a `one-shot` delegate, keep an `ongoing` one.

## Watching a fan-out

To guard a running fan-out against context overflow or the Codex 5h limit, run in the background:

```bash
<skill-dir>/scripts/watch-pi <name-prefix> [context-limit-k] [5h-left-percent]
```

It exits with one `ALERT` line; act on it, then re-arm it.

## Follow-up and closing

- Follow-up goes to the same delegate through step 6: `herdr agent prompt` plus background `wait-report` on the answer file for `pi`, blocked trap plus `SendMessage` with `notify_when_idle: true` for `claude`. Find live delegates with `herdr agent list` (and `ListAgents` for `claude`); the name identifies them across turns.
- Close by tab: take `tab_id` from `herdr agent get <name>`, then `herdr tab close <tab_id>`. Close only tabs you created as delegates. A `claude` delegate's blocked trap then exits with an error; that exit is expected.
- Several delegates may run in parallel: each gets its own name, tab, and notification.
