# Feature Specification: Reclassify a Single Stop End-to-End

**Feature Branch**: `002-reclassify-single-stop`

**Created**: 2026-06-26

**Status**: Draft

**Input**: User description: "turn this file into a specification" based on issue "05 - Reclassify a single stop end-to-end"

## User Scenarios & Testing *(mandatory)*

### User Story 1 - Reclassify an Uncovered Stop (Priority: P1)

An operator reviews uncovered machine stops, selects one uncovered stop, chooses a valid reason from the downtime catalog, confirms the action, and immediately sees the stop represented as covered.

**Why this priority**: This is the core business action that converts raw machine-stop signals into actionable, human-classified downtime data.

**Independent Test**: Can be fully tested by selecting one uncovered stop, choosing a valid reason, confirming, and verifying that the stop is shown as covered and no longer listed as uncovered work.

**Acceptance Scenarios**:

1. **Given** an uncovered machine stop and at least one eligible reason, **When** the operator confirms a selected reason, **Then** the system records the reclassification action and shows the stop as covered.
2. **Given** a newly covered stop, **When** the outstanding uncovered list refreshes, **Then** that stop is no longer shown in outstanding uncovered work.

---

### User Story 2 - Cancel Without Side Effects (Priority: P2)

An operator may open reason selection for an uncovered stop and then cancel before confirming.

**Why this priority**: Operators must be able to back out safely without accidental data creation.

**Independent Test**: Can be fully tested by opening reason selection and cancelling, then verifying no reclassification action is recorded and the stop remains uncovered.

**Acceptance Scenarios**:

1. **Given** an uncovered machine stop, **When** the operator opens reason selection and cancels, **Then** the stop remains uncovered and no new reclassification record is created.

---

### User Story 3 - Protect Data Integrity During Races (Priority: P3)

An operator attempts to classify a stop that has already been covered by another action since the operator loaded the list.

**Why this priority**: Prevents duplicate or conflicting downtime overlays and preserves trust in current system state.

**Independent Test**: Can be fully tested by making the stop covered in one session and then confirming reclassification from another stale session, verifying no duplicate coverage is created and the operator sees current state.

**Acceptance Scenarios**:

1. **Given** a stop appears uncovered in a stale view but is already covered now, **When** the operator confirms reclassification, **Then** the system rejects duplicate creation and returns the current as-now state.
2. **Given** a selected reason is not eligible for that stop at confirm time, **When** the operator submits, **Then** the system rejects the request and no coverage change is recorded.

---

### Edge Cases

- Operator cancels reason selection after choosing a reason but before confirm; no change is persisted.
- The eligible-reason set changes between initial load and confirm; submission with newly ineligible reason is rejected.
- Two operators attempt to classify the same uncovered stop at nearly the same time; only one coverage result is retained, and the other sees current state.
- A single downtime interval may validly cover multiple adjacent machine stops; coverage should be represented consistently across all affected stops.
- Reclassification request references a stop outside the writable window and is rejected without creating any new record.

## Requirements *(mandatory)*

### Functional Requirements

- **FR-001**: System MUST allow an operator to select an uncovered machine stop for reclassification.
- **FR-002**: System MUST present only currently eligible reasons for the selected stop.
- **FR-003**: System MUST require explicit operator confirmation before applying reclassification.
- **FR-004**: System MUST create a new human-input record for each confirmed reclassification action.
- **FR-005**: System MUST create a downtime coverage record for each successful confirmed reclassification.
- **FR-006**: System MUST keep machine stop source records immutable; reclassification MUST NOT edit or delete machine stop data.
- **FR-007**: System MUST create no records when an operator cancels before confirmation.
- **FR-008**: System MUST reject confirmation when the submitted reason is not in the currently eligible set for the stop.
- **FR-009**: System MUST enforce writable-window policy at confirmation time and reject out-of-window requests.
- **FR-010**: System MUST detect when a target stop is already covered at confirmation time and return current as-now state without creating duplicate overlapping downtime.
- **FR-011**: System MUST update outstanding uncovered work views so successfully covered stops are no longer shown as uncovered.
- **FR-012**: System MUST support one downtime coverage entry spanning multiple adjacent machine stops when business rules indicate contiguous coverage.
- **FR-013**: System MUST retain actor identity and action timestamp for each confirmed reclassification event.

### Key Entities *(include if feature involves data)*

- **Machine Stop**: A system-detected stop interval produced by machine signals; remains immutable and can be uncovered or covered by overlays.
- **Downtime Reason**: A catalog reason that may be eligible or ineligible for a specific stop under current rules.
- **Human-Input Record**: An append-only audit event that captures who performed the reclassification, on which stop, with which reason, when, and the associated downtime result.
- **Downtime Overlay**: A human-classified coverage interval drawn over one or more machine stops to represent resolved downtime classification.
- **Outstanding Work List**: The operator-facing list of currently uncovered machine stops requiring classification.

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: In UAT, 95% of successful single-stop reclassification actions are completed by operators in 30 seconds or less from stop selection to confirmation.
- **SC-002**: 100% of cancelled reclassification flows result in zero new human-input records and zero new downtime overlays.
- **SC-003**: 100% of attempts using ineligible reasons are rejected and leave uncovered/covered state unchanged.
- **SC-004**: 100% of race-condition attempts on already covered stops return current state without creating duplicate overlapping coverage.
- **SC-005**: After successful reclassification, the previously uncovered stop is removed from outstanding uncovered work view in 100% of observed test cases.

## Assumptions

- Operators performing this feature are authenticated and authorized through existing access controls.
- Coverage, reason-eligibility, and writable-window policies already exist and are available to this feature.
- Existing downtime catalog management is out of scope; this feature only consumes currently available reasons.
- This slice targets operator workflows for active reclassification and does not include historical bulk backfill.
- Adjacent-stop spanning behavior follows existing domain rules for contiguous coverage determination.