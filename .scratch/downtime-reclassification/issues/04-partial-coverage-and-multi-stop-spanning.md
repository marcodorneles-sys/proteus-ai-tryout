Status: ready-for-agent

# 04 — Partial coverage and multi-stop spanning

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Extend coverage handling beyond the single fully-Uncovered stop. The operator can
classify the remaining red of a Partially Covered `Machine Stop` so the whole stop
becomes Covered, and a single `Downtime` can span several adjacent `Machine Stops`
and the green between them (cardinality one `Downtime` to many `Machine Stops`).
The Coverage Engine reports Partially Covered correctly so the leftover red stays
on the work list until classified.

## Acceptance criteria

- [ ] A `Machine Stop` with a `Downtime` over part of it reports as Partially Covered, with the leftover red still on the work list.
- [ ] Classifying the leftover red yields a fully Covered stop.
- [ ] A single `Downtime` can span multiple adjacent `Machine Stops` and the green between them.
- [ ] The Coverage Engine reports such a span correctly (each underlying stop Covered).
- [ ] Coverage Engine tests cover partial coverage, multi-stop spanning, and overlapping `Downtimes`.

## Blocked by

- [02 — Reclassify a single stop, creating a Downtime](./02-reclassify-single-stop-create-downtime.md)
