# Example finding (generic)

```json
{
  "stage": "S4",
  "doc": "docs/RELEASE_FLOW.md",
  "code": "apps/api/src/releases/submit.ts:88",
  "problem": "Flow doc says submit always moves to pending_review; code also writes priority_review when entitlement flag is set.",
  "severity": "high",
  "action": "update-doc",
  "evidence": "submitRelease sets status from entitlements.priority ? 'priority_review' : 'pending_review'"
}
```

Paths and status names are illustrative — replace with `RepoProfile` values for the target repo.
