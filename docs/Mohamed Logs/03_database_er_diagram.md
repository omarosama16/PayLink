# PayLink — Database Entity-Relationship (ER) Diagram
## AI Design Prompt, Technical Review & Schema Normalization

* **Date:** 2026-11-24
* **Author:** Mohamed Ahmed (Architecture Lead)
* **Workflow:**
  * **Prompt Optimization & Engineering:** Gemini 3.8 Flash (High)
  * **PlantUML Diagram Generation:** ChatGPT
* **Target Deliverable:** [`docs/design/er.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/er.puml)
* **Brief References:** Section 3.4 Database (p. 10), Task Distribution A3 (p. 4), FR-5 (p. 8)

---

### 1. Specification & Context

The PayLink database schema requires at least 10 normalized PostgreSQL tables with schema migrations.

* **Strict Balance Rule:** `wallets` must **never** store `balance`, `available_balance`, or `held_balance`. Balances are strictly aggregated from immutable `ledger_entries`.
* **Precision Rule:** All financial amounts and limits use exact `NUMERIC(19,2)`. Never `FLOAT` or `DOUBLE`.
* **Idempotency Rule:** `payments.idempotency_key` and `payments.reference` must be `UNIQUE` and `NOT NULL`.

---

### 2. Initial Optimized Prompt

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

---

### 3. Technical Review & Defect Identification

The initial diagram generated had 15 tables and good data types, but suffered from 4 defects:

1. **Cardinality Inconsistency:** Modeled `customers ||--o| wallets` (optional wallet), conflicting with `class-domain.puml` where every customer mandatorily owns exactly one wallet (`Customer "1" *-- "1" Wallet`).
2. **Missing Entity for Reconciliation Parity:** Omitted the `settlement_records` table, making record-level reconciliation persistence unclear.
3. **Denormalization Redundancy:** `settlement_files` contained both `biller_id` and redundant `biller_code`.
4. **Visual Clutter (9 Verbose Notes):** Rendered 9 large note blocks across the canvas, making the exported diagram illegibly wide for the PDF report.

---

### 4. Corrective Prompt

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

---

### 5. Final Accepted Output

* **File:** [`docs/design/er.puml`](file:///d:/Projects/fullStack%20in%20progress/PayLink/docs/design/er.puml)
* **Table Count:** 16 normalized tables (1-to-1 parity with Domain Model).
* **Verdict:** Fixed and accepted.
