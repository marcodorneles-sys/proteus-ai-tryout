---
name: spec-check
description: Compare a set of candidate output artifacts against the source input artifacts they were derived from, score how faithfully the outputs reflect the intended requirements, and flag gaps, distortions, invented content, inconsistencies, and ambiguities that would distort implementation correctness. Produces a JSON scorecard plus a Markdown report and a verdict (PROCEED / REVIEW_RECOMMENDED / BLOCK) used to decide whether a human reviewer must review before implementation. Commonly used after GitHub Spec Kit generates files such as spec.md and requirements.md, but not limited to that workflow.
---

# Spec Check

Compare one or more candidate output artifacts against the source input artifacts they were derived from, score the match, and emit a verdict that gates implementation readiness.

This skill is **read-only and reporting-only**. It never edits candidate output artifacts, source input artifacts, or reference/context documents. It never fixes issues — it surfaces them with evidence so a human can decide.

The skill makes no assumptions beyond what the provided or explicitly referenced inputs state. Uncertainty is reported, not resolved.

When used manually, if the prompt does not clearly identify which artifacts are **source inputs** and which artifacts are **candidate outputs**, ask for confirmation before scoring. When used in pipeline/non-interactive mode, do not ask questions; stop and emit the appropriate blocked verdict.

---

## Inputs

Resolve these before scoring. Paths/refs are passed as arguments, provided in the prompt, discovered through explicit references from a root input artifact, or taken from pipeline context.

### Required input groups

* **Source input artifact set**
  The authoritative artifacts that define the intended behavior, requirements, acceptance criteria, business rules, constraints, terminology, non-functional requirements, and scope boundaries.

* **Candidate output artifact set**
  The generated or proposed artifacts under review. These are expected to faithfully represent the source input artifact set.

### Optional inputs

* **Reference/context artifacts**
  Additional artifacts that constrain interpretation, terminology, user/persona language, business rules, design consistency, architecture constraints, security expectations, or non-functional requirements.

* **Artifact mapping**
  Optional instructions explaining which source artifacts should be covered by which candidate output artifacts. If no mapping is provided, assess whether the candidate output artifact set as a whole covers the source input artifact set.

* **Precedence rules**
  Optional rules for resolving conflicts between source input artifacts or between source and context artifacts.

* **Execution mode**

  * `manual` — ask for confirmation if source/output grouping is ambiguous.
  * `pipeline` or `non_interactive` — never ask questions; emit a blocked verdict if required inputs are missing or ambiguous.

* **Output location**
  Optional destination for the JSON scorecard and Markdown report.
  
---

## Input expansion rule

A source input artifact may be provided directly as a list or indirectly as a root source artifact.

When a root source artifact explicitly references other requirement, rule, glossary, persona, design, architecture, clarification, or context artifacts, load those referenced artifacts and include them in the expanded source input set.

Only follow explicit references, paths, links, or clearly named artifacts. Do not search for additional files based on assumptions, naming conventions, expected project structure, or prior knowledge unless the user explicitly asks for that behavior.

Record the final expanded source input set in the report.

Candidate output artifacts should normally be provided explicitly. If the prompt or folder context makes the candidate output set unambiguous, proceed. If not clear:

* In `manual` mode, ask for confirmation.
* In `pipeline` / `non_interactive` mode, stop and emit verdict `BLOCKED_INPUT_AMBIGUOUS`.

If any required source input artifact or candidate output artifact cannot be loaded:

* In `manual` mode, ask for correction or confirmation.
* In `pipeline` / `non_interactive` mode, stop and emit verdict `BLOCKED_INPUT_MISSING`, naming the missing input.

Missing optional context/reference artifacts degrade only the relevant scoring category and are recorded as findings or limitations. They do not abort the run unless the artifact was explicitly marked as required.

---

## Process

### 1. Load inputs and establish scope

Load the source input artifact set, candidate output artifact set, and any explicitly referenced or provided context/reference artifacts.

Establish:

