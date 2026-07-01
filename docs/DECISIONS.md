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

## ADR-0006 — Subsystem-prefixed public function naming

- **Status:** Accepted
- **Context:** All 14 modules share one Pine file. Flat names risk collisions and
  obscure ownership as the file grows.
- **Decision:** Public functions use subsystem prefixes. Core subsystems are `cfg`,
  `ctx`, `util`, `state`, `err`, `log` (e.g. `cfgGet`, `ctxIsNewBar`, `utilClamp`).
  Modules 02–14 use their own registered prefixes ([STYLE_GUIDE.md](STYLE_GUIDE.md)).
- **Consequences:** Collision-resistant, self-describing call sites; a prefix registry
  must be maintained; slightly longer names.

## ADR-0007 — Native Pine v6 `enum` + UDT for shared types

- **Status:** Accepted
- **Context:** Shared vocabulary (direction, session, log level) needs a type-safe,
  readable representation. Pine v6 provides `enum` and user-defined `type`.
- **Decision:** Use native v6 `enum` and UDTs for shared types rather than encoding
  them as integer constants.
- **Consequences:** Type safety, readability, `switch` compatibility; relies on v6
  features (consistent with [ADR-0001](#adr-0001--pine-script-v6-only)).

## ADR-0008 — Immutable `Config` snapshot; modules never read `input.*` directly

- **Status:** Accepted
- **Context:** Scattered `input.*` reads couple configuration to logic and make
  validation and testing hard.
- **Decision:** Inputs are read once by `cfgBuild`, validated, and exposed as an
  immutable `Config` snapshot. Modules consume `Config`; they never call `input.*`.
- **Consequences:** Configuration is centralized, validated once, and testable;
  adding a module input means extending `Config`.

## ADR-0009 — Single `KernelState` persistent object

- **Status:** Accepted
- **Context:** Many independent `var` scalars are hard to reason about and count
  against persistence budgets.
- **Decision:** A single persistent `KernelState` UDT (`var`) is the sole mutable
  global, mutated only through `state*` functions.
- **Consequences:** Centralized, auditable state; fewer `var` slots; all persistent
  fields live in one documented type.

## ADR-0010 — Hybrid configuration validation

- **Status:** Accepted
- **Context:** Invalid configuration must neither silently corrupt behavior nor make
  the strategy needlessly brittle.
- **Decision:** Hybrid policy: **fail-fast** (`runtime.error`) for structurally invalid
  or contradictory settings; **safe-degrade** (clamp to valid + `errWarn`) for
  out-of-range scalar values. Enforced by `errApplyValidation`.
- **Consequences:** Predictable, explainable failure for real misconfiguration;
  resilience to minor scalar mistakes; validation logic must classify severities.

## ADR-0011 — Mandatory Core non-repainting primitives

- **Status:** Accepted
- **Context:** Repainting is prohibited; ad-hoc `request.security` usage across modules
  is the most common source of look-ahead bias.
- **Decision:** All HTF access goes through `ctxHtfValue` (fixing
  `barmerge.lookahead_off`, confirmed values); decisions evaluate on
  `ctxIsConfirmedBar()`. Raw `request.security` in modules is prohibited.
- **Consequences:** Centralized, auditable non-repainting guarantee; a single place to
  enforce and review look-ahead safety; modules depend on Core for HTF data.

## ADR-0012 — API stability levels (Stable / Experimental / Internal)

- **Status:** Accepted
- **Context:** As modules build on one another, contributors need to know which
  functions are safe to rely on long-term.
- **Decision:** Every public function declares `@stable`, `@experimental`, or
  `@internal`. Only stable/experimental are cross-module callable; `@internal` is local.
  Promotion `experimental → stable` and breaking a stable contract require an ADR.
- **Consequences:** Clear, enforceable API contract cataloged in
  [API.md](API.md); disciplined evolution of the public surface.

## ADR-0013 — Permanent performance budgets

- **Status:** Accepted
- **Context:** TradingView imposes hard resource limits; unmanaged growth risks hitting
  them late and unpredictably.
- **Decision:** Adopt permanent, project-wide budgets (`request.security`, arrays,
  labels, boxes, lines, tables, diagnostics entries, persistent objects, per-bar
  complexity) defined in [SPECIFICATION.md](SPECIFICATION.md#performance-budget). Every
  module must stay within them; exceeding a budget requires an ADR.
- **Consequences:** Predictable resource headroom; drawing modules share pooled budgets
  and reuse objects; performance is a first-class design constraint.

## ADR-0014 — Module Ownership Matrix as the dependency contract

- **Status:** Accepted
- **Context:** A 14-module single-file system needs an explicit, enforceable contract to
  prevent tangled dependencies and hidden coupling.
- **Decision:** [MODULE_OWNERSHIP.md](MODULE_OWNERSHIP.md) is the authoritative contract
  defining, per module, responsibilities, data owned/consumed/prohibited, permitted
  callers, and forbidden dependencies. It must agree with the
  [ARCHITECTURE.md](ARCHITECTURE.md) dependency graph.
- **Consequences:** Upward-only, acyclic dependencies are enforceable in review; changes
  to a module's contract require an ADR.
