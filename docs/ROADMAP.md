# Roadmap

Milestone plan and version timeline for XAU AI Institutional Scalper Pro.
Milestones are executed **in order** and are **never skipped**. Each milestone
must compile, pass review, and be committed before the next begins
(see [AGENTS.md](../AGENTS.md), [CONTRIBUTING.md](../CONTRIBUTING.md)).

- **Current version:** `0.0.1` — framework only.
- **Versioning:** [Semantic Versioning](https://semver.org/). See the
  [Semantic Versioning Policy](#semantic-versioning-policy) for bump rules.

---

## Semantic Versioning Policy

The project follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`).
This section is the **authoritative** definition of what triggers each bump; it is
mirrored for convenience in [CONTRIBUTING.md](../CONTRIBUTING.md).

| Bump | Increment | Triggered by |
|------|-----------|--------------|
| **PATCH** | `x.y.Z` | Documentation, bug fixes, refactoring — **no public API changes**. |
| **MINOR** | `x.Y.0` | A new module, a new feature, or a new **stable** public API. |
| **MAJOR** | `X.0.0` | A breaking API change, an architectural redesign, or a repository restructuring. |

Rules:
- Only changes to a **stable** (`@stable`) public API affect the version — adding or
  changing `@experimental`/`@internal` functions does not force a MINOR/MAJOR bump
  (see the [API Stability Policy](API.md) and [ADR-0012](DECISIONS.md)).
- **Pre-`1.0.0` (current phase):** the public API is not yet stable. Breaking/redesign
  changes are absorbed as **MINOR** bumps until `1.0.0`; PATCH still means docs/fixes/
  refactors only. From `1.0.0` onward the full policy above applies.
- Each `vX.Y.Z` in the timeline below is cut on `main` and tagged (see the branch
  strategy and Release Checklist in [CONTRIBUTING.md](../CONTRIBUTING.md)).
- **Incremental module build-out (pre-`1.0.0`):** a module implemented across several
  sub-milestones takes **PATCH** increments for the intermediate steps (framework code
  whose public API is still `@experimental` / not yet stabilized), and a **MINOR** bump
  when the module is completed and its public API is stabilized. Example — Module 01:
  `0.0.2 → 0.0.3 → 0.0.4` for M1.1–M1.3, then `0.1.0` at M1.4 (module complete).

## Milestone principle

> Compile → Pass review → Commit → Next milestone.

No milestone may begin until the previous one is committed and its documentation
is consistent.

## Version timeline

| Version | Milestone | Deliverable |
|---------|-----------|-------------|
| `0.0.1` | M0 — Project Architecture | Repository, documentation set, framework-only `MASTER_STRATEGY.pine`. **(current)** |
| `0.0.2` | M1.1 — Core: types & scaffolding | Enums, UDTs, constants, empty subsystem scaffolding. |
| `0.0.3` | M1.2 — Core: utilities, logging, validation | `util`, `log` (bounded ring buffer), `err`/validation subsystems. |
| `0.0.4` | M1.3 — Core: configuration & context | `cfg` (immutable `Config`) and `ctx` (confirmed-bar/session/timeframe). |
| `0.1.0` | M1.4 — Module 01 Core Framework complete | `KernelState`, lifecycle (`coreInit`/`coreOnBar`), diagnostics, integration. |
| `0.2.0` | M2 — Module 02 Trend Engine | Deterministic trend regime classification. |
| `0.3.0` | M3 — Module 03 Momentum Engine | Momentum / strength assessment. |
| `0.4.0` | M4 — Module 04 Support/Resistance | Structural level detection. |
| `0.5.0` | M5 — Module 05 Market Structure | Swing points, BOS / CHoCH. |
| `0.6.0` | M6 — Module 06 Liquidity | Liquidity pools / sweeps. |
| `0.7.0` | M7 — Module 07 Order Blocks | Order block detection. |
| `0.8.0` | M8 — Module 08 Fair Value Gap | Imbalance / FVG detection. |
| `0.9.0` | M9 — Module 09 Signal Engine | Deterministic, explainable scoring. |
| `0.10.0`| M10 — Module 10 Trade Engine | Entry / exit orchestration. |
| `0.11.0`| M11 — Module 11 Risk Manager | Sizing, stops, exposure control. |
| `0.12.0`| M12 — Module 12 Dashboard | On-chart panel with score breakdown. |
| `0.13.0`| M13 — Module 13 Alerts | Alert routing / messaging. |
| `0.14.0`| M14 — Module 14 Optimization | Optimization helpers / diagnostics. |
| `1.0.0` | Release candidate hardening | Full integration, backtest protocol pass, documentation freeze. |

## Release plan

- **Alpha (`0.x`):** modules land incrementally; behavior may change.
- **Beta:** all fourteen modules integrated; stabilization and
  [backtest protocol](BACKTEST_PROTOCOL.md) validation.
- **`1.0.0`:** first stable, fully documented release.

## Definition of milestone done

A milestone is complete when its module compiles clean, satisfies its
specification (`../src/modules/<NN-name>/README.md`), updates
[CHANGELOG.md](CHANGELOG.md) and [API.md](API.md), and is committed with a
Conventional Commit.
