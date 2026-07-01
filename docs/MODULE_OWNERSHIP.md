# Module Ownership Matrix

The **authoritative dependency contract** for the entire project. For every module
it defines what the module is responsible for, the data it **owns**, the data it may
**consume**, the data it is **prohibited** from touching, which modules **may call**
it, and which modules it **may never depend upon**.

This matrix is binding. It is consistent with the layering and dependency graph in
[ARCHITECTURE.md](ARCHITECTURE.md) (foundation → analysis → decision → execution →
presentation; dependencies flow **upward only**, no cycles) and with the ownership ADR
([ADR-0014](DECISIONS.md)). Any change to a contract row requires an ADR.

## Governing rules

- **01 Core** depends on nothing and is callable by all.
- **Analysis (02–08)** depend only on Core — except **06, 07, 08** may also consume
  **05 (Market Structure)**. Analysis modules never depend on decision/execution/
  presentation modules (09–14), and never issue orders.
- **09 Signal** consumes analysis (02–08); it decides but never places orders.
- **11 Risk** gates **10 Trade**; **10** is the *only* module that issues `strategy.*` orders.
- **12 Dashboard** / **13 Alerts** are read-only consumers; they influence no decision.
- **14 Optimization** reads diagnostics only and is never in the trade path.
- **Data prohibited** = anything a module neither owns nor is contracted to consume.

Legend: "Core" = Module 01. Order/position state = `strategy.*` position, open orders, PnL.

---

## 01 — Core Framework
- **Responsibilities:** configuration, shared types/enums, per-bar context, non-repainting primitives, persistent state kernel, utilities, error handling, diagnostics seam.
- **Data owned:** `Config`, `KernelState`, `BarContext`, all shared enums, diagnostics ring buffer.
- **Data consumed:** raw `input.*`, market data (OHLCV, time), HTF data (via its own wrapper).
- **Data prohibited:** trading signals, order/position state, indicator values.
- **May be called by:** all modules (02–14).
- **May never depend upon:** any other module (foundation).

## 02 — Trend Engine
- **Responsibilities:** classify trend regime (bias + strength). **Complete & frozen at `v0.2.0`**
  ([ADR-0016](DECISIONS.md)).
- **Data owned:** `TrendState` — the immutable per-bar ABI (normalized outputs only: `direction`,
  `strengthBand`, `strength`, `confidence`, `quality`, `phase`, `aligned`, `isActive`, `barIndex`).
  Produced solely by `trendEvaluate`; cross-bar `TrendMemory` is touched only by `trendEvaluate`.
- **Data consumed:** Core (`BarContext`, `Config`, `ctxHtfValue`, utilities), market data.
- **Data prohibited:** order/position state; other analysis modules' internals.
- **May be called by:** 09 Signal; 12 Dashboard (read-only) — via the frozen `trend*` accessors.
- **May never depend upon:** 03–14.

## 03 — Momentum Engine
- **Responsibilities:** assess momentum / strength.
- **Data owned:** `MomentumObservation`.
- **Data consumed:** Core, market data.
- **Data prohibited:** order/position state; other analysis modules' internals.
- **May be called by:** 09 Signal; 12 Dashboard (read-only).
- **May never depend upon:** 02, 04–14.

## 04 — Support / Resistance Engine
- **Responsibilities:** detect structural support/resistance levels.
- **Data owned:** `SrLevel` set (drawing objects within the shared pool).
- **Data consumed:** Core, market data.
- **Data prohibited:** order/position state; other analysis modules' internals.
- **May be called by:** 09 Signal; 10 Trade (level context); 12 Dashboard (read-only).
- **May never depend upon:** 02, 03, 05–14.

## 05 — Market Structure
- **Responsibilities:** swing points; BOS / CHoCH classification.
- **Data owned:** `SwingPoint` store, `StructureState`.
- **Data consumed:** Core, market data.
- **Data prohibited:** order/position state; other analysis modules' internals.
- **May be called by:** 06, 07, 08, 09; 12 Dashboard (read-only).
- **May never depend upon:** 02, 03, 04, 06–14.

## 06 — Liquidity
- **Responsibilities:** liquidity pools; sweep detection.
- **Data owned:** `LiquidityPool` set, sweep events.
- **Data consumed:** Core; **05 Market Structure** (swings/structure); market data.
- **Data prohibited:** order/position state; 02/03/04/07/08 internals.
- **May be called by:** 09 Signal; 12 Dashboard (read-only).
- **May never depend upon:** 02, 03, 04, 07, 08, 09–14.

