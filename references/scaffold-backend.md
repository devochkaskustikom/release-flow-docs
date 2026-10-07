# Backend scaffold — orchestrator + stack subagents

Builds a **local music-distributor API skeleton** for stages **S1–S6** (auth, draft release/tracks, submit→moderation, admin accept/reject, status machine) with user-chosen framework, database, and file storage.

Not a clone of WE.DROP. Exemplar (`exemplar-wedrop.md`) is a depth checklist only — do not copy fees, ISRC registrant, hostnames, or payment providers unless the user asks.

## When this mode runs

Parent enters **scaffold mode** when the user (after loading `/release-flow-docs` or via skill trigger) asks to **create / generate / scaffold / собрать** a local **backend / API / бек** for a music distributor / release moderation product.

Examples (any language):
- «собери мне локально backend для музыкального дистрибьютора»
- «scaffold api: nest + postgres + s3»
- «подними каркас модерации релизов на Adonis и MariaDB»

If the request is only about docs sync, stay in docs modes — do not scaffold.

## Choices (required before writers run)

If the user did not specify all four, use `AskUserQuestion` (do not guess):

| Key | Options |
|-----|---------|
| `framework` | `adonis` (AdonisJS 4.x-style Lucid IoC) \| `nestjs` (NestJS + TypeORM or Prisma — pick one ORM in the ask and stick to it) |
| `database` | `mariadb11` \| `pgsql` |
| `storage` | `local` (disk under `storage/` or `uploads/`) \| `s3` (S3-compatible: endpoint, bucket, key prefix; MinIO-friendly) |
| `targetDir` | Workspace-relative path; if omitted, ask (e.g. `backend`, `backend_api`, `apps/api`). Refuse to overwrite a non-empty project tree without explicit confirm. |

Optional (ask only if relevant):
- `packageManager`: npm \| pnpm \| yarn (default npm)
- `auth`: JWT bearer (default) vs session cookie
- `language`: JS vs TS — **Nest → TS**; **Adonis 4 exemplar style → JS** unless user wants AdonisJS 5+/TS (then say so and scaffold TS)

Record answers in a `ScaffoldBrief` passed to every subagent.

```text
ScaffoldBrief:
  productName, targetDir, framework, database, storage,
  stages: [S1..S6],
  packageManager, auth, language,
  notes[]
```

## Agent roster

All agents may read the skill references. **Only the roles below write application code**, and only under `targetDir` (+ root README link if user agrees).

### 1. Orchestrator — `scaffold-orchestrator`

**Invoke:** parent always starts scaffold by delegating planning to this agent (or acting as this persona itself in-process). User-facing name: «оркестратор сборки API» / «API scaffold orchestrator».

Responsibilities:
1. Confirm `ScaffoldBrief` completeness.
2. Probe `targetDir` (empty / missing / conflict).
3. Emit a file plan: folders, env example keys, migrations list, route list for S1–S6.
4. Dispatch workers in the order below; merge results; run a final consistency pass.
5. Return a handoff: how to `.env`, migrate, start, and where the status machine lives.

Orchestrator **does not** invent stack choices. Escalate / ask user if brief is incomplete.

### 2. Framework worker — `scaffold-framework`

Owns: app bootstrap, module layout, auth stub (S1), HTTP routes/controllers for releases & admin moderation (S2–S6), status constants, ownership checks stub, README section for the API.

| `framework` | Expectations |
|-------------|--------------|
| `adonis` | `server` entry, `start/routes.js`, controllers under `app/Controllers/Http`, Lucid models, `use()` IoC style if Adonis 4; `.env.example` |
| `nestjs` | `main.ts`, modules (`Auth`, `Releases`, `Admin`, `Storage`), DTO validation, guards; TS strict-ish |

Must implement **logical** endpoints (names may vary, document them):

| Stage | Capability |
|-------|------------|
| S1 | register/login (or dev token stub), auth middleware/guard |
| S2 | CRUD release + tracks metadata; create draft status |
| S4 | submit → moves to moderation status |
| S5 | admin list moderation queue |
| S6 | accept → live/ok-equivalent; reject → rejected + reason |

