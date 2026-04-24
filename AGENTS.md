# Agent Rules - ResilienceKit

**This project is managed through AgilePlus.**

## Project Overview

### Name
ResilienceKit (Phenotype Resilience Toolkit)

### Description
ResilienceKit is a multi-language resilience toolkit for the Phenotype ecosystem. It provides fault-tolerance patterns including retry logic, circuit breakers, bulkheads, rate limiting, timeouts, fallbacks, error classification, and health checking, implemented consistently across Python, Rust, and Go.

### Location
`/Users/kooshapari/CodeProjects/Phenotype/repos/ResilienceKit`

### Language Stack
- **Python**: Circuit breaker, retry, bulkhead, timeout, fallback, error handling, health checks
- **Rust**: State machine, async traits, port traits, retry, rate limiting
- **Go**: Retry, circuit breaker, bulkhead, timeout (planned)

### Purpose & Goals
- **Mission**: Provide resilient, fault-tolerant patterns for distributed systems
- **Primary Goal**: Prevent cascading failures through isolation and graceful degradation
- **Secondary Goals**:
  - Implement async-first patterns across all languages
  - Provide strong typing and compile-time safety
  - Enable composability for layered defense
  - Deliver built-in observability and metrics

### Key Responsibilities
1. **Circuit Breaker**: Detect and isolate failing services
2. **Retry**: Automatic retries with configurable backoff strategies
3. **Bulkhead**: Resource isolation to prevent cascading failures
4. **Rate Limiting**: Control request rates to prevent overload
5. **Timeout**: Prevent operations from hanging indefinitely
6. **Fallback**: Alternative behavior when primary operations fail
7. **Error Classification**: Categorize errors for intelligent handling
8. **Health Checking**: Monitor system health and component status

---

## Quick Start Commands

### Prerequisites

```bash
# Rust 1.75+
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh

# Go 1.24+
brew install go@1.24

# Python 3.12+
brew install python@3.12
```

### Installation

```bash
# Navigate to ResilienceKit
cd /Users/kooshapari/CodeProjects/Phenotype/repos/ResilienceKit

# Build Rust components
cd rust && cargo build --workspace

# Install Go dependencies
cd go && go mod download

# Install Python components
pip install -e python/pheno-resilience/
```

### Development Environment Setup

```bash
# Copy environment configuration
cp .env.example .env

# Run health checks
python -m pheno_resilience health-check
```

### Running Examples

```bash
# Rust circuit breaker example
cd rust && cargo run --example circuit_breaker

# Go retry example
cd go && go run examples/retry.go

# Python bulkhead example
python python/pheno-resilience/examples/bulkhead_demo.py
```

### Verification

```bash
# Run Rust tests
cd rust && cargo test --workspace

# Run Go tests
cd go && go test ./...

# Run Python tests
pytest python/ -v

# Health check validation
python -m pheno_resilience doctor
```

---

## Architecture

### Resilience Patterns

