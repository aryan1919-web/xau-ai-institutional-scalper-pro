# Module 04 — Support / Resistance Engine

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** Planned · **Target version:** `0.4.0` · **Depends on:** Module 01

## Overview

Detects structural **support and resistance** levels used as context for signal
scoring and trade management.

## Specification notes

- Deterministic level detection with explicit, documented rules.
- Bounded, reused drawing objects (respect object limits — see
  [../../../docs/KNOWN_LIMITATIONS.md](../../../docs/KNOWN_LIMITATIONS.md)).
- Authoritative behavior tracked here and in
  [../../../docs/SPECIFICATION.md](../../../docs/SPECIFICATION.md).

## Design decisions

- _None yet._

## Research notes

- _None yet._

## Testing notes

- Validated per [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md).

## Performance observations

- _None yet._ Watch drawing-object counts.

## Future implementation checklist

- [ ] Define level representation and lifecycle.
- [ ] Deterministic detection rules.
- [ ] Object reuse/caps for any drawings.
- [ ] Expose levels to Signal Engine / Trade Engine.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
