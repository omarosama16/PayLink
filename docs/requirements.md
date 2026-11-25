# PayLink - Requirements Pack (Phase 1)

Owner: Youssef (Requirements and Product Lead) | Reviewer: Mohamed

## 1. Problem summary
PayLink is an Egyptian company, like Fawry, that lets people pay utility bills (electricity, water, mobile, internet) from a digital wallet, in the app or at agent shops. It earns money from a small fee on each payment and pays agents a commission. Because it holds customers' money, a double charge or a payment the biller never received would destroy trust and could attract regulators. Management wants a new backend that is exact with money, easy to extend with new billers, and able to handle the busiest day of the month. Every step of a payment can fail, so most of the work is deciding what the system does when that happens.

## 2. Stakeholders
| Stakeholder | What they care about |
|---|---|
| Customer | Pays once, is charged once, sees clear status and a receipt |
| Agent | Can pay for customers and gets the 40% commission |
| Biller (ElectroCo, AquaCo, TopUpNet, NetCo) | Gets paid correctly and quickly; matching records |
| Support employee | Finds a payment by reference, sees its history, blocks accounts, reviews differences |
| Admin | Manages billers, fees and limits without redeploying |
| Management / Finance | Fee income, commissions, daily totals, no money lost |
| Regulator / Auditor | Every money movement is recorded (who, when, why) |

## 3. Technology stack (declared)
Java 21, Spring Boot, PostgreSQL, Next.js (simple UI), Docker + docker compose, GitHub Actions. Layered/hexagonal modular monolith.

## 4. User stories
Format: ID | Trace | Priority. Each acceptance criterion is Given / When / Then.

**US-01 | FR-1 | Must** - As a customer I want to register so I can use PayLink.
- Given a new phone number, when I register, then an account and an empty wallet are created with level BASIC.
- Given a phone number already registered, when I register, then the request is rejected with a clear message.

**US-02 | FR-2 | Should** - As an admin I want to upgrade a customer to VERIFIED so limits are raised.
- Given a BASIC customer, when the admin upgrades them, then their daily limit becomes 30,000 EGP.

**US-03 | FR-3 | Must** - As a customer I want to see my available and held balance and history.
- Given a wallet with a 352.50 hold, when I open it, then I see available and held balances and the list of transactions.

**US-04 | FR-4 | Must** - As a customer I want to top up my wallet (simulated card or agent cash).
- Given a valid top-up of 500 EGP, when it succeeds, then a ledger entry of +500 is created and available balance rises by 500.
- Given a failed card simulation, when I top up, then no ledger entry is created.

**US-05 | FR-5 | Must** - As finance I want every balance change recorded as a ledger entry.
- Given any money movement, when it happens, then a ledger entry is written and balances are computed as the sum of entries.
- Given the database, when inspected, then no balance column is ever overwritten directly.

**US-06 | FR-6 | Must** - As an admin I want to add, edit and disable billers and services.
- Given a disabled biller, when a customer lists billers, then it does not appear and cannot be paid.

**US-07 | FR-7 | Must** - As a customer I want to see billers and the input fields each needs.
- Given the catalog, when I open ElectroCo, then I see "meter number"; for TopUpNet I see "phone number".

**US-08 | FR-8 | Must** - As a customer I want a bill inquiry showing amount, fee and total before paying.
- Given meter 123 owes 350.00 and the electricity fee is 2.50, when I inquire, then I see 350.00, 2.50 and total 352.50.

**US-09 | FR-9 | Must** - As a customer I want to pay a bill from my wallet.
- Given enough balance and passing fraud checks, when I pay, then the payment reaches Completed and my wallet is charged the total.

**US-10 | FR-10 | Must | Hard case 1** - As a customer I want a repeated request not to charge me twice.
- Given a payment sent with idempotency key K, when the same request with K arrives again, then no new payment is created and the first result is returned.

