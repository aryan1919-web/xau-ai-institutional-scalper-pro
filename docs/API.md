# API Reference

Catalog of **public functions** exposed by `../src/MASTER_STRATEGY.pine`. This
document grows alongside the code: every public function must be listed here with
its responsibility, parameters, return value, and notes (see
[STYLE_GUIDE.md](STYLE_GUIDE.md) for documentation rules).

- **Status:** Module 01 frozen (`v0.1.0`). Module 02 complete — `v0.2.0` (Trend API + `TrendState` ABI frozen, ADR-0016).
- **Scope:** Only functions intended for reuse across modules are "public" and
  documented here. Private, single-use helpers are documented inline.

---

## API Stability Policy

Every public function declares one of three visibility levels in its doc block, via
an annotation. This is the project's API contract (see [ADR-0012](DECISIONS.md)).

| Level | Annotation | Contract |
|-------|------------|----------|
| **Stable** | `@stable` | Committed public contract. Breaking changes require an ADR **and** a version bump. Any module may rely on it long-term. |
| **Experimental** | `@experimental` | Public but provisional. Signature/behavior may change between minor versions **without** an ADR. Use with caution; not a long-term contract. |
| **Internal** | `@internal` | Private to its subsystem/module. **Never** called across modules. No stability guarantee. Documented inline, not cataloged here. |

Rules:
- Only `@stable` and `@experimental` functions are cross-module callable; `@internal` is local.
- Promotion `@experimental → @stable` requires an ADR.
- Demotion or breaking change to a `@stable` function requires an ADR and a version bump.

## Documentation format

Each entry uses this shape:

```
### functionName(param1, param2)   [Stable | Experimental | Internal]
- Responsibility : one sentence, single responsibility.
- Parameters     : name — type — meaning.
- Returns        : type — meaning.
- Determinism    : deterministic? any state dependence?
- Errors         : fatal / recoverable / none.
- Notes          : repainting considerations, limits, caveats.
```

## Module 01 — Core Framework

Implemented incrementally across milestones M1.1–M1.4. *Implemented* signatures are
live in [`../src/MASTER_STRATEGY.pine`](../src/MASTER_STRATEGY.pine); *Planned* rows are
the design contract. See
[../src/modules/01-core-framework/README.md](../src/modules/01-core-framework/README.md).

> **Design note (reference-passing):** the single `KernelState` (ADR-0009) lives in
> MAIN and is threaded to functions **by reference**; there is no hidden global that
> functions read. Accessors/mutators therefore take an explicit handle (`state`, or a
> `Diagnostics` + gating context) rather than being parameter-less. This is the final
> Module 01 shape; a parameter-less `cfgGet()` is intentionally not provided.

### Implemented — utility (`util`) · Since 0.0.3

| Function | Signature | Returns | Level |
|----------|-----------|---------|-------|
| `utilIsValidNumber` | `utilIsValidNumber(value)` | `bool` | Stable |
| `utilClamp` | `utilClamp(value, lo, hi)` | `float` | Stable |
| `utilSafeDiv` | `utilSafeDiv(num, den, fallback)` | `float` | Stable |
| `utilNormalize` | `utilNormalize(value, lo, hi)` | `float` (0..1) | Stable |
| `utilRoundToTick` | `utilRoundToTick(price)` | `float` | Stable |

### Implemented — logging (`log`) · Since 0.0.3

| Function | Signature | Returns | Level |
|----------|-----------|---------|-------|
| `logWrite` | `logWrite(diag, level, moduleId, errorId, message, threshold, debugEnabled)` | `void` | Stable |
| `logDrain` | `logDrain(diag)` | `array<LogEntry>` | Experimental |
| `logClear` | `logClear(diag)` | `void` | Experimental |

### Implemented — error / validation (`err`) · Since 0.0.3

