# User Personas — Project Proteus (GSCA Machinery & QSens)

**Version:** 1.4 (24 June 2026) | **Source:** User_Personas_v1_4.docx  
**Purpose:** AI context for feature spec generation (/grill-with-docs, /to-prd, GitHub Spec Kit). Every functional requirement and acceptance criterion must be traceable to at least one persona listed here. Refer to personas by ID (e.g., `P-01`) in specs and ACs.

---

## Cross-Cutting Rules (Apply to All Features)

These rules override anything in individual persona sections and must be enforced universally:

1. **Human-input record model.** Machine data is **never** modified by any user action. When a user "reclassifies" a stop, a new human input record is written to a separate table. The output/reporting layer always recalculates from three sources: machine data + MDC mapping + human input. No `UPDATE` or `DELETE` on machine-originated records. Only the MDC sync can update machine data.

2. **Shift handover window.** Write access to previous-shift data expires after a configurable window (default: 30 minutes after shift end). This is a hard constraint; enforce at the API layer, not only in the UI.

3. **Downtime reason creation is release-gated.** No user role may create a new downtime reason at runtime. New reasons arrive only via a platform release with QA validation. Activation/deactivation of existing reasons is a P-05a (Global Admin) action only.

4. **New permission groups are release-gated.** The set of system functions each business role type can access (functional elements) is defined by P-06 (System Admin) only. Not configurable by any business role at runtime.

5. **Factory SAP/non-SAP config flag.** Extra capabilities (PO confirmation, production order creation) are gated by a factory-level configuration flag — never by a separately provisioned role.

6. **No report-level security.** GSCA Machinery does not operate a report-access security model. Any authorized platform user can read any report. Enforce only at the role/factory boundary.

7. **Two-layer user provisioning.** Azure Entra ID manages accounts and group membership. Proteus manages what each group can do inside the platform. Never conflate the two layers in specs.

8. **Speed entry mandatory reason.** No user may save a speed entry without a reason code. This is a hard validation — no exceptions.

9. **QSens sensor check is a write operation.** The QSens Cockpit is bidirectional. Never spec the check-confirmation flow as read-only.

---

## Persona Groups

### Group 1 — Operators

> Largest user group. Blue-collar workers on or near the production line. The system is a side tool, not their core job.

---

#### P-01 — Shop-floor Operator (Equipment Owner)

| Attribute | Value |
|---|---|
| Role code | `biz.SFOperator` |
| Also known as | Line crew, Equipment owner, Machine operator, Technician, Electrician, Mechanic, Automation |
| Scope | Single plant, single or paired production line |
| Non-SAP variant | Includes PO confirmation capability (factory config flag — no separate role) |

