Status: ready-for-agent

# 08 — Bulk reclassification

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Reduce repetitive entry. The operator selects several Uncovered or Partially
Covered `Machine Stops` and applies one `Reason` to all of them in a single
action, producing one `Downtime` per selected stop. `Machine Stops` have no type
of their own; the `Reasons` offered for a bulk action are the **intersection** of
each selected stop's eligible set (from the Reason Eligibility Filter), so the
single chosen `Reason` is valid for every selected stop.

## Acceptance criteria

- [ ] An operator can select multiple Uncovered / Partially Covered `Machine Stops`.
- [ ] The offered `Reasons` are the intersection of each selected stop's eligible set.
- [ ] An empty intersection offers no `Reason` and the bulk action is not possible.
- [ ] Applying one `Reason` produces one `Downtime` per selected stop.
- [ ] Each resulting stop reflects its new coverage.
- [ ] Tests cover the intersection (including empty) and one-Downtime-per-stop output.

## Blocked by

- [03 — Reason eligibility filtering](./03-reason-eligibility-filtering.md)
