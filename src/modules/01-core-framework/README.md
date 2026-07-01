# Module 01 — Core Framework

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** In implementation — **M1.1 (`v0.0.2`) + M1.2 (`v0.0.3`) done**; M1.3–M1.4 pending · **Target version:** `0.1.0` · **Depends on:** none (foundation)

This document is the authoritative **architecture** for the Core Framework. It is
design only — no Pine code exists yet. Implementation is a separate, later,
approved task. Cross-cutting decisions are recorded as ADRs in
[../../../docs/DECISIONS.md](../../../docs/DECISIONS.md) (ADR-0006 – ADR-0014).

Confirmed design decisions:
- **Function naming:** subsystem prefixes — `cfg`, `ctx`, `util`, `state`, `err`, `log`.
- **Shared types:** native Pine v6 `enum` + user-defined `type` (UDT).
- **Config validation:** hybrid — fail-fast for structural errors, safe-degrade for scalars.

---

## 1. Responsibilities

Core owns foundational, **non-trading** services:

- Central configuration: read inputs once, validate, expose an immutable snapshot.
- Shared types/enums reused by all modules (no duplication).
- Per-bar context: time, session state, bar confirmation, new-bar detection.
- Non-repainting primitives (confirmed-bar contract + HTF access wrapper).
- Persistent state kernel (single source of mutable global state).
- Deterministic utility helpers (clamp, safe division, normalize, number validity).
- Layered error handling + validation.
- Diagnostics/logging seam (gated), feeding Dashboard (12) and Optimization (14).

Core owns **no** trading decisions, indicators, entries, or exits.

## 2. Internal architecture

Subsystems, in dependency order (leaf → composite):

```
util  (pure helpers)
  └─ log  (diagnostics primitive)
       └─ err  (uses log)
            └─ cfg  (uses util, err, log)
                 └─ ctx  (uses util, cfg)
                      └─ state  (uses cfg, ctx)
                           └─ core orchestrator (init + per-bar seam)
```

Each subsystem is a clearly-bannered section in the file.

## 3. Public APIs (signatures & contracts — no bodies)

Stability levels per the [API Stability Policy](../../../docs/API.md): **S**table,
**E**xperimental, **I**nternal.

| Fn | Signature | Returns | Level |
|----|-----------|---------|-------|
| `cfgBuild` | `cfgBuild()` | `Config` | E |
| `cfgGet` | `cfgGet()` | `Config` | S |
| `ctxBuild` | `ctxBuild(Config)` | `BarContext` | E |
| `ctxIsConfirmedBar` | `ctxIsConfirmedBar()` | `bool` | S |
| `ctxIsNewBar` | `ctxIsNewBar()` | `bool` | S |
| `ctxSessionState` | `ctxSessionState(Config)` | `SessionState` | E |
| `ctxHtfValue` | `ctxHtfValue(symbol, timeframe, expr)` | `<series>` | S |
| `utilClamp` | `utilClamp(value, lo, hi)` | `float` | S |
| `utilSafeDiv` | `utilSafeDiv(num, den, fallback)` | `float` | S |
| `utilNormalize` | `utilNormalize(value, lo, hi)` | `float` (0..1) | S |
| `utilIsValidNumber` | `utilIsValidNumber(value)` | `bool` | S |
| `stateInit` | `stateInit(Config)` | `KernelState` | E |
| `stateUpdate` | `stateUpdate(BarContext)` | `void` | E |
| `errRaiseFatal` | `errRaiseFatal(message)` | `never` | S |
| `errWarn` | `errWarn(message)` | `void` | S |
| `errApplyValidation` | `errApplyValidation(results)` | `void` | E |
| `logWrite` | `logWrite(LogLevel, message)` | `void` | S |
| `logDrain` | `logDrain()` | `array<LogEntry>` | E |
| `coreInit` | `coreInit()` | `void` | S |
| `coreOnBar` | `coreOnBar()` | `BarContext` | S |

`ctxHtfValue` enforces `barmerge.lookahead_off` and returns confirmed values.
`coreOnBar` is the per-bar seam that modules 02–14 consume.

## 4. Internal helper functions

Leaf helpers live inside their subsystem, defined before the public functions that
call them (Pine define-before-use). All `@internal`, single-responsibility:
`cfgValidate(Config) → array<ValidationResult>`, `ctxComputeSession(Config) → SessionState`,
`logShouldEmit(LogLevel) → bool`, `errFormat(severity, message) → string`,
`stateGet() → KernelState`.

