# SplitSmart Alternative Architecture: Modular Hexagonal Design

**Purpose**: Independent architecture proposal for CIS 285 Lab 7 and later comparison.  
**Status**: Proposed; not an as-built architecture.  
**Requirements reviewed**: [UserRequirement.md](../../UserRequirement.md) and [spec.md](spec.md).  
**Implementation evidence**: The repository contains product/design documents but no application source or build manifest. The components, adapters, and behavior below are proposals, not implementation claims.

## 1. Architecture Overview and Design Rationale

This alternative uses a ports-and-adapters (hexagonal) architecture. At its center are the SplitSmart domain model and application use cases. The core defines inbound ports for user actions and outbound ports for capabilities it needs, such as repositories, identity/authorization, receipt storage, and notifications. Adapters outside the core connect those ports to web, Android, iOS, Swing, storage, and notification technologies.

Dependencies point inward: adapters depend on application-owned ports; application services coordinate domain objects; the domain model has no dependency on UI frameworks, databases, or notification vendors. This keeps split, approval, and balance rules consistent across clients and independently testable.

The proposed initial deployment is a modular monolith with clear internal module boundaries. This is a recommendation, not an existing implementation fact. It keeps financial state transitions within one consistency boundary and avoids premature distributed services. Web and mobile clients would enter through a server-side adapter. The documented Swing MVP can call the same use cases in-process for a local prototype, or connect through the remote client contract if deployed separately. The requirements do not settle the desktop deployment mode or API protocol.

This design preserves the customer scope: accounts, groups, expense entry, equal/percentage splits, optional receipts, notifications, approval before balance effects, bilateral balances, settlement payments, reports, and web-first plus Android/iOS availability. The feature spec adds refinements such as one payer, invitation-controlled membership, positive expense amounts, percentage totals of 100%, largest-remainder allocation, in-app notifications, and partial payments. These are requirements, not proof of implementation.

## 2. Major Components and Responsibilities

| Component | Responsibility | Boundary/status |
| --- | --- | --- |
| Domain model | Account, Group, Membership, Expense, ExpenseShare, ApprovalPolicy/Decision, Payment, and monetary concepts; owns entity invariants. | Inner core; proposed |
| Domain services | Rules spanning entities: split allocation and bilateral balance calculation. | Inner core; framework-independent |
| Application use cases | Coordinate account/group changes, expense submission, review, payment recording, balance queries, and reports; enforce authorization and transaction boundaries. | Application core; proposed |
| Inbound ports | Declare use cases such as SubmitExpense, ReviewExpense, RecordPayment, and QueryReports. | Interfaces owned by application core |
| Outbound ports | Declare repository, identity/access, receipt, notification, and reporting capabilities required by the core. | Interfaces owned by application core |
| Web/API adapter | Translate web requests into inbound-port calls and map results/errors to the client. | Outside core; protocol TBD |
| Android adapter | Present mobile workflows and invoke the same application contract. | Outside core; framework TBD |
| iOS adapter | Present mobile workflows and invoke the same application contract. | Outside core; framework TBD |
| Swing adapter | Provide the documented Java desktop MVP UI and invoke the same use cases. | Outside core; Swing is documented direction, implementation unverified |
| Persistence adapters | Implement account, group, expense, approval, payment, and notification repository ports. | Outside core; storage TBD |
| Receipt adapter | Store/retrieve optional receipt images with access control. | Outside core; technology TBD |
| Notification adapter | Make an in-app notification available to other group members upon expense submission. | Outside core; delivery implementation TBD |
| Reporting adapter/read model | Return authorized expense, balance, and payment summaries for date ranges and parameters. | Outside core; implementation TBD |

### Core domain concepts

- **Account**: Product user; credential handling belongs behind an identity boundary.
- **Group / Membership**: Shared context and member roles; detailed permission/removal policies remain open.
- **Expense**: Amount, date, description, group, submitter, one payer, shares, split method, optional receipt reference, and lifecycle status.
- **ExpenseShare**: A participant's percentage and/or allocated amount.
- **ApprovalPolicy / ApprovalDecision**: Required review levels and reviewer decisions. Approver selection, self-approval, rejection, and resubmission details remain open.
- **Payment**: Retained full or partial settlement between payer and payee.
- **Balance**: Bilateral amount derived from approved shares and payment history, or kept as a reconcilable projection.

## 3. UML Class Diagram (PlantUML)

The diagram shows ports owned by the core and adapters outside it. It describes proposed concepts, not discovered source classes.

