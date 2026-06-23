# Domain Docs

How the engineering skills should consume this repo's domain documentation when exploring the codebase.

## Before exploring, read these

- **`CONTEXT.md`** at the repo root (single-context layout).
- **[_docs/adr/](../adr/)** — read ADRs that touch the area you're about to work in.
- **[_docs/architecture/proteus_architecture_document.md](../architecture/proteus_architecture_document.md)** — the system architecture overview.
- **[_docs/domain/](../domain/)** — personas, modules, and features that define the product surface.

If `CONTEXT.md` doesn't exist yet, **proceed silently**. Don't flag its absence; don't suggest creating it upfront. The producer skill (`/grill-with-docs`) creates it lazily when terms or decisions actually get resolved.

## File structure (single-context)

```
/
├── CONTEXT.md                      ← created lazily by /grill-with-docs
├── _docs/
│   ├── adr/                        ← architectural decision records
│   │   └── 0001-core-technology-runtime-stack.md
│   ├── architecture/
│   │   └── proteus_architecture_document.md
│   └── domain/
│       ├── personas.md
│       ├── modules/
│       └── features/
└── (backend/ frontend/ infrastructure/ — generated via Spec-Kit)
```

> **Note:** This repo uses `_docs/` (with a leading underscore), not the stock `docs/`. ADRs therefore live at `_docs/adr/`, not `docs/adr/`. Honour this when writing new ADRs or referring to existing ones.

## Use the glossary's vocabulary

When your output names a domain concept (in an issue title, a refactor proposal, a hypothesis, a test name), use the term as defined in `CONTEXT.md`. Don't drift to synonyms the glossary explicitly avoids.

If the concept you need isn't in the glossary yet, that's a signal — either you're inventing language the project doesn't use (reconsider) or there's a real gap (note it for `/grill-with-docs`).

The domain language for Proteus also lives in [_docs/domain/](../domain/) (personas, modules, features). Treat module/feature filenames and headings as canonical terms until `CONTEXT.md` says otherwise.

## Flag ADR conflicts

If your output contradicts an existing ADR, surface it explicitly rather than silently overriding:

> _Contradicts ADR-0001 (core technology & runtime stack) — but worth reopening because…_
