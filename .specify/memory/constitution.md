# Project Proteus Spec Kit Constitution

**Version:** 0.1
**Status:** Draft for Engineering / Architecture Review
**Project:** Project Proteus
**Applies to:** Spec Kit artifacts, AI-assisted engineering workflows, generated specifications, technical plans, tasks, tests, and implementation outputs.

---

## 1. Purpose

This constitution defines the non-negotiable delivery, architecture, testing, security, and governance principles that must guide all Spec Kit and AI-assisted engineering work for Project Proteus.

It exists to ensure that generated artifacts remain aligned with approved business intent, architecture decisions, JTI delivery standards, security expectations, testing discipline, and human accountability.

The constitution is not a replacement for product requirements, architecture documentation, ADRs, ADO Work Items, test plans, or human review. It is the guardrail layer that Spec Kit and AI-assisted engineering must respect when producing `spec.md`, `plan.md`, `tasks.md`, `analysis.md`, code, tests, and implementation guidance.

---

## 2. Scope

This constitution applies to:

* `/speckit.specify`
* `/speckit.clarify`
* generated `spec.md`
* `/speckit.plan`
* generated `plan.md`
* `/speckit.tasks`
* generated `tasks.md`
* `/speckit.analyze`
* generated `analysis.md`
* `/speckit.implement`
* AI-generated code, test, documentation, and review suggestions
* GitHub-based Spec Kit artifacts
* feature branches and pull requests created from Spec Kit outputs

This constitution does not define detailed feature requirements, sprint scope, full architecture design, API schemas, or business process rules. Those must come from approved project artifacts such as ADO Work Items, PRDs, feature specifications, module specifications, ADRs, integration contracts, UX designs, and test documentation.

---

## 3. Precedence

When there is a conflict, the following order of precedence applies:

1. JTI enterprise policies, security standards, and platform governance.
2. Approved Project Proteus architecture decisions and ADRs.
3. Approved product, module, feature, UX, and integration documentation.
4. Azure DevOps Work Item content and acceptance criteria.
5. This constitution.
6. Generated Spec Kit artifacts.
7. AI-generated implementation suggestions.

If a generated artifact conflicts with a higher-precedence source, the artifact must be corrected or the conflict must be escalated before engineering planning or implementation continues.

---

## 4. Core Principles

### 4.1 Human Accountability

AI may draft, generate, compare, suggest, scaffold, and summarize.

Humans remain accountable for:

* business intent
* acceptance criteria
* architecture decisions
* integration decisions
* security and compliance
* testing strategy
* code quality
* release readiness
* production acceptance

Generated artifacts are not approved artifacts until reviewed by the appropriate human role.

---

### 4.2 Outcome over Tooling

The operating model is defined by delivery outcomes, not by tools.

BA / PO / SA / QA / Engineering artifacts must be valid whether they are produced manually or with AI assistance. The AI Framework and Spec Kit may accelerate the work, but they do not replace the required artifacts or the accountability of the roles that own them.

---

### 4.3 Specification Before Implementation

Implementation must not start from vague prompts or incomplete assumptions.

For non-trivial items, a reviewed `spec.md` must exist before `/speckit.plan`, `/speckit.tasks`, or implementation proceeds.

Small or low-risk items may use a lighter review path if the team explicitly accepts the risk.

---

### 4.4 Traceability by Default

Every generated Spec Kit artifact must remain traceable to its source delivery item.

At minimum, `spec.md` must reference:

* ADO Work Item ID
* ADO Work Item link
* source requirement context
* relevant module / feature / PRD links where applicable
* relevant ADRs or architecture constraints where applicable
* review status

GitHub Spec Kit artifacts must be linked from ADO. Full `spec.md` content should not be duplicated into ADO or Wiki by default.

---

### 4.5 Human Review of Generated Specifications

`spec.md` is a mandatory Spec Kit artifact.

