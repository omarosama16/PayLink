# PayLink — Sequence Diagram: Refund a Failed Payment
## AI Design Prompt, Technical Audit & First-Time Acceptance

* **Date:** 2026-11-24
* **Author:** Mohamed Ahmed (Architecture Lead)
* **Workflow:**
  * **Prompt Optimization & Engineering:** Gemini 3.8 Flash (High)
  * **PlantUML Diagram Generation:** ChatGPT
* **Target Deliverable:** [`docs/design/sequence-refund-failed.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/sequence-refund-failed.puml)
* **Brief References:** Section 2.5 Payment Life Cycle (pp. 4–5), Hard Case 3 (p. 6), Task Distribution A5 (p. 4)

---

### 1. Specification & Context

This diagram models two critical refund scenarios:
1. **Scenario 1 (Direct Biller Refusal & Automatic Hold Release - Hard Case 3):** AquaCo refuses the payment $\to$ transition to `Failed` $\to$ ledger release of hold (`RELEASE_HOLD`) $\to$ transition to final state `Refunded` $\to$ decoupled observer notification with refusal reason.
2. **Scenario 2 (Unknown Payment Guard — NEVER Refunded Automatically):** Explicitly proves that direct `Unknown -> Refunded` is forbidden by the state machine. Funds remain held until resolution via callback, retry, or reconciliation.

---

### 2. Comprehensive Optimized Prompt (Applied Lessons Learned)

Because Entries 1 through 4 revealed specific AI hallucination patterns (enum mismatches, nested frame glitches, invalid state shortcuts), this prompt was engineered with exhaustive domain and visual constraints upfront:

```text
ROLE:
You are a senior software architect who writes clean, accurate, and syntactically valid UML Sequence Diagrams in PlantUML for Java 21 backend systems.

CONTEXT:
I am designing PayLink, a bill-payment and digital wallet backend (Java 21, Spring Boot, PostgreSQL). This sequence diagram (/docs/design/sequence-refund-failed.puml) models "Refund a Failed Payment" and MUST be 100% consistent with:
- Payment State Diagram (/docs/design/state-payment.puml): exact 8 states and exact 9 transitions.
- Domain Class Diagram (/docs/design/class-domain.puml): exact class, interface, and method names.
- Pay a Bill Sequence Diagram (/docs/design/sequence-pay-bill.puml): consistent actor and participant names.

PARTICIPANTS (IN THIS EXACT ORDER)
1. actor Customer
2. participant PaymentOrchestrator
3. participant Payment
4. participant Wallet
5. participant BillerAdapter
6. participant "Biller\n(AquaCo)" as Biller
7. participant NotificationEventListener
8. participant NotificationProvider

SCENARIO TO MODEL (TWO CLEAR SECTIONS)
== Scenario 1: Direct Biller Refusal & Automatic Hold Release (Hard Case 3) ==
Precondition note over PaymentOrchestrator, Wallet: Payment is currently in state SentToBiller with totalAmount (bill + fee) reserved via a HOLD LedgerEntry.

1. Biller Refusal: AquaCo refuses with a clear error code/reason. BillerPaymentResult(Failed, reason).
2. Transition to Failed: Payment.fail() (SentToBiller -> Failed).
3. Release of the Hold (Ledger-Based): Wallet.releaseHold(totalAmount, reference) -> RELEASE_HOLD LedgerEntry created. Record AuditLog.
4. Transition to Refunded: Payment.refund() -> Failed -> Refunded (final state).
5. Customer Notification (Observer Pattern): onPaymentFailed(payment) -> SMS & PUSH. NotificationRetryCommand on delivery failure up to 3 retries.

== Scenario 2: Unknown Payment Guard — NEVER Refunded Automatically ==
Precondition note over PaymentOrchestrator, Wallet: Payment timed out after 10 seconds and is in state Unknown. Funds MUST stay held.

1. Guard check: attempt auto-refund while Unknown (rejected by UnknownStateHandler; Unknown -> Refunded is forbidden).
2. Resolution:
   - Branch 1: Callback / Retry / Reconciliation confirms payment -> Unknown -> Completed, captureHold (DEBIT).
   - Branch 2: Biller refusal / Reconciliation finds no payment -> Unknown -> Failed, releaseHold (RELEASE_HOLD), Failed -> Refunded, notify customer.

CONSTRAINTS:
- Use autonumber, hide footbox, skinparam responseMessageBelowArrow true.
- Use ONLY top-level dividers outside alt/else; NEVER place dividers inside alt/else.
- Use exact state names and transitions. Ledger-based hold release.
```

---

### 3. Technical Verification & Engineering Audit

The generated output was audited against all architecture invariants:

1. **State Machine Conformance:** Perfectly followed `SentToBiller -> Failed -> Refunded` and `Unknown -> Failed -> Refunded`. Verified that `Unknown -> Refunded` is blocked.
2. **Double-Entry Ledger Integrity:** Modeled `Wallet.releaseHold(...)` and `Wallet.captureHold(...)` through explicit ledger entries, with zero direct balance overwriting.
3. **Decoupled Observer:** Notifications dispatched asynchronously (`->>`) without impeding state finalization.
4. **Visual Layout:** Clean participant alignment, no overlapping frames, and concise notes.

---

### 4. Final Verdict

* **Verdict:** Accepted on First Attempt.
* **Engineering Rationale:** Demonstrates the power of rigorous prompt engineering: applying constraints derived from earlier defects produced 100% valid, production-grade output on turn 1.
* **File:** [`docs/design/sequence-refund-failed.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/sequence-refund-failed.puml)
