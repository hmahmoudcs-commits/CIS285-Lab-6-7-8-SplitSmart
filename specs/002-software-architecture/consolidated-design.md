# SplitSmart Consolidated Design

**Selected architecture**: Architecture B (modular hexagonal), with Architecture A's clear domain model.  
**Status**: Proposed UML/design; no source was available to validate an as-built structure.

## Conceptual Domain Model

- **Account** belongs to groups through Membership; may submit expenses, pay, review, receive notifications, or make/receive payments.
- **Group** contains memberships and expenses.
- **Expense** has one payer, submitter, amount/date/description, split method, status, shares, decisions, and optional receipt.
- **ExpenseShare** links expense to participant and percentage/allocated amount.
- **ApprovalDecision** records reviewer, decision, and level; policy details remain open.
- **Payment** retains payer, payee, amount, date, and settlement history.
- **Balance** is derived or a reconcilable projection from approved shares and payment history.
- **Money** encapsulates exact decimal value and currency. Currency precision is unresolved.

## UML Class Diagram (PlantUML)

Inbound/outbound ports belong to the application core. Adapters depend on those ports; the domain has no adapter dependency.

~~~plantuml
@startuml
title SplitSmart consolidated hexagonal design
skinparam classAttributeIconSize 0
left to right direction
package "Client Adapters" {
  class WebAdapter <<inbound adapter>>
  class AndroidAdapter <<inbound adapter>>
  class IOSAdapter <<inbound adapter>>
  class SwingAdapter <<inbound adapter>>
}
package "Application Core" {
  interface SubmitExpense <<inbound port>>
  interface ReviewExpense <<inbound port>>
  interface RecordPayment <<inbound port>>
  interface QueryReport <<inbound port>>
  class ExpenseUseCase
  class ApprovalUseCase
  class PaymentUseCase
  class ReportingUseCase
  interface ExpenseRepository <<outbound port>>
  interface GroupRepository <<outbound port>>
  interface PaymentRepository <<outbound port>>
  interface IdentityAccessPort <<outbound port>>
  interface NotificationPort <<outbound port>>
  interface ReceiptStoragePort <<outbound port>>
  interface ReportReadPort <<outbound port>>
}
package "Domain" {
  class Account
  class Group
  class Membership
  class Expense
  class ExpenseShare
  class ApprovalDecision
  class Payment
  class Money <<value object>>
  class SplitAllocator <<domain service>>
  class BalanceCalculator <<domain service>>
}
package "Infrastructure Adapters" {
  class ExpenseRepositoryAdapter <<outbound adapter>>
  class GroupRepositoryAdapter <<outbound adapter>>
  class PaymentRepositoryAdapter <<outbound adapter>>
  class IdentityAdapter <<outbound adapter>>
  class InAppNotificationAdapter <<outbound adapter>>
  class ReceiptStorageAdapter <<outbound adapter>>
  class ReportQueryAdapter <<outbound adapter>>
}
WebAdapter ..> SubmitExpense
WebAdapter ..> ReviewExpense
WebAdapter ..> RecordPayment
WebAdapter ..> QueryReport
AndroidAdapter ..> SubmitExpense
AndroidAdapter ..> ReviewExpense
AndroidAdapter ..> RecordPayment
IOSAdapter ..> SubmitExpense
IOSAdapter ..> ReviewExpense
IOSAdapter ..> RecordPayment
SwingAdapter ..> SubmitExpense
SwingAdapter ..> ReviewExpense
SwingAdapter ..> RecordPayment
SubmitExpense <|.. ExpenseUseCase
ReviewExpense <|.. ApprovalUseCase
RecordPayment <|.. PaymentUseCase
QueryReport <|.. ReportingUseCase
ExpenseUseCase ..> ExpenseRepository
ExpenseUseCase ..> GroupRepository
ExpenseUseCase ..> IdentityAccessPort
ExpenseUseCase ..> NotificationPort
ExpenseUseCase ..> ReceiptStoragePort
ExpenseUseCase ..> SplitAllocator
ApprovalUseCase ..> ExpenseRepository
ApprovalUseCase ..> IdentityAccessPort
ApprovalUseCase ..> BalanceCalculator
PaymentUseCase ..> PaymentRepository
PaymentUseCase ..> IdentityAccessPort
ReportingUseCase ..> ReportReadPort
ExpenseRepository <|.. ExpenseRepositoryAdapter
GroupRepository <|.. GroupRepositoryAdapter
PaymentRepository <|.. PaymentRepositoryAdapter
IdentityAccessPort <|.. IdentityAdapter
NotificationPort <|.. InAppNotificationAdapter
ReceiptStoragePort <|.. ReceiptStorageAdapter
ReportReadPort <|.. ReportQueryAdapter
Account "1" -- "0..*" Membership
Group "1" -- "1..*" Membership
Group "1" -- "0..*" Expense
Expense "1" *-- "1..*" ExpenseShare
ExpenseShare "*" --> "1" Account : participant
Expense "*" --> "1" Account : payer
Expense "1" *-- "0..*" ApprovalDecision
ApprovalDecision "*" --> "1" Account : reviewer
Expense --> Money
ExpenseShare --> Money
Payment --> Money
BalanceCalculator ..> Expense
BalanceCalculator ..> Payment
@enduml
~~~