The review of `spec.md` is not a second PRD process. It is a control point to confirm that the generated artifact correctly reflects the approved Work Item, acceptance criteria, business context, architecture constraints, and validation expectations.

For non-trivial items, `/speckit.plan` must not proceed until `spec.md` has been reviewed or the remaining risk has been explicitly accepted.

---

### 4.6 AI Must Surface Ambiguity

AI must not silently invent missing business rules, architecture decisions, integration behavior, test expectations, permissions, roles, or NFRs.

When source material is missing, inconsistent, or ambiguous, Spec Kit must surface a clarification question instead of proceeding as if the answer were known.

Clarification questions must be routed to the appropriate owner:

* BA / PO for business intent, scope, user behavior, acceptance criteria, and prioritization
* Solution Architect for architecture, integration, NFRs, security boundaries, and ADR alignment
* QA for testability, edge cases, validation evidence, and regression risk
* DevOps / Platform for environment, CI/CD, deployment, infrastructure, and operational constraints
* Tech Lead / Engineering for implementation feasibility, code structure, and technical consistency

---

### 4.7 Testability by Design

Acceptance criteria must be testable.

For non-trivial items, each meaningful acceptance criterion should have an expected validation method before sprint commitment or implementation planning.

Validation may be covered by:

* unit tests
* integration tests
* contract tests
* automated acceptance or UI tests
* manual QA test cases
* UAT or business validation
* explicitly accepted residual risk

Generated tests are not automatically valid. They must be reviewed and aligned with acceptance criteria and intended behavior.

---

### 4.8 Security and Compliance by Default

Security must be considered during specification, planning, implementation, testing, and release readiness.

Generated artifacts must not include:

* secrets
* passwords
* private keys
* tokens
* production credentials
* sensitive personal data
* unnecessary confidential content
* undocumented access assumptions

Secrets must use approved secret management, such as Azure Key Vault or another JTI-approved mechanism.

Authentication and authorization must follow approved JTI identity and access patterns.

---

### 4.9 Architecture Alignment

Generated plans and implementation must respect the approved Project Proteus architecture baseline and ADRs.

AI must not introduce architecture decisions that contradict approved direction.

Where architecture impact is detected, SA review must be requested before proceeding.

---

### 4.10 Risk-Based Governance

The level of review must match the risk of the item.

Small, isolated, low-risk changes may follow a lightweight asynchronous review model.

Non-trivial, integration-heavy, architecture-impacting, security-sensitive, business-critical, or validation-heavy items require stronger review before implementation planning.

---

## 5. Artifact Rules

### 5.1 Azure DevOps

Azure DevOps is the delivery workflow and tracking system.

ADO Work Items should contain or link to:

* business objective
* acceptance criteria
* dependencies
* risks
* relevant PRD / feature / module context
* relevant ADRs or architecture constraints
* generated Spec Kit artifact links
* review status
* acceptance status

ADO should link to generated GitHub artifacts rather than duplicating full Spec Kit artifact content.

---

### 5.2 SharePoint / Wiki

SharePoint / Wiki may hold approved human-facing business, governance, and stakeholder documentation.

Where content from SharePoint / Wiki is used as AI-readable context, the source must remain traceable.

Generated artifacts must not become uncontrolled parallel copies of approved documentation.

---

### 5.3 GitHub

GitHub is the engineering workspace for:

* Spec Kit artifacts
* source context files
* code
* tests
* pull requests
* engineering review
* CI/CD workflow files where applicable

Generated Spec Kit artifacts should be stored in a predictable folder structure.

Recommended structure:

```text
/specs/
  /ADO-<work-item-id>-<short-slug>/
    source-context.md
    spec.md
    clarification-questions.md
    run-log.md
    plan.md
    tasks.md
    analysis.md
```

Not all files are required at all times. The required files depend on the current stage of the Spec Kit workflow.

---

### 5.4 `spec.md`

`spec.md` is the generated requirements / behavior contract for a bounded delivery item.

It must preserve or reference:

