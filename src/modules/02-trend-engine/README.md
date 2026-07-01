# Module 02 — Trend Engine

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** ✅ **Complete at `v0.2.0`** (M2.1–M2.5) · Public Trend API + `TrendState` ABI **frozen**
> ([ADR-0016](../../../docs/DECISIONS.md)) · **Depends on:** Module 01

## Overview

Classifies the prevailing **trend regime** (directional bias and strength) for
XAUUSD on M1/M5, producing a structured, explainable observation consumed by the
Signal Engine (Module 09).

## Specification notes

- Emits a deterministic trend state (e.g. bias direction + strength) with named
  contributing factors — no opaque logic ([ADR-0003](../../../docs/DECISIONS.md)).
- Higher-timeframe context is read **only** through Core's `ctxHtfValue(tf, expr)`
  (see the HTF API contract below) — never `request.security()` directly.
- Authoritative behavior tracked here and in
  [../../../docs/SPECIFICATION.md](../../../docs/SPECIFICATION.md).

## HTF API contract (FROZEN)

Module 02 (and every module) accesses higher-timeframe data through the single frozen Core
wrapper ([ADR-0015](../../../docs/DECISIONS.md), [API.md](../../../docs/API.md)):

- **`ctxHtfValue(tf, expr)` is the only approved wrapper around `request.security()`.**
  Modules must **never** call `request.security()` directly.
- `ctxHtfValue()` is owned **exclusively by Core**; Module 02 **consumes** it and may not
  duplicate or replace it.
- **Non-repainting:** `lookahead = barmerge.lookahead_off`, `gaps = barmerge.gaps_off`, and
  previous-bar confirmation `expr[1]` (last **closed** HTF bar; 1 HTF-bar lag; history == realtime).
- **Core owns symbol context:** `ctxHtfValue` always reads `syminfo.tickerid`; Module 02 never
  passes a symbol.
- Any change to `ctxHtfValue`'s behavior requires a **new ADR**. Trend `request.security`
  budget ≤ 2 (project ≤ 8), raised only by ADR.

## Data model (M2.2, v0.1.2)

Types + configuration only — no trend logic yet.

- **Enums:** `TrendStrength {flat, weak, moderate, strong}`, `TrendPhase {absent, forming,
  established, weakening, reversing}`. Trend **direction reuses Core `Direction {long, short,
  flat}`** (D4 — no separate trend-direction enum).
- **`TrendState`** — the immutable per-bar snapshot and public **ABI** (R6–R11): `direction`,
  `strengthBand`, `strength`, `confidence`, `quality`, `phase`, `aligned`, `isActive`,
  `barIndex`. Formula-agnostic (normalized outputs only; no EMA/ADX/etc.). Produced only by
  `trendEvaluate` (M2.3+).
- **`TrendConfig`** — trend configuration, nested as `Config.trend`, populated by `cfgBuild`.
- **`TrendMemory`** — minimal cross-bar memory, nested as `KernelState.trendMemory`, initialized
  empty by `stateInit`: `previousDirection/Phase/Strength`, `barsInTrend`, `reversalCounter`,
  `transitionCounter`.
- **Validation** (in `cfgValidate`, `ValidationResult` only): `TREND-CFG-001` HTF < chart when MTF
  on (fatal, active since `0.1.4`), `TREND-CFG-002` fast ≥ slow (fatal), `TREND-CFG-003` threshold
  out of `[0,1]` (recoverable), `TREND-CFG-004` flatThreshold > strengthThreshold (recoverable,
  since `0.2.0`).

## Chart-timeframe model (M2.3, v0.1.3)

- `trendEvaluate(state, context)` produces the immutable `TrendState` each bar (wired in MAIN;
  result unused — no signals/orders/plots). It is the sole producer of `TrendState` and the sole
  reader/updater of `KernelState.trendMemory` (R8/R9/R10; `trendMemory` is lazily initialized here).
- The chart-TF **formula** (fast/slow EMA separation scaled by ATR, normalized to `[0,1]`) lives
  entirely in the `@internal` helpers (`trendSignedStrength` since M2.4; decoded by `trendChartView`);
  it is **replaceable** without changing the ABI (R2/R6/R7). Raw EMA/ATR values never reach `TrendState` (R6).
- `trendClassifyStrength` maps strength → band; `trendClassifyPhase` derives the lifecycle phase
  from current values + the previous-bar memory (stateless — memory passed as parameters, R10).
- Pure accessors: `trendDirection/Strength/Confidence/Quality/Phase/IsActive/IsAligned`
  (**Stable and frozen** at `v0.2.0`, ADR-0016).

## Multi-timeframe synthesis (M2.4, v0.1.4)

- The trend **formula** is now a single signed-strength scalar in `[-1, 1]` (`@internal`
  `trendSignedStrength` — sign = direction, magnitude = strength). Evaluating one scalar lets the
  **same** expression run on the chart directly and on the HTF through a single `ctxHtfValue` call.
