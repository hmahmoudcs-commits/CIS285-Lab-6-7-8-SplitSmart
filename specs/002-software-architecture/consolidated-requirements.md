# SplitSmart Consolidated Requirements

**Purpose**: Product and quality requirements for CIS 285 Lab 7 using selected Architecture B.  
**Authority**: [UserRequirement.md](../../UserRequirement.md), refined by [spec.md](spec.md).  
**Status**: Requirements do not prove implementation. No application source or build manifest was found.

## Product Scope

The product supports expense sharing for individuals and groups. The customer requires a web-first product and Android and iOS apps. The constitution separately documents a Java Swing desktop MVP direction; it does not replace the required product platforms.

## Functional Requirements

- **FR-001 Accounts**: Each person has an account, can log in, and can access their own expenses and balances. The documented refinement specifies self-service email/password credentials.
- **FR-002 Groups**: Users can create and manage groups for shared expenses/balances. Documented refinement: invitation-only membership managed by group managers.
- **FR-003 Expense details/receipts**: Users can submit an expense with amount, date, description, and optional receipt image. Amount is strictly positive.
- **FR-004 Participants/allocation**: Each expense identifies involved users, one payer, and equal or percentage allocation. Payer may be outside the participant set. Percentages total exactly 100%. Largest-remainder allocation reconciles to the currency's smallest unit; currency scale and deterministic ties remain open.
- **FR-005 Notifications**: Every other group member receives an in-app notification when an expense is submitted, before approval completes.
- **FR-006 Approval**: Expenses are reviewed through a single- or multi-level workflow before affecting balances. Pending, incomplete, and rejected expenses do not affect balances.
- **FR-007 Bilateral balances**: The system tracks what each user owes every other user from approved expenses and recorded payments. Users do not owe themselves.
- **FR-008 Payments**: Users record settlement payments. Full and partial payments are supported by the documented refinement, and payment history is retained.
- **FR-009 Reports**: Users view expense, balance, and payment reports/summaries, including custom date ranges and other parameters.
- **FR-010 Platforms**: The product is primarily web-based and also available on Android and iOS. Swing alone does not satisfy this requirement.

## Nonfunctional Requirements

- **NFR-001 Monetary accuracy**: Monetary values, allocations, and balances use exact decimal arithmetic. Java components use BigDecimal, not binary floating point. Currency scale, percentage precision, and deterministic remainder ties must be specified.
- **NFR-002 Access protection**: Personal, group, receipt, expense, payment, and report information is available only to authenticated and authorized users. Detailed group visibility/roles require definition.
- **NFR-003 Separation/testability**: Presentation, business rules, and persistence are separate. Domain and use-case behavior can be tested independently of GUI components.
- **NFR-004 Cross-client consistency**: Allocation, approval, balance, and payment rules produce consistent results for web, Android, iOS, and any Swing prototype.
- **NFR-005 Data integrity**: Approved expense/payment history remains reliable; balance effects are neither lost nor applied twice.
- **NFR-006 Verification/traceability**: Java tests use JUnit 5; maintained diagrams use PlantUML where useful. Design and verification evidence traces to requirements or explicit decisions.
- **NFR-007 Privacy/retention**: Data access is protected. Receipt retention, backup/recovery, encryption, audit, and availability targets are unspecified.
- **NFR-008 Performance/accessibility**: Performance, capacity, accessibility, and service availability targets must be agreed before production acceptance.

## System and Engineering Constraints

- Java components use Java 21 and Maven.
- Java Swing is the documented desktop MVP GUI direction only.
- Domain models are cohesive and object-oriented, independent of UI widgets.
- Java money calculations use BigDecimal; Java tests use JUnit 5; UML uses PlantUML.
- Web, Android, and iOS remain required. Frameworks and minimum OS/browser versions are unspecified.
- The repository has no source/build files, so actual runtime language/framework usage cannot be verified.
- Storage, identity provider, API protocol, deployment topology, and receipt storage are unselected.

## Outstanding Questions

1. Accounts: email verification, credential recovery, account closure, and session policy.
2. Groups: invitation lifecycle, manager permissions, invitee behavior, and member removal with outstanding balances.
3. Currency/allocation: supported currencies, precision, percentage input precision, and deterministic remainder tie-break.
4. Expense lifecycle: edit/delete after submission/approval and correction history.
5. Approval: reviewer selection, levels, self-approval, rejection, resubmission, and post-rejection edits.
6. Payments: manual vs provider integration; overpayment, correction, reversal, disputes, and evidence.
7. Receipts: formats, size limits, authorization, storage, and retention.
8. Reports: filters, grouping, exports, and time-zone boundaries.
9. Platforms: browser/OS support, distribution, offline expectations, and parity.
10. Quality: performance, availability, capacity, accessibility, security, backup, recovery, and retention targets.
11. Architecture: server/API topology, store, identity, non-Java client stacks, and hosting.

These are open decisions, not implemented features.

## Traceability

| Requirement | Main architecture element |
| --- | --- |
| FR-001 | Account use cases; identity/access port |
| FR-002 | Group/Membership domain and repository port |
| FR-003 | Expense domain; optional receipt port |
| FR-004 | ExpenseShare and SplitAllocator |
| FR-005 | Notification outbound port/adapter |
| FR-006 | Approval use case and domain policy |
| FR-007 | Balance calculation from approved shares/payments |
| FR-008 | Payment use case/repository |
| FR-009 | Authorized report query port |
| FR-010 | Web, Android, iOS adapters; Swing MVP separately |

