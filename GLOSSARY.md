# GLOSSARY — Project Proteus

**Status:** Draft v0.1 · **Last Updated:** 2026-06-25 · **Owner:** Julia Shcherbyna (BA Lead)
**Repository Location:** `specs/_context/GLOSSARY.md`
**Related:** [`PRD.md`](PRD.md) · [`personas.md`](personas.md) · `specs/_rules/BUSINESS_RULES.md` · `specs/_rules/PERMISSIONS_MATRIX.md`

---

## 0. Purpose & How to Use This File

This glossary is the **canonical source of terminology** for Project Proteus. It exists to keep
language consistent across all BA, design, and AI-generated artifacts (feature requirement docs,
tickets, `specify.md`, code). In a spec-driven, AI-first flow, inconsistent terms cause the model
to treat one concept as two — so terminology governance is a correctness requirement, not a style
preference.

**Conventions**

- The **bold canonical term** is the spelling to use in new artifacts.
- *Aliases* are accepted synonyms that may appear in source/legacy material; map them to the canonical term.
- `[confirm]` marks a definition or expansion not yet verified with a stakeholder — treat as **[Low]** confidence.
- When the PRD, personas, or a feature doc conflicts with this file on a *term's meaning*, this file wins. Raise the conflict as an Open Question; do not silently resolve it.

**For AI agents:** Load this file as context for any requirements-generation or gap-analysis task.
Use canonical terms in all output.

---

## 1. Acronyms & Abbreviations

| Acronym | Expansion | Quick definition |
|---|---|---|
| AD | Active Directory | Microsoft directory service; source of user accounts and security groups (via Entra ID). |
| ADF | Azure Data Factory | Data Lake-side pipeline tooling (CDLP-owned; outside Proteus). |
| APIM | Azure API Management | Mandated API gateway for SAP and ServiceNow integrations. |
| BA | Business Analyst | — |
| BIA | Business Impact Assessment | Analysis substantiating availability/RTO/RPO targets; not yet done (OQ-19). |
| BR | Business Rule | Cross-cutting rule; see `BUSINESS_RULES.md`. |
| BTS GSC | BTS Global Supply Chain `[confirm]` | JTI ownership party for the programme. |
| CDLP | Cloud Data Lake Platform `[confirm]` | JTI enterprise Data Lake team/platform (Databricks/ADF). Downstream consumer. |
| DDS | Daily Direction Setting | IWS daily management meeting (~8:45 AM) where KPI data is reviewed. |
| EAM | (SAP) Enterprise Asset Management | SAP module; source of plant hierarchy / functional locations. |
| Entra ID | Azure Entra ID (formerly Azure AD) | Identity, SSO, RBAC group source. |
| FR | Functional Requirement | — |
| GDC | Global Delivery Center `[confirm]` | Architecture party in the hybrid delivery team. |
| GSCA | GSCA Machinery | Legacy manufacturing operations platform being replaced. |
| HA/DR | High Availability / Disaster Recovery | — |
| IaC | Infrastructure as Code | Terraform or Bicep. |
| IWS | Integrated Work Systems | JTI's operational discipline/methodology driving data quality requirements. |
| KPI | Key Performance Indicator | — |
| KU | Cigarette Equivalent Unit | 1 KU = 1,000 cigarettes; cross-product volume normalization unit. |
| MDC | Machine Data Collection | Capture of machine state/volume/quality events at the edge. |
| MFA | Multi-Factor Authentication | Required for administrative roles. |
| MII | (SAP) Manufacturing Integration and Intelligence | Legacy platform GSCA Machinery is built on; end-of-support 2027. |
| MTBF | Mean Time Between Failures | Reliability KPI. |
| NFR | Non-Functional Requirement | — |
| OEE | Overall Equipment Effectiveness | Primary KPI = Availability × Performance × Quality. |
| OPI | Operational Performance Improvement | JTI global performance team; owns global standards. |
| OPI-DPA | OPI — Digital Performance Analysis `[confirm]` | Performance-side global admin (persona P-05a). |
| OPI-GC | OPI — Global Quality `[confirm]` | Quality-side global admin (persona P-05b). |
| OTP | Oral Tobacco Products | Non-cigarette products; converted to KU-equivalents. |
| PCO | Process/edge collection `[confirm]` | Edge-side mechanism delivering machine stop/state events (~3–5 min). |
| PO | Production Order | SAP-sourced order to produce a material quantity on a work center. |
| PoC | Proof of Concept | — |
| PRD | Product Requirements Document | This programme's context anchor. |
| QA | Quality Assurance | Testing role/discipline. |
| RACI | Responsible/Accountable/Consulted/Informed | Decision-rights matrix. |
| RBAC | Role-Based Access Control | — |
| RPO | Recovery Point Objective | Max acceptable data loss window (target ≤ 5 min). |
| RTO | Recovery Time Objective | Max acceptable downtime to restore (target ≤ 30 min). |
| SAP BW | SAP Business Warehouse | Downstream consumer of daily aggregated KPI data. |
| SAP S/4HANA | SAP ERP (S/4HANA) | Source of orders, confirmations, plant hierarchy, material master. |
| SDE | Smart Downtime Entry | Shop-floor operator interface; core data-capture screen. |
| SDM | Service Delivery Management (ServiceNow) | Source of planned activities; destination for shift schedules; lock-rate authority. |
| SLA | Service Level Agreement | — |
| SOW | Statement of Work | — |
| SSO | Single Sign-On | Via Entra ID. |
| TPD | (EU) Tobacco Products Directive | Industry regulation incl. track-and-trace; concrete regs out of scope (see §10 Roles, Access & Security — Audit Log). |
| UAT | User Acceptance Testing | With JTI power users pre-go-live. |
| UoM | Unit of Measure | Material measurement unit from SAP. |
| UTC | Coordinated Universal Time | All timestamps stored in UTC; displayed in local factory time. |
| WI | Work Item | Implementation ticket; status "Ready for Spec" triggers `specify.md` generation. |
| WIP | Work In Progress | Unconfirmed in-progress production; reconciled via Shift Remainder. |

