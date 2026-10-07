# release-flow-docs

Agent skill / plugin for **music / release-distribution** products:

1. **Docs sync** — keep documentation truthful against the live  
   **User → Release → Moderation → Admin → Release (delivery)** path.
2. **Backend scaffold** — generate a local API skeleton for stages **S1–S6** with user-chosen framework (Adonis \| Nest), database (MariaDB 11 \| PostgreSQL), and file storage (local \| S3-compatible).

The skill is **adaptive**: it discovers each repository’s layout. WE.DROP / dealmedia is a **depth exemplar** only (`skills/release-flow-docs/references/exemplar-wedrop.md`) — never copy its fees, ISRC prefixes, payment providers, or hostnames into another product.

## Install

### A. Bare skill (any Agent Skills host)

Works with ZCode, Claude Code, Cursor, Codex, OpenCode, and similar clients that scan `~/.agents/skills/`:

```bash
git clone https://github.com/devochkaskustikom/release-flow-docs.git /tmp/release-flow-docs
mkdir -p ~/.agents/skills
cp -R /tmp/release-flow-docs/skills/release-flow-docs ~/.agents/skills/release-flow-docs
```

Or sparse-checkout only the skill folder:

```bash
git clone --depth 1 --filter=blob:none --sparse \
  https://github.com/devochkaskustikom/release-flow-docs.git
cd release-flow-docs
git sparse-checkout set skills/release-flow-docs
mkdir -p ~/.agents/skills
cp -R skills/release-flow-docs ~/.agents/skills/release-flow-docs
```

Project-local alternative: copy into `<repo>/.agents/skills/release-flow-docs/`.

Restart or reload the agent so it rediscovers skills. Directory name must stay `release-flow-docs` (matches frontmatter `name`).

### B. Claude Code — plugin marketplace

This repo is a Claude Code plugin marketplace (`.claude-plugin/marketplace.json`).

```text
/plugin marketplace add devochkaskustikom/release-flow-docs
/plugin install release-flow-docs@release-flow-docs
```

Then mention `/release-flow-docs` or ask for release-flow docs sync / distributor API scaffold.

### C. ZCode — Personal Plugin Marketplace

1. Clone the repo somewhere stable, e.g. `~/plugins/release-flow-docs`.
2. Open **Plugin Marketplace → Add → Add Plugin Marketplace**.
3. Paste the **repo root** (the folder that contains `marketplace.json`).
4. Open **Personal → release-flow-docs → Release Flow Docs → Install**.
5. New task → pick the skill in the composer / `/` Skills menu.

Manifest: `.zcode-plugin/plugin.json` (`skills: ./skills`). Root `marketplace.json` is the ZCode catalog; Claude’s catalog lives under `.claude-plugin/marketplace.json`.

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
release-flow-docs/                    # git / plugin / marketplace root
├── README.md
├── LICENSE
├── marketplace.json                  # ZCode Personal market catalog
├── .claude-plugin/
│   ├── plugin.json                   # Claude Code plugin manifest
│   └── marketplace.json              # Claude Code marketplace catalog
├── .zcode-plugin/
│   └── plugin.json                   # ZCode plugin manifest
└── skills/
    └── release-flow-docs/            # Agent Skill (SKILL.md + refs)
        ├── SKILL.md
        ├── references/
        │   ├── discovery.md
        │   ├── flow-map.md
        │   ├── doc-inventory.md
        │   ├── workflow-contract.md
        │   ├── scaffold-backend.md
        │   └── exemplar-wedrop.md
        └── assets/
            ├── finding-example.md
            └── scaffold-brief-example.md
```

## Requirements

- An agent host that loads skills from `.agents/skills/` / plugin skill roots (e.g. ZCode, Claude Code).
- Full docs sync: `dynamic-workflows` (or equivalent CreateWorkflow) available.
- Scaffold: Agent tool + `AskUserQuestion` for stack choices.

## Non-negotiables (summary)

- Code is authority when syncing docs (unless the user is changing product rules).
- Discover before prescribing; map stages S1–S10 or mark `absent`.
- Do not invent Agent-only fan-out when the user asked for a workflow on a full docs pass.
- Do not paste exemplar-specific money / ISRC / hostname facts into foreign repos.
- Do not commit, migrate production, or deploy unless the user explicitly asks for local migrate as part of scaffold.

## License

MIT — see [LICENSE](./LICENSE).
