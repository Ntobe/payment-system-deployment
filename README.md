# Payment System Deployment

## Running the System

This system is composed of **two independent Spring Boot services**, each hosted in its own repository:

- **Ledger Service** – Responsible for account balances and audit logs.  
  - Setup and run instructions: [Ledger Service README](https://github.com/Ntobe/ledger/blob/master/README.md)

- **Transfer Service** – Handles transfer requests and delegates to the Ledger Service (with circuit breaker protection).  
  - Setup and run instructions: [Transfer Service README](https://github.com/Ntobe/transfer/blob/master/README.md)

---

### Local Development Workflow

1. Clone and run **Ledger Service** following its [README instructions](https://github.com/Ntobe/ledger/blob/master/README.md).
    - By default, runs on port `8081`.

2. Clone and run **Transfer Service** following its [README instructions](https://github.com/Ntobe/transfer/blob/master/README.md).
    - By default, runs on port `8080`.

3. Access APIs via Swagger UI (only in `dev` profile):
    - Ledger Service → [http://localhost:8081/swagger-ui.html](http://localhost:8081/swagger-ui.html)
    - Transfer Service → [http://localhost:8080/swagger-ui.html](http://localhost:8080/swagger-ui.html)

---

- Each repository contains **build, test, and run instructions** specific to that service. 
