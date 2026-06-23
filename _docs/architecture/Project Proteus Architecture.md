# Project Proteus Architecture — Caveman Markdown

_Source: Project Proteus Architecture Document.docx_  
_Mode: caveman skill — short, dense, no fluff. Technical meaning preserved.

---

## 1. Executive Summary

Project Proteus = new Azure-native operational platform replacing SAP MII-based GSCA Machinery landscape.

No code migration. No SAP MII port. Legacy system = business/process reference only: journeys, rules, integrations, reports, data flows, edge cases, validation.

Target = centrally deployed Azure app for all factories. Factory separation = logical, not separate deployments. Current rollout assumption = single go-live across factories, incremental delivery before cutover.

Main challenge ≠ rebuild screens. Main challenge = resilient, maintainable, observable, integration-ready platform handling:

- machine data
- Smart Downtime Entry (SDE)
- QSens / SmartSENS quality monitoring
- shift + production context
- KPI calculation
- operational reporting
- downstream data publication
- support + reconciliation

Proteus owns core operational + KPI/business calc logic. Downstream systems should consume curated outputs/read models, not reimplement logic.

External boundaries must be explicit: MDC/Edge, SAP, ServiceNow/SDM, Data Lake, BW/GSCA KPI, APIM, plant-facing APIs.

---

## 2. Architecture Goals

- Replace SAP MII GSCA Machinery before 2027 support deadline.
- Build Azure-native platform aligned with JTI MDC standards.
- Avoid reusing SAP MII code, SQL procedures, orchestration jobs, legacy DB structures.
- Preserve valid business behaviours, journeys, outputs, operational capabilities.
- Improve maintainability, scalability, resilience, observability, delivery speed.
- Support AI-first/spec-driven delivery using GitHub, GitHub Copilot Business, Spec Kit-like pattern.
- Define clear contracts with MDC/Edge, SAP, ServiceNow/SDM, Data Lake, BW/GSCA KPI, plant consumers.
- Support testing, shadow production, safe cutover.
- Avoid recreating legacy coupling.

---

## 3. Core Architecture Principles

### New Build, Not Migration
Proteus = new app. Legacy = reference, not implementation source.

### Azure-Native Default
Use Azure-managed services where fit. Follow JTI naming, tagging, security, networking, monitoring, cost, IaC standards.

### Modular Monolith First
Start modular monolith, not microservices. Clear module boundaries. Future extraction possible where justified.

### Integration-First
Contracts early. Treat integrations as architecture artifacts, not dev afterthought.

### Data Contracts Over Coupling
No dependency on external internals. Use versioned, owned, testable contracts.

### Observable By Design
Logs, metrics, traces, health checks, business signals, alerts, dashboards, runbooks = DoD.

### Secure By Design
Security embedded in identity, authz, secrets, infra, CI/CD, ops.

### Spec-Driven, Human-Governed
AI helps draft/build/review. Humans own intent, architecture, security, release quality.

---

## 4. Current-State Summary

Current GSCA Machinery:

- Hosted centrally in Geneva.
- SQL Server backend.
- SAP MII frontend + orchestration.
- Includes GSCA Machinery + QSens.
- SmartSENS term appears around quality sensor enhancement/integration context.
- Long-evolved DB/procedure/orchestration complexity -> direct reuse bad.

Current functional areas:

- machine performance monitoring
- Smart Downtime Entry
- QSens quality monitoring
- master data maintenance
- schedule setup
- workcenter/machine config
- downtime/reject handling
- operational reports/graphs
- downstream feeds

Current user groups:

- end users / operators
- power users / local config/support
- managers / office reviewers
- global/regional KPI consumers

Legacy roles should not be copied. Target RBAC should simplify around capabilities, permissions, factory/global scope.

Current integrations:

- SAP via RFC/staging
- ServiceNow / SDM schedule data
- MDC / machine data collection
- Data Lake pulls from SQL Server
- SAP BW / GSCA KPI consumes aggregated KPI source data
- Local Reject Application API / plant-facing reject/waste capability

Current machine data:

