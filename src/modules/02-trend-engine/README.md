# Module 02 — Trend Engine

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** In implementation — M2.1 (`v0.1.1`) + M2.2 (`v0.1.2`) done · **Target version:** `0.2.0` · **Depends on:** Module 01

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
- **Validation** (in `cfgValidate`, `ValidationResult` only): `TREND-CFG-002` fast ≥ slow (fatal),
  `TREND-CFG-003` threshold out of `[0,1]` (recoverable). `TREND-CFG-001` (HTF ≥ chart) reserved for M2.4.

## Design decisions

- Fast/slow length inputs are **formula-agnostic** lookback slots (labeled generically, not
  "EMA") to keep the algorithm replaceable (R2/R6/R7).

## Research notes

- _None yet._ Candidate methods and references to be recorded here before coding.

## Testing notes

- Validated per [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md);
  verify non-repainting HTF usage.

## Performance observations

- _None yet._

## Future implementation checklist

- [ ] Define trend state representation.
- [ ] Deterministic classification with named factors.
- [ ] Non-repainting HTF handling.
- [ ] Expose observation to Signal Engine.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
