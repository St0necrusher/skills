# skills

Agent skills for a design → delegate → review workflow: a human designs with one agent, a thin coordinator cuts the approved work into slices, cheap workers implement them, and a strong reviewer gates each slice.

## Skills

| Skill | Invoked by | What it does |
|---|---|---|
| `build` | user | Design a feature together, delegate implementation, reconcile, review once |
| `impl` | user | Implement one ticket for human review, without tests or commits |
| `tests` | user | Choose and write durable tests after human review |
| `delegating-slices` | model | Coordinator workflow: roles, slices, briefs, architect questions, review gate |
| `implement-slice` | model | Worker side of one delegated slice |
| `testing-scenarios` | model | Pick critical, optional and excluded test scenarios |
| `multi-agent-delegate` | model | Run delegates in Herdr tabs (pi or Claude); role map of harness, model and thinking in `models.md`; waiting scripts in `scripts/` |

## Install

```bash
npx skills add St0necrusher/skills
```

The workflow also reaches skills from [mattpocock/skills](https://github.com/mattpocock/skills) (`domain-modeling`, `grilling`, `research`, `code-review`, `writing-for-agents`):

```bash
npx skills add mattpocock/skills
```

`multi-agent-delegate` needs [Herdr](https://github.com/herdrdev/herdr) and its `herdr` skill. Adjust `skills/multi-agent-delegate/models.md` to the models your subscriptions offer.
