# Specification

**Authoritative specification** for XAU AI Institutional Scalper Pro. This
document is the single source of truth for *what* the system does. **No
implementation may occur without a corresponding specification here.** Code that
disagrees with this document is a defect in the code or the document — reconcile
before proceeding.

- **Status:** Draft (framework phase, `v0.0.1`)
- **Owner:** Project architect
- **Related:** [ARCHITECTURE.md](ARCHITECTURE.md), [ROADMAP.md](ROADMAP.md),
  [DECISIONS.md](DECISIONS.md)

---

## 1. Product definition

A production-grade **TradingView Strategy** (not an indicator) that trades
**XAUUSD** on the **M1** and **M5** timeframes using an institutional,
order-flow-aware methodology with a **deterministic, explainable** scoring
engine.

## 2. Scope

### In scope
- Single-file Pine Script **v6** strategy compiled by TradingView.
- Fourteen cooperating modules (Section 6).
- Deterministic scoring, non-repainting execution, explicit risk management.

### Out of scope
- Pine Script v5 or earlier.
- Black-box / opaque machine-learning inference on-chart.
- Multi-symbol portfolio management (initial horizon is XAUUSD only).
- External data feeds beyond what TradingView provides.

## 3. Target market and timeframes

- **Instrument:** XAUUSD (Gold vs. US Dollar).
- **Primary timeframes:** M1 and M5.
- **Session sensitivity:** intraday scalping; session handling is defined in
  [BACKTEST_PROTOCOL.md](BACKTEST_PROTOCOL.md) and Module 01.

## 4. Functional principles

The strategy must eventually support and integrate:

- Trend
- Momentum
- Market Structure
- Liquidity
- Order Flow
- Support / Resistance
- Risk Management
- AI Scoring (deterministic and explainable)

## 5. Non-functional requirements

| Requirement    | Definition |
|----------------|------------|
| Determinism    | Same inputs → same outputs. Scoring is reproducible and explainable. No `Math.random`-style nondeterminism. |
| No repainting  | Signals evaluated on confirmed bars; HTF access uses `lookahead_off`. Historical and realtime behavior match. |
| Maintainability| Modular, documented, single-responsibility functions; no duplication; no magic numbers. |
| Performance    | Stay within TradingView limits ([KNOWN_LIMITATIONS.md](KNOWN_LIMITATIONS.md)). |
| Explainability | Every score contribution is attributable to a named, documented factor. |

## 6. Module specifications (summary)

Detailed per-module specifications live in `../src/modules/<NN-name>/README.md`
and are authoritative for that module. Implementation order is defined in
[ROADMAP.md](ROADMAP.md).

| # | Module | Responsibility (summary) |
|---|--------|--------------------------|
| 01 | Core Framework | Foundational services, configuration, shared state/types. |
| 02 | Trend Engine | Directional bias / trend regime classification. |
| 03 | Momentum Engine | Momentum and strength assessment. |
| 04 | Support / Resistance Engine | Structural price levels. |
| 05 | Market Structure | Swing points, BOS / CHoCH classification. |
| 06 | Liquidity | Liquidity pools and sweeps. |
| 07 | Order Blocks | Institutional order block detection. |
| 08 | Fair Value Gap | Imbalance / FVG detection. |
| 09 | Signal Engine | Deterministic, explainable scoring and signal synthesis. |
| 10 | Trade Engine | Entry / exit orchestration (order placement). |
| 11 | Risk Manager | Position sizing, stops, exposure control. |
| 12 | Dashboard | On-chart informational panel. |
| 13 | Alerts | Alert condition routing and messaging. |
| 14 | Optimization | Parameter optimization helpers and diagnostics. |

## 7. Determinism and explainability (binding)

The signal engine (Module 09) MUST:

1. Produce a score that is a documented function of named factor contributions.
2. Be reproducible for identical inputs.
3. Expose each contribution for inspection (dashboard / logs).

No opaque logic is permitted. See [ADR-0003](DECISIONS.md).

## 8. Change control

Changes to this specification require an ADR in [DECISIONS.md](DECISIONS.md) when
they affect architecture, determinism, or module scope. The specification is
updated **before** the corresponding code.