Use **generic status strings** unless user specified: e.g. `draft`, `moderation`, `ok`, `rejected` (document in `TARGET/README.md` or `docs/RELEASE_FLOW.md`). Do not add `priority_moderation` / AI fees unless asked.

### 3. Database worker — `scaffold-database`

Owns: DB config, docker-compose snippet (optional but preferred), migrations/schema for:

- `users` (id, email, password hash, role `user|admin`, timestamps)
- `releases` (id, user_id, title, status, reject_reason nullable, timestamps)
- `tracks` (id, release_id, title, position, audio_key/path nullable, isrc nullable, timestamps)

| `database` | Expectations |
|------------|--------------|
| `mariadb11` | MariaDB **11.x** in compose; charset `utf8mb4`; compatible knex/TypeORM/Prisma driver |
| `pgsql` | PostgreSQL 16-ish in compose; UUID optional — prefer integer IDs for simpler Adonis 4 parity unless user wants UUID |

Coordinates with framework worker on ORM choice (do not install two ORMs).

### 4. Storage worker — `scaffold-storage`

Owns: upload abstraction used by track audio (and optional cover):

| `storage` | Expectations |
|-----------|--------------|
| `local` | Root dir from env (`STORAGE_ROOT`); serve or signed-path stub; gitignore uploads |
| `s3` | S3-compatible client config: `S3_ENDPOINT`, `S3_REGION`, `S3_BUCKET`, `S3_ACCESS_KEY`, `S3_SECRET_KEY`, `S3_FORCE_PATH_STYLE`; put object + delete; no real credentials committed |

Expose a small port/adapter the framework worker calls (`Storage.put`, `delete`, `publicUrl` / `getSignedUrl`).

### 5. (Optional) Flow docs worker — `scaffold-flow-doc`

If the user also wants docs, or orchestrator recommends it: write `docs/RELEASE_FLOW.md` or `md/RELEASE_FLOW.md` **for the new API only**, using discovered statuses. Skip if user declined docs.

## Execution order

```text
AskUserQuestion (gaps)
    → scaffold-orchestrator (plan)
        → parallel when possible:
            scaffold-database  ⎤
            scaffold-storage   ⎦  (interfaces first)
        → scaffold-framework  (consumes DB + storage adapters)
        → orchestrator consistency pass (+ optional scaffold-flow-doc)
```

Prefer **Agent** tool fan-out from the parent for scaffold (interactive, path conflicts). Use **CreateWorkflow** only when the user explicitly asks for a workflow / «через workflow».

If using CreateWorkflow, phases (RU example):  
`Уточнить стек и путь` → `Спланировать каркас` → `Собрать БД и storage` → `Собрать framework и маршруты S1–S6` → `Проверить согласованность и README`.

## Write boundaries

- Write under `targetDir` only (+ compose at repo root only if user ok and no existing compose conflict).
- Do not modify unrelated packages in a monorepo.
- Do not commit secrets; `.env.example` only.
- Do not run destructive docker volumes without asking.
- `npm install` / compose up: OK when user asked to scaffold locally; prefer generating lockfile via install in `targetDir`.

## Definition of done (S1–S6 skeleton)

1. App boots with documented command.
2. Migrations create users/releases/tracks.
3. Authenticated user can create draft release + track row + upload path/key.
4. Submit transitions status to moderation; non-owner blocked.
5. Admin can list moderation, accept (→ live status), reject (reason).
6. README lists env vars, statuses, and example curl/httpie.
7. Storage backend matches choice (local dir or S3 env).

## Anti-patterns

- Generating both Adonis and Nest «just in case».
- Hardcoding WE.DROP YooKassa / RUAGY / 149₽.
- Implementing full DDEX/userscript (S9–S10) in the default skeleton.
- Overwriting existing `backend_api` without explicit confirm.
- Framework worker embedding raw SQL that ignores the database worker’s migrations.
