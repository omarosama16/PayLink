# **PayLink** 

Phase 1 Task Distribution and Execution Plan 

|Course|Software Design - Year Work Project (PayLink Student Brief)|
|---|---|
|Phase|Phase 1: Requirements, Design, DevOps Foundation (30 points)|
|Deadline|Wednesday 25 November 2026 (brief, page 1). Late penalty: minus 10% of the phase points per<br>day, up to 3 days, then zero (brief, page 16).|
|Technology stack|Backend: Java 21, Spring Boot, REST APIs, PostgreSQL. Frontend: Next.js. Containers: Docker<br>and docker compose. CI: GitHub Actions.|
|Document owner|Architecture Lead|
|Status|Working document for the team. Task completion is tracked through GitHub issues (task D6).|



## **1. Purpose of this document** 

This document assigns every Phase 1 task to a named team member, explains what each task requires, states where the requirement is written in the student brief (section and page), and defines when the task counts as done. Page numbers refer to the footer of the brief (Page X of 16). Each member should be able to explain every line of their own deliverables during the individual discussion (brief, page 13). 

## **2. Phase 1 at a glance** 

Phase 1 contains four graded items (brief, page 15). The deliverable list is in Section 4 of the brief, pages 11 and 12. 

|**Graded item**|**Points**|**What the brief requires**|**Brief reference**|
|---|---|---|---|
|Requirements|10|Complete, testable, prioritised requirements with ambiguities<br>resolved. At least 25 user stories with Given/When/Then<br>acceptance criteria, at least 6 measurable non-functional<br>requirements, priorities (Must, Should, Could), and a list of AI<br>questions and ambiguities with decisions.|Section 4, Phase 1 item 1,<br>p.11; grading p.15|
|Diagrams|10|Use-case, domain class, sequence (pay a bill), sequence<br>(refund a failed payment), payment state, and ER diagrams as<br>source files in /docs/design/. Correct, consistent with each<br>other, AI errors documented and fixed.|Section 4, Phase 1 item 2,<br>p.11; grading p.15|
|Docker and first CI<br>workflow|5|Protected main branch with pull-request rules. Dockerfile and<br>docker compose file that start an empty application and the<br>database. GitHub Actions workflow that builds, lints and runs<br>at least one test on every push. Must work from a clean clone.|Section 4, Phase 1 item 3,<br>p.12; grading p.15|
|Prompt log|5|At least 5 entries in /docs/prompt-log.md, including 2 where<br>the first AI answer was wrong and the prompt was improved.<br>Structured prompts and honest verdicts.|Section 4, Phase 1 item 4,<br>p.12; Section 5, pp.13-14|



## **3. Team structure and roles** 

The brief requires roles to rotate between phases and the report to name who led what (brief, pages 15-16, Rules). The following roles apply to Phase 1. Roles are expected to rotate in Phase 2. 

PayLink - Phase 1 Task Distribution and Execution Plan 

Page 1 

|**Member**|**Phase 1 role**|**Primary ownership**|**Approx. points owned**|
|---|---|---|---|
|Youssef|Requirements and<br>Product Lead|Requirements pack, use-case diagram,<br>prompt log|Requirements 10, Prompt log 5<br>(shared evidence), 1 of 6<br>diagrams|
|Omar|Quality and DevOps<br>Lead|Repository, Docker, compose, CI,<br>clean-clone verification|Docker and CI 5|
|Mohamed|Architecture Lead|State, class, ER and two sequence diagrams,<br>diagram consistency, stack declaration,<br>final report and tag|Diagrams 10 (5 of 6 diagrams plus<br>consistency review)|



Note on workload: the 10 diagram points are graded on the set as a whole, including consistency between diagrams. The Architecture Lead therefore owns the consistency review of all six diagrams, even the two drawn by other members. 

## **Quick task summary** 

### **Youssef - Requirements** 

- Write the problem summary in your own words and list the stakeholders (the story is in Section 2.1, pages 1-2). 

- Convert FR-1 to FR-28 into at least 25 user stories in Given / When / Then form (the FR table is on pages 7-9). 

- Write 6 non-functional requirements with numbers (see the Quality table, page 10). 

- Record the questions, ambiguities and decisions. 

- Draw the use-case diagram. 

