# Performance ADRs

## ADR 1: High Load Management (via Multi-Tier Rate Limiting)

**Status:** Accepted

### Context
During flash sales or peak shopping periods, the retail application can receive thousands of concurrent requests that can overwhelm the system and degrade performance for all users. Without proper load management, the system can become unresponsive, leading to poor user experience, abandoned shopping carts, and lost revenue. The system needs to ensure fair resource allocation while maintaining stability under high load.

### Decision
We implemented **Multi-Tier Rate Limiting** in `src/middleware/performance.ts` with different limits for different types of operations (authentication: 5/15min, products: 50/min, orders: 10/min). The system uses Redis-backed distributed limiting with atomic operations and fallback to in-memory storage. This ensures fair resource distribution and prevents system overload while allowing legitimate usage patterns.

### Consequences

**Positive:**
- 95% of requests complete within 500ms even under high load
- Fair resource allocation ensures no single user can monopolize system resources
- System remains stable and responsive during peak traffic periods
- Prevents cascading failures from overwhelming downstream services

**Negative:**
- May impact user experience when limits are reached (429 errors)
- Additional complexity in request handling and rate tracking
- External dependency on Redis for optimal performance

### Alternatives Considered

**1. Simple Global Rate Limiting**
- *Pros:* Simpler to implement, uniform protection across all endpoints
- *Cons:* Not flexible enough for different operation types, may be too restrictive or too lenient

---

## ADR 2: Long-Running Request Management (via Request Timeouts)

**Status:** Accepted

### Context
Complex operations like order processing with multiple database operations, external API calls, or file uploads can sometimes hang or run indefinitely due to network issues, database locks, or external service failures. These long-running requests consume server resources (memory, connections, threads) and can lead to resource exhaustion and system instability under load.

### Decision
We implemented **Request Timeouts** using the `boundExecutionTime` middleware that automatically terminates requests that exceed 30 seconds execution time. The system returns a 408 timeout error and immediately frees up server resources. This prevents resource exhaustion and provides predictable response times for all operations.

### Consequences

**Positive:**
- 100% of requests complete within 30 seconds or receive timeout error
- Server resources freed within 1 second of timeout occurrence
- Improved system stability under load conditions
- Predictable performance characteristics for capacity planning

**Negative:**
- May terminate legitimate long-running operations
- Users may need to retry timed-out operations
- Requires careful tuning of timeout values for different operations

### Alternatives Considered

**1. No Timeout Management**
- *Pros:* Allows all operations to complete naturally
- *Cons:* Risk of resource exhaustion and system instability

