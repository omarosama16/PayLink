# PayLink — Domain Class Diagram
## AI Design Prompt, Technical Review & Multi-Round Corrections

* **Date:** 2026-11-24
* **Author:** Mohamed Ahmed (Architecture Lead)
* **Workflow:**
  * **Prompt Optimization & Engineering:** Gemini 3.8 Flash (High)
  * **PlantUML Diagram Generation:** ChatGPT
* **Target Deliverable:** [`docs/design/class-domain.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/class-domain.puml)
* **Brief References:** Section 3.1 Modules (p. 7), Section 3.4 Technical & Patterns (p. 10), Task Distribution A2 (p. 4)

---

### 1. Specification & Context

PayLink is designed as a **Layered Modular Monolith** in Java 21 following the Clean Architecture Dependency Rule: domain entities and business rules must not depend on web frameworks, JPA annotations, or database drivers.

* **Modules (8 packages):** Accounts and Wallet, Biller Catalog, Payment Processing, Biller Integration, Fees and Commissions, Fraud Checks, Notifications, Reconciliation.
* **Design Patterns (7):** State, Adapter, Factory, Strategy, Chain of Responsibility, Observer, Command.
* **Ledger Invariant:** Wallet balances are strictly derived from immutable `LedgerEntry` records (`getAvailableBalance()`, `getHeldBalance()`). No stored balance attributes!

---

### 2. Initial Optimized Prompt

```text
ROLE:
You are a principal software architect who writes clean, accurate, and syntactically valid UML Domain Class Diagrams in PlantUML for Java 21 backend systems.

CONTEXT:
I am designing PayLink, a bill-payment and digital wallet backend in Java 21 (Spring Boot, PostgreSQL). We are building a Layered Modular Monolith following the Clean Architecture Dependency Rule: domain entities and business rules must NOT depend on web frameworks, controllers, JPA annotations, or database implementations.

This diagram is the Domain Class Diagram (/docs/design/class-domain.puml) and must be consistent with our Payment State Diagram and our PostgreSQL ER Diagram.

MODULES TO INCLUDE (GROUP EACH IN A PLANTUML PACKAGE)
1. Accounts and Wallet (Customer, Agent, Wallet with NO stored balance attributes, LedgerEntry, AuditLog)
2. Biller Catalog (Biller, BillerService, ServiceCategory)
3. Payment Processing (Payment, PaymentTransition, PaymentState, PaymentOrchestrator, RetryCommand)
4. Biller Integration (BillerAdapter, ElectroCoAdapter, AquaCoAdapter, TopUpNetAdapter, NetCoAdapter, BillerAdapterFactory)
5. Fees and Commissions (FeeStrategy, FixedFeeStrategy, PercentageFeeStrategy, TieredFeeStrategy, PromotionFeeStrategy, FeeAndLimitConfig, CommissionCalculator)
6. Fraud Checks (FraudCheckHandler, BlockedAccountCheck, SinglePaymentLimitCheck, DailyLimitCheck, VelocityCheck)
7. Notifications (PaymentEventListener, Notification, NotificationRetryCommand)
8. Reconciliation (SettlementFile, SettlementRecord, ReconciliationService, ReconciliationReport, ReconciliationDiscrepancy)

DESIGN PATTERNS TO MARK EXPLICITLY:
State, Adapter, Factory, Strategy, Chain of Responsibility, Observer, Command.

CONSTRAINTS:
- Layered Modular Monolith: represent each of the 8 modules as its own package.
- Use Java 21 exact decimal type BigDecimal for ALL monetary fields and return types.
- Wallet MUST NOT have stored balance attributes; balances are strictly derived methods.
- Use exact UML relationships with clear multiplicities.
```

---

### 3. First AI Output Review & 6 Defects Identified

The initial diagram generated had several structural and domain defects:

1. **Duplicate Class Declaration:** Declared `class FeeAndLimitConfig` in both `Fees and Commissions` and `Fraud Checks`.
2. **Weak State Pattern:** Only declared `enum PaymentState <<State>>` without an interface or concrete state behavior classes to govern transitions.
3. **Hardcoded Business Numbers in Code:** Strategy and check classes hardcoded defaults (`= 2.50`, `= 0.015`, `= 1.00`, `= 200.00`, `= 0.40`, `= 5`), violating Section 2.6 which demands these values live in runtime configuration (`FeeAndLimitConfig`).
4. **Missing Payment Module Linkages:** `Payment` lacked associations to `Agent` (optional processing) and `Notification` (traceability back to payment).
5. **Missing Domain Operations on Wallet:** `Wallet` only had query methods but lacked state-changing domain methods (`topUp`, `hold`, `releaseHold`, `captureHold`).
6. **Misconfigured AuditLog Scope:** Associated `Customer "1" --> "0..*" AuditLog`, omitting agents, support staff, and admin actors.

---

### 4. First Corrective Prompt (Architecture Refactoring)

```text
Your first diagram has several design and specification defects. Fix ALL 6 issues below and regenerate the complete PlantUML diagram:

1. Remove the duplicate 'class FeeAndLimitConfig' declaration inside 'Fraud Checks'. Reference the single FeeAndLimitConfig defined in 'Fees and Commissions'.
2. Make the State pattern real: keep 'enum PaymentState' with the exact 8 states for ER/State diagram consistency, and add 'interface PaymentStateHandler <<State>>' (holdFunds, sendToBiller, confirm, fail, markUnknown, reject, refund) used by Payment to enforce only the 9 allowed transitions.
3. Fix configuration & business rule violations:
   - Remove ALL hardcoded numeric defaults from strategy/calculator/check classes; inject/read them from FeeAndLimitConfig.
   - In FeeAndLimitConfig, replace single dailyPaymentLimit with dailyLimits: Map<VerificationLevel, BigDecimal> and add promotionBillerCode.
   - Add FeeCalculator class that selects the right FeeStrategy using FeeAndLimitConfig.
4. Wire PaymentOrchestrator cleanly to the modules it coordinates using dependencies (..>): Wallet, FraudCheckHandler, FeeCalculator, CommissionCalculator, BillerAdapterFactory, and PaymentEventListener.
5. Fix missing domain relationships and Observer pattern:
   - Add concrete NotificationEventListener implementing PaymentEventListener <<Observer>>.
   - Add an association from Notification to Payment ("0..*" --> "1").
   - Add an optional association from Payment to Agent ("0..*" --> "0..1").
   - Add actorRole to AuditLog and remove the direct Customer-only association.
   - Add domain methods to Wallet: topUp(amount, ref), hold(amount, ref), releaseHold(amount, ref), captureHold(amount, ref).
6. Ensure 100% valid PlantUML syntax inside a single @startuml ... @enduml block.
```

---

### 5. Second AI Defect Identified (Invalid State Transitions Hallucination)

When the AI regenerated the diagram, it added `PaymentStateHandler` but hallucinated invalid transitions:
```plantuml
note right of PaymentStateHandler
    Exactly nine allowed transitions:
    Initiated -> FundsHeld : holdFunds
    Initiated -> Rejected : reject
    FundsHeld -> SentToBiller : sendToBiller
    FundsHeld -> Failed : fail           <-- DEFECT: Invalid shortcut transition
    SentToBiller -> Completed : confirm
    SentToBiller -> Failed : fail
    SentToBiller -> Unknown : markUnknown
    Unknown -> Completed : confirm      <-- DEFECT: Missing Unknown -> Failed
    Completed -> Refunded : refund      <-- CRITICAL DEFECT: Completed is immutable/final!
end note
```

* **Critical Catch:** The AI allowed `Completed -> Refunded`. In PayLink, `Completed` is a final terminal state. Refunds only apply to failed payments where held money is returned (`Failed -> Refunded`).

---

### 6. Second Corrective Prompt (Student Intervention: "not right states")

```text
not right states
Ensure PaymentStateHandler enforces the exact nine allowed transitions:

Initiated -> FundsHeld : holdFunds
Initiated -> Rejected : reject

FundsHeld -> SentToBiller : sendToBiller

SentToBiller -> Completed : confirm
SentToBiller -> Failed : fail
SentToBiller -> Unknown : markUnknown

Unknown -> Completed : confirm
Unknown -> Failed : fail

Failed -> Refunded : refund

Completed, Rejected, and Refunded are final.
Unknown is never refunded automatically.
```

---

### 7. Final Accepted Output

* **File:** [`docs/design/class-domain.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/class-domain.puml)
* **Verdict:** Fixed and accepted after 2 rounds of critical evaluation.