```
┌─────────────────────────────────────────────────────────────────┐
│                    Resilience Patterns                          │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Circuit Breaker                                          │  │
│  │                                                           │  │
│  │   ┌────────┐     ┌────────┐     ┌────────┐             │  │
│  │   │ CLOSED │────▶│  OPEN  │────▶│HALF-OPEN│             │  │
│  │   │        │fail │        │wait │         │             │  │
│  │   │ normal │────▶│ reject │────▶│  test   │             │  │
│  │   │ ops    │     │ all    │     │  ops    │             │  │
│  │   └────────┘     └────────┘     └────┬───┘             │  │
│  │      ▲                               │                 │  │
│  │      │ success                       │ success         │  │
│  │      └───────────────────────────────┘                 │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Retry with Backoff                                       │  │
│  │                                                           │  │
│  │   Attempt 1 ──▶ Attempt 2 ──▶ Attempt 3 ──▶ Success     │  │
│  │      │            │            │                         │  │
│  │      │            │            │                         │  │
│  │   0ms delay    100ms delay   200ms delay                  │  │
│  │   (exponential backoff)                                   │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Bulkhead (Resource Isolation)                            │  │
│  │                                                           │  │
│  │   Service A          Service B          Service C         │  │
│  │   ┌────────┐         ┌────────┐         ┌────────┐      │  │
│  │   │Thread  │         │Thread  │         │Thread  │      │  │
│  │   │Pool:5  │         │Pool:3  │         │Pool:10 │      │  │
│  │   │        │         │        │         │        │      │  │
│  │   │[░░░░░░]│         │[░░░   ]│         │[░░░░░░░░░░]│      │  │
│  │   │ 5/5   │         │ 2/3   │         │ 8/10  │      │  │
│  │   └────────┘         └────────┘         └────────┘      │  │
│  │                                                           │  │
│  │   Failure in A cannot affect B or C                       │  │
│  │                                                           │  │
│  └───────────────────────────────────────────────────────────┘  │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    Application Layer                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐           │
│  │   Service    │  │   Service    │  │   Service    │           │
│  │      A       │  │      B       │  │      C       │           │
│  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘           │
└─────────┼─────────────────┼─────────────────┼───────────────────┘
          │                 │                 │
          └─────────────────┼─────────────────┘
                            │
                            ▼
┌─────────────────────────────────────────────────────────────────┐
│                    ResilienceKit Layer                            │
├─────────────────────────────────────────────────────────────────┤
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Python Layer                                             │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │ pheno-       │  │ Circuit      │  │ Bulkhead     │   │  │
│  │  │ resilience   │  │ Breaker      │  │              │   │  │
│  │  │              │  │              │  │              │   │  │
│  │  │ • Retry      │  │ • State      │  │ • Isolation  │   │  │
│  │  │ • Timeout    │  │ machine      │  │ • Queuing    │   │  │
│  │  │ • Fallback   │  │ • Metrics    │  │ • Rejection  │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Rust Layer                                               │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │ State        │  │ Async        │  │ Retry        │   │  │
│  │  │ Machine      │  │ Traits       │  │              │   │  │
│  │  │              │  │              │  │              │   │  │
│  │  │ • Hierarchical│ │ • Async fns  │  │ • Exponential│   │  │
│  │  │ • Transitions│  │ • Streams    │  │ • Jitter     │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────────────────┐  │
│  │  Go Layer (Planned)                                       │  │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐   │  │
│  │  │ pheno-retry  │  │ pheno-       │  │ pheno-       │   │  │
│  │  │              │  │ circuit      │  │ bulkhead     │   │  │
│  │  │              │  │ breaker      │  │              │   │  │
│  │  │ • Context    │  │ • State      │  │ • Semaphore  │   │  │
│  │  │   support    │  │   tracking   │  │   based      │   │  │
│  │  └──────────────┘  └──────────────┘  └──────────────┘   │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Circuit Breaker State Machine

```
┌─────────────────────────────────────────────────────────────────┐
│                    Circuit Breaker States                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   ┌─────────┐    failure_threshold    ┌─────────┐               │
│   │ CLOSED  │ ─────────────────────▶ │  OPEN   │               │
│   │         │      exceeded          │         │               │
│   │  • All  │                        │  • All  │               │
│   │    calls│                        │    calls│               │
│   │    pass │                        │    fail │               │
│   │  • Count│                        │    fast │               │
│   │    errs │                        │  • Wait │               │
│   │         │                        │    for  │               │
│   │         │                        │    timeout              │
│   └────┬────┘                        └────┬────┘               │
│        │                                  │                     │
│        │ success                          │ timeout expired   │
│        │ (reset count)                    │                   │
│        │                                  ▼                   │
│        │                            ┌─────────┐               │
│        │         success           │HALF-OPEN│               │
│        │ ◄───────────────────────── │         │               │
│        │                            │  • Limited│               │
│        └──────────────────────────── │    calls  │               │
│              (close breaker)        │  • Test   │               │
│                                      │    health │               │
│                                      └─────────┘               │
│                                                                 │
│  State Transitions:                                             │
│  • CLOSED → OPEN: Failure threshold exceeded                    │
│  • OPEN → HALF-OPEN: Timeout expired                            │
│  • HALF-OPEN → CLOSED: Success with limited calls             │
│  • HALF-OPEN → OPEN: Failure during test                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

### Pattern Composition

```
┌─────────────────────────────────────────────────────────────────┐
│                    Layered Resilience                             │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  Request ──▶ [Retry] ──▶ [Circuit Breaker] ──▶ [Bulkhead] ──▶   │
│                                                                 │
│  Layer 1: Retry                                                 │
│  - 3 attempts with exponential backoff                        │
│  - Jitter to prevent thundering herd                            │
│                                                                 │
│  Layer 2: Circuit Breaker                                       │
│  - 5 failures in 60s opens circuit                                │
│  - 30s timeout before half-open                                 │
│  - 3 successful probes to close                                 │
│                                                                 │
│  Layer 3: Bulkhead                                              │
│  - Max 10 concurrent calls                                       │
│  - Queue of 20 pending                                           │
│  - Reject when full (fail fast)                                 │
│                                                                 │
│  Layer 4: Timeout                                               │
│  - 5s per attempt                                                │
│  - Cancel context on timeout                                     │
│                                                                 │
│  Layer 5: Fallback                                              │
│  - Return cached value on failure                               │
│  - Or return default/empty response                             │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

---

## Quality Standards

### Testing Requirements

#### Test Coverage
- **Minimum Coverage**: 80% for resilience patterns
- **Critical Paths**: 95% for circuit breaker state transitions
- **Failure Injection**: Required for all patterns

#### Test Categories
```bash
# Rust tests
cd rust && cargo test --workspace
cd rust && cargo test --workspace --features stress-test

