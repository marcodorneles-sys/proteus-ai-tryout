# SDE Visual Timeline

## Module Reference
This feature belongs to the **Smart Downtime Entry Module**
# path: docs\domain\modules\smart_downtime_entry.md

## ADR
Arquictecture Decision Record Reference
# path: docs\adr\0001-core-technology-runtime-stack.md

## Overview
The **SDE Visual Timeline** is the primary operational interface used by the Shop-floor Operator (Persona P-01) to interact with machine data. It replaces complex, legacy drop-down forms with a visual, wizard-like approach designed for noisy industrial environments.

## Persona
Shop-floor Operator (Persona P-01)
# path: docs\domain\personas.md

## Scope
One machine per timeline. The persona driving this feature (P-01) works at a kiosk dedicated to a single line, so the timeline is single-machine end-to-end. Multi-machine / multi-line review surfaces are a separate concern owned by a Process Lead (P-02) feature and are deliberately out of scope here.

## Key Capabilities
*   **Two-Layer Visualisation:** The timeline is composed of two stacked layers:
    *   **Base layer (machine state, read-only):** Reflects raw MDC facts for the shift. Green where the machine ran, red where there is a **Machine Stop**. The operator can never edit this layer.
    *   **Overlay layer (operator-owned Downtimes):** Drawn *over* the base. Each overlay segment is a **Downtime** carrying a Reason and flags (e.g. **Long Lasting**), and may sit inside, span, or extend past the Machine Stops it relates to.
    The red regions an operator must act on for Reclassification are exactly the Machine Stops that are **Uncovered** or **Partially Covered** by Downtimes (per the Coverage definition in CONTEXT.md) — i.e. red base still showing through.
*   **Visible Window vs. Writable Window:** These are distinct concerns:
    *   **Visible window** — parameterised. Defaults to the operator's current shift, but the operator may scroll or select earlier windows for read-only context (e.g. to compare today's pattern against yesterday's). Visibility is not gated by shift ownership.
    *   **Writable window** — governed by the [Shift Handover Window](../../../CONTEXT.md) rules. The operator may write to their current shift always, and to the previous shift only while its handover window is still open. Bringing an earlier shift onto the screen does **not** make it editable.
*   **Shift Handover Window:** Once a shift ends, operators are granted a brief, configurable window (e.g., 15 to 30 minutes) to view and finalize the timeline data for that previous shift before it is locked.
*   **Shift Boundary Rendering:** Shifts split Downtimes, not Machine Stops:
    *   **Base layer is shift-agnostic.** A Machine Stop that crosses a shift boundary is drawn as one continuous red base segment, clipped only by the visible window.
    *   **Overlay layer always breaks at shift boundaries.** A Long Lasting Downtime that the rules split at the boundary is drawn as two separate overlay segments — one per shift — even when they abut visually, because Downtime ownership changes there.
    *   **The boundary itself is marked.** A subtle vertical marker on the visible window shows where each shift starts and ends, so the operator can tell where ownership flips.
*   **Data Freshness Indicator:** The timeline never invents runtime for periods MDC has not reported on:
    *   A **Last MDC Sync** marker (timestamp + visual cue) shows how recent the underlying machine-state data is, anchored to the [MDC Sync Watermark](../../../CONTEXT.md) for the displayed machine.
    *   Any portion of the visible window after that watermark is rendered as a distinct **unknown** state — visually different from both green (ran) and red (Stop). Silence is not treated as runtime.
*   **As-Now Rendering Only:** The timeline always shows the **latest effective state** of Downtimes for the visible window — i.e. the most recent [Human-input Record](../../../CONTEXT.md) per stop. Voided Downtimes are not drawn, and earlier versions of corrected Downtimes are not drawn. Audit / as-of-T reconstruction is a separate concern owned by a different feature, not by this timeline.

## Dependencies / Adjacent Features
The timeline is rendered inside an SDE shell that surfaces additional context (e.g. the current production order). That context is sourced from a different system and owned by a separate feature; this feature is responsible only for the timeline itself.