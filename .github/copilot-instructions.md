<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->

# proteus-ai-tryout — Agent Instructions

This is a **Spec-Driven Development** project scaffolded with [Spec Kit](https://github.com/github/spec-kit) v0.8.15. There is **no application code yet** — features are defined, planned, and implemented through the Spec Kit workflow. The concrete tech stack is decided per-feature inside each feature's `plan.md`, so always read the active plan before writing code.

## Workflow (do these in order)

Drive work through the `speckit.*` slash commands / agents, not ad-hoc edits:

1. `speckit.constitution` — establish/update project principles ([.specify/memory/constitution.md](../.specify/memory/constitution.md))
2. `speckit.specify` — turn a feature description into `specs/<NNN>-<slug>/spec.md`
3. `speckit.clarify` — resolve underspecified areas (optional but recommended before planning)
4. `speckit.plan` — produce `plan.md` (tech stack, architecture, design artifacts)
5. `speckit.tasks` — generate a dependency-ordered `tasks.md`
6. `speckit.analyze` — cross-check spec ↔ plan ↔ tasks consistency
7. `speckit.implement` — execute the tasks

Feature artifacts live under `specs/<NNN>-<feature-slug>/` (created on first `speckit.specify`; sequential numbering).

## Conventions specific to this project

- **PowerShell on Windows**: automation scripts are `.ps1` in [.specify/scripts/powershell/](../.specify/scripts/powershell/). Run them with `pwsh`, not bash. Key helpers: `create-new-feature.ps1`, `setup-plan.ps1`, `setup-tasks.ps1`, `check-prerequisites.ps1`.
- **Constitution is the source of truth.** [constitution.md](../.specify/memory/constitution.md) is currently an unfilled template — run `speckit.constitution` before relying on it. Plans and tasks must comply with it once ratified.
- **Templates drive document shape.** Authoring follows [.specify/templates/](../.specify/templates/) (`spec-template.md`, `plan-template.md`, `tasks-template.md`, `checklist-template.md`). Don't invent new section structures.
- **Git integration is GitHub/Copilot.** Feature branches use **sequential** numbering (e.g. `001-feature-slug`). Use the `speckit.git.*` commands for branch creation, validation, commits, and remote detection rather than raw git where possible.
- The `.specify/` and `.github/prompts/` trees are Spec Kit machinery — prefer regenerating them via Spec Kit commands over hand-editing.

## When starting any coding task

1. Confirm you're on the correct feature branch and that `spec.md`, `plan.md`, and `tasks.md` exist for it.
2. Read `plan.md` for the chosen stack, structure, and commands — they are authoritative over any assumption here.
3. Implement against `tasks.md` in dependency order.
