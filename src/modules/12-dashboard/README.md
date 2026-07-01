# Module 12 — Dashboard

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** Planned · **Target version:** `0.12.0` · **Depends on:** Modules 09, 10, 01

## Overview

Renders an **on-chart informational panel**: current score with its factor
breakdown (supporting explainability), trend/momentum/structure state, and trade
status.

## Specification notes

- Read-only presentation; must not influence decisions.
- Surfaces the Signal Engine's rationale to satisfy explainability
  ([../../../docs/SPECIFICATION.md](../../../docs/SPECIFICATION.md) §7).
- Uses a bounded `table`/objects; respects object limits
  ([../../../docs/KNOWN_LIMITATIONS.md](../../../docs/KNOWN_LIMITATIONS.md)).

## Design decisions

- _None yet._

## Research notes

- _None yet._

## Testing notes

- Visual/manual verification plus [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md)
  parity checks (rendering must not affect signals).

## Performance observations

- _None yet._ Watch per-bar rendering cost and object counts.

## Future implementation checklist

- [ ] Panel layout (bounded table).
- [ ] Score + factor-breakdown display.
- [ ] State/trade-status display.
- [ ] Toggle via input.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
