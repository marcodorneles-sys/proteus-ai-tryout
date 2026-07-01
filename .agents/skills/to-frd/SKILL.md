---
name: to-frd
description: Turn the current conversation context into an FRD and publish it to the project issue tracker when requested. Use when the user wants to create a Feature Requirements Document from the current context.
---

This skill takes the current conversation context and codebase understanding and produces a Feature Requirements Document. Do NOT interview the user — just synthesize what you already know.

The issue tracker and triage label vocabulary should have been provided if the user expects the FRD to be published. If publishing details are not available, return the FRD in Markdown instead.

## Process

1. Explore the repo to understand the current state of the codebase, if you haven't already. Use the project's domain glossary vocabulary throughout the FRD, and respect any ADRs in the area you're touching.

2. Sketch out the major feature areas that need to be described to complete the FRD. Focus on feature behavior, not implementation tasks.

This can include:

* The actors and roles involved
* The main user flows
* The functional requirements
* The business rules
* The data that must be captured, displayed, updated, or derived
* The permissions and access rules
* The validation and error-handling rules
* The external systems or dependencies involved
* The acceptance criteria needed to validate the feature

Actively look for unclear or missing requirements. Mark them as `[NEEDS CLARIFICATION]` instead of inventing business rules, permissions, validations, or integration behavior.

Check with the user that these feature areas match their expectations. Check with the user which behaviors need the strongest acceptance criteria or validation coverage.

3. Write the FRD using the template below.

If the user asked you to publish it to the project issue tracker, publish it and apply the agreed review label. If no label vocabulary is available, use `ready-for-review`.

If the user did not ask you to publish it, return the FRD in Markdown.

<frd-template>

## Feature Overview

A concise description of the feature from the user's or business stakeholder's perspective.

Explain:

* What the feature does
* Why it exists
* Who it supports
* What business or user outcome it enables

## Feature Scope

### In Scope

A list of what this feature includes.

### Out of Scope

A list of what this feature explicitly does not include.

## Actors and Roles

A list of actors, users, systems, or roles involved in the feature.

For each actor or role, describe:

* Who they are
* Their ID (if specified in the given context)
* What they need from the feature
* What actions they are expected to perform

## User Stories

A LONG, numbered list of user stories. Each user story should be in the format of:

1. As an <actor>, I want a <feature>, so that <benefit>

<user-story-example>
1. As a request approver, I want to see all requests waiting for my approval, so that I can review and respond to them efficiently
</user-story-example>

This list of user stories should cover all known aspects of the feature.

## Functional Requirements

A numbered list of functional requirements.

Each requirement should describe observable feature behavior and should be testable.

Use the format:

* FR-001: The system shall ...
* FR-002: The user shall be able to ...
* FR-003: The system shall display ...

Functional requirements can include:

* User actions
* System responses
* Status changes
* Data capture
* Data display
* Notifications
* Search, filtering, sorting, or export behavior
* Approval or rejection behavior
* Reporting behavior
* Integration-triggered behavior

Do NOT include implementation tasks or specific file paths.

## Business Rules

A numbered list of business rules that govern the feature.

Use the format:

* BR-001: ...
* BR-002: ...

Business rules can include:

* Eligibility rules
* Calculation rules
* Approval rules
* Status transition rules
* Locking rules
* Visibility rules
* Data-scope rules
* Timing rules
* Exception rules

Do NOT invent business rules. If a rule is implied but not confirmed, mark it as `[NEEDS CLARIFICATION]`.

## Data Requirements

A description of the data required by the feature from a business perspective.

This can include:

* Data entered by users
* Data displayed to users
* Data updated by the feature
* Data received from external systems
* Data sent to external systems
* Derived or calculated data
* Historical or audit data

Do NOT define a physical database schema unless it has already been agreed.

## Permissions and Access Rules

A list of permissions and access rules required by the feature.

This can include:

* Who can view
* Who can create
* Who can edit
* Who can approve
* Who can reject
* Who can cancel or delete
* Who can export
* Who can administer
* Which data scope applies to each role

If permissions are unknown, mark them as `[NEEDS CLARIFICATION]`.

## Validation and Error Handling

A list of validation rules and expected error behavior.

Include:

* Required fields
* Invalid values
* Duplicate submissions
* Invalid status transitions
* Unauthorized actions
* Missing external data
* External dependency unavailable
* Empty states
* Timeout or failure behavior
* User-facing error or confirmation messages where known

Do NOT invent error messages unless they have already been provided.

## Integration and External Dependencies

A list of external systems, services, files, APIs, events, reports, or dependencies involved in the feature.

For each dependency, describe:

* The external system or dependency
* Whether the dependency is inbound or outbound
* What data or action is exchanged
* Who owns the dependency, if known
* Any known assumptions or open questions

Do NOT assume ownership of external systems unless explicitly confirmed.

## Reporting, Audit, and Observability Requirements

A list of known reporting, audit, or operational visibility requirements.

This can include:

* Audit trail requirements
* Status history
* Actor and timestamp tracking
* Before/after values
* Reportable events
* Monitoring needs
* Failure visibility
* Support or troubleshooting needs

If these requirements are unknown, mark them as `[NEEDS CLARIFICATION]` only when they are relevant to the feature.

## UX and User Experience Requirements

A description of user-facing behavior and experience expectations.

This can include:

* Screens or entry points
* Navigation expectations
* Form behavior
* Confirmation messages
* Error messages
* Empty states
* Loading states
* Accessibility expectations
* Localization expectations
* Responsive behavior
* Help text or guidance

Do NOT invent visual design details unless they have already been provided.

## Acceptance Criteria

A numbered list of acceptance criteria.

Use Given / When / Then where helpful.

<acceptance-criteria-example>
AC-001: Submit a valid request

Given a requester has completed all mandatory fields
When the requester submits the request
Then the system records the request and displays the submitted status </acceptance-criteria-example>

Acceptance criteria should be clear, testable, and linked to the expected feature behavior.

## Assumptions

A list of assumptions used while creating the FRD.

Assumptions should be clearly separated from confirmed facts.

For each assumption, include:

* The assumption
* Why it matters
* What the impact would be if the assumption is wrong

## Open Questions

A list of unresolved questions.

Use this section for missing information that prevents the feature requirements from being considered complete.

For each question, include:

* The question
* Why it matters
* Who should answer it, if known
* Whether it blocks design, slicing, build, or validation

## Risks and Dependencies

A list of known risks and dependencies.

This can include:

* Business risks
* Requirement risks
* Data risks
* Integration risks
* Security or permission risks
* UX risks
* Delivery dependencies
* External ownership dependencies

## Further Notes

Any further notes about the feature.

</frd-template>
