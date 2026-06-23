---
status: proposed
---

# Downtime modelled as a time-interval overlay over Machine Stops

A **Downtime** is an operator-owned record carrying a Reason and flags over an interval `[start, end]` for one machine. It is *not* a 1:1 child of a **Machine Stop**. Stops remain raw, immutable MDC facts; Downtimes associate with any Stops whose timestamps fall inside their interval, and may also extend past or between Stops (Long Lasting). Coverage of a Stop by Downtimes is derived, not stored.

## Considered Options

- 1 Stop ↔ 0..1 Downtime — rejected: cannot split a long Stop into multiple Reasons; cannot extend past the last Stop for Long Lasting.
- 1..N Stops → 1 Downtime — rejected: still cannot split a Stop; Long Lasting must "claim" future Stops awkwardly.
- Time-interval overlay (chosen) — handles split, bulk, and Long Lasting uniformly; keeps MDC ingest free of operator-state coupling.

## Consequences

- Two operator-owned Downtimes for the same machine must not overlap; enforced at write time.
- "Uncovered" / "Partially Covered" / "Covered" are derived attributes computed by joining Stops to Downtimes on time range — every worklist query pays this cost.
- KPI reconciliation must materialise Downtime coverage rather than reading a foreign key.
