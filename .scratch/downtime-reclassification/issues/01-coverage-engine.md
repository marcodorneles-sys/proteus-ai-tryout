Status: ready-for-agent

# 01 — Coverage Engine — pure module + tests

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Build the Coverage Engine as a deep, pure, side-effect-free module. It takes a
set of `Machine Stops` and the effective `Downtimes` over a time window, and
returns the Coverage state of each `Machine Stop`: Uncovered, Partially Covered,
or Covered.

Rules:
- Coverage is computed against red (Machine Stop intervals) only; covering green
  runtime is an explicit exception handled by the Override Runtime rule, not by
  this module.
- Cardinality: one `Downtime` may span many `Machine Stops`.
- An Ignored `Machine Stop` (has a `Human-input Record` with `action_type=ignore`)
  is treated as resolved — it must not appear as Uncovered.
- No infrastructure dependencies; inputs and outputs are plain data structures.

Deliver the module with a full table-driven test suite covering:
- Fully Uncovered stop (no Downtime over it)
- Partially Covered stop (Downtime covers part of the interval)
- Fully Covered stop
- One Downtime spanning several Machine Stops and green runtime between them
- An Ignored Machine Stop (not shown as Uncovered)

## Acceptance criteria

- [ ] Pure function / class with no side effects and no I/O dependencies.
- [ ] Returns Uncovered, Partially Covered, or Covered per Machine Stop.
- [ ] Ignored Machine Stops are treated as resolved (not Uncovered).
- [ ] Table-driven tests pass for all scenarios listed above.
- [ ] No infrastructure (DB, HTTP) required to run the tests.

## Blocked by

None — can start immediately.