**US-11 | FR-11 | Must** - As support I want payments to follow only the allowed life cycle.
- Given a payment in Completed, when any transition is attempted, then it is refused.
- Given a payment in SentToBiller, when the biller confirms, then it moves to Completed.

**US-12 | FR-12 | Must | Hard case 3** - As a customer I want money held before the biller is called and released on failure.
- Given a payment in FundsHeld, when the biller refuses with an error code, then it becomes Failed, the hold is released, state becomes Refunded, and I get an SMS with a clear reason.

**US-13 | FR-13 | Must | Hard case 2** - As a customer I want a slow biller not to lose my money.
- Given ElectroCo answers after 12 seconds, when the 10 second timeout passes, then the payment is Unknown, I am told it is being processed and the money stays held.
- Given an Unknown payment, when 1, 5 and 15 minutes pass, then it is re-queried up to 3 times before waiting for reconciliation.

**US-14 | FR-14 | Must | Hard case 6** - As a biller operator I want duplicate callbacks ignored.
- Given TopUpNet sends "success" twice, when the second arrives, then the payment completes once and the duplicate is ignored and logged.

**US-15 | FR-15 | Must** - As a customer I want to pay ElectroCo, AquaCo and TopUpNet even though their formats differ.
- Given the three billers, when I pay each, then each works through its own adapter.
- Given ElectroCo is down, when I pay AquaCo, then it still succeeds.

**US-16 | FR-16 | Must** - As finance I want fees to follow the business rules.
- Given electricity or water, then fee is 2.50. Given top-up, then fee is 1.5% with minimum 1 EGP. Given internet, then fee is 2 up to 200 and 4 above.
- Given an active promotion for one biller, when a payment is made to it, then the fee is 0; outside the dates the normal fee applies.

**US-17 | FR-17 | Should** - As an agent I want to pay for a customer and earn commission.
- Given an agent payment with fee 2.50, when it completes, then 1.00 EGP (40%) is credited to the agent's wallet.
- Given the payment fails, then no commission is credited.

**US-18 | FR-18 | Must** - As a customer I want clear reasons when a limit blocks me.
- Given an amount below 5 or above 10,000, when I pay, then it is rejected with the reason.
- Given a BASIC account that already paid 4,900 today, when I pay 200, then it is Rejected (daily limit 5,000).

**US-19 | FR-19 | Should** - As the business I want to stop rapid-fire payments.
- Given more than 5 payments within any 60‑second window, the 6th payment is rejected and the account is blocked for 10 minutes.

**US-20 | FR-20 | Must** - As support I want to search a payment by reference and see its history.
- Given a reference, when I search, then I see the payment and every state transition with time.

**US-21 | FR-21 | Should** - As support I want to block and unblock an account.
- Given a blocked account, when it tries to pay, then it is Rejected; after unblock it can pay.

**US-22 | FR-22 | Must** - As a customer I want an SMS and push message on completion and failure.
- Given a payment reaches Completed or Failed, then I receive an SMS and a push message.

**US-23 | FR-23 | Should | Hard case 5** - As the business I want notification failures not to break payments.
- Given the SMS provider is down, when a payment completes, then it stays Completed, the SMS is retried 3 times and then marked undelivered.

**US-24 | FR-24 | Should** - As an admin I want to change fees and limits without redeploying.
- Given the admin changes the daily limit, when the next payment is checked, then the new value is used with no restart.

**US-25 | FR-25 | Must** - As an auditor I want every money movement and admin action logged.
- Given any ledger entry or admin change, then an audit record exists with who, when and why.

**US-26 | FR-26 | Should | Hard case 7** - As support I want a nightly reconciliation report.
- Given a settlement file, when it is processed, then payments are compared with PayLink records.
- Given the file lists a payment PayLink has as Failed, then the difference is flagged in the report for support review.

