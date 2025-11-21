# 📘 Architecture Documentation
### Transactional Script + DB-Centric Domain Logic + SQL Modularization

This project follows a database-centric enterprise architecture using:

Transactional Script (Application Layer)
DB-Centric Domain Logic (Business Rules in SQL)
SQL Modularization (Layered Stored Procedures)

## 1. 🔵 Transactional Script (Application Layer Pattern)
Each business use case is executed as one sequential procedure (method or stored procedure).

✔ Characteristics
One use case = One function / One stored procedure
Logic flows top-down (step → step → step)
Very easy to understand
Ideal for CRUD + procedural logic
No DDD entities/aggregates/models

Notes
The stored procedure contains the entire workflow.
This refers to the pattern, not the implementation location.

## 2. 🔴 DB-Centric Domain Logic (Business Rules in SQL)

All core business rules are implemented inside the SQL database rather than C#.

✔ Benefits

Multiple systems can reuse business rules

SQL guarantees consistency and ACID

Optimized for data-heavy operations

Application layer becomes thin (orchestration only)

❌ Not DDD

DDD places logic in entities/value objects inside code.
Here, SQL = domain layer → classic enterprise approach.

## 3. 🟢 SQL Modularization (Structured SQL Architecture)

SQL is organized into 4 layers to avoid spaghetti logic:

Common SPs

Shared Business SPs

Use-Case SPs (Transactional Scripts)

UI SPs (Read-only Query SPs)

# 🔍 Deep Dive into SQL Modularization Architecture
## 1️⃣ 🟦 Common SPs — “Technical Layer”
Purpose

Reusable technical CRUD operations

No business rules

Examples

usp_Common_GetCustomerById

usp_Common_GetBalance

usp_Common_InsertAuditLog

usp_Common_UpdateStatus

usp_Common_InsertLedgerEntry

Allowed Calls
Caller	Allowed?	Why
Application	✔ Yes	Basic CRUD
Common SP	✔ Yes	Shared utilities
Shared SP	✔ Yes	Reusable logic
Use-case SP	✔ Yes	Workflow building blocks
UI SP	✔ Yes	Data fetching
Restrictions

Common SPs must not call any other SP except Common SPs
(to avoid circular dependency)

## 2️⃣ 🟩 Shared Business SPs — “Reusable Domain Logic”
Purpose

Shared reusable business rules used by multiple use cases.

Examples

usp_Shared_ValidateBalance

usp_Shared_CalculateFees

usp_Shared_ValidateKYC

usp_Shared_ApplyPromoCode

usp_Shared_GetCommissionRate

Allowed Calls
Caller	Allowed?	Why
Common SP	❌ No	Prevent circular dependency
Shared SP	✔ Yes	Reuse logic
Use-case SP	✔ Yes	Orchestrate workflow
Application	❌ No	Should not expose domain logic
UI SP	❌ No	UI must not apply business rules
Rules

Cannot call Use-Case SP

Not meant for UI or external calls

Should not modify data (except safe logs)

Returns validations, flags, calculated values

## 3️⃣ 🟥 Use-Case SPs — “Transactional Scripts (Main Workflows)”

These implement one complete business workflow from start to finish.

Purpose

Implements the full use case

Sequential top-down logic

Transactional consistency

Examples

usp_Usecase_ProcessElectricityPayment

usp_Usecase_TopupWallet

usp_Usecase_TransferFunds

usp_Usecase_CreateOrder

usp_Usecase_IssueRefund

Allowed Calls
Caller	Allowed?	Why
Application	✔ Yes	Triggers use case
Shared SP	✔ Yes	Reusable domain rules
Common SP	✔ Yes	CRUD operations
Restrictions

❌ Cannot call another Use-Case SP

❌ Cannot call UI SP

❌ Must NOT be reused for other workflows

❌ Not allowed to become shared/common

Each use case = independent workflow.

## 4️⃣ 🟨 UI SPs — “Query-Optimized SPs for Screens/Reports”

These SPs are read-only and contain no business logic.

Purpose

Filtering

Pagination

Sorting

Aggregated reporting

Dashboard / UI optimized queries

Examples

usp_UI_GetTransactionList

usp_UI_GetSalesSummary

usp_UI_GetWalletHistory

usp_UI_GetCustomerDashboard

usp_UI_GetInvoiceDetails

Allowed Calls
Caller	Allowed?	Why
Application	✔ Yes	Fetch UI data
Common SP	✔ Yes	Shared data access
Restrictions

