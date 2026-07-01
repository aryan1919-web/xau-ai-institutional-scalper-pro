# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

- Nothing yet.

## [0.1.1] - 2026-07-01

Module 02 (Trend Engine) — milestone **M2.1: Core `ctxHtfValue` HTF primitive**. The
reserved Core primitive is implemented (behavior-frozen Core, additively extended per D1);
defined but not yet invoked; no trend logic. (The `v0.1.1` git tag is created manually
after review.)

### Added
- **`ctxHtfValue(tf, expr) → float`** (`ctx`, Stable, Since 0.1.1): the single sanctioned
  higher-timeframe read and the **only** `request.security` in the strategy (ADR-0011).
  Non-repainting — `lookahead_off`, `gaps_off`, `expr[1]` (last-closed HTF bar; 1 HTF-bar lag
  by design). **Core owns symbol context** — always reads `syminfo.tickerid`; modules pass no symbol.
- **`ctxIsHigherTimeframe(tf) → bool`** (internal, Since 0.1.1): HTF ≥ chart-timeframe validation.

### Changed
- `PROJECT_VERSION` `0.1.0` → `0.1.1`.
- **Interface simplification (`refactor(core)`):** `ctxHtfValue(symbol, tf, expr)` →
  `ctxHtfValue(tf, expr)`; the chart symbol (`syminfo.tickerid`) is applied internally. No
  behavior change (modules would have passed exactly `syminfo.tickerid`); still one
  `request.security`, still non-repainting.
- `docs/API.md`: `ctxHtfValue` moved from *Reserved* to *Implemented* (context table); Reserved
  section is now empty.
- `docs/ROADMAP.md`: added the Module 02 sub-milestone version rows (`0.1.1`–`0.2.0`); current
  marker moved to `0.1.1`.