~~~plantuml
@startuml
title SplitSmart alternative modular hexagonal class architecture
skinparam classAttributeIconSize 0
left to right direction

package "Inbound Adapters" {
  class WebApiAdapter <<adapter>>
  class AndroidClientAdapter <<adapter>>
  class IOSClientAdapter <<adapter>>
  class SwingUiAdapter <<adapter>>
}

package "Application Core" {
  interface SubmitExpense <<inbound port>>
  interface ReviewExpense <<inbound port>>
  interface RecordPayment <<inbound port>>
  interface QueryReports <<inbound port>>

  class ExpenseApplicationService <<use case>>
  class ApprovalApplicationService <<use case>>
  class PaymentApplicationService <<use case>>
  class ReportingApplicationService <<use case>>

  interface ExpenseRepository <<outbound port>>
  interface GroupRepository <<outbound port>>
  interface PaymentRepository <<outbound port>>
  interface NotificationPort <<outbound port>>
  interface ReceiptStoragePort <<outbound port>>
  interface IdentityAndAccessPort <<outbound port>>
  interface ReportReadPort <<outbound port>>
}

package "Domain Model" {
  class Account
  class Group
  class Membership
  class Expense
  class ExpenseShare
  class ApprovalPolicy
  class ApprovalDecision
  class Payment
  class Money <<value object>>
  class SplitAllocator <<domain service>>
  class BalanceCalculator <<domain service>>
}

package "Outbound Adapters" {
  class ExpenseRepositoryAdapter <<adapter; storage TBD>>
  class GroupRepositoryAdapter <<adapter; storage TBD>>
  class PaymentRepositoryAdapter <<adapter; storage TBD>>
  class InAppNotificationAdapter <<adapter>>
  class ReceiptStorageAdapter <<adapter; technology TBD>>
  class IdentityProviderAdapter <<adapter; technology TBD>>
  class ReportQueryAdapter <<adapter>>
}

WebApiAdapter ..> SubmitExpense : invokes
WebApiAdapter ..> ReviewExpense : invokes
WebApiAdapter ..> RecordPayment : invokes
WebApiAdapter ..> QueryReports : invokes
AndroidClientAdapter ..> SubmitExpense : invokes
AndroidClientAdapter ..> ReviewExpense : invokes
AndroidClientAdapter ..> RecordPayment : invokes
IOSClientAdapter ..> SubmitExpense : invokes
IOSClientAdapter ..> ReviewExpense : invokes
IOSClientAdapter ..> RecordPayment : invokes
SwingUiAdapter ..> SubmitExpense : invokes
SwingUiAdapter ..> ReviewExpense : invokes
SwingUiAdapter ..> RecordPayment : invokes

SubmitExpense <|.. ExpenseApplicationService
ReviewExpense <|.. ApprovalApplicationService
RecordPayment <|.. PaymentApplicationService
QueryReports <|.. ReportingApplicationService

ExpenseApplicationService ..> ExpenseRepository
ExpenseApplicationService ..> GroupRepository
ExpenseApplicationService ..> NotificationPort
ExpenseApplicationService ..> ReceiptStoragePort
ExpenseApplicationService ..> IdentityAndAccessPort
ExpenseApplicationService ..> SplitAllocator
ApprovalApplicationService ..> ExpenseRepository
ApprovalApplicationService ..> IdentityAndAccessPort
ApprovalApplicationService ..> BalanceCalculator
PaymentApplicationService ..> PaymentRepository
PaymentApplicationService ..> IdentityAndAccessPort
ReportingApplicationService ..> ReportReadPort

ExpenseRepository <|.. ExpenseRepositoryAdapter
GroupRepository <|.. GroupRepositoryAdapter
PaymentRepository <|.. PaymentRepositoryAdapter
NotificationPort <|.. InAppNotificationAdapter
ReceiptStoragePort <|.. ReceiptStorageAdapter
IdentityAndAccessPort <|.. IdentityProviderAdapter
ReportReadPort <|.. ReportQueryAdapter

Group "1" -- "1..*" Membership
Account "1" -- "0..*" Membership
Group "1" -- "0..*" Expense
Expense "1" *-- "1..*" ExpenseShare
ExpenseShare "*" --> "1" Account : participant
Expense "*" --> "1" Account : payer
Expense "1" *-- "0..*" ApprovalDecision
ApprovalPolicy "1" --> "1..*" ApprovalDecision : governs
ApprovalDecision "*" --> "1" Account : reviewer
ExpenseShare --> Money
Payment --> Money
Expense --> Money
BalanceCalculator ..> Expense
BalanceCalculator ..> Payment
@enduml
~~~

