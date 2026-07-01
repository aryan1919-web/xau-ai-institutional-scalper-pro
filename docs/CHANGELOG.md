# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

- Nothing yet.

## [0.0.2] - 2026-07-01

Module 01 (Core Framework) — milestone **M1.1: types & scaffolding**. Declarations only;
no behavior. (The `v0.0.2` git tag is created after review.)

### Added
- **Core enums (5):** `Direction`, `SessionState`, `TimeframeClass`, `LogLevel`,
  `ValidationSeverity`.
- **Core UDTs (8):** `LogEntry` (now with `moduleId` / `errorId`), `Diagnostics`,
  `TimeframeContext`, `SessionContext`, `BarContext`, `ValidationResult`, `Config`,
  `KernelState`.
- **Structural constants:** `MODULE_ID_*` (per module), `DIAG_BUFFER_CAP`, project identity.
- **Empty Core subsystem scaffolding** (bannered, no functions): `util` / `log` / `err` /
  `cfg` / `ctx` / `state` / lifecycle; no-op MAIN.
- Verified: exactly one `strategy()`; zero entries / exits / orders, `request.security`,
  `ta.*`, or plots; no functions or calculations.
- **Documentation carried since 0.0.1:** Module 01 architecture; API Stability Policy;
  Performance Budget; Module Ownership Matrix (`docs/MODULE_OWNERSHIP.md`); STYLE_GUIDE
  standards (prefix registry, ordering, documentation-block rules); ADR-0006 – ADR-0014;
  Semantic Versioning Policy; Git branching model + repository-protection rules; Release
  Checklist.

### Changed
- `PROJECT_VERSION` `0.0.1` → `0.0.2`.

### Notes
- Remaining Module 01 milestones: `0.0.3` (M1.2), `0.0.4` (M1.3), `0.1.0` (M1.4 — complete).

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

[Unreleased]: https://example.com/compare/v0.0.2...HEAD
[0.0.2]: https://example.com/compare/v0.0.1...v0.0.2
[0.0.1]: https://example.com/releases/tag/v0.0.1