| Function | Signature | Returns | Level |
|----------|-----------|---------|-------|
| `errRaiseFatal` | `errRaiseFatal(errorId, message)` | never (halts) | Stable |
| `errWarn` | `errWarn(diag, errorId, message, threshold, debugEnabled)` | `void` | Stable |
| `errAssert` | `errAssert(condition, errorId, message, debugEnabled)` | `void` | Stable |
| `errApplyValidation` | `errApplyValidation(diag, results, threshold, debugEnabled)` | `void` | Experimental |

### Implemented — configuration (`cfg`) · Since 0.0.4

| Function | Signature | Returns | Level |
|----------|-----------|---------|-------|
| `cfgBuild` | `cfgBuild(diag)` | `Config` | Experimental |
| `cfgGet` | `cfgGet(state)` | `Config` | Stable |

`cfgBuild` is the only reader of `input.*` (ADR-0008); `Config` is immutable after build
(by convention). `cfgGet` takes a `KernelState` handle until the M1.4 lifecycle provides
a parameter-less form.

### Implemented — context (`ctx`) · Since 0.0.4 (HTF primitive Since 0.1.1)

| Function | Signature | Returns | Level | Since |
|----------|-----------|---------|-------|-------|
| `ctxBuild` | `ctxBuild(cfg)` | `BarContext` | Experimental | 0.0.4 |
| `ctxIsConfirmedBar` | `ctxIsConfirmedBar()` | `bool` | Stable | 0.0.4 |
| `ctxIsNewBar` | `ctxIsNewBar()` | `bool` | Stable | 0.0.4 |
| `ctxHtfValue` | `ctxHtfValue(tf, expr)` | `float` | Stable | 0.1.1 |

Per-bar context is strictly non-repainting (current-bar builtins only). **`ctxHtfValue` is the
single sanctioned higher-timeframe read** and the **only** place `request.security` appears in
the entire strategy (ADR-0011): it fixes `lookahead = barmerge.lookahead_off`,
`gaps = barmerge.gaps_off`, and reads `expr[1]` so only closed HTF bars are used (1 HTF-bar
lag by design). **Core owns symbol context** — `ctxHtfValue` always reads the chart symbol
(`syminfo.tickerid`), so modules never pass a symbol. Modules consume `ctxHtfValue`, never
`request.security` directly. Session
state is exposed through `BarContext.session` (built by the internal `ctxComputeSession`),
so a separate public `ctxSessionState` is not needed.

**Internal helpers (not cross-module callable):** `utilMinutesOfDay`, `utilFormatFloat`,
`logInitBuffer`, `logLevelRank`, `logShouldEmit`, `logFormat`, `errFormat`; `cfgValidate`,
`ctxComputeSession`, `ctxTimeframe`, `ctxIsHigherTimeframe` (HTF ≥ chart validation, Since 0.1.1);
`stateGet`, `stateReset` (debug). `logLevelRank` is a single-source severity ranking, and
`logInitBuffer` a single-source ring-buffer allocator, that keep the public functions free of
duplicated logic (no-duplication rule).

### HTF API Contract (FROZEN)

`ctxHtfValue(tf, expr)` is the **only** approved wrapper around `request.security()`. This
contract is frozen ([ADR-0015](DECISIONS.md)); changing its behavior requires a new ADR.

- Modules must **never** call `request.security()` directly. **All** higher-timeframe reads
  pass through `ctxHtfValue()`.
- The implementation of `ctxHtfValue()` is owned **exclusively by Core**. Future modules may
  **consume** it but may not **duplicate or replace** it.
- **Non-repainting guarantee** — the value is stable for a confirmed bar and identical in
  history and realtime, via:
  - `lookahead = barmerge.lookahead_off` (no future leakage),
  - `gaps = barmerge.gaps_off` (carry last known HTF value; no `na` holes),
  - `expr[1]` — previous-bar confirmation: only the **last closed** HTF bar is read (1 HTF-bar lag).
- **Core owns symbol context** — `ctxHtfValue` always reads the chart symbol
  (`syminfo.tickerid`); modules never pass a symbol.

