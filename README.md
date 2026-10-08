# skills

Agent skills for a design → delegate → review workflow: a human designs with one agent, a thin coordinator cuts the approved work into slices, cheap workers implement them, and a strong reviewer gates each slice.

## Workflow

```mermaid
flowchart TD
    U([User]) -->|/build ticket| D[Design session<br/>ground, design together,<br/>plan test scenarios]
    D -->|user approves design| RC[Architect readiness check<br/>architect reads docs cold,<br/>design session answers gaps]
    RC -->|implementation-brief.md| C[Coordinator<br/>execution plan: slices, owners, models]
    C -->|user approves plan| BA
    subgraph Phase["Whole phase"]
        AR[(Architect)]
    end
    RC -.->|spawns if none| AR

    subgraph Slice["Per slice"]
        BA[Brief author<br/>writes briefs/slice.md] --> W[Worker<br/>implements, validates in foreground]
        W <-->|questions / answer files| AR
        W -->|reports/slice.md| RV[Reviewer<br/>diff vs brief: verdict]
        RV -->|corrections| W
        RV -->|accept| CM[Coordinator commits,<br/>updates progress.md]
    end

    CM -->|next slice| BA
    CM -->|all slices done| FR

    subgraph Final["Review once"]
        FR[Validate] --> AX[Standards + Spec reviewers]
        AX --> AI[Architecture-intent reviewer]
        AI <-->|risks, unsure findings| AR
    end

    AI --> HR([User review, PR])
    HR -->|/tests| T[Same coordinator loop<br/>for test slices]
```

The coordinator stays thin: code and diffs are read only by one-shot roles (brief author, worker, reviewer), and the architect keeps its context for decisions by sending code research to read-only subagents.

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
