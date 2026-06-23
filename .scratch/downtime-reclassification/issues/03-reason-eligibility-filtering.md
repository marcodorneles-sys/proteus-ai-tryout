Status: ready-for-agent

# 03 — Reason eligibility filtering

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Gate the downtime catalog so the operator can only pick `Reasons` that are valid
for what they selected. The Reason Eligibility Filter takes the selected span and
the catalog's per-`Reason` rules and returns the eligible `Reasons`:

- **Minimum duration** is measured against the `Downtime` span being created and
  applied as a filter at selection time — `Reasons` whose minimum exceeds the
  span are hidden/disabled, never raised as a post-hoc validation error. (e.g. a
  breakdown requires its 10-minute minimum; other `Reasons` allow 0 or 5.)
- **Can-override-runtime** gates whether a `Reason` may cover green runtime;
  `Reasons` that cannot override are unavailable for spans that include green.

## Acceptance criteria

- [ ] `Reasons` whose minimum duration exceeds the selected span are absent from the offered catalog.
- [ ] A breakdown `Reason` is unavailable for a span shorter than its minimum and available at/above it.
- [ ] `Reasons` with 0 or 5 minute minimums are offered for correspondingly short spans.
- [ ] A `Reason` that cannot override runtime is unavailable for a span covering green; one that can is available.
- [ ] Filtering happens at selection time; no post-hoc validation error path is used for these rules.
- [ ] Reason Eligibility Filter has table-driven tests (below / at / above minimum; override on vs off).

## Blocked by

- [02 — Reclassify a single stop, creating a Downtime](./02-reclassify-single-stop-create-downtime.md)