### Implemented — kernel state (`state`) · Since 0.1.0

| Function | Signature | Returns | Level |
|----------|-----------|---------|-------|
| `stateInit` | `stateInit(config, diag)` | `KernelState` | Experimental |
| `stateUpdate` | `stateUpdate(state, context)` | `void` | Experimental |

Internal: `stateGet(state)`, `stateReset(state)` (debug only). `KernelState` is the
single mutable global (ADR-0009), created in MAIN and threaded by reference. `stateInit`
sizes the diagnostics buffer from `config.diagBufferCap`, finalizes `primaryTfClass`, and
enforces `requireSupportedTf` (fatal `CORE-CFG-001` on an unsupported chart timeframe).

### Implemented — lifecycle (`core`) · Since 0.1.0

| Function | Signature | Returns | Level |
|----------|-----------|---------|-------|
| `coreInit` | `coreInit()` | `KernelState` | Stable |
| `coreOnBar` | `coreOnBar(state)` | `BarContext` | Stable |

`coreInit()` runs once (MAIN `var` initializer); `coreOnBar(state)` runs every bar and
returns the `BarContext` seam for modules 02-14. In debug mode `coreInit` runs a
diagnostics harness exercising util / log / err / cfg / ctx.

### Reserved

None. (`ctxHtfValue` was implemented in `0.1.1` — see the context table above.)

## Module 02 — Trend Engine

Module 02 **complete** (M2.5, `v0.2.0`). The `trend*` public API and the `TrendState` ABI are
**Stable and FROZEN** — changes require a new ADR ([ADR-0016](DECISIONS.md)). `TrendState` is the
immutable per-bar ABI (R6–R11); trend direction reuses Core `Direction` (no separate
trend-direction enum).

### Public API (Stable since 0.2.0 · frozen — ADR-0016)

| Function | Signature | Returns | Role |
|----------|-----------|---------|------|
| `trendEvaluate` | `trendEvaluate(state, context)` | `TrendState` | Sole producer of `TrendState`; sole reader/updater of `KernelState.trendMemory` (R8/R10). Wired in MAIN. |
| `trendDirection` | `trendDirection(ts)` | `Direction` | pure accessor |
| `trendStrength` | `trendStrength(ts)` | `float` 0..1 | pure accessor |
| `trendConfidence` | `trendConfidence(ts)` | `float` 0..1 | pure accessor |
| `trendQuality` | `trendQuality(ts)` | `float` 0..1 | pure accessor |
| `trendPhase` | `trendPhase(ts)` | `TrendPhase` | pure accessor |
| `trendIsActive` | `trendIsActive(ts)` | `bool` | pure accessor |
| `trendIsAligned` | `trendIsAligned(ts)` | `bool` | pure accessor (chart/HTF agreement; `true` when MTF off / HTF invalid) |

**Internal helpers:** `trendSignedStrength`, `trendDecodeSigned`, `trendChartView`, `trendHtfView`,
`trendMergeViews` (Since 0.1.4); `trendClassifyStrength`, `trendClassifyPhase` (Since 0.1.3);
`trendSelfCheck` (Since 0.2.0, debug-only ABI assertions). The trend **formula lives entirely in
these helpers** — one signed-strength scalar (fast/slow EMA scaled by ATR) is evaluated on the
chart directly and on the HTF through Core's `ctxHtfValue`, then `trendMergeViews` synthesizes
direction/strength/confidence/quality/alignment. The formula is replaceable without changing
`TrendState` (R2/R7); raw indicator values never reach `TrendState` (R6). `trendHtfView` holds the
**only** `ctxHtfValue` call site in the strategy (HTF budget = 1); `request.security` remains
exclusively inside `ctxHtfValue`.

### Enums (Since 0.1.2)

| Enum | Members |
|------|---------|
| `TrendStrength` | `flat`, `weak`, `moderate`, `strong` (normalized strength band) |
| `TrendPhase` | `absent`, `forming`, `established`, `weakening`, `reversing` |

