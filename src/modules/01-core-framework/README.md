# Module 01 — Core Framework

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** Planned · **Target version:** `0.1.0` · **Depends on:** none (foundation)

## Overview

Foundational services shared by every other module: configuration plumbing,
shared state and types, session/timeframe context, and common utility seams. It
owns no trading decisions — it provides the substrate the analysis, decision, and
execution layers build on.

## Specification notes

- Centralizes configuration access so inputs are declared once and consumed
  cleanly (config separated from logic).
- Provides session/context helpers for XAUUSD M1/M5 scalping.
- Establishes non-repainting primitives (confirmed-bar helpers) used everywhere.
- Authoritative behavior is tracked here and in
  [../../../docs/SPECIFICATION.md](../../../docs/SPECIFICATION.md).

## Design decisions

- Single-file compilation; see [ADR-0005](../../../docs/DECISIONS.md).
- Local design choices recorded here; cross-cutting ones become ADRs.

## Research notes

- _None yet._

## Testing notes

- Validated per [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md);
  emphasis on confirmed-bar / non-repainting primitives.

## Performance observations

- _None yet._ Watch per-bar cost of shared helpers (runtime limits).

## Future implementation checklist

- [ ] Configuration access layer.
- [ ] Shared state/types and constants ownership.
- [ ] Session/timeframe context helpers.
- [ ] Confirmed-bar / non-repainting primitives.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