- Own the prompt log (format in Section 5, pages 13-14; at least 2 entries must be cases where the first AI answer was wrong). 

### **Omar - DevOps** 

- Create a private GitHub repository, add the instructor and TAs, and protect main with pull requests. 

- Set up Docker and docker compose (empty application plus the database). 

- Create the GitHub Actions workflow: build, lint and test on every push (Section 3.4, pages 10-11). 

- Add the Next.js frontend as a service in the compose file. 

- Test by cloning the repository into a new empty folder and running it. 

### **Mohamed - Design** 

- Payment state diagram (table in Section 2.5, pages 4-5). 

- Class diagram. 

- Draw the ER diagram (at least 10 tables). 

- Sequence diagram for paying a bill (steps in Section 2.2, page 2). 

- Sequence diagram for refunding a failed payment. 

- Review all diagrams for consistency, assemble the report and create the phase-1 tag. 

PayLink - Phase 1 Task Distribution and Execution Plan 

Page 2 

## **4. Task assignments in detail** 

Each task lists the brief reference where the requirement is defined, the output that must exist in the repository, and the condition under which the task is considered done. 

### **4.1 Youssef - Requirements and Product Lead** 

|**ID**<br>R1|**Task**<br>Read Sections 2 and 3 of the brief in full: the<br>payment story, the nine-step payment flow, key<br>terms, billers, life cycle, business rules, hard<br>cases, modules and the requirement tables.|**Brief reference**<br>Sec. 2, pp.1-6; Sec. 3,<br>pp.7-10|**Output and done when**<br>Can explain the payment flow and<br>every business rule without the<br>document open.|
|---|---|---|---|
|R2|Write a problem summary in the team's own<br>words (5 to 6 sentences): what PayLink is, what it<br>earns, what risk it carries, and what management<br>wants.|Sec. 2.1, pp.1-2; Sec. 4<br>item 1, p.11|Section 1 of /docs/requirements.md.<br>Not copied from the brief.|
|R3|List the stakeholders and what each cares about:<br>customer, agent, biller, support employee, admin,<br>management/finance, regulator/auditor.|Sec. 2.1-2.3, pp.1-3; FR<br>table, pp.7-9|Stakeholder table in<br>requirements.md.|
|R4|Convert FR-1 to FR-28 into at least 25 user<br>stories. Each story has an ID, priority (Must,<br>Should or Could taken from the FR table), the FR<br>it traces to, and acceptance criteria written as<br>Given / When / Then. Add stories for the hard<br>cases that are not covered by an FR.|FR table, pp.7-9; Hard<br>cases, p.6; Sec. 4 item<br>1, p.11|At least 25 stories. Every FR has at<br>least one story. Every hard case 1 to<br>8 appears in at least one acceptance<br>criterion.|
|R5|Define at least 6 measurable non-functional<br>requirements. Each must contain a number and a<br>way to measure it (for example: 100 payments per<br>second sustained for 1 minute, 70% line coverage<br>on domain logic, zero double charges in<br>concurrency tests).|Sec. 3.3, p.9; Quality<br>table, p.10; Sec. 4 item<br>1, p.11|NFR table with columns: ID, quality,<br>target, how measured.|
|R6|Record ambiguities and decisions. Run the brief<br>through the AI and ask it for questions and gaps.<br>Record each question or ambiguity and the team's<br>decision. Examples to settle: whether the daily<br>limit counts the bill amount or the total with fee,<br>calendar day or rolling 24 hours, what happens to<br>the fee on a failed payment.|Sec. 2.6, pp.5-6; Sec. 4<br>item 1, p.11|Numbered table of questions and<br>decisions in requirements.md.|
|R7|Draw the use-case diagram: actors (customer,<br>agent, support, admin, biller, notification provider)<br>and use cases including include/extend<br>relationships for fraud checks, fee calculation,<br>hold of funds and notification.|Sec. 4 item 2, p.11;<br>Modules, p.7|/docs/design/usecase.puml<br>(PlantUML source, not a screenshot).|
|R8|Own the prompt log. Record every significant AI<br>interaction from all three members using the table<br>format in the brief. Make sure at least 2 entries are<br>cases where the first answer was wrong and the<br>prompt was improved. Verdict must be one of<br>accepted, fixed, rejected, with the reason.|Sec. 4 item 4, p.12;<br>Sec. 5, pp.13-14; table<br>format p.14|/docs/prompt-log.md with at least 5<br>genuine entries.|



