# PayLink AI Interaction & Prompt Log

This document tracks all significant AI interactions throughout the project lifecycle in accordance with **Section 5 (Using AI)** of the PayLink Student Brief.

Each entry records the prompt, the AI-generated answer summary, and an honest technical verdict (**accepted**, **fixed**, or **rejected**) with engineering rationale.

---

## Prompt Log Table

| Date | Author | Stage | Tool and model | Prompt | Summary of answer | Verdict (accepted, fixed, rejected) and why |
|---|---|---|---|---|---|---|
| 2026-11-23 | Mohamed Ahmed | Design (State Modeling) | Gemini 3.8 Flash (High) | **Role:** Senior Software Engineer (UML PlantUML expert).<br>**Context:** PayLink payment lifecycle (8 states, 9 transitions).<br>**Constraints:** Strict 8 states, 9 allowed transitions, final states to `[*]`, no direct `Unknown -> Refunded`, explicit event labels.<br>**Requested Style:** `@startuml ... @enduml`. | Generated a PlantUML state diagram. However, it rendered `Start` and `End` as rectangular states instead of `[*]` pseudostates, created duplicate boxes for `Failed` and `Completed`, routed `Rejected -> Failed`, omitted the `Refunded` state entirely (pointing `Failed -> Completed`), and truncated the event conditions on `Unknown`. | **Fixed:** The first output was rejected due to structural UML flaws and specification violations. A corrective prompt was provided enforcing exact transitions, pseudostates, and restoring `Failed -> Refunded` and terminal markers `[*]`. The resulting diagram was verified and accepted into `docs/design/state-payment.puml`. |
| 2026-11-24 | Mohamed Ahmed | Design (Domain Modeling) | Gemini 3.8 Flash (High) | **Role:** Principal software architect (Java 21 Clean Architecture).<br>**Context:** PayLink Layered Modular Monolith Domain Class Diagram.<br>**Constraints:** 8 modules, 7 design patterns, strict `BigDecimal`, no stored wallet balances.<br>**Requested Style:** PlantUML class diagram. | Generated an initial diagram with 6 structural defects (duplicate config class, hardcoded numbers, weak state pattern, missing linkages). When prompted to add `PaymentStateHandler`, the AI hallucinated invalid transitions (`FundsHeld -> Failed` and `Completed -> Refunded`) and missed `Failed -> Refunded`. | **Fixed:** First output was rejected for 6 architecture flaws. The second output was rejected for invalid state transitions violating Section 2.5 (`Completed` cannot transition to `Refunded`). The student intervened with a second correction enforcing the exact 9 canonical transitions and terminal rules. Accepted into `/docs/design/class-domain.puml`. |
| 2026-11-24 | Mohamed Ahmed | Design (Database Modeling / ER) | Gemini 3.8 Flash (High) | **Role:** Senior database architect (normalized PostgreSQL, PlantUML).<br>**Context:** PayLink PostgreSQL ER Diagram (14+ tables).<br>**Constraints:** 1-to-1 parity with Class/State diagrams, strict `NUMERIC(19,2)`, no wallet balance columns, UNIQUE keys.<br>**Requested Style:** PlantUML Crow's Foot ER diagram. | Generated a 15-table diagram, but it contained subtle consistency and layout defects: wallet cardinality was optional (`||--o|`) instead of mandatory 1-to-1, `settlement_records` entity was omitted, redundant `biller_code` denormalized in `settlement_files`, and excessive notes (9 boxes) created severe visual clutter for the report. | **Fixed:** First output was critiqued and corrected across 4 points: mandatory wallet ownership (`||--||`), added `settlement_records` table, removed redundant column, and streamlined notes to 2 essential domain invariant blocks. Output accepted into `/docs/design/er.puml`. |
| 2026-11-24 | Mohamed Ahmed | Design (Sequence Modeling - Pay a Bill) | Gemini 3.8 Flash (High) | **Role:** Senior software architect (UML Sequence Diagrams, Java 21, Spring Boot, PlantUML).<br>**Context:** PayLink 9-step payment orchestration and hard cases (idempotency, timeout, concurrency).<br>**Constraints:** Strict 8 states, 9 transitions, ledger-based hold/capture, decoupled notifications.<br>**Requested Style:** PlantUML sequence diagram. | Generated a sequence diagram covering the 9 steps, but it had 3 defects: used `BillerPaymentResult(PAID/FAILED)` instead of canonical `PaymentState` enum values (`Completed`/`Failed`), prematurely triggered failure notification and state transition in the refusal branch before releasing the hold, and placed nested `== ... ==` dividers inside `alt` frames which caused visual rendering glitches. | **Fixed:** First output was rejected for enum inconsistency, premature notification dispatch, and nested frame layout errors. A corrective prompt aligned status enums to `PaymentState`, cleanly delegated refusal & refund handling to `sequence-refund-failed.puml` via a `ref` block, and cleaned top-level dividers. Output accepted into `/docs/design/sequence-pay-bill.puml`. |
| 2026-11-24 | Mohamed Ahmed | Design (Sequence Modeling - Refund Flow) | Gemini 3.8 Flash (High) | **Role:** Senior software architect (UML Sequence Diagrams, Java 21, Spring Boot, PlantUML).<br>**Context:** PayLink Refund a Failed Payment flow and Unknown state guard.<br>**Constraints:** 2 distinct scenarios (Hard Case 3 refusal & Unknown guard), ledger-based release (`RELEASE_HOLD`), decoupled observer notifications.<br>**Requested Style:** PlantUML sequence diagram. | Generated a 2-scenario sequence diagram modeling direct refusal with ledger release (`SentToBiller -> Failed -> Refunded`) and the Unknown payment guard enforcing that Unknown is never auto-refunded. Because the prompt applied all structural and syntax lessons learned in Entries 1–4, the generated diagram complied 100% with the domain specification, state machine, and clean layout rules on the first attempt. | **Accepted:** Output verified and accepted without modification. The prompt's strict upfront constraints (preconditions, exact participant ordering, explicit guard checks, and prohibition of nested divider lines) produced a correct diagram with zero domain, state, or visual defects. Saved to `/docs/design/sequence-refund-failed.puml`. |

