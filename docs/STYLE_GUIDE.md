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
- **Functions / locals:** `camelCase` (e.g. `projectVersion`, `swingHigh`).
- **Types / UDT (when introduced):** `PascalCase`.
- Names describe intent, not implementation.

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

- Every **public function** has a documentation block:
  ```
  // functionName(params)
  //   Responsibility : ...
  //   Parameters     : ...
  //   Returns        : ...
  //   Note           : repainting / limits / caveats
  ```
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

## 8. Review checklist (quick)

- [ ] v6 only, compiles clean.
- [ ] No magic numbers, no duplication, no hidden calculations.
- [ ] One responsibility per function; documented.
- [ ] No repainting; deterministic decisions.
- [ ] Docs updated (API / CHANGELOG / module README as relevant).
