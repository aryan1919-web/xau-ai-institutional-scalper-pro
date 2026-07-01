# Module 10 — Trade Engine

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** Planned · **Target version:** `0.10.0` · **Depends on:** Modules 09, 11, 01

## Overview

Translates approved signals into **orders** — entries, exits, and modifications —
using TradingView `strategy.*` calls. Executes only what the Signal Engine
approves and the Risk Manager sizes.

## Specification notes

- Acts on **confirmed bars / bar close** to avoid repainting.
- Consumes sizing and stop levels from the Risk Manager (Module 11).
- Order behavior aligns with the broker-emulator assumptions in
  [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md) and
  [../../../docs/KNOWN_LIMITATIONS.md](../../../docs/KNOWN_LIMITATIONS.md).
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

- [ ] Entry orchestration.
- [ ] Exit orchestration (stops/targets from Risk Manager).
- [ ] Order modification / lifecycle.
- [ ] Emit trade events to Dashboard / Alerts.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
