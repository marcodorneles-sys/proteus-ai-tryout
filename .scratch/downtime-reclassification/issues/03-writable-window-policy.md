Status: ready-for-agent

# 03 — Writable Window Policy — pure module + tests

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Build the Writable Window Policy as a deep, pure, side-effect-free module. It
takes the current time, shift schedule, `Shift Handover Window` configuration,
the requesting user's role, and a target timestamp, and returns a binary
may-write decision.

Rules encoded by this module:
- The current shift is always writable.
- The previous shift is writable only while its `Shift Handover Window` is open
  (default 30 min after shift end; configurable per factory by P-03a).
- Any shift older than the previous shift is never writable.
- Visibility never implies writability — bringing an older shift on screen does
  not grant write access.
- P-02a (Process Lead / Shift Supervisor) has a wider window: current and
  previous shift, factory-configurable duration (wider than P-01's 30 min default).
- P-02b (Production Analyst) can write within the open KPI Period.
- No infrastructure dependencies.

Deliver the module with a test suite covering:
- Current shift writable for all applicable roles
- Previous shift writable while inside the handover window
- Previous shift not writable once the handover window has closed
- Earlier shift never writable regardless of role
- Boundary-exact timestamps (exactly at window close)
- P-02a wider window: previous shift writable within the wider config
- P-02b: writable within the open KPI Period, not outside it

## Acceptance criteria

- [ ] Pure function / class with no side effects and no I/O dependencies.
- [ ] Returns a may-write boolean (or equivalent) given role + target timestamp + config.
- [ ] Current shift always writable.
- [ ] Previous shift writable only within the handover window for P-01.
- [ ] P-02a wider window respected.
- [ ] P-02b KPI Period access respected.
- [ ] All test scenarios including boundary-exact timestamps pass.
- [ ] No infrastructure required to run the tests.

## Blocked by

None — can start immediately.
