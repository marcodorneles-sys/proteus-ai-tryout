---
status: proposed
---

# Downtime Rules: two-tier catalog, dual enforcement

**Downtime Reasons** and the **Downtime Rules** attached to them (minimum duration, Long Lasting allowed, may override runtime, blip threshold, …) live in a two-tier catalog: a global catalog owned by the GSCA Global Administrator (P-09), with per-plant overrides owned by the Local Configuration Administrator (P-04). Resolution rule: **plant override beats global default**, falling back to the global entry when no override exists. Rules are enforced on both the frontend (option filtering for UX) and the backend (request validation for integrity).

## Considered Options

- Hard-code rules in source — rejected as the legacy MII anti-pattern Proteus exists to escape.
- Plant-only catalog — rejected: every plant would re-enter the same handbook constants; no path for global policy.
- Global-only catalog — rejected: contradicts the P-04 persona, which explicitly owns local downtime catalogs.
- Frontend-only enforcement — rejected: violates the architecture's integrity / idempotency stance for transaction APIs.
- Backend-only enforcement — rejected: operators would see options that get rejected on submit; bad kiosk UX.

## Consequences

- Catalog API must expose the *resolved* rule set per plant; frontend never resolves precedence itself.
- Changes to a global Rule propagate to every plant lacking an override on next fetch; this is intentional and must be visible in the P-09 admin UI.
- Rule evaluation logic is shared shape, deployed twice (TS on the client, C# on the server); the rule schema must remain serialisable and side-effect-free.