## 07 — Order Blocks
- **Responsibilities:** institutional order block detection + mitigation lifecycle.
- **Data owned:** `OrderBlock` set (drawing objects within the shared pool).
- **Data consumed:** Core; **05 Market Structure**; market data.
- **Data prohibited:** order/position state; 02/03/04/06/08 internals.
- **May be called by:** 09 Signal; 12 Dashboard (read-only).
- **May never depend upon:** 02, 03, 04, 06, 08, 09–14.

## 08 — Fair Value Gap
- **Responsibilities:** imbalance / FVG detection + fill tracking.
- **Data owned:** `FairValueGap` set (drawing objects within the shared pool).
- **Data consumed:** Core; **05 Market Structure**; market data.
- **Data prohibited:** order/position state; 02/03/04/06/07 internals.
- **May be called by:** 09 Signal; 12 Dashboard (read-only).
- **May never depend upon:** 02, 03, 04, 06, 07, 09–14.

## 09 — Signal Engine
- **Responsibilities:** deterministic, explainable fusion of analysis into a score + signal.
- **Data owned:** `SignalScore` (score + per-factor attribution), derived `Signal`.
- **Data consumed:** observations from 02–08; Core.
- **Data prohibited:** order/position state; `strategy.*` order calls.
- **May be called by:** 11 Risk; 10 Trade; 12 Dashboard; 13 Alerts (read-only).
- **May never depend upon:** 10, 11, 12, 13, 14.

## 10 — Trade Engine
- **Responsibilities:** entry/exit orchestration; the **only** issuer of `strategy.*` orders.
- **Data owned:** order lifecycle / trade events.
- **Data consumed:** `Signal` (09); sizing & stops (11); Core.
- **Data prohibited:** re-deriving analysis or scores; risk sizing (owned by 11).
- **May be called by:** 12 Dashboard; 13 Alerts (read-only).
- **May never depend upon:** 02–08 (directly), 12, 13, 14.

## 11 — Risk Manager
- **Responsibilities:** position sizing, stops/targets, exposure control; gates Trade.
- **Data owned:** `RiskDecision` (size, stop, target, approval flag).
- **Data consumed:** `Signal` (09); Core; account/position metrics.
- **Data prohibited:** placing orders (owned by 10); analysis internals.
- **May be called by:** 10 Trade; 12 Dashboard; 14 Optimization (read-only).
- **May never depend upon:** 02–08, 10, 12, 13, 14.

## 12 — Dashboard
- **Responsibilities:** on-chart panel; render score breakdown, state, trade status.
- **Data owned:** dashboard `table` (within the table budget).
- **Data consumed (read-only):** 09 score/attribution, 10 trade status, 02–08/11 summaries, Core.
- **Data prohibited:** mutating any state; influencing decisions or orders.
- **May be called by:** none (presentation leaf).
- **May never depend upon:** being depended upon by any decision/execution module.

## 13 — Alerts
- **Responsibilities:** route alert conditions; compose messages for signals/trades.
- **Data owned:** alert conditions/messages.
- **Data consumed (read-only):** 09 signals, 10 trade events, Core.
- **Data prohibited:** mutating state; influencing decisions or orders.
- **May be called by:** none (presentation leaf).
- **May never depend upon:** analysis internals; being depended upon by decision/execution.

## 14 — Optimization
- **Responsibilities:** diagnostics/instrumentation; parameter-exploration support.
- **Data owned:** diagnostic aggregates / instrumentation.
- **Data consumed (read-only):** 09 and 11 diagnostics; Core diagnostics buffer.
- **Data prohibited:** any write into the trade path; changing signals or orders.
- **May be called by:** none (diagnostic leaf).
- **May never depend upon:** the trade path (must never affect 10's behavior).

---

## Dependency summary

```
01 Core        -> (none)
02 Trend       -> 01
03 Momentum    -> 01
04 S/R         -> 01
05 Structure   -> 01
06 Liquidity   -> 01, 05
07 Order Blocks-> 01, 05
08 Fair Value  -> 01, 05
09 Signal      -> 01, 02, 03, 04, 05, 06, 07, 08
11 Risk        -> 01, 09
10 Trade       -> 01, 09, 11
12 Dashboard   -> 01, 09, 10 (read-only)
13 Alerts      -> 01, 09, 10 (read-only)
14 Optimization-> 01, 09, 11 (read-only diagnostics)
```

See [ARCHITECTURE.md](ARCHITECTURE.md) for the same relationships as a diagram.
