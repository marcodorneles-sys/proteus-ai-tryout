Status: ready-for-agent

# 06 — Correct and Void a Downtime

## Parent

[PRD: Downtime Reclassification](../PRD.md)

## What to build

Let an operator fix or undo their own entries within their writable window.

**Correct**: changing a `Downtime`'s `Reason` or flags appends a new
`Human-input Record` to the append-only log. The timeline renders only the
latest effective record (as-now); superseded records are retained but not drawn.

**Void**: withdrawing a `Downtime` writes a void `Human-input Record`, makes
the `Downtime` no longer effective, and re-exposes the underlying `Machine Stop`
as Uncovered on the outstanding work list. The `Machine Stop` itself is never
deleted.

Both actions are validated by the Writable Window Policy before any write.

## Acceptance criteria

- [ ] Changing a Downtime's Reason or flags appends a new Human-input Record rather
      than overwriting the existing one.
- [ ] The timeline shows only the latest effective Human-input Record per Downtime.
- [ ] Superseded records are retained in the database but not rendered.
- [ ] Voiding a Downtime re-exposes the Machine Stop as Uncovered on the work list.
- [ ] Void and correct are both rejected by the API if the Writable Window Policy
      returns may-write = false.
- [ ] The Machine Stop is unchanged after either action.

## Blocked by

- [05 — Reclassify a single stop end-to-end](05-reclassify-single-stop.md)
