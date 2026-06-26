---
name: spec-check
description: Compare a generated specify.md against its source implementation ticket and feature requirements (and the global PRD, glossary, personas, and business rules), score how faithfully the spec reflects the intended requirements, and flag gaps, distortions, invented content, and inconsistencies that would distort implementation correctness. Produces a JSON scorecard plus a Markdown report and a verdict (PROCEED / REVIEW_RECOMMENDED / BLOCK) used to decide whether a BA/PO must review before implementation. Use after GitHub Spec Kit generates specify.md and before code generation. Runs non-interactively — asks no questions.
---

# Spec Check

Compare a generated `specify.md` against the requirements it was derived from, score the match, and emit a verdict that gates implementation.

This skill is **read-only and reporting-only**. It never edits `specify.md`, the ticket, or any requirements document. It never fixes issues — it surfaces them with evidence so a human can decide. It runs **non-interactively**: it asks no questions and makes no assumptions beyond what the inputs state; uncertainty is reported, not resolved.

## Inputs

Resolve these before scoring. Paths/refs are passed as arguments or taken from the pipeline context.

<inputs>
- **Subject under review:** `specify.md` (the GitHub Spec Kit output).
- **Direct parents (authoritative for intended behavior):**
  - The **implementation ticket / Work Item (WI-xxx)** the spec was generated from.
  - The **feature requirement document** that fed the ticket (in `features/<feature>/`).
- **Global context (authoritative for scope, terminology, and rules):**
  - `PRD.md` — scope, goals, non-goals (catches out-of-scope invention).
  - `GLOSSARY.md` — canonical terminology.
  - `PERSONAS.md` — personas and IDs (`P-xx`) for traceability.
  - Figma / design system references (if applicable) — for UI/UX consistency.
</inputs>

If any **direct parent** (specify.md, ticket, or feature doc) cannot be loaded, stop and emit verdict `BLOCKED_INPUT_MISSING` naming the missing input. Missing global-context files degrade the relevant category to a capped score and are recorded as findings — they do not abort the run.

## Process

### 1. Load inputs and establish precedence

Load every input above. When two authoritative sources conflict with each other, follow the source-of-truth precedence in `PRD.md` §1.3 (concern-specific canonical file wins) and record the conflict as a finding — do not silently pick one.

### 2. Build the requirement checklist

From the ticket and feature doc, extract an **atomic requirement checklist**: every acceptance criterion, functional requirement (`FR-xxx`), referenced business rule (`BR-xxx`), persona need (`P-xx`), NFR, and explicit scope boundary. Give each a stable local index (R1, R2, …) and carry its source ID and location. This checklist is the backbone for coverage scoring — every item must be accounted for in step 4.

### 3. Inventory what specify.md actually says

Independently list the behaviors, entities, rules, and constraints that `specify.md` specifies. This second list is the backbone for **invention** detection: anything here that does not trace to a checklist item or global context is a candidate invented/out-of-scope element.

### 4. Classify every item (evidence required)

Cross-reference the two lists. Classify each checklist item and each spec element as exactly one of:

<classifications>
- **Covered** — present in specify.md and faithful to intent.
- **Partial** — present but incomplete, weakened, or under-specified.
- **Missing** — required by inputs, absent from specify.md.
- **Distorted** — present but contradicts, alters, or misrepresents the intended requirement (wrong threshold, inverted rule, changed scope, altered persona/permission).
- **Invented** — present in specify.md with no basis in any input (hallucinated behavior, scope creep, fabricated AC, invented entity/rule).
- **Inconsistent** — violates glossary terms, persona IDs, business rules, or contradicts another part of specify.md.
</classifications>

**Every classification that is not "Covered" becomes a finding, and every finding MUST cite evidence** — a short quote and location from the source AND from specify.md (or "absent"). Do not record a finding you cannot cite. Do not infer intent the inputs do not state; if the spec is ambiguous, classify as Partial and say what is unclear, rather than guessing.

### 5. Score, rate severity, and decide the verdict

Score each rubric category 0–100, compute the weighted overall, assign a severity to each finding, then apply the gating rules to pick a verdict. Use the rubric, severity, and band definitions below exactly.

