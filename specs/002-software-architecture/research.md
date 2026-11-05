# Architecture Research and Decisions

**Scope**: Resolve choices needed for a base logical architecture from repository documentation. This research does not infer code or deployed infrastructure; no application source or build manifest was found.

## Decision 1: Use a layered logical architecture

**Decision**: Separate client presentation, application use cases/authorization, domain rules, and persistence/notification/reporting adapters.

**Rationale**: The constitution requires separation of GUI, business logic, and persistence boundaries, and asks for cohesive domain models independent of UI widgets. A logical layered design accommodates the currently documented Swing direction and required web/mobile clients without asserting a deployment arrangement.

**Alternatives considered**: A single UI-centric application would couple money and approval rules to one client and would not provide a clear path to the required additional clients. A detailed distributed service architecture cannot be justified without source, scale, or deployment requirements.

## Decision 2: Keep client and service technology choices open

**Decision**: Represent web, Android, iOS, and Swing as logical clients. Use a technology-neutral application boundary. Do not select a web framework, mobile framework, API protocol, or server runtime in this base design.

**Rationale**: The product documentation specifies web-first and Android/iOS availability and separately documents Swing for a desktop MVP. It does not name implemented frameworks or client/server integration. The repository has no source or manifests to inspect.

**Alternatives considered**: Selecting React, a specific mobile toolkit, REST/GraphQL, or a Java server would invent implementation details not supported by the project evidence.

## Decision 3: Use domain concepts from the requirements as the conceptual model

**Decision**: Model Account, Group, Membership, Expense, ExpenseShare, ApprovalDecision, Receipt, Notification, Payment, and Balance as conceptual entities/value concepts. Treat Balance as either a derived view or a consistent projection; defer the persistence choice.

**Rationale**: The entities correspond directly to customer features and documented refinements. The spec explicitly leaves storage technology open.

**Alternatives considered**: Treating every entity as an existing class/table would falsely claim implementation. Adding unrelated entities (for example, external payment transaction or tax records) is unsupported by requirements.

## Decision 4: Keep financial rules in the domain boundary

**Decision**: Validate split data and approval state in domain/application operations independent of UI. Use `BigDecimal` for Java monetary calculations, and allocate in the currency's smallest unit using the documented largest-remainder method once currency scale and deterministic tie-breaking have been defined.

**Rationale**: Exact money arithmetic, largest-remainder allocation, and separation from GUI code are documented project constraints/refinements.

**Alternatives considered**: Binary floating point is disallowed by the constitution. Choosing a specific currency scale or equal-remainder tie-break now would answer an existing open question without evidence.

## Decision 5: Treat persistence and side effects as ports/adapters

**Decision**: Define conceptual persistence, notification, and reporting boundaries, with no concrete vendor or delivery mechanism. Expense submission persists a pending expense and makes in-app notifications available; approval completion creates the balance effect.

**Rationale**: Persistence and in-app notifications are necessary to the stated behavior, but product and infrastructure choices are absent. Isolating these boundaries makes the intended responsibilities clear while keeping the design vendor-neutral.

**Alternatives considered**: Naming a database, object store, queue, push service, or cloud provider would add unsupported technology. Synchronous versus asynchronous delivery and retry guarantees require operational requirements.

## Decision 6: Keep financial history authoritative and avoid duplicate effects

**Decision**: Preserve approved expenses and payment records as the business history from which balances can be derived or reconciled. Any maintained balance projection must remain consistent with that history and must not apply a financial event twice.

**Rationale**: The customer requires accurate balances and payment history; the spec calls out duplicate application as an edge case. This is an architectural integrity constraint, not a claim that an event ledger or idempotency mechanism already exists.

**Alternatives considered**: A specific event-sourcing pattern or transactional outbox is not selected because no scale, persistence, or delivery guarantees are documented.

## Unresolved Decisions

- Full product deployment topology and whether clients use one shared service or another design.
- Web, Android, and iOS frameworks; browser/OS versions; distribution; parity details.
- Database, receipt storage, backup, recovery, retention, availability, and encryption specifics.
- API protocol, authentication/session mechanism, authorization matrix, and account recovery.
- Currency set/scale, percentage input precision, and deterministic allocation tie-break.
- Approval roles, self-approval, rejection, edit, and resubmission behavior.
- Payment overpayment, reversal, correction, dispute, and external payment provider behavior.
- Notification delivery guarantees, retry policy, and read/retention behavior.
- Report export formats and time-zone boundary convention.
- Performance, scale, accessibility, and service-level targets.

These choices do not block the logical base architecture or its UML diagrams; they are required before a production topology or implementation-specific plan can be considered complete.
