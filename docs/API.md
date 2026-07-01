# API Reference

Catalog of **public functions** exposed by `../src/MASTER_STRATEGY.pine`. This
document grows alongside the code: every public function must be listed here with
its responsibility, parameters, return value, and notes (see
[STYLE_GUIDE.md](STYLE_GUIDE.md) for documentation rules).

- **Status:** Module 01 in progress (`v0.0.3`, M1.2 — utility / logging / validation implemented).
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

**Internal helpers (M1.2, not cross-module callable):** `utilMinutesOfDay`,
`utilFormatFloat`, `logLevelRank`, `logShouldEmit`, `logFormat`, `errFormat`.
`logLevelRank` is a single-source severity ranking that keeps `logShouldEmit` free of
duplicated `switch` logic (no-duplication rule).

### Planned — configuration / context / state / lifecycle

| Function | Signature | Returns | Level | Milestone |
|----------|-----------|---------|-------|-----------|
| `cfgBuild` | `cfgBuild()` | `Config` | Experimental | M1.3 |
| `cfgGet` | `cfgGet()` | `Config` | Stable | M1.3 |
| `ctxBuild` | `ctxBuild(config)` | `BarContext` | Experimental | M1.3 |
| `ctxIsConfirmedBar` | `ctxIsConfirmedBar()` | `bool` | Stable | M1.3 |
| `ctxIsNewBar` | `ctxIsNewBar()` | `bool` | Stable | M1.3 |
| `ctxSessionState` | `ctxSessionState(config)` | `SessionState` | Experimental | M1.3 |
| `ctxHtfValue` | `ctxHtfValue(symbol, timeframe, expr)` | `<series>` | Stable | **Reserved** — deferred until the first module needing `request.security` (≥ M02) |
| `stateInit` | `stateInit(config)` | `KernelState` | Experimental | M1.4 |
| `stateUpdate` | `stateUpdate(context)` | `void` | Experimental | M1.4 |
| `coreInit` | `coreInit()` | `void` | Stable | M1.4 |
| `coreOnBar` | `coreOnBar()` | `BarContext` | Stable | M1.4 |

Planned internal helpers: `cfgValidate`, `ctxComputeSession`, `stateGet`.

## Other module public functions

Populated as each subsequent module is implemented, grouped by module.

| Module | Function | Since | Summary |
|--------|----------|-------|---------|
| 02–14 | — | — | Not yet designed. |