- stops
- states
- rejects
- volume
- defect volumes
- parameters

Target direction = standardized JSON machine events. OPC UA-compatible payloads desirable, not mandatory.

Current retention:

- >1 year moved to archive
- archive >3 years deleted
- frontend does not use archive
- historical analysis expected in Data Lake

---

## 5. Target Architecture Overview

Preferred target stack:

- React frontend
- .NET / ASP.NET Core backend
- API-first design
- Azure Container Apps runtime
- Azure SQL / managed relational store where fit
- event-driven ingestion where needed
- Azure Monitor + Application Insights + Log Analytics
- Microsoft Entra ID auth
- RBAC authorization
- Bicep IaC
- GitHub source
- Azure DevOps backlog/wiki/tracking
- GitHub Copilot Business + spec-driven delivery

Conceptual layers:

1. Presentation — web UX for operators, power users, managers, admins, reporting users.
2. API — app APIs for frontend/external consumers.
3. Application/Domain — SDE, QSens, master data, scheduling, KPI, reporting, notifications, admin.
4. Ingestion — receive standardized machine events.
5. Processing — validate, transform, enrich, aggregate, classify.
6. Data — raw events, processed data, KPIs, config, audit, read models.
7. Integration — SAP, ServiceNow, Data Lake, BW/GSCA KPI, plant APIs.
8. Observability/Ops — logs, metrics, traces, alerts, health, runbooks.

---

## 6. Logical Modules

### Smart MDC Mapping / Reason Library
Purpose: map raw machine stop/reject reasons to standardized business reasons.

Responsibilities:

- machine templates
- raw reason capture
- local + English descriptions
- mapping to global/local reject/breakdown libraries
- mapped/unmapped + validated/unvalidated tracking
- global governance + local factory workflows
- mapping ratio metrics
- semantic dimensions for reporting/KPI/Data Lake

### Ingestion
Purpose: receive, validate, correlate, filter, route machine events.

Responsibilities:

- payload receive
- schema validation
- duplicate/late/missing/out-of-order handling
- correlation IDs
- filtering of irrelevant/repeated telemetry
- raw storage where required
- valid event routing
- invalid event quarantine/dead-letter

### Master Data
Purpose: manage site/factory/workcenter/machine/material/order/reference/config context.

Responsibilities:

- SAP master data consumption
- hierarchy/relationship management
- global/local config
- effective dating/historical correctness
- reference data for SDE, QSens, KPI, reports, integrations

### Shift Schedule + Production Context
Purpose: foundational context for SDE, QSens, KPI, reports.

Responsibilities:

- shift schedules
- workcenter assignment over time
- gap/overlap prevention
- schedule validity periods
- shutdown/idle/non-running periods
- ServiceNow/SDM schedule outputs where required

### Smart Downtime Entry (SDE)
Purpose: downtime workflows, reclassification, operator + office review.

Responsibilities:

- downtime records in operational context
- shop-floor fast entry
- office review/correction/reconciliation
- reject entry / actual speed / shift reminders if confirmed
- downtime classification/reclassification
- constraint/non-constraint correlation
- rule validation
- audit history

### SmartSENS / QSens
Purpose: quality sensor monitoring, adherence, cockpit views.

Responsibilities:

- QSens machine parameters + quality sensor data
- cockpit + aggregated cockpit
- adherence KPI logic
- global/local sensor config
- sensor rules/inactive rules
- template sync if confirmed
- config history/audit
- outputs to KPI/reporting/notifications

### KPI + Aggregation
Purpose: source of truth for Proteus-owned KPI/business calc logic.

Responsibilities:

- aggregate machine/production/downtime/reject/QSens/schedule data
- calculate KPIs for SDE, QSens, reports, dashboards, BW/GSCA KPI
- recalculation when input changes
- hourly/scheduled calcs where needed
- KPI period + lock
- post-lock change control
- period close validation/reconciliation
- curated outputs/read models/contracts

### Reporting
Purpose: operational reports + read models + validation.

Responsibilities:

