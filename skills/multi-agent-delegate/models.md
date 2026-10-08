# Choosing the harness, model and thinking

Every dispatch names three things: the **harness**, the **model**, and the **thinking level**. Take all three from the role map below and pass all three explicitly on the command line. An explicit flag makes the launch deterministic: nothing depends on `modelThinkingLevels` in pi settings or on the session's own effort default.

The user's words override the map («на сол», «луну на хай», «опусом»). A role the map does not cover: propose a row in one line and ask the user before launch.

## Role map

| Role | Harness | Model | Thinking |
|---|---|---|---|
| Implementation slice, test writing | pi | `openai-codex/gpt-6-luna` | `max` |
| Heavy refactor or subtle slice (user approves the step-up in the execution plan) | pi | `openai-codex/gpt-6.1-sol` | `medium` |
| Hardest slice, when the user picks Claude | claude tab | `opus` | `high` |
| Read-only research, investigation, standards/spec review | pi | `openai-codex/gpt-6-luna` | `max` |
| Architect: design questions and decisions | pi | `openai-codex/gpt-6-astra` | `low`; `medium` when the user asks |
| Slice brief author (one-shot) | Agent tool | `sonnet` | `medium` |
| Slice reviewer: diff against brief (one-shot) | Agent tool | `opus` | `high` |
| Coordinator of a delegated phase | claude tab | session default | session default |

Luna always runs at `max`: it is cheap enough that a lower level saves nothing. Codex is available as a harness but is used only when the user names it.

## Thinking flags per harness

| Harness | Model flag | Thinking flag | Levels |
|---|---|---|---|
| pi | `--model <provider/id>` | `--thinking <level>` | off, minimal, low, medium, high, xhigh, max |
| claude tab | `--model <alias>` | `--effort <level>` | low, medium, high, xhigh, max |
| Agent tool | `model` parameter | `effort` parameter | low, medium, high, xhigh, max |

This map is the explicit instruction the Agent tool's `effort` parameter requires: set `effort` from it.

The pi status bar confirms what is running (`<model> • thinking <level>`); read it with `herdr agent read <name> --source visible` when a launch looks wrong.