**Deliverables:** /docs/requirements.md, a PDF export of it for the report, /docs/design/usecase.puml, /docs/prompt-log.md. **Reviewer:** Mohamed. 

PayLink - Phase 1 Task Distribution and Execution Plan 

Page 3 

### **4.2 Omar - Quality and DevOps Lead** 

|**ID**|**Task**|**Brief reference**|**Output and done when**|
|---|---|---|---|
|D1|Create one private GitHub repository. Add the<br>instructor and the TAs as collaborators. Protect<br>the main branch: require pull requests and at least<br>one review before merge. Enable issues.|Sec. 4 item 3, p.12;<br>Rules, pp.15-16<br>(Repository)|Direct push to main is blocked.<br>Instructor and TAs can see the<br>repository.|
|D2|Set up the backend project (Spring Boot) with a<br>Dockerfile, and a docker compose file that starts<br>the application and PostgreSQL. Add database<br>migrations in the repository from the start.|Sec. 4 item 3, p.12;<br>Sec. 3.4 Docker and<br>Database, p.10|docker compose up starts the<br>application and database with no<br>manual steps. Health endpoint<br>responds.|
|D3|Add the Next.js frontend as a separate runnable<br>component with its own Dockerfile and a service<br>in the compose file. A single placeholder page is<br>enough in Phase 1.|Sec. 3.4 Docker (a<br>Dockerfile for each<br>runnable component),<br>p.10; UI note, p.11|Compose starts three services:<br>database, backend, frontend.|
|D4|Create the GitHub Actions workflow. On every<br>push it must build, lint and run at least one test.<br>(Coverage, image build and deployment are<br>added in Phase 2.)|Sec. 4 item 3, p.12;<br>CI/CD requirement,<br>p.10|Green pipeline on the phase-1 tag.|
|D5|Create GitHub issues for all Phase 1 tasks and<br>assign them. Require a pull request with a<br>teammate review for every merge.|Rules, pp.15-16; Using<br>AI, p.13 (every PR<br>reviewed by a<br>teammate)|Every task is an issue linked to a pull<br>request.|
|D6|Clean-clone test: clone the repository into a new<br>empty folder on a different machine or account,<br>run docker compose up, and confirm the<br>application starts and the pipeline is green.|Sec. 4 'Done when',<br>p.12|Written confirmation in the report,<br>with the date and who ran it.|



**Deliverables:** repository settings, Dockerfile(s), docker-compose.yml, .github/workflows/ci.yml, README with the run instructions. **Reviewer:** Youssef, who also performs the clean-clone check independently. 

PayLink - Phase 1 Task Distribution and Execution Plan 

Page 4 

### **4.3 Mohamed - Architecture Lead** 

