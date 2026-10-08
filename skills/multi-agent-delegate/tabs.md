# Delegates in Herdr tabs

The Herdr mechanics of [SKILL.md](SKILL.md): a delegate running in its own tab, which you dispatch, keep working beside, and hear from when its reply is ready.

If `test "${HERDR_ENV:-}" = 1` fails, tell the user you are not inside Herdr and stop. For any Herdr command beyond the ones below, invoke the `herdr` skill. For requests between agents, this file replaces that skill's default of `agent prompt --wait` followed by `agent read`.

## Kind

- `pi`: any non-Anthropic model.
- `claude`: a Claude Code session the user wants to watch or talk to directly.

## Lifecycle

Decide at dispatch and state it to the user in one line together with the delegate's name, kind, model, and tab.

- `one-shot`: the result is a report the delegate hands back once: research, a question, a summary, a one-pass review. Close the tab after you have the result.
- `ongoing`: you will likely come back to the same session: implementation you will review and send fixes for, iterative refactoring, a review whose findings the delegate should then apply. Keep the tab open after each round; close it when the user accepts the work or says the delegate is no longer needed.

The user's wording overrides the default («разово», «потом поправим», «закрой после»). When the kind of task leaves it unclear, pick `ongoing` and ask at the end whether to close it: reopening a closed session costs the delegate's whole context, while an idle tab costs nothing.

Re-evaluate after each result: a `one-shot` result that reveals needed follow-up becomes `ongoing` (tell the user); an `ongoing` delegate whose work is accepted gets closed.

## Dispatch

1. **Resolve the model.** A short name the user gives (luna, sol, astra, …) resolves against `enabledModels` in `~/.pi/agent/settings.json`, newest version first. Ask only if the name matches nothing or several equally recent models.
2. **Name it.** One name serves as agent name, tab label, and (for `claude`) session name: `[a-z][a-z0-9_-]{0,31}`, model plus role: `luna-review`, `claude-impl`. Check `herdr agent list` for a clash.
3. **Open the tab** without stealing focus, in the current directory unless the task needs another:
   ```bash
   herdr tab create --workspace "$HERDR_WORKSPACE_ID" --cwd "$PWD" --label <name> --no-focus
   ```
   Take the pane from `.result.root_pane.pane_id`.

   A **fan-out** (several workers launched at once on parts of one task, such as parallel implementation slices) shares one tab instead: label the tab by the task, open it for the first worker, and give each further worker a split of an existing worker pane (`herdr pane split <pane> --direction right|down --no-focus`), taking its pane id from the result. Lay out every pane before sending any request: moving a pane later breaks Herdr's tracking of the agent in it (`agent_not_running`) while the agent keeps working. Any other delegate keeps its own tab.
4. **Start the agent.** A fresh tab's shell needs a moment: on `agent_pane_busy`, sleep 1 s and retry (up to ~15 tries).
   ```bash
   herdr agent start <name> --kind pi --pane <pane> --timeout 60000 -- --model <provider/id> --thinking <level> --exclude-tools ask_user_question
   herdr agent start <name> --kind claude --pane <pane> --timeout 60000 -- --name <name> --model <model> --effort <level>
   ```
   For `claude`, confirm the session is up: with `ListAgents` when you are Claude (it appears under `<name>`), otherwise with `herdr agent get <name>`.
5. **Write the request** as [SKILL.md](SKILL.md) describes, to a request file.
6. **Send the pointer and wait.** The pointer is one line: `Request <name>-<n> from <your name> (<harness>, Herdr pane <pane>): read <request file> and do what it asks.` Keep it on one line: multi-line text typed into Claude arrives as pasted content, whose instructions Claude does not act on.
   - `pi`, or any callee when you are pi or a subagent (a subagent's `SendMessage` goes out under its parent's address): `wait-report` sends the pointer through `herdr agent prompt` itself:
     ```bash
     <skill-dir>/scripts/wait-report <name> <reply path> --prompt "<pointer>"
     ```
   - `claude` when you are the main Claude session: `SendMessage` the pointer to `<name>`, then run `wait-report <name> <reply path>`. `SendMessage` marks the request as coming from another session rather than the user, and leaves the user's draft in that tab alone. A `[Cross-session delivery notice]` saying the message is held means the two sessions run in different permission classes: tell the user to approve it in the delegate's tab.

   Claude runs `wait-report` with Bash `run_in_background: true`; the exit wakes it. pi runs it in the foreground: its bash tool has no default timeout. A subagent runs it in the foreground with the Bash tool's `timeout` set to 600000 and the script's `--timeout 9`, and reruns it on `TIMEOUT`: a foreground Bash call stops at 10 minutes, and the default is 2. Pass `--quiet 0` for a delegate that talks with the user in its own tab, such as a coordinator, so the user's pauses do not end the wait. `<skill-dir>` is this skill's base directory; pass its absolute path to any delegate that runs the script itself.

   Tell the user the one-line dispatch summary, then keep working on anything independent or end the turn.

While an `ongoing` delegate implements, leave the files it owns alone; review after it settles.

## On completion

Branch on the script's last line:

- `REPORT_READY`: the reply is final; read its summary and relay what matters to the user.
- `BLOCKED` or `NOT_SENT_BLOCKED`: the delegate sits on a permission prompt; with `NOT_SENT_BLOCKED` it has not received your request yet. Read the prompt with `herdr agent read <name> --source visible`, show the user what the delegate asks and its options, and act only on the user's decision: they answer in the delegate's tab, or you send their choice with `herdr agent send-keys <name> <keys>`. Then rerun `wait-report`, with `--prompt` only after `NOT_SENT_BLOCKED`. Herdr's state is the truth about prompts: the delegate's own reply may not mention a prompt answered by keystroke.
- `NO_REPORT`: the delegate settled for the quiet period without finishing the reply. Read the screen: `herdr agent read <name> --source recent-unwrapped --lines 60`. If it is waiting on someone or still working in a way Herdr missed, rerun the wait; if it stopped without a reply, ask it once for the reply file. Resend a request only after the read shows it never arrived.
- `GONE`: the agent is no longer listed. Tell the user; resume or redispatch only on their word.
- `TIMEOUT`: read the screen. If it is still `working`, rerun the wait.
- `SEND_FAILED`: Herdr refused the pointer; the message says why.

Then apply the lifecycle: close a `one-shot` delegate, keep an `ongoing` one.

## Watching a fan-out

To guard a running fan-out against context overflow or the Codex 5h limit, run in the background:

```bash
<skill-dir>/scripts/watch-pi <name-prefix> [context-limit-k] [5h-left-percent]
```

It exits with one `ALERT` line; act on it, then re-arm it.

## Follow-up and closing

- Follow-up goes to the same delegate through step 6 with a new request file and a fresh reply path; the request names the previous reply as input. Find live delegates with `herdr agent list` (and `ListAgents` when you are Claude); the name identifies them across turns.
- Close by tab: take `tab_id` from `herdr agent get <name>`, then `herdr tab close <tab_id>`. Close only tabs you created as delegates. A running `wait-report` on that delegate then exits with `GONE`; that exit is expected.
- Several delegates may run in parallel: each gets its own name, tab, request files, and wait.
