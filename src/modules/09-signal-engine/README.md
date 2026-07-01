# Module 09 — Signal Engine

> Documentation only. Implementation lives in
> [`../../MASTER_STRATEGY.pine`](../../MASTER_STRATEGY.pine).
>
> **Status:** Planned · **Target version:** `0.9.0` · **Depends on:** Modules 02–08

## Overview

The **decision core**. Fuses observations from the analysis modules (02–08) into a
**deterministic, explainable score** and derives trade signals from it. This is
the project's determinism boundary.

## Specification notes

- Score is a **documented function of named factor contributions**; identical
  inputs yield identical output ([ADR-0003](../../../docs/DECISIONS.md),
  [../../../docs/SPECIFICATION.md](../../../docs/SPECIFICATION.md) §7).
- Each contribution is inspectable (exposed to Dashboard/Alerts).
- No black-box or nondeterministic logic. Confirmed-bar evaluation.

## Design decisions

- Weighting scheme and thresholds to be defined and recorded here (and as ADRs if
  cross-cutting).

## Research notes

- _None yet._

## Testing notes

- Validated per [../../../docs/BACKTEST_PROTOCOL.md](../../../docs/BACKTEST_PROTOCOL.md);
  determinism and explainability checks are mandatory.

## Performance observations

- _None yet._

## Future implementation checklist

- [ ] Define factor set and weighting.
- [ ] Deterministic scoring function with attribution.
- [ ] Signal derivation from score.
- [ ] Expose rationale to Dashboard / Alerts.
- [ ] Documentation: update SPECIFICATION, ARCHITECTURE, API, CHANGELOG.
