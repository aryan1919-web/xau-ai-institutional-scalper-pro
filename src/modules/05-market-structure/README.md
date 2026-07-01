# Module 05 — Market Structure

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** Planned · **Target version:** `0.5.0` · **Depends on:** Module 01

## Overview

Identifies **swing points** and classifies structural events — Break of Structure
(BOS) and Change of Character (CHoCH) — providing the structural backbone for
liquidity, order-block, and FVG analysis.

## Specification notes

- Deterministic swing detection and event classification with documented rules.
- Confirmed-bar evaluation; no look-ahead.
- Bounded storage of swing history (respect array/object limits).
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

- [ ] Swing point detection.
- [ ] BOS / CHoCH classification.
- [ ] Bounded swing history store.
- [ ] Expose structure to downstream modules.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