---

## Detailed Log Entries

### Entry 1: Payment Life Cycle State Diagram (`state-payment.puml`)

* **Author:** Mohamed Ahmed (Architecture Lead)
* **Date:** 2026-11-23
* **Target Deliverable:** `/docs/design/state-payment.puml`
* **Brief References:** Section 2.5 (pp. 4–5), Section 4 Phase 1 Item 2 (p. 11), FR-11 (p. 8)

#### 1. Initial Prompt
```text
ROLE:
You are a senior software engineer who writes precise UML diagrams in PlantUML.

CONTEXT:
I am designing PayLink, a bill-payment and wallet backend. Customers pay electricity, water, mobile, and internet bills from a wallet. Every payment is in exactly one state at any time. The allowed states and transitions are fixed by the specification below.

STATES:
Initiated • FundsHeld • SentToBiller • Completed • Failed • Unknown • Rejected • Refunded

ALLOWED TRANSITIONS:
1. Initiated -> FundsHeld: Fraud checks pass and the wallet has enough funds.
2. Initiated -> Rejected: Fraud check fails or funds are insufficient.
3. FundsHeld -> SentToBiller: Request sent to biller.
4. SentToBiller -> Completed: Biller confirms.
5. SentToBiller -> Failed: Biller refuses.
6. SentToBiller -> Unknown: No answer within the timeout (10 seconds).
7. Unknown -> Completed: Biller later confirms (callback, retry, or settlement file).
8. Unknown -> Failed: Biller later refuses, or reconciliation finds no payment.
9. Failed -> Refunded: Hold released to the wallet.

CONSTRAINTS:
- Use exactly the eight states and nine transitions above. Do not add, remove, or rename any state or transition.
- No direct transition from Completed to anything.
- No direct transition from Unknown to Refunded.
- Label every transition with its event text.
- Mark the three final states with a transition to [*].

FINAL-STATE RULE:
Completed, Rejected, and Refunded are final. Unknown must not be refunded automatically.

OUTPUT FORMAT:
One PlantUML state diagram inside a single @startuml ... @enduml block, with no explanation before or after it.
```