**US-27 | FR-27 | Could** - As a customer I want to request a refund of a completed payment.
- Phase 2 (Could): Given a Completed payment, when I request a refund, then it waits for support; on approval the money returns to my wallet as a ledger entry.

**US-28 | FR-28 | Could** - As an admin I want daily totals.
- Given a day of payments, when I open the dashboard, then I see payments, fees earned, commissions paid and failures per biller.

**US-29 | Hard case 4 | Must** - As a customer I want concurrent payments never to overdraw my wallet.
- Given a balance covering one payment, when two payments arrive at the same moment, then one succeeds, the other is Rejected and the balance is never negative.

**US-30 | Hard case 8 | Must** - As a developer I want to add a biller without touching other modules.
- Given a new biller, when it is added, then only a new adapter and configuration are added and existing modules are unchanged.

**US-31 | FR-13 / Sec 2.5 | Must** - As finance I want Unknown payments never refunded automatically.
- Given a payment in Unknown, when retries are exhausted, then no refund happens; money is returned only after the biller confirms failure or reconciliation finds no payment (state Failed, then Refunded).

Coverage check: FR-1 to FR-28 are all covered (US-01 to US-28); hard cases 1 to 8 appear in US-10, US-13, US-12, US-29, US-23, US-14, US-26, US-30.

## 5. Non-functional requirements
| ID | Quality | Target | How measured |
|---|---|---|---|
| NFR-1 | Correctness | 0 double charges in duplicate and concurrency tests | Automated tests for hard cases 1 and 4 |
| NFR-2 | Correctness | 0 negative balances | Concurrency test + DB check after load test |
| NFR-3 | Performance | 100 payments/second sustained for 1 minute against mock billers | Load test script, report in Phase 2 |
| NFR-4 | Resilience | One biller down: 100% of payments to other billers still succeed | Integration test with failing mock biller |
| NFR-5 | Modifiability | Adding a biller changes 0 files outside Biller Integration + config | Git diff of the change request |
| NFR-6 | Auditability | 100% of money movements have an audit record (who, when, why) | Test: count ledger entries = audit records |
| NFR-7 | Testability | At least 70% line coverage on domain logic | Coverage report in CI |
| NFR-8 | Responsiveness | Bill inquiry under 2 s at p95 for ElectroCo | Timing in integration test |

## 6. Priorities
Must: FR-1,3,4,5,6,7,8,9,10,11,12,13,14,15,16,18,20,22,25. Should: FR-2,17,19,21,23,24,26. Could: FR-27,28. Source: FR table in the brief.

## 7. Ambiguities and decisions
| # | Question / ambiguity | Decision |
|---|---|---|
| 1 | Does the daily limit count bill amount or total with fee? | Bill amount only (fee excluded) |
| 2 | Calendar day or rolling 24 hours? | Calendar day, Africa/Cairo time zone |
| 3 | What happens to the fee on a failed payment? | Fee is released with the hold; customer pays no fee |
| 4 | Do held (Unknown) payments count toward the daily limit? | Yes, until Failed/Refunded |
| 5 | Are min/max 5 to 10,000 on bill amount or total? | Bill amount |
| 6 | Does velocity count rejected attempts? | Yes, all payment attempts count |
| 7 | Does the 10 minute block end automatically? | Yes; support can also unblock manually |
| 8 | Is "up to 200" for tiered internet fee inclusive? | Yes: 200.00 pays 2 EGP, above pays 4 |
| 9 | Rounding of 1.5% fee? | Exact decimal, 2 places, HALF_UP, minimum 1 EGP |
| 10 | Commission during a 0-fee promotion? | 40% of 0 = 0 |
| 11 | On completed-payment refund, are fee and commission reversed? | Full refund including fee; agent commission reversed |
| 12 | Is NetCo in scope? | Optional / Deferred to Phase 2 |
| 13 | Available balance definition? | Sum of ledger entries minus held amount |
| 14 | Who pays when an agent pays for a customer? | Customer wallet is charged; agent gets commission |
