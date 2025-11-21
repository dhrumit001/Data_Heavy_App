We use below architectures on this project.

## 📌 Transactional Script + DB-Centric Domain Logic + SQL Modularization

🔵 1. Transactional Script (Application Layer Pattern)
Each business use-case is implemented as a single procedure (method/SP) that executes steps in sequence.

✔ Characteristics
One use case = One function / One stored procedure
Logic flows top-down (step → step → step)
Very easy to understand
Perfect for CRUD + procedural business rules
No entities, aggregates, or domain models

Each SP contains the entire workflow from start to finish.
This is the pattern, not the implementation location.

🔴 2. DB-Centric Domain Logic (Where Business Rules Live)
✔ Your business rules live inside SQL, not in C# code.
All core domain logic is executed in stored procedures:
For ex : Validate balance,Apply markup,Decide provider,Calculate fees,Reverse ledger

This makes the database the domain layer.

✔ Why?
Because:
Multiple systems can reuse it
SQL provides consistency
SQL is optimized for data-heavy operations
Easier to guarantee ACID consistency
Application layer becomes thin/orchestration only

❌ This is NOT DDD
Because DDD requires domain logic inside classes (entities/value objects).
Here, domain logic is in the DB.

This is a classic enterprise approach.

🟢 3. SQL Modularization (How SQL is Organized)
This means breaking SQL into layers so it is not spaghetti logic.


## Let's deep dive into SQL Modularization Architecture (Deep Explanation)
Your stored procedures are organized into 4 Layers:
1️⃣ Common SPs (Pure CRUD / Utilities)
2️⃣ Shared Business SPs (Reusable business rules)
3️⃣ Use-Case SPs (Transactional scripts / workflows)
4️⃣ UI SPs (Reporting / Listing optimized for UI)

🟦 1️⃣ Common SPs — “Technical layer”
Purpose
Reusable CRUD operations
NO business rules.
Pure data access.

Examples
usp_Common_GetCustomerById
usp_Common_GetBalance
usp_Common_InsertAuditLog
usp_Common_UpdateStatus
usp_Common_InsertLedgerEntry

Allowed Calls
Caller	    Allowed?	         Why
Application	✔ Allowed	Simple   CRUD
Common SP	  ✔ Allowed	Shared   utilities
Shared SP	  ✔ Allowed	Reusable logic
Use-case SP	✔ Allowed	Building blocks
UI SP	      ✔ Allowed	         Data fetch only

Please note this SP is not call any other sp aprart from Common SPs.

🟩 2️⃣ Shared Business SPs — “Reusable domain logic”
Purpose
Contains business rules that are used in multiple use-cases.

Examples
usp_Shared_ValidateBalance
usp_Shared_CalculateFees
usp_Shared_ValidateKYC
usp_Shared_ApplyPromoCode
usp_Shared_GetCommissionRate

Allowed Calls
Caller	   Allowed?	   Why
Common SP	 ❌          NO	Prevent circular dependency
Shared SP	 ✔ Yes	      Reuse
Use-case SP	✔Yes	      Orchestrates use case
Application	❌ Avoid	  It exposes business rule directly
UI SP	      ❌ Avoid	  UI should not apply domain logic

Not Allowed
❌ Cannot call Use-Case SP
❌ Not meant to be called directly by UI or application
❌ Must NOT modify data (except safe writes like logs)

These SPs return:
Valid/invalid
Fee amount
Discount
Flags
Calculated values

🟥 3️⃣ Use-Case SPs — “Complete workflows / Transaction Scripts”
This is the heart of the architecture.

Purpose
Implements one business use-case, top-to-bottom.

Examples
usp_Usecase_ProcessElectricityPayment
usp_Usecase_TopupWallet
usp_Usecase_TransferFunds
usp_Usecase_CreateOrder
usp_Usecase_IssueRefund

Allowed Calls
Caller	     Allowed?	Why
Application	 ✔ Yes	  Use case execution
Shared SP	   ✔ Yes	  Reusable business rules
Common SP	   ✔ Yes	  CRUD operations

Not Allowed
❌ Use-case SP cannot be called by another use-case SP
❌ Use-case SP cannot call UI SP
❌ Use-case SP must not be reused for other workflows
❌ Use-case SP cannot be “common”


🟨 4️⃣ UI SPs — “Query-Optimized for UI & Reports”
These are NOT business logic SPs.
They exist only to shape data for the UI screens.

Purpose
Pagination
Sorting
Filtering
Aggregated reporting
Dashboard data
List screens

Examples
usp_UI_GetTransactionList
usp_UI_GetSalesSummary
usp_UI_GetWalletHistory
usp_UI_GetCustomerDashboard
usp_UI_GetInvoiceDetails

Allowed Calls
Caller	    Allowed?	Why
Application	✔ Yes	    UI pages call these
Common SP	  ✔ Yes	    If required

Not Allowed
❌ Cannot call Use-case SP (workflow)
❌ Cannot call Shared Business SP (business logic)
❌ Cannot contain business rules
❌ Cannot be used for updates (UI SP = read-only)
