# Module 11 — Risk Manager

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** Planned · **Target version:** `0.11.0` · **Depends on:** Modules 09, 01

## Overview

Owns **risk**: position sizing, stop-loss / take-profit definition, and exposure
control. Gates the Trade Engine — no order is placed without approved sizing.

## Specification notes

- Deterministic sizing from configured risk parameters (no magic numbers).
- Defines stops/targets and maximum exposure/pyramiding rules.
- Confirmed-bar evaluation; consistent with emulator assumptions.
- Authoritative behavior tracked here and in
  [../../../docs/SPECIFICATION.md](../../../docs/SPECIFICATION.md).

## Design decisions

- _None yet._

## Research notes

- _None yet._

## Testing notes

- Validated per [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md);
  verify drawdown and exposure behavior.

## Performance observations

- _None yet._

## Future implementation checklist

- [ ] Position sizing model.
- [ ] Stop / target definition.
- [ ] Exposure and pyramiding controls.
- [ ] Provide sizing/stops to Trade Engine.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
