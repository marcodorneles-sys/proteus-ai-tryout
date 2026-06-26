Status: ready-for-agent

# PRD: Downtime Reclassification

Feature: Downtime Reclassification (Smart Downtime Entry module)
Personas: P-01 Shop-floor Operator (owner), P-02 Process Lead (reuses capability)
Glossary: see [CONTEXT.md](../../CONTEXT.md)
ADRs: [0001 Core Technology & Runtime Stack](../../docs/adr/0001-core-technology-runtime-stack.md), [0002 Long Lasting Downtimes Are Materialized Per Shift](../../docs/adr/0002-long-lasting-downtime-per-shift-materialization.md)

## Problem Statement

A shop-floor operator stands at a noisy kiosk watching a timeline of their
machine. MDC reports raw **Machine Stops** as red segments, but those stops
carry no human meaning — the system knows the machine stopped, not *why*. The
operator needs to assign a business root cause to that red, fast, between
production tasks, without manual timestamps or long dropdowns. Today (legacy
GSCA Machinery) this is slow, has no timeline, no self-correction, and no way to
handle a machine that is broken with an unknown fix time. The operator cannot
keep production data accurate without friction-free reclassification.

## Solution

From the operator's perspective: on the SDE Visual Timeline, the red showing
through (Uncovered or Partially Covered **Machine Stops**) is the work list. The
operator selects that red, picks a standardized **Reason** from the downtime
catalog, and the system draws an operator-owned **Downtime** over it. The catalog
only ever offers Reasons that are valid for the selected span, so the operator
cannot pick something the handbook forbids. When a machine is broken with an
unknown fix time, the operator flags the **Downtime** as **Long Lasting** and it
keeps extending as MDC reports more, automatically respecting shift ownership.
Operators can correct or void what they entered, and can classify several stops
at once. They never touch the raw machine data — they only add the human layer
on top, and only within the window they are allowed to write.

## User Stories

1. As a shop-floor operator, I want to see the red Machine Stops that still need a Reason, so that I know exactly what is left to classify.
2. As a shop-floor operator, I want Covered stops to visually drop off my work list, so that I can focus only on Uncovered and Partially Covered red.
3. As a shop-floor operator, I want to select an Uncovered Machine Stop and assign a Reason, so that the machine's stop has a business root cause.
4. As a shop-floor operator, I want assigning a Reason to create a Downtime drawn over the red, so that I can see at a glance what has been classified.
5. As a shop-floor operator, I want to never be able to edit or delete the underlying Machine Stop, so that the raw MDC fact stays trustworthy.
6. As a shop-floor operator, I want to classify the remaining red of a Partially Covered Machine Stop, so that the whole stop ends up Covered.
7. As a shop-floor operator, I want a single Downtime to be able to span several adjacent Machine Stops, so that I can classify a connected event in one go.
8. As a shop-floor operator, I want the catalog to only show Reasons valid for the span I selected, so that I cannot pick a forbidden Reason.
9. As a shop-floor operator, I want Reasons whose minimum duration exceeds my selected span to be hidden, so that I do not assign a breakdown to a stop that is too short.
10. As a shop-floor operator, I want a breakdown Reason to require at least its minimum duration (e.g. 10 minutes), so that the handbook rule is enforced automatically.
11. As a shop-floor operator, I want some Reasons to allow a 0 or 5 minute minimum, so that short stops can still be classified where the rules permit.
12. As a shop-floor operator, I want a Downtime to cover green runtime only when the chosen Reason explicitly allows overriding runtime, so that I do not accidentally erase valid run time.
13. As a shop-floor operator, I want to flag a Downtime as Long Lasting when a machine is broken with an unknown fix time, so that I do not have to keep re-entering it.
14. As a shop-floor operator, I want a Long Lasting Downtime rendered as an extended arrow, so that I can see at a glance it is open-ended.
15. As a shop-floor operator, I want a Long Lasting Downtime to extend only up to the latest MDC has reported, so that the timeline never invents runtime in the unknown region.
16. As a shop-floor operator, I want a Long Lasting Downtime to stop extending when MDC reports the machine resumed or the stop ended, so that it reflects reality.
17. As a shop-floor operator, I want a Long Lasting Downtime to close at my shift boundary and a new one to open for the next shift, so that each shift owns its own data.
18. As a shop-floor operator, I want to select several Uncovered or Partially Covered Machine Stops and apply one Reason to all at once, so that I can reduce repetitive entry.
19. As a shop-floor operator, I want a bulk action to only offer Reasons valid for every selected stop, so that the one Reason I pick is legal for all of them.
20. As a shop-floor operator, I want a Downtime per selected stop produced by a bulk action, so that each stop is individually classified.
21. As a shop-floor operator, I want to change the Reason or flags on a Downtime I created, so that I can correct a mistake.
22. As a shop-floor operator, I want a correction to create a new Human-input Record and the timeline to show only the latest, so that the current state is always unambiguous.
23. As a shop-floor operator, I want to void a Downtime and have the red Machine Stop reappear as Uncovered, so that I can undo a misclassification.
24. As a shop-floor operator, I want all my edits limited to my current shift, so that I do not accidentally change another shift's data.
25. As a shop-floor operator, I want a brief window after my shift ends to finalize the previous shift, so that I can finish entries I did not complete in time.
26. As a shop-floor operator, I want writes blocked once the handover window closes, so that locked data stays final.
27. As a shop-floor operator, I want bringing an earlier shift onto the screen to stay read-only, so that viewing history never risks editing it.
28. As a process lead, I want to reclassify downtimes across multiple lines through the office review surface, so that I can correct and complete operator data.
29. As a process lead, I want to write to the current and previous shift within my wider window, so that I can fix data the operator missed.
30. As a process lead, I want the same reclassification rules (Reasons, minimums, override) to apply to me, so that corrections stay consistent with operator entries.
31. As a shop-floor operator, I want to mark a Machine Stop as a false trigger (Ignore) so that it is excluded from KPI output without my needing to create a Downtime over it.
32. As a shop-floor operator, I want an Ignored Machine Stop to disappear from my outstanding work list, so that I am not asked to classify a stop that never should have counted.
33. As a shop-floor operator, I want a Machine Stop whose MDC reason code is unmapped to show a clear "unmapped" indicator and be blocked from classification, so that I know the Mapping Responsible needs to configure it first before I can act on it.
34. As a shop-floor operator, I want a Machine Stop that was auto-classified by a planned activity from ServiceNow SDM to be overridable by a normal Reclassification, so that I can correct it if the planned activity assignment is wrong.
35. As a production analyst, I want to reclassify within the open KPI period, so that I can correct data before the period locks.

