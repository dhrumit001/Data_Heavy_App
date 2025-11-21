We use below architectures on this project.
📌 Transactional Script + DB-Centric Domain Logic + SQL Modularization

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

You use 3 layers:
