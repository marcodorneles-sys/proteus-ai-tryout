# PERMISSIONS MATRIX — Project Proteus

**Status:** Scaffold v0.1 (to be completed with PO/SME) · **Last Updated:** 2026-06-25 · **Owner:** Julia Shcherbyna (BA Lead)
**Repository Location:** `specs/_rules/PERMISSIONS_MATRIX.md`
**Related:** [`personas.md`](personas.md) · [`FEATURE_INVENTORY.md`](FEATURE_INVENTORY.md) · [`PRD.md`](PRD.md) · `specs/_quality/SECURITY_REQUIREMENTS.md`

---

## Purpose & Scope

Canonical **role × feature access** mapping. This file is where access levels and the AD-group →
role → permission model live — they are **not** duplicated in the PRD or in feature docs (feature
docs reference role IDs from here). Security *requirements* (encryption, TLS, secrets, audit log,
network isolation, MFA) live separately in `specs/_quality/SECURITY_REQUIREMENTS.md`.

> **This is a scaffold.** Module-level access is seeded from persona write-scopes in `personas.md`;
> feature-level cells are partially seeded and marked where confirmation is needed. Complete it
> feature-by-feature with the PO and System Admin (P-06) before ticket/spec generation.

---

## Conventions

**Access-level legend**

| Code | Meaning |
|---|---|
| **C** | Create / Configure — full manage (create, edit, deactivate) of configuration or master data |
| **W** | Write — create/edit operational data within the role's scope and time window |
| **A** | Approve / Activate / Validate — governance action (e.g., activate reason, lock period, validate library) |
| **R** | Read only |
| **G** | Global write — same as C/W but across **all** factories (cross-factory scope) |
| **—** | No access |
| **?** | To be confirmed |

**Scope qualifiers** (suffix): `^p` = single plant · `^g` = global/all plants · `^win` = limited to an edit/time window.
**Default scope** for P-01…P-03 is single plant (`^p`); P-04 is plant or global; P-05/P-06 are global (`^g`).

---

## Cross-Cutting Access Rules (override individual cells)

These come from `personas.md` (Cross-Cutting Rules) and the User & Permission Management Model. They
take precedence over anything in the matrices below:

1. **Human-input record model.** No role may `UPDATE`/`DELETE` machine-originated data. "Write" on machine data means writing a separate human-input record; output is recalculated. Only MDC sync updates machine data.
2. **Shift handover window.** Write access to previous-shift data expires after a configurable window (default 30 min after shift end). Enforce at the API layer, not only UI.
3. **Downtime reason creation is release-gated.** No role creates downtime reasons at runtime. Activation/deactivation of existing reasons is **P-05a only**.
4. **New permission groups / functional elements are release-gated.** What each business role type can access is defined by **P-06 only**, via change request + release — never self-service.
5. **Factory SAP/non-SAP config flag.** Extra capabilities (PO confirmation, PO creation) are gated by a factory-level flag, not a separate role.
6. **No report-level security.** Any authorized user may read any report. Enforce only at the role/factory boundary.
7. **Two-layer provisioning.** Entra ID manages accounts/group membership; Proteus manages what each group can do. Never conflate.
8. **Speed entry mandatory reason.** Hard validation — no save without a reason code.
9. **QSens sensor check is a write operation.** Cockpit is bidirectional; never spec as read-only.
10. **Plant-level data isolation.** Single-plant roles never see other factories' data; global roles need no per-factory provisioning.

---

## Role Catalog

| Persona ID | Role | Role Code (`biz.*`) | Default Scope |
|---|---|---|---|
| P-01 | Shop-floor Operator | `biz.SFOperator` | Plant / line |
| P-02a | Process Lead / Shift Supervisor | `biz.ShiftLeader` | Plant / area |
| P-02b | Production Analyst | `biz.ProdOffice` | Plant (or multi) |
| P-02c | Production Machine Expert | TBD (OQ-01) | Plant |
| P-03a | Local Configuration Admin (Performance) | `biz.ConfigAdmin` | Plant |
| P-03b | QSens Local Admin (Quality) | `biz.QSensConfigAdmin` | Plant (QSens) |
| P-04 | Data Consumer | `biz.GlobalViewer` (global) | Plant or global |
| P-05a | GSCA Global Administrator (OPI-DPA) | `biz.GlobConfigAdmin` | Global |
| P-05b | Global QSens Administrator (OPI-GC) | `biz.QSensGlobalAdmin` | Global (QSens) |
| P-06 | System Administrator (IT) | TBD (IT-managed) | Platform |

---

## Module-Level Access Matrix (seeded)

> Best-effort from persona write-scopes. Confirm before relying on it for specs.

