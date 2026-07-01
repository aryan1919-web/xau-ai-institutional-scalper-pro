# Roadmap

Milestone plan and version timeline for XAU AI Institutional Scalper Pro.
Milestones are executed **in order** and are **never skipped**. Each milestone
must compile, pass review, and be committed before the next begins
(see [AGENTS.md](../AGENTS.md), [CONTRIBUTING.md](../CONTRIBUTING.md)).

- **Current version:** `0.0.1` — framework only.
- **Versioning:** [Semantic Versioning](https://semver.org/). Pre-`1.0.0` the
  API and behavior may change between minor versions.

---

## Milestone principle

> Compile → Pass review → Commit → Next milestone.

No milestone may begin until the previous one is committed and its documentation
is consistent.

## Version timeline

| Version | Milestone | Deliverable |
|---------|-----------|-------------|
| `0.0.1` | M0 — Project Architecture | Repository, documentation set, framework-only `MASTER_STRATEGY.pine`. **(current)** |
| `0.1.0` | M1 — Module 01 Core Framework | Config plumbing, shared state/types, session context. |
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
