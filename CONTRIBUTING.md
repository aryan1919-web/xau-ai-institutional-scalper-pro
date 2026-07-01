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
2. **Branch** from the default branch (see naming below).
3. **Implement** the change following the style guide and architecture rules.
4. **Verify** it compiles clean in the TradingView Pine Editor.
5. **Update documentation** (SPECIFICATION, ARCHITECTURE, API, CHANGELOG, module
   README) as relevant.
6. **Commit** using Conventional Commits.
7. **Open a pull request** using the template and request review.

## Branch naming

Use short, descriptive, hyphenated branch names prefixed by type:

```
feat/02-trend-engine
fix/risk-position-sizing
docs/architecture-dataflow
refactor/signal-scoring
chore/ci-setup
```

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

## Review process

A change is mergeable only when it:

1. **Compiles** clean in the TradingView Pine Editor (no errors).
2. **Passes review** against [AGENTS.md](AGENTS.md) and the
   [style guide](docs/STYLE_GUIDE.md):
   - single responsibility per function,
   - no magic numbers, no duplication, no repainting, no hidden calculations,
   - documentation present and accurate.
3. **Keeps documentation consistent** (cross-links valid, CHANGELOG updated).

## Code of conduct

Be professional, precise, and constructive. Prefer clarity over cleverness.
