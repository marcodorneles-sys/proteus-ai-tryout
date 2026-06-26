Status: ready-for-agent

# 02 — Reclassify a single stop, creating a Downtime

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

The core reclassification path. An operator selects an Uncovered `Machine Stop`
from the red work list, picks a standardized `Reason` from a seeded downtime
catalog, and the system creates an operator-owned `Downtime` drawn over the red.
The Coverage Engine then reports that `Machine Stop` as Covered and it drops off
the work list. Rendering is as-now (the effective `Downtime`).

This introduces the thin Reclassification Service (create command) and `Downtime`
persistence. The underlying `Machine Stop` is never mutated — the `Downtime` is a
separate overlay record.

## Acceptance criteria

- [ ] Selecting an Uncovered `Machine Stop` and choosing a `Reason` creates a `Downtime` over it.
- [ ] The created `Downtime` carries the chosen `Reason`.
- [ ] The Coverage Engine reflects the stop as Covered and it leaves the work list.
- [ ] The `Machine Stop` record is unchanged after reclassification.
- [ ] The new `Downtime` is rendered as-now in the React surface.
- [ ] Reclassification Service create path has tests through its interface.

## Blocked by

- [01 — Walking skeleton: surface Uncovered Machine Stops](./01-walking-skeleton-uncovered-machine-stops.md)
