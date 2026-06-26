Status: ready-for-agent

# 09 — Ignore a Machine Stop

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Let an operator mark an Uncovered `Machine Stop` as a false trigger (Ignore),
excluding it from KPI output without creating a `Downtime` over it.

The Ignore action is distinct from Void: Void withdraws a `Downtime` the
operator already created. Ignore acts directly on an Uncovered `Machine Stop`
with no `Downtime` over it.

End-to-end scope:
- **API**: `POST /ignore` endpoint. Writes a `Human-input Record` with
  `action_type=ignore` directly against the `Machine Stop` (no `Downtime` is
  created). Validated by Writable Window Policy. The `Machine Stop` record is
  never deleted or modified.
- **Coverage Engine** (issue 01): the engine already treats Ignored stops as
  resolved. This slice wires the ignore flag through to the engine's input.
- **KPI Engine integration**: the KPI engine must exclude Ignored stops from
  recalculation. Ensure the `action_type=ignore` record is visible to the KPI
  consumer.
- **UI**: Ignore action available on Uncovered Machine Stops; after confirming,
  the stop disappears from the outstanding work list. No mandatory comment —
  single tap sufficient (action is audited by user_id + timestamp).

## Acceptance criteria

- [ ] Ignoring a Machine Stop writes a Human-input Record with action_type=ignore;
      no Downtime is created.
- [ ] The underlying Machine Stop record is unchanged.
- [ ] The Ignored stop disappears from the outstanding work list.
- [ ] The KPI engine excludes Ignored stops from recalculation.
- [ ] Ignore is rejected at the API if outside the Writable Window Policy.
- [ ] No mandatory comment is required to complete the action.

## Blocked by

- [05 — Reclassify a single stop end-to-end](05-reclassify-single-stop.md)
