# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

### Added
- **Module 01 — Core Framework architecture** (design only; no code). Full architecture
  in `src/modules/01-core-framework/README.md`: responsibilities, subsystems
  (`util`/`log`/`err`/`cfg`/`ctx`/`state`/orchestrator), planned public API, shared
  enums/UDTs, hybrid validation, non-repainting primitives, single `KernelState`.
- **API Stability Policy** (`Stable` / `Experimental` / `Internal`) in `docs/API.md`
  and `docs/STYLE_GUIDE.md`.
- **Performance Budget** (permanent, project-wide caps) in `docs/SPECIFICATION.md`,
  mirrored in `docs/STYLE_GUIDE.md`.
- **Module Ownership Matrix** — new `docs/MODULE_OWNERSHIP.md`, the authoritative
  dependency contract for all 14 modules; indexed in `docs/README.md` and linked from
  `docs/ARCHITECTURE.md`.
- Coding standards in `docs/STYLE_GUIDE.md`: subsystem/module prefix registry, function
  and file section ordering, expanded documentation-block requirements.
- Architectural Decision Records **ADR-0006 – ADR-0014** in `docs/DECISIONS.md`.
- **Semantic Versioning Policy** (PATCH / MINOR / MAJOR bump rules) — authoritative in
  `docs/ROADMAP.md`, mirrored in `CONTRIBUTING.md`.
- **Official Git branching model** in `CONTRIBUTING.md` (`main` / `develop` /
  `feature/*` / `bugfix/*` / `docs/*` / `hotfix/*`), with merge rules and a
  repository-protection-rules table (protected branches, direct-commit / force-push /
  review / merge-strategy policy per branch).
- **Release Checklist** in `CONTRIBUTING.md` (compile, docs, CHANGELOG, review report,
  version, tag, release notes).

_No changes to `src/MASTER_STRATEGY.pine` (architecture only)._

### Planned — Module 01 implementation

Module 01 is implemented across four independently-compiled, independently-reviewed,
independently-committed milestones. Each milestone is released and tagged on its own
version (see [ROADMAP.md](ROADMAP.md#version-timeline)); dated sections are added below
as each milestone lands:

- **`0.0.2`** — M1.1: enums, UDTs, constants, empty subsystem scaffolding.
- **`0.0.3`** — M1.2: utility, logging (bounded ring buffer), and validation subsystems.
- **`0.0.4`** — M1.3: configuration (immutable `Config`) and context (confirmed-bar /
  session / timeframe) subsystems. `ctxHtfValue` remains reserved (docs only).
- **`0.1.0`** — M1.4: `KernelState`, lifecycle (`coreInit`/`coreOnBar`), debug
  diagnostics, and final integration — Module 01 complete.

## [0.0.1] - 2026-07-01

### Added
- Initial project architecture and repository scaffold.
- Documentation set: `README`, `AGENTS`, `CONTRIBUTING`, and `docs/`
  (`SPECIFICATION`, `ARCHITECTURE`, `ROADMAP`, `CHANGELOG`, `DECISIONS`, `API`,
  `STYLE_GUIDE`, `BACKTEST_PROTOCOL`, `KNOWN_LIMITATIONS`).
- `.github/` pull request and issue templates.
- Repository configuration: `LICENSE` (MIT), `.editorconfig`, `.gitattributes`,
  `.gitignore`.
- Framework-only `src/MASTER_STRATEGY.pine` (Pine Script v6): strategy
  declaration, structural constants, per-module input group banners, documented
  placeholder utility functions, Module 01–14 section placeholders, and a TODO
  roadmap. No trading logic, indicators, entries, exits, or calculations.
- Documentation-only `src/modules/` tree with one folder per planned module.

[Unreleased]: https://example.com/compare/v0.0.1...HEAD
[0.0.1]: https://example.com/releases/tag/v0.0.1
