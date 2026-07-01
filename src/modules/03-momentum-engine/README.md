# Module 03 — Momentum Engine

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** In implementation — M3.1 (`v0.2.1`) done · **Target version:** `0.3.0` · **Depends on:** Module 01 (Core) + Module 02 (Trend, read-only)

## Overview

Assesses **momentum and strength** of price movement to complement trend context,
producing a structured, explainable observation for the Signal Engine (Module 09).

## Specification notes

- Deterministic momentum reading with named factor contributions.
- Evaluated on confirmed bars to avoid repainting.
- Higher-timeframe context is read **only** through Core's `ctxHtfValue(tf, expr)`
  ([ADR-0015](../../../docs/DECISIONS.md)) — never `request.security()` directly (arrives in M3.3).
- Consumes `TrendState` **read-only** (Trend remains the single source of truth for trend
  direction); never mutates it. Depends only on Core + Trend.
- Authoritative behavior tracked here and in
  [../../../docs/SPECIFICATION.md](../../../docs/SPECIFICATION.md).

## Data model (M3.1, v0.2.1)

Types + configuration only — no momentum logic yet.

- **Enums:** `MomentumStrength {flat, weak, moderate, strong}`, `MomentumPhase {neutral,
  accelerating, sustained, decelerating, exhausted}`. Momentum **direction reuses Core
  `Direction {long, short, flat}`** (no separate momentum-direction enum).
- **`MomentumState`** — the immutable per-bar snapshot and public **ABI**: `direction`,
  `strengthBand`, `strength`, `acceleration`, `confidence`, `quality`, `phase`, `aligned`
  (chart/HTF), `trendAligned` (vs Trend), `isActive`, `barIndex`. Formula-agnostic (normalized
  outputs only; no EMA/RSI/etc.). Produced only by `momentumEvaluate` (M3.2+).
- **`MomentumConfig`** — momentum configuration, nested as `Config.momentum`, populated by `cfgBuild`.
- **`MomentumMemory`** — minimal cross-bar memory, nested as `KernelState.momentumMemory`,
  declared but **not yet used**: `previousDirection/Phase/Strength/Acceleration`, `barsInMomentum`,
  `reversalCounter`, `transitionCounter`. Touched only by `momentumEvaluate` (M3.2+).
- **`MomentumTimeframeView`** — internal transport (never public); kept **separate** from
  `TrendTimeframeView` by design (see Design decisions).
- **Validation** (centralized in `cfgValidate`, `ValidationResult` only): `MOM-CFG-001` HTF ≥ chart
  when MTF on (fatal), `MOM-CFG-002` fast ≥ slow (fatal), `MOM-CFG-003` threshold out of `[0,1]`
  (recoverable), `MOM-CFG-004` flatThreshold > strengthThreshold (recoverable), `MOM-CFG-005`
  accelLength < 1 (recoverable clamp). No separate momentum validator (single source of truth).

## Design decisions

- Fast/slow/acceleration length inputs are **formula-agnostic** lookback slots (labeled
  generically, not "EMA/RSI") to keep the algorithm replaceable.
- **`MomentumTimeframeView` is kept separate from `TrendTimeframeView`.** They are structurally
  identical today but semantically distinct (different owners) and are expected to diverge
  (momentum may carry acceleration). A generic shared type would dilute ownership and risk the
  Module 02 freeze for negligible DRY benefit; the tie-breaker "prefer separate types" applies.
- `MomentumMemory` holds only PREVIOUS-bar values + counters — never current `MomentumState` values.

## Research notes

- _None yet._

## Testing notes

- Validated per [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md).

## Performance observations

- _None yet._

## Future implementation checklist

- [ ] Define momentum representation.
- [ ] Deterministic computation with named factors.
- [ ] Confirmed-bar evaluation.
- [ ] Expose observation to Signal Engine.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