**Physical context.** Kiosk terminal (17" screen on a pole or under a conveyor belt), 5–30 m from the machine. Generic shared AD account per line area (factory-line-area naming convention, flagged as non-employee). Default plant, workcenter, and machine must be pre-configured so the operator opens directly in their line context — no selection prompts on login. User may have greasy hands, be interrupted, or be standing in a noisy environment.

**Key capabilities.**
- Enter or reclassify downtime stops in Smart Downtime Entry (SDE). Bulk reclassification for multiple stops of the same type.
- Enter actual machine speed per shift with a mandatory reason code (lock rate / deviation).
- Submit reject quantities linked to production orders.
- Perform sensor checks in QSens Cockpit; record off-reason explanations for out-of-range parameters.
- View machine performance (Office Screen) and current PO context (Information Header in SDE).
- Perform changeovers.
- Non-SAP factories only: add, edit, and delete PO confirmations (factory config flag).

**Write scope.** Current shift only. Previous shift editable within the shift handover window only.

**Read scope.** All operational reports and dashboards (same as P-04 Data Consumer). No write access outside the shift window.

**Spec constraints for AI.**
- All SDE UI flows: wizard/step-by-step, one decision at a time, large tap targets, minimal cognitive load. Not form-based. Treat SDE as a kiosk/industrial environment, not a standard office web app.
- Never generate specs requiring manual timestamp entry — derive from MDC data or provide pre-filled suggestions (exception: no MDC data available).
- Shift handover locking is a hard constraint at the API layer.
- Reclassification business rules (Override Runtime, Ignore MDC Stop, Min. duration, Can insert into Future, Long lasting) are system-embedded — never expose as user-facing choices; they filter available options automatically.
- Speed entry: mandatory reason code; never generate specs that allow saving speed data without a reason.
- Self-correction for speed and reject entries is a **planned improvement, not in initial release** — do not assume this capability.
- PO confirmation for non-SAP factories: gated by factory config flag, not a separate role.
- "Reclassify" is a user-facing label only. Implementation: human input record → separate table; output recalculated. No `UPDATE`/`DELETE` on machine data. Ever.

---

### Group 2 — Support Functions

> Roles close to the shop floor that verify data correctness, resolve discrepancies, and own machine expertise. In small factories one person may cover all three sub-roles.

---

#### P-02a — Process Lead / Shift Supervisor (Data Verification)

| Attribute | Value |
|---|---|
| Role code | `biz.ShiftLeader` (closest mapping; IWS-specific title varies by factory) |
| Also known as | Shift leader, Group leader, Team lead, Cell lead, SAP operator, IWS Cell Lead |
| Scope | Single plant; multiple production lines within their area |
| Non-SAP variant | May include production order creation (factory config flag — no separate role) |

**Description.** Senior shop-floor or office role responsible for verifying, correcting, and owning production performance data for their shift/area. Primary accountability: data accuracy and completeness of all entities used in KPI calculations. Timeliness is critical — late corrections cause significant KPI swings visible to downstream consumers.

**Daily data verification sequence (order matters).**
1. Review downtime entries in SDE: MDC data gaps, unclassified stops >10 min, production sequence correctness.
2. Check actual speed entries: all machines must have speed recorded; flag entries not at lock rate without valid reason.
3. Correct weekend/outage anomalies (e.g., unregistered volumes from MII unavailability) — primary responsibility on Monday morning before DDS.
4. Review Import Issue Resolution report: no unprocessed SAP confirmation errors.
5. Review PO Confirmation Resolution report: no confirmations mismatched to wrong shift or PO.
6. Review Shift Remainders: investigate any remainder exceeding one pallet.
7. Review Reject Entry: check for production periods without reject entry, and entries without associated production.
8. Review OEE / KPI reports: confirm MTBF, unplanned stop count, attainment to plan, maker reject ratio, process failures, breakdown count, changeover time variance.

**Write scope.** Current and previous shifts (broader window than P-01). Cannot modify master data or closed KPI periods. Limited to assigned plant.

**Spec constraints for AI.**
- SDE (corrections) and operational reports (review) must be connected — seamless transition between them, not separate isolated flows.
- PO Confirmation Resolution and Import Issue Resolution are **daily routine tasks** — design for efficient zero-result confirmation (green = done), not for exception-only usage.
- Bulk operations (bulk reclassification, bulk resolution) are required — single-record workflows are insufficient.
- Shift handover window for this role is wider than for P-01; exact duration is factory-configurable.
- Do not assume a single shift schedule model; month-end has a special 3-segment structure (pre-midnight / post-midnight / next day).
- Data health dashboard (unified landing point) is a confirmed planned improvement — leave a clear integration point.
- UI/UX: office environment, desktop. Multi-line views. Do not apply shop-floor kiosk patterns.
- Tablet viewport (~1024px) must be considered for Production Manager / Cell Lead use.
- Non-SAP production order creation: factory config flag only, not a separate role.
- Daily data locking (human-verified shift status) is a **planned future enhancement** — design verification workflows to leave a clear integration point, but do not include in initial release.

---

#### P-02b — Production Analyst (KPI & Period Management)

| Attribute | Value |
|---|---|
| Role code | `biz.ProdOffice` / GSCA Global Admin role (varies by factory) |
| Also known as | Production Process Manager, Process Analysis & Improvement Manager, Automatic Systems Engineer |
| Scope | Single plant (local) or multiple plants (global variant) |

**Description.** Technical-analytical office role owning month-end KPI locking, SAP reconciliation, PO confirmation resolution for complex cases, and factory-level reporting. In some factories also covers P-02a daily verification tasks — the boundary is factory-specific and fluid.

**Key capabilities.**
- Daily: review PO confirmation resolution; resolve delayed/misaligned SAP confirmations.
- Daily: check non-recorded (unaccounted) time metrics across all lines.
- Monthly: verify all POs reconciled between GSCA Machinery and SAP (zero discrepancy target).
- Monthly: lock the KPI period (auto-triggered day 5–7; analyst confirms data is clean before confirming).
- Monthly: review reject entries per line for completeness and accuracy.
- Review OEE, availability, MTBF, downtime analysis, reject ratio reports.
- Manage and validate shift schedules (including month-end 3-segment schedule).
- Escalate data quality issues to Process Leads or SAP teams.

**Write scope.** Downtime and reject records within the open KPI period. PO confirmation resolution. KPI corrections (within period). Shift schedule. Read access to system audit log (who changed what, when). Cannot modify system configuration or master data structure.

**Spec constraints for AI.**
- Favourites / bookmarked reports feature is explicitly valued — include in the new system.
- Month-end KPI locking is time-critical and high-stakes: specs must include explicit confirmation steps, period status indicators, and guards preventing data modification after lock.
- Non-recorded time indicator must be visible in the daily OEE report without a separate query.
- PO confirmation resolution: support both bulk (system-suggested allocation) and manual per-record resolution.
- Cross-shift and cross-period data views are essential; single-shift-only views are insufficient.
- No data release gate or verified-status that blocks other users from reading data — KPI period locking is the only data-freeze mechanism.
- Do not generate specs that replicate local-factory Excel report patterns (explicitly out of scope).
- Do not expose raw SQL query endpoints or direct DB connections (not in initial release).

---

#### P-02c — Production Machine Expert

| Attribute | Value |
|---|---|
| Role code | TBD (OQ-01 — not yet defined in role workbook) |
| Also known as | Machine data expert, Automation engineer, MDC mapping specialist |
| Scope | Single plant — deep technical machine knowledge |

**Description.** Technically specialized role owning the mapping between physical machine events (stops, rejects) and the system's classification structures (breakdown catalogs, reject catalogs), and the configuration of machine sensor parameters in QSens. Cannot be substituted by a process lead or analyst — requires deep machine engineering knowledge. In some factories combined with P-03b (QSens Local Admin); system must support both patterns without separate accounts.

**Key capabilities.**
- Define and maintain MDC breakdown-to-reason mappings (global library → local library).
- Define and maintain reject cause mappings for machine-level events.
- Identify correct PCO tag codes for QSens sensor parameters.
- Validate that configured sensor parameters reflect actual machine measurement points.
- Advise P-03b on sensor configuration questions requiring machine engineering knowledge.
- Support factory go-live: verify all MDC mappings are complete and correct before activation.

**Write scope.** MDC breakdown and reject mapping forms. QSens sensor tag code configuration (technical parameter mapping, not business threshold configuration). No access to shift schedule, user management, or financial configuration.

**Spec constraints for AI.**
- Technical counterpart to P-03b: machine expert owns *what* to measure (tag codes, physical mapping); P-03b owns *how* to evaluate it (business thresholds, adherence rules).
- MDC mapping forms must be accessible without P-03a (Local Config Admin) privileges — mapping is a specialist task, not a general admin task.
- Mapping forms must support two levels: global library (P-05a only) and local library (this persona). Local level cannot modify global definitions.
- When combined with P-03b in a single factory user, both permission sets apply without two separate accounts.
- Office Screen secondary use case: MDC fit testing (verifying newly connected machine's data is flowing). Refresh cycle must be fast enough to support this.

---

### Group 3 — Local Factory Admin

> Power users who own the system at factory level. Two focal points: performance side (P-03a) and quality side (P-03b). May be the same person in smaller factories. RBAC must support both patterns.

---

#### P-03a — Local Configuration Administrator (Performance Side)

| Attribute | Value |
|---|---|
| Role code | `biz.ConfigAdmin` |
| Also known as | Power user, Local GSCA Power User |
| Scope | Single plant — full configuration authority for that plant |

**Description.** Local system owner for the performance/production side of GSCA Machinery. Keeps system configuration aligned with actual factory operations. Acts as first-line support and liaison to GSCA Global for configuration questions. Responsible for all permissions they create; needs a role-assignment overview screen to fulfil this accountability.

**Key capabilities.**
- Ensure plant/workcenter/machine hierarchy data matches SAP EAM (confirm before BTS deploys to production; manual creation available but restricted).
- Manage shift schedule templates including month-end special schedules.
- Set factory-level default schedule (baseline for all users; users may override for themselves). Set default plant, workcenter, and machine per user for kiosk pre-configuration.
- Maintain local translation strings (local language for system labels and object names).
- Configure workcenter groups for reporting filters.
- Set lock rates / speed loss entries for specific production types.
- Set brand change time for workcenters (initial setup only; may be locked thereafter).
- Configure notification recipient lists for automated mail reports.
- Review and correct data beyond the period allowed to P-02a.
- Manage application-level roles for users in their plant (request/approval via IT portal; account creation itself is IT-managed via AD).
- Language and regional settings (decimal separators, date format) — auto-detect from browser; local admin can set explicit override per user.

**Access scope.** Full configuration for assigned plant. Cannot modify global templates (read-only). Cannot modify data for closed KPI periods. Cannot access other factories' configuration or data.

**Spec constraints for AI.**
- ~90 admin forms are largely templated CRUD — generate consistent list + detail form patterns. Do not generate unique layouts per form unless functional requirements explicitly differ.
- Translation management must allow editing local-language values without switching the UI language.
- User provisioning specs: two-layer model (Azure Entra ID account managed externally + application role managed here). Do not conflate.
- Shift schedule forms must support the month-end 3-segment pattern as a first-class configuration option.
- SAP master data is synced into GSCA Machinery DB but not yet surfaced in admin UI — design for SAP-driven pre-population with power user confirmation, not purely manual creation.
- Dynamic workcenter management (add/remove without redeployment) is a planned enhancement — do not assume a static list.
- Downtime location structure: **view-only** for this persona (maintained by P-05a as global master data).
- Nominal speed (rate): P-03a requests; P-05a sets (after SAP EAM validation).
- Downtime reason activation/deactivation: **global-only** (P-05a) — not a local function.
- This persona must have a role-assignment overview screen: who holds which role in their plant.

---

#### P-03b — QSens Local Admin (Quality Side)

| Attribute | Value |
|---|---|
| Role code | `biz.QSensConfigAdmin` |
| Also known as | Local quality specialist, QSens configuration owner |
| Scope | Single plant, QSens module specifically |

**Description.** Quality department representative responsible for configuring QSens sensor definitions at factory level using globally defined templates as a baseline, monitoring sensor adherence, and maintaining local quality parameter settings. Distinct from P-01 (who performs sensor checks) — this persona defines the rules those checks are validated against. In some factories combined with P-03a or P-02c; system must support all patterns.

**Boundary with P-02c.** Machine expert identifies correct technical tag codes and physical mapping. QSens Local Admin configures business parameters (thresholds, adherence rules, formula types) against those mapped sensors.

**Key capabilities.**
- Map global sensor template parameters to local machine parameter tag codes.
- Configure business parameters: target values, adherence thresholds, formula type per sensor.
- Set local-language labels for sensor names in QSens Cockpit.
- Review sensor utilization and adherence KPI reports.
- Coordinate with P-02c on correct tag code identification.

**Access scope.** Write access to QSens sensor configuration for their plant. Read access to all QSens reports. No access to SDE configuration. Cannot modify global QSens templates (read-only — owned by P-05b).

**Spec constraints for AI.**
- QSens formula types are an enumerated business list: True/False, Greater/Lower, Range-Based, Control Degree Calculated, Control Limit Calculated, Percentage Control Limit, Range Ruled Control Limit, Range Ruled SD Control Limit — represent as constrained selection, not free-text.
- QSens Cockpit is bidirectional (read + write) — sensor check confirmation is a write operation; never generate this screen as read-only.
- Global template to local configuration sync is a known pain point and planned enhancement — leave a clear integration point in configuration form specs.
- When combined with P-02c, machine-side tag code configuration and quality-side business parameter configuration must be distinguishable in the UI.

---

### Group 4 — Data Consumers (Read-Only)

---

#### P-04 — Data Consumer

| Attribute | Value |
|---|---|
| Role codes | `biz.GlobalViewer` (global) |
| Also known as | ReadOnly, Global viewer, Finance user, Quality observer, Production Manager, Production Director, Non-production office user, Make & Pack Manager, Cell structure lead, Global Manufacturing / Regional Manufacturing Directors |
| Scope (local) | Single plant |
| Scope (global) | All plants (auto-provisioned via a single AD group) |

**Description.** Everyone who consumes system data without modifying it. Two flavors: local (scoped to one plant) and global (all plants, auto-provisioned so new factories require no per-factory provisioning). Global Viewer resolves a confirmed gap in the legacy system where global directors held Global Admin access as a workaround.

**Key capabilities.**
- Run operational and KPI reports.
- Monitor the Office Screen (live shift KPI overview).
- View downtime statistics and OEE trends.
- Access DDS-relevant KPIs for daily direction-setting meetings.

**Access scope.** Read-only to all reports and dashboards. No write access. No configuration access.

**Spec constraints for AI.**
- All write controls must be hidden (not just disabled) for this role.
- Color coding on the Office Screen (green / red / blue / gold / grey / orange workcenter states) is a **hard functional requirement**, not cosmetic.
- Office Screen must support drill-down to a per-machine KPI popup showing at minimum: reject ratio, top stops, volume attainment, and MTBF — without navigating away from the main screen.
- No report-level permission configuration — do not generate report-access security features.
- Global Viewer provisioning: single AD security group — do not generate specs requiring per-factory provisioning for global viewers.
- API endpoints must enforce read-only at the authorization layer, not only in the UI.
- Production Manager / Director use case: Office Screen continuously during working hours (often on a dedicated large display); KPI reports daily and monthly.

---

### Group 5 — Global Admin

> Per June 19, 2026 review: split into two distinct security groups. One person or team may hold both, but they are modelled and secured separately.

---

#### P-05a — GSCA Global Administrator (OPI-DPA)

| Attribute | Value |
|---|---|
| Role code | `biz.GlobConfigAdmin` |
| Also known as | OPI team member, Operational Performance team, Global GSCA owner; OPI-DPA (Operational Performance Improvement — Digital Performance Analysis) |
| Scope | Cross-factory / all plants |

**Description.** Global business owner of the performance and machinery side of GSCA Machinery. Two responsibility clusters: (1) global performance/KPI governance — cross-factory data quality, global KPI consolidation and SAP BW distribution, global master-data stewardship (downtime reason catalog, breakdown/reject library, downtime location structure), and demand control over factory change requests; (2) global machine administration — single global owner of machine-template assignment and MDC mapping standards. Quality-side global governance belongs to P-05b.

**Key capabilities.**
- Review and validate SAP EAM (plant hierarchy) data downloaded to GSCA Machinery.
- Maintain global library of downtime reasons and breakdown/reject classifications.
- Activate or deactivate existing global downtime reasons (creation is release-gated — no user-accessible "create new reason" action).
- Own machine-template assignment and MDC mapping standards as the single global authority; P-02c applies these standards locally.
- Maintain downtime location structure as global master data.
- Validate GSCA KPI consolidation and distribution to SAP BW.
- Govern user demand: filter and prioritize change requests from factories before forwarding to IT.
- Set nominal speed (rate) per P-03a request, validated against SAP EAM.

**Access scope.** Read and write across all plants. Write to global configuration objects (templates, global catalogs, global library). Daily GSCA KPI monitoring and cross-factory data quality checks. Monthly KPI consolidation and SAP BW distribution.

**Spec constraints for AI.**
- Requires cross-factory views — single-plant views are insufficient; all multi-plant aggregation features are primarily for this persona.
- GSCA KPI consolidation trigger must include explicit confirmation steps and status feedback.
- Global template management must be clearly separated from local configuration — enforce read-only on global objects for all non-global personas.
- Sole owner of global breakdown/reject classification library — local mapping forms must clearly show global definitions are read-only at local level.
- This persona is not a frequent SDE/shop-floor user — do not default to shop-floor UX patterns for their workflows.
- P-05a (performance) and P-05b (quality) are distinct security groups — never collapse them into one, even if one user holds both.
- Activate/deactivate downtime reasons is P-05a-only — never expose a "create new downtime reason" action in any runtime UI.
- Machine-template / MDC mapping assignment has exactly one global owner (P-05a) — the UI must not let P-05b or P-02c assign templates; only apply or validate them locally.

---

#### P-05b — Global QSens Administrator (OPI-GC)

| Attribute | Value |
|---|---|
| Role code | `biz.QSensGlobalAdmin` |
| Also known as | Global QSens admin, Global Quality admin, QSens global owner; OPI-GC (Operational Performance Improvement — Global Quality) |
| Scope | Cross-factory / all plants (QSens / quality) |

**Description.** Global owner of QSens quality standards across all factories. Defines the global model that factories validate against: which sensors a machine type must have, business parameters, thresholds, adherence rules, and formulas. Creates the global QSens structure inside machine templates; P-03b then validates locally (confirming sensors/tags exist); P-02c provides machine-side tag identification. Owns cross-factory benchmarking so thresholds stay consistent across sites.

**Key capabilities.**
- Create and maintain global QSens sensor and business-parameter definitions inside machine templates.
- Set thresholds, adherence rules, and formulas for business parameters (draws on R&D, product-spec, quality-manual inputs).
- Define which sensors and parameters a given machine type must have.
- Govern cross-factory benchmarking for threshold consistency.
- Review and resolve escalations from P-03b / P-02c for parameters not covered by the current standard.

**Access scope.** Read and write to global QSens sensors, business parameters, thresholds, and formulas across all plants. Read access to QSens reports and sensor adherence across all plants. Cannot assign machine templates or MDC mapping (owned by P-05a). Cannot create or activate downtime reasons.

**Spec constraints for AI.**
- P-03b validates locally and cannot modify global definitions — enforce read-only on global QSens objects for all non-global personas.
- Once a business parameter is linked to a global sensor, the local tag-to-sensor link should follow automatically — local roles confirm existence rather than re-create the link; do not require local re-entry of links already implied by the standard.
- Quality thresholds and formulas may draw on SAP product-spec and quality-manual data — model these inputs explicitly; do not assume P-02c holds them.
- Keep P-05b (quality) distinct from P-05a (performance): separate security group, separate screens (sensor/business-parameter standards vs. KPI and admin forms).

---

### Group 6 — System Admin (IT)

---

#### P-06 — System Administrator

| Attribute | Value |
|---|---|
| Role code | TBD (IT-managed; not a business role) |
| Also known as | IT admin, BTS IT, Technical admin |
| Known users | Florin and the IT/DevOps team |
| Scope | Global / platform level |

**Description.** IT-level technical administration for exceptional, low-frequency tasks beyond what the business configuration UI provides (e.g., time zone changes, hard-coded value parameterization, edge server connection management, database-level issue resolution). Does not run the system daily and is not a primary actor in business workflows.

**Key exclusive capabilities.**
- Manage functional role-to-business-role-type assignments (which system functions, pages, and capabilities each business role type can access). Not available to any business role; changes require a formal change request and release.
- Full platform access including technical configuration forms not accessible to business users.
- Database-level access for exceptional corrections (intended to be reduced via parameterization in the new system).

**Spec constraints for AI.**
- System design should aim to surface parameterizable settings to P-05a or P-03a rather than requiring IT intervention.
- Do not generate functional requirements for this persona's workflows — their actions are outside the standard application spec scope.
- Any functionality currently requiring System Admin intervention should be flagged as a candidate for a dedicated admin UI feature in the new system.

---

## System Actors

> These are non-human actors that interact with the platform. Listed for integration spec context only — **do not derive functional requirements from this section**; it reflects current state.

| ID | Name | Role | Serves |
|---|---|---|---|
| SA-01 | Edge Servers (MDC) | Push machine events (stops, volumes, rejects, sensor readings) via HTTPS/JSON | P-01, P-02a, P-02b |
| SA-02 | SAP S/4HANA | Bidirectional data exchange: 18 entity types (materials, production orders, confirmations, etc.) | P-02b, P-03a, P-05a |
| SA-03 | ServiceNow SDM | Bidirectional: receives shift schedules outbound; sends planned activities inbound | P-03a, P-05a |
| SA-04 | Data Lake | Consumes production facts (15–30 min lag) and master data snapshots (2–24h lag) via Event Hubs consumer group | P-05a |
| SA-05 | SAP BW / GSCA KPI | Pulls consolidated KPI data daily for global reporting | P-05a |
| SA-06 | Romania Waste Management | Calls GSCA REST API on demand for waste tracking records | P-02b (Romania only) |
| SA-07 | Azure Entra ID | Authenticates all users via SSO; enforces MFA for admin roles | All personas |
| SA-08 | Notification Service | Dispatches scheduled automated email reports and critical sensor alerts to configured recipient lists (distribution lists managed in AD) | P-02b, P-04, P-05a, P-05b |

---

## User and Permission Management Model

**Architecture.** One AD security group maps one-to-one to one Proteus business role type scoped to a factory (or global). A user may belong to multiple AD groups (e.g., a small-factory analyst may hold P-02a + P-02b + P-03a simultaneously). Users with no AD group linked to Proteus receive no access and are shown a prompt to request access via IT portal.

**Provisioning channels.** Each business role type has two AD groups:
- **Birthright** — auto-provisioned based on employee attributes (country, department, org hierarchy). Automation removes members who no longer meet criteria; manual additions must not be mixed into this group.
- **Request-based** — provisioned via IT portal (MyIT).

**Scale.** At full rollout: ~360 factory-specific groups (30 factories × 6 business role types × 2 channels) plus a small set of global groups (Global Viewer, P-05a, P-05b, P-06). All factory groups are created as part of new factory onboarding.

**Functional element management.** The mapping of what each business role type can do in the platform is managed by P-06 (System Admin) only. It is not self-service. Changes require a formal change request, development, QA, and a release. The initial release will include all anticipated business role types pre-configured.

---

## Open Questions

| ID | Question | Impact |
|---|---|---|
| OQ-01 | Role code for P-02c (Production Machine Expert) not yet defined in role workbook. What `biz.*` code? | Blocks RBAC design and permission scope specs |
| OQ-02 | Security groups in role workbook not yet mapped to personas — review needed | Gaps may exist in RBAC coverage |
| OQ-03 | Confirm acronym expansions: OPI-DPA = Operational Performance Improvement — Digital Performance Analysis; OPI-GC = Operational Performance Improvement — Global Quality | Unblocks final persona naming and role-code labelling in RBAC matrix |
| OQ-04 | P-06 (System Admin) specific technical capabilities not fully documented — session with Florin needed | Required before system-level config specs can be finalized |
| OQ-05 | Cell Lead (IWS) not formally defined with system role code or permission scope | May need a persona entry once IWS role mapping is confirmed |
| OQ-06 | Maintenance Lead / `biz.MechanicMaintenanceManager` — role workbook lists this code; no meeting has elaborated on system interactions | May need a persona or permission extension once IWS maintenance lead mapping is confirmed |

---

## Persona Quick Reference

| ID | Name | Role Code | Scope | Write Level |
|---|---|---|---|---|
| P-01 | Shop-floor Operator | `biz.SFOperator` | Single plant / line | Current shift only |
| P-02a | Process Lead / Shift Supervisor | `biz.ShiftLeader` | Single plant / area | Current + previous shift (wider window) |
| P-02b | Production Analyst | `biz.ProdOffice` | Single plant (or multi) | Open KPI period |
| P-02c | Production Machine Expert | TBD | Single plant | MDC mapping + tag config only |
| P-03a | Local Config Admin (Perf) | `biz.ConfigAdmin` | Single plant | Full plant config |
| P-03b | QSens Local Admin (Quality) | `biz.QSensConfigAdmin` | Single plant (QSens) | QSens config for plant |
| P-04 | Data Consumer | `biz.GlobalViewer` | Local or global | Read-only |
| P-05a | GSCA Global Admin (OPI-DPA) | `biz.GlobConfigAdmin` | All plants | Global config + all plant data |
| P-05b | Global QSens Admin (OPI-GC) | `biz.QSensGlobalAdmin` | All plants (QSens) | Global QSens standards |
| P-06 | System Administrator | TBD (IT) | Platform | Technical/IT only |