- Pipeline: `trendChartView` / `trendHtfView` → `trendMergeViews` → `trendEvaluate`
  (D-invariant #3 — chart-TF and HTF evaluation stay separated; HTF stays optional and replaceable).
  `trendDecodeSigned` turns a signed scalar into a `TrendTimeframeView` for both paths.
- `trendMergeViews`: chart drives `direction`/`strength`; the HTF **confirms** — `confidence` is the
  chart/HTF strength average when they `align`, and is penalized (`chart − htf`, floored at 0) when
  they disagree; `quality` averages `strength` and `confidence`. With MTF off or an invalid HTF read
  the chart view stands alone (`aligned = true`). `isActive` additionally requires `aligned` when MTF is on.
- **`ctxHtfValue` is the ONLY `request.security` wrapper and `trendHtfView` its ONLY call site**
  (HTF budget = 1 ≤ 2; project ≤ 8). Non-repainting is inherited from the frozen wrapper
  (`expr[1]`, `lookahead_off`, `gaps_off`; 1 HTF-bar lag). `trendHtfView` is invoked unconditionally
  (Pine v6 series safety); the merge decides whether to use it.
- `TrendTimeframeView` remains an **internal transport** only — never part of the public API; raw
  EMA/ATR/slope values never reach `TrendState` (R6).

## Finalization & API freeze (M2.5, v0.2.0)

Module 02 is **complete**. M2.5 is stabilization/diagnostics/documentation only — **no** behavior
change, no signals, no orders, no plots, no alerts.

- **API freeze (ADR-0016):** `trendEvaluate` and the seven accessors are promoted to `@stable` and
  **frozen**; the `TrendState` ABI is **locked** (normalized outputs only — raw
  EMA/ATR/slope/separation/volatility fields stay forbidden). Renaming or redesigning any of them
  requires a new ADR.
- **Diagnostics:** `trendSelfCheck` (`@internal`, debug-only) asserts the `TrendState` ABI
  invariants each bar (normalized ranges; `isActive` ⇒ directional and ≥ threshold) via `errAssert`.
  Read-only — it never constructs a `TrendState` or touches `TrendMemory`; wired in MAIN behind
  `config.debugEnabled` (no-op in production). Mirrors the Core self-tests.
- **Validation:** `TREND-CFG-004` (recoverable) enforces `flatThreshold ≤ strengthThreshold` so the
  strength bands stay ordered.
- **Ownership invariants (now permanent):** `trendEvaluate` is the sole producer of `TrendState`
  and the sole toucher of `KernelState.trendMemory`; `ctxHtfValue` remains the only
  `request.security` wrapper with a single executable call site.

## Contract freeze — dependency & future compatibility (v0.2.0)

Final architecture-freeze audit (documentation/validation only):

- **Dependency:** the Trend Engine depends **only on Core (Module 01)** — `BarContext`, `Config`
  (`cfgGet`), `ctxHtfValue`, `util*`, `err*` — plus market data. It consumes **no** other module,
  so **no circular dependency** is possible. Consumed **read-only** by Modules 03–14 via the frozen
  `trend*` accessors (see [MODULE_OWNERSHIP.md](../../../docs/MODULE_OWNERSHIP.md),
  [ARCHITECTURE.md](../../../docs/ARCHITECTURE.md)).
- **Performance (verified):** exactly **1** executable `request.security` (inside `ctxHtfValue`);
  exactly **1** active HTF read (`trendHtfView`); **O(1)** per bar (no loops in the module); no
  duplicated calculations (one shared `trendSignedStrength`); non-repainting; no hidden mutable
  state (all cross-bar state is in `KernelState.trendMemory`, touched only by `trendEvaluate`).
- **API sufficiency:** the eight-function public API covers all future consumers (Momentum,
  S/R, Market Structure, Liquidity, Order Blocks, FVG, Signal, Trade, Risk, Dashboard, Alerts,
  Optimization). No future module requires a **new** public Trend function.
- **Reserved additive surface (documented, NOT implemented):** two frozen `TrendState` fields have
  no accessor yet — `trendStrengthBand(ts) → TrendStrength` and `trendBarIndex(ts) → int`. If a
  consumer (e.g. Dashboard, Signal Engine) later needs them via the public surface, these are
  **purely additive** accessors over already-frozen fields (non-breaking, normal MINOR bump); they
  do not unfreeze or alter any existing API. Full rationale in
  [API.md](../../../docs/API.md).

## Design decisions

- Fast/slow length inputs are **formula-agnostic** lookback slots (labeled generically, not
  "EMA") to keep the algorithm replaceable (R2/R6/R7).
- `TrendMemory` holds only PREVIOUS-bar values + counters — never current `TrendState` values (R5).

## Research notes

- _None yet._ Candidate methods and references to be recorded here before coding.

## Testing notes

- Validated per [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md);
  verify non-repainting HTF usage.

## Performance observations

- _None yet._

## Implementation checklist (complete)

- [x] Define trend state representation (`TrendState`, M2.2).
- [x] Deterministic classification with named factors (chart-TF M2.3; MTF synthesis M2.4).
- [x] Non-repainting HTF handling (via Core `ctxHtfValue`, M2.4).
- [x] Expose observation to Signal Engine (frozen public API + `TrendState` ABI, M2.5/ADR-0016).
- [x] Documentation: SPECIFICATION, API, CHANGELOG, ROADMAP, DECISIONS synchronized.
