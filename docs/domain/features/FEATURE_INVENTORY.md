# FEATURE INVENTORY — Project Proteus

**Status:** Draft v0.1 · **Last Updated:** 2026-06-25 · **Owner:** Julia Shcherbyna (BA Lead)
**Repository Location:** `specs/_context/FEATURE_INVENTORY.md`
**Related:** [`PRD.md`](PRD.md) · [`personas.md`](personas.md) · [`GLOSSARY.md`](GLOSSARY.md) · `specs/_rules/PERMISSIONS_MATRIX.md`

---

## Purpose

Canonical, machine-readable list of in-scope features and the **traceability spine** for the
spec-driven flow (`Module → Feature → FR → Work Item → specify.md`). It mirrors the human planning
source `General/Implementation Scope & Roadmap.xlsx` (sheet *4. Scope - Features*) and exists so
completeness can be verified at every stage — most importantly the gap check that compares
`specify.md` back to its ticket and feature requirement doc.

**Authoritative for:** feature IDs (`F-xx.y`), feature → module → persona mapping, scope status.
**Not authoritative for:** effort estimates, wave/scheduling, and risk levels — those live in the
xlsx (*8. Release & Phasing*, *9. Roadmap*). Improvements and post-release items live in xlsx
sheets *5. Improvements* and *6. Post-Release Scope*; see PRD §13.4.

**Conventions:** IDs are immutable (retire, never renumber). Priority values: Critical / High /
Medium / Low. Personas use IDs from `personas.md` (P-05 = P-05a and/or P-05b where the source is
not split).

---

## Module Index

| Module ID | Module Name | Architecture module (PRD App. B) |
|---|---|---|
| M-01 | Production Data Capture (Machinery) | MCH |
| M-02 | Quality Monitoring (QSens) | QSN |
| M-03 | MDC Ingestion & Classification (SmartMDC) | ING |
| M-04 | Master Data & Configuration | MDT |
| M-05 | Performance KPI & Reporting | RPT |
| M-06 | External Integration | INT |
| M-07 | Platform Services (cross-cutting) | (cross-cutting) |

---

## Feature Inventory

### M-01 — Production Data Capture (Machinery)

| Feature ID | Feature | Priority | Primary Personas | Phase | Outcome (one-line) |
|---|---|---|---|---|---|
| F-01.1 | Smart Downtime Entry (SDE) | Critical | P-01; P-02a | Phase 1 | Complete, correctly classified downtime record per shift — backbone of all performance KPIs |
| F-01.2 | Reject Entry (shop-floor + office variant) | Critical | P-01; waste handlers; P-02a | Phase 1 | Accurate reject data per PO/shift feeding reject KPIs |
| F-01.3 | Actual Speed Entry | Critical | P-01; P-02a | Phase 1 | Correct speed basis for performance KPIs; deviations always explained (mandatory reason) |
| F-01.4 | Shift Remainders | Critical | P-01; P-02a | Phase 1 | Leftover-pallet carry-over between shifts accounted for; anomalies (>1 pallet) investigated |
| F-01.5 | Data Review & Correction forms | Critical | P-02a; P-02b | Phase 1 | Daily data verification complete before DDS; trustworthy KPI inputs |
| F-01.6 | Office Screen | Medium | P-04; P-02a | Phase 1 | Live production shift-KPI visibility for direction-setting (distinct from F-02.2) |
| F-01.7 | Non-SAP factory PO entry & confirmation | Critical | P-02a; P-01 | Phase 1 | Non-SAP factories operate fully on the platform — no parallel legacy system |

### M-02 — Quality Monitoring (QSens)

| Feature ID | Feature | Priority | Primary Personas | Phase | Outcome (one-line) |
|---|---|---|---|---|---|
| F-02.1 | QSens Cockpit (machine / work center view) | Critical | P-01; P-04 | Phase 1 | Real-time quality visibility; sensor checks completed; KPI losses explainable in place |
| F-02.2 | Aggregated Cockpit (factory level) | Critical | P-04; P-03b; P-05 | Phase 1 | Factory-wide quality oversight; fast detection of disconnected/out-of-sync machines (distinct from F-01.6) |
| F-02.3 | Local sensor configuration | Critical | P-03b; P-02c | Phase 1 | Machines correctly instrumented for quality KPIs without code changes |
| F-02.4 | Global sensor governance | Critical | P-05b | Phase 1 | Consistent, comparable quality configuration/KPIs across factories |
| F-02.5 | Global template synchronization | High | P-03b; P-05b | Phase 1 | Local configs stay aligned with global standards without manual cross-checking |
| F-02.6 | Sensor adherence KPI engine | Critical | System (consumed by P-04/P-03b/P-05) | Phase 1 | Trustworthy quality KPIs — the primary value driver of QSens |
| F-02.7 | Sensor check rules & procedures | Medium | P-03b/P-05b; P-01 | Phase 1 | Calibration checks happen when needed and are traceable |
| F-02.8 | Sensor inactive rules | Medium | P-05b; System | Phase 1 | KPIs not distorted by sensors irrelevant to current production |
| F-02.9 | Quality breakdown alerts | Medium | P-03b | Phase 1 | Bypassed quality checks become visible and investigable |
| F-02.10 | Sensor reject reason assignment & trends | Critical | P-03b/P-02c; P-01 | Phase 1 | Reject patterns attributable to specific sensors. *Implemented within F-02.4* |