### 6. Emit outputs

Write the JSON scorecard and the Markdown report (schemas below). Output both; modify nothing else. The JSON is the contract your gating framework consumes; the Markdown is for the BA/PO.

---

## Scoring rubric

Per-category 0–100. Overall = weighted sum (weights tunable by the team; defaults below sum to 100).

<scoring-rubric>
| Category | Weight | Scores how well specify.md… |
|---|---|---|
| Completeness / Coverage | 25 | represents every requirement, AC, FR, and referenced BR from the ticket and feature doc (penalize Missing/Partial) |
| Fidelity / Accuracy | 20 | matches intent without distortion, weakening, or contradiction (penalize Distorted) |
| Absence of invention | 15 | adds nothing untraceable to the inputs; no scope creep or fabricated requirements (penalize Invented; 100 = nothing invented) |
| Consistency & terminology | 10 | uses canonical glossary terms, correct persona IDs, and conforms to business rules; no internal contradictions |
| Traceability | 10 | preserves and correctly maps source IDs (`FR-xxx`, `F-xx.y`, `BR-xxx`, `P-xx`, parent WI) |
| Testability & clarity | 10 | states requirements unambiguously and measurably enough to implement and test |
| NFR & security/permission coverage | 10 | carries over non-functional, security, data-residency, and RBAC/permission requirements |
</scoring-rubric>

Scoring guidance: start each category at 100 and deduct per finding by severity (Critical −40, Major −15, Minor −5 as a default; floor at 0). Be conservative — when unsure, deduct. Coverage and Absence-of-invention should reflect the proportion of items correctly handled, not a vibe.

---

## Severity levels

<severity-levels>
- **Critical** — would cause incorrect implementation if not fixed: a Missing or Distorted core behavior, an Invented in-scope behavior, a violated business rule, or a wrong NFR/permission/data-residency constraint.
- **Major** — a meaningful gap, ambiguity, or inconsistency that needs BA/PO input but does not by itself corrupt core behavior.
- **Minor** — low-risk: cosmetic wording, a non-canonical term with obvious meaning, or a small clarity issue.
</severity-levels>

---

## Verdict bands

<verdict-bands>
Apply in order; the first matching rule wins.

1. Any required direct-parent input missing → **BLOCKED_INPUT_MISSING**.
2. Any **Critical** finding → **BLOCK** (regardless of score).
3. Overall score **< 60** → **BLOCK**.
4. Overall score **60–84**, or any **Major** finding → **REVIEW_RECOMMENDED** (route to BA/PO).
5. Overall score **≥ 85** and no Critical/Major findings → **PROCEED**.

Bands and weights are defaults — keep them in one place so the team can calibrate without editing the rest of the skill.
</verdict-bands>

---

## JSON output schema

<json-output-schema>
```json
{
  "skill": "spec-check",
  "version": "1.0",
  "generated_at": "<ISO-8601>",
  "subject": { "specify_md": "<path-or-ref>" },
  "inputs": {
    "ticket": "<WI-xxx ref/path>",
    "feature_doc": "<path>",
    "prd": "<path|null>",
    "glossary": "<path|null>",
    "personas": "<path|null>",
    "business_rules": "<path|null>",
    "feature_inventory": "<path|null>"
  },
  "coverage": {
    "total_requirements": 0,
    "covered": 0,
    "partial": 0,
    "missing": 0,
    "distorted": 0,
    "invented": 0,
    "inconsistent": 0
  },
  "categories": [
    {
      "name": "Completeness / Coverage",
      "weight": 25,
      "score": 0,
      "comment": "<one-paragraph rationale citing the main drivers>"
    }
    // …one object per rubric category
  ],
  "overall_score": 0,
  "verdict": "PROCEED | REVIEW_RECOMMENDED | BLOCK | BLOCKED_INPUT_MISSING",
  "gating_reason": "<which band/rule produced the verdict>",
  "findings": [
    {
      "id": "F-001",
      "category": "Fidelity / Accuracy",
      "type": "missing | partial | distorted | invented | inconsistent",
      "severity": "critical | major | minor",
      "requirement_ref": "<R3 / FR-012 / BR-004 / P-01 / ticket-AC2>",
      "evidence_source": "<short quote + location in ticket/feature/PRD/glossary>",
      "evidence_spec": "<short quote + location in specify.md, or 'absent'>",
      "comment": "<what is wrong and why it matters for implementation>",
      "recommendation": "<what a BA/PO should add, correct, or remove>"
    }
  ]
}
```
</json-output-schema>