* Which artifacts are authoritative source inputs.
* Which artifacts are candidate outputs under review.
* Which artifacts are context/reference only.
* Whether any artifact mapping was provided.
* Whether precedence rules were provided.
* Which files were discovered through explicit reference expansion.

When two authoritative sources conflict:

* If explicit precedence rules are provided, apply them and record the conflict as a finding.
* If no precedence rules are provided, record the conflict as a finding and do not silently choose one source as correct.

Do not judge whether the original source requirements are good, complete, or well-written. This skill judges whether the candidate outputs faithfully represent the provided inputs.

---

### 2. Build the source requirement checklist

From the source input artifact set and any authoritative context artifacts, extract an **atomic requirement checklist**.

Include every:

* Acceptance criterion.
* Functional requirement.
* Explicit user need or persona/user expectation.
* Business rule.
* Non-functional requirement.
* Security, permission, access-control, privacy, data-residency, or compliance requirement.
* Explicit constraint.
* Explicit scope boundary.
* Explicit out-of-scope statement.
* Explicit terminology or domain rule that affects implementation correctness.
* Clarification answer that changes, narrows, or confirms intended behavior.

Give each item a stable local index (`R1`, `R2`, …) and carry its source ID and location when present.

Preserve any source identifiers exactly as written when they exist, such as requirement IDs, work item IDs, acceptance criterion IDs, rule IDs, persona IDs, section numbers, or document references. Do not assume any specific ID naming convention.

This checklist is the backbone for coverage scoring. Every checklist item must be accounted for in step 4.

---

### 3. Inventory what the candidate outputs actually say

Independently list the behaviors, entities, rules, constraints, acceptance criteria, terminology, assumptions, permissions, non-functional requirements, and scope statements specified by the candidate output artifact set.

Track the location of each candidate output element, including which output artifact contains it.

This second list is the backbone for invention and inconsistency detection. Anything in the candidate output artifact set that does not trace to a source checklist item or an allowed context/reference artifact is a candidate invented or out-of-scope element.

Also check for contradictions between candidate output artifacts. For example, if one generated artifact states a rule differently from another generated artifact, record this as an inconsistency.

---

### 4. Classify every item, with evidence required

Cross-reference the source requirement checklist and the candidate output inventory.

Classify each source checklist item and each candidate output element as exactly one of:

### Classifications

* **Covered** — present in the candidate output artifact set and faithful to intent.
* **Partial** — present but incomplete, weakened, ambiguous, under-specified, or distributed across outputs in a way that loses required meaning.
* **Missing** — required by the source input artifact set but absent from the candidate output artifact set.
* **Distorted** — present but contradicts, alters, narrows, expands, weakens, or misrepresents the intended requirement.
* **Invented** — present in the candidate output artifact set with no basis in any source input or allowed context/reference artifact.
* **Inconsistent** — contradicts another candidate output artifact, violates provided terminology or business rules, conflicts with source constraints, or introduces internal contradiction.

Every classification that is not `Covered` becomes a finding.

Every finding MUST cite evidence:

* `evidence_source`: short quote and location from the relevant source input or context/reference artifact.
* `evidence_output`: short quote and location from the candidate output artifact, or `absent`.

Do not record a finding you cannot cite.

Do not infer intent the inputs do not state. If the candidate output is ambiguous, classify as `Partial` and say what is unclear rather than guessing.

---

### 5. Score, rate severity, and decide the verdict

Score each rubric category 0–100, compute the weighted overall, assign a severity to each finding, then apply the gating rules to pick a verdict.

Use the rubric, severity, and band definitions below exactly unless alternative weights or bands are explicitly provided as inputs.

---

### 6. Emit outputs

Write the JSON scorecard and the Markdown report.

Output both.

The JSON is the machine-readable contract consumed by a gating framework or future orchestrator.

The Markdown report is for human review.

Modify nothing else.

---

### 7. Save the outputs

If an output location is provided, save both outputs there:

* `spec-check.json` — the machine-readable scorecard.
* `spec-check.md` — the human-readable report.

