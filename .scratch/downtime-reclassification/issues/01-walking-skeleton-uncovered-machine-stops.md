Status: ready-for-human

# 01 — Walking skeleton: surface Uncovered Machine Stops

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

The first end-to-end slice and the application's walking skeleton. Seed a small
set of `Machine Stops` into Azure SQL, expose them read-only through an API, and
render the operator's red work list in React — a minimal strip showing the
Uncovered red, **not** the full SDE Visual Timeline (that is a separate feature).
With no `Downtimes` yet, the Coverage Engine returns every `Machine Stop` as
Uncovered.

This slice also stands up the project scaffolding the rest of the feature builds
on: repository layout, the Smart Downtime Entry module boundary inside the
modular monolith, CI, and Bicep/Azure SQL provisioning (per ADR 0001). Because it
fixes architectural shape, it is HITL — a human should review the skeleton before
the AFK slices land on top of it.

The `Machine Stop` base is read-only end-to-end: there is no path in the API or
UI to edit or delete it.

## Acceptance criteria

- [ ] Repository, module boundary, CI, and IaC scaffolding are in place and documented enough for the next slices to build on.
- [ ] Seeded `Machine Stops` for one machine/shift persist in Azure SQL.
- [ ] An API returns the `Machine Stops` for a machine over a window, read-only.
- [ ] The Coverage Engine computes Uncovered / Partially Covered / Covered regions and, with no `Downtimes`, reports everything Uncovered.
- [ ] A minimal React surface renders the Uncovered red work list from the API.
- [ ] No API or UI path exists to edit or delete a `Machine Stop`.
- [ ] Coverage Engine has table-driven tests (fully uncovered case at minimum).

## Blocked by

- None - can start immediately
