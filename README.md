# ResilienceKit

Circuit breakers, retry primitives, and CI/CD + deploy helpers for the Phenotype ecosystem. Polyglot — shipped as Go, Rust, and Python bindings from a single source of truth.

**Part of the [Phenotype org](https://github.com/KooshaPari) ecosystem.** Shares CI reusables and conventions with [phenoShared](https://github.com/KooshaPari/phenoShared). Follows org conventions: conventional commits, `<type>/<topic>` branching, Apache-2.0 + MIT dual license.

## What it does

ResilienceKit gives Phenotype services a uniform way to fail well: circuit breakers, bounded retries with jitter, timeout policy, and the CI/CD plus deploy helpers that wrap those policies into pipelines. Instead of every service hand-rolling `tenacity` or a bespoke breaker, they pull from here.

## Status

**Active.** Python packages (`pheno-resilience`, `ci-cd-kit`, `deploy-kit`) are the primary surface today; Go and Rust bindings mirror the same contracts. See [CHANGELOG.md](./CHANGELOG.md).

## Requirements

- **Python** (primary): 3.11+ with `uv` or `pip`
- **Go**: 1.22+
- **Rust**: stable, edition 2021
- For `deploy-kit` / `ci-cd-kit`: a reachable CI runner and deploy target (Kubernetes, Nomad, etc.) at runtime

## Quick start

### Python

```bash
cd python
uv sync                    # or: pip install -e '.[dev]'
uv run pytest              # or: pytest
uv run ruff check .
uv run mypy .
```

### Go

```bash
cd go
go build ./...
go test ./...
```

### Rust

```bash
cd rust
cargo build --workspace
cargo test --workspace
cargo clippy --workspace --all-targets -- -D warnings
```

## Structure

```
python/
  pheno-resilience/   # Circuit breakers, retry/backoff, timeout, bulkhead primitives
  ci-cd-kit/          # CI pipeline helpers — build, test, quality-gate orchestration
  deploy-kit/         # Deploy helpers — rollout, health-gate, rollback
go/                   # Go bindings mirroring the Python contracts
rust/                 # Rust bindings mirroring the Python contracts
```

## Design principles

- **Fail loudly, recover deliberately.** Breakers open visibly; retries surface attempt counts; timeouts are explicit, not implicit.
- **Wrap, do not hand-roll.** Builds on proven upstream libraries (tenacity, pybreaker, gobreaker, tower) and adds Phenotype policy.
- **Policy as config.** Breaker thresholds, retry budgets, and timeout policy live in config, not scattered through call sites.
- **Observability baked in.** State transitions (closed → open → half-open) emit structured events for the Phenotype observability stack.

## Contributing

See [CONTRIBUTING.md](./CONTRIBUTING.md). Ownership lives in [CODEOWNERS](./CODEOWNERS). Report security issues per [SECURITY.md](./SECURITY.md).

## License

Dual-licensed under Apache-2.0 OR MIT. See [LICENSE-APACHE](./LICENSE-APACHE) and [LICENSE-MIT](./LICENSE-MIT).
