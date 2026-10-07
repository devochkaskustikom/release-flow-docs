# CreateWorkflow contract — adaptive full docs sync

Parent **must** load `dynamic-workflows` before `CreateWorkflow`.
Script: plain TypeScript workflow facade (no `import` / `export` / `declare`).
Phase names, logs, report prose: **user’s language** (examples below in Russian; translate if the session is English).

## Goals

1. Discover the repo’s distributor-shaped surfaces (`RepoProfile`).
2. Detect doc↔code drift for stages marked present/partial.
3. Confirm high/medium findings.
4. Apply doc patches within the profile write policy (unless `apply=false`).
5. Create or refresh the flow SSOT.
6. Publish a primary markdown report + return `WorkflowReport`.

## Suggested args (saved workflow)

```yaml
scope: string    # "full" | "moderation" | "payments" | "identifiers" | "delivery" | "frontend" | "auth"
apply: boolean   # default true
flowSsotHint: string  # optional path hint; empty = discover
```

Inline runs may hardcode `scope="full"`, `apply=true`.

## Types (copy into script)

```ts
interface RepoProfile {
  /** Short product name from README or folder. */
  productName: string;
  /** Doc directories treated as SSOT roots. */
  docRoots: string[];
  /** Flow SSOT path existing or to create. */
  flowSsotPath: string;
  /** Agent-rule files. */
  agentRuleFiles: string[];
  /** Spec trees. */
  specRoots: string[];
  /** Exact status strings from code. */
  releaseStatuses: string[];
  /** stage id → present|partial|absent */
  stages: Record<string, string>;
  /** role → path */
  anchors: Record<string, string>;
  scopePackages: string[];
  notes: string[];
}

interface DocFinding {
  /** S1–S10 or index. */
  stage: string;
  /** Doc path or "unowned". */
  doc: string;
  /** path:line or empty. */
  code: string;
  /** One sentence. */
  problem: string;
  severity: "low" | "medium" | "high";
  action: "update-doc" | "flag-product" | "leave";
  evidence: string;
}

interface SurfaceAudit {
  surface: string;
  findings: DocFinding[];
  coveredDocs: string[];
  coveredCode: string[];
}

interface Confirmation {
  findingKey: string;
  status: "verified" | "unconfirmed";
  note: string;
}

interface PatchResult {
  edited: string[];
  notes: string[];
}

interface WorkflowReport {
  conclusion: string;
  findings: Array<{
    where: string;
    what: string;
    evidence: string;
    status: "verified" | "unconfirmed";
    severity: "low" | "medium" | "high";
  }>;
  verified: string[];
  notCovered: string[];
}
```

## Phases (Russian example — adapt language)

1. `Собрать профиль репозитория`
2. `Инвентаризировать документы`
3. `Проверить присутствующие стадии потока`
4. `Независимо подтвердить важные находки`
5. `Собрать план правок`
6. `Обновить документацию и карту потока` — only if `apply`; otherwise fold into plan phase (no empty phase)
7. `Выпустить отчёт и перечитать его`

Each phase needs ≥1 `ask` or `world.run`.

## Topology

### 1. Discoverer (required)

Name: `discoverer`  
Read-only. Follow `references/discovery.md`. Return `RepoProfile`.  
May use `files.glob` / `world.run` for `rg` from the script as well — prefer script globs for listing, agent for judgment.

### 2. Inventory

Name: `inventory`  
Input: profile. Build ownership table vs `references/doc-inventory.md` patterns. Note unowned claims.

### 3. Auditors — **dynamic from profile**

Do **not** always spawn six WE.DROP-named agents. Build the list from `stages` + `scope`:

| When present/partial | Suggested agent name | Focus |
|----------------------|----------------------|--------|
| S2,S4,S5,S6 | `audit-flow-core` | Status machine, submit/accept/reject |
| S3 | `audit-monetization` | Payments, fees, entitlements, commercial messaging |
| S5,S6 + identifier anchors | `audit-identifiers` | ISRC/UPC/EAN issuance rules |
| S8 | `audit-change-requests` | Lock + edit-request docs |
| S9 | `audit-delivery` | DDEX/delivery docs |
| S1 + user app | `audit-client-auth` | Session, 401, role gates, status labels in UI |
| agent/spec files exist | `audit-agent-rules` | AGENTS/spec drift |

Skip auditors for fully `absent` stages. Unique names; `Promise.all` fan-out.

Persona (adapt language): music-distributor docs auditor; compare docs↔code; no edits; return `SurfaceAudit`; escalate on contradictory instructions.

Pass **profile JSON + paths**, not file bodies. Tell them **not** to import facts from other products.

### 4. Confirmers

Fresh agent per high/medium non-`leave` finding: `confirm-${key}`. Reproduce; no edits; `Confirmation`. Keep unconfirmed labelled.

### 5. Planner

Name: `doc-planner`  
Ordered minimal doc patches. `flag-product` → report only.

### 6. Writer (`apply`)

Name: `doc-writer`  
Edit only profile-allowed paths. Must refresh `flowSsotPath` with:

- observed statuses  
- stage table with present/absent  
- links to feature SSOTs  
- `Last verified: YYYY-MM-DD`

Max 2 writer rounds if planner feedback loops.

### 7. Reader-proxy

Name: `report-reader`  
Draft report text only — clarity, unsupported claims, likely user questions.

## Deterministic helpers

Script-level globs/greps sized to the profile, e.g. search observed status tokens across docRoots + anchors. Example:

```ts
phase("Собрать профиль репозитория");
const discoverer = agent("discoverer", {
  system:
    "Ты профилируешь репозиторий музыкального дистрибьютора. Только чтение. Верни RepoProfile по skill references/discovery.md. Не переноси факты из чужих продуктов.",
});
const profile = await discoverer.ask<RepoProfile>(
  "Построй RepoProfile этого workspace. Выбери один flowSsotPath. Отметь стадии S1–S10.",
);
log(`Профиль: ${profile.productName}, SSOT: ${profile.flowSsotPath}`);
```

Survey real check scripts only if the user asked for behavioral proof beyond docs; docs sync is not a test-runner mandate.

## Reporting

- `report(finding)` as confirmations land.
- `artifact.markdown(..., { primary: true })` for the long-form report.
- `WorkflowReport` conclusion states what was in/out of profile scope.

## Anti-patterns

- Hardcoding `backend_api` / `frontend_lkpo` / YooKassa / RUAGY in the script.
- Spawning auditors for absent stages.
- Writer editing application source to match docs.
- Empty phases.
- Copying exemplar royalty/fee numbers into foreign docs.
- Agent-only fan-out for an explicit full sync / workflow request.

## Exemplar

If `productName` / paths clearly match the WE.DROP dealmedia layout, auditors **may** read `references/exemplar-wedrop.md` as a checklist of depth — still verify every claim against **this** tree.