|**ID**<br>A1|**Task**<br>Draw the payment state diagram. States: Initiated,<br>FundsHeld, SentToBiller, Completed, Failed,<br>Unknown, Rejected, Refunded. Include only the<br>nine transitions listed in the brief. Completed,<br>Rejected and Refunded are final.|**Brief reference**<br>Sec. 2.5, pp.4-5; FR-11,<br>p.8|**Output and done when**<br>/docs/design/state-payment.puml.<br>Transition count matches the table<br>exactly.|
|---|---|---|---|
|A2|Draw the domain class diagram covering the<br>modules: Accounts and Wallet, Biller Catalog,<br>Payment Processing, Biller Integration, Fees and<br>Commissions, Notifications, Fraud Checks,<br>Reconciliation. Show the wallet with available and<br>held balance computed from ledger entries. Mark<br>where each suggested pattern (Adapter, Factory,<br>Strategy, State, Chain of Responsibility, Observer,<br>Command) will apply.|Sec. 3.1, p.7; Key<br>terms, pp.2-3; Sec. 3.4<br>Design patterns, p.10|/docs/design/class-domain.puml.|
|A3|Draw the ER diagram for the database. It must<br>show at least 10 tables including customers,<br>wallets, ledger entries, billers, services, payments,<br>payment transitions, fee and limit configuration,<br>notifications, and audit log. Balances must not be<br>a stored column: they are computed from ledger<br>entries.|Sec. 3.4 Database,<br>p.10; Key terms<br>(Ledger), p.3; FR-5, p.8;<br>Sec. 4 item 2, p.11|/docs/design/er.puml (source file).|
|A4|Draw the sequence diagram for paying a bill<br>following the nine steps of the payment story:<br>login and limits, bill inquiry, fee and total, fraud<br>checks, hold, biller call, confirmation, receipt, plus<br>the idempotency check and the timeout path.|Sec. 2.2, p.2; Hard<br>cases 1, 2, 4, p.6; Sec.<br>4 item 2, p.11|/docs/design/sequence-pay-bill.puml.|
|A5|Draw the sequence diagram for refunding a failed<br>payment: biller refusal, transition to Failed,<br>release of the hold, transition to Refunded,<br>customer notification. Show that an Unknown<br>payment is never refunded automatically.|Sec. 2.5, pp.4-5; Hard<br>case 3, p.6; Sec. 4 item<br>2, p.11|/docs/design/sequence-refund-failed.<br>puml.|
|A6|Consistency review of all six diagrams: same<br>state names in the state, class and sequence<br>diagrams; same entities and fields in the class and<br>ER diagrams; same actors in the use-case and<br>sequence diagrams. Document every AI error<br>found and how it was fixed.|Grading, p.15 ('correct,<br>consistent with each<br>other, AI errors<br>documented and fixed')|Consistency checklist in the report.<br>Errors listed in the prompt log.|
|A7|Declare the technology stack and document the<br>intended architecture: a layered or hexagonal<br>modular monolith with one package per module<br>and a clear public interface. Apply the<br>Dependency Rule from the Clean Architecture<br>reference (business rules do not depend on<br>frameworks or the database).|Sec. 3.4 Language and<br>Architecture, p.10;<br>Clean Architecture<br>project file, Ch. 22 and<br>Ch. 34|Stack declaration in<br>requirements.md. Short architecture<br>note in /docs/design/.|
|A8|Assemble the report: one PDF containing the<br>requirements pack, the design pack, the DevOps<br>description, the prompt log summary, the role list<br>(who led what) and the clean-clone confirmation.<br>Create the phase-1 tag and upload the PDF to the<br>course page.|Sec. 4 'How to submit',<br>p.11; Rules, pp.15-16|Tag phase-1 exists, contains all files,<br>and the pipeline is green on it.|



**Deliverables:** five design source files, consistency checklist, final report PDF, phase-1 tag. **Reviewer:** Omar reviews the diagrams; Youssef reviews the report. 

## **5. Review matrix** 

The brief requires that another teammate reviews every pull request before merge and that AI-generated code or text is run and tested before it is trusted (brief, page 13). 

PayLink - Phase 1 Task Distribution and Execution Plan 

Page 5 

|**Work authored by**|**Reviewed by**|**Review focus**|
|---|---|---|
|Youssef (requirements, use-case,<br>prompt log)|Mohamed|Every FR covered; stories testable; numbers present in each<br>NFR; use-case consistent with stories.|
|Mohamed (state, class, ER, sequences,<br>report)|Omar|State diagram matches section 2.5 exactly; class and ER<br>agree; report contains every required item.|
|Omar (repository, Docker, CI)|Youssef|Runs a fresh clone and docker compose up; confirms<br>pipeline is green.|



## **6. Timeline** 

|**Date**|**Goal**|**Owner(s)**|
|---|---|---|
|Sunday 22 Nov|Team agrees the roles in this document. Repository created, instructor<br>and TAs added, main protected, issues created. Everyone reads<br>Sections 2 and 3 of the brief.|Omar (repo); all<br>(reading)|
|Monday 23 Nov|First complete drafts: requirements, all six diagrams, Docker and CI<br>running. Prompt log updated with real entries.|All|
|Tuesday 24 Nov|Cross-reviews done. Errors fixed and logged. Consistency review<br>complete. Clean-clone test by a second person.|All; Mohamed<br>(consistency);<br>Youssef (clone test)|
|Wednesday 25 Nov<br>(morning)|Report PDF assembled, phase-1 tag created, pipeline confirmed green,<br>PDF uploaded. Submit early in the day to leave margin.|Mohamed|



