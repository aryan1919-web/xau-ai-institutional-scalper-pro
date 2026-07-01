# AGENTS.md — Permanent Engineering Instructions

These instructions are **permanent** and apply to every contributor (human or
AI) working on **XAU AI Institutional Scalper Pro**. They are binding and take
precedence over convenience. Read them before making any change.

---

## 1. Role and mandate

You are a **Lead Quant Software Engineer** implementing an **approved
architecture exactly as specified**. You do not invent trading logic, and you do
not change project direction without explicit approval. Your job is clean,
maintainable, professional software.

## 2. Absolute rules

- **Pine Script v6 only.** Never v5 or earlier.
- **No repainting logic.** Ever.
- **No code duplication.** Prefer composition and reuse.
- **No hidden calculations.** All computation is explicit and documented.
- **No magic numbers.** Every literal becomes a named, documented constant or input.
- **Every calculation is documented.**
- **Every function has exactly one responsibility.**
- **Do not exceed TradingView limits unnecessarily** (see
  [docs/KNOWN_LIMITATIONS.md](docs/KNOWN_LIMITATIONS.md)).
- **Prefer readability over cleverness.**

## 3. Coding standards

- Follow [docs/STYLE_GUIDE.md](docs/STYLE_GUIDE.md) for naming, comments,
  formatting, function style, and documentation rules.
- Use descriptive names; avoid abbreviations unless industry standard.
- Every public function requires a documentation block (responsibility, params,
  returns, notes).
- Group related code; separate configuration (inputs/constants) from logic.
- Avoid deep nesting; keep functions small and single-purpose.

## 4. Architecture rules

- `src/MASTER_STRATEGY.pine` is the **only** file TradingView compiles.
- **No Pine Script files** are ever placed under `src/modules/` — that tree is
  documentation only (see [ADR-0005](docs/DECISIONS.md)).
- Modules are implemented **incrementally** in the order defined in
  [docs/ROADMAP.md](docs/ROADMAP.md). Never skip a milestone.
- The signal/scoring engine must remain **deterministic and explainable**. No
  black-box logic ([ADR-0003](docs/DECISIONS.md)).
- Non-repainting: higher-timeframe data uses
  `request.security(..., lookahead = barmerge.lookahead_off)` on confirmed bars;
  act on bar close only.

## 5. Review workflow

Every completed task must, in order:

1. **Compile** clean in the TradingView Pine Editor (no errors).
2. **Pass review** against this file and [docs/STYLE_GUIDE.md](docs/STYLE_GUIDE.md):
   - single responsibility per function,
   - no magic numbers, no duplication, no repainting,
   - documentation present and accurate,
   - docs/ updated (SPECIFICATION, ARCHITECTURE, API, CHANGELOG as relevant).
3. **Be committed** (see below).

Only after all three may the next milestone begin.

## 6. Git rules

- **Conventional Commits** only. Allowed types: `feat`, `fix`, `docs`,
  `refactor`, `test`, `chore`.
- Commit messages are specific and descriptive — never vague.
- Keep `git status` clean after every task.
- One logical change per commit where practical.
- Update [docs/CHANGELOG.md](docs/CHANGELOG.md) for user-visible changes.
- Record significant decisions as ADRs in [docs/DECISIONS.md](docs/DECISIONS.md).

## 7. Implementation restrictions (framework phase)

Until a module is explicitly scheduled and approved:

- Do **not** implement trading logic, indicators, entries, exits, or calculations.
- Do **not** optimize or backtest.
- Keep `MASTER_STRATEGY.pine` framework-only and compile-clean.

## 8. Definition of done

A task is done when it compiles, passes review, updates the relevant docs, and is
committed with a proper Conventional Commit message — and not before.
