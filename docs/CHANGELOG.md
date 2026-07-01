# Changelog

All notable changes to this project are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/).

## [Unreleased]

- Nothing yet.

## [0.2.0] - 2026-07-01

Module 02 (Trend Engine) **complete** — milestone **M2.5: stabilization, finalization & API
freeze**. The Trend Engine is functionally done; this milestone freezes its public surface and
adds diagnostics/validation. Still no signals/orders/plots/alerts. **MINOR bump** — the Trend API
is promoted from Experimental to **Stable** (per the Semantic Versioning Policy). (The `v0.2.0`
git tag is created manually after review.)

### Added
- **`trendSelfCheck` (`@internal`, debug-only):** asserts the `TrendState` ABI invariants
  (normalized `strength`/`confidence`/`quality` ranges; `isActive` ⇒ directional and ≥ threshold)
  once per bar via `errAssert`. Read-only — never constructs a `TrendState` or touches
  `TrendMemory`; wired in MAIN behind `config.debugEnabled` (no-op in production).
- **Validation `TREND-CFG-004` (recoverable):** `flatThreshold` must be ≤ `strengthThreshold`,
  else the strength bands are unordered (weak band unreachable); clamped down with a warning.
- **[ADR-0016](DECISIONS.md):** freezes the Trend Engine public API and the `TrendState` ABI.

### Changed
- `PROJECT_VERSION` `0.1.4` → `0.2.0`.
- **Trend public API promoted to `@stable` and frozen** (ADR-0016): `trendEvaluate` and the
  accessors `trendDirection`/`trendStrength`/`trendConfidence`/`trendQuality`/`trendPhase`/
  `trendIsActive`/`trendIsAligned`. Renaming/redesigning any of them now requires a new ADR.
- `TrendState` ABI **locked** — normalized outputs only; raw EMA/ATR/slope/separation/volatility
  fields remain forbidden.
- Module 02 milestone banner marked COMPLETE; `docs/ROADMAP.md` current marker moved to `0.2.0`.

### Notes
- **No behavior change** to the trend computation itself — M2.5 is stabilization, diagnostics,
  validation and documentation only. No new public trading logic, signals, entries/exits/orders,
  plots or alerts.
- **Invariants preserved:** exactly one executable `request.security` (inside `ctxHtfValue`);
  exactly one executable `ctxHtfValue` call (`trendHtfView`); `TrendState.new()` only inside
  `trendEvaluate`; `TrendMemory` touched only by `trendEvaluate`; `TrendTimeframeView` internal.
  Deterministic, non-repainting, O(1) per bar; performance budgets unchanged.
- Next: M3 (`0.3.0`) — Module 03 Momentum Engine.

## [0.1.4] - 2026-07-01

Module 02 (Trend Engine) — milestone **M2.4: multi-timeframe synthesis**. The chart-timeframe
trend is now confirmed against a higher timeframe, with independent confidence and quality; still
no signals/orders/plots/alerts. (The `v0.1.4` git tag is created manually after review.)

### Added
- **Internal helpers:** `trendSignedStrength` (one signed-strength scalar in `[-1, 1]` — the
  replaceable formula evaluated on the current series), `trendDecodeSigned` (decodes a signed
  scalar into a `TrendTimeframeView`), `trendHtfView` (reads the same formula on the HTF), and
  `trendMergeViews` (synthesizes chart + HTF into direction/strength/confidence/quality/alignment).
- **Validation:** `TREND-CFG-001` (fatal) now active — when MTF confirmation is on, the HTF
  timeframe must be `>=` the chart timeframe.

### Changed
- `PROJECT_VERSION` `0.1.3` → `0.1.4`.
- **`ctxHtfValue` is now invoked** for the first time, from `trendHtfView` — the single
  `ctxHtfValue` call site in the strategy (HTF budget = 1). The lone `request.security` still
  lives **only** inside `ctxHtfValue` (executable count = 1).
- `trendChartView` refactored to decode `trendSignedStrength` (shared formula) via
  `trendDecodeSigned` — behavior on the chart is unchanged.
- `trendEvaluate` now consumes `trendChartView` + `trendHtfView` through `trendMergeViews`;
  `confidence`/`quality` are independent of `strength` and `isActive` is gated by `aligned`
  when MTF is on. Remains the sole producer of `TrendState` and sole toucher of `trendMemory`.
- `trendIsAligned` now reports real chart/HTF agreement (was trivially `true`).
- `docs/ROADMAP.md`: current marker moved to `0.1.4`.

### Notes
- **Non-repainting preserved:** the HTF read uses Core's frozen `ctxHtfValue` (`expr[1]`,
  `lookahead_off`, `gaps_off`; 1 HTF-bar lag; history == realtime). `trendHtfView` is called
  unconditionally for Pine v6 series safety; `trendMergeViews` ignores it when MTF is off or the
  HTF read is invalid, so the chart view stands alone.
- **Budgets:** total executable `request.security` = 1; Trend active HTF usage = 1 (≤ 2);
  project ≤ 8. No new public API surface; `TrendTimeframeView` stays internal-only.
- Next: M2.5 (`0.2.0`) — Module 02 integration; Trend API promoted to Stable; `TrendState` ABI locked.

## [0.1.3] - 2026-07-01

