# Architectural Decision Records (ADRs)

This log records significant decisions. Each ADR has an **ID**, **Context**,
**Decision**, **Consequences**, and **Status**. ADRs are immutable once
`Accepted`; to change a decision, add a new ADR that supersedes the old one.

Statuses: `Proposed` · `Accepted` · `Superseded` · `Deprecated`.

---

## ADR-0001 — Pine Script v6 only

- **Status:** Accepted
- **Context:** TradingView offers multiple Pine Script versions. Mixing versions
  fragments syntax and behavior and harms long-term maintainability.
- **Decision:** The project uses **Pine Script v6 exclusively**. v5 and earlier
  are prohibited.
- **Consequences:** Access to v6 language features and semantics; contributors
  must target v6; migration cost is avoided by never adopting older versions.

## ADR-0002 — Strategy, not Indicator

- **Status:** Accepted
- **Context:** The product must place and manage simulated trades and be
  backtestable, not merely plot signals.
- **Decision:** Implement as a TradingView **strategy** (`strategy(...)`), not an
  indicator.
- **Consequences:** Full backtesting and order simulation via the broker
  emulator; subject to broker-emulator limitations
  ([KNOWN_LIMITATIONS.md](KNOWN_LIMITATIONS.md)).

## ADR-0003 — Deterministic and explainable AI scoring (no black box)

- **Status:** Accepted
- **Context:** "AI scoring" must remain trustworthy, auditable, and reproducible
  for a multi-year trading system.
- **Decision:** The signal/scoring engine (Module 09) MUST be **deterministic
  and explainable**. Every score is a documented function of named factor
  contributions; no opaque or nondeterministic logic.
- **Consequences:** Scores are reproducible and inspectable (dashboard/logs);
  rules out black-box on-chart inference; enables meaningful review and testing.

## ADR-0004 — Incremental milestone workflow

- **Status:** Accepted
- **Context:** Building everything at once risks an unmaintainable, unreviewable
  codebase.
- **Decision:** Build in strict, ordered milestones. Each must **compile → pass
  review → be committed** before the next begins. No milestone is skipped.
- **Consequences:** Continuous compilability and reviewability; clear version
  timeline ([ROADMAP.md](ROADMAP.md)); slower but safer delivery.

## ADR-0005 — Single-file compilation; module docs as markdown

- **Status:** Accepted
- **Context:** TradingView compiles a single Pine file and does not support
  local includes. We still want modular, well-documented development.
- **Decision:** All Pine logic lives in `src/MASTER_STRATEGY.pine`. The
  `src/modules/` tree contains **documentation only** — one folder per module —
  and **never** contains Pine Script files.
- **Consequences:** One compilation unit for TradingView; per-module design and
  research history is preserved as markdown; contributors must not add `.pine`
  files under `src/modules/`.
