# Module 07 — Order Blocks

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** Planned · **Target version:** `0.7.0` · **Depends on:** Modules 01, 05

## Overview

Detects institutional **order blocks** — origin zones of significant moves — and
tracks their validity/mitigation for use by the Signal Engine.

## Specification notes

- Deterministic detection and lifecycle (formed → tested → mitigated) with
  documented rules.
- Confirmed-bar evaluation; bounded storage; reuse drawing objects.
- Authoritative behavior tracked here and in
  [../../../docs/SPECIFICATION.md](../../../docs/SPECIFICATION.md).

## Design decisions

- _None yet._

## Research notes

- _None yet._

## Testing notes

- Validated per [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md).

## Performance observations

- _None yet._ Watch object counts for zone rendering.

## Future implementation checklist

- [ ] Order block detection rules.
- [ ] Validity / mitigation lifecycle.
- [ ] Bounded storage / object reuse.
- [ ] Expose zones to Signal Engine.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