## UML Activity Diagram (PlantUML)

Notification is produced at submission; balance effects occur only after all required approval.

~~~plantuml
@startuml
title SplitSmart expense submission, approval, and balance effect
start
|Client Adapter|
:Collect expense details, payer, participants, split, optional receipt;
:Invoke SubmitExpense;
|Application Core|
:Authorize actor and group access;
if (Authorized?) then (yes)
  |Domain|
  :Validate amount, payer, participants, and split;
  if (Valid?) then (yes)
    :Calculate shares using exact money rules;
    :Mark expense pending;
    |Application Core|
    :Persist pending expense and shares;
    :Notify other group members in-app;
    |Client Adapter|
    :Display pending status;
    :Reviewer invokes ReviewExpense;
    |Application Core|
    :Authorize reviewer and record decision;
    while (More required approval?) is (yes)
      :Keep pending and collect next decision;
    endwhile (no)
    if (Approved?) then (yes)
      |Domain|
      :Calculate bilateral balance effect;
      :Do not create self-owed amount;
      |Application Core|
      :Persist approved state and effect once;
      |Client Adapter|
      :Display approved status and balance;
    else (rejected)
      |Application Core|
      :Record rejection; leave balances unchanged;
      |Client Adapter|
      :Display rejected status;
    endif
  else (invalid)
    |Client Adapter|
    :Display validation errors; do not save;
  endif
else (no)
  |Client Adapter|
  :Deny request;
endif
stop
@enduml
~~~

## UML Component Diagram (PlantUML)

~~~plantuml
@startuml
title SplitSmart consolidated component architecture
left to right direction
actor User
component "Web Client" as Web
component "Android Client" as Android
component "iOS Client" as IOS
component "Swing Desktop MVP" as Swing
component "Inbound API/UI Adapters" as Inbound
component "Application Use Cases\nExpense / Approval / Payment / Reports" as App
component "Domain Model and Services\nAccounts / Groups / Expenses / Splits / Balances" as Domain
component "Repository Adapters" as Repos
component "Identity and Access Adapter" as Identity
component "In-App Notification Adapter" as Notify
component "Receipt Storage Adapter" as Receipt
component "Report Query Adapter" as Reports
database "Persistent Store\ntechnology TBD" as Store
User --> Web
User --> Android
User --> IOS
User --> Swing
Web --> Inbound
Android --> Inbound
IOS --> Inbound
Swing --> Inbound
Inbound --> App : inbound ports
App --> Domain
App --> Repos : repository ports
App --> Identity : identity/access port
App --> Notify : notification port
App --> Receipt : receipt port
App --> Reports : report query port
Repos --> Store
Reports --> Store
Notify --> Store
@enduml
~~~

## Data and State Rules

- Expense lifecycle: submitted/pending approval → approved → balance effect; rejected expenses do not affect balances.
- Only approved expense shares contribute to bilateral amounts owed.
- Payment records are retained; only the recorded payment amount affects the outstanding balance.
- Use exact decimal arithmetic; Java uses BigDecimal. Currency scale and remainder tie-break require a separate decision.
- Receipt is optional and associated with an expense; storage/retention are unspecified.

