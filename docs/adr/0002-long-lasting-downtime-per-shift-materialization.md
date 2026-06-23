---
status: accepted
---

# Long Lasting Downtimes Are Materialized Per Shift

A Long Lasting Downtime (machine broken, fix time unknown) conceptually continues
for as long as the machine stays down, but we materialize it as **one Downtime per
shift** rather than a single open-ended segment. At each shift boundary the current
Long Lasting Downtime closes and a new one opens for the next shift, and a Long
Lasting Downtime never extends past the lesser of the shift end and the MDC Sync
Watermark (it never enters the unknown region).

## Considered Options

* **Single continuous segment spanning shifts (rejected).** Simpler to reason about
  as "one ongoing event," but it breaks two established rules from the SDE Visual
  Timeline: the overlay layer always breaks at shift boundaries (Downtime ownership
  changes there), and the timeline never invents state past the MDC Sync Watermark.
  A single segment would also be hard to attribute correctly when each shift owns
  its own KPI/period.

## Consequences

* Downtime ownership and shift attribution stay clean — each shift owns its own
  Long Lasting Downtime, consistent with the per-shift overlay rendering.
* "Continuously extends" becomes a re-evaluation as MDC advances (extend up to the
  current watermark, stop when the machine resumes or the Stop ends), not a single
  mutable open-ended record.
* Reversing this later (collapsing to one segment) would change the Downtime data
  model and every consumer that reasons about per-shift ownership, which is why it
  is recorded here.
