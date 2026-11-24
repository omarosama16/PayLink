PayLink Year Work Project - Student Brief 

# PayLink Year Work Project - Student Brief 

Your team builds PayLink, a bill-payment and wallet backend, from requirements to a locally deployed CI/CD pipeline, using AI at every step and applying what you learn in lectures and labs. 

## 1. Project at a glance 

|What you build|PayLink: a backend system where customers pay<br>utility bills (electricity, water, mobile, internet)<br>from a digital wallet|
|---|---|
|Grading|100 points, rescaled to the year-work marks|
|Tools you must use|AI assistant (prompting), PlantUML or Mermaid,<br>Docker and docker compose, GitHub Actions, a<br>unit-testing framework|
|Concepts you must apply|Prompt engineering, AI-augmented design, test-<br>driven development, 5 design patterns, an<br>architecture pattern, deployment options|
|Phase 1 due|Wednesday 25 November (30 points)|
|Phase 2 (final) due|Friday 18 December (70 points)|
|Discussion|In your lab, starting Saturday 19 December|



Read sections 2 and 3 first: they say what to build. Section 4 says what to hand in and when. 

## 2. The problem in detail 

### 2.1 The story 

PayLink is a company in Egypt, similar to Fawry or Vodafone Cash. Millions of people pay bills every month through its mobile app and through agents (small shops with a PayLink terminal). Customers do not want to queue at the electricity office, and billers (the utility companies) want to be paid fast and correctly. 

Page 1 of 16 

PayLink Year Work Project - Student Brief 

PayLink earns money by charging a small **service fee** on each payment and by paying **commission** to agents. It also carries a heavy responsibility: it holds customers' money in **wallets** . If a customer is charged twice, or charged for a payment the biller never received, the company loses trust and money, and may face regulators. 

You are the engineering team. Management wants a new backend that is correct with money, easy to extend with new billers, and ready to handle the busiest day of the month. 

### 2.2 One payment, step by step 

Sara opens the app and pays her electricity bill from her wallet: 

1. **Login and limits.** The system identifies Sara and checks her verification level (basic accounts may pay at most 5,000 EGP per day). 

2. **Bill inquiry.** Sara enters her meter number. PayLink asks the electricity company (ElectroCo) for the bill and shows 350.00 EGP due. 

3. **Fee and total.** PayLink adds its service fee (2.50 EGP) and shows a total of 352.50 EGP. 

4. **Fraud checks.** The system checks daily limit, number of payments in the last minute, and whether the account is blocked. 

5. **Hold the money.** 352.50 EGP is moved from Sara's available balance to a held balance. It is not spent yet. 

6. **Call the biller.** PayLink sends the payment to ElectroCo with a unique reference. 

7. **Confirmation.** ElectroCo answers: paid. PayLink turns the hold into a final charge, credits its own fee income, and records the commission if an agent was involved. 

8. **Receipt.** Sara gets an SMS and a push notification. 

9. **Night reconciliation.** ElectroCo sends a settlement file listing every payment it received. PayLink compares it with its own records and reports any difference. 

Every step can fail. Most of your project is about what the system does when it does. 

### 2.3 Key terms you must understand 

|Term|Meaning in this project|
|---|---|
|Biller|A company that issues bills and receives payments<br>(electricity, water, mobile operator, internet<br>provider)|



Page 2 of 16 

PayLink Year Work Project - Student Brief 

|Term|Meaning in this project|
|---|---|
|Bill inquiry|Asking the biller what a customer owes before<br>paying|
|Wallet|A customer account holding money in EGP, with an<br>available balance and a held balance|
|Ledger|A list of entries that records every money<br>movement. A balance is the sum of its ledger<br>entries; you never overwrite a balance directly|
|Hold|Money reserved for a payment that is not yet<br>confirmed|
|Service fee|What PayLink charges the customer per payment|
|Commission|What PayLink pays an agent for a payment made at<br>the agent's terminal|
|Idempotency key|A unique value sent with a request so that sending<br>it twice has the effect of sending it once|
|Settlement file|The biller's daily list of payments it received|
|Reconciliation|Comparing PayLink's payments with the<br>settlement file and flagging differences|
|Refund|Returning held or charged money to the wallet<br>after a failed or cancelled payment|



