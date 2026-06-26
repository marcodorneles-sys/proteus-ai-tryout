# downtime_reclassification.md

## Module Reference
This feature belongs to the **Smart Downtime Entry Module**.
# path: docs\domain\modules\smart_downtime_entry.md

## ADR
Arquictecture Decision Record Reference
# path: docs\adr\0001-core-technology-runtime-stack.md

## Overview
**Downtime Reclassification** is the feature that allows operators to interact with the raw machine stops (red segments) captured on the timeline, actively assigning human context and root causes to automated machine events. 

## Persona
Shop-floor Operator (Persona P-01) — owner.
The same reclassification capability is reused by the Process Lead (Persona P-02)
through the office review surface, with a wider writable window (current and
previous shift, multiple lines). Permissions are defined in personas.md, not here.
# path: docs\domain\personas.md

## Key Capabilities
*   **Root Cause Allocation:** Operators select an **Uncovered** or **Partially Covered** **Machine Stop** from the timeline (the red base showing through) and choose a standardized business **Reason** (e.g., from the downtime catalog). This creates an operator-owned **Downtime** drawn over the red. The Machine Stop itself is never edited; reclassification covers it with a Downtime (see Coverage in [CONTEXT.md](../../../CONTEXT.md)).
*   **Minimum-Duration Filtering:** Each **Reason** carries a minimum duration (e.g., 10 minutes for a breakdown; 0 or 5 minutes for other stops). The minimum is measured against the **span of the Downtime being created** (the covered red for a normal Downtime). It is applied as a **filter on the downtime catalog at selection time** — Reasons whose minimum exceeds the selected span are hidden/disabled rather than raised as a post-hoc validation error (consistent with the SDE module's automatic option filtering). For a Long Lasting Downtime the minimum is satisfied by the extending duration.
*   **Whether a Downtime May Override Runtime:** A Downtime covers red **Machine Stops** by default. Covering green (runtime) base is permitted **only** when the chosen Reason explicitly allows overriding runtime; this is a per-Reason embedded rule, not the norm. Coverage (Uncovered / Partially Covered) is therefore tracked against red only.
*   **Long Lasting Downtimes:** Operators may flag a Downtime as **Long Lasting** (rendered as an extended arrow) when a machine is broken and the fix time is unknown. A Long Lasting Downtime extends only up to the **lesser of the current shift's end and the MDC Sync Watermark** — it never enters the unknown region. It is **materialized as one Downtime per shift**: at a shift boundary the current Long Lasting Downtime closes and a new one opens for the next shift (overlay always breaks at shift boundaries; ownership flips). "Continuously extends" means the extension is re-evaluated as MDC advances, not that it is a single open-ended segment. When MDC reports the machine resumed (green) or the Stop ended, extension stops there. See ADR 0002.
*   **Bulk Reclassification:** Operators may select multiple Uncovered or Partially Covered **Machine Stops** and apply **one Reason** to all of them in a single action, producing one **Downtime** per selected stop. Machine Stops have no type of their own; the eligible Reasons for a bulk action are the **intersection** of what each selected stop's span permits under Minimum-Duration Filtering.
*   **Correcting and Voiding:** An operator may re-reclassify a Downtime they own — changing its Reason or flags creates a new **Correction Record**, and the timeline renders only the latest (as-now). An operator may also **void** a Downtime, re-exposing the underlying red Machine Stop as Uncovered. Operators can **never delete the underlying Machine Stop**. All edits are bounded by the **Shift Handover Window**.