| Module | P-01 | P-02a | P-02b | P-02c | P-03a | P-03b | P-04 | P-05a | P-05b | P-06 |
|---|---|---|---|---|---|---|---|---|---|---|
| M-01 Production Data Capture | W^win | W^win | W^p | — | C^p | — | R | G | — | — |
| M-02 Quality Monitoring (QSens) | W (checks) | R | R | C (tag codes) | — | C^p | R | R | G | — |
| M-03 MDC Ingestion & Classification | — | — | — | C^p (local map) | — | — | — | G/A | — | R |
| M-04 Master Data & Configuration | R | R | W^p (schedule) | W (machine tech attrs) | C^p | — | R | G | — | C (technical) |
| M-05 Performance KPI & Reporting | R | W (verify) | W/A (lock) | — | R | R | R | A (consolidate) | R | — |
| M-06 External Integration | — | W (import issues) | W (import issues) | — | R | — | — | A (BW export) | — | C (APIM/tech) |
| M-07 Platform Services | use | use | R (audit) | use | C^p (role assign) | use | use | use | use | C (func. elements) |

---

## Feature-Level Matrix

Access codes per feature × role. Seeded for M-01 and M-02 as worked examples; M-03–M-07 list rows
with the primary actor seeded and remaining cells `?` to complete.

### M-01 — Production Data Capture (Machinery)

| Feature | P-01 | P-02a | P-02b | P-03a | P-04 | P-05a | Notes |
|---|---|---|---|---|---|---|---|
| F-01.1 SDE | W^win | W^win | W^p | C^p (correct) | R | G | Reclassify = human-input record |
| F-01.2 Reject Entry | W^win | W | W^p | R | R | G | Office variant for waste handlers |
| F-01.3 Actual Speed Entry | W^win | W | W^p | R | R | G | Mandatory reason code |
| F-01.4 Shift Remainders | W^win | W (review) | W^p | R | R | G | Flag remainder > 1 pallet |
| F-01.5 Data Review & Correction forms | — | W | W | R | R | G | Daily routine; bulk + zero-result confirm |
| F-01.6 Office Screen | R | R | R | R | R | R | Read/monitor; no write |
| F-01.7 Non-SAP PO entry & confirmation | W (confirm)¹ | W (create)¹ | R | R | R | G | ¹ Gated by factory non-SAP flag |

### M-02 — Quality Monitoring (QSens)

| Feature | P-01 | P-02c | P-03b | P-04 | P-05a | P-05b | Notes |
|---|---|---|---|---|---|---|---|
| F-02.1 QSens Cockpit | W (checks) | R | R | R | — | R | Check = write |
| F-02.2 Aggregated Cockpit | R | R | R | R | — | R | Factory-level quality aggregation |
| F-02.3 Local sensor configuration | — | C (tag codes) | C^p | R | — | R | P-02c tags vs P-03b business params |
| F-02.4 Global sensor governance | — | — | R | R | — | G/A | Global templates/parameters |
| F-02.5 Global template synchronization | — | — | W^p (accept) | R | — | A (initiate) | Review/accept/escalate workflow |
| F-02.6 Sensor adherence KPI engine | R | — | R | R | — | R | System-computed |
| F-02.7 Sensor check rules & procedures | W (perform) | — | C^p | R | — | C/A | Rules at template level |
| F-02.8 Sensor inactive rules | — | — | R | R | — | C/A | PO/material-driven auto-deactivation |
| F-02.9 Quality breakdown alerts | — | — | C^p | R | — | R | Threshold config |
| F-02.10 Sensor reject reason assignment | — | C | C^p | R | — | A | Within F-02.4 |

### M-03 — MDC Ingestion & Classification (SmartMDC)

| Feature | Primary actor | P-02c | P-05a | P-06 | Others |
|---|---|---|---|---|---|
| F-03.1 Edge event ingestion pipeline | System | ? | ? | R | ? |
| F-03.2 MDC classification hierarchy mgmt | P-05a | ? | G/C | ? | ? |
| F-03.3 Global breakdown/reject libraries + templates | P-05a | R | G/C | ? | ? |
| F-03.4 Local translation & mapping workflow | P-02c | C^p | A (validate) | ? | ? |
| F-03.5 Stop reason translations | P-02c | W^p | ? | ? | ? |
| F-03.6 MDC raw data reports | P-02c | R | ? | R | integration team R |

### M-04 — Master Data & Configuration

| Feature | Primary actor | P-03a | P-02b | P-02c | P-05a | P-06 | Others |
|---|---|---|---|---|---|---|---|
| F-04.1 Organizational hierarchy mgmt | P-03a | C^p | ? | ? | G (validate) | ? | ? |
| F-04.2 Workcenters & machines master data | P-03a | C^p | ? | W (tech attrs) | ? | ? | ? |
| F-04.3 Shift Schedule Management | P-03a | C^p | W^p | ? | ? | ? | ? |
| F-04.4 Downtime configuration | P-03a | C^p | ? | ? | G (locations/reasons) | ? | location structure view-only for P-03a |
| F-04.5 Translation management | P-03a | C^p | ? | ? | ? | ? | ? |
| F-04.6 System config & factory preferences | P-03a | C^p | ? | ? | ? | C (technical params) | ? |
| F-04.7 Machine go-live activation | P-03a | C^p | ? | W | A (oversight) | ? | within F-04.2 |
| F-04.8 Reject type definitions | P-03a | C^p | ? | ? | ? | ? | ? |

