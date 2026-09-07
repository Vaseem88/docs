# Senior .NET / Database Engineer Interview Preparation Guide (Part 3: SQL & Database Management)
**Candidate Persona:** Senior Software Engineer (8+ Years Experience in .NET Framework / .NET Core, SQL Server, EF Core, and Relational Database Architecture)

---

## 26. Temporary Tables (Temp Table)
**Interview Answer:**
> "In SQL Server / relational database systems, temporary tables are physical tables instantiated inside the system database `tempdb` to hold interim calculation or staging data during batch operations, stored procedures, or complex data migrations.
>
> They come in two primary scopes:
> 
> 1. **Local Temporary Tables (`#TableName`):**
>    - Prefixed with a single hash (`#`).
>    - Visible only to the current user session or database connection that instantiated it.
>    - Dropped automatically when the connection terminates or when the scope (such as a stored procedure) completes.
> 
> 2. **Global Temporary Tables (`##TableName`):**
>    - Prefixed with a double hash (`##`).
>    - Visible across all concurrent database sessions.
>    - Dropped automatically when the session that created it disconnects and all active queries referencing it have finished executing.
> 
> **Senior Architecture Nuance (Temp Table vs. Table Variable `@Table`):**
> - **Temp Tables (`#`)**: Have full distribution statistics, support non-clustered indexes, parallel execution plans, and transaction log logging. Best suited for large datasets ($> 1,000$ rows) where the query optimizer needs accurate cardinalities.
> - **Table Variables (`@`)**: Live primarily in memory (spilling to `tempdb` if under memory pressure), have limited statistics (traditionally estimated as 1 row prior to modern SQL Server compilation enhancements), and do not support dynamic non-clustered index creation after declaration. Best for tiny datasets (< 100 rows)."

---

## 27. Difference between `UNION` and `UNION ALL`
**Interview Answer:**
> "Both set operators combine the result sets of two or more `SELECT` queries into a single unified result set, requiring identical column counts, order, and compatible data types. However, their mechanics and performance profiles differ fundamentally:
>
> | Feature | `UNION` | `UNION ALL` |
> |---|---|---|
> | **Duplicate Handling** | Removes duplicate records (returns distinct rows only) | Retains all records, including duplicate rows |
> | **Execution Mechanism** | Performs an internal `Sort` / `Distinct Sort` or `Hash Aggregate` operation | Performs a simple concatenation of streams |
> | **Memory & CPU Impact** | Higher CPU, memory, and `tempdb` overhead due to duplicate detection | Minimal overhead; stream-based pass-through |
> | **Performance** | Slower | Significantly faster |
>
> **Production Best Practice:**
> Default to `UNION ALL` unless your business requirements explicitly require distinct deduplication. Using `UNION` unnecessarily forces SQL Server to perform costly sorting and hashing, which introduces bottlenecks in high-throughput enterprise pipelines."

---

## 28. Copying One Table to Another (`INSERT INTO ... SELECT` vs. `SELECT ... INTO`)
**Interview Answer:**
> "There are two standard approaches to copy data between tables in SQL Server, depending on whether the target table already exists:
>
> 1. **Target Table Already Exists: `INSERT INTO ... SELECT`**
>    - Syntax:
>      ```sql
>      INSERT INTO TargetTable (Col1, Col2, Col3)
>      SELECT Col1, Col2, Col3
>      FROM OriginalTable
>      WHERE IsActive = 1;
>      ```
>    - The schema, constraints, data types, indexes, and nullability must already be defined on `TargetTable`.
>    - Best used for ETL, transactional auditing, and recurring data staging pipelines.
>
> 2. **Target Table Does Not Exist: `SELECT ... INTO`**
>    - Syntax:
>      ```sql
>      SELECT Col1, Col2, Col3
>      INTO NewTargetTable
>      FROM OriginalTable;
>      ```
>    - SQL Server creates a brand new table dynamically on the fly based on the source column names, types, and nullability.
>    - **Caveat:** It copies data and basic schemas, but does **not** copy primary keys, foreign key constraints, triggers, or non-clustered indexes. It operates as a minimally logged operation under Simple or Bulk-Logged recovery models, making it ideal for quick staging and backups."

---

## 29. What is a DBMS (Database Management System)?
**Interview Answer:**
> "A Database Management System (DBMS) is specialized system software that serves as the administrative interface between end-users/applications and persistent physical data storage. It orchestrates the definition, creation, querying, update, and administration of structured data.
>
> Core functions include:
> - **Data Concurrency & Isolation:** Managing multiple simultaneous transactions safely via locking and multi-version concurrency control (MVCC).
> - **ACID Compliance:** Guaranteeing Atomicity, Consistency, Isolation, and Durability across database transactions.
> - **Data Integrity & Security:** Enforcing referential constraints, schemas, encryption (TDE), authentication, and granular role-based authorization (RBAC).
> - **Performance & Storage Management:** Utilizing query optimizers, indexing structures (B-Trees, ColumnStore), buffer caches, and transaction write-ahead logs (WAL).
>
> **Evolutionary Context:** In enterprise .NET architectures, we primarily utilize **RDBMS** engines (SQL Server, PostgreSQL) for relational, structured business entities, paired with **NoSQL DBMS** (CosmosDB, Redis, MongoDB) for distributed caches, unstructured documents, and high-throughput real-time telemetries."

---

## 30. Difference between Primary Key and Unique Key
**Interview Answer:**
> "Both enforce entity uniqueness and data integrity across table columns, but they have distinct structural constraints:
>
> | Feature | Primary Key (PK) | Unique Key (UK) |
> |---|---|---|
> | **Nullability** | Rejects `NULL` values strictly (`NOT NULL` mandatory) | Allows `NULL` values (in SQL Server, allows exactly one `NULL` row; standard ANSI permits multiple `NULL`s) |
> | **Limit per Table** | Only **one** Primary Key allowed per table | Multiple Unique Keys can be declared per table |
> | **Default Index Type** | Creates a **Clustered Index** by default (physically ordering table data pages) | Creates a **Non-Clustered Index** by default (can be overridden to clustered if PK is non-clustered) |
> | **Architectural Purpose** | Serves as the primary immutable identifier for entity identity and Foreign Key references | Enforces alternate candidate keys and domain constraints (e.g., `EmailAddress`, `NationalInsuranceNumber`) |
>
> **Senior Architectural Tip:**
> While a Primary Key defaults to a clustered index, in high-insertion rate environments where PKs use `Guid.NewGuid()`, non-sequential values cause catastrophic index page splits and fragmentation. As a best practice, use sequential keys (`NEWSEQUENTIALID()`), or designate an integer surrogate key as clustered while placing a Unique Key on natural identifiers."