- native operational reports where workflow-adjacent
- reporting read models
- validation/reconciliation/shadow-production comparison
- near-real-time operational data
- Power BI integration only via ADR-approved fit
- retire obsolete reports/exports unless business need survives

### Notification
Purpose: email reports, alerts, operational messages.

Responsibilities:

- rationalized email reports
- configurable alerts
- recipient config
- QSens alerts where confirmed
- data-quality/integration notifications
- no full notification center unless confirmed

### Administration + Access Control
Purpose: admin, role/permission enforcement, support visibility.

Responsibilities:

- Entra ID auth integration
- consume identity/group/role/claim signals
- enforce RBAC + factory/global scope
- role catalogue + permission matrix
- audit admin/access actions
- no local self-service registration
- no independent access approval in Proteus

---

## 7. Data Architecture

Target data model = new design. Do not inherit legacy SQL schema/procedures/staging.

Separate:

- atomic operational transactions
- reporting/read models
- curated outputs/contracts

Proteus owns business logic:

- downtime allocation
- production-order context
- shift interpretation
- QSens adherence
- KPI calc
- KPI locking
- reporting-ready aggregation

Downstream consumers get calculated/curated outputs when possible.

### Mutation / Deletion

Avoid hard deletes for critical business data. Prefer:

- soft delete
- status exclusion
- correction records
- audit history

Define insert/update/correct/delete/restore/post-lock mutation rules.

### Data Categories

- raw machine events
- validated events
- processed operational records
- master data
- config
- calculated KPIs
- reporting datasets
- audit records
- error/dead-letter records
- observability data

### Key Data Concepts

Core concepts:

- Site
- Area
- Workcenter
- Production Line
- Machine
- Equipment
- Material
- Production Order
- Shift Schedule
- Schedule Assignment
- Shift
- Production Context
- Downtime Event
- Downtime Reason
- Production Event
- Production Quantity
- Reject Transaction
- Waste Transaction
- QSens Result
- KPI Definition
- KPI Result
- KPI Period
- KPI Lock
- Raw Event
- Integration Message
- Data Quality Rule
- Data Quality Exception
- Quarantine Record
- Audit Record
- User
- Role
- Notification
- Reporting Read Model
- Data Lake Export
- BW / GSCA KPI Output
- Operational Dashboard View
- Historical Snapshot

Conceptual flow:

Factories -> workcenters/lines -> machines/equipment -> machine events -> validation/transform -> operational records -> KPI/read models/exports/audit.

### Raw Event Storage

Decision needed: store raw events permanently, temporarily, or only on error.

Benefits:

- replay
- audit
- recalculation
- debugging
- dual-run validation
- data-quality analysis

Costs:

- storage
- governance
- retention
- operational complexity

### Data Quality

Proteus needs cloud-side validation/filter/dedup/quarantine/error visibility after ingestion boundary.

Candidate checks:

- schema
- required fields
- duplicates
- timestamp validity
- sequence/order
- machine/workcenter refs
- production order/schedule context
- plausibility
- business rules
- quarantine/flagging
- support visibility
- notifications for high-impact issues
- rejected/corrected/reprocessed traceability

### Event Handling

Define handling for:

- late events
- duplicates
- missing events
- out-of-order events
- corrected events
- replayed events
- events after downtime
- bursts after upstream recovery

### Retention

Current reference = ~1 year online, longer in archive/Data Lake. Target retention must be defined by category.

---

## 8. Integration Architecture

### Principles

Each integration needs owner, payload, auth, env strategy, testing, versioning, cutover.

Proteus owns its side only:

- inbound consumption
- outbound contracts
- validation
- persistence
- audit
- monitoring
- reconciliation support

External teams own their platforms.

APIM selective, not default for every API.

### MDC / Edge

Boundary starts at agreed cloud ingestion endpoint + standardized machine-event contract.

MDC/PCo/Edge remains outside Proteus scope:

- machine connectivity
- PCo config
- edge runtime
- drivers
- OT/IT network
- patching
- local monitoring

Target:

- standardized JSON machine events
- move away from file transfer
- Azure Event Hubs leading candidate for high-volume ingestion
- ADR needed after volume/order/replay/retention/retry/dead-letter validation