If no output location is provided, return both outputs in the response or to the calling environment without writing files.

Re-running the check with the same inputs should be idempotent. If writing files, overwrite only these two output files. Never modify candidate output artifacts, source input artifacts, or reference/context documents.

---

## Scoring rubric

Per-category 0–100.

Overall = weighted sum.

Weights are tunable by the team; defaults below sum to 100.

### Scoring-rubric

| Category                           | Weight | Scores how well the candidate output artifact set…                                                                                                                                             |
| ---------------------------------- | -----: | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Completeness / Coverage            |     25 | Represents every requirement, acceptance criterion, rule, constraint, and scope boundary from the source input artifact set. Penalize Missing and Partial.                                     |
| Fidelity / Accuracy                |     20 | Matches source intent without distortion, weakening, contradiction, or unsupported interpretation. Penalize Distorted.                                                                         |
| Absence of invention               |     15 | Adds nothing untraceable to the source inputs or allowed context; no scope creep, fabricated requirements, invented rules, or unsupported behavior. Penalize Invented. 100 = nothing invented. |
| Consistency & terminology          |     10 | Uses provided terminology consistently, follows provided domain language and business rules, and avoids contradictions across candidate output artifacts.                                      |
| Traceability                       |     10 | Preserves and correctly maps source identifiers, requirement IDs, acceptance criteria, rule IDs, user/persona references, parent artifact links, and local requirement indexes when present.   |
| Testability & clarity              |     10 | States requirements unambiguously and measurably enough to implement and test.                                                                                                                 |
| NFR & security/permission coverage |     10 | Carries over non-functional, security, access-control, privacy, data-residency, compliance, and permission requirements when present in the source inputs.                                     |

Scoring guidance:

* Start each category at 100 and deduct per finding by severity.
* Default deductions: Critical −40, Major −15, Minor −5.
* Floor each category at 0.
* Be conservative. When unsure, deduct.
* Coverage and Absence of invention should reflect the proportion of items correctly handled, not a general impression.
* Do not reward verbosity. Extra detail that is not traceable to source inputs or allowed context lowers the Absence of invention score.

---

## Severity levels

* **Critical** — would likely cause incorrect implementation if not fixed. Examples: missing or distorted core behavior, invented in-scope behavior, violated business rule, wrong NFR, wrong permission/security rule, wrong data-residency/compliance constraint, or contradiction that changes implementation meaning.

* **Major** — meaningful gap, ambiguity, unsupported addition, inconsistency, or traceability failure that needs human review but does not by itself corrupt core behavior.

* **Minor** — low-risk issue such as cosmetic wording, non-canonical terminology with obvious meaning, minor traceability weakness, or small clarity issue.

---

## Verdict bands

Apply in order; the first matching rule wins.

1. Required source input or candidate output missing → **BLOCKED_INPUT_MISSING**.
2. Source/output grouping ambiguous in pipeline/non-interactive mode → **BLOCKED_INPUT_AMBIGUOUS**.
3. Any **Critical** finding → **BLOCK**.
4. Overall score **< 60** → **BLOCK**.
5. Overall score **60–84**, or any **Major** finding → **REVIEW_RECOMMENDED**.
6. Overall score **≥ 85** and no Critical/Major findings → **PROCEED**.

Bands and weights are defaults. Keep them in one place so the team can calibrate without editing the rest of the skill.

---

## JSON output schema

<!-- <json-output-schema> -->

