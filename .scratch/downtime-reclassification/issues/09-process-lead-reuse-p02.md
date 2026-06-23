Status: ready-for-human

# 09 — Process Lead reuse (P-02)

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Expose the reclassification capability to the Process Lead (Persona P-02) through
the office review surface, so they can correct and complete operator data across
multiple lines, with a wider writable window (current and previous shift). The
same reclassification rules — `Reason` eligibility, minimum durations,
override-runtime — apply to P-02 as to P-01.

This is HITL: it requires decisions on RBAC wiring and how far the multi-line
office surface goes (the broader multi-machine review surface is largely out of
scope per the PRD). A human should shape the surface and permission model before
implementation.

## Acceptance criteria

- [ ] A Process Lead can reclassify downtimes through the office review surface across more than one line.
- [ ] P-02's writable window spans the current and previous shift per their permissions.
- [ ] `Reason` eligibility, minimum-duration, and override-runtime rules apply identically to P-02.
- [ ] RBAC distinguishes P-01 and P-02 access per personas.md.
- [ ] The scope boundary against the broader multi-line review surface is documented and agreed.

## Blocked by

- [06 — Writable Window Policy](./06-writable-window-policy.md)