❌ Cannot call Use-case SP

❌ Cannot call Shared Business SP

❌ Cannot contain business rules

❌ Must be read-only (no inserts/updates)



## Other Questions
1. Why Shared Business SPs not have change db state and what it can do.
They contain reusable business logic, but DO NOT change state.

✔ They DO:
Validate things
Calculate values
Apply rules
Return results
Perform safe writes like logs (optional)

❌ They DO NOT:
INSERT / UPDATE / DELETE business tables
Make workflow decisions
Commit transactions

Why? 
Because they are reusable.
If a reusable SP modifies data:
It becomes unpredictable
Hard to reason about
Many use-cases depend on it
A small change breaks many flows
So they must be pure logic (except logs).

2. What's use of Shared Business SP
A Shared Business Stored Procedure is a database routine that:
✔ Contains pure business rules
✔ Does NOT modify data (no INSERT/UPDATE/DELETE)
✔ Is reusable by multiple use cases
✔ Returns intermediate results, not final workflows
✔ Helps avoid repeating logic in many SPs

3. Why We Avoid Calling Shared Business SPs Directly From App/UI
   This is the MOST important part.

   1. Shared SP returns only PARTIAL logic

    Example:
    usp_Discount_Calculate returns only the discount value, not full pricing:
    
    subtotal = 1000
    discount = 10%
    final price = subtotal - discount
    
    If UI/App does final price differently → inconsistency.

   2. UI/App may misuse the SP

    Shared SP is low-level.
    Not workflow-specific.

    UI/App might:
     call it with wrong parameters
     skip required steps
     use outdated values
     bypass conditions the use-case workflow enforces
    This breaks business rules.

   3. When Shared SP logic changes → UI breaks
   Example:
    Original:
    Discount = 10% for Gold members
    Later new rule:
    Discount = 10% only if KYC completed
    Otherwise 5%


  UseCase SP correctly passes:
    customerId
    orderId
    KYC status

  But UI was calling shared SP directly → no idea about KYC.
  → UI shows wrong discount.

4.Can Workflow SPs Only GET Data?
    ✅ Can Workflow SPs Only GET Data?
    ✔ Sometimes YES
    ✔ Sometimes NO
    
    It depends on the type of workflow.
    
    Workflow (Use Case) SPs come in two types:
    
    1️⃣ READ-ONLY WORKFLOW SPs
    
    These DO ONLY GET data.
    They do not write/update anything.
    
    Examples:
    
    usp_Checkout_Preview
    usp_Loan_Eligibility_Preview
    usp_Order_GetDetails
    usp_User_ProfileView
    usp_Transaction_History
    usp_Invoice_Preview
    
    These SPs:
    
    ✔ Combine multiple shared SPs
    ✔ Validate input
    ✔ Return final output needed by UI
    ❌ No insert/update/delete
    ❌ No business side-effects
    
    These are “preview / query” use cases.
    
    2️⃣ WRITE WORKFLOW SPs
    
    These perform GET + INSERT + UPDATE + DELETE as required.
    
    Examples:
    
    usp_Order_Create
    usp_Payment_Process
    usp_Refund_Initiate
    usp_Address_Update
    usp_Cart_AddItem
    usp_OrderStatus_Update
    
    These SPs:
    
    ✔ Validate business logic
    ✔ Read necessary data
    ✔ Call shared business SPs
    ✔ Write to DB
    ✔ Commit as a single transaction
    ❌ Should not do small repeated reusable logic (only orchestrate)

5. Which SPs can the Application Layer (API/backend) call directly?
   SP Type	Can App Call Directly?	Notes
   UseCase / Workflow SP	✅ Yes	Full business logic. App must call this for workflows that may read/write data.
   Shared Business SP	⚠️ Only for “preview / read-only / partial computation”	Never for final business decision. App can call it if it just needs reusable logic (e.g., compute discount  for preview). Should not write/commit anything.
   Common SP (read-only)	✅ Yes	Safe to call for lookups, lists, reference data. No business logic or writes.
   UI SP (read-only / preview)	✅ Yes	App can call to get display-only information. Internally may call shared SPs. Never writes.

6. Where to manage transactions for data integrity
   ✅ Rule of Thumb
   All transactions should be managed in the Workflow / UseCase SP.

  Why?
  Workflow SP represents the complete use case.
  It knows all the steps, dependencies, and business rules.
  Shared Business SPs / UI SPs / Common SPs are stateless, partial computations, not meant to control DB integrity.
