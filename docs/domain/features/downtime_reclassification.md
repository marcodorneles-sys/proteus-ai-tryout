# downtime_reclassification.md

## Module Reference
This feature belongs to the **Smart Downtime Entry Module** (M-01 Production Data Capture).
Feature ID: **F-01.1** (Smart Downtime Entry — Critical, Phase 1).
`# path: docs\domain\modules\smart_downtime_entry.md`

---

## ADR
Architecture Decision Record Reference
`# path: docs\adr\0001-core-technology-runtime-stack.md`

---

## Overview
**Downtime Reclassification** is the capability within Smart Downtime Entry that allows operators to interact with raw machine stops captured by MDC and assign human context, root causes, and standardized classification to automated machine events.

The machine produces raw stop records (start time + end time + PLC reason code + reason description). These reason codes are machine-specific, often cryptic or non-English, and not actionable on their own. SmartMDC enriches them with a standardized classification hierarchy (group → type → unit → sub-reason → assembly) and pre-populates the available options the operator sees. The operator's act of selecting and confirming a stop reason writes a **separate human-input record** — the machine-originated record is never modified.

The output/reporting layer (KPI engine, Downtime List report, OEE) always recalculates from three sources combined: machine data + MDC mapping + human input. This is a hard architectural constraint — no `UPDATE` or `DELETE` on machine-originated records under any user action.
For the full project context, see `docs\domain\PRD.md` and `docs\domain\GLOSSARY.md`

---

## Persona
**Shop-floor Operator (P-01 — `biz.SFOperator`)** — primary owner on the shop floor.
Write scope: current shift only; previous shift within the shift handover window (default: 30 min after shift end, configurable per factory).

The same reclassification capability is reused by the **Process Lead / Shift Supervisor (P-02a — `biz.ShiftLeader`)** through the office review surface, with a wider writable window (current and previous shift, multiple lines, factory-configurable duration). The **Production Analyst (P-02b — `biz.ProdOffice`)** can reclassify within the open KPI period.

Permissions are governed by `PERMISSIONS_MATRIX.md` (F-01.1 row). 
Use `PERSONAS.md` document for the context about user personas that use this functionality
# path: `docs\domain\PERSONAS.md`
# path: `docs\domain\PERMISSIONS_MATRIX.md`

---

## Key Capabilities

### 1. View Machine Stops on the Timeline
The operator sees raw machine stop events (red segments) on the SDE timeline for the current shift and their assigned line. Each stop displays: start time, end time, duration, and the raw MDC reason code/description as received from the machine. Stops are displayed in chronological order. The operator does not manually enter timestamps — all time boundaries are derived from MDC data.

### 2. Assign a Downtime Reason (Reclassify)
The operator selects a stop and assigns a standardized downtime reason from a pre-populated list. The available options are sourced from the SmartMDC classification hierarchy (the MDC mapping configured by the Mapping Responsible / P-02c). The operator never sees raw PLC codes — they see the human-readable standardized English description.

On save, the system writes a **new human-input record** to a separate table. The machine stop record is untouched. The KPI engine picks up the human-input record at next recalculation.

### 3. Bulk Reclassification
The operator or Process Lead can select multiple stops and assign a single Reason in one action. Selection is operator-driven — no system-enforced "same type" constraint. Single-record workflows are insufficient; bulk operations are required for both P-01 and P-02a.

### 4. System-Embedded Business Rules (Non-Configurable at Runtime)
The following reclassification rules are embedded in system logic and are **never exposed as user-facing choices**. They filter or constrain available options automatically:

| Rule | Description |
|---|---|
| **Override Runtime** | Determines whether a human-input reason overrides the MDC-mapped reason in the KPI output, or whether both are preserved separately. |
| **Ignore MDC Stop** | Marks a machine-generated stop as excluded from downtime reporting (e.g., false trigger). The stop record is retained; output recalculates without it. |
| **Minimum Duration** | Stops below a configured minimum duration threshold are either suppressed or handled differently (e.g., not shown as classifiable entries). Threshold is system/config-defined, not user-set per action. |
| **Can Insert into Future** | Controls whether a user may insert a manual downtime entry for a time window that is in the future relative to the current shift progress. |
| **Long Lasting** | Identifies stops that exceed a configured duration threshold, triggering different handling or mandatory classification before the stop is considered complete. |

These rules filter the options displayed and the actions available — operators make choices from the filtered set only.

