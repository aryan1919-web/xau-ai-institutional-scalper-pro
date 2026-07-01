# XAU AI Institutional Scalper Pro

Production-grade **TradingView strategy** for **XAUUSD** intraday scalping on the
**M1** and **M5** timeframes, written in **Pine Script v6**.

This is a long-term software project, not a throwaway script. It is engineered
for maintainability over years: strict specifications, incremental milestones,
deterministic and explainable logic, and disciplined git hygiene.

> **Current status: `v0.0.1` — framework only.**
> The strategy skeleton compiles and does nothing yet. No indicators, no
> entries, no exits, no calculations. Trading logic is added one module at a
> time per [docs/ROADMAP.md](docs/ROADMAP.md).

---

## Overview

- **Type:** TradingView Strategy (not an indicator).
- **Market:** XAUUSD (Gold / US Dollar).
- **Timeframes:** M1, M5.
- **Language:** Pine Script **v6 only**.
- **Design principles:** deterministic, explainable, non-repainting, no magic
  numbers, one responsibility per function.

The strategy is being built as fourteen cooperating modules — trend, momentum,
market structure, liquidity, order flow, support/resistance, signal scoring,
trade execution, risk management, dashboard, alerts, and optimization. See
[docs/SPECIFICATION.md](docs/SPECIFICATION.md) and
[docs/ARCHITECTURE.md](docs/ARCHITECTURE.md).

## Installation

1. Open the [TradingView](https://www.tradingview.com/) chart for `XAUUSD`.
2. Open the **Pine Editor**.
3. Copy the full contents of [`src/MASTER_STRATEGY.pine`](src/MASTER_STRATEGY.pine).
4. Paste into the Pine Editor and click **Add to chart**.

TradingView compiles **only** `src/MASTER_STRATEGY.pine`. Everything under
[`src/modules/`](src/modules/) is documentation, not compiled code.

## Repository structure

```
.
├── README.md                 Project overview (this file)
├── AGENTS.md                 Permanent engineering + AI contributor rules
├── CONTRIBUTING.md           Contribution workflow
├── LICENSE                   MIT License
├── .editorconfig             Formatting standardization
├── .gitattributes            Line-ending normalization
├── .gitignore                Ignored paths
├── .github/                  PR + issue templates
├── docs/                     Authoritative documentation set
│   ├── README.md             Documentation index
│   ├── SPECIFICATION.md      Authoritative project specification
│   ├── ARCHITECTURE.md       System design, data flow, dependency graph
│   ├── ROADMAP.md            Milestones and version timeline
│   ├── CHANGELOG.md          Semantic version history
│   ├── DECISIONS.md          Architectural Decision Records
│   ├── API.md                Public function catalog (future)
│   ├── STYLE_GUIDE.md        Coding + documentation conventions
│   ├── BACKTEST_PROTOCOL.md  Official testing methodology
│   └── KNOWN_LIMITATIONS.md  TradingView / Pine Script limits
└── src/
    ├── MASTER_STRATEGY.pine  The only compiled Pine file (framework only)
    └── modules/              Documentation-only, one folder per module
```

## Development workflow

Work proceeds in strict, incremental milestones. Every completed task must:

1. **Compile** clean in the TradingView Pine Editor.
2. **Pass review** against [AGENTS.md](AGENTS.md) and
   [docs/STYLE_GUIDE.md](docs/STYLE_GUIDE.md).
3. **Be committed** using [Conventional Commits](https://www.conventionalcommits.org/).

Milestones are never skipped. See [CONTRIBUTING.md](CONTRIBUTING.md).

## Goals

- Institutional-grade, maintainable codebase with a multi-year horizon.
- Deterministic, fully explainable signal scoring — no black-box logic.
- Non-repainting behavior throughout.
- Clean separation of configuration from logic.

## Non-goals

- Not a "YouTube indicator" or hype project.
- No black-box machine-learning inference on-chart.
- No Pine Script v5 (or earlier).
- No premature optimization or backtesting before the logic exists.

## License

Released under the [MIT License](LICENSE).
