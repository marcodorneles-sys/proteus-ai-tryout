Status: ready-for-agent

# 07 — Long Lasting Downtime flag end-to-end

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Wire up the Long Lasting flag through the full stack. When an operator classifies
a stop and marks the `Downtime` as `Long Lasting`, the system keeps extending
that Downtime as MDC advances — automatically respecting shift boundaries and
never inventing state past the `MDC Sync Watermark`.

End-to-end scope:
- **API**: accept `long_lasting=true` on the classify endpoint; persist the flag
  on the `Downtime` record.
- **Long Lasting Extension Resolver** (issue 04): integrate into the
  Reclassification Service to compute per-shift materialized segments on each MDC
  advance cycle.
- **UI**: Long Lasting toggle in the Reason picker; render the active Long Lasting
  Downtime as an extended arrow; stop the arrow at the MDC Sync Watermark; close
  the arrow when the machine resumes or the stop ends.

Per ADR 0002, a Long Lasting Downtime is stored as one record per shift, not a
single open-ended record. The resolver closes the current segment at the shift
boundary and opens a new one for the next shift.

## Acceptance criteria

- [ ] Flagging a Downtime as Long Lasting persists the flag and triggers per-shift
      materialization via the Extension Resolver.
- [ ] The Long Lasting Downtime renders as an extended arrow on the timeline.
- [ ] The arrow ends at the MDC Sync Watermark, never past it.
- [ ] When MDC reports the machine resumed or the stop ended, the Long Lasting
      Downtime closes and stops extending.
- [ ] At each shift boundary, the current segment closes and a new one opens for
      the next shift.

## Blocked by

- [04 — Long Lasting Extension Resolver](04-long-lasting-extension-resolver.md)
- [05 — Reclassify a single stop end-to-end](05-reclassify-single-stop.md)