Open decisions:

- Event Hubs vs other service
- public protected vs private/hybrid endpoint
- schema versioning
- retry/dead-letter/replay/backpressure
- validation split between MDC/Edge and Proteus

### SAP S/4HANA / SAP PM

Target: SAP Integration Suite where applicable. Reuse/adapt existing SAP APIs where possible.

Expected domains:

- production orders
- confirmations
- material data
- functional location / PM hierarchy
- equipment
- SAP PM references
- product configuration

Proteus should expose controlled endpoints/ingestion points, not accept direct SQL dumps as target pattern.

Open decisions:

- API catalogue
- auth
- payloads
- test endpoints
- object ownership
- versioning
- rate limits
- near-real-time vs scheduled flows
- existing flows to reuse
- retry/logging/failure responsibility split

### ServiceNow / SDM

Current: schedule data from Machinery to ServiceNow/SDM. Future: planned activities back to Proteus possible.

Direction:

- dedicated APIs preferred
- Service Bus internal to Proteus possible for resilience
- ServiceNow should not be assumed Service Bus subscriber

Open decisions:

- current outbound structure compatibility
- inbound planned activities in first release or later
- payloads
- test env
- ServiceNow-side ownership/lead time

### Data Lake

Current: Data Lake pulls from SQL Server read-only, incremental ~15 min.

Possible initial option:

- Proteus internal new model
- expose curated SQL views/read models for Data Platform extraction
- minimal Data Lake disruption

Not confirmed. Needs Data Platform / Data & Analytics / Power BI / governance validation.

Private connectivity required. No public DB exposure.

Open decisions:

- curated SQL views vs Event Hubs vs files/drop zone vs APIs vs managed pipeline
- current consumed tables/views
- compatibility read models
- volumes/frequencies/incremental rules
- Power BI dependencies
- 15–30 min freshness acceptable?
- private endpoint/DNS/VNet/routing

### BW / GSCA KPI

Proteus must continue source data for BW / GSCA KPI.

Open:

- preserve current structure?
- hourly aggregation?
- view/extract/API/file?
- reconciliation
- cutover

### Integration Platform / APIM

Use JTI Integration Platform where value exists:

- API reuse
- governance
- security
- throttling
- caching
- auth
- multi-consumer exposure

Do not deploy own APIM by default. Publish through centrally managed APIM workspace if needed.

Messaging guidance:

- Event Hubs = high-volume streaming/telemetry
- Service Bus = reliable ordered business messages
- Event Grid = discrete events
- APIM = governed synchronous API exposure

### Local Reject / Waste API

Proteus should expose generic plant-facing reject/waste API. Romania Waste Management = known example, not special-case design.

Proteus owns:

- API contract
- validation
- authz
- processing
- audit
- persistence
- errors

Plant-local systems own:

- local UX
- orchestration
- implementation

Open:

- endpoints
- payloads
- compatibility
- consumers
- auth/RBAC/consumer identity
- APIM publication
- idempotency
- retry
- validation
- audit
- volume/perf/cutover

---

## 9. Azure + Deployment Architecture

### Runtime

Azure Container Apps preferred. AKS not preferred due operational/org overhead.

### Frontend / Backend

Baseline:

- React frontend container
- .NET backend container
- separate app registrations
- API scopes between FE/BE
- ACA for both

Azure Static Web Apps not baseline. Revisit only via explicit JTI approval/ADR.

### Region / Availability

Initial assumption: single Azure region with Availability Zones. No active-active multi-region unless BIA demands.

Need BIA-approved availability, RTO, RPO.

### Environments / Subscriptions

Expected:

- Development
- QA / Stage
- Production

Subscription model:

- non-prod subscription for Dev + QA/Stage resource groups
- prod subscription separate
- prod isolated
- shared ACR possible if approved

### IaC

Bicep preferred.

Rules:

- JTI naming guide
- Azure policies
- Key Vault secrets
- RBAC where possible
- some RBAC may need ServiceNow/IT portal requests

### Endpoint Exposure