---

## 2. Platforms, Systems & Modules

| Term | Definition | Aliases / Notes |
|---|---|---|
| **Project Proteus** | The new cloud-native, Azure-hosted manufacturing operations platform replacing GSCA Machinery and QSens. | "Proteus", "the platform" |
| **GSCA Machinery** | Legacy JTI manufacturing operations platform (machine data collection, KPI tracking, downtime classification) built on SAP MII; being replaced. | — |
| **QSens** | Quality Sensor Management module — sensor monitoring, adherence KPIs, sensor checks. Part of Proteus. | — |
| **SmartSENS** | Legacy combined package = GSCA Machinery + MDC + QSens (used interchangeably in some sources). | — |
| **SmartMDC** | Proteus module name for MDC ingestion & classification (edge event pipeline, mapping libraries). | Module M-03 |
| **SAP MII** | SAP Manufacturing Integration and Intelligence — the legacy stack GSCA Machinery runs on; end-of-support 2027; retained read-only ≤ 6 months post-cutover. | — |
| **SAP S/4HANA** | JTI's ERP; source of truth for plant hierarchy, production orders, confirmations, material master. | — |
| **SAP Integration Suite** | Integration layer exposing SAP entities via REST; live since April 2026. | — |
| **SAP BW** | SAP Business Warehouse; consumes daily aggregated KPI data from Proteus for global reporting. | — |
| **SAP EAM** | SAP Enterprise Asset Management; source of plant hierarchy / functional locations validated by admins. | — |
| **ServiceNow SDM** | Service Delivery Management; bidirectional integration (shift schedules out, planned activities in); authoritative for Lock Rate. | — |
| **Enterprise Data Lake** | JTI's downstream analytics platform (Databricks/ADF) owned by the CDLP team; Proteus delivers data to its boundary via Event Hubs. | "Data Lake", CDLP |
| **Databricks Zero Bus** | Alternative near-real-time streaming approach proposed for Data Lake delivery; under evaluation vs. Event Hubs (OQ-09). | — |
| **DDS Board** | IWS Power App / Power BI board for daily direction setting; **out of scope** — Proteus supplies source data only. | — |
| **Edge Server** | On-premise server (Siskon/Cisco-owned) at each factory collecting machine events and pushing JSON to Proteus; fixed integration boundary (~30 servers). | MDC server |
| **Azure Event Hubs** | Azure streaming service used for MDC ingestion and Data Lake export (near-real-time, from Day 1). | — |
| **Azure Container Apps** | Compute platform hosting Proteus services. | — |
| **Azure SQL Database** | Primary operational persistence layer. | — |
| **Azure Service Bus** | Async messaging for ServiceNow and other integrations. | — |
| **APIM** | Azure API Management; mandated API gateway for SAP and ServiceNow. | — |
| **Power BI** | Reporting tool; used for Data Lake analytics and (legacy) local factory workarounds. | — |

