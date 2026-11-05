# Implementation Plan: SplitSmart Base Software Architecture

**Branch**: `002-software-architecture` | **Date**: 2026-11-05 | **Spec**: [spec.md](spec.md)

**Input**: Feature specification from `specs/002-software-architecture/spec.md`

**Status note**: This plan describes a proposed logical architecture for a design assignment. Repository inspection found no application source, build manifest, or tests, so it does not describe or claim an existing implementation.

## Summary

Define a base, layered architecture for the documented SplitSmart product. Required clients (web, Android, iOS; plus the documented Swing MVP direction) invoke application use cases. A domain layer owns group, expense, split, approval, balance, and payment rules. Persistence, notification delivery, and reporting are separate boundaries. The first implementation could use Java 21/Maven/Swing for the documented desktop prototype, with `BigDecimal` for money and JUnit 5 for Java tests. A service topology, database, APIs, and web/mobile stacks are intentionally undecided.

## Technical Context

**Language/Version**: Java 21 for Java components (documented constraint); actual source language is unverified. Web/mobile languages are unspecified.

**Primary Dependencies**: Maven, Java Swing for the documented desktop MVP direction, JUnit 5 for Java tests, PlantUML for diagrams. These are documented constraints, not observed dependencies. Web/mobile frameworks and application dependencies are unspecified.

**Storage**: Persistent storage is required by product behavior; database and file/object storage technologies are unspecified. Logical repository boundaries are proposed.

**Testing**: JUnit 5 for Java components. No test sources or configured test command are present in the repository. The current design artifacts can be reviewed against the acceptance scenarios; executable application validation requires implementation.

**Target Platform**: Customer scope: web-first, Android, and iOS. Documented prototype direction: desktop Java Swing. Browser and OS versions are unspecified.

**Project Type**: Multiplatform expense-sharing product; documented Java desktop MVP direction. The checked-in project is currently documentation-only.

**Performance Goals**: Unspecified by customer/project documents.

**Constraints**: Preserve all customer requirements. Keep presentation separate from business and persistence rules. Use Java 21/Maven for Java components, Swing for the desktop MVP direction, `BigDecimal` for Java money calculations, JUnit 5 for Java tests, and PlantUML for maintained diagrams. Swing alone does not fulfill web/Android/iOS scope. Do not present proposals as implemented features.

**Scale/Scope**: Requirements cover accounts, groups, expenses, approvals, notifications, bilateral balances, payments, reports, and web/Android/iOS availability. User volume, data volume, and deployment scale are unspecified.

## Constitution Check

| Gate | Result | Evidence / handling |
| --- | --- | --- |
| Preserve customer baseline | PASS | All ten customer feature areas remain in scope and trace to the spec. |
| Java and build constraints | PASS | Java 21 and Maven are retained for Java components; no source or build configuration is claimed to exist. |
| Object-oriented domain | PASS | Proposed domain entities encapsulate expense, split, approval, payment, and balance concepts. |
| Presentation/domain/persistence separation | PASS | Clients, application use cases, domain rules, and persistence are separate logical components. |
| Monetary correctness | PASS | Java money values use `BigDecimal`; currency precision and tie-break policy remain explicit prerequisites. |
| Verification and diagrams | PASS | JUnit 5 and PlantUML are planned for Java verification and maintained UML. |
| Traceability and human review | PASS | Design elements map to specification requirements; artifacts are proposals requiring review. |
| Platform completeness | PASS WITH OPEN DESIGN | Web, Android, and iOS remain required. Their client stacks and integration topology are unresolved; Swing is not treated as satisfying them. |

No constitution violation is proposed. Technical choices that cannot be supported by repository evidence are left open rather than invented.

## Phase 0: Research and Decisions

See [research.md](research.md). The current design resolves logical responsibility boundaries only. It deliberately leaves deployment topology, storage product, API protocol, and non-Java client technologies for later evidence and stakeholder decisions.

## Phase 1: Architecture and Design

- [architecture.md](architecture.md): proposed components, design decisions, data flows, UML class diagram, and UML activity diagram.
- [data-model.md](data-model.md): conceptual entities, relationships, validation rules, and lifecycle.
- [contracts/application-boundary.md](contracts/application-boundary.md): technology-neutral proposed use-case contract; no network API or serialization format is selected.
- [quickstart.md](quickstart.md): review-based design validation scenarios and implementation validation prerequisites.

## Project Structure

### Existing Repository Structure (observed)

```text
UserRequirement.md
requirements.md
.specify/memory/constitution.md
specs/001-splitsmart/spec.md
specs/002-software-architecture/
  spec.md
  plan.md
```

No application source, build manifest, or tests were found in the working tree at planning time.

### Proposed Logical Source Organization (not present in repository)

```text
clients/
  web/                    # required product client; technology TBD
  android/                # required product client; technology TBD
  ios/                    # required product client; technology TBD
  desktop-swing/           # documented MVP direction if retained
application/               # use cases, authorization and orchestration
domain/                     # accounts, groups, expenses, approvals, splits, payments
adapters/
  persistence/              # storage adapter; technology TBD
  notifications/            # in-app notification adapter
  reporting/                # report queries/read models
tests/
  unit/                     # domain and application behavior
  integration/              # selected adapter boundaries
```

**Structure Decision**: This is a responsibility map, not a committed filesystem layout. A later implementation plan should select a repository structure after the full product topology and client technologies are decided. For a Java desktop MVP, Java presentation should remain outside domain rules, with Maven-managed Java 21 sources and JUnit 5 tests as required by the constitution.

## Data Flow Summary

1. A client submits an expense through an application use case. The application checks identity and group access, and domain rules validate amount, payer, participants, split, and receipt metadata.
2. The pending expense and its shares are persisted. The notification boundary makes an in-app notification available to the other group members. Notification timing at submission follows the documented requirement; delivery failure and retry behavior need a later operational decision.
3. A reviewer records a decision. The approval workflow checks whether all required levels have approved. Pending or rejected expenses produce no balance effect.
4. When approval completes, the domain calculates allocations and bilateral effects using exact decimal arithmetic. Effects are persisted once. A future design must decide whether balances are derived from approved expenses/payments or maintained as a verified projection.
5. A user records a payment. The application validates the payer/payee and amount against the chosen policy, retains payment history, and updates or derives the outstanding bilateral amount.
6. Reporting queries authorized expense, approval, allocation, payment, and balance data for a requested date range and returns summaries to the client.

Detailed component responsibilities and diagrams are in [architecture.md](architecture.md).

## Complexity Tracking

No constitution violations requiring justification.