Open decisions:

- public protected vs private endpoint
- APIM vs direct/internal APIs
- Front Door/WAF routing + enforcement
- hybrid connectivity for SAP/data center
- ingestion endpoint technology

### Web Access

Working assumption: central HTTPS web app, Entra ID auth, Azure Front Door/WAF.

Frontend may be public-protected. Backend/db/secrets/internal deps private/restricted.

Need confirm: public HTTPS with Entra/AFD/WAF vs private factory-to-Azure network.

### Developer Access

DBs/sensitive resources not public by default.

DEV Key Vault/DB/private resource access via developer Entra identity + Conditional Access. Extra patterns only if JTI requires.

---

## 10. Security Architecture

### Authentication

Microsoft Entra ID baseline. App processes JWT tokens.

### Authorization

RBAC confirmed direction. Need ADR for:

- role catalogue
- permission matrix
- factory/global scope
- Entra group vs app role vs hybrid
- admin ownership
- audit

Capability buckets:

- operators / shop-floor users
- support/review users
- power users / local config managers
- machine data experts
- local viewers
- global viewers
- global admins
- system/technical admins
- technical/integration actors

Do not recreate legacy roles.

### Access Lifecycle

No local registration. No local access approval.

Access flow:

1. User requests access via JTI process.
2. Governance approval.
3. Entra group/app role updated outside Proteus.
4. Proteus consumes identity/authz signals.
5. Proteus enforces RBAC + scope.
6. Audit access-relevant actions.
7. Removal via JTI IAM process.

### Secrets

Key Vault. Managed identities where possible.

### Security Scanning

Use DevSecOps controls:

- static code analysis
- dependency/vuln scanning
- secret scanning
- container image scanning
- IaC validation
- cloud posture scanning
- repo/pipeline security
- PR quality gates
- finding triage/remediation/risk acceptance

### Audit

Audit must support support/governance/troubleshooting.

Audit scope:

- manual entries
- downtime reclassification
- QSens config
- master data
- schedule/production context
- user access changes
- admin actions
- data corrections
- KPI recalculation
- integration actions/corrections
- rejected/quarantined/reprocessed data

MVP needs backend audit/change history. Full audit UI not assumed.

### Security Assessment

Required lifecycle:

- early architecture/security review
- ongoing DevSecOps scanning
- review major architecture/integration changes
- pre-go-live security / penetration-style testing in QA/pre-prod
- remediation or risk acceptance before prod

Security review covers:

- Azure architecture
- identity/authz
- API exposure/APIM
- network exposure
- secrets
- logging/audit
- data protection
- integrations

---

## 11. Observability + Monitoring

Baseline stack:

- Azure Monitor
- Application Insights
- Log Analytics

New Relic optional later if stronger APM need appears.

### Technical Telemetry

Capture:

- app logs
- API req/errors
- traces
- dependency calls
- DB perf
- background jobs
- Event Hub / queue processing
- report generation
- integration failures
- deployment events

### Business / Ops Telemetry

Monitor:

- ingestion delay
- missing/duplicate/out-of-order events
- SAP sync failures
- Data Lake availability failures
- ServiceNow failures
- BW/GSCA KPI extract failures
- delayed KPI calcs
- QSens/SDE issues
- queue backlog
- data quality exceptions
- Local Reject API failures
- invalid/duplicate waste/reject transactions
- unmapped machine reasons
- mapping ratio by plant/template
- new reason detection
- translation completeness

### Health Checks

Cover:

- frontend
- backend API
- DB connectivity
- ingestion endpoint
- queue/event processing
- background jobs
- external deps
- processing lag
- Local Reject API endpoint

### Dashboards

Types:

- technical ops
- integration health
- ingestion/processing
- business data freshness
- deployment/release
- support

### Alerts

Alert on:

- API failure/latency
- queue backlog
- ingestion delay
- processing failure
- external dependency failure
- DB degradation
- report generation failure
- Data Lake failures
- SAP / ServiceNow failures
- auth/authz failures
- Local Reject API failure rate
- validation failures over threshold
- unusual duplicate/retry patterns