Module 02 (Trend Engine) — milestone **M2.3: chart-timeframe trend model**. The trend now
computes each bar; no signals/orders/plots/alerts. (The `v0.1.3` git tag is created manually
after review.)

### Added
- **`trendEvaluate(state, context) → TrendState`** (Experimental): the sole producer of the
  immutable `TrendState` and sole reader/updater of `KernelState.trendMemory` (R8/R9/R10).
  Wired into MAIN — computed every bar; result unused (seam for Modules 03+/09).
- **Pure accessors** (Experimental): `trendDirection`, `trendStrength`, `trendConfidence`,
  `trendQuality`, `trendPhase`, `trendIsActive`, `trendIsAligned`.
- **Internal helpers:** `trendChartView`, `trendClassifyStrength`, `trendClassifyPhase`. The
  chart-TF formula (fast/slow EMA + ATR separation, normalized) lives entirely here and is
  replaceable without changing the ABI (R2/R6/R7); raw indicator values never reach `TrendState`.
- **Type:** `TrendTimeframeView` (internal) — one timeframe's normalized view.

### Changed
- `PROJECT_VERSION` `0.1.2` → `0.1.3`.
- `trendMemory` initialization moved from `stateInit` to `trendEvaluate` (lazy) so that
  `trendEvaluate` is the **only** function that touches `TrendMemory` (strict R10).
- `MODULE PLACEHOLDERS` banner now covers modules 03–14 (02 implemented).
- `docs/ROADMAP.md`: current marker moved to `0.1.3`.

### Notes
- Chart timeframe only; multi-timeframe confirmation and independent confidence/quality arrive
  in M2.4. In M2.3 `confidence`/`quality` baseline to `strength` and `aligned` is trivially true.
- `ta.ema`/`ta.atr` are now used (chart-TF, `@internal`, non-repainting at bar close). The lone
  `request.security` remains inside `ctxHtfValue` and is still **not invoked** (count = 1).
- No repainting: chart-TF indicators on confirmed bars; deterministic; O(1) per bar.
- Next: M2.4 (`0.1.4`) — multi-timeframe synthesis + confidence/quality.

## [0.1.2] - 2026-07-01

Module 02 (Trend Engine) — milestone **M2.2: trend types & configuration**. Data model only;
no trend logic. (The `v0.1.2` git tag is created manually after review.)

### Added
- **Enums:** `TrendStrength {flat, weak, moderate, strong}`, `TrendPhase {absent, forming,
  established, weakening, reversing}`. Trend direction reuses Core `Direction` (no new enum).
- **Types:** `TrendConfig` (config snapshot), `TrendState` (immutable per-bar ABI —
  formula-agnostic normalized outputs), `TrendMemory` (minimal cross-bar memory).
- **Extensions:** `Config.trend` (`TrendConfig`, built by `cfgBuild`);
  `KernelState.trendMemory` (`TrendMemory`, initialized empty by `stateInit`).
- **Trend inputs** (group *02 · Trend Engine*, read only by `cfgBuild`): enable, use-HTF,
  HTF timeframe, fast/slow length, flat/strength thresholds, confirmation bars.
- **Validation** in `cfgValidate` (returns `ValidationResult` only; no `runtime.error`):
  `TREND-CFG-002` fast ≥ slow (fatal), `TREND-CFG-003` threshold out of `[0,1]` (recoverable).
- **Error-ID registry:** `TREND-CFG-001` (reserved, M2.4), `TREND-CFG-002`, `TREND-CFG-003`.

### Changed
- `PROJECT_VERSION` `0.1.1` → `0.1.2`.
- `inpEnableTrend` relabeled to a real "Enable Trend Engine" toggle (still default off).
- `docs/ROADMAP.md`: current marker moved to `0.1.2`.

### Notes
- No trend logic: **no EMA, no `ta.*`, no scoring, no signals, no plots, no orders**. The lone
  `request.security` remains inside `ctxHtfValue` and is not yet invoked.
- Per the approved architecture, `TrendState` is the immutable UDT **ABI** (not an enum); the
  M2.2-requested `{Bullish, Bearish, Neutral}` maps to Core `Direction {long, short, flat}`.
- Trend fast/slow length inputs are **formula-agnostic** lookback slots (labeled generically,
  not "EMA") per R2/R6/R7.
- Next: M2.3 (`0.1.3`) — chart-timeframe trend model.

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

[Unreleased]: https://example.com/compare/v0.2.0...HEAD
[0.2.0]: https://example.com/compare/v0.1.4...v0.2.0
[0.1.4]: https://example.com/compare/v0.1.3...v0.1.4
[0.1.3]: https://example.com/compare/v0.1.2...v0.1.3
[0.1.2]: https://example.com/compare/v0.1.1...v0.1.2
[0.1.1]: https://example.com/compare/v0.1.0...v0.1.1
[0.1.0]: https://example.com/compare/v0.0.4...v0.1.0
[0.0.4]: https://example.com/compare/v0.0.3...v0.0.4
[0.0.3]: https://example.com/compare/v0.0.2...v0.0.3
[0.0.2]: https://example.com/compare/v0.0.1...v0.0.2
[0.0.1]: https://example.com/releases/tag/v0.0.1