## Implementation Decisions

Modules to be built (within the Smart Downtime Entry module of the modular
monolith; React frontend, .NET/ASP.NET Core backend, Azure SQL per ADR 0001):

- **Coverage Engine** (deep, pure). Input: the set of `Machine Stops` and the
  effective `Downtimes` over a window. Output: the Uncovered / Partially Covered /
  Covered red regions. Coverage is computed against red only; green-covering is an
  explicit exception, not part of the to-do list. Cardinality is one `Downtime`
  to many `Machine Stops`. Ignored `Machine Stops` are excluded from the
  outstanding work list and from KPI output; the Coverage Engine treats an Ignored
  stop as resolved (it will not show as Uncovered).
- **Reason Eligibility Filter** (deep, pure). Input: a selected span (or a set of
  spans for bulk) plus the downtime catalog with per-`Reason` rules (minimum
  duration, can-override-runtime). Output: the eligible `Reasons`. For bulk, the
  result is the **intersection** of each selected stop's eligible set. Minimum
  duration is measured against the `Downtime` span being created and applied as a
  catalog filter at selection time, never as a post-hoc validation error. For
  unmapped stops, the filter returns an empty eligible set, which blocks
  classification at the API layer (not just the UI). The `Long Lasting` duration
  threshold is configurable per factory (possibly per work center), owned by
  P-03a, and read from factory configuration by this module.
- **Long Lasting Extension Resolver** (deep, pure). Input: a `Long Lasting`
  `Downtime`, shift boundaries, the `MDC Sync Watermark`, and the machine-state
  stream. Output: the per-shift materialized `Downtime` segments and the point
  where extension stops. Encodes ADR 0002: extend up to the lesser of the shift
  end and the watermark; close at the shift boundary and open a new segment for
  the next shift; stop when the machine resumes or the stop ends.
- **Writable Window Policy** (deep, pure). Input: current time, shift schedule,
  `Shift Handover Window` configuration, and a target timestamp. Output: a
  may-write decision. Encodes "current shift always writable; previous shift
  writable only while its handover window is open; visibility never implies
  writability." Default handover window: 30 min; configurable per factory (P-03a).
  P-02a has a wider window (current + previous shift, factory-configurable).
  P-02b (Production Analyst) can write within the open KPI Period.
- **Reclassification Service** (thin, side-effecting). Orchestrates the four pure
  modules to handle reclassify, bulk reclassify, correct, void, and ignore commands.
  It validates against the Writable Window Policy and Reason Eligibility Filter,
  then creates `Downtimes`, appends `Human-input Records`, performs voids, and
  records Ignore actions. It never mutates a `Machine Stop`.

Data / state decisions:

- A `Downtime` is a separate overlay record over the read-only `Machine Stop`
  base; the base is never edited or deleted by these workflows.