#### 2. Initial AI Defects Identified (Review & Critique)
1. **Missing State & Incorrect Refund Destination:** The diagram routed `Failed` to `Completed` with the label *"hold released to wallet"*. The `Refunded` state was omitted entirely.
2. **Invalid Outgoing Transition:** Diagram added `Rejected -> Failed`. `Rejected` is an immutable final state and cannot transition further.
3. **Duplicate State Boxes:** `Failed` and `Completed` each appeared twice instead of transitions converging to single state entities.
4. **Incorrect UML Notation:** Used rectangular state boxes for `Start` and `End` instead of `[*]` initial and final pseudostates.
5. **Only Completed Connected to Terminal Marker:** Only `Completed` connected to `End`. `Rejected` and `Refunded` did not connect to terminal markers.
6. **Incomplete Event Labels:** Transition labels on `Unknown` omitted the required details (*callback, retry or settlement file* / *or reconciliation finds no payment*).

#### 3. Corrective Prompt
```text
Fix and apply the following PlantUML state diagram:
- Restore the missing 'Refunded' state.
- Remove the invalid transition 'Rejected -> Failed'.
- Remove duplicate state boxes for 'Failed' and 'Completed'.
- Use standard UML pseudostates [*] for initial and final states (Completed, Rejected, Refunded must each connect to [*]).
- Ensure all nine transitions and their exact event labels match the specification.
```

