# Choosing the harness, model and thinking

Every dispatch names three things: the **harness**, the **model**, and the **thinking level**. Take all three from the role map below and pass all three explicitly on the command line. An explicit flag makes the launch deterministic: nothing depends on `modelThinkingLevels` in pi settings or on the session's own effort default.

The user's words override the map («на сол», «луну на хай», «опусом»). A role the map does not cover: propose a row in one line and ask the user before launch.

## Roles

The role names are stable keys: skills name them, and a dispatch looks them up here.

- **Implementation slice**: a worker that implements one slice of approved work (`implement-slice`).
- **Test slice**: a worker that writes the approved tests of one slice (`tests`).
- **Research**: read-only investigation for a design session or the architect, so source stays out of their context.
- **Architect**: holds a phase's design decisions and answers delegates' questions (`architect`).
- **Slice brief author**: writes one slice's brief from the plan, so the coordinator never reads source.
- **Slice reviewer**: checks one slice's diff against its brief before the coordinator accepts it.
- **Axis reviewer**: one axis (Standards or Spec) of the review of a finished change (`final-review`).
- **Intent reviewer**: checks a finished change against the design's intent, with the architect (`final-review`).
- **Coordinator**: runs a delegated phase with the user: plan, dispatch, gate, commit (`delegating-slices`).

## Role map

| Role | Harness | Model | Thinking |
|---|---|---|---|
| Implementation slice | pi | `openai-codex/gpt-6-luna` | `max` |
| Test slice | pi | `openai-codex/gpt-6-luna` | `max` |
| Research | pi | `openai-codex/gpt-6-luna` | `max` |
| Architect | pi | `openai-codex/gpt-6-astra` | `low`; `medium` when the user asks |
| Slice brief author | Agent tool | `sonnet` | `medium` |
| Slice reviewer | Agent tool | `opus` | `high` |
| Axis reviewer | pi | `openai-codex/gpt-6-luna` | `max` |
| Intent reviewer | Agent tool | `opus` | `high` |
| Coordinator | claude tab | session default | session default |

Brief authors and all reviewers are one-shot. Implementation slice has two overrides under the same key:

- a heavy refactor or subtle slice, when the user approves the step-up in the execution plan: pi, `openai-codex/gpt-6.1-sol`, `medium`;
- the hardest slice, when the user picks Claude: claude tab, `opus`, `high`.

Luna always runs at `max`: it is cheap enough that a lower level saves nothing. Codex is available as a harness but is used only when the user names it.

## Thinking flags per harness

| Harness | Model flag | Thinking flag | Levels |
|---|---|---|---|
| pi | `--model <provider/id>` | `--thinking <level>` | off, minimal, low, medium, high, xhigh, max |
| claude tab | `--model <alias>` | `--effort <level>` | low, medium, high, xhigh, max |
| Agent tool | `model` parameter | `effort` parameter | low, medium, high, xhigh, max |

This map is the explicit instruction the Agent tool's `effort` parameter requires: set `effort` from it.

The pi status bar confirms what is running (`<model> • thinking <level>`); read it with `herdr agent read <name> --source visible` when a launch looks wrong.
