# Exemplar: WE.DROP / dealmedia

This is a **worked reference** for a full User → Release → Moderation → Admin → Delivery stack. Use it to judge *depth* and *SSOT shape*, not as defaults for other repos.

## Why this exemplar

- Clear status machine with priority lane
- Paywalls (subscription + per-release AI fee) gated before moderation
- Own ISRC registrant issuance on accept
- Edit-request subsystem when statuses lock
- DDEX delivery docs + ops userscript sidecar with hard boundaries
- Agent rules (`AGENTS.md`) summarizing cross-cutting contracts

## Layout (illustrative)

| Area | Path |
|------|------|
| API | `backend_api/` (Adonis 4) |
| Cabinet | `frontend_lkpo/` |
| Ops notes | `md/` |
| Flow SSOT | `md/RELEASE_FLOW.md` |
| Agent SSOT | root `AGENTS.md`, `frontend_lkpo/AGENTS.md`, `userscript/AGENTS.md` |
| Specs | `.trellis/spec/**` |

## Status strings observed in exemplar

`draft`, `moderation`, `priority_moderation`, `ok`, `rejected`

## Stage → exemplar anchors (checklist)

| Stage | Exemplar hints |
|-------|----------------|
| S1 | `frontend_lkpo/src/services/api.js` 401 → full reload `/login` |
| S2 | `ReleaseController` draft/update; upload limits `md/UPLOAD_LIMITS.md` |
| S3 | `md/PAYMENTS.md`, `md/AI_RELEASE_PAYMENTS.md`; RU royalty 70/30 vs 80/20 in AGENTS |
| S4 | `POST /user/send_release` |
| S5 | Admin moderation lists; `POST /admin/assign_isrc` |
| S6 | `accept_release` / `reject_release`; ISRC `RUAGY`+YY+serial via `IsrcGenerator` |
| S7 | status `ok`; plus upsell lifecycle spec |
| S8 | `md/EDIT_REQUESTS.md` |
| S9 | `md/com_ddex.md`, `dmb_ddex.md`, … |
| S10 | `userscript/` parser → Sheets → Broma; parser does not issue codes |

## High-severity facts **in this product only**

Do **not** paste into other projects:

- AI fee **149 ₽**; Plus consumption `consumed_at`
- YooKassa verify via API GET + parsed body (no rawBody requirement)
- ISRC registrant **RU-AGY**; Broma is not the issuer for those codes
- Royalty messaging split RU vs eng_landing “100%”
- Quarterly report calendar dates in AGENTS

## When profiling dealmedia

Auditors may treat this file as a depth checklist and still confirm every line against the workspace. Flow SSOT to maintain: `md/RELEASE_FLOW.md`.
