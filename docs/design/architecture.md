# PayLink — Architectural Blueprint & Design Note

* **Document:** System Architecture Note (`/docs/design/architecture.md`)
* **Project:** PayLink Bill-Payment & Digital Wallet Backend
* **Course:** Software Design — Year Work Project
* **Author:** Mohamed Ahmed (Architecture Lead)
* **Date:** 2026-11-24
* **Status:** Phase 1 Architectural Baseline (Approved)
* **Brief & Task References:** Brief Section 3.4 (p. 10), Task Distribution A7 (p. 4)

---

## 1. Architectural Style: Layered Modular Monolith

PayLink is designed as a **Layered Modular Monolith**. Instead of premature microservices deployment—which introduces distributed transaction complexities, network latency, and operational overhead—the system is organized into strictly bounded, loosely coupled modules within a single, highly maintainable Java 21 application.

### 1.1 Module Boundaries and Public Interfaces
The domain is partitioned into **8 distinct functional modules**, each residing in its own dedicated package. Each module communicates across boundaries exclusively through a **well-defined public interface**, keeping internal domain entities, state handlers, and helper classes package-private or internal:

| Module / Package | Primary Responsibility | Public Interface / Entry Point | Internal Dependencies |
| :--- | :--- | :--- | :--- |
| **1. Accounts & Wallet** | Customer accounts, verification levels, agents, immutable ledger entries, hold/release lifecycle. | `WalletService`, `Wallet` domain API | None (Core Domain) |
| **2. Biller Catalog** | Biller registry, service metadata, input validation schemas, min/max constraints. | `BillerCatalogService` | None (Catalog Domain) |
| **3. Payment Processing** | Payment state machine, idempotency control, end-to-end payment orchestration. | `PaymentOrchestrator` | Accounts & Wallet, Biller Catalog, Fees & Commissions, Fraud Checks, `BillerAdapter` (Domain Interface) |
| **4. Biller Integration** | Protocol adapters (REST/JSON, SOAP/XML, Async Callback, Batch CSV) and retry logic. | `BillerAdapterFactory`, Concrete Adapters | Biller Catalog, Shared Payment Contracts (`BillerAdapter` implementation via Dependency Inversion) |
| **5. Fees & Commissions** | Dynamic fee strategies (Fixed, Percentage, Tiered, Promotion), agent commissions, runtime config. | `FeeCalculator`, `CommissionCalculator` | Biller Catalog |
| **6. Fraud Checks** | Chain of Responsibility rules (Blocked account, velocity check, daily limits, single limits). | `FraudCheckHandler` | Accounts & Wallet, Fees & Limits |
| **7. Notifications** | Decoupled notification dispatch (SMS, Push, Email), retry policies. | `PaymentEventListener` (Observer) | Payment Processing (Events) |
| **8. Reconciliation** | Nightly batch comparison of settlement files against PayLink ledger; discrepancy reporting. | `ReconciliationService` | Payment Processing, Settlement Records |

> **Architectural Note on Dependency Inversion (Zero Circular Dependency):**
> `Payment Processing` does **not** depend on the concrete biller adapters. It depends solely on the abstract domain interface `BillerAdapter`. The `Biller Integration` module implements this interface (`ElectroCoAdapter ..|> BillerAdapter`). Source code dependencies point strictly inward toward the domain abstraction, fully adhering to Clean Architecture with no circular coupling.

```
+-----------------------------------------------------------------------------------+
|                               PAYLINK MONOLITH                                    |
|                                                                                   |
|  +--------------------+   +---------------------+   +--------------------------+  |
|  |  Accounts & Wallet |   |   Biller Catalog    |   |   Fees & Commissions     |  |
|  +---------^----------+   +----------^----------+   +------------^-------------+  |
|            |                         |                           |                |
|            +-------------------------+---------------------------+                |
|                                      |                                            |
|                        +-------------+--------------+                             |
|                        |     Payment Processing     |                             |
|                        |   (PaymentOrchestrator)    |                             |
|                        +-------------+--------------+                             |
|                                      |                                            |
|            +-------------------------+---------------------------+                |
|            |                         |                           |                |
|  +---------v----------+   +----------v----------+   +------------v-------------+  |
|  | Biller Integration |   |     Fraud Checks    |   |      Notifications       |  |
|  +--------------------+   +---------------------+   +--------------------------+  |
|                                                                                   |
|  +--------------------+                                                           |
|  |   Reconciliation   |  (Nightly Batch Verification against Settlement Files)     |
|  +--------------------+                                                           |
+-----------------------------------------------------------------------------------+
```

---

## 2. Clean Architecture & The Dependency Rule

PayLink strictly enforces **The Dependency Rule** from Clean Architecture (Robert C. Martin):

> **"Source code dependencies must point only inward, toward higher-level policies."**

