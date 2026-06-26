# Smart Downtime Entry

The shared language operators and Process Leads use to talk about machine
downtime on the shop floor: what the machine reported, what the human says
about it, and how the two layers relate on the timeline.

## Machine State

**Machine Stop**:
A raw machine-state fact reported by MDC — a period where the machine was not
running. Lives on the read-only base layer of the timeline (drawn red) and can
never be edited by an operator.
_Avoid_: Stop, downtime (when you mean the raw fact), red segment, machine event

**MDC Sync Watermark**:
The latest point in time for which MDC has reported machine state for a given
machine. Anything after it is unknown, not runtime.
_Avoid_: Last sync, data cutoff

## Operator Classification

**Downtime**:
An operator-owned overlay segment carrying a Reason and optional flags (e.g.
Long Lasting). Drawn over the base layer. A Downtime always has a Reason — a
period with no Reason is a Machine Stop, not a Downtime.
_Avoid_: Classification, downtime entry, unclassified downtime

**Reason**:
The standardized business root cause assigned to a Downtime, chosen from the
downtime catalog (e.g. a breakdown or a specific stop category).
_Avoid_: Root cause, category, code (in isolation)

**Reclassification**:
The act of covering an Uncovered or Partially Covered Machine Stop with an
operator-owned Downtime that carries a Reason. The operator never edits the
Machine Stop itself; they draw a Downtime over the red showing through.
_Avoid_: Classification, allocation, tagging

**Void**:
To withdraw a Downtime so it is no longer effective, re-exposing the underlying
Machine Stop as Uncovered. Voiding never deletes the Machine Stop.
_Avoid_: Delete, remove, cancel

**Long Lasting**:
A flag on a Downtime used when a machine is broken and the fix time is unknown,
extending the Downtime alongside the underlying machine state. Rendered as an
extended arrow.
_Avoid_: Long-lasting, ongoing, open-ended

## Timeline Relationships

**Coverage**:
The relationship between Downtimes and the Machine Stops beneath them. A Machine
Stop is Uncovered (no Downtime over it), Partially Covered (Downtime over part
of it), or Covered (Downtime over all of it). Uncovered and Partially Covered
red is what the operator must still act on.
_Avoid_: Overlap, mapping

**Correction Record**:
The latest effective version of a Downtime. The timeline renders as-now — only
the most recent Correction Record per Downtime is drawn; voided and superseded
versions are not.
_Avoid_: Revision, edit, version

**Shift Handover Window**:
A brief, configurable window after a shift ends during which the previous
shift's data remains writable before it locks.
_Avoid_: Grace period, edit window
