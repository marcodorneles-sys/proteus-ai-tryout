# PRODUCT REQUIREMENTS DOCUMENT (PRD)

---

## 1. DOCUMENT GOVERNANCE

### 1.1 Document Information

| Field | Value |
|---|---|
| Product / Initiative | Project Proteus — Cloud-Native Manufacturing Operations Platform |
| Document Name | Product Requirements Document (PRD) |
| Owner | Julia Shcherbyna (BA Lead) |
| Status | Draft |
| Repository Location | `specs/_context/PRD.md` |
| Related Artifacts | `specs/_context/PERSONAS.md`, `specs/_context/GLOSSARY.md`, `specs/_context/FEATURE_INVENTORY.md`, `specs/_domain/DOMAIN_MODEL.md`, `specs/_rules/BUSINESS_RULES.md`, `specs/_rules/PERMISSIONS_MATRIX.md`, `specs/_quality/NFR.md` |
| Created Date | 2026-06-03 |
| Last Updated | 2026-06-25 (v0.3) |

### 1.2 Governance & Revision History

| Version | Date | Author | Description of Change | Reviewers |
|---|---|---|---|---|
| 0.1 | 2026-06-03 | Julia Shcherbyna | Initial draft — generated from Discovery Phase elicitation | Florin, Andronic |
| 0.2 | 2026-06-25 | Julia Shcherbyna | Section 13 rewritten — Implementation Scope & Roadmap aligned to xlsx v0.2 (Jun 15); go-live targets updated (Aug/Oct 2027); Data Lake architecture updated to Event Hubs from Day 1 (A-13, Jun 2 decision); sensor utilization added to out-of-scope; new open questions flagged | |
| 0.3 | 2026-06-25 | Julia Shcherbyna | Added §1.3 source-of-truth precedence and §1.4 traceability/ID governance; trimmed Section 4 to a role roster pointing to PERSONAS.md; de-conflated Office Screen (F-01.6) from Aggregated Cockpit (F-02.2); corrected terminology to "Shift Remainder" and "GSCA Machinery" throughout; added data-residency constraint (Frankfurt-only); added audit-log scope note and tobacco track-and-trace exclusion; marked availability/RTO/RPO/concurrency NFRs provisional pending BIA | |

---

### 1.3 Source-of-Truth & Precedence (for AI agents and reviewers)

This PRD is the business-context anchor. When generating feature requirements, tickets, or
`specify.md`, load the canonical file for each concern:

| Concern | Canonical source |
|---|---|
| Business context, scope, goals | `PRD.md` (this file) |
| Terminology | `specs/_context/GLOSSARY.md` |
| Personas & goals | `specs/_context/PERSONAS.md` |
| Feature inventory & traceability | `specs/_context/FEATURE_INVENTORY.md` |
| Cross-cutting rules | `specs/_rules/BUSINESS_RULES.md` |
| Roles → permissions | `specs/_rules/PERMISSIONS_MATRIX.md` |
| NFRs / security | `specs/_quality/NFR.md`, `specs/_quality/SECURITY_REQUIREMENTS.md` |

**Precedence when documents conflict:** the concern-specific canonical file wins over the PRD
summary. If a conflict surfaces during gap analysis (`specify.md` vs. ticket vs. feature doc),
do **not** silently resolve it — raise it as an Open Question for BA/PO review.

### 1.4 Traceability & ID Governance

All artifacts are linked through a stable ID chain so completeness can be verified at every stage
(notably the gap check that compares `specify.md` back to its ticket and feature doc):

```
Module (M-xx)  →  Feature (F-xx.y)  →  Functional Requirement (FR-xxx)  →  Work Item / ticket (WI-xxx)  →  specify.md
                        ▲                         ▲
                  Persona (P-xx)           Business Rule (BR-xxx)
```

Rules:

- IDs are immutable once assigned; retire, never renumber.
- Every Feature (`F-xx.y`) traces to its Module (`M-xx`) and at least one Persona (`P-xx`); IDs are owned in `FEATURE_INVENTORY.md` and `PERSONAS.md`.
- Every Functional Requirement (`FR-xxx`) cites the Feature(s) it serves; every Work Item cites its parent `FR`/`F` IDs.
- **Gap-check rule:** every `F-xx.y` and `FR-xxx` in scope must map to at least one Work Item, and every Work Item must trace back to a Feature. Unmapped items on either side are flagged, not assumed.
- Glossary terms are referenced by their canonical name (not by ID) — terminology is not part of the traceability ID chain.

---

## 2. PRODUCT OVERVIEW

### 2.1 Business Context

JTI operates cigarette manufacturing factories across ~27 global sites. These factories run on a platform called GSCA Machinery, which was built on SAP MII (Manufacturing Integration and Intelligence). SAP MII reaches end of vendor support in 2027, and the existing system cannot be ported without a full redesign — its business logic is tightly embedded in SQL stored procedures, its frontend is static HTML/CSS/JS served through SAP MII, and the entire stack runs from a single server in a Geneva data center.

This context creates an unavoidable rebuild deadline and simultaneously an opportunity to eliminate known architectural bottlenecks: a SQL stored-procedure backend that slows feature delivery, an inability to support analytics and ML, and a single-point-of-failure infrastructure. The company is also moving toward IWS (Integrated Work Systems) maturity, requiring manufacturing data quality, completeness, and timeliness that the current system cannot reliably deliver.

The hybrid delivery team consists of BTS GSC (ownership), GDC (architecture), Siskon (edge/MDC infrastructure), and Intellias (platform development).

### 2.2 System Summary

Project Proteus is a cloud-native platform that replaces GSCA Machinery and QSens for JTI's global manufacturing operations. It ingests machine data from ~1,300 machines across 30 factory edge servers, provides shop-floor operators and factory management with real-time KPI dashboards and data entry tools, and integrates with SAP S/4HANA, ServiceNow SDM, and the enterprise Data Lake. The platform rebuilds approximately 90 admin forms, 70+ operational reports, and 5 operator screens, while also delivering a full redesign of the user experience with a modern cloud-native stack.

> **Note for AI coding agents:** This is a manufacturing operations data platform. Its core function is accurate, time-bounded KPI calculation from machine-event data. Business logic complexity is concentrated in KPI formulas, QSens adherence rules, and stop classification. When in doubt, correctness of calculation takes precedence over performance.

### 2.3 Strategic Goals

| Strategic Goal | Description | Priority |
|---|---|---|
| SG-1: Eliminate platform end-of-life risk | SAP MII reaches end of support by 2027; continued operation is a material business risk requiring a hard migration deadline | High |
| SG-2: Enable data-driven operational improvement | Cloud-native architecture positions the organization to leverage analytics, automation, and machine learning | High |
| SG-3: Improve time-to-market for process changes | Current architecture couples business logic to infrastructure via SQL stored procedures, slowing feature delivery | High |
| SG-4: Improve user experience | Retire legacy frontend; deliver modern, role-appropriate interface for shop floor and office users | Medium |
| SG-5: Decouple from single vendor lifecycle | No single product end-of-life should again dictate system viability | Medium |

### 2.4 Problem Statement

**Background**

GSCA Machinery is JTI's primary system for machine data collection (MDC), production KPI tracking, downtime classification, and quality sensor monitoring across 27 global factories. The platform is critically flawed in its current state:

- **Architectural**: Business logic embedded in SQL stored procedures; static HTML frontend served via SAP MII; single Geneva server.
- **Operational**: Data reaches the enterprise Data Lake with a ~4-hour lag (3h pipeline + 1h processing + 30min BI refresh), making operational reporting unusable for real-time management decisions.
- **End-of-life**: SAP MII end of support is 2027; no migration path exists; full rebuild is required.
- **User experience**: Dual-menu navigation, inconsistent forms, no unified menu hierarchy, shop-floor UI not optimized for industrial use.
- **Data quality**: Hard deletes exist (should be soft); timestamps stored in local time causing data lake reconciliation issues; no daily data verification flag; SDE page has had performance issues causing 30-minute stop display delays.

**Current Pain Points**

- Factory operators and process leads spend significant time each day on manual data verification across multiple disconnected forms (Import Issue Resolution, PO Confirmation Resolution, Shift Remainders, Reject records).
- KPI aggregation is computed at runtime in the Aggregated Cockpit, taking ~1 hour every 4 hours — a known performance bottleneck.
- Global Power BI reports have a 1–2 day delay; locally-built workarounds (factory-level Power BI using direct SQL queries) are the de-facto operational reporting tool in some factories.
- Users navigate via search or bookmarked URLs rather than a coherent menu hierarchy — the two legacy menus (old + new) coexist.
- Global template changes in QSens do not automatically cascade to local configurations; manual re-linking is required.
- Machine timestamps may drift from server time, causing cascading misclassification errors in bulk stop operations.

**Desired Outcome**

After implementation, factory operators interact with a single, unified platform where machine data appears in SDE within ~5 minutes of capture, KPIs are pre-computed and persisted (not calculated at runtime), all data entry is in one place (stop classification, speed entry, reject entry, shift remainders all accessible from SDE), and the Data Lake receives data with sub-5-minute latency via Event Hubs. Factory management can verify data quality at a glance from a single dashboard instead of navigating five separate forms. The platform is resilient, globally deployable to new factories without code changes, and not dependent on any single vendor's product lifecycle.

