# Module 02 — Trend Engine

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** Planned · **Target version:** `0.2.0` · **Depends on:** Module 01

## Overview

Classifies the prevailing **trend regime** (directional bias and strength) for
XAUUSD on M1/M5, producing a structured, explainable observation consumed by the
Signal Engine (Module 09).

## Specification notes

- Emits a deterministic trend state (e.g. bias direction + strength) with named
  contributing factors — no opaque logic ([ADR-0003](../../../docs/DECISIONS.md)).
- Higher-timeframe context (if used) via `request.security(..., lookahead_off)`.
- Authoritative behavior tracked here and in
  [../../../docs/SPECIFICATION.md](../../../docs/SPECIFICATION.md).

## Design decisions

- _None yet._

## Research notes

- _None yet._ Candidate methods and references to be recorded here before coding.

## Testing notes

- Validated per [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md);
  verify non-repainting HTF usage.

## Performance observations

- _None yet._

## Future implementation checklist

- [ ] Define trend state representation.
- [ ] Deterministic classification with named factors.
- [ ] Non-repainting HTF handling.
- [ ] Expose observation to Signal Engine.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