---

## 3. Plant & Organizational Hierarchy

| Term | Definition |
|---|---|
| **Company** | Top of the hierarchy — JTI. |
| **Country / Region** | Geographic grouping of factories (e.g., Ukraine, Romania, Philippines). |
| **Factory** | A physical production site (~27 global sites). Scope boundary for most user access. |
| **Area** | Functional area within a factory (e.g., Make & Pack, Filter Making, Logistics). |
| **Work Center** | A production line within an Area (e.g., MP6, MP12); **the primary unit of OEE calculation**. |
| **Equipment / Machine** | A physical machine with a serial number, belonging to a Work Center (Maker, Packer, Wrapper, etc.). |
| **Functional Location** | Virtual location in the SAP hierarchy (~250,000 rows); synced daily from SAP EAM. |
| **Construction Type / Group Assembly** | Sub-equipment grouping below Machine. |
| **Assembly / Unit** | Lowest-level component within an Equipment. |
| **Maker** | Machine producing the product (e.g., cigarette rods); feeds the Packer. |
| **Packer** | Machine packaging product output from the Maker. |
| **Wrapper / Cellofanner / Cartooner / Palletizer / Tax Stamper** | Downstream packaging-line machine types. |

---

## 4. Production & Operations

| Term | Definition | Notes |
|---|---|---|
| **Production Order (PO)** | SAP-sourced order to produce a specific material quantity on a work center. | Non-SAP factories create POs manually (config flag). |
| **PO Confirmation** | SAP confirmation of produced quantity against a PO; synced near-real-time. | Resolved via PO Confirmation Resolution form. |
| **Shift** | Defined production time period (typically 8 hours). | Month-end uses a special 3-segment schedule. |
| **Changeover** | Switching a line to a new product/PO; a **planned** activity. Selecting the new PO triggers QSens sensor target recalculation. | "Brand change" is a related setup parameter. |
| **Brand Change Time** | Configured expected changeover duration per work center; baseline for changeover-variance KPI. | Set by P-03a at setup. |
| **Run / Run Hours** | Time a machine is actively producing; basis for Performance in OEE. | — |
| **Test Production** | Non-production run (testing) flagged to be **excluded** from KPI calculation; human input overrides machine data here. | — |
| **Training Run** | Operator training interval excluded from KPI; human input overrides machine data. | — |
| **Technical Study** | Engineering study interval; human input overrides machine data. | — |
| **Actual Speed** | Operator-entered real machine throughput per shift per PO; requires a mandatory reason code. | Compared to Lock Rate. |
| **Reject** | Defective product quantity, entered by type/quantity/posting date, linked to a PO. | — |
| **Reclaimer (inline)** | Equipment that reprocesses reject material inline; its presence makes "production without reject" a valid (non-anomalous) state. | — |
| **Shift Remainder** | Manual volume booking entered by an operator during an active shift to record unconfirmed work-in-progress (leftover pallets) spanning the shift boundary; reconciles SAP confirmed volumes against shift-level OEE. Optional per factory. A remainder > 1 pallet is treated as suspicious. | Leftover pallets carried to another shift |
| **Pallet** | Unit of leftover/unconfirmed volume tracked by a Shift Remainder. | — |
| **WIP (Work In Progress)** | Production not yet confirmed by SAP; captured via Shift Remainder. | — |
| **Attainment to Plan** | Degree to which actual volume met the planned volume; reviewed in KPI reports. | — |
| **Primaries / Primary Processing** | Tobacco primary processing (leaf/blend stage) — different line structure; **deferred** scope (future). | SmartSENS for primaries is future. |
| **Parallel Flows** | One Maker feeding two or more Packers (e.g., Trier, Tenerife, OTP lines). | Production line variant. |
| **Parallel Production** | Multiple lines sharing one SAP PO (Philippines only). | Variant; measurement open (OQ-03). |
| **Cross-Production** | A Maker physically re-linked between lines for flexibility (~3 factories). | Variant; relinking handling open (OQ-02). |

---

## 5. Downtime & Stop Classification

