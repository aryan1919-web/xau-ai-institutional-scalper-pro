# Style Guide

Coding and documentation conventions for XAU AI Institutional Scalper Pro. These
rules are binding and complement [AGENTS.md](../AGENTS.md).

- **Language:** Pine Script **v6 only**.
- **Guiding principle:** readability over cleverness.

---

## 1. Naming

- Use **descriptive names**; avoid abbreviations unless industry standard
  (e.g. `atr`, `rsi`, `htf`, `ohlc` are acceptable; invented shorthand is not).
- **Constants:** `UPPER_SNAKE_CASE` (e.g. `PROJECT_VERSION`, `MODULE_COUNT`).
- **Input group titles:** `G_<NN>_<NAME>` constants (e.g. `G_02_TREND`).
- **Inputs:** `inp` prefix + `camelCase` (e.g. `inpEnableTrend`).
- **Public functions:** `<subsystemPrefix><Action>` in `camelCase`
  (e.g. `ctxIsNewBar`, `utilClamp`) — see the prefix registry below ([ADR-0006](DECISIONS.md)).
- **Locals:** `camelCase`.
- **Types / UDT:** `PascalCase` (e.g. `Config`, `BarContext`, `KernelState`).
- **Enum members & UDT fields:** `camelCase` (e.g. `Direction.long`, `cfg.debugEnabled`).
- Names describe intent, not implementation.

### Subsystem / module prefix registry

Public functions are prefixed by the subsystem (Core) or module that owns them. This
keeps a single-file, 14-module codebase collision-free and self-describing.

| Prefix | Owner |
|--------|-------|
| `cfg`, `ctx`, `util`, `state`, `err`, `log` | 01 Core Framework (its subsystems) |
| `trend` | 02 Trend Engine |
| `mom` | 03 Momentum Engine |
| `sr` | 04 Support / Resistance |
| `struct` | 05 Market Structure |
| `liq` | 06 Liquidity |
| `ob` | 07 Order Blocks |
| `fvg` | 08 Fair Value Gap |
| `sig` | 09 Signal Engine |
| `trade` | 10 Trade Engine |
| `risk` | 11 Risk Manager |
| `dash` | 12 Dashboard |
| `alert` | 13 Alerts |
| `opt` | 14 Optimization |

## 2. Constants and configuration

- **No magic numbers.** Every literal that carries meaning becomes a named,
  documented constant or an `input.*`.
- **Separate configuration from logic.** Inputs and constants are declared in
  their dedicated sections, not interleaved with calculations.
- Group input controls with `group=` and explain them with `tooltip=`.

## 3. Functions

- **One responsibility per function.** If a function needs the word "and" to
  describe it, split it.
- Keep functions small; avoid deep nesting (prefer early, flat structure).
- Prefer **composition over duplication** — extract shared helpers rather than
  copy-paste.
- Public (reusable) functions are cataloged in [API.md](API.md).

## 4. Documentation and comments

- Every **public function** has a documentation block that includes a stability
  annotation (`@stable` / `@experimental` / `@internal`, see §9):
  ```
  // functionName(params)   @stable
  //   Responsibility : one sentence, single responsibility.
  //   Parameters     : name — type — meaning.
  //   Returns        : type — meaning.
  //   Determinism    : deterministic? any state dependence?
  //   Errors         : fatal / recoverable / none.
  //   Note           : repainting / limits / caveats.
  ```
- Every **enum and UDT** documents each member/field.
- Every non-obvious **calculation is documented** with *why*, not just *what*.
- Use section banners to group related code:
  ```
  // =============================================================================
  //  SECTION TITLE
  // -----------------------------------------------------------------------------
  //  Short description of the section's responsibility.
  // =============================================================================
  ```
- Mark pending work with `// TODO(<module>): ...`.
- No hidden calculations — nothing meaningful happens without a comment nearby.

## 5. Formatting

- UTF-8, **LF** line endings, final newline, no trailing whitespace
  (enforced by [`.editorconfig`](../.editorconfig)).
- **4-space** indentation in Pine files.
- One statement per line; align related declarations for readability where it
  helps.
- Keep lines reasonably short; wrap long `input.*` / function calls across lines.

## 6. Pine Script v6 conventions

- Always begin with `//@version=6` and exactly one `strategy(...)` declaration.
- **No repainting:** higher-timeframe access uses
  `request.security(..., lookahead = barmerge.lookahead_off)`; evaluate on
  confirmed bars (`barstate.isconfirmed`); act on bar close.
- Declare explicit type qualifiers/types for constants (`const int`, `const string`).
- Prefer `strategy.*` order calls over manual state where the platform provides it.
- Respect platform limits — see [KNOWN_LIMITATIONS.md](KNOWN_LIMITATIONS.md).

## 7. Determinism

- No nondeterministic constructs in decision logic.
- Scoring contributions must be explicit and attributable
  ([SPECIFICATION.md](SPECIFICATION.md) §7, [ADR-0003](DECISIONS.md)).

## 8. Function and section ordering

**Function ordering** (within a subsystem/module): private/leaf helpers first, then the
public functions that use them (Pine requires define-before-use). Group by subsystem;
within a subsystem, order by dependency (leaf → composite).

**Section ordering** (whole file) — canonical and enforced in review:

```
license + //@version  ->  header doc  ->  strategy()  ->  CONSTANTS  ->  ENUMS  ->  TYPES
->  INPUTS  ->  Core: util -> log -> err -> cfg -> ctx -> state -> orchestrator
->  Modules 02–08 (analysis)  ->  09 Signal  ->  11 Risk  ->  10 Trade
->  12 Dashboard  ->  13 Alerts  ->  14 Optimization  ->  MAIN  ->  TODO
```

Inputs are declared before any function; only `cfgBuild` consumes them
([ADR-0008](DECISIONS.md)).

## 9. API stability levels

Every public function declares its stability in the doc block ([ADR-0012](DECISIONS.md)):

- `@stable` — committed contract; breaking it needs an ADR + version bump.
- `@experimental` — provisional; may change between minor versions without an ADR.
- `@internal` — local to its subsystem/module; never called across modules.

Only `@stable`/`@experimental` are cross-module callable. Promotion
`@experimental → @stable` requires an ADR. Full policy and catalog: [API.md](API.md).

## 10. Performance budget (quick reference)

Binding project caps — mirror of [SPECIFICATION.md](SPECIFICATION.md#performance-budget)
([ADR-0013](DECISIONS.md)). Exceeding any requires an ADR.

| `request.security` | arrays | labels | boxes | lines | tables | diagnostics | `var`/`varip` | per-bar |
|---|---|---|---|---|---|---|---|---|
| ≤ 8 | ≤ 32 | ≤ 100 | ≤ 100 | ≤ 100 | ≤ 4 | ≤ 100 | ≤ 16 | O(1); loop ≤ 64 |

Label/box/line caps are a **shared pool** across drawing modules (04–08, 12); reuse objects.

## 11. Review checklist (quick)

- [ ] v6 only, compiles clean.
- [ ] No magic numbers, no duplication, no hidden calculations.
- [ ] One responsibility per function; documented (with stability annotation).
- [ ] Subsystem/module prefix + correct section/function ordering.
- [ ] No repainting; deterministic decisions.
- [ ] Within the performance budget.
- [ ] Docs updated (API / CHANGELOG / module README / OWNERSHIP as relevant).
