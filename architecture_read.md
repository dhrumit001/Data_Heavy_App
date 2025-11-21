📘 Architecture Documentation
Transactional Script + DB-Centric Domain Logic + SQL Modularization

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

✅ Summary Diagram
                 ┌────────────────────────┐
                │      Application       │
                │   (Thin Orchestration) │
                └───────────┬────────────┘
                            │
                            ▼
                  ┌──────────────────┐
                  │  Use-Case SPs    │  (Workflows)
                  └──────┬─────┬─────┘
                         │     │
          ┌──────────────┘     └───────────────┐
          ▼                                     ▼
┌────────────────────┐              ┌────────────────────┐
│ Shared Business SPs │              │    Common SPs      │
│  (Reusable Rules)   │              │  (CRUD Utilities)   │
└────────────────────┘              └────────────────────┘

               ┌────────────────────┐
               │       UI SPs       │  (Read-Only)
               └────────────────────┘
