# API Reference

Catalog of **public functions** exposed by `../src/MASTER_STRATEGY.pine`. This
document grows alongside the code: every public function must be listed here with
its responsibility, parameters, return value, and notes (see
[STYLE_GUIDE.md](STYLE_GUIDE.md) for documentation rules).

- **Status:** Module 01 (Core Framework) **COMPLETE** (`v0.1.0`).
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

### Implemented — context (`ctx`) · Since 0.0.4

| Function | Signature | Returns | Level |
|----------|-----------|---------|-------|
| `ctxBuild` | `ctxBuild(cfg)` | `BarContext` | Experimental |
| `ctxIsConfirmedBar` | `ctxIsConfirmedBar()` | `bool` | Stable |
| `ctxIsNewBar` | `ctxIsNewBar()` | `bool` | Stable |

Strictly non-repainting (current-bar builtins only; **zero `request.security`**). Session
state is exposed through `BarContext.session` (built by the internal `ctxComputeSession`),
so a separate public `ctxSessionState` is not needed.

**Internal helpers (not cross-module callable):** `utilMinutesOfDay`, `utilFormatFloat`,
`logLevelRank`, `logShouldEmit`, `logFormat`, `errFormat` (M1.2); `cfgValidate`,
`ctxComputeSession`, `ctxTimeframe` (M1.3). `logLevelRank` is a single-source severity
ranking that keeps `logShouldEmit` free of duplicated `switch` logic (no-duplication rule).

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

| Function | Signature | Returns | Level | Status |
|----------|-----------|---------|-------|--------|
| `ctxHtfValue` | `ctxHtfValue(symbol, timeframe, expr)` | `<series>` | Stable | Reserved — deferred until the first module needing `request.security` (≥ M02) |

## Other module public functions

Populated as each subsequent module is implemented, grouped by module.

| Module | Function | Since | Summary |
|--------|----------|-------|---------|
| 02–14 | — | — | Not yet designed. |