Adapter names do not select a database, identity provider, or receipt vendor. They identify replaceable boundaries only.

## 4. UML Activity Diagram (PlantUML)

This flow represents expense submission, notification, approval, and balance effect. Pending or rejected expenses do not affect balances.

~~~plantuml
@startuml
title Submit, approve, and apply a SplitSmart expense (proposed)
start
|Client Adapter|
:Collect amount, date, description, payer, participants, split, optional receipt;
:Invoke SubmitExpense inbound port;

|Application Core|
:Check actor identity and group access;
if (Authorized?) then (yes)
  :Load group and approval policy;
  |Domain Model|
  :Validate positive amount, payer, and participants;
  :Validate split method and percentage total;
  if (Valid expense?) then (yes)
    :Calculate shares with largest-remainder allocation;
    :Set expense status to pending;

    |Application Core|
    :Persist pending expense and shares;
    :Make in-app notification available to other group members;
    :Return pending result;
    |Client Adapter|
    :Show submitted / awaiting review;
    :Reviewer invokes ReviewExpense;

    |Application Core|
    :Authorize reviewer and record decision;
    while (Required approval levels remain?) is (yes)
      :Keep expense pending;
      :Collect next required decision;
    endwhile (no)

    if (Workflow approved?) then (yes)
      |Domain Model|
      :Calculate bilateral effects from payer and shares;
      :Exclude self-owed amount;
      |Application Core|
      :Persist approved state and financial effect once;
      |Client Adapter|
      :Show approved status and updated balance;
    else (rejected)
      |Application Core|
      :Persist rejected decision; do not change balances;
      |Client Adapter|
      :Show rejected status;
    endif
  else (no)
    |Client Adapter|
    :Show validation errors; do not save or notify;
  endif
else (no)
  |Client Adapter|
  :Show access denied;
endif
stop
@enduml
~~~

Currency scale and deterministic remainder tie-breaking must be decided before implementing allocation. Approval policy details such as self-approval, edits, rejection/resubmission, and concurrent review remain open.

## 5. Component Interactions and Data Flows

### Expense submission

1. A client adapter gathers expense details and invokes the SubmitExpense inbound port.
2. The application use case checks identity and group access through IdentityAndAccessPort, then loads group rules through repository ports.
3. Domain objects validate the positive amount, payer, participants, split, and percentage total. SplitAllocator computes shares; Java monetary calculations use BigDecimal.
4. The use case persists the expense in pending state through ExpenseRepository.
5. It calls NotificationPort so other group members can see an in-app notification at submission. Retry and delivery guarantees are unspecified.
6. The client displays pending status. No balance changes before approval.

### Approval and balance application

1. An authorized reviewer invokes ReviewExpense.
2. ApprovalApplicationService records the decision and evaluates the group's required approval levels.
3. If more review is required, the expense remains pending. A rejected expense is excluded from balance calculations.
4. After required approval completes, BalanceCalculator derives bilateral effects from payer and participant shares. The payer can be outside the participant set; a payer who is also a participant does not owe themselves.
5. The application persists approval and the effect consistently. A later implementation must decide whether balances are derived, ledger-backed, or maintained as a reconcilable projection, and prevent duplicate application.

### Settlement and reporting

- RecordPayment validates actor access, stores the full/partial payment through PaymentRepository, and makes it part of balance calculation. Overpayment, reversal, correction, and dispute rules remain open.
- QueryReports applies authorization and date-range filters and returns expense, balance, or payment summaries without changing financial state.
- ReceiptStoragePort keeps image bytes outside the domain model; the expense refers to its receipt. Access, format, size, and retention rules remain open.

## 6. Technology and Deployment Considerations

### Documented project constraints

- Java 21 and Maven for Java components.
- Java Swing for the documented desktop MVP direction, with UI handling separated from domain rules.
- BigDecimal for Java money calculations; binary floating point is unsuitable.
- JUnit 5 for Java tests, especially domain allocation and approval/balance rules independent of Swing.
- PlantUML for maintained UML diagrams.
- Web-first, Android, and iOS are customer requirements. Swing alone does not meet them.

These are documented project directions, not evidence that a working implementation exists.

### Proposed deployment

