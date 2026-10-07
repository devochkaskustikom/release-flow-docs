# Discovery — build a RepoProfile before auditing

Every full or targeted sync starts here. Output is a structured profile used to size auditors and choose write paths. Do not skip this in an unfamiliar repo.

## What to detect

### 1. Product shape
- Monorepo vs single package; package managers; obvious apps (`api`, `web`, `admin`, `landing`, `worker`).
- Domain keywords in README/AGENTS: release, track, album, moderation, DDEX, ISRC, UPC, distributor, catalog, label.

### 2. Doc roots (first match wins per category; record all candidates)

| Category | Glob / probe patterns |
|----------|------------------------|
| Ops / product notes | `md/**/*.md`, `docs/**/*.md`, `documentation/**/*.md` |
| Agent rules | `AGENTS.md`, `**/AGENTS.md`, `CLAUDE.md`, `.cursorrules` |
| Spec / guidelines | `.trellis/spec/**/*.md`, `specs/**/*.md`, `adr/**/*.md` |
| Existing flow map | `**/RELEASE_FLOW.md`, `**/*release*flow*.md`, `**/moderation*.md` |

If multiple roots exist, prefer the one already linked from README or AGENTS.

### 3. Code anchors (search, don’t assume filenames)

Search (ripgrep-style) for signals; keep the winning paths in the profile:

| Signal | Example patterns |
|--------|------------------|
| Status enum | `moderation`, `priority_moderation`, `in_review`, `pending_review`, `approved`, `rejected`, `draft`, `published`, `live` |
| Submit | `send_release`, `submitRelease`, `submit_for_review`, `/moderation` |
| Accept/reject | `accept_release`, `reject_release`, `approveRelease`, `rejectRelease` |
| Identifiers | `ISRC`, `UPC`, `EAN`, `catalogNumber` |
| Payments | `webhook`, `yookassa`, `stripe`, `paypal`, `plus`, `subscription` |
| Delivery | `DDEX`, `ERN`, `FTP`, `delivery`, `distributor` |
| Auth | `Bearer`, `401`, `jwt`, `session` |

Record **observed status strings** exactly as in code — never normalize them to the exemplar’s names in docs for this repo.

### 4. Stage coverage

For each S1–S10: `present` | `partial` | `absent` with one evidence path.
`partial` = related code exists but no clear user/admin path.

### 5. Write policy

Default allow:
- Chosen doc root (`md/**` or `docs/**`)
- Root and package `AGENTS.md` / equivalent agent rule files
- Spec trees already used by the project

Default deny unless user expands:
- Marketing landing copy (except when it contradicts legal/royalty SSOT the user asked to fix)
- `dist/`, `build/`, generated clients
- `.env`, secrets, migrations **source** (docs *about* migrations OK)
- Unrelated packages (e.g. pure marketing site) when profile marks them out of scope

### 6. Language & locale
Infer UI/docs language from existing docs; reports follow the **user’s** language for the session.

## RepoProfile sketch (for workflow typing)

```ts
interface RepoProfile {
  /** Short product name from README or folder. */
  productName: string;
  /** Doc directories to treat as writable SSOT roots. */
  docRoots: string[];
  /** Chosen flow SSOT path (existing or to-create). */
  flowSsotPath: string;
  /** Agent-rule files found. */
  agentRuleFiles: string[];
  /** Spec/guideline trees. */
  specRoots: string[];
  /** Status strings observed in code. */
  releaseStatuses: string[];
  /** Map stage id → present|partial|absent. */
  stages: Record<string, "present" | "partial" | "absent">;
  /** Key code anchors: logical role → path. */
  anchors: Record<string, string>;
  /** Packages/dirs in scope for this run. */
  scopePackages: string[];
  /** Notes for auditors (odd layout, dual frontends, etc.). */
  notes: string[];
}
```

## Light discovery (guidance / targeted)

You do not need a full workflow node: parent (or one agent) still:
1. Glob doc roots + flow SSOT.
2. Grep status/submit/accept signals.
3. Proceed with only the surfaces the user named.

## Anti-patterns

- Assuming `frontend_lkpo` or `backend_api` exist.
- Writing WE.DROP royalty/ISRC numbers into a foreign repo.
- Creating both `md/RELEASE_FLOW.md` and `docs/RELEASE_FLOW.md` — pick one home.
- Marking a stage `present` from a comment or dead string match alone; prefer controller/route/UI usage.
