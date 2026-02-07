# WorldBite - International Snacks E-Commerce Platform

A full-stack e-commerce platform built with **Node.js/TypeScript** and **Express**, demonstrating enterprise-grade architectural patterns including circuit breaker, retry with exponential backoff, multi-tier rate limiting, and structured observability.

> **Note:** This is a documentation-only showcase repository. The source code is maintained in a private repository.

**Demo Video:** [Watch on Google Drive](https://drive.google.com/file/d/1G00v-e9X5pWtjM8sbdRXn3g-gxd9xSYY/view?usp=sharing)

---

## Tech Stack

| Layer | Technology | Purpose |
|-------|-----------|---------|
| **Runtime** | Node.js 20 + TypeScript 5.9 | Type-safe backend with ES modules |
| **Framework** | Express.js 5.1 | REST API with middleware chain |
| **Database** | SQLite3 / PostgreSQL (via Knex) | Relational data with query builder |
| **Auth** | JWT + bcrypt | Token-based sessions, password hashing |
| **Validation** | Zod | Schema-based input validation |
| **Observability** | Winston + custom Prometheus collectors | Structured logging, metrics, request tracing |
| **Testing** | Vitest + Supertest | Unit, integration, and E2E tests |
| **Containerization** | Docker (multi-stage) + Compose | Production and development environments |
| **Frontend** | HTML5/CSS3/JavaScript | 21 responsive pages served by Express |

---

## Architecture

```
Client (Browser)
     |
     v
Express Server ─── Middleware Chain ─────────────────────────────
     |               |            |           |         |
     |          Rate Limiter   Auth (JWT)   Logger   Metrics
     |
     v
Route Handlers ──── /api/products, /api/orders, /api/auth,
     |               /api/partner, /api/sales, /api/returns,
     |               /api/monitoring, /api/profile
     v
Service Layer ───── Payment Strategies (Card/Cash)
     |               Feed Intermediary (CSV processing)
     |               Resilience Service
     |                  ├── Circuit Breaker
     |                  ├── Retry (exponential backoff + jitter)
     |                  └── Transaction Rollback
     v
DAO Layer ────────── ProductDAO, OrderDAO, UserDAO, SaleDAO, PartnerDAO
     |
     v
SQLite / PostgreSQL
```

---

## Architectural Patterns

### Circuit Breaker

Monitors downstream dependencies (database, payment processor) and fails fast when they're unhealthy, preventing cascading failures.

- **States:** CLOSED → OPEN → HALF_OPEN
- **Threshold:** 5 consecutive failures triggers circuit open for 60 seconds
- **Monitoring:** Real-time state visible via `/health/detailed` endpoint

### Retry with Exponential Backoff

Handles transient failures (network timeouts, database locks) with intelligent retry logic.

- **Attempts:** 3 max with 1s → 2s → 4s delays
- **Jitter:** +/-25% to prevent thundering herd
- **Retryable errors:** ECONNRESET, ETIMEDOUT, SQLITE_BUSY, 5xx responses
- **Coordination:** All retry attempts count as one circuit breaker attempt

### Multi-Tier Rate Limiting

Different rate limits per endpoint based on risk profile:

| Endpoint | Limit | Rationale |
|----------|-------|-----------|
| Auth (login) | 5 req / 15 min | Brute-force prevention |
| Orders | 10 req / min | Prevent duplicate orders |
| Products | 50 req / min | Allow browsing |
| Sales analytics | 20 req / 5 min | Heavy queries |
| Partner uploads | 15 req / 2 min | CSV processing load |

### Transaction Rollback

Wraps multi-step database operations (order creation: validate stock → create order → deduct inventory → process payment) with automatic rollback on any step failure.

### Strategy Pattern (Payments)

Abstract `IPaymentStrategy` interface with `CardPayment` and `CashPayment` implementations. New payment methods can be added without modifying existing code.

### DAO Pattern (Data Access)

Five DAO classes encapsulate all database operations, isolating query logic from business logic and enabling easy migration between SQLite and PostgreSQL.

---

## Observability

### Structured Logging (Winston)

Every request gets a UUID correlation ID that flows through all log entries:

```
[2025-12-05T13:05:22.026Z] INFO:  POST /api/orders [48964090-...] - 201 - 45ms
[2025-12-05T13:05:22.027Z] INFO:  Database query  [48964090-...] - INSERT INTO sale
[2025-12-05T13:05:22.031Z] INFO:  Payment processed [48964090-...] - CARD - $42.50
```

When debugging, search by request ID to trace the complete lifecycle of any request.

### Metrics Collection

Prometheus-compatible metrics tracking:
- Request count, duration, status code distribution per endpoint
- Circuit breaker state transitions
- Rate limit trigger events
- P50/P95/P99 response times

### Health Endpoints

- `GET /health` — Basic liveness check
- `GET /health/detailed` — Circuit breaker states, rate limiting status, active connections, uptime

### Dashboards

- **Monitoring Dashboard:** Real-time API performance graphs
- **Admin Log Viewer:** Search/filter logs by request ID, user, level
- **Admin Dashboard:** Business KPIs and system health

---

## Features

### E-Commerce Core
- Product catalog with 21 international snacks from 15+ countries
- Shopping cart with localStorage persistence
- Checkout with stock validation and tax calculation
- Flash sales with time-based 30% discounts

### Order Management
- Complete order history with status/date/keyword filtering
- Return request workflow with multi-stage tracking (Submitted → Received → Inspected → Approved → Refunded)
- Automatic inventory restocking on approved returns

### Partner System
- CSV product feed upload with validation and error reporting
- Sales analytics dashboard with time-period filtering
- Undo functionality for batch uploads
- Low stock alerts with severity-based color coding

### Role-Based Access Control
- **Customer:** Browse, purchase, track orders, request returns
- **Partner:** Manage products, view analytics, monitor inventory
- **Admin:** System oversight, return disposition, health monitoring

### Notification System
- Low stock alerts (out of stock / critical / low) with notification bell
- Return status auto-refresh every 30 seconds
- Role-based notification filtering

---

## Testing

- **Unit tests:** DAO layer business logic (Vitest)
- **Integration tests:** API endpoint validation (Supertest)
- **E2E tests:** Complete user workflows (checkout, returns)
- **Resilience tests:** Circuit breaker behavior, rate limiting verification

---

## Deployment

- **Docker:** Multi-stage build (Alpine Node 20, ~150MB optimized)
- **Docker Compose:** Production and development configurations
- **Health checks:** Built into container for orchestrator integration
- **Database auto-init:** Detects first run and seeds schema + data

---

## Documentation

- [Architecture Decision Records](docs/architecture/) — Detailed ADRs for each quality attribute
- [Quality Scenarios Catalog](docs/architecture/QSCatalog.md) — 14 scenarios across 6 quality attributes
- [UML Diagrams](docs/diagrams/) — Package, deployment, class, and sequence diagrams

---

## My Contributions

- Designed and implemented the full backend architecture (Express + TypeScript + SQLite/Knex)
- Built the circuit breaker, retry, and transaction rollback resilience patterns
- Implemented the observability stack (Winston logging, Prometheus metrics, request correlation)
- Created the multi-tier rate limiting strategy
- Built the complete frontend (21 HTML pages with shopping cart, dashboards, monitoring)
- Containerized the application with Docker multi-stage builds
- Wrote unit, integration, and E2E test suites
- Authored Architecture Decision Records for all quality attributes

**Team:** Sophie Lin, Shahram