---

## 12. Non-Functional Requirements — Key Points

### User / Usage

- Total users not quantified.
- Concurrency not final. Discovery raised ~1,500 concurrent reporting users; validate.
- Multi-factory/global access expected.
- Role counts unknown; confirm during RBAC design.

### Security

- Entra ID auth.
- RBAC authz.
- Access approval/attestation via JTI IAM / IT Service Portal.
- Low PII footprint.
- Public-protected web access expected via Azure Front Door/WAF.
- DBs not public.
- Key Vault for secrets.

### Scalability / Performance

- Machine event volumes unknown; must analyze MDC/edge payloads.
- Backlog/reconnect bursts must be handled gracefully.
- ACA dynamic scaling.
- Scale-to-zero only where no ingestion/user impact.
- API/UI/reporting targets must be defined.
- Operational screens must not be blocked by analytical/reporting load.

### Availability / Resilience

- Business-important, not factory-stopping.
- Formal availability/RTO/RPO via BIA.
- Single-region + AZ assumed for first release.
- ACA revision rollback possible, but DB migration compatibility needed.

### Data Freshness / Integration

- Data Lake freshness TBD. Legacy ~15–30 min reference only.
- Operational reports should reflect corrections/reclassification with minimal business-acceptable delay.
- DDS prior shift/day data must be ready for morning routines.
- Event-based ingestion baseline for machine events.
- Data Lake initial pattern may be curated SQL read models, pending ADR.

### Data Quality / Error Handling

Must support:

- validation before persistence/downstream use
- deduplication
- idempotency/replay safety
- quarantine/error handling
- retry/backoff/dead-letter
- reconciliation
- transformation traceability
- plausibility checks
- data-quality notifications for high-impact issues

### Delivery / Governance

- Spec-driven delivery.
- Human accountability.
- ADRs for major choices.
- PR/deploy quality gates.
- traceability from Work Item -> spec -> branch -> PR -> commit -> image -> deployment -> release.
- release readiness evidence required.

### UX

- Do not copy legacy UI 1:1.
- Redesign around validated journeys.
- Factory UX: clear, fast, readable; night/low-brightness themes if confirmed.
- Prototype validation via Figma/equivalent.
- Accessibility baseline.
- Localization required; exact languages TBD.

### Maintainability

- Modular monolith.
- Legacy as reference only.
- Business logic in app/domain services, not hidden stored procs.
- API versioning TBD.
- Data Lake compatibility views = explicit contracts if used.

---

## 13. Reporting Architecture

Current baseline:

- 70+ reports/graphs
- 10+ mail reports/notifications
- ~90 admin/master-data forms

This is discovery/estimation baseline, not rebuild commitment.

Rationalize each report/form:

- recreate as-is where justified
- redesign into modern workflow
- consolidate
- replace with read model / Power BI / Data Lake
- retire
- exclude local-only process

Legacy raw export not rebuilt by default.

### Reporting Baseline Decision

Native Proteus reporting for:

- SDE/downtime classification
- reject/waste review
- shift/production context
- QSens cockpit
- MDC validation/troubleshooting
- correction/reconciliation
- KPI validation/locking
- near-real-time factory visibility

Power BI = option for analytical/management reporting only after ADR.

### Logic Ownership

Proteus owns:

- downtime split by shift
- production order allocation
- KPI period allocation
- production context resolution
- QSens adherence
- operational KPI aggregation

Downstream tools consume curated/calculated outputs. No duplicate core KPI logic in DAX/Data Lake unless ADR-approved.

### Report Categories

| Category | Delivery direction |
|---|---|
| Operational app views | Native Proteus UI |
| Near-real-time dashboards | Native Proteus unless Power BI proven |
| Operational reports | Native UI / read models |
| Analytical reports | Power BI candidate |
| Management reports | Power BI / Data Lake candidate |
| Audit/sign-off reports | Case-by-case, likely Proteus read models |
| Email reports/notifications | Proteus notification capability |

### Power BI Assessment

Good fit:

- historical trends
- management dashboards
- cross-factory analytics
- extended non-operational reports

