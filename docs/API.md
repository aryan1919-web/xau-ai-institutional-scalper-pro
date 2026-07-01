# API Reference

Catalog of **public functions** exposed by `../src/MASTER_STRATEGY.pine`. This
document grows alongside the code: every public function must be listed here with
its responsibility, parameters, return value, and notes (see
[STYLE_GUIDE.md](STYLE_GUIDE.md) for documentation rules).

- **Status:** Module 01 in progress (`v0.0.4`, M1.3 — configuration & context implemented).
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

> **Milestone note (M1.2):** Config (M1.3) and `KernelState` (M1.4) are not yet wired,
> so the logging / validation functions take their gating context — a `Diagnostics`
> handle, the `threshold` level, and `debugEnabled` — as **explicit parameters**. The
> M1.4 lifecycle will supply these from the cached `Config` / `KernelState`, so later
> modules call the simpler forms. These signatures are therefore `@stable` in intent
> but may gain convenience wrappers at M1.4.

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

### Planned — state / lifecycle

| Function | Signature | Returns | Level | Milestone |
|----------|-----------|---------|-------|-----------|
| `ctxHtfValue` | `ctxHtfValue(symbol, timeframe, expr)` | `<series>` | Stable | **Reserved** — deferred until the first module needing `request.security` (≥ M02) |
| `stateInit` | `stateInit(config)` | `KernelState` | Experimental | M1.4 |
| `stateUpdate` | `stateUpdate(context)` | `void` | Experimental | M1.4 |
| `coreInit` | `coreInit()` | `void` | Stable | M1.4 |
| `coreOnBar` | `coreOnBar()` | `BarContext` | Stable | M1.4 |

Planned internal helper: `stateGet`.

## Other module public functions

Populated as each subsequent module is implemented, grouped by module.

| Module | Function | Since | Summary |
|--------|----------|-------|---------|
| 02–14 | — | — | Not yet designed. |