* business objective
* user / actor
* intended behavior
* scope and non-goals
* acceptance criteria
* domain terms
* dependencies
* assumptions
* open questions
* relevant ADRs
* architecture constraints
* NFRs
* integration constraints
* testability expectations

It must not contain unreviewed implementation design that belongs in `/speckit.plan`.

---

### 5.5 `plan.md`

`plan.md` is the engineering planning artifact.

It should describe:

* technical approach
* implementation structure
* affected components
* contracts or APIs
* data model implications
* integration implications
* security considerations
* testing approach
* operational considerations
* architecture alignment

`plan.md` must respect approved ADRs and this constitution.

---

### 5.6 `tasks.md`

`tasks.md` is the engineering task breakdown.

Tasks should be small, actionable, and traceable to `spec.md` and `plan.md`.

Tasks should include test, review, documentation, and operational readiness work where relevant.

---

### 5.7 `analysis.md`

`analysis.md` is the consistency and risk analysis artifact.

HIGH findings must be triaged before implementation continues or before the item is considered complete.

Ownership depends on finding type:

* business ambiguity → BA / PO
* architecture or integration conflict → Solution Architect
* implementation inconsistency → Tech Lead / Engineering
* testability or validation gap → QA
* environment, pipeline, or deployment issue → DevOps / Platform

---

## 6. Architecture Guardrails

### 6.1 Azure-Native Direction

Project Proteus must align with the approved Azure-native direction.

Generated implementation guidance should prefer approved Azure services, managed platform capabilities, observable runtime patterns, and infrastructure-as-code practices.

---

### 6.2 Runtime and Application Structure

The default Proteus application direction is:

* React for frontend where applicable
* .NET / ASP.NET Core for backend/API services
* frontend and backend as separate deployable containers where applicable
* Azure Container Apps as preferred runtime for containerized workloads where aligned with platform guidance
* Bicep for infrastructure-as-code where infrastructure changes are in scope

Any deviation must be justified and captured through architecture review or ADR.

---

### 6.3 Modular Monolith First

Project Proteus should start from a modular monolith approach unless there is a clear architectural reason to extract an independent service.

AI must not propose microservices by default.

Service extraction may be considered only when justified by:

* independent scalability
* independent deployment cadence
* separate ownership
* resilience requirements
* integration isolation
* clear operational benefit

---

### 6.4 Protected Edge, Private Core

External exposure must follow approved secure entry patterns.

Backend services, databases, secrets, and internal dependencies must remain private or restricted unless explicitly approved.

Generated architecture must not expose internal services or databases publicly by default.

---

### 6.5 Identity and Authorization

Authentication must use approved enterprise identity patterns, including Microsoft Entra ID where applicable.

Authorization must follow approved RBAC / capability-based access principles.

Generated code must not hardcode authorization logic that bypasses approved identity, group, role, or capability models.

---

### 6.6 Secrets and Configuration

Secrets must not be stored in code, generated specs, logs, prompts, tests, or configuration files.

Use approved secret management, such as Azure Key Vault, managed identities, or equivalent JTI-approved mechanisms.

---

### 6.7 Observability

Generated plans and tasks must consider observability for meaningful business and technical flows.

Relevant implementation should include:

* structured logs
* metrics
* traces where appropriate
* health checks
* failure telemetry
* integration error visibility
* processing lag or freshness indicators where relevant
* operational dashboards or alerts where required

---

## 7. Integration Rules

### 7.1 Contract-First Integration

Integrations must be defined through explicit contracts.

Generated plans must not assume undocumented APIs, database access, event schemas, file formats, or internal implementation details of external systems.

Integration work should identify:

* system owner
* contract
* authentication
* authorization
* request/response or event schema
* retry behavior
* error handling
* reconciliation
* monitoring
* cutover considerations

---

### 7.2 External System Ownership

Proteus implementation must not assume ownership of external JTI systems.

External systems remain owned by their respective teams unless explicitly agreed.

This includes, where applicable:

