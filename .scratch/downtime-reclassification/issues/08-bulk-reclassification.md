Status: ready-for-agent

# 08 — Bulk Reclassification

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Let an operator select multiple `Machine Stops` and apply a single `Reason` to
all of them in one action. Each selected stop gets its own `Downtime`; the
operator picks one `Reason` from the intersection of eligible Reasons across all
selected stops.

End-to-end scope:
- **API**: `POST /bulk-reclassify` endpoint. Accepts a list of stop IDs and a
  single `Reason`. Validates via Reason Eligibility Filter (intersection mode)
  and Writable Window Policy. Writes one `Downtime` and one `Human-input Record`
  per stop. Atomic — all succeed or none do.
- **UI**: multi-stop selection mode on the timeline; Reason picker shows only
  the intersection of eligible Reasons across all selected stops; if the
  intersection is empty, the picker communicates that no shared Reason is
  available and prevents confirmation.

Grouping is operator-driven — no system-enforced "same type" constraint.

## Acceptance criteria

- [ ] Selecting multiple stops and confirming a Reason creates one Downtime per
      stop and one Human-input Record per stop.
- [ ] The Reason picker shows only the intersection of eligible Reasons across all
      selected stops.
- [ ] If the intersection is empty, the picker communicates this and blocks confirmation.
- [ ] The bulk write is atomic: if any stop write fails, no records are committed.
- [ ] All selected stops drop off the outstanding work list and render as Covered.
- [ ] Bulk action is rejected at the API if any selected stop falls outside the
      Writable Window Policy.

## Blocked by

- [05 — Reclassify a single stop end-to-end](05-reclassify-single-stop.md)