### 5. Downtime Reason Source (SmartMDC Dependency)
Stop reason options visible to the operator are pre-populated from SmartMDC machine template mappings. A work center must have its MDC mapping complete and its Smart MDC Go-Live date set before it appears in SDE. Unmapped reason codes do not appear as selectable options — they surface as unclassified stops requiring attention.

### 6. Scope and Time Window Constraints
- **P-01:** current shift only; previous-shift write access expires at shift handover window close.
- **P-02a:** current and previous shifts; window is factory-configurable and wider than P-01.
- **P-02b:** reclassification within the open KPI period.
- **Downtime reason creation at runtime is prohibited** for all roles. Available reasons are defined and activated/deactivated only by P-05a (Global Administrator) via a platform release.
- Write access is enforced at the **API layer**, not only in UI — shift window validation is a backend constraint.

---

## Data Model Notes
- Machine stop record: `start_time`, `end_time`, `reason_code`, `reason_description` (immutable after MDC ingestion).
- Human-input record (separate table): `stop_ref_id` (FK to machine stop), `human_reason_id` (FK to MDC classification), `user_id`, `timestamp`, `action_type` (classify / bulk-classify / ignore).
- KPI calculation always reads: machine data + MDC mapping + human-input records combined.
- Audit trail required: every human-input record is traceable to user and timestamp (≥12-month retention, per F-07.3).

---

## Dependencies
| Dependency | Type | Notes |
|---|---|---|
| SmartMDC classification hierarchy (F-03.2 / F-03.3) | Upstream | Provides the available reason options displayed to operator |
| Local translation & mapping workflow (F-03.4) | Upstream | Work center must be mapped before appearing in SDE |
| Downtime configuration (F-04.4) | Configuration | Defines which reasons are active per factory |
| Shift Schedule Management (F-04.3) | Configuration | Defines shift boundaries used for window enforcement |
| MDC Go-Live activation (F-04.7) | Prerequisite | Smart MDC Go-Live must be set for the work center |
| KPI calculation engine (F-05.1) | Downstream | Consumes human-input records for OEE/OE calculation |
| Downtime List report (F-05.4) | Downstream | Surfaces the combined machine + human record view |

---

## Open Questions
| ID | Question | Impact | Status | Resolution |
|---|---|---|---|---|
| OQ-DR-01 | What is the exact configurable range for the shift handover window per factory? Is there a global default other than 30 min? | Blocks API-layer window enforcement spec | **Closed** | Default 30 min confirmed. Configurable per factory; assume 0–120 min range pending factory config spec. |
| OQ-DR-02 | Are there stops the system marks as auto-classified (e.g., planned maintenance from ServiceNow SDM)? If so, can the operator override them? | Affects reclassification flow and Override Runtime rule | **Closed** | Yes — ServiceNow SDM planned activities may auto-classify stops. Operators can override; human input takes precedence in KPI output via the Override Runtime rule. |
| OQ-DR-03 | What is the operator-facing behaviour when a stop's MDC reason is unmapped (no option in the list)? | Affects UX flow and data completeness | **Closed** | Block classification. Stop remains Uncovered with a visual "unmapped" indicator. No free-text fallback. P-02c must map the reason code before the stop becomes classifiable. |
| OQ-DR-04 | Does the Ignore MDC Stop action require a mandatory reason/comment, or is a single tap sufficient? | Affects audit trail completeness | **Closed** | Single tap sufficient. No mandatory comment. Action is audited by user_id + timestamp. Optional comment field deferred to future enhancement. |
| OQ-DR-05 | For bulk reclassification, what defines "same type"? Same raw reason code, same duration band, or operator-selected grouping? | Affects bulk selection logic spec | **Closed** | Operator-selected grouping — any combination the operator selects. No system-enforced "same type" constraint. §3 updated accordingly. |
| OQ-DR-06 | Is the Long Lasting threshold global or configurable per machine/factory? Who sets it? | Affects configuration ownership | **Closed** | Configurable per factory (possibly per work center). Owned by factory admin (P-03a). Global default TBD in factory config spec. |

---

## Out of Scope
- Unified SDE/QSense timeline view — deferred; not a core requirement per session notes (May 2026).
- Free-text reason entry at runtime — downtime reason creation is release-gated; operators select from a managed list only.
- Self-correction for speed and reject entries — planned improvement, not in initial release.
- Daily shift locking (human-verified shift status) — planned future enhancement; design reclassification workflow to leave a clear integration point.
