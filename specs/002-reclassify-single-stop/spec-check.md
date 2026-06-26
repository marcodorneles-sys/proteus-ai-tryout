# Spec Check Report - Reclassify a Single Stop End-to-End (WI-05)

**Verdict:** REVIEW_RECOMMENDED ⚠  ·  **Overall:** 82/100  
**Subject:** spec.md (specs/002-reclassify-single-stop/spec.md)  ·  **Generated:** 2026-06-26T13:13:49Z  
**Gating reason:** Verdict band 4: overall score 60-84 and Major findings present.

## Scorecard

| Category | Weight | Score | Comment |
|---|---|---|---|
| Completeness / Coverage | 25 | 82 | Core WI-05 behavior is mostly covered, with partial carry-over on dependency/orchestration and role/window constraints. |
| Fidelity / Accuracy | 20 | 88 | Core intent is preserved (append-only human-input model, immutable machine stop), with minor under-specification areas. |
| Absence of invention | 15 | 80 | Success criteria include numeric targets not traceable to source inputs. |
| Consistency & terminology | 10 | 82 | Terms mostly align; verification is limited by missing canonical BR file and lack of BR references. |
| Traceability | 10 | 75 | Internal FR IDs exist, but source IDs/persona-ID mapping expected by governance is missing. |
| Testability & clarity | 10 | 90 | Stories and FRs are clear and testable with explicit edge/race/cancel behavior. |
| NFR & security/permission coverage | 10 | 70 | Category capped due to missing canonical NFR/security/business-rules inputs; retention detail is also omitted. |

## Coverage summary

15 requirements - 10 covered, 4 partial, 1 missing, 0 distorted, 1 invented, 0 inconsistent.

## Findings

### ⚠ Major
- **[R14 / F-01.1 / WI-05] Missing source-ID and persona-ID traceability**
  - Source: "Module -> Feature -> FR -> Work Item -> specify.md" and gap-check mapping required (docs/domain/PRD.md:55, docs/domain/PRD.md:65); "Every functional requirement and acceptance criterion must be traceable to at least one persona" (docs/domain/PERSONAS.md:4).
  - Spec: FR list exists but no source ID or persona-ID mapping (specs/002-reclassify-single-stop/spec.md:69, specs/002-reclassify-single-stop/spec.md:81).
  - Recommendation: Add explicit FR-to-source mapping (WI-05 ACs, F-01.1, persona IDs).

- **[R11] Dependency/orchestration constraints only implicit**
  - Source: "Reclassification Service ... calling Coverage Engine (01), Reason Eligibility Filter (02), and Writable Window Policy (03)" (.scratch/downtime-reclassification/issues/05-reclassify-single-stop.md:26).
  - Spec: Dependency appears as assumption only (specs/002-reclassify-single-stop/spec.md:104).
  - Recommendation: Elevate to explicit requirements or dependency constraints tied to blockers 01/02/03.

- **[R12 / F-07.3] Audit retention not carried into requirements**
  - Source: "traceable to user and timestamp (>=12-month retention, per F-07.3)" (docs/domain/features/downtime_reclassification.md:81); F-07.3 retention outcome (docs/domain/features/FEATURE_INVENTORY.md:128).
  - Spec: Actor and timestamp retained, but no retention duration requirement (specs/002-reclassify-single-stop/spec.md:81).
  - Recommendation: Add explicit retention requirement and audit immutability wording.

- **[R-INV-01] Untraceable numeric success targets introduced**
  - Source: WI/feature inputs define behavior, not numeric timing/percent targets for this slice (.scratch/downtime-reclassification/issues/05-reclassify-single-stop.md:35, docs/domain/features/downtime_reclassification.md:42).
  - Spec: "95% ... in 30 seconds" and multiple "100%" targets (specs/002-reclassify-single-stop/spec.md:95, specs/002-reclassify-single-stop/spec.md:99).
  - Recommendation: Either cite authoritative source for each metric or replace with source-traceable outcomes.

- **[Global context] Missing canonical BR/NFR/security inputs in workspace**
  - Source: PRD precedence references canonical cross-cutting rules and NFR/security sources (docs/domain/PRD.md:44, docs/domain/PRD.md:49).
  - Spec: No BR/NFR anchor references; full check is blocked by missing context files (specs/002-reclassify-single-stop/spec.md:65).
  - Recommendation: Add BUSINESS_RULES/NFR/SECURITY artifacts (or approved equivalent paths) and rerun spec-check.

### Minor
- **[R15] Operator-facing reason-display rule is implicit**
  - Source: "operator never sees raw PLC codes ... sees human-readable standardized English description" (docs/domain/features/downtime_reclassification.md:45).
  - Spec: Eligible reasons are required but PLC-code suppression is not explicit (specs/002-reclassify-single-stop/spec.md:70).
  - Recommendation: Add explicit wording to prevent UI interpretation drift.

## Recommended action

Do not move directly to implementation yet. Route this spec through BA/PO review to resolve the major traceability and governance gaps (ID mapping, dependency contract explicitness, audit retention carry-over, and untraceable success thresholds). After updates, rerun spec-check to confirm a clean PROCEED verdict.
