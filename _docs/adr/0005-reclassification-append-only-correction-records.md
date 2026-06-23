---
status: proposed
---

# Reclassification is append-only via Correction Records

Every Reclassification or post-lock edit to a Downtime writes a new immutable **Correction Record**. Prior versions are retained; the latest record is the effective state. Downtimes are never hard-deleted — a "delete" is a Correction Record that marks the Downtime as void. Concurrent writers are arbitrated by optimistic concurrency on a `version` field; the second writer must refresh and retry. Reclassification API requests carry a client-supplied idempotency key; the server deduplicates within a 24-hour window.

## Considered Options

- In-place mutation with a separate audit log — rejected: two sources of truth diverge under load; audit log becomes a write afterthought.
- Hard delete with audit log — rejected by the architecture's "soft-delete / correction records only" hard constraint.
- Last-write-wins on the shared kiosk — rejected: the shared login means we cannot reconstruct who silently overwrote whom; optimistic concurrency forces the conflict to the surface.

## Consequences

- Reads must resolve the latest non-void Correction Record per Downtime; a materialised "current state" view is likely required for hot queries.
- The `version` field is bumped per Correction Record and returned to clients; clients echo it on the next write.
- Idempotency keys must be persisted long enough to cover the dedup window; recommend the same store as Correction Records, indexed by `(downtime_id, idempotency_key)`.
- Bulk Reclassification is N Correction Records in one transaction, not one combined record — each resulting Downtime retains its own audit line.
