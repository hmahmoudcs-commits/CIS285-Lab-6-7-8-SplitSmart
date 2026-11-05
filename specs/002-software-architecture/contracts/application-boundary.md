# Proposed Application Boundary Contract

**Status**: Logical use-case contract for architecture planning only. This is not a deployed or public API. No protocol, endpoint paths, serialization format, or authentication mechanism is selected.

## Responsibilities

The application boundary accepts authenticated actor context, checks authorization for the requested group/account data, validates command shape, invokes domain rules, persists accepted state transitions, and returns a result suitable for a client. Clients are responsible for presentation and input collection; they are not authoritative for split, approval, or balance calculations.

## Proposed Use Cases

| Use case | Input concept | Result concept | Key rule |
| --- | --- | --- | --- |
| Create account / sign in | Credential data | Account/session result | Credentials and account recovery policy remain unspecified beyond documented email/password self-service refinement. |
| Create/manage group | Actor, group details, membership invitation/change | Group or membership result | Membership changes follow manager/invitation rules. |
| Submit expense | Actor, group, payer, participants, amount, date, description, split, optional receipt | Pending expense and notification result | Validate positive amount, participants, percentage total, authorization; do not affect balance. |
| Record approval | Actor, expense, decision | Updated workflow state | Enforce configured approval levels; policy details remain open. |
| View balances | Actor, group/account scope | Authorized bilateral balances | Include approved expenses and recorded settlements only. |
| Record payment | Actor, payer/payee, amount, date, context | Retained payment and updated/derivable balance | Partial amounts allowed; correction/overpayment policy remains open. |
| Request report | Actor, scope, date range, parameters | Expense, balance, or payment summary | Enforce authorization and date boundaries; extra filters/export formats remain open. |
| Read notifications | Actor, notification scope | Authorized in-app notifications | Notifications are made available to other group members when expense is submitted. |

## Common Outcome Categories

- Accepted: operation completed and returns the resulting domain identifier/state/summary.
- Validation rejected: input violates a documented rule, such as nonpositive amount or percentage shares not totaling 100%.
- Unauthorized: actor is not signed in or lacks access to the requested account/group data.
- Conflict: requested transition is inconsistent with current state, such as trying to apply a decision to an ineligible expense. Exact conflict policy and retry semantics require implementation design.
- Temporarily unavailable: persistence or notification boundary could not complete the operation. Atomicity, retry, and user-visible recovery behavior are unspecified.

## Contract Decisions Deferred

Transport (REST, GraphQL, or another option), request/response schema, versioning, pagination, identity/session mechanism, idempotency keys, error codes, and delivery guarantees must be selected after product topology and operational requirements are known. These are not present implementation features.
