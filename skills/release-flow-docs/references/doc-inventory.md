# Doc inventory — adaptive ownership

There is no fixed file list. After discovery, build an ownership table for **this** repo. The tiers below are patterns.

## Tier 0 — flow SSOT

| Pattern | Owns |
|---------|------|
| `RepoProfile.flowSsotPath` | Stage map, actors, live status table, links to feature SSOTs |

If missing on a sync that touches stages → create it (one path only).

## Tier 1 — feature SSOTs (ops / product)

Look under `docRoots` for topic files. Common names (examples, not requirements):

| Topic | Name heuristics |
|-------|-----------------|
| Payments / webhooks | `*PAYMENT*`, `*BILLING*`, `*YOOKASSA*`, `*STRIPE*` |
| Per-release fees | `*AI*RELEASE*`, `*FEE*`, `*CREDITS*` |
| Edit / change requests | `*EDIT*REQUEST*`, `*CHANGE*REQUEST*` |
| Upload limits | `*UPLOAD*`, `*LIMITS*` |
| Delivery / DDEX | `*DDEX*`, `*DELIVERY*`, `*ERN*` |
| Analytics | `*ANALYTIC*`, `аналитика*` |
| Backlog / audit | `TODO.md`, `*AUDIT*` |

Each claim in the flow SSOT should link to a Tier-1 owner or be fully stated inline (short facts only).

## Tier 2 — agent / contributor rules

| Pattern | Owns |
|---------|------|
| Root `AGENTS.md` / `CLAUDE.md` | Cross-cutting contracts summarized for agents |
| Package `**/AGENTS.md` | Stack-specific UI/API rules |

Update these when they state **false** behavioral contracts. Do not dump long ops runbooks here if Tier-1 exists.

## Tier 3 — coding guidelines / specs

| Pattern | Owns |
|---------|------|
| `.trellis/spec/**`, `specs/**`, ADRs | How to implement, not host-specific ops |

Patch when a guideline contradicts code; leave pure style guides alone unless the user asked.

## Unowned claims

If auditors find a behavioral fact with no doc home:

1. Prefer adding a section to the flow SSOT (if small).
2. Else create a focused Tier-1 file under the chosen doc root and link it from the flow SSOT + doc index/README.
3. Record `doc: "unowned"` in the finding until a home is chosen.

## High-severity claim classes (any repo)

Confirm independently when docs assert:

1. Money movement, fees, subscription entitlements, webhook verification.
2. Auth/session failure behavior (401, logout, role gates).
3. Release status transition table.
4. Identifier issuance (ISRC/UPC/EAN) — who generates, who may type what.
5. Delivery pipeline success/failure semantics.
6. Commercial messaging that regulators or artists rely on (royalty splits, reporting cadence).

Exact numbers and providers come from **this** repo’s code — never from the exemplar.
