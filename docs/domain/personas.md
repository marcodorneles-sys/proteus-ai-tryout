# User Personas: Project Proteus (GSCA Machinery & QSens)

* Status: Discovery phase. 
* Source: Stakeholder interviews (05/2026), User Role Analysis, Solution Spec.
* Updated: 28.05.2026.
* Use: SDD via GitHub Spec-Kit / Copilot. Trace all AC/requirements to Persona ID.

## Governance
* Version: 1.0 (28.05.2026)
* Author: Julia Shcherbyna
* Reviewers: Sergey Morozov, Nicolae Andronic Rosu

## Usage
Define user, task, reason, constraints. Use Persona ID (e.g., P-01) in Spec-Kit files.

---

## Human Personas

### P-01: Shop-floor Operator (Equipment Owner)
* Role code: `biz.SFOperator`
* Alias: Line crew, Equipment owner, Machine operator, Technician, Electrician, Mechanic, Automation
* Scope: Factory / production line
* Access: Single plant. Single/paired line.
* Desc: Largest user group. Blue-collar. Stationed at line. Produce product. Use shared kiosk terminal. Login via shared shop-floor credential.
* Goals: Record/reclassify MDC downtime fast. Enter rejects. Check QSens Cockpit. Fast UI in/out. Maintain production flow.
* Tasks: Enter/reclassify SDE stops (allow bulk). Enter actual machine speed (require reason code). Submit rejects. Check sensors. View performance/context.
* Permissions: 
    * Write: Current shift downtime (prev shift ONLY in handover window). 
    * Delete MDC stops: Denied. 
    * Reclassify MDC stops: Allowed.
* Context: Factory floor kiosk. Noisy. High frequency.
* Pain (Legacy): Long dropdowns. Manual timestamps. No timeline. Slow. No self-correction.
* Spec Hints: UI requires kiosk wizard/step-by-step. Large tap targets. Manual timestamps strictly forbidden. Shift handover lock (30 min) is hard constraint. Personalization denied (shared login). Speed entry requires reason code.

### P-02: Process Lead (Shift Supervisor)
* Role code: `biz.ShiftLeader`
* Alias: Shift leader, Group leader, Team lead, Cell lead, SAP operator
* Scope: Factory / cell / production area
* Access: Single plant. Multiple lines.
* Desc: Senior shop-floor/office role. Verify, correct, own production performance data.
* Goals: Ensure data accuracy/completeness for KPIs/reports.
* Tasks: Verify daily stops/speed entries. Review SAP errors. Review rejects. Review OEE/KPI reports. Enter missing downtimes.
* Permissions: Write: Current/prev shift. Read: OEE reports. Limit: Assigned plant.
* Context: Shift end/start. Office desktop. Time-pressured.
* Pain (Legacy): No audit trail. Manual SAP reconciliation. Delayed SAP data. No multi-line view.
* Spec Hints: Seamless SDE to report transition. Support bulk operations. Support varying shift schedules. UX target: Desktop.

### P-03: Production Analyst
* Role code: `biz.ProdOffice` / GSCA Global Admin
* Scope: Factory (local) or GSCA Global (cross-factory)
* Access: Single plant or multiple plants
* Desc: Technical-analytical office role. Own KPI data integrity. Own month-end KPI lock/SAP reconciliation.
* Tasks: Month-end period lock. KPI corrections. SAP variance reconciliation. Resolve unclassified stops. Configure shift schedules.
* Permissions: Read: All plant reports. Write: Downtime/rejects (open KPI period). Write: Shift schedule. Config: Denied.
* Pain (Legacy): Complex "unrecorded time" calc. No auto SAP reconciliation. No favorites.

### P-04: Local Configuration Administrator
* Role code: `biz.ConfigAdmin`
* Scope: Factory
* Access: Single plant
* Desc: Local system owner. Configure platform to match factory setup.
* Tasks: Maintain plant/workcenter/machine hierarchies. Manage shift schedules. Maintain downtime catalogs. Configure MDC breakdown/reject mapping. Manage users/translations.
* Permissions: Config: Full (assigned plant). App roles: Grant/revoke.

### P-05: Local Data Administrator
* Role code: `biz.DataAdmin`
* Scope: Factory
* Access: Single plant
* Tasks: Modify downtime/rejects (open KPI period). Enter KPI corrections. Review audit log.
* Permissions: Write: Downtime/rejects (open KPI period). Read: Audit logs/reports. Config: Denied.

### P-06: Non-Production Office User
* Role code: `biz.NonProdOffice`
* Alias: Finance user, Quality observer, Other dept
* Scope: Factory
* Access: Single plant. Configured report subset.
* Desc: Non-production dept. Read-only configured reports.
* Spec Hints: Enforce report access at API level (UI hidden).

### P-07: Production Manager / Director
* Role code: `biz.ProdMan`
* Scope: Factory
* Access: Single plant. Read: Full.
* Desc: Senior factory management. Monitor Office Screen/KPI reports. Data entry/correction: Denied.

### P-09: GSCA Global Administrator
* Role code: GSCA Global Administration / GSCA Power User
* Scope: Cross-factory / global
* Access: All plants
* Desc: Global business owner. Cross-factory data quality. Master data stewardship. Global config.
* Permissions: Read/Write: All plants. Write: Global config (templates, global catalogs).

### P-12: Read-Only User
* Role code: `biz.ReadOnly`
* Scope: Factory
* Access: Configured per user
* Desc: Data visibility only. Modify: Denied (e.g., auditors, visiting managers).
* Spec Hints: Hide/disable all write controls.

---

## System Actors
* **SA-03 ServiceNow SDM:** Outbound: Shift schedules. Inbound: Planned activities.
* **SA-04 Data Lake:** Consume production facts (15-30m) & master data (2-24h) via `Azure Event Hubs`.
* **SA-05 SAP BW / GSCA KPI:** Pull daily consolidated KPI data.
* **SA-06 Romania Waste Management:** Call GSCA REST API for waste tracking.
* **SA-07 Azure Entra ID:** AuthN (SSO). Enforce MFA for admins.
* **SA-08 Notification Service:** Dispatch 10+ scheduled email reports.

---

## Open Questions
| ID | Question | Impact |
| :--- | :--- | :--- |
| **OQ-01** | Shared kiosk login global vs local? RBAC/audit implications? | Affects NFR (AuthN/Audit), P-01 spec gen. |

---

## Known Gaps
| Role Code | Issue |
| :--- | :--- |
| `biz.MechanicMaintenanceManager` | No persona. Potential IWS Maintenance Lead. |
| `biz.NotificationCreation*` | No persona. Scoped notification variants. |
| `biz.PrimaryProcessing` | No persona. |
| `biz.QSensCockpit` | No persona. Likely view-only QSens (distinct from P-08). |
| `biz.ShiftLeader-Planning` | No persona. Planning-scoped variant of `biz.ShiftLeader` (P-02). |