## 5. Global state

Exactly **one** persistent object: `var KernelState` (ADR-0009). No scattered `var`
scalars. Fields: cached `Config`, bar counters, `lastProcessedBarIndex`, error latch,
bounded diagnostics ring-buffer. Mutated only through `state*` functions; initialized on
`barstate.isfirst`.

## 6. Constants

Structural/config constants only, `UPPER_SNAKE_CASE`, documented inline. Extends the
existing identity constants with Core defaults (session bounds, diagnostics buffer cap,
default log level). No trading thresholds. No magic numbers.

## 7. Enumerations (native v6 `enum`, Core-owned, shared)

| Enum | Members | Reused by |
|------|---------|-----------|
| `Direction` | `long`, `short`, `flat` | 02, 05, 09, 10 |
| `SessionState` | `inactive`, `active` | 02–10, 13 (extensible to killzones) |
| `LogLevel` | `error`, `warn`, `info`, `debug` | all (diagnostics) |
| `TimeframeClass` | `m1`, `m5`, `other` | Core (primary-TF validation) |
| `ValidationSeverity` | `ok`, `warning`, `fatal` | Core (hybrid validation) |

## 8. Configuration philosophy

Inputs are declared **once** and consumed **only** by `cfgBuild`. Modules receive an
immutable `Config` snapshot and never read `input.*` directly (ADR-0008). Configuration
is separated from logic; defaults are conservative; every literal is an input or named
constant.

## 9. Shared utilities

Pure, deterministic, side-effect-free `util*` helpers. Composition over duplication:
modules reuse these rather than re-derive them. These are foundational math/guards, not
indicators.

## 10. Error handling (hybrid — ADR-0010)

- **Fatal** (structural/contradictory config, invalid primary timeframe): `errRaiseFatal`
  → `runtime.error()` with a descriptive message; strategy refuses to run.
- **Recoverable** (out-of-range scalar): clamp to nearest valid + `errWarn` diagnostic; continue.
- **Runtime anomalies** (e.g. division by zero): defensive `util*` helpers return fallbacks; never crash.
- **Assertions** (`errAssert`): active only in debug mode; no production halts.

Severity is classified via `ValidationSeverity`; `errApplyValidation` enforces the policy.

## 11. Validation strategy

Three layers: (a) declarative input bounds (`minval`/`maxval`) at the input site;
(b) cross-field `cfgValidate` at init producing `ValidationResult[]`; (c) debug-mode
invariant checks (confirmed-bar / non-repainting assertions). Validation runs once at init.

## 12. Data flow

```
input.* → cfgBuild (validate → errApplyValidation) → immutable Config
        → stateInit caches Config
per bar → coreOnBar → ctxBuild → BarContext → stateUpdate
        → BarContext handed to modules 02–14
diagnostics → bounded ring buffer → logDrain → Dashboard (12) / Optimization (14)
```

## 13. Module dependencies

Core depends on **nothing**; every other module depends on Core. Dependencies flow upward
only; no cycles. Authoritative contract: [../../../docs/MODULE_OWNERSHIP.md](../../../docs/MODULE_OWNERSHIP.md).

## 14. Non-repainting guarantees (ADR-0011)

- All HTF access goes through `ctxHtfValue` (`lookahead = barmerge.lookahead_off`, confirmed
  values). Raw `request.security` in modules is prohibited.
- Signal-affecting logic evaluates on `ctxIsConfirmedBar()` (`barstate.isconfirmed`).
- `calc_on_every_tick = false` for decisions (set in the strategy declaration).
- Contract: modules obtain time/session/HTF data via Core primitives only.

## 15. Performance considerations