### M-03 — MDC Ingestion & Classification (SmartMDC)

| Feature ID | Feature | Priority | Primary Personas | Phase | Outcome (one-line) |
|---|---|---|---|---|---|
| F-03.1 | Edge event ingestion pipeline | Critical | Edge Servers; all modules (consumers) | Phase 1 | Every machine event reliably lands once, validated and enriched (30–50K/min, ≤10s) |
| F-03.2 | MDC classification hierarchy management | Critical | P-05a | Phase 1 | Single standardized vocabulary for machine events across factories |
| F-03.3 | Global breakdown & reject libraries + machine templates | Critical | P-05a | Phase 1 | One validated global truth per reason, reusable across factories/machines |
| F-03.4 | Local translation & mapping workflow | Critical | P-02c; P-05a (validation) | Phase 1 | New machine reasons become reportable quickly with minimal duplicated effort |
| F-03.5 | Stop reason translations | Medium | P-02c / local users | Phase 1 | Operators see stop reasons in their language — higher data-entry quality |
| F-03.6 | MDC raw data reports | Critical | P-02c; P-06; integration team | Phase 1 | New machine connections verifiable; data issues diagnosable at source |

### M-04 — Master Data & Configuration

| Feature ID | Feature | Priority | Primary Personas | Phase | Outcome (one-line) |
|---|---|---|---|---|---|
| F-04.1 | Organizational hierarchy management | Critical | P-03a; P-05a | Phase 1 | One correct org backbone consumed by every module and report filter |
| F-04.2 | Workcenters & machines master data | Critical | P-03a; P-02c | Phase 1 | Machines/lines accurately represented (incl. parallel production config, line history) |
| F-04.3 | Shift Schedule Management | Critical | P-03a; P-02b | Phase 1 | Valid schedules always available (incl. month-end 3-segment); prerequisite for SDE/QSens/KPIs |
| F-04.4 | Downtime configuration | Critical | P-03a | Phase 1 | Operators see exactly the right downtime options — faster, cleaner classification |
| F-04.5 | Translation management | Critical | P-03a | Phase 1 | Consistent multilingual UI and data labels across factories |
| F-04.6 | System config & factory preferences | Critical | P-03a; P-06 | Phase 1 | Factory specifics configurable without code or IT tickets (incl. SAP/non-SAP flag) |
| F-04.7 | Machine go-live activation | Critical | P-03a; P-02c; P-05a | Phase 1 | Controlled, transparent machine activation. *Implemented within F-04.2* |
| F-04.8 | Reject type definitions | Critical | P-03a | Phase 1 | Consistent reject classification driving entry and reporting |

### M-05 — Performance KPI & Reporting

| Feature ID | Feature | Priority | Primary Personas | Phase | Outcome (one-line) |
|---|---|---|---|---|---|
| F-05.1 | Central KPI calculation engine | Critical | System (all KPI consumers) | Phase 1 | One trustworthy set of numbers everywhere — no divergent KPIs across screens/reports/BW |
| F-05.2 | OE / OEE Report | Critical | P-02a; P-02b; P-04 | Phase 1 | Daily performance steering from one report (adds IWS metrics, equipment-level) |
| F-05.3 | Reject Rate Report | Critical | P-02a; P-02b; P-04 | Phase 1 | Reject performance visibility and quality steering |
| F-05.4 | Downtime List / Reject List reports | Critical | P-02a; P-02b | Phase 1 | Trustworthy record-level analysis behind the KPIs (human + machine input) |
| F-05.5 | Common report framework | Critical | All report users | Phase 1 | Consistent report UX; faster delivery of each subsequent report |
| F-05.6 | Email notifications & alerts | Critical | P-02b; P-04; P-05 | Phase 1 | Right people informed automatically (email-only initial release) |
| F-05.7 | KPI period management & month-end locking | Critical | P-02b; P-05a | Phase 1 | Clean, locked monthly KPI data for Finance/Global/BW — on time |
| F-05.8 | Extended report set (~75 reports) | Medium | P-02; P-04 | Phase 1 | Full reporting-surface parity where still justified |
| F-05.9 | Team Comparison Report | Critical | P-02b; P-04 | Phase 1 | Team/shift performance steering on the same KPI definitions |