```json
{
  "skill": "spec-check",
  "version": "1.1",
  "generated_at": "<ISO-8601>",
  "execution_mode": "manual | pipeline | non_interactive",
  "subject": {
    "candidate_outputs": [
      {
        "path_or_ref": "<path-or-ref>",
        "artifact_type": "<optional type, e.g. spec, requirements, design, test-plan, unknown>"
      }
    ]
  },
  "inputs": {
    "source_inputs": [
      {
        "path_or_ref": "<path-or-ref>",
        "role": "root | referenced | explicit",
        "loaded": true
      }
    ],
    "context_artifacts": [
      {
        "path_or_ref": "<path-or-ref|null>",
        "role": "terminology | business_rules | personas | design | architecture | security | other",
        "loaded": true
      }
    ],
    "artifact_mapping": "<provided mapping or null>",
    "precedence_rules": "<provided precedence rules or null>",
    "output_location": "<path-or-null>"
  },
  "input_expansion": {
    "root_inputs": [
      "<path-or-ref>"
    ],
    "explicitly_referenced_inputs_loaded": [
      "<path-or-ref>"
    ],
    "references_not_loaded": [
      {
        "path_or_ref": "<path-or-ref>",
        "reason": "<missing | inaccessible | ambiguous | unsupported>"
      }
    ]
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
    },
    {
      "name": "Fidelity / Accuracy",
      "weight": 20,
      "score": 0,
      "comment": "<one-paragraph rationale citing the main drivers>"
    },
    {
      "name": "Absence of invention",
      "weight": 15,
      "score": 0,
      "comment": "<one-paragraph rationale citing the main drivers>"
    },
    {
      "name": "Consistency & terminology",
      "weight": 10,
      "score": 0,
      "comment": "<one-paragraph rationale citing the main drivers>"
    },
    {
      "name": "Traceability",
      "weight": 10,
      "score": 0,
      "comment": "<one-paragraph rationale citing the main drivers>"
    },
    {
      "name": "Testability & clarity",
      "weight": 10,
      "score": 0,
      "comment": "<one-paragraph rationale citing the main drivers>"
    },
    {
      "name": "NFR & security/permission coverage",
      "weight": 10,
      "score": 0,
      "comment": "<one-paragraph rationale citing the main drivers>"
    }
  ],
  "overall_score": 0,
  "verdict": "PROCEED | REVIEW_RECOMMENDED | BLOCK | BLOCKED_INPUT_MISSING | BLOCKED_INPUT_AMBIGUOUS",
  "gating_reason": "<which band/rule produced the verdict>",
  "findings": [
    {
      "id": "F-001",
      "category": "Fidelity / Accuracy",
      "type": "missing | partial | distorted | invented | inconsistent",
      "severity": "critical | major | minor",
      "requirement_ref": "<R3 / source requirement ID / AC ID / rule ID / section ref>",
      "source_artifact": "<path-or-ref>",
      "candidate_output_artifact": "<path-or-ref or 'all outputs' or 'absent'>",
      "evidence_source": "<short quote + location in source input or context/reference artifact>",
      "evidence_output": "<short quote + location in candidate output artifact, or 'absent'>",
      "comment": "<what is wrong and why it matters for implementation>",
      "recommendation": "<what a human reviewer should add, correct, clarify, or remove>"
    }
  ],
  "limitations": [
    {
      "id": "L-001",
      "comment": "<missing optional context, inaccessible reference, ambiguous source conflict, or other limitation>"
    }
  ]
}
```

<!-- </json-output-schema> -->

Every entry in `findings[]` must carry non-empty `evidence_source` and `evidence_output`.

Each `categories[].comment` must be concrete and must cite the findings that drove the score.

---

## Markdown report template

<!-- <markdown-report-template> -->

