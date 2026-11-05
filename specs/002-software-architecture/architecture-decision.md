# Architecture Decision Record: Select Modular Hexagonal Architecture

**Status**: Accepted for the CIS 285 Lab 7 design package  
**Date**: 2026-11-05  
**Decision**: Select Architecture B (Codex modular hexagonal / ports-and-adapters), using Architecture A's clear domain model.

## Context

Architecture A is a simpler layered logical design. Architecture B separates the domain/application core from inbound and outbound adapters using explicit ports. The ChatGPT review recommends B because it makes client and infrastructure boundaries explicit and isolates expense rules. No application source was found; both are proposals, not descriptions of implemented software.

SplitSmart requirements span web, Android, iOS, and documented Java Swing MVP direction. Allocation, approval, balance, and payment rules should remain consistent across clients. The constitution requires Java 21/Maven for Java components, object-oriented models independent of UI widgets, exact money with BigDecimal, JUnit 5, and PlantUML.

## Decision

- Adopt modular hexagonal architecture as the final design direction.
- Keep Account, Group, Membership, Expense, ExpenseShare, ApprovalDecision, Payment, and Money as clear domain concepts.
- Application use cases expose inbound ports and coordinate workflows.
- Outbound ports express persistence, identity/access, receipt, notification, and reporting needs.
- Web, Android, iOS, and Swing are inbound adapters; storage/service integrations are outbound adapters.
- Begin with a proposed modular monolith to preserve a financial consistency boundary. Defer microservices and vendor/protocol choices until justified.

## Reasons

1. Split, approval, and balance behavior must serve multiple required clients.
2. Domain/use-case behavior can be tested without GUI widgets or a selected database.
3. Persistence, identity, receipt, notification, and reporting technologies can change behind core-owned ports.
4. Explicit approval and payment boundaries make financial transitions easier to reason about and review.
5. The choice supports separation, traceability, exact-money, JUnit 5, and PlantUML principles while preserving customer platform scope.

## Trade-offs and Consequences

- **Accepted cost**: More interfaces, adapters, and mapping than Architecture A, accepted for clearer client independence and test seams.
- **Not an implementation claim**: No source was found; this decision does not assert deployment.
- **Platform work remains**: Choosing B does not implement web, Android, or iOS. Swing alone does not satisfy those requirements.
- **Technology choices remain open**: Client frameworks, API protocol, persistence, identity, notification mechanism, receipt storage, and hosting require separate decisions.
- **Quality targets remain open**: Performance, availability, scale, accessibility, retention, backup, recovery, and security thresholds need agreement.
- **Domain policy remains open**: Currency/rounding tie-break, approval roles, expense correction, payment reversal/overpayment, and report boundary rules need decisions before production.
- **Human review required**: This design proposal needs review for correctness, security, privacy, and fit.

## Alternatives Considered

### Architecture A: Layered logical design

**Benefit**: Simpler to explain and likely lower initial structural overhead; its domain concepts are retained.  
**Reason not selected overall**: It defines less explicit port contracts between application behavior and clients/infrastructure.

### Distributed microservices

**Benefit**: Could support independent deployment/scaling of bounded capabilities.  
**Reason deferred**: No measured scale, team topology, or independent release requirement exists. Distributed financial consistency adds complexity without current evidence.

## References

- [UserRequirement.md](../../UserRequirement.md)
- [Consolidated requirements](consolidated-requirements.md)
- [Consolidated architecture](consolidated-architecture.md)
- [Consolidated design](consolidated-design.md)
- [Architecture review](Architecture_Review.md)
- [Original layered proposal](architecture.md)
- [Original hexagonal proposal](alternative-architecture.md)

