Status: ready-for-agent

# 05 — Correct and void a Downtime

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Let an operator fix their own entries. Re-reclassifying a `Downtime` (changing its
`Reason` or flags) appends a new `Correction Record`; the timeline renders only
the latest effective record (as-now), with superseded versions retained but not
drawn. Voiding a `Downtime` withdraws it and re-exposes the underlying red
`Machine Stop` as Uncovered. The `Machine Stop` itself can never be deleted.

This introduces append-only `Correction Records` and the as-now projection used
for rendering.

## Acceptance criteria

- [ ] Changing a `Downtime`'s `Reason` or flags appends a new `Correction Record` rather than overwriting.
- [ ] As-now rendering shows only the latest `Correction Record` per `Downtime`.
- [ ] Superseded records are retained but not drawn.
- [ ] Voiding a `Downtime` re-exposes the `Machine Stop` as Uncovered on the work list.
- [ ] No path deletes the underlying `Machine Stop`.
- [ ] Tests cover correction (new record, latest wins) and void (red re-exposed).

## Blocked by

- [02 — Reclassify a single stop, creating a Downtime](./02-reclassify-single-stop-create-downtime.md)
