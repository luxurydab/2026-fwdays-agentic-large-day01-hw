# AGENTS.md

## Purpose

This file is the entry point to the project memory bank.

Use it to bootstrap context before making changes, reviewing code, or updating documentation. The source of truth for project context lives under `docs/`.

## Recommended Read Order

1. `docs/memory/projectbrief.md`
2. `docs/memory/productContext.md`
3. `docs/memory/systemPatterns.md`
4. `docs/memory/techContext.md`
5. `docs/memory/activeContext.md`
6. `docs/memory/progress.md`
7. `docs/memory/decisionLog.md`
8. Relevant product and technical docs for the task at hand

## Memory Bank Index

### Core Memory

- [docs/memory/projectbrief.md](docs/memory/projectbrief.md) - High-level summary of the product, stakeholders, business value, and system boundaries.
- [docs/memory/productContext.md](docs/memory/productContext.md) - UX goals, primary scenarios, expected outcomes, and product risks.
- [docs/memory/systemPatterns.md](docs/memory/systemPatterns.md) - Core architectural and data-flow patterns used across the system.
- [docs/memory/techContext.md](docs/memory/techContext.md) - Stack, repo layout, services, commands, environment model, and deployment notes.
- [docs/memory/activeContext.md](docs/memory/activeContext.md) - Current priorities, blockers, open questions, assumptions, and immediate risks.
- [docs/memory/progress.md](docs/memory/progress.md) - Completed work, current work, next work, and overall progress assessment.
- [docs/memory/decisionLog.md](docs/memory/decisionLog.md) - ADR-style decisions plus undocumented behavior findings captured from the codebase.

### Product Docs

- [docs/product/PRD.md](docs/product/PRD.md) - Product requirements, target users, jobs to be done, features, and main user flows.
- [docs/product/domain-glossary.md](docs/product/domain-glossary.md) - Shared terminology, naming conventions, abbreviations, and ambiguity to avoid.

### Technical Docs

- [docs/technical/architecture.md](docs/technical/architecture.md) - System breakdown, runtime topology, infrastructure, storage design, and critical sequences.
- [docs/technical/behaviors.md](docs/technical/behaviors.md) - Implicit behaviors, edge cases, maintainer gotchas, and technical debt.
- [docs/technical/dev-setup.md](docs/technical/dev-setup.md) - Local setup, build and test workflow, Docker flow, and common troubleshooting paths.

## How To Use This File

- For broad product understanding, start with the Core Memory section.
- For feature work, read the relevant Product Docs after the core memory files.
- For implementation work, debugging, or refactoring, read the relevant Technical Docs after the core memory files.
- For changes that might affect current priorities or assumptions, check `docs/memory/activeContext.md` and `docs/memory/progress.md` before editing.
- For changes that alter an architectural choice or reveal undocumented behavior, update `docs/memory/decisionLog.md`.

## Maintenance Rule

When a new markdown file is added under `docs/`, add it to this index so AGENTS.md remains the single entry point to the memory bank.