| Term | Definition | Notes |
|---|---|---|
| **MDC Event** | A machine state change or measurement received from an edge server (stop, volume, quality parameter, reject). | Raw input; never modified by users. |
| **Stop** | A machine non-running interval detected from MDC data. | Split by shift boundary (BR-002). |
| **Downtime Event** | A Stop that has been classified with a downtime reason and location by an operator. | Stored as a human-input record, not by mutating machine data. |
| **Downtime Reason** | Coded classification of why a machine stopped (e.g., Breakdown, Changeover, Planned Maintenance). | Creation is release-gated; only P-05a activates/deactivates. |
| **Downtime Location** | Where on the equipment the stop occurred; global master data owned by P-05a. | View-only for local admins. |
| **Breakdown** | A stop > 10 minutes **with** a spare-part change. | BR-004. |
| **Process Failure** | A stop > 10 minutes **without** a spare-part change. | BR-004. |
| **Planned Downtime** | Scheduled activities (changeovers, maintenance, meetings); deducted from Available Time in OEE. | — |
| **Unplanned Downtime** | Unexpected stops > 10 minutes; losses against Run Hours. | — |
| **Long-Lasting Downtime** | A stop spanning multiple shifts; flagged so the system auto-ends it when the machine restarts. | — |
| **Non-Recorded Stops** | Unaccounted time in the OEE waterfall (ramp-up/down, micro-stops below MDC threshold); monitored separately at month level. | "Unaccounted time" |
| **Reclassify** | User-facing action to assign/correct a downtime reason. Implementation: writes a new human-input record (no UPDATE/DELETE on machine data). | Label only — not a data mutation. |
| **Bulk Classify** | Apply the same downtime reason to multiple stops of the same root cause in one action. | — |
| **Apply to All Equipments** | Cascade one classification across all machines in a work center for the same interval, using the largest overlapping stop from each machine. | BR-015. |
| **Override Runtime / Ignore MDC Stop / Min. Duration / Can Insert Into Future** | System-embedded reclassification rules that auto-filter available options; never exposed as user choices. | Spec constraint. |

---

## 6. KPIs & Performance Metrics

| Term | Definition |
|---|---|
| **OEE (Overall Equipment Effectiveness)** | Primary KPI = Availability × Performance × Quality. Waterfall: Calendar Hours → less Planned Downtime → Available Time → less Unplanned Downtime → Run Hours → adjusted for Rate Loss and Quality Loss. ~60% of all report runs. |
| **Availability** | Share of available time the machine was actually running (Run Hours ÷ Available Time). |
| **Performance** | Speed efficiency vs. target (rate) during running time. |
| **Quality** | Good output vs. total output (reject-adjusted). |
| **Rate Loss** | OEE loss from running below target speed. |
| **Quality Loss** | OEE loss from rejects/defects. |
| **Lock Rate** | Agreed target throughput speed per machine for a 90-day cycle, defined in ServiceNow SDM. Operators compare actual speed to it each shift; deviation requires a reason. |
| **Nominal Speed (Rate)** | Configured machine speed baseline; P-03a requests, P-05a sets after SAP EAM validation. |
| **Speed Loss** | Configured/entered loss against nominal/lock rate for specific production types. |
| **MTBF (Mean Time Between Failures)** | Average running time between machine failures; reliability KPI. |
| **Changeover Variance** | Actual changeover duration vs. target; too fast = steps possibly skipped, too slow = inefficiency. |
| **Reject Rate / Reject Ratio** | Rejects relative to production (e.g., maker reject ratio). |
| **IWS Lost 3** | IWS waterfall report (calendar time → working time → OEE breakdown); second-priority report. |
| **Adherence KPI** | QSens metric: sensor-in-target time during scheduled production. |
| **Non-Recorded Stops Metric** | Month-level measure of unaccounted time (see §5 Downtime & Stop Classification). |

---

## 7. Quality & QSens

| Term | Definition |
|---|---|
| **Sensor / QSens Parameter** | A quality sensor with defined target ranges, measured against production each shift. |
| **Sensor Tag / Tag Code** | The technical PCO code identifying a physical machine measurement point; mapped by P-02c. |
| **Business Parameter** | The business-side configuration of a sensor: target values, adherence thresholds, formula type; owned globally by P-05b, configured locally by P-03b. |
| **Sensor Check** | An operator/system action confirming sensor settings are within target at a point in time; a **write** operation in the QSens Cockpit. |
| **Off-Reason** | Classification recorded when a sensor parameter is out of target (machine condition, process issue, etc.). |
| **Adherence** | Proportion of scheduled production time a sensor stayed within target; basis of the Adherence KPI. |
| **Green / Yellow / Red Status** | QSens Cockpit card states: in-target / warning / out-of-target. |
| **Global Sensor Template** | Global definition of which sensors/parameters a machine type must have; owned by P-05b. |
| **Local Sensor Configuration** | Factory-level mapping of template parameters to local tag codes and labels; owned by P-03b. |
| **QSens Formula Type** | Enumerated calculation type for a parameter: True/False, Greater/Lower, Range-Based, Control Degree Calculated, Control Limit Calculated, Percentage Control Limit, Range Ruled Control Limit, Range Ruled SD Control Limit. |
| **Sensor Utilization** | Deprecated/excluded metric (A-15); QSens relies on adherence KPIs only. **Out of scope.** |