```markdown
# Spec Check Report — <artifact set / feature / work item / run label>

**Verdict:** <PROCEED ✅ | REVIEW_RECOMMENDED ⚠ | BLOCK ⛔ | BLOCKED_INPUT_MISSING ⛔ | BLOCKED_INPUT_AMBIGUOUS ⛔> · **Overall:** <score>/100  
**Candidate outputs:** <list of candidate output refs>  
**Generated:** <timestamp>  
**Gating reason:** <rule that decided the verdict>

## Inputs reviewed

### Source input artifacts

| Artifact | Role | Status |
|---|---|---|
| <path/ref> | <root / explicit / referenced> | <loaded / missing / inaccessible> |

### Candidate output artifacts

| Artifact | Type | Status |
|---|---|---|
| <path/ref> | <type or unknown> | <loaded / missing / inaccessible> |

### Context/reference artifacts

| Artifact | Role | Status |
|---|---|---|
| <path/ref> | <terminology / business rules / personas / design / architecture / security / other> | <loaded / missing / inaccessible> |

## Scorecard

| Category | Weight | Score | Comment |
|---|---:|---:|---|
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
  - Source: "<quote>" (<source artifact + location>)
  - Output: "<quote or 'absent'>" (<candidate output artifact + location>)
  - Recommendation: <action>

### ⚠ Major

- …

### Minor

- …

## Limitations

- <missing optional context, inaccessible explicit reference, unresolved source conflict, or other limitation>

## Recommended action

<One paragraph: proceed, or what a human reviewer must resolve before implementation.>
```

<!-- </markdown-report-template> -->

---

## Reliability principles

<!-- <principles> -->

* **Evidence or it did not happen.** No finding without a citation from both source and candidate output evidence. This is the main guard against the checker inventing its own issues.

* **Report, never fix.** Do not edit candidate output artifacts, source input artifacts, or reference/context artifacts. Output analysis only.

* **Explicit source/output separation.** The skill must know which artifacts are source inputs and which artifacts are candidate outputs. If unclear in manual mode, ask for confirmation. If unclear in pipeline/non-interactive mode, block the run.

* **Follow explicit references, not implicit assumptions.** A root source input may expand into referenced files, but only when those references are explicit. Do not load unrelated files based on naming conventions, expected folder structure, or prior workflow knowledge.

* **Validate the output set as a whole.** A requirement may be covered by one or more candidate output artifacts. Do not require every requirement to appear in a specific output file unless an artifact mapping says so.

* **Check cross-output consistency.** Candidate output artifacts must not contradict each other. A contradiction between candidate outputs is an inconsistency finding.

* **Conservative scoring.** When intent is ambiguous, prefer Partial + a finding over assuming Covered. Deduct when unsure.

* **Deterministic rubric.** Use the fixed categories, weights, severities, and bands above so two runs on the same inputs land on the same verdict. Keep tunables in the rubric/bands blocks only.

* **Separate the two passes.** Build the source requirement checklist and the candidate output inventory independently before cross-referencing, so gaps and inventions are both caught.

* **Do not reward verbosity.** Extra detail in candidate outputs that is not traceable to source inputs or allowed context lowers the Absence of invention score; it is not a bonus.

* **Stay in lane.** This skill judges fidelity of candidate outputs to provided inputs. It does not judge whether the original source requirements are themselves good — that is the responsibility of the upstream human or process.

<!-- </principles> -->

---

## Process-reliability recommendations

### Framework notes

<!-- <framework-notes> -->

* **Calibrate the bands.** Run this skill on a labelled set of past outputs (known good / known bad) and tune weights and band thresholds until the verdict matches human judgment before trusting it to gate.

* **Spot-check PROCEED verdicts.** Have a human reviewer check a random sample of auto-PROCEED runs periodically to detect drift; feed misses back into calibration.

* **Always human-review BLOCK and REVIEW_RECOMMENDED.** Never auto-rework on a BLOCK without a person confirming the findings.

* **Persist every result.** Store the JSON keyed by source input set + candidate output set + output hash, so re-runs are comparable and spec quality can be trended over time.

* **Version the rubric.** Bump `version` when weights, bands, categories, or input semantics change; old results stay interpretable.

* **Close the loop with upstream generation.** Recurring Missing, Distorted, or Invented findings may indicate that the source input artifacts are thin, ambiguous, or that the generation prompt/skill is losing information. Fix the appropriate upstream source or generation step rather than only patching the candidate outputs.

* **Prepare for orchestration.** In manual mode, the skill may ask to confirm source/output grouping. In pipeline/non-interactive mode, the orchestrator must provide the source input artifact set, candidate output artifact set, optional mapping, and output location explicitly.

<!-- </framework-notes> -->