### 2.5 Supporting Evidence & Strategic Context

| Evidence Type | Summary | Why It Matters | Confidence Level | Source Date |
|---|---|---|---|---|
| Platform end-of-life | SAP MII end of vendor support confirmed for 2027 | Hard migration deadline; cannot defer | High | 2026 |
| Current volume metrics | ~1,300 machines, 30 edge servers, ~100,000 MDC rows/hour delta | Sizes ingestion and performance requirements | High | June 2026 (Katrina) |
| User interviews | 5-week Discovery Phase elicitation with Andronic (GSCA Machinery owner), Florin (IT Architect), Stanislav (factory process lead), Sergey (OPI global) | Confirmed scope, personas, business rules, UX requirements | High | May–June 2026 |
| Data Lake latency gap | Current: ~4 hours end-to-end. Target: 1–5 minutes via Event Hubs | Management needs same-day data for operational decisions; 4-hour delay makes reports "useless" for reactive control | High | June 2026 (Stanislav, Sergey C.) |
| OEE report usage | OEE report accounts for ~60% of all report runs; every factory uses it | Must be preserved and prioritized in rebuild | High | May 2026 (Florin) |

### 2.6 Product Vision

Project Proteus will be a single cloud-native platform serving as the operational backbone for JTI's global manufacturing sites. It will replace a fragmented, end-of-life, on-premise system with a modular, Azure-hosted application that accurately captures machine events, calculates production KPIs in near-real-time, supports operator data entry from the shop floor, and feeds trusted data to downstream reporting and analytics platforms.

The platform will scale to any factory, be deployable to new sites through configuration alone, and support the operational discipline required by IWS standards — giving operators, process leads, and factory management a single source of truth that they can trust within minutes of a shift event occurring, not hours later.

### 2.7 Goals & Success Metrics

| Goal | Metric | Current State | Target State |
|---|---|---|---|
| Eliminate data latency to Data Lake | End-to-end latency from machine event to Data Lake availability | ~4 hours | ≤ 5 minutes via Event Hubs (Day 1) |
| Real-time operational dashboards | SDE / Office Screen rendering time for current shift data | Not measured | ≤ 3 seconds under normal load |
| Platform availability | Uptime for core services | Single server, no SLA | 99.9% (≤ 43.8 min/month unplanned) |
| Data ingestion throughput | MDC events processed per minute without loss | Current capacity unknown | 30,000–50,000 events/minute |
| Machine state freshness | Edge-to-cloud latency for stop events appearing in SDE | ~3–5 min via PCO | ≤ 5 minutes (maintained or improved) |
| API response time | 95th-percentile response for standard API calls | Not measured | ≤ 500 ms |
| Eliminate platform end-of-life risk | SAP MII dependency removed | Full dependency | Zero dependency by 2027 |
| User experience | Single unified menu; no dual-menu navigation | Two parallel menus | One unified navigation hierarchy |

### 2.8 Non-Goals

| Excluded Item | Reason |
|---|---|
| DDS Board / IWS Power BI Board | Separate system with a different business owner; Proteus provides source data only |
| SAP BW / GC KPI reporting application | Out of scope; Proteus writes KPI data to BW; the reporting layer is a separate system |
| Historical data migration from GSCA Machinery SQL Server | Incompatible data models; clean cutover is faster and reduces risk |
| MDC / PCO edge server infrastructure | Owned by Cisco/Siskon; fixed integration boundary |
| Mobile-native application | Responsive CSS design only; no dedicated mobile UI |
| Push notifications / Microsoft Teams integration | Email notifications only in initial release |
| In-app feedback / ideas capture mechanism | Governance handled via Teams channels and OPI monthly meetings |
| Active Directory / IT identity management | Proteus reads from AD groups; does not manage them |
| Factory-floor hardware and industrial protocols | Physical equipment and on-site collection infrastructure is out of scope |
| Iran de-cloud deployment (CDLP) | Excluded per client decision |
| Automated DDS board sync (without human confirmation) | Human acknowledgment before DDS submission is a business requirement |
| Japan-specific features not applicable globally | Proteus only includes functionality applicable across all factories |

---

## 3. SCOPE DEFINITION

### 3.1 In Scope

| Area | Description |
|---|---|
| Smart Downtime Entry (SDE) | Shop-floor operator interface: multi-machine timeline, stop reclassification, bulk classify, apply-to-all-equipment cascade, actual speed entry, reject entry, shift remainder entry, statistics tab (expanded to line-level, all machines) |
| QSens Cockpit | Quality sensor monitoring cockpit: sensor cards per work center, parameter drill-down, binary green/yellow/red status, shift-level sensor check scheduling |
| QSens Configuration | Global sensor template, local factory configuration, sensor tag definition, business parameter definition, MDC Go-Live Date form |
| Office Screen | Near-real-time, shift-level **production** KPI overview across workcenters and machines for factory management; color-coded states; per-machine drill-down (F-01.6, M-01). Distinct from Aggregated Cockpit |
| Aggregated Cockpit (QSens) | Factory-level **quality/sensor** KPI aggregation by product group; per-workcenter sensor KPIs with drill-down; machine connection and clock-sync status (F-02.2, M-02). Distinct from Office Screen |
| Data Review Forms | Import Issue Resolution, PO Confirmation Resolution, Actual Speed, Shift Remainder review, Reject Entry (office user) |
| Downtime Issue List | Cross-line scan for data quality anomalies with direct links to SDE at the relevant shift |
| KPI & Operational Reports | ~70+ reports including OEE (highest priority), IWS Lost 3 report, Downtime Statistics, Reject reports, non-recorded stops metric at month level |
| Scheduled Email Notifications | ~10+ automated email reports on configurable schedules |
| Master Data Management | ~90 administrative forms: plant/workcenter/machine hierarchy, downtime reason catalogs, product mappings, sensor definitions, shift schedules, user management |
| SAP S/4HANA Integration | 18 entity types (production orders, PO confirmations, materials, work centers, plant hierarchy, etc.) via SAP Integration Suite |
| ServiceNow SDM Integration | Bidirectional: shift schedules (out), planned activities (in) |
| Data Lake Export | Azure Event Hubs push (near-real-time); SQL-views phased approach superseded 2026-06-02 (A-13) |
| SAP BW / GSCA KPI Export | Daily aggregated KPI data sent to SAP BW |
| MDC Event Ingestion | Azure Event Hubs-based ingestion of machine stop/state/volume/quality events from 30 edge servers |
| Authentication & Authorization | Azure Entra ID SSO, RBAC, plant-level data isolation, generic machine accounts for shop floor |
| Non-SAP factory support | Manual PO creation and PO confirmation entry for factories not integrated with SAP |
| Platform services | Shift schedule management, multi-timezone support, multi-language UI (12+ languages), soft delete everywhere, UTC timestamp storage |

### 3.2 Out of Scope

| Area | Description |
|---|---|
| DDS Board | Power App with Power BI backend; separate business owner; Proteus supplies source data only |
| Historical data migration | Clean cutover; no migration of GSCA Machinery SQL Server data |
| Edge server infrastructure | Physical MDC hardware and Siskon-owned collection layer |
| SAP MII platform | Retained as read-only fallback up to 6 months post-cutover; no development work |
| Data Lake internal processing | Proteus delivers data to the Data Lake boundary; internal Databricks/ADF pipelines are separate |
| Mobile native app | No dedicated mobile UX; responsive CSS only |
| Push notifications / Teams | Email only in initial release |
| QSens report (bottom of QSens module) | Deprioritized; analytics consumers use Power BI on Data Lake data instead |
| QSens Historical Data Refresh module | Obsolete; candidate for removal |
| Sensor Utilization (QSens) | Deprecated and excluded; QSens relies on adherence KPIs only; sensor utilization cards, KPIs, and automated email summaries are not in scope (A-15, confirmed with Florin) |
| Real-time predictive report | Unused in current system; deprioritized |
| In-app feedback capture | Not in scope; handled via Teams and OPI processes |
| Root cause → symptom mapping for DDS | Organization not ready; deferred |
| Japan-specific factory features | Only globally applicable features are in scope |
| Tobacco regulatory track-and-trace (e.g., EU TPD serialization) | Concrete external regulatory compliance is out of scope for now. **Change traceability (audit log) is in scope** — all data modifications are logged with user, timestamp, and before/after values (see FR set and §10 Security) — but compliance with specific tobacco regulations is not a Phase 1 deliverable |

### 3.3 Assumptions

