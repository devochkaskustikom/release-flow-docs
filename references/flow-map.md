# Logical flow map (repo-agnostic)

Canonical human SSOT in a given repo is whatever `RepoProfile.flowSsotPath` points at. This file defines **what auditors should look for**, not fixed paths.

## Reference shape (exemplar)

A mature music distributor often looks like:

```text
draft ──submit──► in_review / moderation
                      │
                   accept ──► live / ok ──► delivery
                      │
                   reject ──► rejected ──► edit ──► draft/resubmit
```

Locked states may route edits through a **change-request** subsystem instead of direct PATCH.

WE.DROP’s concrete names and extras (priority queue, AI fee, RU-AGY ISRC, userscript ops) are documented in `exemplar-wedrop.md` — use them as a depth checklist, not as required features.

## Actors (generic)

| Actor | Responsibility |
|-------|----------------|
| Guest | Register / login |
| Artist / label user | Create releases, pay if required, submit, view status |
| Moderator / admin | Queue, accept/reject, identifiers, catalog ops |
| Payment provider | Webhooks / reconcile |
| Delivery / aggregator | DDEX or API delivery |
| Ops tooling | Optional sidecars (sheets, userscripts, RPA) |

## Stage audit prompts (agnostic)

### S1 Auth
- Where is the session stored? What happens on 401?
- Are admin and user gates distinct?

### S2 Draft
- What entities exist (release, track, artist, artwork)?
- Upload limits and format rules — documented where?
- Which fields are user-editable vs system-owned (UPC/ISRC)?

### S3 Monetization
- Is submission gated by subscription, per-release fee, or free?
- Webhook verification model — document without inventing providers.
- Entitlement flags that change moderation priority or fee waivers.

### S4 Submit
- Exact transition: from which statuses, to which statuses?
- Server-side guards (ownership, payment, validation) vs UI-only disables.

### S5 Admin queue
- List filters, priority lanes, assignment.
- Pre-accept tools (manual identifier allocation).

### S6 Decision
- Accept side effects (status, emails, code allocation, audit log).
- Reject requires reason? Are fields cleared on later accept?
- Transactionality for multi-track code assignment.

### S7 Live
- Public vs authenticated visibility.
- Upsell / lifecycle hooks on first approval.

### S8 Change requests
- Which statuses lock direct edit?
- Diff/approve/reject API and file cleanup on cancel.

### S9 Delivery
- Pipeline steps and status vocabulary (separate from release status).
- Retry / manual sync entry points.

### S10 Ops sidecars
- Hard boundaries: what the sidecar must never do (e.g. issue codes, auto-submit without click).

## Status documentation rules

1. Document **only** statuses evidenced in code or migrations.
2. UI labels may differ from enum values — map both in the flow SSOT.
3. Backlog wishes (future `rework` status, etc.) belong in backlog docs, not in the live state machine table.
4. Payment and delivery state machines get their own sections or files — do not overload the release enum table.

## Depth bar

Borrowed from the exemplar without copying facts:

- Money and identifier claims are high severity.
- Status transitions cite controller/route lines.
- Royalty / commercial messaging has a single SSOT and marketing surfaces must not contradict it.
- Auth failure modes are user-visible and documented.
- Ops tools have explicit hard boundaries in docs.