---

## 8. Data, Integration & Events

| Term | Definition |
|---|---|
| **MDC Reprocessing** | System cycle every 90 minutes that re-downloads the last 720 minutes of machine data and overlays it onto records, restoring data lost to network gaps. Critical before KPI computation (BR-014). |
| **Human-Input Record** | Separate record capturing a user's classification/entry; the reporting layer recalculates from machine data + MDC mapping + human input. Machine data is never updated/deleted by users. |
| **MDC Mapping** | Mapping of physical machine events (stops, rejects) to classification structures (breakdown/reject catalogs); global library (P-05a) → local library (P-02c). |
| **Soft Delete** | Logical deletion flag; hard delete is prohibited system-wide (BR-005). |
| **Import Issue Resolution** | Form/report to resolve SAP confirmations that failed to attach to a shift. |
| **PO Confirmation Resolution** | Form to reassign delayed/misallocated SAP confirmations to the correct shift; auto-clears matching Shift Remainders. |
| **Reconciliation** | Month-end matching of POs/volumes between Proteus and SAP; target zero discrepancy. |
| **Consumer Group** | Event Hubs mechanism allowing multiple downstream consumers (e.g., Data Lake) to read the same event stream independently. |
| **KU (Cigarette Equivalent Unit)** | Cross-product volume normalization unit; 1 KU = 1,000 cigarettes; OTP converted via SAP factors. |
| **Big-Bang Go-Live** | Rollout strategy: all factories cut over simultaneously (vs. phased). |
| **Cutover** | The switch from GSCA Machinery to Proteus; SAP MII retained read-only ≤ 6 months after. |
| **Shadow Validation / Dual-Run** | Running Proteus against live SAP MII data pre-go-live to compare outputs. |
| **Legacy Parity Reconciliation** | Verifying Proteus reproduces legacy outputs (KPIs/reports) within tolerance. |
| **Hypercare** | Intensive post-go-live support period. |

---

## 9. Time, Periods & Editing Windows

| Term | Definition |
|---|---|
| **Edit Window (Shift)** | 30-minute window after shift end during which shift data is editable; then locked (BR-007). Enforced at the API layer. |
| **Shift Handover Window** | Configurable window during which a role may edit previous-shift data (default 30 min; wider for P-02a). |
| **KPI Period** | A calendar month over which KPIs are calculated; "open" for entry/correction, "locked" ~5th business day of the following month. |
| **Period Lock** | Automated freeze of a KPI period (~5th business day); data exported to SAP BW afterward (BR-020). |
| **Period Unlock** | Exceptional reopening of a locked period; requires Global Admin (P-05a) approval. |
| **Month-End 3-Segment Schedule** | Special schedule structure spanning pre-midnight / post-midnight / next day at month boundary. |
| **Reject Entry Window** | Current month + 5 business days of next month (BR-012). |

---

## 10. Roles, Access & Security

> **Canonical persona definitions:** [`personas.md`](personas.md). **Role × feature/permission mapping:** `specs/_rules/PERMISSIONS_MATRIX.md`. This section defines only shared access vocabulary.

| Term | Definition |
|---|---|
| **RBAC** | Role-Based Access Control; access granted by business role + factory scope. |
| **Two-Layer Provisioning** | Entra ID manages accounts and group membership; Proteus manages what each group can do. Never conflate the two layers. |
| **AD Security Group** | An Entra ID group mapping one-to-one to a Proteus business role type scoped to a factory or global. |
| **Birthright Group** | AD group auto-provisioned from employee attributes (country, department, org). |
| **Request-Based Group** | AD group provisioned via the IT portal (MyIT). |
| **Generic Machine Account** | Shared login tied to a production line (e.g., "Line 7"), not an individual; fixed permissions (SDE + QSens for that line); flagged non-employee. |
| **Plant-Level Data Isolation** | Users see only their assigned factory's data unless granted global scope. |
| **Global Viewer** | Read-only role across all factories; auto-provisioned via a single AD group (no per-factory provisioning). |
| **Functional Element** | A system function/page/capability a business role type can access; mapped by System Admin (P-06) only, release-gated. |
| **Factory SAP/Non-SAP Config Flag** | Factory-level flag enabling extra capabilities (PO creation/confirmation) — never a separate role. |
| **Audit Log / Change History** | Immutable record of who changed what and when (security and data-change events). A platform requirement; concrete regulatory track-and-trace (e.g., EU TPD) is **out of scope for now** — only change traceability is in scope. |

