# API Reference

Catalog of **public functions** exposed by `../src/MASTER_STRATEGY.pine`. This
document grows alongside the code: every public function must be listed here with
its responsibility, parameters, return value, and notes (see
[STYLE_GUIDE.md](STYLE_GUIDE.md) for documentation rules).

- **Status:** Framework phase (`v0.0.1`).
- **Scope:** Only functions intended for reuse across modules are "public" and
  documented here. Private, single-use helpers are documented inline.

---

## Documentation format

Each entry uses this shape:

```
### functionName(param1, param2)
- Responsibility : one sentence, single responsibility.
- Parameters     : name — type — meaning.
- Returns        : type — meaning.
- Determinism    : deterministic? any state dependence?
- Notes          : repainting considerations, limits, caveats.
```

## Placeholder utilities (v0.0.1)

These are framework stubs with no trading logic. Signatures are provisional.

### isFrameworkOnly()
- Responsibility : Report whether this build is framework-only (no logic).
- Parameters     : none.
- Returns        : `bool` — always `true` in `v0.0.1`.
- Determinism    : deterministic; no state.
- Notes          : Self-description helper; removed or repurposed once modules land.

### projectVersion()
- Responsibility : Expose the compiled project version string.
- Parameters     : none.
- Returns        : `string` — value of `PROJECT_VERSION`.
- Determinism    : deterministic; no state.
- Notes          : Accessor only.

### noop()
- Responsibility : Explicit no-operation seam.
- Parameters     : none.
- Returns        : `na` (void by design).
- Determinism    : deterministic; no state.
- Notes          : Placeholder pattern for future void utilities.

## Module public functions

Populated as each module is implemented, grouped by module. None exist yet.

| Module | Function | Since | Summary |
|--------|----------|-------|---------|
| — | — | — | No public module functions yet. |