- Begin with one deployable modular monolith containing the application core and selected adapters. Keep internal boundaries around ports and domain ownership.
- Web, Android, and iOS clients connect through an inbound API adapter. HTTP with a versioned JSON contract is one candidate, but the protocol is not selected in project documents and requires a later decision.
- Swing may invoke the same core in-process for a local prototype, or call the API if deployed as a remote client. Choose based on actual MVP packaging requirements.
- Select persistence, receipt storage, and identity adapters after defining consistency, privacy, backup, recovery, and operational requirements.
- Use an in-app notification adapter for the stated channel. Do not infer push, email, or SMS.
- Do not split into microservices without measured scaling or independent deployment needs. A modular monolith reduces distributed consistency and operations burdens initially.
- Define browser/OS support, offline behavior, availability, accessibility, performance, scale, retention, encryption, backup, and recovery targets before production deployment.

## 7. Advantages, Disadvantages, and Architectural Risks

### Advantages

- Split, approval, and balance rules are centralized and consistent across clients.
- Domain and use-case tests can run without a GUI or concrete database.
- Persistence, identity, receipt, and notification technologies can change behind ports.
- Approval gates balance effects, and balances can be reconciled against retained expense/payment history.
- Swing and future web/mobile clients can use the same business behavior without presenting Swing as the complete product.

### Disadvantages

- Ports, adapters, mappings, and module boundaries add effort that may be heavy for a small prototype.
- The architecture does not itself deliver web, Android, or iOS clients; each remains a substantial workstream.
- Authentication and authorization policies remain unresolved even if an identity port is defined.
- Report queries may need read models distinct from the write-side domain model.
- Ports alone do not guarantee atomic consistency between approval state, balance effects, and notification delivery.

### Risks and mitigations

| Risk | Impact | Mitigation / decision required |
| --- | --- | --- |
| Currency scale or remainder ties are undefined | Different clients/runs could allocate cents differently. | Specify currencies, scale, percentage precision, and deterministic tie-break; verify boundary cases with JUnit 5. |
| Approval policy is incomplete | A disallowed reviewer could change balances. | Define reviewer eligibility, levels, self-approval, rejection, edits, and resubmission before workflow implementation. |
| Payment correction and overpayment rules are incomplete | Recorded balances may differ from settlements. | Define reversals, corrections, disputes, and overpayments; preserve history or use compensating entries. |
| Adapter authorization is incomplete | Personal, receipt, or group data may be exposed. | Enforce authorization in use cases and test each inbound adapter against access rules. |
| Persistence and notification can fail independently | A notification may describe a missing expense, or an approved expense may lack its balance effect. | Define transaction/retry semantics for selected storage; add durable delivery coordination only if justified. |
| Stored balance projection drifts from history | Users may see incorrect amounts owed. | Derive balances initially or add reconciliation if maintaining a projection; make effect application idempotent. |
| No quality targets exist | Deployment could be inadequate or overbuilt. | Agree performance, availability, accessibility, backup, recovery, capacity, and privacy objectives before production. |
| Source code is absent in this checkout | Proposal may differ from code maintained elsewhere. | Compare with source/build artifacts when available; keep this architecture labeled proposed until verified. |

## 8. Requirements Traceability

| Requirement | Architecture response |
| --- | --- |
| Accounts (FR-001) | Client inbound ports plus identity/access boundary and account queries |
| Groups (FR-002) | Group/Membership model and repository port; manager policy enforced by use cases |
| Expense details/receipts (FR-003) | Expense rules in core; optional receipt reference through storage port |
| Participants/splits (FR-004) | ExpenseShare and SplitAllocator in core |
| Notifications (FR-005) | NotificationPort called at expense submission |
| Approval (FR-006) | Approval use case and policy/decision concepts gate balance effects |
| Balances (FR-007) | BalanceCalculator uses approved shares and payment history |
| Payments (FR-008) | RecordPayment and PaymentRepository retain full/partial settlements |
| Reporting (FR-009) | QueryReports port and read adapter enforce authorization and date ranges |
| Platforms (FR-010) | Web, Android, and iOS adapters required; Swing is only documented MVP direction |

## 9. Proposed Versus Implemented

No application source, package structure, build manifest, database, or executable was available in the inspected repository. Every class, port, adapter, interaction, and deployment option in this document is proposed. Java 21, Maven, Swing, BigDecimal, JUnit 5, and PlantUML are documented constraints/directions, not proof of implementation. Storage products, web/mobile frameworks, API protocols, identity systems, and deployment platforms remain unverified and unselected. No application source code was changed.

