---
name: release-flow-docs
description: >
  Adaptive audit/update of docs for any music / release-distribution product with a
  User → Release → Moderation → Admin → Release flow, AND local backend/API scaffolding
  for that flow. Use whenever the user mentions release flow, moderation queue,
  distributor docs, ISRC/UPC/DDEX docs drift, RELEASE_FLOW, syncing specs after code
  changes, OR asks to build/generate/scaffold/собрать a local backend/API/бек for a
  music distributor (Adonis or Nest, MariaDB 11 or Postgres, local or S3-compatible
  storage) — even casually («дока устарела», «карта релиза», «собери бек», «scaffold
  api»). Docs full sync uses CreateWorkflow; backend scaffold uses an orchestrator
  subagent plus framework/database/storage workers after AskUserQuestion for stack
  choices. Do not hardcode one repo's paths; do not invent Agent-only fan-out when the
  user asked for a workflow on a full docs pass.
---

# Release Flow Docs

Two capabilities, one domain:

1. **Docs sync** — keep documentation truthful against the live  
   **User → Release → Moderation → Admin → Release (delivery)** path.
2. **Backend scaffold** — locally generate an API skeleton for **S1–S6** with user-chosen
   framework, database, and file storage, driven by an **orchestrator subagent** and
   specialized workers.

The skill is **adaptive**: discover each repo’s layout. WE.DROP / dealmedia is the
**reference exemplar** of depth (`references/exemplar-wedrop.md`), not a hard dependency.
Never copy exemplar fees, ISRC prefixes, or hostnames into another product.

## Operating modes

Loading the skill does **not** authorize mass edits or codegen.

| Mode | When | What to do |
|------|------|------------|
| **Guidance** | Flow questions, “where is X documented?” | Answer from flow SSOT + `path:line`. No workflow, no scaffold. |
| **Targeted sync** | Named doc topics | Light discovery; edit named docs (+ flow SSOT if stages change). |
| **Full sync** | Explicit full docs pass | **CreateWorkflow** per `references/workflow-contract.md`. |
| **Scaffold backend** | «собери backend/api/бек», scaffold/generate local distributor API | Follow `references/scaffold-backend.md`: clarify stack → **`scaffold-orchestrator`** → workers. |

If docs vs scaffold is ambiguous, ask one question before writing files.

## Non-negotiables (all modes)

1. **Code is authority for behavior** when syncing docs (unless the user is changing product rules).
2. **Discover before you prescribe** in existing repos (`references/discovery.md`).
3. **Logical stages S1–S10** are stable ids; map or mark `absent` — do not invent product surface.
4. **Flow SSOT path is negotiable**; one home only.
5. **Full docs sync → CreateWorkflow** (load `dynamic-workflows` first).
6. **Scaffold → ask for missing stack choices**; never silently pick Adonis vs Nest, MariaDB vs Postgres, or local vs S3.
7. **No silent averaging** of code vs docs; findings use `update-doc` \| `flag-product` \| `leave`.
8. **High-severity classes** (money, auth, identifiers, delivery, status machine) need confirmation on docs sync.
9. **User-facing prose** in the user’s language; paths/ids as in the repo.
10. **Do not commit, migrate production, or deploy** unless the user explicitly asks for local migrate/up as part of scaffold.
11. **Do not paste exemplar-specific facts** into foreign docs or scaffolds.

## Progressive disclosure

| Need | Read |
|------|------|
| Profile an unknown repo | `references/discovery.md` |
| Logical stages & audit prompts | `references/flow-map.md` |
| Doc ownership patterns | `references/doc-inventory.md` |
| Docs CreateWorkflow contract | `references/workflow-contract.md` |
| **Backend scaffold + subagents** | **`references/scaffold-backend.md`** |
| WE.DROP depth checklist | `references/exemplar-wedrop.md` |

---

## Docs sync (summary)

Canonical stage ids, discovery, inventory, and workflow topology: see the reference files above.
Modes Guidance / Targeted / Full sync behave as in prior revisions: code wins, confirm money/auth/identifier claims, refresh flow SSOT on stage changes.

Finding shape:

```text
stage: S4
doc: <path or "unowned">
code: <path:line or "">
problem: one sentence
severity: high|medium|low
action: update-doc | flag-product | leave
evidence: short quote
```

---

## Scaffold backend mode

**Trigger:** user asks to build/scaffold/собрать a local **backend / API / бек** for a music
distributor or release-moderation product (with or without `/release-flow-docs`).

**Depth default:** S1–S6 skeleton only (auth, drafts, submit, admin queue, accept/reject).
Not full WE.DROP (no DDEX/userscript/payments unless user expands scope).

### Subagents (call by role)

| Name | Role | When |
|------|------|------|
| **`scaffold-orchestrator`** | Plan, dispatch, consistency, handoff README | **Always** first; user may say «запусти оркестратор» / «scaffold orchestrator» |
| **`scaffold-framework`** | App + routes/controllers for S1–S6 | After brief; `adonis` \| `nestjs` |
| **`scaffold-database`** | Config, compose, migrations | `mariadb11` \| `pgsql` |
| **`scaffold-storage`** | Upload adapter | `local` \| `s3` (S3-compatible) |
| **`scaffold-flow-doc`** | Optional flow SSOT for the new API | If user wants docs in the same pass |

Parent should **name these agents** when using the Agent tool (`description` / prompt identity) so the user can re-invoke one worker («доделай только storage на s3»).

### Required choices

Before writers run, collect via `AskUserQuestion` if missing:

1. **Framework:** AdonisJS \| NestJS  
2. **Database:** MariaDB 11 \| PostgreSQL  
3. **Files:** local disk \| S3-compatible  
4. **Target directory:** ask (e.g. `backend`, `backend_api`, `apps/api`) — do not overwrite a non-empty tree without confirm  

Full contracts, endpoint expectations, definition of done: **`references/scaffold-backend.md`**.

### Execution style

- Default: parent + **Agent** fan-out (orchestrator → DB/storage parallel → framework).  
- **CreateWorkflow** only if the user explicitly wants a workflow for the scaffold.  
- Install deps / `docker compose` locally when appropriate; no secrets in git.

---

## Guidance-mode answers

- Prefer repo flow SSOT; else reconstruct from code and note missing docs.  
- Cite `path:line`.  
- Offer targeted sync, full sync, or scaffold — do not silently edit or generate.

## After success

- **Docs:** `Last verified` on flow SSOT; optional SaveWorkflow if user wants repeats.  
- **Scaffold:** hand off env, migrate, start commands; point at status constants and upload adapter.