# Go tests
cd go && go test ./...
cd go && go test ./... -race

# Python tests
pytest python/ -v
pytest python/ --stress
```

### Code Quality

#### Python Standards
```bash
# Linting
ruff check python/
mypy python/pheno-resilience/

# Formatting
black python/
ruff format python/

# Testing
pytest python/ -v --cov=pheno_resilience
```

#### Rust Standards
```bash
# Linting
cargo clippy --workspace --all-targets --all-features -- -D warnings

# Formatting
cargo fmt --all

# Testing
cargo test --workspace
cargo test --workspace --features stress-test

# Documentation
cargo doc --workspace --no-deps
```

#### Go Standards
```bash
# Linting
golangci-lint run

# Formatting
gofmt -l -w .

# Testing
go test -race ./...
```

### Resilience Standards

| Pattern | Metric | Target | Test |
|---------|--------|--------|------|
| Circuit Breaker | Response time (OPEN) | < 1ms | unit test |
| Retry | Recovery time | < 10s | integration |
| Bulkhead | Isolation | 100% | failure injection |
| Timeout | Cancellation | 100% | unit test |
| Fallback | Availability | 99.9% | chaos test |

---

## Git Workflow

### Branch Strategy

```
main
  │
  ├── feature/circuit-breaker-metrics
  │   └── PR #33 → squash merge ──┐
  │                               │
  ├── feature/bulkhead-queueing    │
  │   └── PR #34 → squash merge ──┤
  │                               │
  ├── fix/retry-race-condition     │
  │   └── PR #35 → squash merge ──┤
  │                               │
  └── hotfix/state-machine ─────────┘
      └── PR #36 → merge commit
```

### Branch Naming

```
feature/<pattern>-<enhancement>
fix/<component>-<issue>
perf/<scope>-<optimization>
refactor/<language>-<scope>
docs/<topic>
chaos/<test-scenario>
```

### Commit Conventions

```
feat(circuit-breaker): add metrics and health endpoint

Exposes circuit breaker state and statistics via HTTP endpoint
for monitoring and alerting integration.

- State transitions counter
- Failure/success rate tracking
- Response time percentiles
- Health check integration

Closes #88

fix(bulkhead): resolve queue overflow under load

Queue size was not properly limited, causing memory issues
under extreme load. Now properly rejects when queue full.
```

---

## File Structure

```
ResilienceKit/
├── docs/                       # Documentation
│   ├── SPEC.md                 # This specification
│   ├── adr/                    # Architecture decisions
│   │   ├── ADR-001-circuit-breaker.md
│   │   ├── ADR-002-retry-strategy.md
│   │   └── ADR-003-rate-limiting.md
│   └── research/
│       └── RESILIENCE_PATTERNS_SOTA.md
│
├── python/                     # Python implementation
│   └── pheno-resilience/
│       ├── pyproject.toml
│       ├── src/pheno_resilience/
│       │   ├── __init__.py
│       │   ├── circuit_breaker.py  # Circuit breaker impl
│       │   ├── retry.py            # Retry strategies
│       │   ├── bulkhead.py         # Bulkhead pattern
│       │   ├── timeout.py          # Timeout handling
│       │   ├── fallback.py         # Fallback mechanisms
│       │   ├── error_handling.py   # Error classification
│       │   ├── error_handler.py    # Generic error handler
│       │   └── health.py           # Health checking
│       ├── examples/
│       │   ├── circuit_breaker_demo.py
│       │   ├── bulkhead_demo.py
│       │   └── retry_demo.py
│       └── tests/
│           ├── test_circuit_breaker.py
│           ├── test_retry.py
│           └── test_bulkhead.py
│
├── rust/                       # Rust implementation
│   ├── Cargo.toml
│   ├── phenotype-state-machine/  # State machine framework
│   │   └── src/lib.rs
│   ├── phenotype-async-traits/   # Async trait utilities
│   │   └── src/lib.rs
│   └── (additional crates TBD)
│
├── go/                         # Go implementation (planned)
│   ├── go.work
│   ├── pheno-retry/
│   ├── pheno-circuitbreaker/
│   └── pheno-bulkhead/
│
├── README.md
└── AGENTS.md                   # This file
```

---

## CLI Commands

### Python Commands

```bash
# Run resilience examples
python -m pheno_resilience examples

# Health check
python -m pheno_resilience health-check

# Run tests
pytest python/pheno-resilience/ -v

# Stress test
pytest python/pheno-resilience/ --stress --duration 60
```

### Rust Commands

```bash
# Build all crates
cd rust && cargo build --workspace

# Run tests
cd rust && cargo test --workspace

