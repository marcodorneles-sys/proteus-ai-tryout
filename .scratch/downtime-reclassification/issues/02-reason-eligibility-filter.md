Status: ready-for-agent

# 02 — Reason Eligibility Filter — pure module + tests

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Build the Reason Eligibility Filter as a deep, pure, side-effect-free module. It
takes a selected span (or a set of spans for bulk) and the downtime catalog with
per-`Reason` rules, and returns the set of eligible `Reasons` the operator is
allowed to pick.

Rules:
- Minimum duration: a `Reason` is eligible only if the selected span meets or
  exceeds the Reason's configured minimum. Applied as a catalog filter at
  selection time — never as a post-hoc validation error.
- Override Runtime: a `Reason` that does not allow runtime override is ineligible
  when the selected span includes green (running) intervals.
- Unmapped stops: return an empty eligible set (blocks classification — this is
  enforced at the API layer using this module's output, not just in the UI).
- Bulk: return the **intersection** of eligible Reason sets across all selected
  stops. If the intersection is empty, no Reason can be picked.
- The `Long Lasting` duration threshold is read from factory configuration
  (configurable per factory / work center, owned by P-03a).
- No infrastructure dependencies.

Deliver the module with a full test suite covering:
- Span just below / at / above a minimum duration threshold
- Override-runtime on vs off against a span that includes green runtime
- Bulk intersection across stops with differing eligible sets
- Bulk intersection that resolves to empty
- Unmapped stop returning empty eligible set

## Acceptance criteria

- [ ] Pure function / class with no side effects and no I/O dependencies.
- [ ] Filters by minimum duration at selection time, not post-hoc.
- [ ] Filters by override-runtime rule when the span includes green runtime.
- [ ] Returns empty set for unmapped stops.
- [ ] Bulk returns the intersection; empty intersection is a valid output.
- [ ] All test scenarios pass.
- [ ] No infrastructure required to run the tests.

## Blocked by

None — can start immediately.
