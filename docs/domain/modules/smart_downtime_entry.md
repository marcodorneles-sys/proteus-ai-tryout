# Smart Downtime Entry (SDE)
## Overview
The Smart Downtime Entry (SDE) module is the primary interface used by shop-floor operators (Persona P-01) and Process Leads (Persona P-02) to manage machine downtime, actual speeds, and rejects. Designed for noisy and demanding industrial environments via shared kiosk terminals, the UI prioritizes wizard-like, step-by-step patterns over complex web forms to minimize friction.

# Persona Reference
path: docs\domain\personas.md

## Key Features & Capabilities

*   **Visual Timeline:** Displays a timeline of machine states, using green segments for uptime (run hours) and red segments for machine stops/downtimes.
*   **Downtime Reclassification:** Operators select unclassified machine stops and reclassify them based on specific root causes [3, 8].
*   **Embedded Business Rules:** SDE automatically filters options based on embedded business rules, such as minimum duration thresholds (e.g., 10 minutes for breakdowns), "Long Lasting" flags, and rules dictating whether a stop can override runtime.
*   **Shift Handover Enforcement:** A strict locking mechanism ensures operators can only edit data for their current shift. After a shift ends, they have a brief, configurable window (defaulting to 30 minutes) to finalize entries before the system locks them out.
*   **Actual Speed Entry:** Allows operators to input machine speeds. The system mandates that a "speed loss reason" code is provided if the speed deviates from the nominal rate [2, 16]. 
*   **Constraint / Non-Constraint Correlation:** Correlates downtimes across the production l