- Edits are append-only as `Human-input Records`; rendering is as-now (latest
  effective record per `Downtime`). Voided and superseded versions are retained
  but not drawn.
- The Ignore action writes a `Human-input Record` with `action_type=ignore`
  directly against the `Machine Stop` (no `Downtime` is created). The KPI engine
  excludes Ignored stops from recalculation. The `Machine Stop` is retained.
- ServiceNow SDM planned activities may produce a system-generated `Human-input
  Record` (auto-classification). Operators and Process Leads can override this
  with a normal Reclassification — the human override takes precedence in KPI
  output via the Override Runtime rule.
- Unmapped `Machine Stops` (no SmartMDC classification available) are surfaced
  as Uncovered with a visual "unmapped" indicator. Classification is blocked at
  the API layer until the Mapping Responsible (P-02c) maps the reason code.
- `Long Lasting` `Downtimes` are materialized as one record per shift (ADR 0002),
  not a single open-ended record.
- Reclassification never writes past the `MDC Sync Watermark` into the unknown
  region.
- Bulk Reclassification is operator-selected grouping — no system-enforced
  "same type" constraint. Each selected stop receives its own `Downtime`; the
  Reason Eligibility Filter returns the intersection of eligible Reasons across
  all selected stops.

## Testing Decisions

A good test here asserts **external behavior through a module's interface**, not
its internals: feed inputs (intervals, catalog rules, watermarks, shift configs)
and assert the returned regions / eligible Reasons / materialized segments /
write decision. The four pure modules are designed precisely so this is possible
without infrastructure.

Modules to be tested (per maintainer selection):

- **Coverage Engine** — table-driven interval scenarios: fully uncovered stop;
  partially covered stop; fully covered stop; one `Downtime` spanning several
  `Machine Stops` and the green between them; overlapping `Downtimes`.
- **Reason Eligibility Filter** — span just below / at / above each minimum;
  override-runtime on vs off against green-covering spans; bulk intersection
  across stops with differing eligible sets, including the empty intersection.
- **Long Lasting Extension Resolver** — extension capped by watermark; capped by
  shift end; split into two segments across a boundary; termination on machine
  resume; termination on stop end; never crossing into the unknown region.
- **Writable Window Policy** — current shift writable; previous shift inside vs
  outside the handover window; earlier shift never writable; boundary-exact
  timestamps.

Prior art: none yet — this is the first feature with application code. These four
suites establish the prior art (pure-module, table-driven, infrastructure-free)
that later SDE features should follow. The thin Reclassification Service is
covered indirectly through these modules and is not separately unit-tested under
this PRD.

## Out of Scope

- The SDE Visual Timeline rendering itself (separate feature) — this PRD consumes
  its concepts (`Coverage`, `MDC Sync Watermark`, shift-boundary rendering) but
  does not build the timeline UI.
- Multi-machine / multi-line review surfaces beyond reuse of the reclassification
  capability by P-02.
- Actual speed entry, reject entry, and constraint / non-constraint correlation
  (other SDE capabilities).
- Audit / as-of-T historical reconstruction (a separate feature); this PRD only
  guarantees as-now rendering and the retention of `Human-input Records`.
- The downtime catalog authoring/configuration workflow (owned by the Smart MDC
  Mapping / Reason Library and configuration-admin personas).
- Shift schedule configuration (owned by Shift Schedule + Production Context).

## Further Notes

- **Grilling session (2026-06-26)** resolved all six open questions in the
  `downtime_reclassification.md` feature doc and introduced two new concepts now
  captured in `CONTEXT.md`: **Ignore** (distinct from Void) and **Human-input
  Record** (replacing the previously used "Correction Record" term throughout all
  artifacts).
- **Ignore vs. Void**: Void withdraws a `Downtime` the operator created, returning
  the `Machine Stop` to Uncovered. Ignore acts directly on an Uncovered `Machine
  Stop` with no `Downtime` over it, marking it as a false trigger excluded from
  KPI output. Both actions are append-only writes; neither deletes the `Machine Stop`.
- **Bulk Reclassification grouping**: the "same type" constraint in earlier drafts
  is retired. Grouping is operator-driven; the system only constrains the eligible
  Reason set to the intersection across all selected stops.
- The grilling session that produced this PRD reconciled a real contradiction:
  the reclassification spec's "continuously extends" `Long Lasting` language
  versus the timeline's hard rules that overlays break at shift boundaries and
  nothing is invented past the `MDC Sync Watermark`. The resolution is recorded
  in ADR 0002 and reflected in the Long Lasting Extension Resolver.
- `CONTEXT.md` (root glossary) is the source of truth for all terms used
  throughout this PRD.
- Backend stack and runtime follow ADR 0001 (React, .NET/ASP.NET Core, Azure SQL,
  Azure Container Apps), within the modular-monolith approach from the Proteus
  architecture document.
