Here are professional, interview-ready answers framed from the perspective of an experienced backend and database engineer.

---

### 26. Temp Table (Temporary Tables)

In SQL Server, **Temporary Tables** are physical tables created inside the system database `tempdb`. They temporarily store intermediate result sets for complex querying, transformations, or batching within stored procedures.

There are two primary types:

* **Local Temporary Tables (`#TableName`):** Visible exclusively to the current user session/connection that created it. Automatically dropped when the session disconnects or the defining stored procedure finishes execution.
* **Global Temporary Tables (`##TableName`):** Accessible across all concurrent user sessions and database connections. Automatically dropped once the creating session closes and no other active session holds a reference to it.

```text
[ Client Session 1 ] ──> Creates #Orders  (Stored in tempdb, visible only to Session 1)
[ Client Session 2 ] ──> Cannot see #Orders

[ Any Session ]      ──> Can access ##GlobalCache (Stored in tempdb until fully released)

```

```sql
-- Creating and using a local temporary table
CREATE TABLE #ActiveUsers (
    UserId INT,
    Username NVARCHAR(50)
);

INSERT INTO #ActiveUsers (UserId, Username)
SELECT Id, Name FROM Users WHERE IsActive = 1;

-- Querying the temp table
SELECT * FROM #ActiveUsers;

-- Explicit cleanup (best practice)
DROP TABLE #ActiveUsers;

```

> **Production Insight:** For lightweight sets (under a few thousand rows), a table variable (`@TableVar`) works well in memory. For large data sets that require indexes, foreign statistics, or parallel plans, `#TempTables` are preferred because the query optimizer generates full distribution statistics for them.

---

### 27. Difference Between UNION and UNION ALL

Both operators vertically combine the result sets of two or more `SELECT` queries into a single output stream. Every query must project the same number of columns with compatible data types.

```text
Dataset A: { 1, 2, 3 }
Dataset B: { 2, 3, 4 }

UNION     ──> Distinct Sort / De-duplicate ──> Result: { 1, 2, 3, 4 }
UNION ALL ──> Direct Stream Concatenation  ──> Result: { 1, 2, 2, 3, 3, 4 }

```

| Feature | `UNION` | `UNION ALL` |
| --- | --- | --- |
| **Duplicates** | Removes duplicate records from the final result set. | Retains all records, including exact duplicates. |
| **Performance** | Slower; forces a distinct sort or hash match operation. | Significantly faster; streams and concatenates rows directly. |
| **Memory / Tempdb** | High resource consumption due to deduplication overhead. | Negligible resource overhead beyond standard output streaming. |

```sql
-- Returns only unique employee IDs
SELECT EmployeeId FROM PayrollDepartment
UNION
SELECT EmployeeId FROM HRDepartment;

-- Preserves all records without deduplication cost
SELECT EmployeeId FROM PayrollDepartment
UNION ALL
SELECT EmployeeId FROM HRDepartment;

```

---

### 28. Copy One Table to Another

There are two primary patterns in SQL, depending on whether the target table already exists:

#### Scenario A: Target Table Already Exists (`INSERT INTO ... SELECT`)

Inserts data from a source query into an existing, predefined schema.

```sql
-- Target table already created
INSERT INTO TargetEmployees (Id, FullName, Department)
SELECT Id, FullName, Department
FROM SourceEmployees
WHERE IsActive = 1;

```

#### Scenario B: Target Table Does Not Exist (`SELECT ... INTO`)

Creates a new physical table on the fly matching the source data types, column nullability, and schema, then copies the data.

```sql
-- Automatically creates TargetEmployeesBackup and populates it
SELECT Id, FullName, Department
INTO TargetEmployeesBackup
FROM SourceEmployees;

```

```text
INSERT INTO ... SELECT:
[ Existing Table Schema ] <── Writes Rows ── [ Query on Source Table ]

SELECT ... INTO:
[ Generates New Schema ] + [ Populates Rows ] <── [ Source Table Scan ]

```

> **Note:** `SELECT ... INTO` copies column names, data types, and `IDENTITY` attributes, but it does **not** copy existing Indexes, Primary Keys, Foreign Keys, or Default Constraints to the newly created table.

---

### 29. What is DBMS?

A **Database Management System (DBMS)** is a software system designed to define, construct, query, update, and administer structured data. It acts as an abstraction interface between the underlying storage layer and end-user applications.

```text
+-----------------------------------------------------------+
|               Client / Application (.NET)                 |
+-----------------------------------------------------------+
                             │  (SQL / Queries)
                             ▼
+-----------------------------------------------------------+
|                          DBMS                             |
|  ┌───────────────────┐  ┌──────────────────────────────┐  |
|  |   Query Engine    |  | Transaction Engine (ACID)    |  |
|  ├───────────────────┤  ├──────────────────────────────┤  |
|  | Concurrency/Locks |  | Security & User Privileges   |  |
|  └───────────────────┘  └──────────────────────────────┘  |
+-----------------------------------------------------------+
                             │  (File I/O / Pages)
                             ▼
+-----------------------------------------------------------+
|              Physical Storage (Data & Log Files)          |
+-----------------------------------------------------------+

```

Core responsibilities include:

* **Data Integrity & Consistency:** Enforces business constraints (foreign keys, uniqueness, check constraints).
* **ACID Compliance:** Guarantees Atomicity, Consistency, Isolation, and Durability across transactions.
* **Concurrency Control:** Manages multi-user read/write access via locking and row versioning (MVCC).
* **Security & Recovery:** Manages role-based access control, write-ahead logging (WAL), automated backups, and point-in-time recovery.

---

### 30. Difference Between Primary Key and Unique Key

Both enforce entity integrity and guarantee column value uniqueness, but they serve different relational and structural roles.

```text
Table: Users
┌──────────┬────────────────────────────┬─────────────────┐
│  Id (PK) │ Email (Unique Key)         │ AlternateEmail  │
├──────────┼────────────────────────────┼─────────────────┤
│  101     │ alex@example.com           │ NULL            │  <-- 1 NULL allowed in UK
│  102     │ dev@example.com            │ support@ex.com  │
│  NULL    │ (Rejected by PK Constraint)│                 │
└──────────┴────────────────────────────┴─────────────────┘

```

| Feature | Primary Key (PK) | Unique Key (UK) |
| --- | --- | --- |
| **Nullability** | Strictly disallows `NULL` (`NOT NULL` mandatory). | Allows `NULL` (SQL Server allows exactly one `NULL` row). |
| **Quantity per Table** | Exactly **one** Primary Key per table. | **Multiple** Unique Keys permitted per table. |
| **Default Index** | Creates a **Clustered Index** by default. | Creates a **Non-Clustered Index** by default. |
| **Purpose** | Uniquely identifies each row in the relational model. | Enforces non-primary unique constraints (e.g., Email, SSN). |

```sql
CREATE TABLE Customers (
    CustomerId INT NOT NULL,
    Email NVARCHAR(100) NULL,
    SSN VARCHAR(11) NOT NULL,
    
    -- Exactly one PK (Clustered Index by default)
    CONSTRAINT PK_Customers PRIMARY KEY (CustomerId),
    
    -- Multiple UKs allowed (Non-Clustered Indexes by default)
    CONSTRAINT UQ_Customers_Email UNIQUE (Email),
    CONSTRAINT UQ_Customers_SSN UNIQUE (SSN)
);

```

---

Would you like to explore indexing strategies for these tables, such as clustered versus non-clustered index trade-offs?
