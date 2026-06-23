---
status: proposed
---

# Long Lasting Downtimes auto-extend, freeze at shift boundary

While a Downtime carries the **Long Lasting** flag, its `end` auto-advances to track the underlying machine state — brief run blips shorter than the Reason's configured blip threshold are ignored. The `end` freezes when the machine returns to run for longer than the threshold, or when the current shift's handover window closes, whichever comes first. A machine still down across a shift boundary produces a new Downtime owned by the incoming shift.

## Considered Options

- Open-ended (`end = null` until cleared) — rejected: nullable end leaks into every duration query and KPI rollup; risk of forgotten flags running indefinitely.
- Auto-extend tracking `now()` — rejected: ignores the doc's stated behaviour ("extends alongside the underlying machine state"); cannot distinguish a machine genuinely back up from a polling delay.
- Auto-extend tracking machine state, freeze at shift boundary (chosen) — matches operator intent and bounds the open window.

## Consequences

- The blip threshold is a per-Reason attribute on **Downtime Rule** and must be configurable by the catalog admins.
- A Long Lasting Downtime that spans shift change is split: outgoing shift owns the first segment (frozen at handover-window close), incoming shift owns a freshly created Downtime if the machine is still down. Both are auditable to their respective owners.
- The `end` field is always non-null after the handover window closes, simplifying downstream queries.
