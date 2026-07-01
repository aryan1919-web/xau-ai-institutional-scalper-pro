# Backtest Protocol

The **official testing methodology** for XAU AI Institutional Scalper Pro. It
defines how the strategy is validated so results are honest, reproducible, and
comparable across versions. **No backtests are run during the framework phase**
(`v0.0.1`) — this protocol governs testing once trading logic exists.

- **Status:** Draft (framework phase).
- **Related:** [SPECIFICATION.md](SPECIFICATION.md),
  [KNOWN_LIMITATIONS.md](KNOWN_LIMITATIONS.md), [ROADMAP.md](ROADMAP.md).

---

## 1. Objectives

- Validate that a module or release behaves per specification.
- Detect repainting, look-ahead bias, and non-determinism.
- Produce comparable, reproducible performance metrics.

## 2. Test instrument and timeframes

- **Instrument:** XAUUSD.
- **Timeframes:** M1 and M5 (tested independently).
- **Data source:** TradingView chart data for the account/exchange in use
  (record the exact data feed used, as feeds differ).

## 3. Data ranges

- **In-sample (development):** a fixed historical window used while building.
- **Out-of-sample (validation):** a disjoint, later window never used during
  development.
- Record exact start/end timestamps for every run. TradingView's available
  history depends on plan and timeframe — document what was actually loaded.

## 4. Broker-emulator assumptions

Because TradingView fills orders via the broker emulator, every run must fix and
record:

- **Initial capital.**
- **Commission** (type and value).
- **Slippage** (ticks).
- **Order fill assumptions** (e.g. `process_orders_on_close`, tick vs. bar-close
  evaluation).
- **Pyramiding** (default `0`).

See broker-emulator caveats in [KNOWN_LIMITATIONS.md](KNOWN_LIMITATIONS.md).

## 5. Session handling

- Define and record trading sessions and any session filters.
- Scalping is session-sensitive; report metrics per session where relevant.
- Account for gaps, rollover, and low-liquidity periods.

## 6. Repainting and look-ahead checks (mandatory)

A run is invalid if any of these fail:

1. **Confirmed-bar evaluation:** signals computed on `barstate.isconfirmed`.
2. **`lookahead_off`:** all `request.security` calls use
   `barmerge.lookahead_off`.
3. **Historical vs. realtime parity:** bar-replay / forward behavior matches
   historical behavior for the same bars.
4. **Determinism:** identical inputs and data produce identical results.

## 7. Metrics to record

- Net profit, profit factor, max drawdown.
- Win rate, average win/loss, expectancy.
- Number of trades, average trade duration.
- Per-session and per-timeframe breakdowns.
- Score-factor attribution (once Module 09 exists), to confirm explainability.

## 8. Acceptance criteria

Defined per milestone in the module's `../src/modules/<NN-name>/README.md`.
A release candidate must additionally:

- Pass all repainting / look-ahead / determinism checks (§6).
- Show consistent behavior between in-sample and out-of-sample within documented
  tolerances.
- Stay within TradingView platform limits.

## 9. Reproducibility

Every reported result must include: version/commit, instrument, timeframe, data
range, emulator assumptions, and session configuration. Results without this
metadata are not accepted.
