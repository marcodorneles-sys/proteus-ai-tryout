Status: ready-for-agent

# 10 — Unmapped stop indicator and classification block

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Surface `Machine Stops` whose MDC reason code has no SmartMDC classification
mapping with a clear "unmapped" visual indicator, and block classification at
the API layer until the Mapping Responsible (P-02c) maps the code.

An unmapped stop remains on the outstanding work list as Uncovered so the
operator can see it, but they cannot classify it — they need to alert P-02c.

End-to-end scope:
- **Reason Eligibility Filter** (issue 02): already returns an empty eligible set
  for unmapped stops. This slice wires that output into an API-layer rejection.
- **API**: classify and bulk-reclassify endpoints return a clear error when called
  against an unmapped stop (Reason Eligibility Filter returns empty).
- **UI**: unmapped `Machine Stops` display a distinct visual indicator (e.g.
  "unmapped" badge or icon). The Reason picker is disabled or communicates that
  no Reasons are available. No free-text fallback is offered.

## Acceptance criteria

- [ ] An unmapped Machine Stop is shown on the timeline with a visible "unmapped"
      indicator distinguishing it from a normally Uncovered stop.
- [ ] Attempting to classify an unmapped stop is rejected at the API layer (not
      only blocked in the UI).
- [ ] The Reason picker communicates that no Reasons are available for this stop.
- [ ] No free-text or fallback classification path is available.
- [ ] The stop remains on the outstanding work list as Uncovered until P-02c
      maps the reason code.

## Blocked by

- [05 — Reclassify a single stop end-to-end](05-reclassify-single-stop.md)
