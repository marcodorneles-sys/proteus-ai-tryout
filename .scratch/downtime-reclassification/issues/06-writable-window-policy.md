Status: ready-for-agent

# 06 — Writable Window Policy

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Constrain *when* reclassification, correction, and voiding are allowed. The
Writable Window Policy decides whether a write to a given timestamp is permitted:
the current shift is always writable; the previous shift is writable only while
its `Shift Handover Window` is still open; bringing an earlier shift onto the
screen stays read-only (visibility never implies writability). The
Reclassification Service consults this policy before any write.

This slice consumes a **seeded** shift schedule; the real Shift Schedule +
Production Context integration is out of scope.

## Acceptance criteria

- [ ] Writes to the current shift are always permitted.
- [ ] Writes to the previous shift are permitted only within the open `Shift Handover Window` and blocked once it closes.
- [ ] Writes to any earlier shift are blocked even when it is visible.
- [ ] The Reclassification Service rejects writes the policy disallows.
- [ ] Boundary-exact timestamps (window open/close edges) behave deterministically.
- [ ] Writable Window Policy has table-driven tests for current / previous-in-window / previous-out-of-window / earlier shifts.

## Blocked by

- [02 — Reclassify a single stop, creating a Downtime](./02-reclassify-single-stop-create-downtime.md)
