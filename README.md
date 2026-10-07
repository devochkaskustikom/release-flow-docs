# release-flow-docs

Agent skill for **music / release-distribution** products:

1. **Docs sync** — keep documentation truthful against the live  
   **User → Release → Moderation → Admin → Release (delivery)** path.
2. **Backend scaffold** — generate a local API skeleton for stages **S1–S6** with user-chosen framework (Adonis \| Nest), database (MariaDB 11 \| PostgreSQL), and file storage (local \| S3-compatible).

The skill is **adaptive**: it discovers each repository’s layout. WE.DROP / dealmedia is a **depth exemplar** only (`references/exemplar-wedrop.md`) — never copy its fees, ISRC prefixes, payment providers, or hostnames into another product.

## Install

Clone into a skills discovery path (pick one):

```bash
# User-wide (recommended)
git clone https://github.com/devochkaskustikom/release-flow-docs.git \
  ~/.agents/skills/release-flow-docs

# Or project-local
git clone https://github.com/devochkaskustikom/release-flow-docs.git \
  .agents/skills/release-flow-docs
```

On Windows (Git Bash / PowerShell), `~` is your user home. Restart or reload the agent so it rediscovers skills.

Directory name must stay `release-flow-docs` (matches frontmatter `name`).

## When it triggers

Load via `/release-flow-docs` or natural language, for example:

- «дока устарела», «синхронизируй спеки», «карта релиза», RELEASE_FLOW
- «собери локально backend для музыкального дистрибьютора»
- «scaffold api: nest + postgres + s3»

## Modes

| Mode | Behavior |
|------|----------|
| **Guidance** | Answer from flow SSOT + `path:line`. No mass edits. |
| **Targeted sync** | Edit named docs (+ flow SSOT if stages change). |
| **Full sync** | **CreateWorkflow** (load `dynamic-workflows` first) — see `references/workflow-contract.md`. |
| **Scaffold backend** | Ask stack + target dir → orchestrator + framework/DB/storage workers — see `references/scaffold-backend.md`. |

## Layout

```text
release-flow-docs/
├── SKILL.md                         # entrypoint (name + description + modes)
├── README.md
├── LICENSE
├── references/
│   ├── discovery.md                 # RepoProfile probing
│   ├── flow-map.md                  # S1–S10 audit prompts
│   ├── doc-inventory.md             # ownership patterns
│   ├── workflow-contract.md         # CreateWorkflow for docs sync
│   ├── scaffold-backend.md          # scaffold orchestrator + workers
│   └── exemplar-wedrop.md           # WE.DROP depth checklist (not defaults)
└── assets/
    ├── finding-example.md
    └── scaffold-brief-example.md
```

## Requirements

- An agent host that loads skills from `.agents/skills/` or `~/.agents/skills/` (e.g. ZCode).
- Full docs sync: `dynamic-workflows` skill available for `CreateWorkflow`.
- Scaffold: Agent tool + `AskUserQuestion` for stack choices.

## Non-negotiables (summary)

- Code is authority when syncing docs (unless the user is changing product rules).
- Discover before prescribing; map stages S1–S10 or mark `absent`.
- Do not invent Agent-only fan-out when the user asked for a workflow on a full docs pass.
- Do not paste exemplar-specific money / ISRC / hostname facts into foreign repos.
- Do not commit, migrate production, or deploy unless the user explicitly asks for local migrate as part of scaffold.

## License

MIT — see [LICENSE](./LICENSE).
