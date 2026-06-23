# Feature Specification: Reclassify a Single Stop, Creating a Downtime

**Feature Branch**: `001-reclassify-single-stop`

**Created**: 2026-06-23

**Status**: Draft

**Input**: Issue [02 — Reclassify a single stop, creating a Downtime](../../.scratch/downtime-reclassification/issues/02-reclassify-single-stop-create-downtime.md) (parent PRD: [Downtime Reclassification](../../.scratch/downtime-reclassification/PRD.md))

> Domain glossary: [CONTEXT.md](../../CONTEXT.md). Terms in `Title Case` (Machine Stop, Downtime, Reason, Coverage, Correction Record) are used as defined there.

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Assign a Reason to an unclassified Machine Stop (Priority: P1)

A Shop-floor Operator (P-01) at the kiosk sees a red **Machine Stop** that has no
human explanation. They select it, choose a standardized **Reason** from the
downtime catalog, and confirm. The system records an operator-owned **Downtime**
over that red, giving the machine's stop a business root cause.

**Why this priority**: This is the core value of the whole feature — turning a
raw, meaningless red stop into classified, reportable downtime. Without it
nothing else in the Downtime Reclassification feature has a foundation. On its
own it is a viable MVP: an operator can classify stops.

**Independent Test**: Seed one Uncovered Machine Stop and a catalog with one
valid Reason; select the stop, pick the Reason, confirm; verify a Downtime now
exists over the stop carrying that Reason.

**Acceptance Scenarios**:

1. **Given** an Uncovered Machine Stop and a catalog containing a valid Reason, **When** the operator selects the stop, chooses the Reason, and confirms, **Then** a Downtime carrying that Reason exists over the stop.
2. **Given** the operator has opened the Reason picker for a stop, **When** they cancel without choosing, **Then** no Downtime is created and the stop stays Uncovered.

---

### User Story 2 - Classified work drops off the outstanding list (Priority: P2)

Once the operator classifies a Machine Stop, the red they just acted on should no
longer demand attention, so they can move to the next unclassified stop without
re-reading what is already done.

**Why this priority**: Closes the loop on the core action and makes progress
visible, but depends on US1 having produced the Downtime first.

**Independent Test**: After classifying a previously Uncovered Machine Stop,
re-read the operator's outstanding work list and confirm that stop is absent and
now renders as Covered.

**Acceptance Scenarios**:

1. **Given** a Machine Stop that has just been fully covered by a new Downtime, **When** the operator's outstanding work list is recomputed, **Then** that Machine Stop no longer appears as outstanding.
2. **Given** a newly created Downtime, **When** the timeline is rendered, **Then** it shows the effective (as-now) Downtime over the region that was previously red.

---

### User Story 3 - The raw machine fact stays untouched (Priority: P3)

The operator adds a human layer on top of the machine's report; they must never
be able to alter or erase what the machine actually reported, so the raw fact
stays trustworthy for KPI and reconciliation.

**Why this priority**: A correctness/trust guarantee that underpins the data, but
it is a property of the action rather than a new operator-facing step.

**Independent Test**: Capture the Machine Stop record before classification,
classify it, and confirm the Machine Stop record is unchanged afterward and that
no operation is offered to edit or delete it.

**Acceptance Scenarios**:

1. **Given** an Uncovered Machine Stop, **When** the operator classifies it, **Then** the underlying Machine Stop is unchanged and only a separate Downtime overlay is added.
2. **Given** any classified or unclassified Machine Stop, **When** the operator looks for a way to edit or delete the raw stop, **Then** no such action is available.

---

### Edge Cases

- **Stop already classified by someone else**: If the selected Machine Stop has become Covered since the work list was loaded, confirming does not create a duplicate overlapping Downtime; the operator is shown the current (as-now) state instead.
- **Empty or invalid catalog selection**: Confirming without a chosen Reason creates nothing; a Downtime always carries a Reason.
- **No Uncovered stops**: When nothing is outstanding, the operator is presented with an empty work list rather than an error.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST present the operator with the Uncovered Machine Stops available to classify for the displayed machine and window.
- **FR-002**: System MUST allow the operator to select a single Uncovered Machine Stop and choose exactly one Reason from the downtime catalog for it.
- **FR-003**: On confirmation, System MUST create one operator-owned Downtime that covers the selected Machine Stop and carries the chosen Reason.
- **FR-004**: System MUST compute Coverage so that the newly classified Machine Stop reports as Covered and is removed from the operator's outstanding work list.
- **FR-005**: System MUST NOT modify or delete the underlying Machine Stop when a Downtime is created; the Downtime is a separate overlay record.
- **FR-006**: System MUST render the timeline as-now, drawing the effective Downtime over the region that was previously red.
- **FR-007**: System MUST guarantee that every Downtime carries a Reason; an attempt to confirm without a Reason MUST create nothing.
- **FR-008**: System MUST avoid creating a duplicate Downtime when the selected Machine Stop is already Covered at confirmation time.

### Key Entities *(include if feature involves data)*

- **Machine Stop**: The raw, read-only machine-state fact (a period the machine did not run). The base this feature classifies but never edits.
- **Downtime**: The operator-owned overlay created by this feature, covering a Machine Stop and carrying a Reason. The unit of classification.
- **Reason**: The standardized business root cause selected from the downtime catalog and attached to the Downtime.
- **Coverage**: The Uncovered / Partially Covered / Covered relationship that determines whether a Machine Stop still appears on the operator's outstanding work list.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: An operator can take an Uncovered Machine Stop to Covered in a single select-Reason-confirm interaction, with no manual timestamp entry.
- **SC-002**: 100% of created Downtimes carry exactly the Reason the operator selected.
- **SC-003**: 0 mutations occur to the underlying Machine Stop record across classification — it is identical before and after.
- **SC-004**: A classified Machine Stop disappears from the operator's outstanding work list immediately upon confirmation (next recompute of the list).
- **SC-005**: Confirming an already-Covered Machine Stop produces no duplicate Downtime in 100% of cases.

## Assumptions

- **Single whole-stop classification**: This feature classifies one whole Uncovered Machine Stop at a time. Partial coverage of a stop and a single Downtime spanning multiple stops are out of scope here (covered by issue 04).
- **Reason eligibility is not yet enforced**: The seeded catalog Reason is assumed valid for the selected span. Minimum-duration filtering and the can-override-runtime rule are out of scope here (covered by issue 03); the catalog is seeded.
- **Writable-window enforcement deferred**: Shift ownership and the Shift Handover Window are not enforced in this slice (covered by issue 06); the operator is assumed to be acting within their current shift.
- **Correction and voiding deferred**: Changing or withdrawing a Downtime is out of scope here (covered by issue 05); this slice only creates a Downtime. As-now rendering nonetheless shows the single effective Downtime.
- **Long Lasting deferred**: Open-ended Long Lasting Downtimes are out of scope here (covered by issue 07).
- **Single machine / single timeline**: One machine per timeline, consistent with the SDE Visual Timeline scope for P-01.
- **Builds on the walking skeleton**: The seeded Machine Stops, read-only API, Coverage Engine, and minimal work-list surface from issue 01 are in place.