```
                 +---------------------------------------+
                 |       Frameworks & Drivers            |
                 |  (Spring Boot, Web MVC, PostgreSQL)   |
                 |   +-------------------------------+   |
                 |   |     Interface Adapters        |   |
                 |   |   (REST Controllers, DTOs,    |   |
                 |   |    JPA Repositories/Mappers)  |   |
                 |   |   +-----------------------+   |   |
                 |   |   |   Application Services|   |   |
                 |   |   |   +---------------+   |   |   |
                 |   |   |   | Domain Core   |   |   |   |
                 |   |   |   | (Entities,    |   |   |   |
                 |   |   |   |  State Pattern|   |   |   |
                 |   |   |   |  Ledger Rules)|   |   |   |
                 |   |   |   +---------------+   |   |   |
                 |   |   +-----------------------+   |   |
                 |   +-------------------------------+   |
                 +---------------------------------------+
                           Inward Dependency Direction --->
```

### 2.1 Concrete Architectural Guardrails:
1. **Domain Isolation:** Domain entities (`Payment`, `Wallet`, `LedgerEntry`, `FeeRule`) are pure Java 21 classes (POJOs and Records). They have **zero dependencies** on Spring Boot, Web MVC annotations (`@RestController`, `@RequestParam`), or database libraries.
2. **Persistence Decoupling (Inversion of Control):** Domain models do **not** contain JPA annotations (`@Entity`, `@Table`, `@Id`). Persistence is an implementation detail handled by outer-layer JPA entities and data mappers. The domain defines repository interfaces; outer infrastructure modules implement them.
3. **Immutable Append-Only Ledger Protection:** In strict alignment with the Brief (Section 2.3) and `er.puml`, `ledger_entries` is an immutable, append-only transaction log recording singular money movements (`TOP_UP`, `HOLD`, `RELEASE_HOLD`, `DEBIT`, `FEE_INCOME`, `COMMISSION`, `REFUND`) per wallet. The system never performs in-place balance overwrites. Account balances (`availableBalance` and `heldBalance`) are computed dynamically by aggregating ledger entries, providing an unalterable audit trail without the overhead of dual-account book balancing.
4. **Isolated Biller Implementations:** External protocol nuances (e.g., AquaCo SOAP XML vs. ElectroCo REST JSON) are isolated inside adapter implementations (`AquaCoAdapter`, `ElectroCoAdapter`). The internal payment engine interacts only through the domain `BillerAdapter` contract.

---

## 3. Phase 2 Architectural Evolution (Roadmap)

While Phase 1 establishes the Modular Monolith baseline with in-process event dispatching (`Spring ApplicationEventPublisher` / direct observer calls), **Phase 2 will evolve the architecture toward an event-driven style** for high-latency and asynchronous operations:

```
+---------------------------+                      +-----------------------------+
|    Payment Processing     |                      |       Message Broker        |
|  (Completes core payment  | -- publish events -> |   (RabbitMQ / Kafka via     |
|   and commits ledger)     |                      |      Docker Compose)        |
+---------------------------+                      +--------------+--------------+
                                                                  |
                                      +---------------------------+---------------------------+
                                      |                                                       |
                                      v                                                       v
                       +-----------------------------+                         +-----------------------------+
                       |    Notifications Service    |                         |   Reconciliation Service    |
                       |  (Asynchronous SMS / Push   |                         |  (Nightly file processing   |
                       |   with 3x retries & DLQ)    |                         |   & discrepancy analytics)  |
                       +-----------------------------+                         +-----------------------------+
```

### 3.1 Key Evolution Pillars for Phase 2:
1. **Asynchronous Message Broker Integration:**
   * Notifications (`SMS`, `Push`, `Email`) and nightly Reconciliation will be decoupled from the core HTTP request-response cycle using a message broker (RabbitMQ or Kafka) running inside `docker-compose.yml`.
   * A failure or latency spike in the third-party SMS provider will not delay or fail payment processing.
2. **Transactional Outbox Pattern:**
   * To prevent dual-write inconsistencies between the PostgreSQL ledger and the message broker, events will be written to an `outbox_events` table within the same database transaction that captures the payment hold, then dispatched asynchronously.
3. **Architecture Decision Records (ADRs):**
   * As required by Section 4 (Phase 2 Item 4) of the brief, three comprehensive ADRs will be created in `/docs/adr/`:
     * **ADR-01:** Monolith-first modular architecture vs. Microservices (Architecture Style).
     * **ADR-02:** In-memory events vs. External Message Broker for Notifications & Reconciliation (How Modules Communicate).
     * **ADR-03:** Data Ownership across Modules & Immutable Ledger Storage (Who owns which tables/data vs. shared database access, and append-only ledger vs. stored balances).