### M-06 — External Integration

| Feature ID | Feature | Priority | Primary Personas | Phase | Outcome (one-line) |
|---|---|---|---|---|---|
| F-06.1 | SAP S/4HANA master data sync | Critical | System; P-05a (validate) | Phase 1 | Platform reference data stays aligned with the enterprise source of truth |
| F-06.2 | SAP S/4HANA production order & confirmation exchange | Critical | System; P-02a/P-02b | Phase 1 | Current PO context everywhere; zero-discrepancy volume reconciliation target |
| F-06.3 | ServiceNow SDM integration | Critical | System; P-03a/P-05a | Phase 1 | SDM always has current schedules; planned activities visible in platform |
| F-06.4 | Data Lake export | Critical | Data Lake team; P-05a | Phase 1 | Enterprise analytics supplied (15–30 min freshness) without coupling to Lake internals |
| F-06.5 | GSCA KPI export to SAP BW | Critical | P-05a (trigger/validate) | Phase 1 | Comparable global KPI reporting across factories in SAP BW |
| F-06.6 | Local reject application / waste management API | Critical | External system; P-02b (Romania) | Phase 1 | Regulatory waste tracking and local reject-app continuity (on-demand REST via APIM) |
| F-06.7 | Non-SAP factory (QIMS/KIMS) integration | Critical | System; non-SAP factories | Phase 1 | Non-SAP factories fully served — no parallel legacy operation |
| F-06.8 | APIM gateway configuration | Critical | P-06 / DevOps | Phase 1 | Governed, secured, observable external API surface |

### M-07 — Platform Services (cross-cutting)

| Feature ID | Feature | Priority | Primary Personas | Phase | Outcome (one-line) |
|---|---|---|---|---|---|
| F-07.1 | Authentication & SSO | Critical | All personas | Phase 1 | Secure, context-appropriate access (office + shared kiosk); MFA for admins |
| F-07.2 | RBAC & user role management | Critical | P-03a; all (subject) | Phase 1 | Right access per factory pattern; local admins accountable for granted permissions |
| F-07.3 | Audit trail & data change history | Critical | Auditors; P-02b (read); P-06 | Phase 1 | Traceability of every change — closes the legacy 'no audit trail' gap (≥12-month retention) |
| F-07.4 | Navigation, menu & homepage | Critical | All personas | Phase 1 | Fast orientation across ~165 forms/reports; ~two-click reach |
| F-07.5 | Global navigation search | Critical | All personas (esp. power users) | Phase 1 | Direct access without memorizing menu paths (navigation search only) |
| F-07.6 | Reusable table search & filter component | High | All personas (tabular views) | Phase 1 | Consistent, improved filtering UX everywhere |
| F-07.7 | Favourites | Medium | P-02b; P-03a; power users | Phase 1 | Quicker access for office power users |
| F-07.8 | Data quality check framework | Medium | P-02b; P-05 | **TBD** | Bad data caught at entry instead of during month-end correction (phase under review) |

---

## Coverage Summary

| Module | # Features | Critical | High | Medium |
|---|---|---|---|---|
| M-01 | 7 | 6 | 0 | 1 |
| M-02 | 10 | 7 | 1 | 4* |
| M-03 | 6 | 5 | 0 | 1 |
| M-04 | 8 | 8 | 0 | 0 |
| M-05 | 9 | 8 | 0 | 1 |
| M-06 | 8 | 8 | 0 | 0 |
| M-07 | 8 | 6 | 1 | 1 |
| **Total** | **56** | **48** | **2** | **6** |

\* M-02 priority counts: 7 Critical, 1 High (F-02.5), 4 Medium (F-02.7–F-02.9) — table totals are
indicative; the xlsx remains authoritative for priority.

---

## Notes & Open Questions

| ID | Note / Question | Owner |
|---|---|---|
| FI-01 | F-02.10 and F-04.7 are delivered *within* parent features (F-02.4 and F-04.2). Keep their IDs for traceability but expect no standalone Work Item. | BA |
| FI-02 | F-07.8 (Data quality check framework) phase is TBD — confirm Phase 1 vs post-release before ticket generation. | BA / PO |
| FI-03 | Persona references use P-05 where the xlsx is not split; confirm P-05a vs P-05b per feature during PERMISSIONS_MATRIX build. | BA |
| FI-04 | Effort/wave/risk intentionally excluded — keep them only in `Implementation Scope & Roadmap.xlsx` to avoid dual-maintenance. | BA |
