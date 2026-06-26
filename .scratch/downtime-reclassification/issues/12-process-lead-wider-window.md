Status: ready-for-agent

# 12 — Process Lead (P-02a) wider writable window

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Extend the Writable Window Policy to support the P-02a (Process Lead / Shift
Supervisor) wider write window. P-02a can write to the current and previous
shift within a factory-configurable duration that is wider than the P-01 default.
P-02a also accesses multiple lines through the office review surface, not just
their own assigned line.

End-to-end scope:
- **Writable Window Policy** (issue 03): add role-aware branching so that P-02a
  receives the wider factory-configurable window rather than the P-01 default
  30 min.
- **API**: all write endpoints pass the requesting user's role to the Policy;
  P-02a requests are evaluated against the wider window.
- **UI (office review surface)**: P-02a can navigate to any line in their factory
  scope and perform reclassification. The same reclassification rules (Reasons,
  minimums, Override Runtime) apply.

## Acceptance criteria

- [ ] P-02a write requests within the wider factory-configured window are accepted.
- [ ] P-02a write requests outside their wider window are rejected with the same
      clear error as P-01 out-of-window rejections.
- [ ] P-01 window is unchanged by this change — P-01 still sees the 30 min default.
- [ ] The office review surface allows P-02a to reclassify stops on any line
      within their factory scope.
- [ ] The same Reason eligibility rules apply to P-02a as to P-01.

## Blocked by

- [11 — Shift window enforcement in UI and API](11-shift-window-enforcement.md)