Weak fit:

- SDE screens
- QSens cockpit/operator monitoring
- review/correction workflows
- troubleshooting
- immediate reflection of changes
- write-back/acknowledgement flows

Planning impact if meaningful Power BI use:

- Reporting & KPI estimate: possible 10–25% reduction
- Overall estimate: possible 3–8% reduction
- BI/data engineering capacity increases
- QA/business validation mostly remains

Recommended ADR option: hybrid — operational native, analytical Power BI.

---

## 14. Testing, Shadow Production, Cutover

### Environment Reality

External systems have mixed non-prod maturity. MDC currently production-connected due real machine connectivity.

Testing must include mocks/sample payloads/shadow feeds/controlled prod-connected validation where needed.

### Testing Scope

Include:

- unit
- component/module
- integration
- contract
- API
- E2E
- performance/load
- security
- UAT
- reconciliation
- shadow production
- plant API contract/idempotency/retry
- KPI locking/post-lock corrections
- data-quality validation
- reporting read-model validation
- operational freshness validation

Most important risks:

- data correctness
- integration behaviour
- KPI consistency
- reporting freshness
- authorization correctness
- auditability
- cutover safety

### Early Technical Integration Validation

Validate early:

- MDC/Edge ingestion
- SAP payloads
- ServiceNow/SDM schedule integration
- Data Lake pattern
- BW/GSCA KPI outputs
- Local Reject API consumers
- non-SAP production-order/local integrations

Validate:

- sample payloads
- schema
- auth/authz
- connectivity
- retry/errors
- idempotency
- data quality
- correlation IDs
- logging/monitoring
- reconciliation

### Data Quality + Reconciliation Testing

Cover:

- invalid/incomplete events
- duplicates
- late/out-of-order events
- impossible values
- missing context
- unmapped reasons
- invalid reject/waste submissions
- failed/partial integrations
- corrections/reprocessing
- KPI/report impact

### Security Testing

Cover:

- role-based access
- API security
- authorization boundaries
- factory/global access
- operator vs power user
- admin permissions
- technical/integration actors
- plant-facing consumers
- privileged audit
- backend/db/secret protection

### Shadow Production

Need possible prod-connected validation mode where Proteus consumes real/duplicated feeds but does not produce official downstream outputs.

Validate:

- ingestion completeness/timing/ordering
- SDE outputs
- QSens outputs
- KPI consistency
- reports
- Data Lake/read models
- BW/GSCA KPI source data
- data quality
- legacy reconciliation

### Dual Run

Avoid both systems sending official outputs. During validation:

- Proteus can process data
- official downstream remains old system until cutover
- compare outputs
- classify/fix/accept differences

Plant APIs need explicit endpoint switch plan.

### Cutover

Current assumption = single go-live across factories.

Cutover defines:

- entry/exit criteria
- reconciliation evidence
- security assessment completion
- critical defect threshold
- rollback
- downstream switch sequence
- integration readiness
- API consumer readiness
- user/support readiness
- comms
- hypercare

At cutover:

- old system stops receiving new official data where applicable
- Proteus becomes source for new data/outputs
- downstream consumers switch
- plant APIs switch endpoints
- old system may remain read-only/historical for limited period
- support monitors ingestion/integrations/reports/KPIs/security/data quality

---

## 15. AI-First / Spec-Driven Delivery

Tooling:

- GitHub Copilot Business approved
- GitHub Spec Kit / GitHub-based spec pattern expected/familiar

Spec-driven delivery needs:

- spec artifacts
- review gates
- branch strategy
- vertical slices
- Copilot instructions
- skills/folder guidance
- PR review rules
- ADR alignment
- backlog alignment
- acceptance criteria linkage

AI-ready spec minimum:

- business intent / user outcome
- scope / non-goals
- domain terms/glossary
- inputs/outputs
- data contracts / JSON schemas
- API contracts
- acceptance criteria
- test scenarios / BDD examples
- affected modules/interfaces
- security/audit/observability/NFRs
- assumptions/decisions
- ADR links

