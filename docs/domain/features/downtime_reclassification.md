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
Shop-floor Operator (Persona P-01)
# path: docs\domain\personas.md

## Key Capabilities
*   **Root Cause Allocation:** Operators select an unclassified downtime entry from the timeline and choose a standardized business reason (e.g., from a downtime catalog) to correctly allocate the time.
*   **Embedded Business Rules:** The system dynamically applies embedded handbook rules to the reclassification process. For instance, a breakdown must have a minimum duration of 10 minutes, while other stops might have a 0 or 5-minute minimum.
*   **Long-Lasting Overrides:** Operators can flag a downtime entry as "long-lasting" (visually indicated by an extended arrow) if a machine is broken and the fix time is unknown, which continuously extends the classification alongside the underlying machine state [19, 20].
*   **Bulk Reclassification:** Allows operators to select multiple stops of the same type and classify them simultaneously to reduce friction.