### Types (Since 0.1.2)

| Type | Role |
|------|------|
| `TrendConfig` | Trend configuration snapshot, nested as `Config.trend` (built by `cfgBuild`). Config values only. |
| `TrendState` | Immutable per-bar trend snapshot (the ABI, **frozen at 0.2.0** — ADR-0016): `direction:Direction`, `strengthBand:TrendStrength`, `strength/confidence/quality:float(0..1)`, `phase:TrendPhase`, `aligned:bool`, `isActive:bool`, `barIndex:int`. Formula-agnostic (no implementation values). |
| `TrendMemory` | Minimal cross-bar memory, nested as `KernelState.trendMemory` (built by `stateInit`): `previousDirection`, `previousPhase`, `previousStrength`, `barsInTrend`, `reversalCounter`, `transitionCounter`. |

### Configuration & validation
- **Inputs** (group *02 · Trend Engine*, read only by `cfgBuild`): `inpEnableTrend`,
  `inpTrendUseMtf`, `inpTrendHtf`, `inpTrendFastLength`, `inpTrendSlowLength`,
  `inpTrendFlatThreshold`, `inpTrendStrengthThreshold`, `inpTrendConfirmBars`.
- **Validation** (in `cfgValidate`; returns `ValidationResult` only): `TREND-CFG-001` HTF < chart
  timeframe when MTF is on (fatal, active since `0.1.4`); `TREND-CFG-002` fast ≥ slow (fatal);
  `TREND-CFG-003` threshold out of `[0,1]` (recoverable clamp+warn); `TREND-CFG-004` flatThreshold
  > strengthThreshold (recoverable clamp+warn, since `0.2.0`).
- **Diagnostics:** in debug mode, `trendSelfCheck` asserts the `TrendState` ABI invariants
  (normalized ranges + output consistency) once per bar via `errAssert` (read-only; no output).

### Dependency & consumption (frozen)
- **Depends only on Core (Module 01):** `BarContext`, `Config`/`cfgGet`, `ctxHtfValue`, `util*`,
  `err*`. It consumes **no** other module ([MODULE_OWNERSHIP.md](MODULE_OWNERSHIP.md),
  [ARCHITECTURE.md](ARCHITECTURE.md) §4–5). No circular dependency is possible: every Trend input
  is a Core primitive or market data.
- **Consumed by** the analysis/decision/presentation modules (03–14) purely by calling the frozen
  accessors on the per-bar `TrendState` — a read-only, one-directional data flow (Trend → Signal
  Engine → …). Consumers never mutate `TrendState` or `TrendMemory`.

### Future compatibility — reserved additive surface (post-freeze, **not implemented**)
The frozen `TrendState` ABI already carries two fields with **no** dedicated accessor yet:
`strengthBand` (`TrendStrength`) and `barIndex` (`int`). If a future module needs them through the
public surface (e.g. Dashboard rendering the discrete band, or the Signal Engine bucketing by
band), the following **additive** accessors may be introduced later:

| Reserved accessor | Would return | Rationale |
|-------------------|--------------|-----------|
| `trendStrengthBand(ts)` | `TrendStrength` | discrete band already in the ABI; convenience over `ts.strengthBand`. |
| `trendBarIndex(ts)` | `int` | snapshot freshness; convenience over `ts.barIndex`. |

These are **purely additive** over already-frozen ABI fields — they do **not** unfreeze or change
any existing API, require **no** new `TrendState` field, and would land as a normal MINOR bump.
The freeze (ADR-0016) forbids renaming/redesigning existing APIs and adding raw implementation
fields; it does **not** forbid additive accessors over frozen fields. No other future consumer
(Modules 03–14) requires a new public Trend function — the eight-function API is otherwise
sufficient.

## Other module public functions

Populated as each subsequent module is implemented, grouped by module.

| Module | Function | Since | Summary |
|--------|----------|-------|---------|
| 03–14 | — | — | Not yet designed. |