AI-generated code must pass human review, tests, type checks, static analysis, security scans, architecture rules.

---

## 16. ADR Backlog — Priority Decisions

High-priority ADRs / decisions:

1. Modular monolith + internal boundaries.
2. ACA runtime + scaling model.
3. React frontend baseline.
4. .NET backend baseline.
5. FE/BE deployment separation + Entra app model.
6. Bicep IaC.
7. Env/subscription model.
8. Auth model.
9. RBAC model.
10. Standardized navigation/sitemap.
11. Shift Schedule domain model.
12. SDE entry vs review model.
13. QSens positioning.
14. Machine event ingestion pattern.
15. Event/message handling per use case.
16. Data quality/pre-processing MVP vs fast-follow.
17. Central KPI ownership.
18. Reporting delivery approach.
19. Reporting read-model strategy.
20. Data Lake integration pattern.
21. Data Lake private connectivity.
22. BW/GSCA KPI output pattern.
23. SAP integration pattern.
24. ServiceNow/SDM integration pattern.
25. APIM usage.
26. Azure Front Door/WAF pattern.
27. Public vs private endpoint exposure.
28. Raw event retention.
29. Data mutation/deletion model.
30. Audit scope.
31. Security assessment lifecycle.
32. Shadow production/reconciliation.
33. Cutover model.
34. Local Reject API.
35. AI-first/spec-driven delivery workflow.

Use ADR backlog during implementation planning. Prioritize decisions that affect estimate, design, external deps, security approval, roadmap.

---

## 17. Recommended Next Steps

- Confirm final architecture baseline with JTI MDC, BTS Manufacturing, Security, Integration Platform, SAP, Data Platform, delivery stakeholders.
- Prioritize ADR backlog.
- Decide reporting delivery approach.
- Confirm KPI calculation ownership.
- Rationalize reports/forms.
- Confirm Shift Schedule + Production Context design.
- Confirm SDE scope split: shop-floor vs office/review.
- Confirm QSens scope/positioning.
- Define standard sitemap/navigation.
- Confirm Data Quality initial vs fast-follow scope.
- Build integration dependency map.
- Collect integration contracts/payloads/auth/env/error/cutover/reconciliation.
- Engage JTI Integration Platform for APIM decisions.
- Engage SAP team early for SAP Integration Suite/APIs.
- Engage Data Platform/Data & Analytics/Power BI stakeholders for Data Lake pattern.
- Engage ServiceNow/SDM team.
- Start JTI security assessment early.
- Define canonical data model v1 + read-model strategy.
- Confirm Event Hubs ingestion ADR.
- Define RBAC implementation.
- Confirm environments/subscriptions/OCM/provisioning.
- Define observability dashboards/alerts/runbooks.
- Define BIA-driven NFRs.
- Define test/shadow/reconciliation/security/cutover strategy.
- Confirm Reject/Waste API contract and consumers.
- Confirm AI/spec-driven governance.

---

## 18. Implementation Readiness Checklist

Before full implementation, confirm or risk-accept:

- Reporting ADR done.
- Critical go-live reports identified.
- Report/form rationalization done or staged.
- KPI ownership confirmed.
- Shift Schedule + Production Context design confirmed.
- SDE shop-floor/review flows confirmed.
- QSens unified app positioning confirmed.
- Standard sitemap/navigation agreed.
- Data Quality scope agreed.
- Security assessment initiated.
- Pre-go-live security testing approach agreed.
- Audit scope agreed.
- Power BI feasibility assessed if used.
- Early technical integration validation plan agreed.
- Integration/reconciliation strategy agreed.
- JTI access provisioning flow confirmed.

---

## 19. Closing Position

Proteus = new business platform, not code migration.

Core risk ≠ ability to build modern web app. Core risk = unclear data model, integration contracts, NFRs, observability, test strategy, cutover controls.

Convert assumptions into explicit decisions, contracts, measurable readiness criteria -> implementation safer, faster, easier to govern.

Plant-facing APIs, including Reject/Waste Management API, should be reusable Proteus contracts, not one-off local integrations.
