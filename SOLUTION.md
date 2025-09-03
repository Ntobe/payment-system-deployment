## 1. Architecture Overview

This solution is composed of two services:

- ### Ledger Service

  -   Manages accounts and balances.

  -   Ensures transfers are recorded atomically.

  -   Maintains an append-only ledger for auditing.

- ### Transfer Service

  -   Orchestrates transfers between accounts by calling the Ledger service.

  -   Handles retries and error responses from the Ledger service.

### Communication
- Services communicate over REST using Spring WebClient.

## 2. Transaction Model
Each transfer must be atomic:

- Debit the source account.

- Credit the destination account.

- Insert two LedgerEntry records (one DEBIT, one CREDIT).

This is implemented using `Spring transactions` with `pessimistic row-level locking`.

To prevent race conditions and negative balances, the following options were considered:

#### 1.  Optimistic locking

- Use JPA’s @Version column on Account.

- Requires retry logic when concurrent updates collide.

#### 2.  Pessimistic locking (SELECT … FOR UPDATE)

- Locks rows while a transfer is in progress.

- Prevents race conditions and avoids retries, at the cost of reduced concurrency under heavy load.

**Decision**: Pessimistic locking was chosen. This ensures correctness and consistency in financial transactions, where avoiding double-spend and overdrafts is more important than maximum throughput.

### 3.  Design Trade-offs

- #### Database

  -   H2 for local development (simplifies setup, no external dependency).

  -   PostgreSQL for production (robust, supports transactions and row-level locking).

- #### Build Tool

  -   **Gradle** (wrapper included so contributors don’t need Gradle installed).

  -   Faster incremental builds compared to Maven, better support for modern tooling.

- #### Error Handling

  -   Transfers that fail due to insufficient funds are still logged in the ledger with a FAILED status.

  -   Ensures visibility of all attempts, successful or not.

### 4.  Alternatives Considered

- Locking approach:

  -   Optimistic locks scale better but require retries.

  -   Pessimistic locks reduce concurrency but guarantee correctness. Chosen for safety in financial systems.

- **Gradle vs Maven**: Maven is simpler, but Gradle provides better performance and flexibility. Chose Gradle.

- **Direct JDBC vs Spring Data JPA**: JPA simplifies code and supports pessimistic locking natively. JDBC would provide finer control but at higher complexity.

### 5.  Future Improvements

- **Resilience**: Add retry with exponential backoff for transient downstream failures (e.g., network issues).

- **Scalability**: Introduce message queues (e.g., Kafka) for async transfer processing.

- **Tracing**: Add correlation ID for tracking user journey.
- **Monitoring**: Add metrics via Micrometer + Prometheus/Grafana dashboards.

- **Security**: Secure endpoints with OAuth2/JWT.

- **Testing**: Add integration tests with Testcontainers for Postgres.