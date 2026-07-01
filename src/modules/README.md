# Module Documentation

This directory holds **documentation only**. TradingView compiles **only**
[`../MASTER_STRATEGY.pine`](../MASTER_STRATEGY.pine); all Pine logic lives there.

> **No Pine Script (`.pine`) files are ever placed in this tree.** See
> [ADR-0005](../../docs/DECISIONS.md).

Each module has its own folder containing a `README.md` that grows over the
module's life with:

- **Overview** — what the module does and why.
- **Specification notes** — authoritative behavior for the module.
- **Design decisions** — local choices (link to ADRs for cross-cutting ones).
- **Research notes** — investigations, references, experiments.
- **Testing notes** — how the module is validated (per [BACKTEST_PROTOCOL.md](../../docs/BACKTEST_PROTOCOL.md)).
- **Performance observations** — measured behavior, limits encountered.
- **Future implementation checklist** — remaining work.

## Module index

| # | Module | Folder |
|---|--------|--------|
| 01 | Core Framework | [01-core-framework/](01-core-framework/) |
| 02 | Trend Engine | [02-trend-engine/](02-trend-engine/) |
| 03 | Momentum Engine | [03-momentum-engine/](03-momentum-engine/) |
| 04 | Support / Resistance Engine | [04-support-resistance/](04-support-resistance/) |
| 05 | Market Structure | [05-market-structure/](05-market-structure/) |
| 06 | Liquidity | [06-liquidity/](06-liquidity/) |
| 07 | Order Blocks | [07-order-blocks/](07-order-blocks/) |
| 08 | Fair Value Gap | [08-fair-value-gap/](08-fair-value-gap/) |
| 09 | Signal Engine | [09-signal-engine/](09-signal-engine/) |
| 10 | Trade Engine | [10-trade-engine/](10-trade-engine/) |
| 11 | Risk Manager | [11-risk-manager/](11-risk-manager/) |
| 12 | Dashboard | [12-dashboard/](12-dashboard/) |
| 13 | Alerts | [13-alerts/](13-alerts/) |
| 14 | Optimization | [14-optimization/](14-optimization/) |

See [../../docs/ROADMAP.md](../../docs/ROADMAP.md) for implementation order and
[../../docs/ARCHITECTURE.md](../../docs/ARCHITECTURE.md) for how modules relate.