## **7. Submission checklist** 

The tag must contain everything listed. The TA grades the tagged version, not later commits (brief, page 11). 

- /docs/requirements.md and its PDF: at least 25 user stories (Given/When/Then), at least 6 measurable NFRs, priorities, problem summary, stakeholders, ambiguities and decisions. 

- /docs/design/: use-case, class, sequence (pay a bill), sequence (refund), state and ER diagrams as source files, not screenshots. 

- Repository with protected main, pull-request rules, Dockerfile(s) and docker compose that start the empty application and the database. 

- GitHub Actions workflow that builds, lints and runs at least one test on every push, green on the phase-1 tag. 

- /docs/prompt-log.md with at least 5 entries, including 2 where the first AI answer was wrong. 

- One PDF report uploaded to the course page, naming who led what. 

- Done when: a teammate can clone the repository, run docker compose up, see the application start, and see a green pipeline on the phase-1 tag (brief, page 12). 

## **8. Rules for using AI** 

These rules come from Section 5 of the brief (pages 13-14). Breaking them is an academic integrity issue regardless of what the log says. 

- Use structured prompts: role, context, constraints, output format, and an example when useful. 

- Record every significant prompt in /docs/prompt-log.md using the table columns: Date, Author, Stage, Tool and model, Prompt, Summary of answer, Verdict (accepted, fixed, rejected) and why. Rejected answers earn marks when the reason is explained. 

- Understand every line that is committed. Name the AI tool in the commit message when AI wrote more than half of a change. 

- Run and test AI output before trusting it. A teammate reviews every pull request before merge. 

PayLink - Phase 1 Task Distribution and Execution Plan 

Page 6 

- Never paste passwords, real personal data or another team's code into an AI tool. 

- Copying another team's work, or being unable to explain your own code at the discussion, is academic misconduct. 

Starter material note: draft versions of the requirements, the six diagrams, a Spring Boot skeleton, Dockerfile, compose file and CI workflow were generated with AI assistance as a starting point. None of it has been run or verified. It must be reviewed, corrected and understood by the owner of each task, and the interaction must be recorded in the prompt log with an honest verdict. 

## **9. Risks and mitigations** 

|**Risk**|**Mitigation**|
|---|---|
|Diagrams disagree with each other (names,<br>states, entities).|Single shared glossary of state and entity names; consistency review by<br>the Architecture Lead before submission (task A5).|
|Pipeline or docker compose fails only on a fresh<br>machine.|Clean-clone test by a second person on Tuesday (task D7).|
|Prompt log is written at the end and looks<br>artificial.|Log entries as they happen. Youssef checks the log daily; at least 2<br>entries must be real wrong-first-answer cases.|
|A member cannot explain their deliverable at the<br>individual discussion.|Each member walks the others through their deliverables on Tuesday;<br>the reviewer asks questions as if grading.|
|Late submission penalty (10% per day).|Submit Wednesday morning. Treat Tuesday evening as the real<br>deadline.|



## **Appendix A. Where to find each requirement in the brief** 

|**Topic**|**Section**|**Pages**|
|---|---|---|
|Project summary, tools, deadlines|1. Project at a glance|1|
|Story and nine-step payment flow|2.1, 2.2|1-2|
|Key terms (wallet, ledger, hold, idempotency, reconciliation)|2.3|2-3|
|Billers: ElectroCo, AquaCo, TopUpNet, NetCo; ElectroCo v2 change|2.4|3-4|
|Payment life cycle and allowed transitions|2.5|4-5|
|Business rules (limits, fees, commission, velocity, timeouts, retries)|2.6|5-6|
|Eight hard cases|2.7|6|
|Modules (minimum 6)|3.1|7|
|Functional requirements FR-1 to FR-28|3.2|7-9|
|Non-functional requirements guidance|3.3|9|
|Technical requirements, quality targets|3.4|10|
|Phase 1 and Phase 2 deliverables, how to submit|4|11-13|
|Using AI, prompt log format|5|13-14|
|Grading|6|14-15|
|Rules: teams, roles, repository, late work|7|15-16|



PayLink - Phase 1 Task Distribution and Execution Plan 

Page 7 

