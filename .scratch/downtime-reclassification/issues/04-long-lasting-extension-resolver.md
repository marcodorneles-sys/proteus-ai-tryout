Status: ready-for-agent

# 04 — Long Lasting Extension Resolver — pure module + tests

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Build the Long Lasting Extension Resolver as a deep, pure, side-effect-free
module. It takes a `Long Lasting` `Downtime`, shift boundaries, the
`MDC Sync Watermark`, and the machine-state stream, and returns the set of
per-shift materialized `Downtime` segments and the point at which extension stops.

Rules (encoding ADR 0002):
- A Long Lasting Downtime is materialized as **one Downtime per shift**, not a
  single open-ended record.
- Each segment extends up to the lesser of: the shift end or the MDC Sync
  Watermark. It never enters the unknown region past the watermark.
- At each shift boundary, the current segment closes and a new one opens for the
  next shift.
- Extension stops when MDC reports the machine resumed or the Stop ended.
- No infrastructure dependencies.

Deliver the module with a test suite covering:
- Extension capped by the MDC Sync Watermark (watermark is before shift end)
- Extension capped by shift end (watermark is after shift end)
- Segment split across a shift boundary into exactly two segments
- Termination when the machine resumes (MDC reports running state)
- Termination when the Stop ends
- Never producing a segment that crosses into the unknown region past the watermark

## Acceptance criteria

- [ ] Pure function / class with no side effects and no I/O dependencies.
- [ ] Produces one Downtime record per shift for the duration of the Long Lasting state.
- [ ] Each segment ends at min(shift end, MDC Sync Watermark).
- [ ] Stops extending on machine resume or stop end.
- [ ] Never produces a segment past the MDC Sync Watermark.
- [ ] All test scenarios pass.
- [ ] No infrastructure required to run the tests.

## Blocked by

None — can start immediately.