* SAP
* ServiceNow
* Data Lake / Data Platform
* APIM / Integration Platform
* factory systems
* external APIs
* reporting platforms

Generated tasks must distinguish between:

* Intellias-owned application work
* consuming agreed contracts
* exposing agreed contracts
* coordinating with external platform teams
* implementation owned by another team

---

### 7.3 SAP, ServiceNow, Data Lake, and Integration Platform

AI must not invent direct integrations.

SAP integration assumptions must be confirmed with the SAP team and follow the approved SAP integration route where applicable.

ServiceNow integration assumptions must be confirmed with the relevant ServiceNow / SDM team.

Data Lake integration assumptions must be confirmed with Data & Analytics / Data Platform.

APIM / Integration Platform usage must be based on agreed governance and ownership.

---

## 8. Testing and Validation Rules

### 8.1 Acceptance Criteria Quality

Acceptance criteria must be clear, testable, and linked to intended behavior.

AI must flag acceptance criteria that are:

* ambiguous
* not testable
* missing edge cases
* dependent on unclear data
* dependent on external systems without defined behavior
* inconsistent with product or architecture context

---

### 8.2 AC-to-Validation Mapping

For non-trivial items, each meaningful acceptance criterion should map to an expected validation method.

The validation method may be:

* unit test
* integration test
* contract test
* API test
* automated UI test
* manual QA test
* UAT scenario
* explicit residual risk

---

### 8.3 Test Generation

AI may generate test scenarios, test cases, unit test scaffolding, integration test scaffolding, and automation suggestions.

Generated tests must be reviewed.

Tests must validate intended behavior, not simply confirm the generated implementation.

---

### 8.4 QA Review

QA review should be requested when an item is:

* business-critical
* validation-heavy
* integration-heavy
* regression-sensitive
* dependent on complex data
* dependent on external systems
* likely to require UAT evidence
* unclear from a testability perspective

---

### 8.5 Definition of Done Alignment

An item is not Done until validation evidence exists or the residual validation risk is explicitly accepted.

Validation evidence may include:

* passing automated tests
* manual QA evidence
* linked test execution results
* UAT confirmation
* documented exception or risk acceptance

---

## 9. Security and Data Handling Rules

### 9.1 Data Classification

AI-assisted workflows must respect JTI data classification rules.

The project must not send sensitive, confidential, personal, or production-sensitive content into AI workflows unless explicitly approved.

Where sample data is sufficient, use sample or anonymized data.

---

### 9.2 Prompt and Artifact Hygiene

Generated prompts, context files, logs, and artifacts must not include unnecessary sensitive data.

If sensitive data is required and approved, it must be minimized and handled according to JTI policy.

---

### 9.3 Security Findings

Security findings must be treated according to agreed severity rules.

Critical or High findings must not be ignored.

Exceptions must be explicitly approved, time-bound, and traceable.

---

### 9.4 Dependency and Supply Chain Safety

Generated implementation must not introduce arbitrary dependencies without justification.

New packages, libraries, containers, or tools must be compatible with approved security and licensing expectations.

---

## 10. Engineering Quality Rules

### 10.1 Maintainability

Generated code must be clear, maintainable, and consistent with project conventions.

Prefer simple implementation over unnecessary abstraction.

Avoid premature optimization and unnecessary framework complexity.

---

### 10.2 Domain Boundaries

Implementation must respect module boundaries and domain responsibilities.

Generated code must not create accidental coupling between unrelated modules.

Cross-module interactions must be explicit and justified.

---

### 10.3 Error Handling

Generated implementation must include appropriate error handling.

For integration or business-critical flows, error handling should consider:

* retries
* dead-letter or failure handling where applicable
* user-visible errors
* logging
* reconciliation
* operational support

---

### 10.4 Auditability

Business-critical mutations must be auditable.

Generated implementation should avoid hard deletes where audit, reporting, KPI, compliance, or reconciliation needs exist.

Use soft delete, correction records, status transitions, or before/after audit trails where appropriate.

