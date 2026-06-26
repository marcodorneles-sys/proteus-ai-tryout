Status: ready-for-agent

# 11 — Shift window enforcement in UI and API

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Wire the Writable Window Policy (issue 03) through the full stack so that
write access is enforced at the API layer and the UI reflects the read-only state
correctly. Visibility must never imply writability.

Rules to enforce end-to-end:
- Current shift: always writable for P-01.
- Previous shift: writable while the `Shift Handover Window` is open (default
  30 min; configurable per factory). Locked once the window closes.
- Any earlier shift: always read-only regardless of role.
- Bringing an older shift onto the screen does not grant write access.

End-to-end scope:
- **API**: all write endpoints (classify, bulk-reclassify, correct, void, ignore)
  call the Writable Window Policy and return a 403 with a clear reason when the
  window is closed.
- **UI**: when viewing a locked shift, all write actions (Reason picker, Ignore,
  Void, Correct) are hidden or disabled. The UI must not rely solely on the API
  rejection to communicate read-only state — it should render visually read-only
  before the user attempts a write.

## Acceptance criteria

- [ ] All write API endpoints reject requests for out-of-window timestamps with
      a clear error (not a generic 500).
- [ ] The UI renders shifts outside the writable window as read-only — write
      actions are not offered.
- [ ] Viewing a locked shift never triggers a write.
- [ ] The current shift remains writable throughout.
- [ ] The previous shift becomes read-only once the handover window closes
      (verifiable with a configurable window set to a short duration in tests).

## Blocked by

- [03 — Writable Window Policy](03-writable-window-policy.md)
- [05 — Reclassify a single stop end-to-end](05-reclassify-single-stop.md)