| Assumption | Impact |
|---|---|
| New platform fully replaces SAP MII — no residual MII usage post-migration | Scope and architecture boundary |
| All 18 SAP entity types will have REST API equivalents via SAP Integration Suite, managed by JTI | Integration design; any gap could force scope change |
| Central cloud application with logical per-factory data isolation (RBAC), not separate instances per factory | Architecture; if changed to per-factory instances, cost and deployment model changes significantly |
| Backend: .NET (C#) on Azure PaaS services. Frontend: React. DB: Azure SQL Database | Stack is fixed; no POC/validation phase for stack choice |
| Azure Entra ID for authentication, SSO, and RBAC integrated with JTI's corporate identity provider | Auth design; if Entra ID is not available for all factories, provisioning model breaks |
| Peak MDC ingestion: 30,000–50,000 events/minute across ~1,300 machines | Sizing; if significantly exceeded, Event Hubs tier must be revisited |
| MDC systems deliver events as structured JSON payloads via push; OPC UA is aspirational | If push is not available at all sites, ingestion architecture may need fallback polling |
| Data Lake integration via Azure Event Hubs from Day 1; SQL-views phased approach was superseded 2026-06-02 (A-13) | Agreed approach eliminates the 4-hour data availability delay from the start; CDLP team must confirm their implementation timeline; if infeasible, phased approach (A-12) may be reinstated |
| Big-bang go-live: all factories cut over simultaneously | Rollout strategy; if phased rollout is required, deployment architecture changes |
| No separate POC phase; architectural validation happens within early implementation sprints | Timeline assumption |
| Operational database sized at tens of GBs; data retained 12 months in primary database, older archived to Data Lake | Storage and archival design |
| SAP Integration Suite was implemented April 2026 and is live | Dependency met |
| Notifications via email; push/Teams out of scope for initial release | NFR scoping |
| Three environments: Development, Staging/QA, Production | Infrastructure planning |
| All factory rollout: no code changes required to onboard a new factory | Configuration-driven multi-tenancy is a design requirement |

### 3.4 Constraints

| Constraint Type | Description |
|---|---|
| Technical — SAP MII deadline | SAP MII reaches end of support in 2027 — hard migration deadline that determines the outer bound of the go-live window |
| Technical — Azure-only | All cloud workloads must be hosted on Microsoft Azure; no multi-cloud or alternative providers |
| Technical — Data residency (Frankfurt) | All data resides in the Azure Frankfurt (Germany) region; no local / in-country storage. Single-region deployment (latency for non-European factories tracked in OQ-21) |
| Technical — Cloud-native | Must follow cloud-native principles; lift-and-shift is not acceptable |
| Technical — APIM gateway | Azure API Management (APIM) is mandated as the API gateway for ServiceNow and SAP integrations |
| Technical — Edge infrastructure fixed | 30 on-premise edge servers (Siskon-owned) are a fixed integration boundary; Proteus cannot modify them |
| Organizational — Multi-party team | Four parties (BTS GSC, GDC, Siskon, Intellias) must coordinate; decisions require cross-team alignment |
| Timeline — ~2 years | Initial production release targeted approximately June of Year 2 of the programme |
| Regulatory — SAP integration cadence | SAP-side changes take 6+ months; existing SAP integration patterns must be preserved to meet the 2027 deadline |
| Business — Global applicability | Only features applicable to all factories are in scope; no factory-specific features |

---

## 4. USERS & PERSONAS

> **Canonical source:** [`personas.md`](personas.md) (PERSONAS.md). This section is a summary roster only.
> Access levels and the AD-group → role → permission mapping are maintained in
> `specs/_rules/PERMISSIONS_MATRIX.md` — not duplicated here, to prevent drift.

### 4.1 Role Roster (summary)

| ID | Persona | One-line purpose |
|---|---|---|
| P-01 | Shop-floor Operator (Equipment Owner) | Per-shift data entry (downtime, speed, rejects, remainders, sensor checks) via shared line/kiosk account |
| P-02a | Process Lead / Shift Supervisor | Data verification and correction across lines in their area; daily review before DDS |
| P-02b | Production Analyst | Month-end KPI locking, SAP reconciliation, complex PO confirmation resolution |
| P-02c | Production Machine Expert | MDC breakdown/reject mapping and QSens sensor tag-code identification |
| P-03a | Local Configuration Administrator (Performance) | Local performance-side master data and configuration |
| P-03b | QSens Local Admin (Quality) | Local QSens sensor configuration and business parameters |
| P-04 | Data Consumer | Read-only report/dashboard consumption — local (one factory) or global (all factories, single AD group) |
| P-05a | GSCA Global Administrator (OPI-DPA) | Global performance/KPI governance, global master data, MDC mapping standards |
| P-05b | Global QSens Administrator (OPI-GC) | Global QSens quality standards, thresholds, formulas, benchmarking |
| P-06 | System Administrator (IT) | Technical/platform configuration; functional-element → role mapping (release-gated) |

> Detailed goals, write-access scope, factory-role patterns, role codes, and cross-cutting access rules
> (release-gated reason/permission creation, two-layer provisioning, no report-level security) live in
> `personas.md`. The role × feature access matrix lives in `PERMISSIONS_MATRIX.md`.

---

## 5. HIGH-LEVEL USER JOURNEYS

### 5.1 User Journey: Daily KPI Data Verification (Production Analyst)

**Objective:** Ensure all machine data for the day is complete, correctly classified, and confirmed before the DDS meeting (~8:45 AM each morning).

**Preconditions:**
- The previous day's shifts have ended and the 30-minute edit window has closed
- MDC reprocessing cycle (every 90 minutes) has completed — ensures all data gaps have been backfilled
- SAP PO confirmations have been synced (near-real-time)

**Main Flow:**

1. Open the Downtime Issue List — scan for flagged anomalies across all lines (idle with production, missing speed entry, unexpected PO confirmations). Click any flag to open SDE at that exact shift.
2. In SDE, review downtime timeline per line. Reclassify any unclassified stops (red segments) with the correct downtime reason and location.
3. Verify all machines have actual speed entered for the shift. Enter or correct where missing.
4. Open Import Issue Resolution — identify SAP confirmations that failed to attach to a shift; resolve or escalate.
5. Open PO Confirmation Resolution — reassign delayed/misallocated SAP confirmations to correct shifts; this auto-clears matching shift remainders. Use Bulk Resolution for small quantities.
6. If the factory uses shift remainders: open Shift Remainder review — flag any shift remainder > 1 pallet as suspicious.
7. Open Reject Entry review — check for "production without reject" and "rejects without production" anomalies; verify reject quantities.
8. Open OEE report — confirm working hours, downtime reasons, actual speed, and non-recorded stops are reasonable.
9. Data is ready for DDS meeting.

**Expected Outcome:** All shift data for the previous day is verified, classified, and accurately reflected in KPI calculations before the daily management meeting.

**Alternative Flows:**
- If Process Lead has already completed the data review (step 1b), the Production Analyst may skip directly to the Downtime Issue List.
- In small factories where one person plays multiple roles, all steps are performed by that person sequentially.

**Failure Scenarios:**
- MDC reprocessing has not yet completed — data gaps may still exist; analyst must wait for cycle to finish before PO Confirmation Resolution.
- SAP integration lag — PO confirmations may not yet be available; analyst should note the time and check again.
- Discrepancy found that cannot be self-resolved — escalate to Global Admin (Sergey/OPI) for period unlock and correction.

---

### 5.2 User Journey: Shop Floor Operator — Shift Data Entry

**Objective:** Ensure all machine events for the current or just-ended shift are correctly classified and all required data (speed, rejects, WIP) is entered within the 30-minute edit window.

**Preconditions:**
- Shift is active or has just ended (edit window open for 30 minutes post-shift)
- Operator is logged in via generic line account (e.g., "Line 7")
- MDC has sent machine events to SDE timeline

**Main Flow:**

1. Open SDE for the current line. View the timeline: green = running, red = unplanned stop, blue = planned activity, grey = excluded time.
2. For each red (unclassified) segment: click to open the reclassification panel. Select downtime reason (category + location). Add comment if required.
3. For planned changeovers: select "Planned Changeover" reason. Select the new Production Order from the SAP-synced PO list. This also signals QSens to recalculate sensor targets for the new material.
4. For test production or training runs: flag the running interval as non-production so it is excluded from KPI calculation.
5. Enter actual machine speed for the current production order and shift.
6. Enter reject data: type, quantity, posting date.
7. If work-in-progress spans the shift boundary (product not yet confirmed by SAP): enter a shift remainder with the unconfirmed volume.
8. Submit and verify the timeline is fully classified (no unresolved red segments).

**Expected Outcome:** All machine events are classified, actual speed and rejects are recorded, and the shift data is complete before the edit window closes.

**Alternative Flows:**
- For long-lasting downtimes spanning multiple shifts: flag as long-lasting; system auto-ends the event when the machine restarts.
- Bulk classify: select multiple stops with the same root cause and apply the same reason in one action.
- Apply to all equipments: cascade one classification across all machines in the work center for the same time interval (system takes the largest overlapping stop from each machine).

**Failure Scenarios:**
- Network disconnect: red status bar displayed. Operator can still enter downtimes manually. When MDC data restores, machine data prevails (except test production, training — where human input overrides).
- 30-minute edit window closes before entry is complete: data is locked; escalation to Process Lead or Production Analyst required.

---

### 5.3 User Journey: Monthly KPI Period Close

**Objective:** Confirm all production data is reconciled and lock the KPI period for global reporting.

**Preconditions:**
- All daily data verification tasks have been completed throughout the month
- It is the 1st of the new month (or the agreed close day within the first 5 business days)

**Main Flow:**

1. Extract all production orders from Proteus and SAP for the closing month.
2. Reconcile volumes — target is zero difference per order. Investigate and resolve any discrepancies (wrong branch, wrong date, wrong order assignment).
3. Confirm reconciliation is complete on Day 1 of close to avoid cascading delays to Finance and global reporting.
4. At the automated trigger (~5th business day of new month): KPI period is locked. All historical data is frozen. KPI data is exported to SAP BW for global reporting.
5. If a post-lock correction is required: request period unlock from Global Admin (Sergey/OPI team).

**Expected Outcome:** KPI period is locked with zero reconciliation discrepancies; data is available in SAP BW for global KPI reporting.

---

### 5.4 User Journey: QSens Sensor Check (Operator / Quality Admin)

**Objective:** Verify that all quality sensors for a production run are within target parameters before or during production.

**Preconditions:**
- A production order is active on the line
- Sensor check has been triggered (by shift, by changeover, or by a scheduled rule)

**Main Flow:**

1. Open QSens Cockpit. View sensor cards grouped by work center. Each card shows green/yellow/red status.
2. For any card showing yellow or red: click to open parameter detail view. See all sensor parameters with actual vs. target ranges.
3. If sensor is out of target: classify the off-reason (machine condition, process issue, etc.).
4. Mark check as complete.
5. Adherence KPI is calculated based on sensor-in-target time during scheduled production.

**Expected Outcome:** All sensors are verified within target; any exceptions are documented with off-reasons for KPI accuracy.

---

## 6. FUNCTIONAL SCOPE

| Feature | Description | Priority |
|---|---|---|
| Smart Downtime Entry (SDE) | Multi-machine timeline, stop reclassification, bulk classify, apply-to-all-equipment, speed entry, reject entry, shift remainders, statistics tab (line-level) | High |
| QSens Cockpit | Sensor cards per work center, parameter drill-down, binary status, shift-level sensor check scheduling | High |
| Office Screen | Near-real-time production shift-level KPI overview for factory management; color-coded; per-machine drill-down (F-01.6) | Medium |
| Aggregated Cockpit (QSens) | Factory-level quality/sensor KPI aggregation by product group; per-workcenter drill-down; connection/clock-sync status (F-02.2) | High |
| OEE Report | Most-used report; all factories depend on it; ~60% of all report runs | High |
| IWS Lost 3 Report | IWS waterfall structure (calendar time → working time → OEE breakdown); second priority report | High |
| Data Review Forms | Import Issue Resolution, PO Confirmation Resolution, Actual Speed, Shift Remainder review, Reject Entry | High |
| Downtime Issue List | Cross-line scan form with direct links to SDE at the flagged shift | High |
| Master Data Management | ~90 admin forms: plant/workcenter/machine hierarchy, downtime reasons, product mappings, sensor parameters, shift schedules | High |
| MDC Event Ingestion | Azure Event Hubs-based ingestion, schema validation, enrichment, classification | High |
| SAP S/4HANA Integration | 18 entity types via SAP Integration Suite; bidirectional; scheduled + near-real-time | High |
| Authentication & RBAC | Azure Entra ID SSO; role-based access; plant-level isolation; generic machine accounts | High |
| KPI Computation & Persistence | Pre-computed, persisted KPI calculations (not runtime); supports Data Lake sync | High |
| Scheduled Email Notifications | ~10+ automated email reports on configurable schedules | Medium |
| QSens Configuration | Global sensor template, local factory config, sensor tag definition, business parameters | Medium |
| Non-SAP factory PO support | Manual PO creation and confirmation for non-SAP factories | Medium |
| Data Lake Export | Azure Event Hubs push for near-real-time data delivery from Day 1 (A-13, supersedes phased approach) | High |
| ServiceNow SDM Integration | Bidirectional: shift schedules (out), planned activities (in) | Medium |
| SAP BW KPI Export | Daily aggregated KPI data pushed to SAP BW | Medium |
| Dynamic workcenter management | Add/remove/reconfigure workcenters without redeployment | Medium |
| Romania waste management API | On-demand REST API for waste tracking; Romania-only | Low |
| 70+ Operational Reports | All remaining reports beyond OEE and IWS Lost 3; prioritized by usage frequency | Medium |
| Factory Map / Homepage | Navigation portal showing user-accessible factories; low visual priority | Low |

---

## 7. BUSINESS RULES (HIGH LEVEL)

> Full rules: `specs/_rules/BUSINESS_RULES.md` (to be created)

| Rule ID | Rule Description | Scope |
|---|---|---|
| BR-001 | Machine data prevails over human input in most cases; exceptions are test production, training, and technical study intervals (where human input overrides) | SDE, MDC ingestion |
| BR-002 | Stops are split by shift boundary for reporting — a stop spanning two shifts is stored as two records | SDE, MDC post-processing |
| BR-003 | KPI calculations must be persisted (pre-computed) — not computed at runtime; runtime computation causes performance bottlenecks and incorrect data lake sync | KPI engine |
| BR-004 | A stop > 10 minutes with a spare part change = Breakdown; without a spare part change = Process Failure | Stop classification |
| BR-005 | Hard delete is prohibited system-wide; all data mutations use soft delete | All modules |
| BR-006 | All timestamps are stored in UTC; displayed in local factory time | All modules |
| BR-007 | Operator edit window: 30 minutes after shift end. Monthly edit window: ~5th business day of the following month. Exceptional reopening requires Global Admin approval | SDE, data editing |
| BR-008 | Shift remainder may only be entered during an active shift, not at order end or month end | SDE, shift remainders |
| BR-009 | Lock Rate = agreed target speed per machine per 90-day cycle (defined in ServiceNow SDM). Operators must enter actual speed AND a reason each shift | SDE, speed entry |
| BR-010 | KPI aggregation unit = work center + shift + production order | KPI engine |
| BR-011 | "Production without reject" is an anomaly flag (unless the line has an inline reclaimer). "Rejects without production" is always an anomaly | Reject validation |
| BR-012 | Reject entry open window = current month + 5 business days of next month | Reject entry |
| BR-013 | Changeover duration variance vs. target is tracked as a KPI; too fast = process steps possibly skipped; too slow = inefficiency | KPI engine |
| BR-014 | MDC reprocessing runs every 90 minutes; re-downloads 720 minutes of line data and overlays it onto records, restoring any data lost to network gaps | MDC ingestion |
| BR-015 | "Apply to all equipments" cascade applies the largest overlapping stop interval from each machine in the work center | SDE |
| BR-016 | Generic machine accounts (one per production line) are supported; personal accounts are not required for shop floor operators | Authentication |
| BR-017 | Plant hierarchy is synced from SAP daily (full snapshot batch) because SAP does not log changes; real-time sync is not possible | SAP integration |
| BR-018 | SAP is the source of truth for plant hierarchy and master data; Proteus must not be used to duplicate or override it | Data integrity |
| BR-019 | QSens sensor targets are dynamically recalculated when a planned changeover is entered in SDE and a new Production Order is selected | QSens, SDE |
| BR-020 | Monthly KPI period lock is triggered ~5th working day of the new month; all historical data is frozen for that period | KPI engine |

---

## 8. DOMAIN & DATA OVERVIEW

### 8.1 Core Entities

| Entity | Description |
|---|---|
| Factory | A physical production site (one of ~27 global sites) |
| Area | Functional area within a factory (e.g., Make & Pack, Filter Making, Logistics) |
| Work Center | A production line within an Area (e.g., MP6, MP12); the primary unit of OEE calculation |
| Equipment / Machine | A physical machine with a serial number; belongs to a Work Center (e.g., Maker, Packer, Wrapper) |
| Shift | A defined time period (typically 8 hours) during which production occurs |
| Production Order (PO) | An SAP-sourced order for producing a specific material quantity on a work center |
| MDC Event | A machine state change or measurement event received from an edge server (stop, volume, quality parameter, reject) |
| Downtime Event | A machine stop that has been classified with a downtime reason and location by an operator |
| Downtime Reason | A coded classification for why a machine stopped (e.g., Breakdown, Changeover, Planned Maintenance) |
| Actual Speed Entry | Operator-entered actual machine throughput for a given shift and production order |
| Reject Entry | Operator-entered defect quantities by reject type for a production order |
| Shift Remainder | A volume booking at shift boundary for unconfirmed work-in-progress |
| KPI Period | A calendar month over which KPIs are calculated; locked after ~5 business days of the following month |
| Sensor / QSens Parameter | A quality sensor with defined target ranges; measured against production during each shift |
| Sensor Check | An operator or system action confirming sensor settings are within target at a specific point in time |
| User | A named or generic account with a role, factory assignment, and set of permissions |

### 8.2 High-Level Relationships

- Factory has many Work Centers; Work Center has many Machines
- Shift belongs to a Work Center; Shift is scheduled within a KPI Period
- Production Order (from SAP) is assigned to a Work Center; one PO may be processed across multiple Shifts
- MDC Event is generated by a Machine; post-processed into Downtime Events and volume records
- Downtime Event is classified by an Operator against a Downtime Reason
- KPI is aggregated per Work Center + Shift + Production Order
- Sensor belongs to a Machine; Sensor Parameters are defined globally and configured locally per factory
- Sensor Check is associated with a Shift and a production Material
- User belongs to one or more Factories (or is a Global Viewer); User has a Role per Factory

### 8.3 Data Ownership & Sources

| Data Domain | Source System | Owner |
|---|---|---|
| Plant Hierarchy (Factory, Area, Work Center, Equipment) | SAP S/4HANA | JTI SAP team; synced daily to Proteus |
| Production Orders and Confirmations | SAP S/4HANA | JTI SAP team; pushed near-real-time |
| Material Master (UoM, KU equivalents) | SAP S/4HANA | JTI SAP team; synced scheduled batch |
| Machine State Events (MDC) | Edge Servers (Cisco/Siskon) | Siskon; pushed via Event Hubs |
| Downtime Classifications | Proteus (operator entry) | Factory operators |
| Actual Speed | Proteus (operator entry) | Factory operators |
| Reject Data | Proteus (operator entry) | Factory operators |
| Shift Schedules | Proteus (admin entry) + ServiceNow SDM | Factory admin; SDM is authoritative for lock rate |
| Sensor Definitions (global) | Proteus (global admin) | Global Admin / OPI team |
| Sensor Configurations (local) | Proteus (local factory admin) | Factory Quality Admin |
| KPI Computations | Proteus (computed) | Proteus platform; source of truth for downstream reporting |
| User Accounts / Roles | Azure Entra ID (Active Directory groups) | IT / System Admin |

### 8.4 Business Concepts

| Concept | Definition & Business Significance |
|---|---|
| OEE (Overall Equipment Effectiveness) | The product of Availability × Performance × Quality. Calculated as: Calendar Hours → subtract Planned Downtime → Available Time → subtract Unplanned Downtime → Run Hours → adjusted for Rate Loss (speed variance) and Quality Loss (rejects). This is the primary KPI for every factory. |
| Lock Rate | The agreed target throughput speed per machine for a 90-day cycle, defined in ServiceNow SDM. Operators compare actual speed against lock rate each shift; deviation requires a reason code. |
| KU (Cigarette Equivalent Unit) | A cross-product volume normalization unit; 1 KU = 1,000 cigarettes. OTP (oral tobacco/non-cigarette) products are converted to KU-equivalents using material-specific conversion factors from SAP. |
| MDC Reprocessing | A system-initiated cycle every 90 minutes that re-downloads the last 720 minutes of machine data and overlays it onto existing records, restoring any data lost to network interruptions. Critical for data completeness before KPI computation. |
| Planned vs. Unplanned Downtime | Planned = scheduled activities (changeovers, maintenance, meetings); these are deducted from Available Time in OEE. Unplanned = unexpected stops > 10 minutes; these are losses against Run Hours. |
| Non-Recorded Stops | Time that is unaccounted for in the OEE waterfall — often speed ramp-up/ramp-down or minor stoppages below the MDC detection threshold. Monitored as a separate metric at month level. |
| KPI Period | A calendar month that is "open" for data entry and correction, and "locked" at approximately the 5th business day of the following month. Once locked, data is frozen and exported to SAP BW. |
| Shift Remainder | A manual volume booking entered by an operator at shift boundary to record work-in-progress that spans shifts. Used to reconcile SAP confirmed volumes against shift-level OEE calculations. Optional per factory. |
| Generic Machine Account | A shared login account tied to a production line (e.g., "Line 7"), not an individual. Used by shop-floor operators who rotate through shifts. Has a fixed set of permissions (SDE + QSens for that line). |

### 8.5 Domain Boundary

| Inside This Domain | Outside This Domain (owned elsewhere or excluded) |
|---|---|
| Machine state ingestion and post-processing | Edge server firmware and MDC collection infrastructure (Siskon) |
| Downtime classification and stop reclassification | DDS Board / IWS Power BI Board (separate system) |
| KPI calculation and period management | SAP BW / GC KPI reporting application (Proteus feeds it, does not own it) |
| Shop floor data entry (SDE, QSens, rejects, speed) | Active Directory / identity management (Proteus reads groups; does not manage them) |
| Operational reporting and notifications | Data Lake internal processing (Databricks / ADF pipelines) |
| Quality sensor monitoring (QSens) | Historical data from legacy GSCA Machinery SQL Server |
| Master data management (admin forms) | Factory-floor hardware and industrial protocols |
| Integration layer to SAP, ServiceNow, Data Lake | SAP MII (retained read-only; no development) |

---

## 9. HIGH-LEVEL FUNCTIONAL REQUIREMENTS

| Requirement ID | Requirement |
|---|---|
| FR-001 | System must ingest MDC machine events from edge servers via Azure Event Hubs at a rate of up to 50,000 events/minute without data loss |
| FR-002 | System must classify ingested MDC events (stop type, volume, reject, quality parameter) and enrich with production context (PO, shift, work center) |
| FR-003 | System must re-process MDC data every 90 minutes for the previous 720 minutes to backfill gaps caused by network interruptions |
| FR-004 | System must provide a Smart Downtime Entry (SDE) interface showing a real-time machine timeline per work center, updated within 5 minutes of an MDC event |
| FR-005 | System must allow operators to reclassify unplanned stops with downtime reason and location codes, individually or in bulk |
| FR-006 | System must support "apply to all equipments" cascade in SDE, applying one classification to all machines in a work center for the same time interval, using the largest overlapping stop from each machine |
| FR-007 | System must allow operators to enter actual machine speed per shift per production order, with a mandatory reason code |
| FR-008 | System must allow reject data entry (type, quantity, posting date) from SDE and from a dedicated office-user reject entry form |
| FR-009 | System must allow optional shift remainder entry during active shifts (not at order end or month end) |
| FR-010 | System must enforce a 30-minute edit window after shift end; after which, shift data is locked |
| FR-011 | System must support planned changeover entry in SDE, including selection of the next Production Order from a SAP-sourced list; this must trigger QSens sensor target recalculation |
| FR-012 | System must pre-compute and persist KPI calculations (OEE, availability, performance, quality, MTBF) per work center + shift + production order; runtime calculation of KPI data is not permitted |
| FR-013 | System must provide a QSens Cockpit showing sensor status (green/yellow/red) per work center, with sensor parameter drill-down |
| FR-014 | System must support configurable QSens sensor check scheduling (per shift, per day, per week, or after specified downtime types) |
| FR-015 | System must provide a Data Review dashboard (or equivalent forms) covering: Import Issue Resolution, PO Confirmation Resolution, Actual Speed review, Shift Remainder review, Reject records review |
| FR-016 | System must provide a Downtime Issue List with cross-line anomaly flags and direct links to SDE at the relevant shift |
| FR-017 | System must provide an Office Screen showing near-real-time, multi-line, shift-level production KPI overview for factory management, with color-coded work center states and per-machine drill-down (reject ratio, top stops, volume attainment, MTBF) |
| FR-017b | System must provide an Aggregated Cockpit (QSens) showing factory-level quality/sensor KPI aggregation by product group, per-workcenter drill-down, and machine connection / clock-sync status. The Aggregated Cockpit is a distinct feature from the Office Screen (FR-017) and the two must never be conflated |
| FR-018 | System must provide an OEE report, IWS Lost 3 report, Downtime Statistics, Reject reports, and non-recorded stops metric at month level |
| FR-019 | System must provide ~90 master data management forms for work centers, machines, downtime reason catalogs, product mappings, sensor definitions, and shift schedules |
| FR-020 | System must lock the KPI period at approximately the 5th business day of the following month; exceptional reopening requires Global Admin approval |
| FR-021 | System must integrate with SAP S/4HANA for 18 entity types via SAP Integration Suite; integration must support both scheduled batch and near-real-time push |
| FR-022 | System must integrate with ServiceNow SDM: push shift schedules (outbound) and receive planned activities (inbound) |
| FR-023 | System must export data to the enterprise Data Lake via Azure Event Hubs (near-real-time push); SQL-views phased approach superseded 2026-06-02 (A-13) |
| FR-024 | System must export aggregated KPI data to SAP BW daily |
| FR-025 | System must authenticate users via Azure Entra ID SSO; enforce role-based access control; enforce plant-level data isolation |
| FR-026 | System must support generic shared machine accounts for shop floor operators (one per production line) |
| FR-027 | System must support manual PO creation and PO confirmation entry for non-SAP factories |
| FR-028 | System must support global factory onboarding through configuration alone, without code changes |
| FR-029 | System must support multi-timezone display (all storage in UTC; display in local factory time) |
| FR-030 | System must support multi-language UI with a minimum of English, Mandarin, Arabic, Russian, Ukrainian, Filipino, and Spanish |
| FR-031 | System must use soft delete for all data mutations; hard delete is not permitted anywhere in the system |
| FR-032 | System must provide automated email notifications (~10+) on configurable schedules |
| FR-033 | System must provide the Romania waste management API endpoint (on-demand REST; Romania-only) |

---

## 10. NON-FUNCTIONAL REQUIREMENTS (NFRs)

> Full NFR specifications: `specs/_quality/NFR.md` (to be created)

> **⚠ Provisional — pending BIA (OQ-19) and concurrency validation (OQ-20).** The availability,
> RTO, RPO, and concurrent-user targets below are not yet substantiated by a Business Impact
> Assessment. Treat them as working assumptions, not firm acceptance thresholds, until the BIA
> completes.

| Category | Requirement | Measurement |
|---|---|---|
| Performance | Operational dashboards (SDE, Office Screen) render within 3 seconds for current shift data under normal load | ≤ 3 seconds at ~500 concurrent users |
| Performance | 95th-percentile API response for standard calls | ≤ 500 ms |
| Performance | Reporting queries | ≤ 5 seconds |
| Performance | Nightly aggregation and archival jobs complete without impacting online operations | ≤ 2 hours |
| Ingestion throughput | Sustain peak MDC event rate without data loss | 30,000–50,000 events/minute |
| Data freshness | Edge-to-cloud latency for machine events appearing in SDE | ≤ 10 seconds under normal conditions |
| Data freshness — Data Lake | End-to-end latency (Day 1, Event Hubs) | ≤ 5 minutes |
| Availability | Core services (SDE, QSens, ingestion) uptime | 99.9% (≤ 43.8 min/month unplanned) |
| Availability | RPO for core services | ≤ 5 minutes |
| Availability | RTO for core services | ≤ 30 minutes |
| Resilience | Edge servers buffer data during cloud outages; automatic resynchronization on reconnection | No MDC event loss during transient connectivity gaps |
| Scalability | Architecture accommodates 2× current volumes without architectural change | ~2,600 machines, 60 edge servers via configuration |
| Scalability | Concurrent user capacity | ~500 at go-live; scalable to 1,000 without redesign |
| Security | SSO via Azure Entra ID; no local credential storage; MFA for administrative roles | Per Azure Entra ID policy |
| Security | All communication over TLS 1.2+ | Enforced at all layers |
| Security | All persistent data encrypted at rest using platform-managed keys | Azure SQL / Blob default encryption |
| Security | Centralized secrets management; no secrets in source code or config | Azure Key Vault |
| Security | Immutable audit log of security events with user identity and timestamp | Retained ≥ 12 months |
| Security | Backend services in private networks; no direct internet exposure | Azure VNet / Private Endpoints |
| Maintainability | All infrastructure via Infrastructure as Code; no manual production configuration | Terraform or Bicep |
| Maintainability | Automated CI/CD with zero-downtime rolling updates | Azure DevOps pipelines |
| Observability | All services emit metrics and logs to centralized platform | Azure Monitor + Application Insights |
| Observability | Automated alerts for: pipeline lag, edge disconnection, resource saturation | Alert thresholds defined per service |
| Observability | Application logs retained | 90 days hot; 12 months archive |
| Accessibility | Desktop web browsers; optimized for 1920×1080+ | Current + previous major version of Chrome and Edge |
| Accessibility | Responsive layout for future mobile/tablet use without re-implementation | CSS responsive framework |
| Internationalization | All UI strings externalized; i18n framework; English primary | Minimum 7 languages at launch |

---

## 11. DEPENDENCIES

| Dependency | Type | Impact |
|---|---|---|
| SAP S/4HANA + SAP Integration Suite | External | Core integration for production orders, confirmations, plant hierarchy, material master; Integration Suite live since April 2026; SAP-side changes take 6+ months |
| Azure Entra ID | External | Authentication and RBAC; must be connected to JTI's corporate identity provider |
| Edge Servers (Siskon/Cisco) | External | Source of all MDC events; fixed integration boundary; Siskon must deliver stable JSON payloads via push |
| ServiceNow SDM | External | Source of planned activities (inbound); destination for shift schedules (outbound) |
| Enterprise Data Lake (CDLP) | External | Downstream consumer of all operational and master data; Azure Event Hubs from Day 1 (A-13, decided 2026-06-02); CDLP team (Sergey Kulinchenko) must confirm implementation timeline |
| SAP BW | External | Downstream consumer of daily aggregated KPI data |
| Azure API Management (APIM) | External | Mandated API gateway for SAP and ServiceNow integrations |
| Azure Event Hubs | Internal (Azure) | Core ingestion infrastructure for MDC events |
| Azure Container Apps | Internal (Azure) | Compute platform for all Proteus services |
| Azure SQL Database | Internal (Azure) | Primary persistence layer |
| Azure Service Bus | Internal (Azure) | Async messaging for ServiceNow and other integrations |

---

## 12. RISKS, QUESTIONS & DECISIONS

### 12.1 Risks

| Risk | Impact | Mitigation |
|---|---|---|
| SAP MII end-of-support deadline (2027) — schedule slips past the support cliff | Critical | Lock scope early; maintain contingency for extended SAP support if go-live slips beyond Q4 2027 |
| SQL stored-procedure re-architecture — business logic embedded in SPs, some undocumented (~10K LOC KPI logic alone) | High | Discovery sprint to catalog and classify all procedures; prioritize critical-path logic; pair with domain experts |
| MDC/cloud scope boundary — Siskon must produce a stable, baselined data contract before cloud integration development begins | High | Freeze data contract before development starts; joint working group with formal sign-off gate |
| Big-bang go-live — simultaneous rollout to all factories with unfamiliar UI carries high adoption risk | High | Parallel testing with power users; training and onboarding; SAP MII kept as read-only fallback until end 2027 |
| Report and form volume — ~90 admin forms + 70+ reports must be rebuilt, creating large regression surface | High | Automate template generation; prioritize by usage frequency; validate against production screenshots |
| QSens formula complexity — sensor-calculation logic is undocumented and embedded in legacy code | Medium-High | Formula audit workshop with domain experts; verified catalogue with unit tests before reimplementation |
| Four-party team coordination (BTS GSC, GDC, Siskon, Intellias) — alignment risk on architecture and release cadence | Medium | Single architecture authority; shared bi-weekly sync; RACI matrix for decision rights |
| Data consistency during parallel testing — dual feeds create source-of-truth ambiguity | Medium | Designate SAP MII as system of record until cutover; reconciliation jobs comparing both systems |
| Historical data exclusion — incompatible data models may make future historical migration expensive | Medium | Design new data model with historical-import interface; document mapping even if migration is deferred |

### 12.2 Open Questions

| OQ | Question | Owner | Target Resolution Date |
|---|---|---|---|
| OQ-01 | Daily locking mechanism: should shifts be marked as "verified" by the process lead? What granularity (shift/day/month)? Requires IWS business alignment | Andronic / IWS business | TBD |
| OQ-02 | Cross-production machine linking form: how does Proteus handle maker-packer relinking between production lines (~3 factories)? | BA + GDC | TBD |
| OQ-03 | Parallel production volume splitting (Philippines — multiple lines, one SAP PO): how is performance measured? | Andronic / BA | TBD |
| OQ-04 | Tobacco primary processing (primaries): separate session needed — different line structure and sub-processes | Florin / Andronic | TBD |
| OQ-05 | SDM Lock Rate integration: currently not integrated; Proteus could receive lock rate data from ServiceNow to auto-validate speed entries. In scope from December? | Andronic | December 2026 |
| OQ-06 | Global Admin and System Admin detailed permission mapping: not yet decomposed below team level (Sergey = Global Admin, Florin = System Admin) | Andronic / Florin | TBD |
| OQ-07 | Email notification subscription model: currently hard-coded AD lists per factory; self-service subscription UI needed? | Andronic | TBD |
| OQ-08 | Machine time synchronization: does Proteus need automated escalation/alerting when machine clock drifts vs. server time? | Marco / BA | TBD |
| OQ-09 | Data Lake integration architecture: Azure Event Hubs vs. Databricks Zero Bus — A-13 assumes Event Hubs but Databricks Zero Bus was proposed as an alternative; resource allocation and CDLP implementation timeline outstanding; cost assessment not yet complete | Sergey C / Florin | TBD |
| OQ-10 | Non-recorded stops metric: confirm formal scoping decision for native reporting in Proteus | Andronic | TBD |
| OQ-11 | QSens validate with global quality team (Jerry-san, Japan): confirm QSens cockpit redesign works for global quality users | Florin | TBD |
| OQ-12 | Simplified persona model confirmation: Andronic's 4+2+1 model needs alignment with broader team | Andronic / Florin / Sergey | TBD |
| OQ-13 | Scheduling UI design: confirmed as one of three backbone elements; no design sessions held yet — high-risk area | BA / UX | TBD |
| OQ-14 | Small timeline interval clickability: known UX problem in SDE for short stop intervals; solution not yet defined | UX team | TBD |
| OQ-15 | "Machinery Light Analysis 2.0" (Andronic's form review document): needs to be shared with BA/UX team to inform form redesign | Andronic | TBD |
| OQ-16 | Full-field audit trail: Andronic indicated desire but noted cost vs. value tradeoff; decision not confirmed | Andronic / BA | TBD |
| OQ-17 | **[DEFERRED — Personas v1.4]** P-05 was split in v1.3 into P-05a (GSCA Global Admin, OPI-DPA) and P-05b (Global QSens Admin, OPI-GC). Section 4 not yet updated to reflect this split. OPI-DPA / OPI-GC acronym expansions also pending confirmation. | BA | TBD |
| OQ-18 | **[DEFERRED — Personas v1.4]** P-02c (Production Machine Expert) added as a distinct persona in v1.1 — owns MDC mapping and QSens sensor tag codes; not substitutable by P-02a/P-03a. Section 4 currently includes this as "Production Machine Expert" but does not carry the v1.4 clarifications on write-access scope and factory role patterns. | BA | TBD |
| OQ-19 | **[DEFERRED — RAID R-9 / R-10 / I-1, severity 25]** No Business Impact Assessment (BIA) exists for Project Proteus. A-14 confirmed false (no legacy BIA to reuse). All NFRs for availability, RTO, RPO, and HA/DR in Section 10 are currently unsubstantiated. BIA workshops with JTI Enterprise Architect (D-4) must be scheduled before infrastructure design is locked. | Florin / Enterprise Architect | TBD |
| OQ-20 | **[DEFERRED — RAID R-13]** Concurrent reporting user estimates are inconsistent across documents (~500 vs. ~1,500). Current NFR target (~500) in Section 10 may be undersized. Must be validated against actual active user data from legacy system. | BA / Marco | TBD |
| OQ-21 | **[DEFERRED — RAID R-8]** Single Azure region deployment may introduce unacceptable latency for non-European factories (e.g., Japan, Philippines). A latency/performance PoC for Asian edge users is called out in the roadmap but no decision recorded. | Marco / Florin | TBD |
| OQ-22 | **[DEFERRED — RAID D-3]** SOC security assessment listed as a high-importance dependency with no owner or timeline assigned. | Florin / IT Security | TBD |
| OQ-23 | **[DEFERRED — RAID D-6]** Support model (who handles support post-go-live) requires alignment with Service Delivery (Radu Lazarescu). Blocked by BIA completion. | Florin / Radu Lazarescu | TBD |

### 12.3 Key Decisions

| Decision | Reason | Date |
|---|---|---|
| Big-bang go-live across all factories simultaneously | Phased rollout adds complexity; SAP MII read-only fallback until end 2027 mitigates risk | May 2026 |
| No historical data migration | Incompatible data models; clean cutover is faster and reduces risk | May 2026 (Florin) |
| SAP is source of truth for plant hierarchy; daily batch sync (not real-time) | SAP does not log changes; real-time sync not possible without SAP-side investment | May 2026 (Andronic) |
| DDS Board is out of scope | Separate system; different business owner; Proteus provides source data only | May 2026 (Andronic) |
| Soft delete everywhere; hard delete prohibited | Hard delete is a legacy bug that causes data integrity issues | May 2026 (Andronic) |
| All timestamps stored in UTC | Local-time storage in legacy system was a major cause of data lake reconciliation errors | May 2026 (Andronic) |
| Azure Event Hubs as MDC ingestion mechanism (chosen over Azure IoT Hub and Direct REST) | Best fit for volume, throughput, and consumer group model needed for Data Lake | May 2026 (Architecture) |
| User roles redesigned from scratch based on IWS job profiles | Legacy GSCA Machinery security roles are not meaningful to business users; IWS roles (cell lead, process lead, operator) are universally understood | May 2026 (Andronic / Sergey) |
| KPI calculations must be pre-computed and persisted (not runtime) | Runtime computation in Aggregated Cockpit takes ~1 hour every 4 hours; also prevents correct Data Lake sync | May 2026 (Andronic) |
| Email templates are hard-coded (no management UI) | Change frequency is ~once per 2 years; management UI not worth the investment | May 2026 (Andronic) |
| Data-to-Lake: Event Hubs from Day 1 (supersedes phased approach) | Phased SQL-views approach (A-12) superseded by A-13; agreed 2026-06-02 by Florin, Sergey C., Marco, Eduardo; eliminates 4-hour data latency from go-live; CDLP team to confirm their implementation timeline | 2 June 2026 |
| Japan-specific features excluded from scope | Only globally applicable features in Proteus | May 2026 (Andronic) |
| Generic machine accounts supported (one per line) | Shop floor reality: shared PCs, rotating operators, no individual login on the production floor | May 2026 (Andronic) |
| Canonical term is "Shift Remainder" (not "Shift Reminder") | "Reminder" was a long-standing misspelling; the concept is leftover/unconfirmed volume (pallets) at shift boundary. Enforced via GLOSSARY.md | June 2026 (Julia) |
| Data residency: Frankfurt (Germany) Azure region only; no local/in-country storage | Single-region deployment confirmed; in-country storage not required (OQ-25 resolved). Non-European latency tracked separately (OQ-21) | June 2026 (Julia) |
| Audit log (change traceability) in scope; concrete tobacco regulations (e.g., EU TPD) out of scope | All data changes logged with user/timestamp/before-after; specific regulatory compliance deferred (OQ-26 resolved) | June 2026 (Julia) |
| Office Screen and Aggregated Cockpit are distinct features (F-01.6 vs F-02.2) | Office Screen = production shift-KPI overview (M-01); Aggregated Cockpit = factory-level quality/sensor KPI aggregation (M-02 QSens). Never conflate | June 2026 (Julia) |

---

## 13. RELEASE / PHASE PLANNING

> Full delivery detail, feature IDs, effort estimates, and wave definitions are maintained in **`General/Implementation Scope & Roadmap.xlsx`** (v0.2, 2026-06-15). This section summarizes the key milestones and structure; consult the xlsx for delivery planning and scheduling.

### 13.1 Programme Phases

| Phase | Scope | Timeline | Status |
|---|---|---|---|
| Discovery Phase | Architecture direction, integration patterns, engineering backbone, UI/UX vision for key journeys, implementation roadmap | 11 May – 12 June 2026 | **Complete.** Deliverables: ADRs, integration patterns, effort estimates, Figma prototypes for core workflows |
| Implementation (Phase 1) | Full platform replacing SAP MII: all 7 modules (see §13.2), ~90 admin forms, 80+ reports, all integrations, all factories | Jul 2026 – **Aug 2027** (optimistic) / **Oct 2027** (pessimistic) | In progress. Big-bang go-live across all factories simultaneously; SAP MII retained read-only for up to 6 months post-cutover |
| Post-Release Enhancements | Data quality dashboards, operator self-correction, extended SDE features, lock rate SDM integration, on-screen notifications, downtime task workflows | After go-live; timeline TBD | Backlog. See §13.4 |
| Future | Tobacco primary processing; SmartSENS for primary processing; additional analytics/ML capabilities | Post-release; scope TBD | Not started |

### 13.2 Phase 1 Module Breakdown

Phase 1 delivers 7 platform modules. All are in scope for the initial go-live release.

| Module ID | Module Name | Key Features |
|---|---|---|
| M-01 | Production Data Capture (Machinery) | SDE, reject entry, actual speed entry, shift remainders, data review & correction forms, Office Screen, non-SAP PO entry & confirmation |
| M-02 | Quality Monitoring (QSens) | QSens cockpit, aggregated cockpit, local and global sensor configuration, sensor adherence KPI engine, sensor check rules & procedures, sensor inactive rules, quality breakdown alerts, sensor reject reason assignment |
| M-03 | MDC Ingestion & Classification (SmartMDC) | Edge event ingestion pipeline, MDC classification hierarchy management, global breakdown & reject libraries and machine templates, local translation and mapping workflow, MDC raw data reports |
| M-04 | Master Data & Configuration | Organizational hierarchy, workcenters & machines master data, shift schedule management, downtime configuration, translation management, system config & factory preferences, machine go-live activation, reject type definitions |
| M-05 | Performance KPI & Reporting | Central KPI calculation engine, OEE report, reject rate report, downtime & reject list reports, common report framework, email notifications & alerts, KPI period management & month-end locking, extended report set (~75 reports), team comparison report |
| M-06 | External Integration | SAP S/4HANA master data sync, SAP S/4HANA PO & confirmation exchange, ServiceNow SDM integration, Data Lake export (Event Hubs), GSCA KPI export to SAP BW, local waste management API, non-SAP factory integration (QIMS/KIMS), APIM gateway configuration |
| M-07 | Platform Services | Authentication & SSO, RBAC & user role management, audit trail & data change history, navigation / menu / homepage, global navigation search, reusable table search & filter component, favourites, data quality check framework (phase TBD) |

### 13.3 Delivery Structure

Implementation is organized into 15 delivery waves (Wave 0–14), running July 2026 through October 2027. Full wave definitions, feature-to-wave assignments, and effort estimates are maintained in `Implementation Scope & Roadmap.xlsx`.

| Wave | Approx. Timing | Objective |
|---|---|---|
| Wave 0 | M1–M2 | Application foundation: authentication, RBAC, navigation |
| Waves 1–2 | M2–M5 | Core master data, shift configuration, downtime configuration |
| Wave 3 | M5–M7 | First production data capture: SDE, speed entry, reject entry, shift remainders |
| Wave 4 | M8–M10 | Data review & correction forms, Office Screen, non-SAP PO |
| Waves 5–6 | M3–M8 | MDC ingestion and classification pipeline |
| Waves 7–8 | M6–M11 | KPI engine, common report framework, first operational reports |
| Waves 9–10 | M7–M12 | QSens quality monitoring: cockpit, configuration, rules, sync |
| Wave 11 | M4–M10 | External integrations (incremental — SAP, ServiceNow, Data Lake, BW) |
| Wave 12 | M11–M12 | Extended reporting, KPI period management, email notifications |
| Wave 13 | M10–M12 | Platform services: audit trail, global search, favourites, data quality check |
| Wave 14 | M10–M13 | Stabilization, UAT, rollout readiness, legacy parity reconciliation |

**Go-live targets:** Optimistic **Aug 2027** · Pessimistic **Oct 2027**

**Estimated engineering effort:** 1,724 days (optimistic, with 30% AI productivity factor applied) to 2,664 days (pessimistic). Source: `Implementation Scope & Roadmap.xlsx`.

**Non-feature delivery work** (architecture baseline, Azure environments, CI/CD, performance testing, shadow validation, cutover, hypercare) is tracked separately in the xlsx (PRT-DW-01 through PRT-DW-13).

### 13.4 Post-Release Enhancements

The following items were scoped during Discovery but deferred from Phase 1. They represent confirmed future backlog items. Full list with priorities maintained in `Implementation Scope & Roadmap.xlsx` (Post-Release Scope tab).

| ID | Area | Item | Priority |
|---|---|---|---|
| IMP-08 | KPI Data Control | At-a-glance data quality dashboard | High |
| IMP-09 | KPI Data Control | Automated business rule validation | High |
| IMP-10 | KPI Data Control | Pre-filtered contextual navigation (drill-down from anomaly to source record) | High |
| IMP-11 | SDE | Operator self-correction workflow | High |
| IMP-13 | SDE | Real-time operator data entry validation | High |
| IMP-05 | Admin Forms | Downtime Reason – Task Assignment | Medium |
| IMP-06 | Admin Forms | Downtime Task List | Medium |
| IMP-07 | Admin Forms | Reject types report enhancement (link reject to equipment) | Medium |
| IMP-12 | SDE | Extended activity auto-split | Medium |
| IMP-14 | Master Data | Units of measure refactoring | Medium |
| IMP-03 | Platform | On-screen notifications | Low |
| IMP-04 | Platform | Change notifications | Low |
| IMP-16 | System Integration | Lock rate integration with ServiceNow SDM | Low |
| IMP-17 | System Integration | Real machine speed derivation from sensor data | Low |

### 13.5 Testing Approach Pre-Go-Live

Unit testing, integration testing, performance and load testing (with full concurrency simulation), KPI/formula validation against legacy reference outputs, UAT with JTI power users, shadow validation / dual-run against live SAP MII data, legacy parity reconciliation, security and compliance validation. SAP MII retained as read-only fallback for up to 6 months post-cutover.

---

## 14. RELATED ARTIFACTS & REFERENCES

| Artifact | Purpose | Location |
|---|---|---|
| `PERSONAS.md` | Canonical user persona definitions | `specs/_context/PERSONAS.md` |
| `GLOSSARY.md` | Domain terminology and definitions (canonical) | `specs/_context/GLOSSARY.md` |
| `FEATURE_INVENTORY.md` | Canonical feature list (F-IDs) and traceability spine | `specs/_context/FEATURE_INVENTORY.md` |
| `CURRENT_SYSTEM_ANALYSIS.md` | Analysis of legacy GSCA Machinery system, forms, and reports | `specs/_context/CURRENT_SYSTEM_ANALYSIS.md` |
| `IN_SCOPE_OUT_OF_SCOPE.md` | Extended scope boundary reference | `specs/_context/IN_SCOPE_OUT_OF_SCOPE.md` |
| `DOMAIN_MODEL.md` | System entities, relationships, and bounded contexts | `specs/_domain/DOMAIN_MODEL.md` |
| `STATE_MODELS.md` | Lifecycle/state transitions (machine, downtime event, KPI period) | `specs/_domain/STATE_MODELS.md` |
| `DATA_DICTIONARY.md` | Field-level data definitions, types, units, sources | `specs/_domain/DATA_DICTIONARY.md` |
| `EVENT_MODEL.md` | Event flows, triggers, state transitions, orchestration | `specs/_domain/EVENT_MODEL.md` |
| `BUSINESS_RULES.md` | Full shared business rules (KPI logic, OEE, downtime, QSens) | `specs/_rules/BUSINESS_RULES.md` |
| `PERMISSIONS_MATRIX.md` | Role × feature access matrix | `specs/_rules/PERMISSIONS_MATRIX.md` |
| `NFR.md` | Full non-functional requirements specifications | `specs/_quality/NFR.md` |
| `SECURITY_REQUIREMENTS.md` | Detailed security requirements | `specs/_quality/SECURITY_REQUIREMENTS.md` |
| `DECISIONS.md` | Running log of client-confirmed decisions | `specs/_elicitation/DECISIONS.md` |
| Solution Specification (Updated) | Architecture and integration specification from GDC | `Solution Specification for Proteus Project - Updated.docx` |
| KPI Reporting User Flow — Daily | Authoritative daily KPI flow diagram (updated May 26) | `KPI Reporting User Flow - Daily (Updated May 26).html` |
| BA Scope for Discovery Phase | BA scope and delivery plan for Discovery Phase | `BA scope for Discovery phase.docx` |
| Meeting Transcripts | Elicitation session transcripts (Andronic, Florin, Stanislav, Sergey) | `specs/_elicitation/meetings/` |

---

## 15. APPROVALS

| Name | Role | Approval Status | Date |
|---|---|---|---|
| Florin | IT Architect / System Owner (JTI) | Pending | |
| Andronic | GSCA Machinery Business Owner (JTI) | Pending | |
| Sergey | OPI Global Operations (JTI) | Pending | |
| Julia Shcherbyna | BA Lead (Intellias) | Pending | |

---

## APPENDIX

### A. Technology Stack

| Area | Technology |
|---|---|
| Backend | .NET (C#) |
| Frontend | React / Node.js |
| Primary Database | Azure SQL Database |
| Event Streaming | Azure Event Hubs |
| Async Messaging | Azure Service Bus |
| Compute | Azure Container Apps |
| Identity & Access | Azure Entra ID |
| API Gateway | Azure API Management (APIM) |
| Standard Reporting | Power BI Embedded |
| Real-time Dashboards | React (custom, with SignalR) |
| Monitoring | Azure Monitor + Application Insights |
| CI/CD | Azure DevOps |
| Infrastructure as Code | Terraform or Bicep (TBD per JTI standards) |
| File Storage | Azure Blob Storage |

### B. System Architecture — Module Overview

The platform is structured as a modular monolith with six enforced domain modules:

| Module Code | Module Name | Key Responsibilities |
|---|---|---|
| ING | Ingestion | Receives JSON events from edge servers; schema validation, enrichment, event classification; MDC reprocessing |
| MCH | Machinery | SDE tracking, reject entry, speed entry, shift remainders, Office Screen, downtime statistics |
| QSN | QSens | QSens Cockpit, Aggregated QSens Cockpit, sensor check workflows, adherence formulas |
| MDT | Master Data | ~90 configuration forms — plant/work center/machine hierarchy, product catalogs, sensor parameters, shift templates |
| RPT | Reporting & KPI | ~20 KPI formula groups (~10K LOC legacy equivalent), 70+ reports, scheduled email notifications |
| INT | Integration Layer | Event streaming, scheduled export, async messaging, API gateway — all external system communication encapsulated here |

### C. Plant Hierarchy

```
Company (JTI)
  └── Country / Region (e.g., Ukraine, Romania, Philippines)
        └── Factory (~27 factories globally)
              └── Area (e.g., Make & Pack, Filter Making, Logistics)
                    └── Work Center / Production Line (e.g., MP6, MP12)
                          └── Equipment / Machine (Maker, Packer, Wrapper, 
                                                    Cellofanner, Cartooner, 
                                                    Palletizer, Tax Stamper, etc.)
                                └── Construction Type / Group Assembly
                                      └── Assemblies / Units
```

Functional locations (~250,000 rows total) = virtual places; synced daily from SAP. Equipment = physical assets with serial numbers. Changes can occur daily (e.g., machine moved between lines).

### D. Production Line Variants (Edge Cases)

| Variant | Description | Factories Affected |
|---|---|---|
| Standard linear | One production sequence, one line | ~90% of volume |
| Parallel flows | One maker feeding two or more packers | Germany (Trier), Tenerife, OTP lines |
| Parallel production | Multiple lines sharing one SAP PO | Philippines only |
| Cross-production | Maker physically re-linked between lines for product flexibility | ~3 factories, ~2% of volume |

### E. Key Integrations — Data Flow Overview

```
Edge Servers (Siskon/Cisco)
    → [JSON push] → Azure Event Hubs → ING module (ingestion + enrichment)
                                              ↓
                                        Azure SQL Database (operational store)
                                              ↓
                        ┌─────────────────────┴──────────────────────┐
                        ↓                                            ↓
               SAP BW (daily KPI batch)                Data Lake (Azure Event Hubs, near-real-time)
                        
SAP S/4HANA → [SAP Integration Suite] → INT module → Master Data, PO/Confirmation sync
ServiceNow SDM ← → INT module (shift schedules out, planned activities in)
```
