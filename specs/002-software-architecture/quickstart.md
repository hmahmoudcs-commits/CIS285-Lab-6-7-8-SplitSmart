# Architecture Design Validation Guide

**Status**: Review guide for the proposed design. The repository has no application source, build manifest, or test suite, so end-to-end software execution is not currently available from this checkout.

## Prerequisites

- Read [spec.md](spec.md), [plan.md](plan.md), [architecture.md](architecture.md), [data-model.md](data-model.md), and [application-boundary.md](contracts/application-boundary.md).
- PlantUML renderer is optional for viewing the embedded class and activity diagrams. The project constitution names PlantUML as the intended diagram format but no renderer configuration is checked in.

## Review Scenarios

1. **Requirement coverage**: Trace FR-001 through FR-010 from the spec into the architecture component table and diagrams. Confirm web, Android, and iOS remain required, and Swing is labeled only as documented MVP direction.
2. **Pending expense**: Walk the activity diagram from submission through pending approval. Confirm a pending expense emits the documented notification but has no balance effect.
3. **Approved expense**: Continue through completion of required approvals. Confirm shares are reconciled, payer and participants determine bilateral balances, and no self-owed balance is created.
4. **Rejected expense**: Follow rejection. Confirm the expense is retained as rejected and balances remain unchanged.
5. **Partial payment**: Trace a payment through Payment and Balance. Confirm the history retains the payment and only its recorded amount is applied.
6. **Access control**: Check account, group, notification, receipt, and report reads. Confirm each is scoped to the requesting actor's authorization.
7. **Unknown implementation details**: Verify database, APIs, frameworks, deployment, OS minimums, and quality targets remain unspecified rather than stated as deployed choices.

## Executable Validation Status

No commands such as `mvn test` or application launch can be given as a valid project procedure because `pom.xml`, source, and test files are absent. Once an implementation is added, provide its actual build/run commands and use JUnit 5 tests for Java domain calculations and workflow behavior, as required by the constitution.
