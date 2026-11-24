# PayLink — Sequence Diagram: Pay a Bill
## AI Design Prompt, Technical Review & Frame Refactoring

* **Date:** 2026-11-24
* **Author:** Mohamed Ahmed (Architecture Lead)
* **Workflow:**
  * **Prompt Optimization & Engineering:** Gemini 3.8 Flash (High)
  * **PlantUML Diagram Generation:** ChatGPT
* **Target Deliverable:** [`docs/design/sequence-pay-bill.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/sequence-pay-bill.puml)
* **Brief References:** Section 2.2 Payment Story (p. 2), Hard Cases 1, 2, 4, 5 (p. 6), Task Distribution A4 (p. 4)

---

### 1. Specification & Context

This diagram models the complete 9-step payment orchestration flow, covering:
* **Step 1:** Authentication & limits check.
* **Step 2:** Bill inquiry via `BillerAdapterFactory`.
* **Step 3:** Dynamic fee calculation (`FeeCalculator` & `FeeAndLimitConfig`).
* **Step 4 + Hard Case 1:** Idempotency check (returns original payment if duplicate key).
* **Step 4 (Fraud Checks):** `FraudCheckHandler` Chain of Responsibility.
* **Step 5 + Hard Case 4 (Atomic Concurrency Hold):** Check balance and append `HOLD` ledger entry under wallet lock (prevents negative balance).
* **Step 6:** Dispatch to biller with 10-second timeout.
* **Step 7 & 8:** Confirm (`SentToBiller -> Completed`), capture hold (`DEBIT` ledger entry), fee/commission distribution, asynchronous notification (`PaymentEventListener`), SMS/PUSH receipt.
* **Hard Case 2 (Timeout):** Transition `SentToBiller -> Unknown`, money stays held, `RetryCommand` scheduled (1, 5, 15 min).
* **Step 9:** Nightly reconciliation note.

---

### 2. Initial Optimized Prompt

```text
ROLE:
You are a senior software architect who writes clean, accurate, and syntactically valid UML Sequence Diagrams in PlantUML for Java 21 backend systems.

CONTEXT:
I am designing PayLink, a bill-payment and digital wallet backend built as a Layered Modular Monolith (Java 21, Spring Boot, PostgreSQL). This sequence diagram (/docs/design/sequence-pay-bill.puml) models the complete "Pay a Bill" flow and MUST be 100% consistent with:
- Payment State Diagram (/docs/design/state-payment.puml): exact 8 states and 9 transitions.
- Domain Class Diagram (/docs/design/class-domain.puml): exact class, interface, and method names.
- Use-Case Diagram: consistent actor names (Customer, Biller, NotificationProvider).

PARTICIPANTS (IN THIS EXACT ORDER):
Customer, PaymentOrchestrator, FeeCalculator, FraudCheckHandler, Wallet, Payment, BillerAdapter, Biller (ElectroCo), NotificationEventListener, NotificationProvider.

SCENARIO TO MODEL (ALL 9 STEPS + HARD CASES 1, 2, AND 4):
1. Login & Limits (identification & limits check).
2. Bill Inquiry (meter inquiry via adapter factory).
3. Fee & Total (FeeCalculator & runtime FeeAndLimitConfig).
4. Payment Submission + Hard Case 1 (Idempotency check):
   - Duplicate idempotencyKey: return original result immediately.
   - New idempotencyKey: create Payment (Initiated), FraudCheckHandler chain.
   - Hard Case 4 (Atomic concurrency hold): reject if insufficient or fraud fails; hold funds and transition to FundsHeld.
   - Call biller (SentToBiller).
   - Responses:
     * Confirm within 10s: Completed, DEBIT ledger entry, FEE_INCOME, COMMISSION if agent, async NotificationEventListener, SMS/PUSH receipt.
     * Timeout (>10s): Unknown, funds stay held, RetryCommand scheduled (1, 5, 15 min), inform customer.
     * Refusal: delegate to sequence-refund-failed.puml.
5. Nightly Reconciliation note across.

CONSTRAINTS:
- autonumber, exact 8 states, ledger-based hold/capture, decoupled notifications.
```

---

### 3. Technical Review & Defect Identification

The first generated sequence diagram covered the steps but introduced 3 defects:

1. **Enum Inconsistency with Class Diagram:** Used `BillerPaymentResult(PAID)` and `BillerPaymentResult(FAILED)`. In `class-domain.puml`, the state attribute is of type `PaymentState` (`Completed` and `Failed`), whereas `PAID` is a `SettlementStatus`.
2. **Premature Notification & Duplication in Refusal Branch:** In `else Biller refuses`, the AI called `Payment.fail()` and sent failure notifications *before* the hold was released, and then added a `ref` box covering the same transitions.
3. **Nested Divider Layout Glitches:** Placing `== ... ==` section divider lines inside deeply nested `alt`/`else` frames broke PlantUML's visual frame grouping.

---

### 4. Corrective Prompt

```text
Your sequence diagram is very strong, but fix these 3 consistency and clarity issues and regenerate the complete PlantUML diagram:

1. Fix consistency with our Domain Class Diagram (BillerPaymentResult.state is of type PaymentState):
   - Replace BillerPaymentResult(PAID) with BillerPaymentResult(Completed).
   - Replace BillerPaymentResult(FAILED) with BillerPaymentResult(Failed).

2. Clean up the 'else Biller refuses' branch to avoid duplicating steps from sequence-refund-failed.puml and to avoid notifying before releasing the hold:
   - Keep ONLY:
     * Biller --> BillerAdapter : refused
     * BillerAdapter --> PaymentOrchestrator : BillerPaymentResult(Failed)
     * Immediately followed by 'ref over PaymentOrchestrator, NotificationProvider' stating: "Biller Refusal & Refund Flow — see sequence-refund-failed.puml (SentToBiller -> Failed -> releaseHold -> Refunded -> Notify Customer)".
   - Remove premature Payment.fail() and NotificationEventListener calls from this branch.

3. Remove all '== ... ==` dividers that appear INSIDE alt/else blocks so dividers do not cut across nested alt frames. Use clean comments or brief labels inside branches instead.
```

---

### 5. Final Accepted Output

* **File:** [`docs/design/sequence-pay-bill.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/sequence-pay-bill.puml)
* **Verdict:** Fixed and accepted.
