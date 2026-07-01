# API Reference

Catalog of **public functions** exposed by `../src/MASTER_STRATEGY.pine`. This
document grows alongside the code: every public function must be listed here with
its responsibility, parameters, return value, and notes (see
[STYLE_GUIDE.md](STYLE_GUIDE.md) for documentation rules).

- **Status:** Framework phase (`v0.0.1`).
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

## Placeholder utilities (v0.0.1)

These are framework stubs with no trading logic. Signatures are provisional.

### isFrameworkOnly()
- Responsibility : Report whether this build is framework-only (no logic).
- Parameters     : none.
- Returns        : `bool` — always `true` in `v0.0.1`.
- Determinism    : deterministic; no state.
- Notes          : Self-description helper; removed or repurposed once modules land.

### projectVersion()
- Responsibility : Expose the compiled project version string.
- Parameters     : none.
- Returns        : `string` — value of `PROJECT_VERSION`.
- Determinism    : deterministic; no state.
- Notes          : Accessor only.

### noop()
- Responsibility : Explicit no-operation seam.
- Parameters     : none.
- Returns        : `na` (void by design).
- Determinism    : deterministic; no state.
- Notes          : Placeholder pattern for future void utilities.

## Module 01 — Core Framework (planned API)

Architecture-approved public surface for the Core Framework. Signatures are the
design contract; bodies are implemented in a later, separately-approved task. See
[../src/modules/01-core-framework/README.md](../src/modules/01-core-framework/README.md).

| Function | Signature | Returns | Level | Responsibility |
|----------|-----------|---------|-------|----------------|
| `cfgBuild` | `cfgBuild()` | `Config` | Experimental | Read inputs, validate, return an immutable config snapshot. |
| `cfgGet` | `cfgGet()` | `Config` | Stable | Accessor for the cached config in `KernelState`. |
| `ctxBuild` | `ctxBuild(config)` | `BarContext` | Experimental | Compute the per-bar context. |
| `ctxIsConfirmedBar` | `ctxIsConfirmedBar()` | `bool` | Stable | True on confirmed bars (`barstate.isconfirmed`). |
| `ctxIsNewBar` | `ctxIsNewBar()` | `bool` | Stable | True on the first tick of a new bar. |
| `ctxSessionState` | `ctxSessionState(config)` | `SessionState` | Experimental | Current session state. |
| `ctxHtfValue` | `ctxHtfValue(symbol, timeframe, expr)` | `<series>` | Stable | Non-repainting HTF read (`barmerge.lookahead_off`, confirmed). |
| `utilClamp` | `utilClamp(value, lo, hi)` | `float` | Stable | Clamp a value to `[lo, hi]`. |
| `utilSafeDiv` | `utilSafeDiv(num, den, fallback)` | `float` | Stable | Division guarded against zero/`na`. |
| `utilNormalize` | `utilNormalize(value, lo, hi)` | `float` | Stable | Scale a value to `0..1`. |
| `utilIsValidNumber` | `utilIsValidNumber(value)` | `bool` | Stable | Finite, non-`na` check. |
| `stateInit` | `stateInit(config)` | `KernelState` | Experimental | Initialize the persistent kernel on the first bar. |
| `stateUpdate` | `stateUpdate(context)` | `void` | Experimental | Advance persistent state for the current bar. |
| `errRaiseFatal` | `errRaiseFatal(message)` | `never` | Stable | Halt via `runtime.error` with a descriptive message. |
| `errWarn` | `errWarn(message)` | `void` | Stable | Record a non-fatal diagnostic warning. |
| `errApplyValidation` | `errApplyValidation(results)` | `void` | Experimental | Apply the hybrid validation policy (fatal → halt; scalar → clamp+warn). |
| `logWrite` | `logWrite(level, message)` | `void` | Stable | Append a diagnostic entry (gated by debug flag + level). |
| `logDrain` | `logDrain()` | `array<LogEntry>` | Experimental | Return buffered diagnostics for Dashboard/Optimization. |
| `coreInit` | `coreInit()` | `void` | Stable | First-bar setup of config + kernel state. |
| `coreOnBar` | `coreOnBar()` | `BarContext` | Stable | Per-bar seam that modules 02–14 consume. |

**Internal helpers** (not cross-module callable): `cfgValidate`, `ctxComputeSession`,
`logShouldEmit`, `errFormat`, `stateGet`.

## Other module public functions

Populated as each subsequent module is implemented, grouped by module.

| Module | Function | Since | Summary |
|--------|----------|-------|---------|
| 02–14 | — | — | Not yet designed. |