---

## 11. Screens, Reports & Forms

| Term | Definition |
|---|---|
| **SDE (Smart Downtime Entry)** | Shop-floor kiosk interface: multi-machine timeline, stop reclassification, bulk classify, apply-to-all-equipments, speed/reject/Shift Remainder entry, statistics tab. |
| **Information Header** | PO/context header shown in SDE for the current line. |
| **QSens Cockpit** | Bidirectional quality screen: sensor cards per work center, parameter drill-down, sensor-check confirmation. |
| **Office Screen** | Near-real-time, shift-level **production** KPI overview across workcenters/machines for factory management; color-coded work center states; per-machine drill-down popup (reject ratio, top stops, volume attainment, MTBF). Feature F-01.6 (M-01). **A distinct feature from the Aggregated Cockpit — never use the two interchangeably.** |
| **Aggregated Cockpit** | Factory-level **quality/sensor** KPI aggregation (QSens) split by product group; per-workcenter sensor KPIs with drill-down; shows machine connection status and clock-sync issues. Feature F-02.2 (M-02). **A distinct feature from the Office Screen — never use the two interchangeably.** |
| **Downtime Issue List** | Cross-line scan for data-quality anomalies with direct links to SDE at the relevant shift. |
| **Data Review Forms** | Set of office-user review/correction forms: Import Issue Resolution, PO Confirmation Resolution, Actual Speed review, Shift Remainder review, Reject Entry review. |
| **OEE Report** | Highest-priority, most-used report; every factory depends on it. |
| **Factory Map / Homepage** | Navigation portal showing user-accessible factories (low visual priority). |
| **Office Screen Color States** | Green / red / blue / gold / grey / orange work center states — a hard functional requirement, not cosmetic. |

---

## 12. Methodology, Delivery & AI Workflow

| Term | Definition |
|---|---|
| **IWS (Integrated Work Systems)** | JTI's operational excellence methodology; drives data quality, completeness, and timeliness requirements; defines roles (cell lead, process lead, operator). |
| **DDS (Daily Direction Setting)** | IWS daily meeting (~8:45 AM) where verified KPI data drives decisions. |
| **OPI (Operational Performance Improvement)** | JTI global team owning global standards; governs change requests; OPI-DPA (performance) and OPI-GC (quality). |
| **PRD** | Product Requirements Document — business-context anchor for the spec-driven flow. |
| **Feature Requirement Document** | Feature-level requirements derived from PRD + personas + glossary via AI (`/grill-with-docs`, `/to-prd`); reviewed with PO. |
| **GitHub Spec Kit** | Tooling that generates `specify.md` from a finalized ticket and breaks features into tasks. |
| **specify.md** | GitHub Spec Kit specification file; compared back to the ticket and feature doc for gaps before code generation. |
| **/grill-with-docs, /to-prd, /to-issues** | Copilot skills used in the BA workflow to elicit requirements, produce the requirements doc, and generate implementation tickets. |
| **Work Item (WI)** | Implementation ticket; "Ready for Spec" status triggers `specify.md` generation. |
| **Hybrid Delivery Team** | BTS GSC (ownership), GDC (architecture), Siskon (edge/MDC infra), Intellias (platform dev). |
| **Discovery Phase** | 11 May – 12 June 2026 elicitation/architecture phase; complete. |
| **RAID Log** | Risks, Assumptions, Issues, Dependencies register. |

---

## Open Questions (Glossary)

| OQ | Question | Owner |
|---|---|---|
| G-01 | Confirm acronym expansions marked `[confirm]`: BTS GSC, GDC, CDLP, OPI-DPA, OPI-GC, PCO. | BA / Florin / Sergey |
| G-02 | Confirm "SmartSENS" capitalization and whether it remains a live term or legacy-only. | Andronic |
| G-03 | ~~Confirm Office Screen vs Aggregated Cockpit~~ — **Resolved:** distinct features (Office Screen = production overview F-01.6; Aggregated Cockpit = QSens quality aggregation F-02.2); never synonyms. | Closed |
| G-04 | Should a domain SME formally own and sign off this glossary going forward? | BA / SME |
