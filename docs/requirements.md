# PayLink — System Requirements & Architecture Specification

* **Project:** PayLink Utility Bill-Payment & Digital Wallet System
* **Course:** Software Design — Year Work Project
* **Phase:** Phase 1 Deliverable (Requirements, Design, DevOps Foundation)
* **Status:** Final Declared Stack (Locked for Phase 1 & Phase 2)
* **Document Owners:** Mohamed Ahmed (Architecture Lead), Youssef (Requirements & Product Lead)

---

## 1. Technology Stack Declaration

As mandated by Section 3.4 of the *PayLink Student Brief* and Task Distribution item **A7**, the engineering team formally declares the technology stack below. This declared stack is locked in Phase 1 and establishes the foundational runtime, architectural, and deployment standards for all subsequent development throughout Phase 2.

### 1.1 Declared Stack Summary Table

| Component / Layer | Technology & Version | Role in PayLink System | Architectural Justification & Brief Compliance |
| :--- | :--- | :--- | :--- |
| **Backend Runtime** | **Java 21 (LTS)** | Core backend language | Modern long-term support release providing records, pattern matching, robust type safety, and high-throughput virtual threads. Strict usage of `BigDecimal` for all monetary arithmetic. |
| **Backend Framework** | **Spring Boot** | Modular Monolith application framework | Spring WebMvc for REST controllers, Spring Validation for boundary validation, Spring Data JPA for persistence, and Spring Actuator for health checks (`/actuator/health`). |
| **Architectural Style** | **Layered Modular Monolith (Clean Architecture)** | System structure & domain boundaries | Exactly 8 distinct functional packages (`Accounts and Wallet`, `Biller Catalog`, `Payment Processing`, `Biller Integration`, `Fees and Commissions`, `Fraud Checks`, `Notifications`, `Reconciliation`). Domain entities and business logic enforce the Clean Architecture Dependency Rule (zero dependencies on web controllers or database drivers). |
| **Relational Database** | **PostgreSQL 17** | Persistent relational storage & ACID transactions | Enterprise ACID compliance with row-level locking (`PESSIMISTIC_WRITE`) for atomic wallet hold operations to prevent concurrency race conditions and negative balances. Native `JSONB` for flexible biller parameters, and `NUMERIC(19,2)` for zero-loss financial precision. |
| **Database Migrations** | **Flyway** | Schema versioning and DDL migrations | All table definitions, foreign keys, unique constraints, and audit tables are maintained via versioned SQL scripts stored in the repository (`backend/src/main/resources/db/migration/`). |
| **Frontend Client** | **Next.js (React / TypeScript)** | Customer and agent web interface | Lightweight, responsive web interface for customer bill inquiries, payment confirmation, transaction history, and wallet top-ups. |
| **API Specification** | **RESTful APIs + OpenAPI 3.0** | Public & internal API contracts | Minimum of 20 OpenAPI-documented REST endpoints with structured JSON request/response DTOs and HTTP status codes. |
| **Asynchronous Events** | **Spring Application Events (Evolving to Broker in Phase 2)** | Decoupled event-driven flows | Decoupled Observer pattern for `PaymentEventListener` to process notifications (SMS, Push) and ledger reconciliation without blocking real-time payment completion. |
| **Containerization** | **Docker & Docker Compose** | Local containerized deployment | Multi-stage `Dockerfile` for backend and frontend. Single `docker-compose.yml` orchestrating `postgres`, `backend`, `frontend`, and Phase 2 mock billers (`ElectroCo`, `AquaCo`, `TopUpNet`). |
| **CI/CD Automation** | **GitHub Actions** | Automated build, test, and quality pipeline | Multi-stage pipeline (`.github/workflows/ci.yml`) triggered on every push and pull request to `main`: builds backend/frontend, enforces linting, and runs automated tests against a containerized PostgreSQL service. |
| **Testing Frameworks** | **JUnit 5, Mockito, AssertJ** | Automated unit, concurrency, and hard-case testing | Automated verification of domain rules, 8 hard cases (idempotency, 10s timeouts, concurrent debits, biller refusals), and domain logic test coverage target (>70%). |

---

### 1.2 Architectural Commitment & Invariants

1. **Monetary Correctness:** All monetary attributes (`amount`, `fee`, `totalAmount`, `dailyLimits`, `commission`) are strictly implemented with Java `BigDecimal` and PostgreSQL `NUMERIC(19,2)`. Floating-point types (`float`, `double`) are strictly prohibited across all domain entities, APIs, and database columns.
2. **Double-Entry Style Ledger Integrity:** The `Wallet` entity and database table **never store a balance column**. `availableBalance` and `heldBalance` are computed dynamically by summing immutable `LedgerEntry` records. Direct overwriting of account balances is architecturally impossible.
3. **Strict Payment Lifecycle:** Payments transition exclusively across the 8 declared states (`Initiated`, `FundsHeld`, `SentToBiller`, `Completed`, `Failed`, `Unknown`, `Rejected`, `Refunded`) through the 9 canonical transitions governed by `PaymentStateHandler <<State>>`. Direct automatic refunds from `Unknown` are prohibited.
4. **Isolated Biller Integration:** Each biller protocol (REST/JSON, SOAP/XML, Async Callback, Batch CSV) is encapsulated behind `BillerAdapter <<Adapter>>` and instantiated via `BillerAdapterFactory <<Factory>>`. Adding or modifying a biller requires zero modifications to core payment orchestration or ledger modules.
5. **Phase 2 Stability Lock:** This declared stack is locked for Phase 1 and will remain unchanged through Phase 2. Architecture evolution in Phase 2 will focus on externalizing the event bus for Notifications and Reconciliation through an asynchronous message broker, without altering the core stack foundation.