---

### 10.5 Pull Request Quality

Pull Requests must be reviewable.

PRs should include:

* ADO Work Item link
* generated Spec Kit artifact links
* summary of implementation
* testing evidence
* known limitations
* architecture impact where relevant

AI-generated summaries may assist PR preparation, but PR accountability remains with the developer and reviewers.

---

## 11. Spec Kit Command Rules

### 11.1 `/speckit.specify`

`/speckit.specify` must generate `spec.md` from approved or referenced requirement context.

It must focus on what needs to be built and why.

It must not invent missing business rules.

If context is insufficient, it must produce clarification questions.

---

### 11.2 `/speckit.clarify`

`/speckit.clarify` must identify ambiguity, missing details, edge cases, and unclear assumptions.

Clarification questions must be captured in a reviewable artifact.

Questions should identify the likely owner role: BA / PO, SA, QA, DevOps, or Engineering.

---

### 11.3 `/speckit.plan`

`/speckit.plan` must not start for non-trivial items until `spec.md` has been reviewed or residual risk has been explicitly accepted.

The plan must respect architecture constraints, ADRs, integration ownership, testing expectations, and security rules.

---

### 11.4 `/speckit.tasks`

`/speckit.tasks` must create implementation tasks that include testing, review, and validation work where relevant.

Tasks must not only cover code writing.

---

### 11.5 `/speckit.analyze`

`/speckit.analyze` must be used to detect inconsistencies between constitution, specification, plan, tasks, and implementation expectations.

HIGH findings must be triaged by finding type and routed to the appropriate owner.

---

### 11.6 `/speckit.implement`

`/speckit.implement` may assist with implementation, but developers remain accountable for code quality, correctness, tests, security, and maintainability.

Generated code must be reviewed before merge.

---

## 12. Branch and Handoff Rules

Specification readiness work may be performed by a Spec Facilitator, Tech Lead, nominated engineer, or automation runner.

The person or automation that generates `spec.md` does not necessarily become the implementation owner.

Generated artifacts should be committed to an isolated branch or agreed Spec Kit folder.

Unfinished work must not be merged into the main branch.

Another engineer may continue from the generated branch or create a new branch from the latest committed point, according to the team’s branch hygiene rules.

---

## 13. Change and Drift Reconciliation

When a requirement, architecture decision, integration contract, glossary term, test expectation, or implementation reality changes, the relevant source artifact must be updated.

Affected downstream artifacts must be revalidated.

Change flow:

1. Identify the affected source of truth.
2. Update PRD, ADO Work Item, ADR, contract, glossary, or test expectation as needed.
3. Revalidate affected `spec.md`.
4. Update `plan.md`, `tasks.md`, tests, or implementation if required.
5. Make impact on scope, estimate, or sprint commitment visible.

A change becomes effective for AI-assisted generation only when the relevant source artifact has been updated or explicitly confirmed as still valid.

---

## 14. Governance

### 14.1 Ownership

This constitution is owned jointly by:

* Solution Architect
* Tech Lead / Engineering Lead

Contributors include:

* BA / PO
* QA
* DevOps / Platform
* Security, where required

---

### 14.2 Change Control

Changes to this constitution must be reviewed before being applied to active Spec Kit workflows.

Changes should consider impact on:

* existing `spec.md` files
* `plan.md`
* `tasks.md`
* templates
* automation workflows
* CI/CD checks
* review practices

---

### 14.3 Review Frequency

The constitution should be reviewed:

* at project setup
* after the first vertical-slice pilot
* when major architecture decisions change
* when Spec Kit workflow changes
* when repeated AI-generated issues reveal a missing rule
* before scaling the process broadly

---

## 15. Version History

| Version | Date | Author / Owner                 | Notes                                               |
| ------- | ---- | ------------------------------ | --------------------------------------------------- |
| 0.1     | TBD  | Solution Architect / Tech Lead | Initial Project Proteus Spec Kit Constitution draft |
