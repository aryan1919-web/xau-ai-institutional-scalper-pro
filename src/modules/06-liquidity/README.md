# Module 06 — Liquidity

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** Planned · **Target version:** `0.6.0` · **Depends on:** Modules 01, 05

## Overview

Maps **liquidity pools** (equal highs/lows, prior session extremes) and detects
**liquidity sweeps**, giving the Signal Engine order-flow context.

## Specification notes

- Deterministic pool identification and sweep detection with documented rules.
- Built on market-structure swings (Module 05); confirmed-bar evaluation.
- Bounded storage; reuse drawing objects.
- Authoritative behavior tracked here and in
  [../../../docs/SPECIFICATION.md](../../../docs/SPECIFICATION.md).

## Design decisions

- _None yet._

## Research notes

- _None yet._

## Testing notes

- Validated per [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md).

## Performance observations

- _None yet._

## Future implementation checklist

- [ ] Liquidity pool identification.
- [ ] Sweep detection.
- [ ] Bounded storage / object reuse.
- [ ] Expose observations to Signal Engine.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
