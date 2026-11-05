# SplitSmart Conceptual Data Model

**Status**: Proposed model derived from requirements; not a description of existing classes or database tables. See [architecture.md](architecture.md) for UML and component boundaries.

## Entities

| Entity | Conceptual fields | Relationships |
| --- | --- | --- |
| Account | Identifier, email/credential reference, account status | Has memberships; submits expenses; can be payer, participant, reviewer, notification recipient, or payment party |
| Group | Identifier, name, created date, status | Has memberships, expenses, and related settlement context |
| Membership | Account, group, role, status, joined date | Joins one account to one group; manager role controls documented membership changes |
| Expense | Identifier, group, submitter, payer, amount, currency, date, description, split method, status | Has participant shares, approval decisions, optional receipt, and notifications |
| ExpenseShare | Expense, participant account, percentage (for percentage split), allocated amount | Belongs to one expense and one participant |
| ApprovalDecision | Expense, reviewer, decision, approval level, decision time | Belongs to one expense; reviewer is an account |
| Receipt | Expense, image reference/metadata | Optional association to one expense; storage and retention unspecified |
| Notification | Recipient account, expense reference, message, created/read times | Informs a group member about an expense submission |
| Payment | Identifier, payer account, payee account, amount, currency, date, context | Retained financial history; reduces the related bilateral debt according to policy |
| Balance | Debtor, creditor, currency, amount/as-of time | Derived from approved shares and payments or maintained as a reconcilable projection |

## Business Rules and Validation

- Expense amount is strictly positive.
- An expense has one payer and one or more participants; payer can be outside participant set.
- Split method is equal or percentage. Percentage values total exactly 100%.
- Allocated amounts reconcile to the expense amount in the currency's smallest unit using largest remainder. Currency scale and deterministic tie-breaking remain open.
- Pending or rejected expenses have no balance effect. Apply effects only after the required approval workflow completes.
- A payer who is also a participant does not owe themselves.
- Partial payments are retained and reduce only the recorded amount from an outstanding bilateral balance.
- Payment reversal, overpayment, correction, and dispute behavior is not specified.
- Personal, group, receipt, and payment information must be visible only to authorized users.

## Expense Lifecycle

```text
Submitted -> Pending approval -> Approved -> Balance effect applied
                             \-> Rejected -> No balance effect
```

The requirements do not define edits, deletion, correction, or resubmission after rejection/approval. The exact state model and audit behavior require a product decision.

## Storage Notes

No database or receipt storage technology is selected. The model is technology-neutral. A future implementation must preserve expense and payment history and ensure balance calculations can be reconciled. Whether `Balance` is persisted or computed is an implementation decision.
