Status: ready-for-agent

# 13 — Production Analyst (P-02b) KPI Period write access

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Extend the Writable Window Policy to support P-02b (Production Analyst) write
access within the open KPI Period. P-02b can reclassify stops on any shift that
falls within the current open KPI Period (a calendar month); once the period
locks (~5th business day of the following month), writes are blocked.

End-to-end scope:
- **Writable Window Policy** (issue 03): add a P-02b branch that evaluates the
  target timestamp against the open KPI Period boundary rather than a shift
  handover window.
- **API**: all write endpoints pass the requesting user's role to the Policy;
  P-02b requests are evaluated against the KPI Period boundary.
- **UI**: P-02b sees reclassification write actions available for any stop within
  the open period; stops in a locked period render as read-only.

## Acceptance criteria

- [ ] P-02b can reclassify any stop within the open KPI Period.
- [ ] P-02b write requests against a locked period are rejected at the API layer.
- [ ] P-01 and P-02a window behaviour is unchanged.
- [ ] The UI renders stops in a locked period as read-only for P-02b.

## Blocked by

- [11 — Shift window enforcement in UI and API](11-shift-window-enforcement.md)