### M-05 — Performance KPI & Reporting

| Feature | Primary actor | P-02a | P-02b | P-04 | P-05a | Others |
|---|---|---|---|---|---|---|
| F-05.1 Central KPI calculation engine | System | R | R | R | R | system-computed |
| F-05.2 OE / OEE Report | P-02a/P-02b | W?/R | R | R | R | review/correct via source forms |
| F-05.3 Reject Rate Report | P-02a/P-02b | R | R | R | R | ? |
| F-05.4 Downtime List / Reject List reports | P-02a/P-02b | R | R | ? | ? | ? |
| F-05.5 Common report framework | All report users | R | R | R | R | ? |
| F-05.6 Email notifications & alerts | Recipients | ? | R (recipient) | R | R | recipient lists set by P-03a |
| F-05.7 KPI period mgmt & month-end locking | P-02b | ? | W/A | ? | A (consolidate) | guardrails post-lock |
| F-05.8 Extended report set | P-02/P-04 | R | R | R | R | ? |
| F-05.9 Team Comparison Report | P-02b | R | R | R | ? | ? |

### M-06 — External Integration

| Feature | Primary actor | P-02b | P-03a | P-05a | P-06 | Others |
|---|---|---|---|---|---|---|
| F-06.1 SAP master data sync | System | ? | ? | A (validate) | ? | ? |
| F-06.2 SAP PO & confirmation exchange | System | W (resolve issues) | ? | ? | ? | P-02a resolve |
| F-06.3 ServiceNow SDM integration | System | ? | stakeholder | stakeholder | ? | ? |
| F-06.4 Data Lake export | System | ? | ? | ? | ? | Data Lake team |
| F-06.5 GSCA KPI export to SAP BW | P-05a | ? | ? | A (trigger) | ? | ? |
| F-06.6 Local reject / waste management API | External | W (Romania) | ? | ? | ? | Romania only |
| F-06.7 Non-SAP (QIMS/KIMS) integration | System | ? | ? | ? | ? | non-SAP factories |
| F-06.8 APIM gateway configuration | P-06/DevOps | — | — | — | C | ? |

### M-07 — Platform Services

| Feature | Primary actor | P-03a | P-02b | P-06 | All | Notes |
|---|---|---|---|---|---|---|
| F-07.1 Authentication & SSO | All | use | use | C (config) | use | shared kiosk accounts; MFA admins |
| F-07.2 RBAC & user role management | P-03a | C^p (assign) | ? | C (catalog) | subject | role-assignment overview screen |
| F-07.3 Audit trail & data change history | Auditors | ? | R | R | — | immutable; ≥12-month retention |
| F-07.4 Navigation, menu & homepage | All | ? | ? | ? | use | role-based menu visibility |
| F-07.5 Global navigation search | All | ? | ? | ? | use | ? |
| F-07.6 Reusable table search & filter | All | ? | ? | ? | use | ? |
| F-07.7 Favourites | Power users | use | use | ? | use | limited value on shared accounts |
| F-07.8 Data quality check framework | P-02b/P-05 | ? | R (alerts) | ? | ? | phase TBD |

---

## Open Questions

| OQ | Question | Owner |
|---|---|---|
| PM-01 | Role code for P-02c (Production Machine Expert) — `biz.*` TBD (personas OQ-01). | BA / Florin |
| PM-02 | Confirm P-05a vs P-05b split per feature where the matrix shows P-05a/P-05b ambiguity (esp. M-02, M-03). | BA / OPI |
| PM-03 | Confirm P-02a write access to KPI reports vs read-only (corrections flow through source forms). | BA / P-02a SMEs |
| PM-04 | Map each `biz.*` role to its birthright vs request-based AD groups (~360 factory groups + global). | P-06 / IT |
| PM-05 | Confirm P-06 functional-element catalog: exact list of release-gated functional elements per role. | P-06 / Florin |
| PM-06 | Define access for non-human/system actors (SA-01…SA-08 in personas.md) at the integration boundary. | BA / Architecture |

---

## AD-Group & Provisioning Model (reference)

- One AD security group ↔ one Proteus business role type, scoped to a factory or global.
- A user may hold multiple groups (small-factory user may be P-02a + P-02b + P-03a).
- Two provisioning channels per role type: **Birthright** (auto from employee attributes) and **Request-based** (via MyIT portal).
- Scale at full rollout: ~360 factory-specific groups (30 factories × 6 business role types × 2 channels) + a small set of global groups (Global Viewer, P-05a, P-05b, P-06).
- Users with no Proteus-linked AD group get no access and a prompt to request via IT portal.
