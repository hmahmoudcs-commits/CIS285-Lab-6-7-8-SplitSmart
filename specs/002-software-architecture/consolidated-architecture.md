# SplitSmart Consolidated Architecture

**Selected approach**: Architecture B, modular ports-and-adapters (hexagonal), retaining Architecture A's clear domain model.  
**Status**: Proposed design, not an as-built description.  
**Requirements**: [consolidated-requirements.md](consolidated-requirements.md).

## Overview

The system centers on a framework-independent domain and application core. The core owns use cases and inbound/outbound port interfaces. Inbound adapters translate product-client actions into use-case calls; outbound adapters implement identity/access, repositories, receipt storage, notifications, and report queries.

Dependencies point inward. Domain rules do not depend on UI, frameworks, persistence products, or notification vendors. The proposed initial deployment is a modular monolith, preserving module boundaries without distributed transactions for approval and balance changes. This is a proposal; no code verifies this topology.

## Components

### Domain model

Account, Group, Membership, Expense, ExpenseShare, ApprovalDecision/Policy, Payment, and Money concepts. Domain services calculate splits and bilateral balances. Invariants include positive expense amounts, valid split percentages, and approved-only balance effects.

### Application core

- **Inbound ports**: account/group workflows, submit expense, review expense, record payment, query balances, and request reports.
- **Use cases**: authorize the actor, load domain state, invoke rules, persist state transitions, and return results.
- **Outbound ports**: repositories, identity/access, receipt storage, notification delivery, and report reads.
- Pending/rejected expenses do not affect balances. Apply an expense effect only after full configured approval.

### Inbound adapters

- Web adapter: required primary client connection; framework/protocol open.
- Android and iOS adapters: required clients using the same business use cases.
- Swing adapter: documented Java desktop MVP direction; it does not replace web/Android/iOS.
- Adapters handle presentation and mapping, not authoritative allocation, approval, or balance logic.

### Outbound adapters

- Persistence adapters implement repository ports; storage technology is unselected.
- Identity/access adapter supplies actor context and authorization; provider/session mechanism is unselected.
- Receipt adapter stores/retrieves optional images; format, size, and retention remain open.
- Notification adapter makes in-app notifications available to other group members at submission. Push/email/SMS are not implied.
- Report query adapter returns authorized expense, balance, and payment summaries for requested ranges and parameters.

## Component Interactions and Data Flows

### Expense submission and notification

1. Client invokes the submit-expense inbound port.
2. Application checks actor access to the group.
3. Domain validates positive amount, payer, participants, and split. Percentages total exactly 100%; equal/percentage shares use largest-remainder reconciliation. Currency precision and deterministic ties must be specified before implementation.
4. Application stores expense and shares as pending.
5. Application calls notification port. Other group members can see an in-app notification at submission, before approval.
6. Client receives pending status. No balance changes yet.

### Review and balance effect

1. Reviewer invokes review-expense; application checks authorization and records the decision.
2. If approval levels remain, expense stays pending. If rejected, it remains excluded from balances.
3. Once all required approval completes, domain logic calculates bilateral effects from payer and participant shares.
4. Application persists approved state and effect consistently. Payer may be outside participants; a payer who is a participant does not owe themselves.
5. Balance may be derived from approved expense/payment history or maintained as a reconcilable projection. The choice is open; effects must not apply twice.

### Settlement and reports

- Payment use case authorizes the payer/payee context, retains full/partial payment history, and incorporates only the recorded amount. Reversal/overpayment policies are open.
- Report queries enforce authorization/date ranges and return summaries without changing financial state.
- Receipt data crosses a storage port; access is governed by authorization for its expense.

## Technology and Deployment

- Java components use Java 21 and Maven.
- Documented desktop MVP uses Java Swing separated from domain/application rules.
- Java money uses BigDecimal; tests use JUnit 5; diagrams use PlantUML.
- Product platforms are web-first plus Android and iOS. Frameworks, protocol, and supported versions are undecided.
- Proposed deployment is a modular monolith. Versioned HTTP/JSON is one candidate for client connections, not a confirmed choice.
- Select data store, receipt storage, identity, hosting, and notification delivery after quality/privacy/operational requirements are established.
- Do not invent performance, availability, capacity, accessibility, backup, recovery, retention, or encryption targets.

## Proposed Versus Implemented

The repository has no application source or build manifest. Components, ports, and flows here are proposals. Documented technology constraints do not prove a working application uses them.

