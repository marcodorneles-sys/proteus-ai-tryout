Status: ready-for-agent

# 07 — Long Lasting Downtimes

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Support the broken-machine, unknown-fix-time case. An operator flags a `Downtime`
as `Long Lasting` (rendered as an extended arrow). The Long Lasting Extension
Resolver materializes it **per shift** (ADR 0002): the extension reaches only up
to the lesser of the shift end and the `MDC Sync Watermark`, never into the
unknown region; at a shift boundary the current `Long Lasting` `Downtime` closes
and a new one opens for the next shift; extension stops when MDC reports the
machine resumed or the stop ended. "Continuously extends" means re-evaluated as
MDC advances, not a single open-ended record.

## Acceptance criteria

- [ ] An operator can flag a `Downtime` as `Long Lasting`; it renders as an extended arrow.
- [ ] Extension is capped at the `MDC Sync Watermark` and never enters the unknown region.
- [ ] Extension is capped at the shift end; a new `Long Lasting` `Downtime` opens for the next shift (one record per shift).
- [ ] Extension terminates when MDC reports the machine resumed or the stop ended.
- [ ] The resolver re-evaluates as the watermark advances rather than holding one open-ended record.
- [ ] Long Lasting Extension Resolver has table-driven tests (watermark cap, shift-end split, resume termination, stop-end termination, never crossing the unknown region).

## Blocked by

- [06 — Writable Window Policy](./06-writable-window-policy.md)