Every entry in `findings[]` must carry non-empty `evidence_source` and `evidence_spec`. Each `categories[].comment` is the per-score comment the requester asked for — concrete and citing the findings that drove the score.

---

## Markdown report template

<markdown-report-template>
```markdown
# Spec Check Report — <feature / WI-xxx>

**Verdict:** <PROCEED ✅ | REVIEW_RECOMMENDED ⚠ | BLOCK ⛔>  ·  **Overall:** <score>/100
**Subject:** specify.md (<ref>)  ·  **Generated:** <timestamp>
**Gating reason:** <rule that decided the verdict>

## Scorecard

| Category | Weight | Score | Comment |
|---|---|---|---|
| Completeness / Coverage | 25 | <n> | <comment> |
| Fidelity / Accuracy | 20 | <n> | <comment> |
| Absence of invention | 15 | <n> | <comment> |
| Consistency & terminology | 10 | <n> | <comment> |
| Traceability | 10 | <n> | <comment> |
| Testability & clarity | 10 | <n> | <comment> |
| NFR & security/permission coverage | 10 | <n> | <comment> |

## Coverage summary

<total> requirements — <covered> covered, <partial> partial, <missing> missing, <distorted> distorted, <invented> invented, <inconsistent> inconsistent.

## Findings

### ⛔ Critical
- **[<ref>] <title>** — <comment>
  - Source: "<quote>" (<location>)
  - Spec: "<quote or 'absent'>" (<location>)
  - Recommendation: <action>

### ⚠ Major
- …

### Minor
- …

## Recommended action
<One paragraph: proceed, or what the BA/PO must resolve before implementation.>
```
</markdown-report-template>

---

## Reliability principles (apply throughout)

<principles>
- **Evidence or it didn't happen.** No finding without a citation from both sides. This is the main guard against the checker inventing its own issues.
- **Report, never fix.** Do not edit specify.md or requirements. Output analysis only.
- **Conservative scoring.** When intent is ambiguous, prefer Partial + a finding over assuming Covered. Deduct when unsure.
- **Deterministic rubric.** Use the fixed categories, weights, severities, and bands above so two runs on the same inputs land on the same verdict. Keep tunables in the rubric/bands blocks only.
- **Separate the two passes.** Build the requirement checklist (step 2) and the spec inventory (step 3) independently before cross-referencing, so gaps and inventions are both caught.
- **Don't reward verbosity.** Extra detail in specify.md that isn't traceable to inputs lowers the Absence-of-invention score; it is not a bonus.
- **Stay in lane.** This skill judges fidelity of specify.md to its inputs. It does not judge whether the original requirements are themselves good — that is the BA/PO's job upstream.
</principles>

## Process-reliability recommendations (for the team's framework)

<framework-notes>
- **Calibrate the bands.** Run spec-check on a labelled set of past specs (known good / known bad) and tune weights and band thresholds until the verdict matches human judgment before trusting it to gate.
- **Spot-check PROCEED verdicts.** Have a BA review a random sample of auto-PROCEED specs periodically to detect drift; feed misses back into calibration.
- **Always human-review BLOCK and REVIEW_RECOMMENDED.** Never auto-rework on a BLOCK without a person confirming the findings.
- **Persist every result.** Store the JSON keyed by WI + spec hash, so re-runs are comparable and you can trend spec quality over time and by feature/module.
- **Version the rubric.** Bump `version` when weights/bands/categories change; old results stay interpretable.
- **Close the loop with upstream skills.** Recurring Missing/Distorted findings usually mean the feature doc or ticket (from /to-prd, /to-issues) was thin — fix the source, not just the spec.
- **Idempotent + non-interactive.** Same inputs → same output; no prompts. This is what makes it safe to run automatically on every spec.
</framework-notes>