### 2.4 The billers you integrate with 

Each biller is a mock server. The instructors give you the contract below; you may write the mock servers yourselves (a small program that follows the contract and can be told to fail on purpose). 

|Biller|Service|Protocol and format|Behaviour you must handle|
|---|---|---|---|
|ElectroCo|Electricity<br>bills|REST, JSON|Fast (under 1 second). Inquiry by<br>meter number. Payment confirmed<br>immediately|



Page 3 of 16 

PayLink Year Work Project - Student Brief 

|Biller|Service|Protocol and format|Behaviour y|ou must handle|
|---|---|---|---|---|
|AquaCo|Water bills|SOAP-style XML over<br>HTTP|Slower (2 to<br>field names<br>payments a|4 seconds). Different<br>and error codes. Rejects<br>bove the bill amount|
|TopUpNet|Mobile top-<br>up|REST, JSON,<br>asynchronous|Replies "acc<br>your callbac<br>"success" o|epted" first, then calls<br>k URL minutes later with<br>r "failed"|
|NetCo|Internet bills|Daily batch CSV file|No live inqu<br>of payment<br>receives a r<br>morning|iry; PayLink uploads a file<br>s once per day and<br>esult file the next|
|Phase 2 (th<br>hange, amou<br>rchitecture<br>he system.<br>.5 Payment<br>very payme<br>his is the ba<br>From|e change requ<br>nts move from<br>must absorb t<br>life cycle<br>nt is in exactly<br>sis of your sta<br>Event|est) ElectroCo announc<br>pounds to piasters, an<br>his by changing only the<br>one state. Your design m<br>te diagram and State pa|es a new API<br>d every reque<br>ElectroCo in<br>ust allow onl<br>ttern).|version: field names<br>st must be signed. Your<br>tegration, not the rest o<br>y the transitions below<br>To|
|Initiated|Fraud check|s pass and wallet has enou|gh funds|FundsHeld|
|Initiated|Fraud check|fails or funds are insuffici|ent|Rejected|
|FundsHeld|Request sen|t to biller||SentToBiller|
|SentToBiller|Biller confirm|s||Completed|
|SentToBiller|Biller refuses|||Failed|
|SentToBiller|No answer w|ithin the timeout||Unknown|
|Unknown|Biller later co<br>file)|nfirms (callback, retry or|settlement|Completed|
|Unknown|Biller later re<br>payment|fuses, or reconciliation fin|ds no|Failed|



In Phase 2 (the change request) ElectroCo announces a new API version: field names change, amounts move from pounds to piasters, and every request must be signed. Your architecture must absorb this by changing only the ElectroCo integration, not the rest of the system. 

### 2.5 Payment life cycle 

Every payment is in exactly one state. Your design must allow only the transitions below (this is the basis of your state diagram and State pattern). 

Page 4 of 16 

PayLink Year Work Project - Student Brief 

|From|Event|To|
|---|---|---|
|Failed|Hold released to the wallet|Refunded|



Completed, Rejected and Refunded are final. A payment in Unknown is never refunded automatically: money is only returned once the system is sure the biller did not receive it. 

### 2.6 Business rules 

These numbers are fixed so that every team solves the same problem. Put them in configuration, not in code. 

|Rule|Value|
|---|---|
|Daily payment limit|5,000 EGP for basic accounts, 30,000 EGP for<br>verified accounts|
|Minimum and maximum single<br>payment|5 EGP and 10,000 EGP|
|Fee for electricity and water|Fixed 2.50 EGP|
|Fee for mobile top-up|1.5% of the amount, minimum 1 EGP|
|Fee for internet|Tiered: 2 EGP up to 200 EGP, 4 EGP above|
|Promotion|During a promotion period the fee is 0 for one<br>chosen biller (configurable start and end dates)|
|Agent commission|40% of the service fee, credited to the agent's<br>wallet when the payment completes|
|Velocity rule|More than 5 payments in 60 seconds from one<br>account is blocked for 10 minutes|
|Biller timeout|10 seconds for live billers; after that the payment<br>becomes Unknown|
|Retries|A payment in Unknown is re-queried up to 3 times<br>(after 1, 5 and 15 minutes) before it waits for<br>reconciliation|
|Notification retry|An SMS or email that fails is retried 3 times, then<br>marked undelivered|



Page 5 of 16 

PayLink Year Work Project - Student Brief 

|Rule|Value|
|---|---|
|Rounding|All money uses exact decimal types, never floating<br>point|



### 2.7 Hard cases your system must handle 

These are the situations that make the project realistic. Each one becomes at least one automated test. 

|#|Situation|Required outcome|
|---|---|---|
|1|The mobile app sends the same payment<br>twice because the network was slow|One payment, one charge; the second<br>request returns the first result|
|2|ElectroCo takes 12 seconds to answer|Payment becomes Unknown, customer is<br>told it is being processed, money stays<br>held|
|3|AquaCo refuses with an error code|Payment becomes Failed, hold is<br>released, customer gets an SMS with a<br>clear reason|
|4|Two payments from the same wallet<br>arrive at the same moment and the<br>balance covers only one|One succeeds, the other is Rejected; the<br>balance never goes negative|
|5|The SMS provider is down when a<br>payment completes|The payment still completes; the SMS is<br>retried and finally marked undelivered|
|6|TopUpNet calls the callback twice with<br>"success"|Payment completes once; the second<br>callback is ignored and logged|
|7|The nightly settlement file lists a<br>payment PayLink has as Failed|The difference is flagged in the<br>reconciliation report for support to<br>review|
|8|A new biller must be added|Only a new integration and configuration<br>are added; existing modules are<br>unchanged|



Page 6 of 16 

PayLink Year Work Project - Student Brief 

## 3. Requirements 

### 3.1 Modules (minimum 6) 

Each module has its own package or service, its own tests and a clear public interface. 

|Module|Responsibility|
|---|---|
|Accounts and Wallet|Customers, agents, verification levels, wallets,<br>ledger entries, holds, top-ups, refunds|
|Biller Catalog|Billers, services, required input fields (meter<br>number, phone number), min and max amounts,<br>active or disabled|
|Payment Processing|Payment life cycle, idempotency, orchestration of<br>the steps in section 2.2|
|Biller Integration|One adapter per biller that hides its protocol;<br>timeout, retry and callback handling|
|Fees and Commissions|Fee calculation (fixed, percentage, tiered,<br>promotion) and agent commission|
|Notifications|SMS, email and push messages with retry; message<br>templates|
|Fraud Checks (extra)|Daily limit, velocity and blocked-account checks as<br>a chain of rules|
|Reconciliation (extra)|Nightly comparison with settlement files and a<br>difference report|



### 3.2 Functional requirements (minimum 25 user stories) 

Turn the list below into at least 25 user stories with acceptance criteria in Phase 1. You may add more; you may not remove the ones marked must. 

|ID|Requirement|Priority|
|---|---|---|
|FR-1|A customer can register and has a basic|Must|
||verification level||



Page 7 of 16 

PayLink Year Work Project - Student Brief 

|ID|Requirement|Priority|
|---|---|---|
|FR-2|A customer can be upgraded to verified, which<br>raises limits|Should|
|FR-3|A customer can view wallet balance (available and<br>held) and transaction history|Must|
|FR-4|A customer can top up the wallet (simulated card<br>or agent cash)|Must|
|FR-5|Every balance change creates a ledger entry;<br>balances are computed from the ledger|Must|
|FR-6|An admin can add, edit and disable billers and<br>services|Must|
|FR-7|A customer can see the list of billers and the input<br>fields each requires|Must|
|FR-8|A customer can run a bill inquiry and see the<br>amount, fee and total before paying|Must|
|FR-9|A customer can pay a bill from the wallet|Must|
|FR-10|A repeated request with the same idempotency<br>key returns the original result|Must|
|FR-11|A payment follows the life cycle in section 2.5 and<br>no other transitions|Must|
|FR-12|Funds are held before the biller is called and<br>released if the payment fails|Must|
|FR-13|A payment that times out becomes Unknown and<br>is re-queried by the retry rules|Must|
|FR-14|The system accepts asynchronous callbacks from<br>TopUpNet and ignores duplicates|Must|
|FR-15|The system supports at least 3 billers with<br>different formats (ElectroCo, AquaCo, TopUpNet);<br>NetCo batch is optional|Must|
|FR-16|Fees follow the rules in section 2.6, including the<br>promotion period|Must|



Page 8 of 16 

PayLink Year Work Project - Student Brief 

|ID|Requirement|Priority|
|---|---|---|
|FR-17|An agent can pay on behalf of a customer and<br>earns commission|Should|
|FR-18|Payments above the single-payment limit or the<br>daily limit are rejected with a clear reason|Must|
|FR-19|More than 5 payments in 60 seconds blocks the<br>account for 10 minutes|Should|
|FR-20|A support employee can search a payment by<br>reference and see its full history|Must|
|FR-21|A support employee can block and unblock an<br>account|Should|
|FR-22|A customer receives an SMS and a push message<br>on completion and on failure|Must|
|FR-23|Failed notifications are retried 3 times then marked<br>undelivered|Should|
|FR-24|An admin can edit fee and limit values without<br>redeploying|Should|
|FR-25|Every money movement and admin action is<br>written to an audit log|Must|
|FR-26|The system produces a daily reconciliation report<br>from a settlement file|Should|
|FR-27|A customer can request a refund of a completed<br>payment; support approves it|Could|
|FR-28|An admin can view daily totals: payments, fees<br>earned, commissions paid, failures per biller|Could|



### 3.3 Non-functional requirements 

Choose and measure at least 6 in Phase 1. The following are the expected ones; your tests or reports must show the number. 

Page 9 of 16 

PayLink Year Work Project - Student Brief 

|Quality|Target|
|---|---|
|Correctness|Zero double charges and zero negative balances in<br>the concurrency and duplicate tests|
|Performance|100 payments per second sustained for 1 minute<br>against the mock billers|
|Resilience|A biller outage does not stop payments to other<br>billers|
|Modifiability|Adding a biller touches only the Biller Integration<br>module and configuration|
|Auditability|100% of money movements have an audit record<br>with who, when and why|
|Testability|At least 70% line coverage on domain logic,<br>measured in CI|



### 3.4 Technical requirements 

- **Language and framework:** your choice (Java Spring Boot, .NET, Node NestJS or Python FastAPI), declared in Phase 1 and fixed afterwards. 

- **Database:** a relational database with at least 10 tables, schema migrations stored in the repository. 

- **API:** at least 20 REST endpoints documented with OpenAPI, plus one asynchronous flow (a queue, or events inside the application first, then through a broker in Phase 2). **Design patterns:** at least 5 different patterns applied for a real reason. Suggested: Adapter and Factory (billers), Strategy (fees), State (payment), Chain of Responsibility (fraud checks), Observer (notifications), Command (retries). 

- **Architecture:** design a layered or hexagonal modular monolith in Phase 1 and build it first in Phase 2; then evolve at least Notifications and Reconciliation to an eventdriven or service-based style, with an architecture decision record explaining the choice and the alternatives. 

- **Docker:** a Dockerfile for each runnable component and one docker compose file that starts the application, the database and the mock billers. 

- **CI/CD:** a GitHub Actions workflow that builds, lints, tests, reports coverage, builds images and deploys the stack locally. 

Page 10 of 16 

PayLink Year Work Project - Student Brief 

**Tests:** unit tests, integration tests for the API and database, and one automated test for each hard case in section 2.7. 

**UI:** a simple web page or a command-line client is enough. Marks are for the backend. 

## 4. Deliverables and deadlines 

**How to submit:** for each phase, tag your repository ( `phase-1` , `final` ) and upload one PDF report to the course page by the deadline. The tag must contain everything listed. The TA grades the tagged version, not later commits. 

|Phase|Deadline|Points|You submit|
|---|---|---|---|
|1. Requirements, design,<br>DevOps foundation|Wednesday 25<br>November|30|Requirements pack, design<br>pack, Docker and CI<br>skeleton, prompt log|
|2. Final system|Friday 18 December|60|Working system with tests,<br>5 patterns, architecture<br>evolution, change request,<br>local deployment, report,<br>demo video|
|Individual discussion|In your lab, from<br>Saturday 19<br>December|10|Live change and questions,<br>alone|



### Phase 1 (due 25 November): requirements, design, DevOps foundation 

1. **Requirements pack (PDF +** **`/docs/requirements.md` ):** problem summary in your own words; stakeholders; at least 25 user stories with acceptance criteria written as given/when/then; at least 6 measurable non-functional requirements; priorities (Must, Should, Could); a list of questions the AI asked or ambiguities you found, and the decisions you made. 

2. **Design pack (** **`/docs/design/` , source files, not screenshots):** use-case diagram; domain class diagram; sequence diagrams for "pay a bill" and "refund a failed payment"; state diagram of the payment (section 2.5); ER diagram of the database. Use PlantUML or Mermaid, generated by prompts and then corrected by you. 

Page 11 of 16 

PayLink Year Work Project - Student Brief 

3. **DevOps foundation:** repository with protected main branch and pull-request rules; a Dockerfile and docker compose file that start an empty application and the database; a GitHub Actions workflow that builds, lints and runs at least one test on every push. 

4. **Prompt log:** at least 5 entries, including 2 where the first AI answer was wrong and you improved the prompt. 

**Done when:** a teammate can clone the repository, run `docker compose up` , see the application start, and see a green pipeline on the `phase-1` tag. 

### Phase 2, final (due 18 December): the complete system 

You have three weeks (the midterm is on 12 December, so plan around it). Phase 2 includes everything built after Phase 1. 

1. **Working system:** Accounts and Wallet, Biller Catalog, Payment Processing, Biller Integration, Fees and Commissions, and Notifications, running with the database against at least three mock billers (ElectroCo, AquaCo, TopUpNet), with the API documented in OpenAPI. 

2. **Tests with TDD evidence:** for at least 3 features, the commit history shows the failing test first, then the code, then the cleanup. Coverage above 70% on domain logic, reported by the pipeline. One automated test for each of the 8 hard cases in section 2.7. 

3. **Five design patterns:** for each, a short note with the problem, the code before, the AI refactoring prompt and answer, your critique of the answer (what was wrong, unnecessary or risky), and the final code. Add a table mapping each pattern to the folders in your repository. 

4. **Architecture:** at least 3 architecture decision records in `/docs/adr/` (architecture style, how modules communicate, who owns which data), each comparing at least 2 options with trade-offs. Add a one-page analysis of performance, availability and modifiability for your choice. Then evolve Notifications and Reconciliation through a message broker or event bus (or an equivalent service split), running in docker compose. 

5. **Change request:** apply the ElectroCo API v2 change (section 2.4). The report lists exactly which files changed and shows that the other modules did not. 

6. **Pipeline and local deployment:** on every push the pipeline builds, lints, runs all tests, reports coverage and builds the images. On main it deploys the full stack locally (selfhosted runner or compose on a team laptop), runs a health check, and documents a rollback step. 

Page 12 of 16 

PayLink Year Work Project - Student Brief 

7. **Deployment comparison (one page):** your local deployment against two cloud options (for example a virtual machine and a managed container service) in cost, scaling and risk. 

8. **Final report (maximum 15 pages):** architecture overview, pattern catalogue, test and coverage results, pipeline description, and lessons about AI (where it helped, where it failed, with numbers from your prompt log). 

9. **Demo video (8 minutes):** pay a bill through a mock biller, show a failed payment being refunded, then show the pipeline deploying. 

10. **Prompt log:** at least 5 new entries, including at least 5 AI suggestions you rejected or fixed, with the reason. The README lets a stranger run the system with two commands. 

**Done when:** every minimum item in section 3 is met, a push to main runs the tests and redeploys locally, and the ElectroCo v2 change works. 

### Individual discussion (in your lab, from 19 December) 

Your TA announces the lab schedule. You attend alone. You receive a small change on your own team's code, such as a new biller, a new fee rule or a new notification channel, and implement it live with a test (20 minutes). Then you answer questions about your commits and your team's decisions (10 minutes). 

## 5. Using AI 

Use AI at every stage: requirements, diagrams, code, tests, refactoring, pipeline files and ADRs. You are graded on your judgement of the output, not on the output itself. 

- Write structured prompts: role, context, constraints, output format, and an example when useful. 

- Keep the prompt log in `/docs/prompt-log.md` using the table below. Be honest: rejected answers earn marks when you explain why. 

- You must understand every line you commit. Name the AI tool in the commit message when AI wrote more than half of a change. 

- Another teammate reviews every pull request before merge. Run and test AI code before you trust it. 

- Never paste passwords, real personal data or other teams' code into an AI tool. 

- Copying another team's work, or not being able to explain your own code at the discussion, is academic misconduct, whatever your log says. 

Page 13 of 16 

PayLink Year Work Project - Student Brief 

|Date|Author|Stage|Tool and|Prompt|Summary of|Verdict (accepted, fixed,|
|---|---|---|---|---|---|---|
||||model||answer|rejected) and why|
|2026-|Student|Design|(tool|(paste your|(2 lines)|Fixed: the AI missed the held|
|11-12|A||name)|prompt)||balance in the wallet class|



## 6. Grading (100 points) 

The total is rescaled to the year-work marks. 

Page 14 of 16 

PayLink Year Work Project - Student Brief 

|Phase|Criterion|Points|
|---|---|---|
|1|Requirements: complete, testable, prioritised,<br>ambiguities resolved|10|
|1|Diagrams: correct, consistent with each other, AI<br>errors documented and fixed|10|
|1|Docker and first CI workflow work from a clean<br>clone|5|
|1|Prompt log: structured prompts, honest verdicts|5|
|2|Working system and documented API; all 8 hard<br>cases pass|10|
|2|TDD evidence and coverage above 70%|10|
|2|Design patterns: right pattern for a real problem,<br>critique of AI refactoring|10|
|2|Architecture decision records, quality analysis and<br>event-driven evolution|10|
|2|ElectroCo v2 change absorbed by changing only<br>the integration|5|
|2|Pipeline, local deployment with health check and<br>rollback, deployment comparison|10|
|2|Final report, README and 8-minute demo|5|
|2|Individual discussion: live change and questions|10|
||**Total**|**100**|



## 7. Rules 

- **Teams:** register through the form in section 1. Changes to the team need instructor approval before Phase 2. 

- **Roles rotate between phases:** requirements and product lead, quality and DevOps lead, architecture lead. Your report names who led what. 

Page 15 of 16 

PayLink Year Work Project - Student Brief 

- **Repository:** one private GitHub repository; add the instructor and TAs; main branch protected; work through pull requests and issues. 

- **Late work:** minus 10% of that phase's points for each day late, up to 3 days; after that the phase gets zero. 

**Questions:** ask in the lab or on the course page; do not share code between teams. 

Page 16 of 16 