### Notes
- No trading behavior: zero entries/exits/orders, `ta.*`, or plots. The lone `request.security`
  lives inside `ctxHtfValue` and is **not yet invoked** (defined for Module 02's M2.4 use).
- Parameter is named `tf` (not `timeframe`) to avoid shadowing the `timeframe.*` builtin.
- Next: M2.2 (`0.1.2`) — trend types, inputs, configuration.

## [0.1.0] - 2026-07-01

Module 01 (Core Framework) — milestone **M1.4: kernel state, lifecycle, diagnostics,
integration**. **Module 01 is complete.** Foundation only — still zero trading behavior.

### Added
- **KernelState subsystem (`state`):** `stateInit(config, diag)`, `stateUpdate(state,
  context)` (public); `stateGet`, `stateReset` (internal, debug). `stateInit` sizes the
  diagnostics buffer from `config.diagBufferCap`, finalizes `primaryTfClass`, and enforces
  `requireSupportedTf` (fatal `CORE-CFG-001` on an unsupported timeframe).
- **Lifecycle subsystem (`core`):** `coreInit()` (once) and `coreOnBar(state)` (per bar).
- **Integration:** single `var KernelState coreState = coreInit()` + one `coreOnBar(coreState)`
  call in MAIN. `coreOnBar` returns the `BarContext` seam for modules 02-14.
- **Debug diagnostics harness** inside `coreInit` (runs when `debugEnabled`): exercises
  util / log / err / cfg / ctx and asserts validation, ring-buffer round-trip, config access,
  and state initialization; plus a per-bar `barCount` parity invariant in `coreOnBar`
  (proves single initialization + monotonic increment).

### Changed
- `PROJECT_VERSION` `0.0.4` → `0.1.0`.
- `docs/API.md`: `state` and `core` moved to *implemented* (Since 0.1.0); documented the
  final reference-passing design (no parameter-less `cfgGet`); only `ctxHtfValue` remains reserved.
- `docs/ROADMAP.md`: current-version marker moved to `0.1.0`.

### Fixed (stabilization before freeze)
- **`errApplyValidation`**: guarded the results loop with `n > 0` so a clean config
  (empty results array) never evaluates `for i = 0 to -1` — removes a potential
  out-of-bounds on the common path.
- **Duplication**: extracted `logInitBuffer(diag, capacity)`; `logWrite` and `stateInit`
  now share one ring-buffer allocator (no-duplication rule).
- **UDT naming**: renamed the `time` field of `LogEntry` and `BarContext` to `barTime`
  to avoid shadowing the `time` builtin (fields were write-only; no behavior change).
- Removed obsolete milestone/scaffolding comments; refreshed section banners now that
  Module 01 is complete.
- **Pine v6 compiler fixes:** `Config.diagBufferCap` default is now the literal `100`
  (Pine forbids named constants in UDT field defaults; `cfgBuild` still overrides it from
  the input); `errApplyValidation` uses two independent `if` statements instead of an
  `if/else-if` chain to avoid mismatched branch return types `(void; series int)`.
  Behavior unchanged.

### Notes
- Verification: one `strategy()`; zero entries/exits/orders, `request.security`, `ta.*`, or
  plots. Non-repainting (current-bar builtins only). Single initialization via `var`.
- **Module 01 is complete.** Next: Module 02 — Trend Engine (`0.2.0`).

## [0.0.4] - 2026-07-01

Module 01 (Core Framework) — milestone **M1.3: configuration & context**.
Subsystems are defined but not yet invoked (MAIN remains no-op). (The `v0.0.4` git
tag is created after review.)

### Added
- **Configuration subsystem (`cfg`):** `cfgBuild` (the sole reader of `input.*`,
  ADR-0008), `cfgGet` (public); `cfgValidate` (internal). Produces an immutable
  `Config` snapshot; recoverable issues are clamped and the M1.2 `err` subsystem
  applies the hybrid policy (no duplicated validation).
- **Context subsystem (`ctx`):** `ctxBuild`, `ctxIsConfirmedBar`, `ctxIsNewBar`
  (public); `ctxComputeSession`, `ctxTimeframe` (internal). Strictly non-repainting
  (current-bar builtins only; `barstate.isnew` for new-bar detection). **Zero
  `request.security`.** `ctxHtfValue` remains reserved (docs only).
- **Core configuration inputs** (group *01 · Core Framework*): debug toggle, log
  threshold (`input.enum`), require-supported-timeframe, session enable + start/end
  minute-of-day, diagnostics buffer size — all read only by `cfgBuild`.
- Constants: `DIAG_BUFFER_CAP_MIN`, `MINUTES_PER_DAY`, `TF_M1_MINUTES`, `TF_M5_MINUTES`.

### Changed
- `PROJECT_VERSION` `0.0.3` → `0.0.4`.
- `Config`: session window is now integer minute-of-day bounds
  (`sessionStartMinute` / `sessionEndMinute`) instead of a session string — a
  `series`-typed field cannot be passed to `time(tf, session)`, and integer bounds
  reuse `utilMinutesOfDay` and give a clean `start < end` validation.
- `docs/API.md`: `cfg`/`ctx` moved to *implemented* (Since 0.0.4); `cfgGet` documented
  as taking a `KernelState` handle until the M1.4 lifecycle.

### Notes
- No trading behavior: no entries/exits/orders, `request.security`, `ta.*`, or plots.
- Subsystems defined but not called; wired via the lifecycle in M1.4.
- `primaryTfClass` and the `requireSupportedTf` enforcement are finalized by the
  M1.4 lifecycle (which holds both `Config` and `ctxTimeframe`).
- Remaining Module 01 milestone: `0.1.0` (M1.4 — complete).

## [0.0.3] - 2026-07-01

Module 01 (Core Framework) — milestone **M1.2: utility, logging, validation**.
Subsystems are defined but not yet invoked (MAIN remains no-op). (The `v0.0.3` git
tag is created after review.)

### Added
- **Utility subsystem (`util`):** `utilIsValidNumber`, `utilClamp`, `utilSafeDiv`,
  `utilNormalize`, `utilRoundToTick` (public); `utilMinutesOfDay`, `utilFormatFloat`
  (internal). Pure and deterministic.
- **Logging subsystem (`log`):** bounded **O(1)** diagnostics ring buffer —
  `logWrite`, `logDrain`, `logClear` (public); `logShouldEmit`, `logFormat`,
  `logLevelRank` (internal). Level + debug gated.
- **Error / validation subsystem (`err`):** `errRaiseFatal`, `errWarn`, `errAssert`,
  `errApplyValidation` (public); `errFormat` (internal). Hybrid policy — first fatal
  halts, warnings are logged.
- **Error-ID registry:** `ERR_CFG_UNSUPPORTED_TF`, `ERR_CFG_INVALID_SESSION`,
  `ERR_CFG_DIAG_CAP_RANGE`, `ERR_STATE_REINIT`, `ERR_ASSERT_FAILED`, `ERR_NONE`.
- **Utility constants:** `UTIL_NORMALIZED_MIN/MAX`, `MINUTES_PER_HOUR`, `DECIMAL_BASE`.

### Changed
- `PROJECT_VERSION` `0.0.2` → `0.0.3`.
- `docs/API.md`: replaced the planned-only catalog with implemented `util`/`log`/`err`
  signatures (Since 0.0.3); removed the obsolete M0 placeholder utilities; documented the
  current explicit-parameter forms (Config/KernelState wiring lands in M1.4).

### Notes
- No trading behavior: no entries/exits/orders, `request.security`, `ta.*`, or plots.
- Subsystems are defined but not called; they are wired via the lifecycle in M1.4.
- Remaining Module 01 milestones: `0.0.4` (M1.3), `0.1.0` (M1.4 — complete).

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

[Unreleased]: https://example.com/compare/v0.1.1...HEAD
[0.1.1]: https://example.com/compare/v0.1.0...v0.1.1
[0.1.0]: https://example.com/compare/v0.0.4...v0.1.0
[0.0.4]: https://example.com/compare/v0.0.3...v0.0.4
[0.0.3]: https://example.com/compare/v0.0.2...v0.0.3
[0.0.2]: https://example.com/compare/v0.0.1...v0.0.2
[0.0.1]: https://example.com/releases/tag/v0.0.1
