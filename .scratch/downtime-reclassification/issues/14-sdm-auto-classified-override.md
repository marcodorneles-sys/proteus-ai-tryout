Status: ready-for-agent

# 14 — ServiceNow SDM auto-classified stop override

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Support overriding a `Machine Stop` that was auto-classified by a ServiceNow SDM
planned activity. SDM planned activities may produce a system-generated
`Human-input Record` (auto-classification with `action_type=classify`,
`user_id=system`). Operators and Process Leads must be able to override this with
a normal Reclassification; the human override takes precedence in KPI output via
the Override Runtime rule.

**Note:** This slice is marked HITL because the SDM auto-classification ingest
boundary (how SDM writes the system-generated Human-input Record into Proteus)
has not been fully designed. A brief integration design review with the SDM team
is required before implementation can start. Once the ingest mechanism is agreed,
this issue should be updated and re-marked as AFK.

End-to-end scope (post design review):
- **Ingest**: system-generated `Human-input Record` written by the SDM integration
  pipeline with `user_id=system` and `action_type=classify`.
- **Override**: an operator or Process Lead performs a normal Reclassification on
  the stop. The Reclassification Service appends a new Human-input Record; the
  as-now rendering shows the human record. The KPI engine uses the human record
  via the Override Runtime rule.
- **UI**: auto-classified stops are visually distinguishable from
  operator-classified stops (e.g. a "planned activity" badge). Write actions
  remain available for eligible roles within their writable window.

## Acceptance criteria

- [ ] A system-generated Human-input Record (action_type=classify, user_id=system)
      is correctly ingested from ServiceNow SDM.
- [ ] An operator or Process Lead can perform a normal Reclassification over an
      auto-classified stop.
- [ ] The as-now rendering shows the human override, not the system record.
- [ ] The KPI engine uses the human override in its output.
- [ ] Auto-classified stops are visually distinguishable from operator-classified stops.
- [ ] The SDM integration design has been reviewed and agreed before implementation
      starts (HITL gate).

## Blocked by

- [05 — Reclassify a single stop end-to-end](05-reclassify-single-stop.md)
- **HITL gate**: SDM auto-classification ingest design must be agreed with the
  SDM integration team before implementation begins.
