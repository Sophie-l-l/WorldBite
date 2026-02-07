# ADR 005: Resilience Tactics

## Status
Accepted

## Context
The application must tolerate transient failures (database locks, slow operations), prevent cascading degradation, and provide predictable error boundaries. Early versions lacked systematic retry/circuit breaking and allowed long-running operations to tie up resources.

## Decision
Adopt the following resilience tactics implemented in custom utilities and middleware:

### Tactics
1. Circuit Breaker (`resilientService.executeResilient`)
   - Tracks state (CLOSED → OPEN → HALF_OPEN) for named resources (e.g., `database-products`).
   - Fails fast when error threshold exceeded, reducing load and giving time to recover.
2. Targeted Retries
   - Retry only for retryable errors (e.g., `SQLITE_BUSY`, `SQLITE_LOCKED`).
   - Exponential backoff built into resilience layer (deferred full implementation but pattern established).
3. Execution Time Bounding (`boundExecutionTime` middleware)
   - Global 5-minute cap for large payload/edge cases; earlier tighter bounds were applied per route.
4. Rate Limiting (per-router via `rateLimiters`)
   - Prevents abusive bursts; different limits for auth, orders, products, partner actions.
5. Transaction Rollback (`TransactionHandler.executeWithRollback`)
   - Ensures consistency when creating products or sales; partial failures do not corrupt state.
6. Input Validation + Authorization
   - Guards against malformed payloads (Zod schemas) and permission misuse, reducing unexpected runtime paths.
7. Observability Tie-In
   - Error rate & latency metrics feed health assessment; high error bursts escalate status to `warning`/`critical`.

## Alternatives Considered
| Option | Why Rejected |
|--------|--------------|
| External service mesh resilience (Istio/Linkerd) | Overkill for monolith + SQLite; operational complexity |
| Generic library for circuit breaking (e.g., `opossum`) | Custom needs were simple; reduced dependency surface |
| Global retry wrapper around all DB queries | Risk of amplifying load; chosen targeted approach instead |

## Consequences
Positive:
- Fast-fail behavior protects system under persistent DB lock conditions.
- Reduced risk of partial writes via transaction wrapping.
- Metrics-driven health endpoints enable automated alerting integration later.

Trade-offs:
- Custom resilience code needs future hardening (jitter, backoff, metrics export for breaker state).
- Circuit breaker granularity currently by logical service name; may need finer partitioning.

## Future Improvements
- Add configurable backoff/jitter policy object.
- Expose breaker state via `/api/monitoring/health/detailed` for dashboards.
- Separate read vs. write circuit contexts for database operations.
