# Known Limitations

TradingView and Pine Script impose hard limits and behavioral caveats that shape
the architecture. Every module must be designed to respect them. Exact numeric
limits vary by TradingView plan and can change over time — **verify against the
current [Pine Script v6 reference](https://www.tradingview.com/pine-script-docs/)
before relying on a specific figure.**

- **Status:** Living document. Update when limits are hit or platform changes.

---

## 1. Execution model limits

- A script runs once per bar (historical) and per tick (realtime) if
  `calc_on_every_tick = true`; otherwise on bar close. Design for bar-close
  evaluation to avoid intrabar nondeterminism.
- Total script runtime and compilation are bounded; overly heavy per-bar work can
  cause timeouts. Keep per-bar computation lean.
- Loop iteration counts are bounded; avoid unbounded or large loops.

## 2. Drawing object limits

- `line`, `label`, `box`, and `table`/`polyline` objects are capped
  (e.g. `max_lines_count`, `max_labels_count`, `max_boxes_count`, with platform
  maximums). The Dashboard (Module 12) and any level drawing (Modules 04–08)
  must **reuse and cap** objects rather than create unboundedly.
- Excess objects are silently dropped (oldest first), which can hide state — cap
  deliberately and document the cap.

## 3. Array / collection limits

- Arrays (and maps/matrices) have a maximum element count and contribute to
  memory limits. Bound the history you retain (e.g. cap swing/order-block stores).
- Prefer fixed-size, explicitly-managed buffers over ever-growing collections.

## 4. `request.security()` limitations

- There is a maximum number of `request.*` calls per script; consolidate
  higher-timeframe requests.
- **Repainting risk:** always use `lookahead = barmerge.lookahead_off` and
  reference confirmed values. `barmerge.lookahead_on` leaks future data and is
  prohibited (see [ADR-0003](DECISIONS.md), [STYLE_GUIDE.md](STYLE_GUIDE.md)).
- Higher-timeframe values update only when the HTF bar closes; account for this
  in signal timing.
- **Single sanctioned wrapper (Since 0.1.1):** all `request.security` in this strategy
  goes through Core's `ctxHtfValue(symbol, tf, expr)` (ADR-0011). It fixes
  `lookahead = barmerge.lookahead_off`, `gaps = barmerge.gaps_off`, and reads `expr[1]`
  (the last **closed** HTF bar) — a deliberate **1 HTF-bar lag** that guarantees
  historical == realtime. No module may call `request.security` directly, and the project
  budget (≤ 8 calls; Trend ≤ 2) may only be raised via an ADR.

## 5. Repainting caveats

- Realtime bars are unconfirmed and can change until close; evaluate signals on
  `barstate.isconfirmed`.
- Functions using future information, `lookahead_on`, or unconfirmed HTF data
  cause repainting. All decision logic must be repaint-free and validated per
  [BACKTEST_PROTOCOL.md](BACKTEST_PROTOCOL.md) §6.

## 6. Broker-emulator limitations

- Backtests fill orders via TradingView's **broker emulator**, an approximation:
  - Fills assume price passed through the order level within the bar; intrabar
    fill order is estimated, not exact.
  - No real spread/partial-fill/latency modeling beyond configured
    commission/slippage.
  - Only the chart timeframe's OHLC is used for fills unless intrabar features
    are enabled.
- Live/forward results will differ from backtests; treat emulator output as
  indicative, not guaranteed.

## 7. Session limitations

- Session/timezone handling depends on the symbol's exchange calendar; sessions,
  holidays, and rollovers affect bar formation.
- Intraday history depth is limited by plan and timeframe, constraining how far
  back M1/M5 backtests can reach.
- Gaps and low-liquidity periods can distort scalping signals; filter and
  document per [BACKTEST_PROTOCOL.md](BACKTEST_PROTOCOL.md) §5.

## 8. Design implications

- Reuse drawing objects; cap collections; consolidate `request.security` calls.
- Evaluate on confirmed bars; never use look-ahead.
- Keep per-bar work lean to stay within runtime limits.
- Document any limit encountered and the mitigation chosen.