# Run examples
cd rust && cargo run --example circuit_breaker
cd rust && cargo run --example state_machine

# Stress tests
cd rust && cargo test --features stress-test
```

### Go Commands (Planned)

```bash
# Build
cd go && go build ./...

# Run tests
cd go && go test ./...

# Run examples
cd go && go run examples/retry.go
```

---

## Troubleshooting

### Common Issues

#### Issue: Circuit breaker not opening despite failures

**Symptoms:**
Circuit stays CLOSED even with multiple failures.

**Diagnosis:**
```bash
# Check failure threshold
python -c "
from pheno_resilience import CircuitBreaker
cb = CircuitBreaker(failure_threshold=5, recovery_timeout=30)
print(f'Threshold: {cb.failure_threshold}')
print(f'Current failures: {cb.failure_count}')
"

# Monitor state transitions
python -c "
from pheno_resilience import CircuitBreaker
cb = CircuitBreaker()
cb.on_state_change = lambda old, new: print(f'{old} -> {new}')
"
```

**Resolution:**
- Verify failure_threshold is appropriate
- Check that exceptions are being counted
- Ensure not catching exceptions before circuit breaker
- Review sliding window vs consecutive failure settings

---

#### Issue: Retry storms causing overload

**Symptoms:**
Multiple clients retrying simultaneously overwhelm recovering service.

**Diagnosis:**
```bash
# Check retry configuration
python -c "
from pheno_resilience import RetryPolicy
policy = RetryPolicy()
print(f'Max retries: {policy.max_retries}')
print(f'Base delay: {policy.base_delay}')
print(f'Jitter: {policy.jitter}')
"
```

**Resolution:**
- Enable jitter: `RetryPolicy(jitter=True)`
- Use exponential backoff with cap
- Add circuit breaker before retry
- Implement rate limiting at edge

---

#### Issue: Bulkhead rejecting all requests

**Symptoms:**
All requests rejected with "Bulkhead full" error.

**Diagnosis:**
```bash
# Check bulkhead capacity
python -c "
from pheno_resilience import Bulkhead
bh = Bulkhead(max_concurrent=10, max_queue=20)
print(f'Available: {bh.available_concurrent}')
print(f'Queue size: {bh.queue_size}')
print(f'Utilization: {bh.utilization_percent}%')
"
```

**Resolution:**
- Increase max_concurrent if CPU/memory allows
- Check for request leaks (not releasing bulkhead)
- Review timeout settings (long timeouts hold slots)
- Consider separate bulkheads for different priorities

---

#### Issue: Timeouts not being enforced

**Symptoms:**
Operations run longer than configured timeout.

**Diagnosis:**
```bash
# Check timeout propagation
python -c "
import asyncio
from pheno_resilience import timeout

async def test():
    async with timeout(5):
        await long_operation()
"
```

**Resolution:**
- Ensure using async timeout with async operations
- Check that operation respects cancellation
- Verify timeout is being passed through call chain
- Use proper context manager: `async with timeout:`

---

### Debug Mode

```bash
# Python debug logging
export RESILIENCE_DEBUG=1
export RESILIENCE_LOG_LEVEL=debug
python -m pheno_resilience 2>&1 | tee debug.log

# Rust debug
export RUST_LOG=debug
cargo run --example circuit_breaker 2>&1 | tee debug.log

# Metrics export
export RESILIENCE_METRICS_ENABLED=1
export RESILIENCE_METRICS_PORT=9090
```

### Recovery Procedures

```bash
# Manual circuit breaker reset
python -c "
from pheno_resilience import CircuitBreaker
cb = CircuitBreaker()
cb.reset()  # Force to CLOSED
"

# Clear retry caches
python -c "
from pheno_resilience import RetryPolicy
RetryPolicy.clear_cache()
"

# Emergency bulkhead release
python -c "
from pheno_resilience import Bulkhead
bh = Bulkhead()
bh.release_all()  # Caution: may cause issues
"

# Reset all health checks
python -m pheno_resilience health-reset
```

---

## Agent Self-Correction & Verification Protocols

### Critical Rules

1. **State Consistency**
   - Circuit breaker states must be thread-safe
   - State transitions must be atomic
   - Metrics updates must not block operations
   - Health checks must not affect performance

2. **Resource Management**
   - Always release bulkhead slots in finally blocks
   - Cancel contexts on timeout
   - Clean up retry state on success
   - Don't leak goroutines/threads

3. **Observability**
   - Log all state transitions
   - Emit metrics for all patterns
   - Provide health endpoints
   - Support distributed tracing

4. **Testing**
   - Use failure injection for tests
   - Verify behavior under extreme load
   - Test race conditions
   - Validate recovery procedures

---

*This AGENTS.md is a living document. Update it as ResilienceKit evolves.*
