Status: ready-for-agent

# 05 — Reclassify a single stop end-to-end

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Wire up a complete end-to-end path for the core reclassification action: an
operator selects an Uncovered `Machine Stop`, picks a `Reason` from the
downtime catalog, confirms, and a `Downtime` is drawn over the red. This is the
foundational vertical slice that all subsequent reclassification slices build on.

End-to-end scope:
- **Schema**: `human_input_records` table (append-only; fields: `id`,
  `stop_ref_id`, `human_reason_id`, `user_id`, `timestamp`, `action_type`,
  `downtime_id`); `downtimes` overlay table.
- **API**: `POST /reclassify` endpoint. Validates via Writable Window Policy
  (issue 03) and Reason Eligibility Filter (issue 02). Writes a new
  `Human-input Record` and `Downtime`. Never mutates the `Machine Stop`.
- **UI**: operator selects a red Machine Stop → Reason picker showing only
  eligible Reasons → confirm → stop renders as Covered; outstanding work list
  updates.
- **Reclassification Service**: thin orchestrator calling Coverage Engine (01),
  Reason Eligibility Filter (02), and Writable Window Policy (03).

Constraints:
- Cancelling the picker without choosing leaves the stop Uncovered; no record written.
- If the stop has become Covered between list load and confirm (race), surface
  the current as-now state rather than creating a duplicate overlapping Downtime.
- The underlying Machine Stop is never edited or deleted.

## Acceptance criteria

- [ ] Selecting an Uncovered Machine Stop and confirming a Reason creates a Downtime
      over it and a Human-input Record in the separate table.
- [ ] The Machine Stop record is unchanged after classification.
- [ ] Cancelling the picker creates no record and leaves the stop Uncovered.
- [ ] Confirming a Reason that is not in the eligible set is rejected at the API layer.
- [ ] The classified stop drops off the outstanding work list and renders as Covered.
- [ ] A single Downtime can span several adjacent Machine Stops.
- [ ] Attempting to classify a stop that became Covered since load surfaces the
      current as-now state without creating a duplicate.

## Blocked by

- [01 — Coverage Engine](01-coverage-engine.md)
- [02 — Reason Eligibility Filter](02-reason-eligibility-filter.md)
- [03 — Writable Window Policy](03-writable-window-policy.md)
