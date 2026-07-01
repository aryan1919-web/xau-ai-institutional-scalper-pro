# Contributing

Thank you for contributing to **XAU AI Institutional Scalper Pro**. This project
is engineered for long-term maintainability. Please read
[AGENTS.md](AGENTS.md) and [docs/STYLE_GUIDE.md](docs/STYLE_GUIDE.md) before you
start — they are binding.

---

## Contribution workflow

1. **Confirm scope.** Work must map to an approved milestone in
   [docs/ROADMAP.md](docs/ROADMAP.md). No implementation without a specification
   in [docs/SPECIFICATION.md](docs/SPECIFICATION.md).
2. **Branch** from `develop` (see the [branching model](#official-branching-model) below).
3. **Implement** the change following the style guide and architecture rules.
4. **Verify** it compiles clean in the TradingView Pine Editor.
5. **Update documentation** (SPECIFICATION, ARCHITECTURE, API, CHANGELOG, module
   README) as relevant.
6. **Commit** using Conventional Commits.
7. **Open a pull request** using the template and request review.

## Official branching model

The project uses a `main` / `develop` model with short-lived topic branches.

| Branch | Purpose | Branched from | Merges into |
|--------|---------|---------------|-------------|
| `main` | Production-ready, released code. Every merge is a tagged `vX.Y.Z` release. | — | — (receives from `develop`, `hotfix/*`) |
| `develop` | Integration branch for delivered work; base for all new branches. | `main` | `main` (at release) |
| `feature/module-xx-name` | New module / feature (e.g. `feature/module-02-trend-engine`). | `develop` | `develop` |
| `bugfix/*` | Non-urgent fixes (e.g. `bugfix/risk-position-sizing`). | `develop` | `develop` |
| `docs/*` | Documentation-only changes (e.g. `docs/architecture-dataflow`). | `develop` | `develop` |
| `hotfix/*` | Urgent production fix; triggers a PATCH release. | `main` | `main` **and** back to `develop` |

**Merge rules**

- No direct commits to `main` or `develop`; all changes land via reviewed pull request.
- Every PR must compile clean and pass review ([AGENTS.md](AGENTS.md) +
  [docs/STYLE_GUIDE.md](docs/STYLE_GUIDE.md)) and use Conventional Commits.
- `feature/*`, `bugfix/*`, `docs/*` → `develop` (squash-merge); delete the branch after merge.
- Release: `develop` → `main` via merge commit, then tag `vX.Y.Z` (matches the
  [ROADMAP timeline](docs/ROADMAP.md#version-timeline)).
- `hotfix/*` → `main` (tag a PATCH) then **back-merge** to `develop` so the fix is not lost.
- **Tooling exception:** automated/agent sessions may use a `claude/*` working branch
  (e.g. `claude/xau-scalper-setup-nicysf`); it follows the same review and merge rules
  when integrated.

### Repository protection rules

| Branch | Protected | Direct commits | Force-push | Required reviews | Preferred merge strategy |
|--------|-----------|----------------|-----------|------------------|--------------------------|
| `main` | Yes | No — PR only | No — blocked | ≥ 1 approval + passing checks | Merge commit (release), then tag `vX.Y.Z` |
| `develop` | Yes | No — PR only | No — blocked | ≥ 1 approval + passing checks | Squash merge (from `feature`/`bugfix`/`docs`) |
| `feature/*` | No | Yes (own branch) | Allowed pre-merge (tidy history) | Review at PR into `develop` | Squash into `develop` |
| `bugfix/*` | No | Yes (own branch) | Allowed pre-merge | Review at PR into `develop` | Squash into `develop` |
| `docs/*` | No | Yes (own branch) | Allowed pre-merge | Review at PR into `develop` | Squash into `develop` |
| `hotfix/*` | No | Yes (own branch) | Allowed pre-merge | Review at PR into `main` | Merge commit into `main` (tag), then back-merge to `develop` |

Protected branches (`main`, `develop`) block direct pushes and force-pushes and require a
passing review before merge. Topic branches may be force-pushed **only before** their PR is
merged; **never** force-push a protected branch.

## Commit rules

This project uses [Conventional Commits](https://www.conventionalcommits.org/).

| Type       | Use for                                             |
|------------|-----------------------------------------------------|
| `feat`     | New user-visible functionality (a module/feature)   |
| `fix`      | Bug fixes                                            |
| `docs`     | Documentation-only changes                          |
| `refactor` | Code change that neither fixes a bug nor adds a feature |
| `test`     | Adding or adjusting tests / test protocol            |
| `chore`    | Tooling, scaffolding, maintenance                    |

Format:

```
<type>(optional-scope): <imperative, concise summary>

<optional body explaining what and why>
```

Commit messages must be specific. Vague messages (e.g. "update", "fixes") are
rejected.

## Semantic Versioning

The project follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`).
The **authoritative** policy lives in
[docs/ROADMAP.md](docs/ROADMAP.md#semantic-versioning-policy); the bump rules are:

| Bump | Triggered by |
|------|--------------|
| **PATCH** | Documentation, bug fixes, refactoring — **no public API changes**. |
| **MINOR** | A new module, a new feature, or a new **stable** public API. |
| **MAJOR** | A breaking API change, an architectural redesign, or a repository restructuring. |

Only changes to a **stable** (`@stable`) public API affect the version. Pre-`1.0.0`,
breaking/redesign changes are absorbed as MINOR bumps (see ROADMAP for the full rule).

## Review process

A change is mergeable only when it:

1. **Compiles** clean in the TradingView Pine Editor (no errors).
2. **Passes review** against [AGENTS.md](AGENTS.md) and the
   [style guide](docs/STYLE_GUIDE.md):
   - single responsibility per function,
   - no magic numbers, no duplication, no repainting, no hidden calculations,
   - documentation present and accurate.
3. **Keeps documentation consistent** (cross-links valid, CHANGELOG updated).

## Release Checklist

Complete every item, in order, before tagging a release on `main`:

- [ ] **Compile passes** — `src/MASTER_STRATEGY.pine` compiles clean in the TradingView Pine Editor.
- [ ] **Documentation updated** — SPECIFICATION / ARCHITECTURE / API / module README as relevant.
- [ ] **CHANGELOG updated** — `[Unreleased]` moved under the new `vX.Y.Z` with its date.
- [ ] **REVIEW REPORT approved** — the milestone's review report is signed off.
- [ ] **Version updated** — `PROJECT_VERSION` in `MASTER_STRATEGY.pine` and the
      [ROADMAP timeline](docs/ROADMAP.md#version-timeline) match `vX.Y.Z`.
- [ ] **Git tag created** — annotated tag `vX.Y.Z` on the `main` merge commit.
- [ ] **Release notes prepared** — summary of changes derived from the CHANGELOG.

## Code of conduct

Be professional, precise, and constructive. Prefer clarity over cleverness.