#### 4. Final Accepted Output
Saved directly to [`/docs/design/state-payment.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/state-payment.puml).

---

### Entry 2: Domain Class Diagram (`class-domain.puml`)

* **Author:** Mohamed Ahmed (Architecture Lead)
* **Date:** 2026-11-24
* **Target Deliverable:** `/docs/design/class-domain.puml`
* **Brief References:** Section 3.1 Modules (p. 7), Section 3.4 Technical & Patterns (p. 10), Task Distribution A2 (p. 4)

#### 1. Initial Prompt
```text
ROLE:
You are a principal software architect who writes clean, accurate, and syntactically valid UML Domain Class Diagrams in PlantUML for Java 21 backend systems.

CONTEXT:
I am designing PayLink, a bill-payment and digital wallet backend in Java 21 (Spring Boot, PostgreSQL). We are building a Layered Modular Monolith following the Clean Architecture Dependency Rule: domain entities and business rules must NOT depend on web frameworks, controllers, JPA annotations, or database implementations.

This diagram is the Domain Class Diagram (/docs/design/class-domain.puml) and must be consistent with our Payment State Diagram and our PostgreSQL ER Diagram.

MODULES TO INCLUDE (GROUP EACH IN A PLANTUML PACKAGE):
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
- Layered Modular Monolith: 8 packages, clean domain classes only.
- Strict BigDecimal for monetary values.
- Wallet balances strictly computed from LedgerEntry records.
- Complete UML relationships with multiplicities.
```

#### 2. Initial AI Defects Identified (Review & Critique)
1. **Duplicate Class Declarations:** The AI declared `class FeeAndLimitConfig` twice—once in `Fees and Commissions` and once in `Fraud Checks`, violating UML and namespace scoping.
2. **Weak State Pattern Representation:** The AI only used `enum PaymentState <<State>>` without an interface or concrete state behavior classes to govern transitions.
3. **Hardcoded Business Rules in Code:** Strategy and check classes hardcoded defaults (`= 2.50`, `= 0.015`, `= 1.00`, `= 200.00`, `= 0.40`, `= 5`), violating Section 2.6 which demands these values live in runtime configuration (`FeeAndLimitConfig`).
4. **Missing Payment Module Linkages:** `Payment` lacked associations to `Agent` (optional processing) and `Notification` (traceability back to payment).
5. **Missing Domain Operations on Wallet:** `Wallet` only had query methods `getAvailableBalance()` and `getHeldBalance()` but lacked state-changing domain methods (`topUp`, `hold`, `releaseHold`, `captureHold`).
6. **Misconfigured AuditLog Actor Scope:** Modeled `Customer "1" --> "0..*" AuditLog`, omitting agents, support staff, and admin actors.

#### 3. First Corrective Prompt (Architecture Refactoring)
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

#### 4. Second AI Defect Identified (Invalid State Transitions Hallucination)
Upon reviewing the AI's regenerated diagram, a critical domain logic error was discovered in the state transition rules rendered for `PaymentStateHandler`:
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
    Completed -> Refunded : refund      <-- CRITICAL DEFECT: Completed is immutable/final; refund only occurs from Failed!
end note
```
1. **Severe Lifecycle Violation (`Completed -> Refunded`):** The AI allowed a direct transition from `Completed` to `Refunded`. In PayLink's domain specification (Section 2.5), `Completed`, `Rejected`, and `Refunded` are terminal states. Refunds apply to held funds when a payment fails (`Failed -> Refunded`). A completed payment is never refunded via state transition.
2. **Invalid Shortcut (`FundsHeld -> Failed`):** Bypassed the biller dispatch stage.
3. **Missing Allowed Transitions:** The AI omitted `Unknown -> Failed` (resolution via settlement discrepancy or query failure) and `Failed -> Refunded`.

#### 5. Second Corrective Prompt (Student Intervention)
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

#### 6. Final Accepted Output
Saved directly to [`/docs/design/class-domain.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/class-domain.puml).

---

### Entry 3: Database Entity-Relationship Diagram (`er.puml`)

* **Author:** Mohamed Ahmed (Architecture Lead)
* **Date:** 2026-11-24
* **Target Deliverable:** `/docs/design/er.puml`
* **Brief References:** Section 3.4 Database (p. 10), Task Distribution A3 (p. 4), FR-5 (p. 8)

#### 1. Initial Prompt
```text
ROLE:
You are a senior database architect who designs normalized PostgreSQL schemas and writes clean, syntactically valid Entity-Relationship (ER) diagrams in PlantUML.

CONTEXT:
I am designing PayLink, a bill-payment and digital wallet backend built with Java 21, Spring Boot, and PostgreSQL. This ER diagram (/docs/design/er.puml) must align 1-to-1 with our Domain Class Diagram (/docs/design/class-domain.puml) and our Payment State Diagram (/docs/design/state-payment.puml).

REQUIRED TABLES (MINIMUM 10 — INCLUDE THESE 14 TABLES FOR FULL DOMAIN CONSISTENCY):
1. customers (id, name, phone UNIQUE, verification_level, blocked, blocked_until, created_at)
2. agents (customer_id PK/FK, terminal_id UNIQUE, active, created_at)
3. wallets (id, customer_id FK UNIQUE, currency. CRITICAL: NO stored balance columns)
4. ledger_entries (id, wallet_id FK, payment_id FK nullable, type, amount, reference, created_at)
5. billers (id, code UNIQUE, name, protocol, active, created_at)
6. services (id, biller_id FK, name, category, required_fields JSONB, min_amount, max_amount, active)
7. payments (id, customer_id, wallet_id, service_id, agent_id nullable, reference UNIQUE, idempotency_key UNIQUE, amount, fee, total_amount, current_state, retry_count, timestamps)
8. payment_transitions (id, payment_id FK, from_state, to_state, event, occurred_at)
9. fee_and_limit_config (id, basic/verified daily limits, min/max single amount, commission_rate, velocity rules, promotion config, updated_at)
10. fee_rules (id, config_id FK, strategy_type, biller_code nullable, service_category nullable, parameters JSONB, enabled)
11. notifications (id, customer_id FK, payment_id FK, channel, recipient, template, status, retry_count, created_at)
12. audit_logs (id, actor_id, actor_role, action, target_entity, target_id nullable, reason, created_at)
13. settlement_files (id, biller_id FK, settlement_date, uploaded_at)
14. reconciliation_reports & reconciliation_discrepancies

CONSTRAINTS:
- PlantUML Crow's Foot notation with PostgreSQL types (UUID, NUMERIC(19,2), TIMESTAMPTZ, JSONB).
- NO balance columns in wallets table; balances strictly computed from ledger_entries.
- Exact 8 payment states and 9 transitions in domain notes.
```

#### 2. Initial AI Defects Identified (Review & Critique)
1. **Cardinality Mismatch with Class Diagram:** The AI modeled `customers ||--o| wallets` (optional wallet for a customer), whereas the domain class diagram specifies strong composition `Customer "1" *-- "1" Wallet` (every customer owns exactly one wallet).
2. **Missing Entity for Reconciliation Parity:** In `class-domain.puml`, `SettlementRecord` is defined as a core value object inside `SettlementFile`. The initial ER diagram omitted a `settlement_records` table, making settlement record persistence ambiguous.
3. **Denormalization Redundancy:** `settlement_files` contained both `biller_id` (FK to `billers`) and redundant `biller_code`.
4. **Severe Visual Clutter (9 Large Note Blocks):** The AI generated 9 sprawling note boxes alongside 15 tables, rendering the diagram illegible and impractically wide when exported to the final PDF report.

#### 3. Corrective Prompt
```text
Your PostgreSQL ER diagram is very close, but it has 4 minor consistency and readability issues. Fix ALL of them and regenerate the complete PlantUML ER diagram:

1. Fix cardinality to match our Domain Class Diagram (Customer "1" *-- "1" Wallet):
   - Change 'customers ||--o| wallets' to 'customers ||--|| wallets : "owns wallet"'.

2. Ensure 1-to-1 entity consistency with the Reconciliation package in our Domain Class Diagram:
   - Add the 'settlement_records' table (id, settlement_file_id FK, biller_reference, payment_reference, amount, settlement_status).
   - Add the relationship 'settlement_files ||--o{ settlement_records : "contains records"'.
   - Remove redundant 'biller_code' from settlement_files since biller_id links to billers.

3. Reduce visual clutter for PDF readability:
   - Keep ONLY two concise notes: note right of wallets (no stored balance; computed from ledger_entries) and note right of payments (8 states, 9 transitions, final states, UNIQUE keys).
   - Remove all other verbose note blocks.

4. Preserve all existing requirements:
   - 16 tables, PostgreSQL types, NUMERIC(19,2), NO balance column in wallets, UNIQUE idempotency_key/reference.
```

#### 4. Final Accepted Output
Saved directly to [`/docs/design/er.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/er.puml).

---

### Entry 4: Sequence Diagram: Pay a Bill (`sequence-pay-bill.puml`)

* **Author:** Mohamed Ahmed (Architecture Lead)
* **Date:** 2026-11-24
* **Target Deliverable:** `/docs/design/sequence-pay-bill.puml`
* **Brief References:** Section 2.2 Payment Story (p. 2), Hard Cases 1, 2, 4, 5 (p. 6), Task Distribution A4 (p. 4)

#### 1. Initial Prompt
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

#### 2. Initial AI Defects Identified (Review & Critique)
1. **Enum Mismatch with Domain Class Diagram:** The AI generated `BillerPaymentResult(PAID)` and `BillerPaymentResult(FAILED)`. In `class-domain.puml`, `BillerPaymentResult.state` is typed as `PaymentState` (canonical values `Completed` and `Failed`). `PAID`/`FAILED` belong strictly to `SettlementStatus` in the reconciliation package.
2. **Premature Notification & Duplication in Refusal Branch:** In `else Biller refuses`, the AI called `Payment.fail()` and dispatched `onPaymentFailed` to `NotificationEventListener` *before* releasing the hold, and then appended a `ref` box showing `SentToBiller -> Failed -> Refunded`. This duplicated the transition and violated the correct sequence where customer notification happens after the wallet hold has been safely released to `Refunded`.
3. **Nested Divider Layout Glitches:** Placing section dividers (`== Step 5 ==`, `== Step 6 ==`, `== Observer ==`) inside deeply nested `alt`/`else` frames broke PlantUML's visual grouping boxes, causing unsightly horizontal rule overlaps.

#### 3. Corrective Prompt
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

#### 4. Final Accepted Output
Saved directly to [`/docs/design/sequence-pay-bill.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/sequence-pay-bill.puml).

---

### Entry 5: Sequence Diagram: Refund a Failed Payment (`sequence-refund-failed.puml`)

* **Author:** Mohamed Ahmed (Architecture Lead)
* **Date:** 2026-11-24
* **Target Deliverable:** `/docs/design/sequence-refund-failed.puml`
* **Brief References:** Section 2.5 Payment Life Cycle (pp. 4–5), Hard Case 3 (p. 6), Task Distribution A5 (p. 4)

#### 1. Initial Prompt
```text
ROLE:
You are a senior software architect who writes clean, accurate, and syntactically valid UML Sequence Diagrams in PlantUML for Java 21 backend systems.

CONTEXT:
I am designing PayLink, a bill-payment and digital wallet backend (Java 21, Spring Boot, PostgreSQL). This sequence diagram (/docs/design/sequence-refund-failed.puml) models "Refund a Failed Payment" and MUST be 100% consistent with:
- Payment State Diagram (/docs/design/state-payment.puml): exact 8 states and exact 9 transitions.
- Domain Class Diagram (/docs/design/class-domain.puml): exact class, interface, and method names.
- Pay a Bill Sequence Diagram (/docs/design/sequence-pay-bill.puml): consistent actor and participant names.

PARTICIPANTS (IN THIS EXACT ORDER):
Customer, PaymentOrchestrator, Payment, Wallet, BillerAdapter, Biller (AquaCo), NotificationEventListener, NotificationProvider.

SCENARIO TO MODEL (TWO CLEAR SECTIONS):
== Scenario 1: Direct Biller Refusal & Automatic Hold Release (Hard Case 3) ==
Precondition: Payment in SentToBiller, totalAmount held.
1. Biller Refusal: AquaCo refuses (errorCode, reason). BillerPaymentResult(Failed, reason).
2. Transition to Failed: Payment.fail() (SentToBiller -> Failed).
3. Release Hold: Wallet.releaseHold(totalAmount, ref) -> RELEASE_HOLD LedgerEntry. AuditLog recorded.
4. Transition to Refunded: Payment.refund() -> Failed -> Refunded (final state).
5. Notification: Observer pattern onPaymentFailed(payment) -> SMS & PUSH. RetryCommand on failure up to 3 times.

== Scenario 2: Unknown Payment Guard — NEVER Refunded Automatically ==
Precondition: Timed out after 10s, state is Unknown, funds stay held.
1. Guard check: attempt auto-refund while Unknown is rejected by UnknownStateHandler (Unknown -> Refunded is forbidden).
2. Resolution:
   - Branch 1: Callback / Retry / Settlement confirms -> Unknown -> Completed, captureHold (DEBIT).
   - Branch 2: Biller refusal / settlement no payment -> Unknown -> Failed, releaseHold (RELEASE_HOLD), Failed -> Refunded, notify customer.

CONSTRAINTS:
- autonumber, hide footbox, responseMessageBelowArrow true.
- ONLY top-level dividers outside alt/else; NO dividers inside alt/else.
- Exact state names and transitions. Ledger-based hold release.
```

#### 2. Review and Verification of AI Output
The AI output was evaluated against all architectural, domain, and visual criteria:
1. **Lifecycle & State Machine Integrity:** Exactly followed the canonical 9 transitions (`SentToBiller -> Failed`, `Failed -> Refunded`, `Unknown -> Completed`, `Unknown -> Failed`). It explicitly verified that direct `Unknown -> Refunded` is forbidden.
2. **Double-Entry Ledger Invariant:** Modeled `Wallet.releaseHold(...)` creating a `RELEASE_HOLD` ledger entry and `Wallet.captureHold(...)` creating a `DEBIT` ledger entry, with no direct balance field overwriting.
3. **Decoupled Notification Observer:** `PaymentEventListener` dispatched asynchronously (`->>`); failure of `NotificationProvider` does not alter the `Refunded` or `Completed` state.
4. **Layout and Rendering:** No inner `== ... ==` dividers inside `alt` frames; clean participant spacing and notes that render without horizontal clipping in the PDF export.

#### 3. Verdict: Accepted
* **Verdict:** Accepted without modification.
* **Engineering Rationale:** Applying the lessons learned from the critiques in Entries 1 through 4 (supplying exact method signatures, participant sequences, explicit state guard invariants, and forbidding nested divider lines) eliminated AI hallucinations. The prompt yielded a 100% compliant, production-grade PlantUML sequence diagram on the first attempt.
* Saved directly to [`/docs/design/sequence-refund-failed.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/sequence-refund-failed.puml).