Per-bar context computed once and cached; modules read cached values. `request.security`
calls consolidated in Core and minimized. O(1) per-bar context; no heavy loops. Diagnostics
gated behind the debug flag. Bound by the project [Performance Budget](../../../docs/SPECIFICATION.md#performance-budget).

## 16. Memory considerations

Single `KernelState` (fewer `var` slots). Diagnostics buffer is a **bounded ring buffer**
(fixed cap) respecting array limits. No unbounded collections; reuse over allocate.

## 17. Pine Script limitations

Single-file compilation (no includes) — Core is a section, not an import. `request.security`
count cap — consolidate. `var` init timing — initialize on first bar. Enum/UDT are v6 features
(available). String/label limits — bound diagnostics output. See
[../../../docs/KNOWN_LIMITATIONS.md](../../../docs/KNOWN_LIMITATIONS.md).

## 18. Future extension points

`Config` and `KernelState` UDTs are extended field-by-field per module. `coreInit` /
`coreOnBar` are the hook seams for modules 02–14. `Direction`/`SessionState` are shared
vocabulary. The diagnostics buffer is the integration point for Dashboard/Optimization.

## 19. Testing strategy

Compile check (v6, clean). Non-repainting validation per
[../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md) §6 (historical vs.
realtime parity). Determinism: identical inputs → identical `Config`/`BarContext`. No Pine unit
framework → a **debug diagnostics harness** (debug-gated labels/table) verifies `util*` edge
cases (clamp bounds, `safeDiv` zero denominator, normalize 0..1) and context values.

## 20. Acceptance criteria

For the eventual implementation:

- Compiles clean, Pine v6, no errors.
- All inputs validated; invalid config fails fast, out-of-range degrades with warning.
- Single `KernelState`; deterministic init; no scattered globals.
- Non-repainting primitives present; modules have no direct `request.security`/`input.*` access.
- Diagnostics bounded and debug-gated; within the Performance Budget.
- No trading logic (Core is foundation only).

For **this** milestone (design): all 20 items documented, standards defined, ADRs recorded,
docs cross-linked and consistent, zero code changes.

## Design decisions

Recorded as ADR-0006 – ADR-0014 in [../../../docs/DECISIONS.md](../../../docs/DECISIONS.md).

## Research notes

- _None yet._

## Performance observations

- _None yet._ Design targets the project [Performance Budget](../../../docs/SPECIFICATION.md#performance-budget).

## Error-ID registry

Stable identifiers stamped into `LogEntry.errorId` and used by fatal/recoverable
validation. Format `CORE-<AREA>-<NNN>` (see `err` subsystem, ADR-0010).

| ID | Constant | Meaning |
|----|----------|---------|
| `CORE-CFG-001` | `ERR_CFG_UNSUPPORTED_TF` | Primary timeframe not supported (fatal). |
| `CORE-CFG-002` | `ERR_CFG_INVALID_SESSION` | Malformed / inverted session window (fatal). |
| `CORE-CFG-003` | `ERR_CFG_DIAG_CAP_RANGE` | Diagnostics buffer cap out of range (recoverable → clamp). |
| `CORE-STATE-001` | `ERR_STATE_REINIT` | Attempted re-initialization (fatal). |
| `CORE-ASSERT-001` | `ERR_ASSERT_FAILED` | Debug assertion failed (fatal, debug-gated). |
| _(empty)_ | `ERR_NONE` | No associated error (non-error log entries). |

## Implementation checklist

- [x] **M1.1 (`v0.0.2`)** — Enums (5) + UDTs (8, incl. `LogEntry` with `moduleId`/`errorId`),
      structural constants, `MODULE_ID_*`, `DIAG_BUFFER_CAP`, empty subsystem scaffolding, no-op MAIN.
- [x] **M1.2 (`v0.0.3`)** — `util` (clamp, safeDiv, normalize, isValidNumber, roundToTick; + internal minutesOfDay, formatFloat).
- [x] **M1.2 (`v0.0.3`)** — `log` subsystem (bounded O(1) ring buffer, level/debug gating).
- [x] **M1.2 (`v0.0.3`)** — `err` subsystem (fatal/warn/assert, hybrid validation) + Error-ID registry.
- [ ] **M1.3 (`v0.0.4`)** — `cfg` subsystem (build, validate, get; immutable `Config`).
- [ ] **M1.3 (`v0.0.4`)** — `ctx` subsystem (build, confirmed-bar, new-bar, session, timeframe).
      `ctxHtfValue` **reserved** (docs only) until first module needing `request.security`.
- [ ] **M1.4 (`v0.1.0`)** — `state` subsystem (init, update, get).
- [ ] **M1.4 (`v0.1.0`)** — `core` orchestrator (`coreInit`, `coreOnBar`) + MAIN integration.
- [ ] **M1.4 (`v0.1.0`)** — debug diagnostics harness; finalize docs (API, CHANGELOG, ROADMAP).
