### 1. Difference Between `DELETE` and `TRUNCATE`

`DELETE` is a Data Manipulation Language (DML) command that removes rows one by one based on an optional `WHERE` clause, logging each deleted row in the transaction log. `TRUNCATE` is a Data Definition Language (DDL) command that deallocates entire data pages, making it significantly faster with minimal logging.

| Feature | `DELETE` | `TRUNCATE` |
| --- | --- | --- |
| **Command Type** | DML | DDL |
| **Filtering** | Supports `WHERE` clause | Cannot use `WHERE`; clears entire table |
| **Performance** | Slower; row-by-row deletion & logging | Extremely fast; deallocates pages |
| **Identity Reset** | Preserves current identity seed value | Resets identity seed to initial value |
| **Triggers** | Fires `DELETE` triggers | Does not fire triggers |
| **Rollback** | Fully rollbackable inside a transaction | Rollbackable if wrapped in explicit transaction |
| **Foreign Keys** | Works if children are handled | Fails if referenced by FK constraints |

```sql
-- DML: Deletes specific records, fires triggers
DELETE FROM Employees WHERE DepartmentId = 5;

-- DDL: Resets table and identity seed
TRUNCATE TABLE TempLogProcessing;

```

---

### 2. SQL Language Subsets: DDL, DML, DCL, and TCL

SQL operations are partitioned into four standard functional sub-languages based on their scope of execution.

```
       ┌─────────────────────────────── SQL ───────────────────────────────┐
       │                                                                   │
 ┌───────────┐                 ┌───────────┐         ┌───────────┐   ┌───────────┐
 │    DDL    │                 │    DML    │         │    DCL    │   │    TCL    │
 │ (Schema)  │                 │  (Data)   │         │(Security) │   │(Boundary) │
 └─────┬─────┘                 └─────┬─────┘         └─────┬─────┘   └─────┬─────┘
  CREATE/ALTER/DROP/TRUNCATE     SELECT/INSERT/UPDATE/DELETE   GRANT/REVOKE    COMMIT/ROLLBACK

```

* **DDL (Data Definition Language):** Defines, modifies, and deletes database schema structures.
* *Commands:* `CREATE`, `ALTER`, `DROP`, `TRUNCATE`.


* **DML (Data Manipulation Language):** Retrieves, inserts, modifies, and deletes actual business data.
* *Commands:* `SELECT`, `INSERT`, `UPDATE`, `DELETE`.


* **DCL (Data Control Language):** Manages user access, roles, and object-level permissions.
* *Commands:* `GRANT`, `REVOKE`, `DENY`.


* **TCL (Transaction Control Language):** Manages transactional units of work to preserve consistency.
* *Commands:* `COMMIT`, `ROLLBACK`, `SAVEPOINT`.



---

### 3. Joins in SQL (Inner, Left, Right, Full)

Joins combine columns from two or more tables based on a related column between them.

```
   INNER JOIN            LEFT JOIN            RIGHT JOIN           FULL OUTER
   ┌───┬───┬───┐        ┌───┬───┬───┐        ┌───┬───┬───┐        ┌───┬───┬───┐
   │ A │∩∩∩│ B │        │███│███│ B │        │ A │███│███│        │███│███│███│
   └───┴───┴───┘        └───┴───┴───┘        └───┴───┴───┘        └───┴───┴───┘
   Matching only        All Left + Match     All Right + Match    All from both

```

* **INNER JOIN:** Returns records containing matching values in both tables.
* **LEFT (OUTER) JOIN:** Returns all rows from the left table, with matching rows from the right table (or `NULL` if no match).
* **RIGHT (OUTER) JOIN:** Returns all rows from the right table, with matching rows from the left table (or `NULL` if no match).
* **FULL (OUTER) JOIN:** Returns all rows when there is a match in either left or right table, filling missing sides with `NULL`.

```sql
SELECT e.Name, d.DepartmentName
FROM Employees e
INNER JOIN Departments d ON e.DepartmentId = d.Id;

```

---

### 4. Difference Between `CHAR` and `VARCHAR` / `VARCHAR2`

* **`CHAR(n)`:** Fixed-length character storage. If stored text is shorter than defined length $n$, the engine right-pads the data with trailing spaces.
* **`VARCHAR(n)` / `VARCHAR2(n)`:** Variable-length character storage. It stores only the actual characters provided plus 1–2 prefix bytes tracking length overhead.
* *Note:* `VARCHAR2` is the Oracle-specific implementation that guarantees backward compatibility and prevents treating empty strings as identical to spaces, whereas SQL Server uses `VARCHAR(n)` and Unicode `NVARCHAR(n)`.

```sql
-- Storage comparison for string "NET":
-- CHAR(10)     -> 'NET       ' (Uses 10 bytes)
-- VARCHAR(10)  -> 'NET'        (Uses 3 bytes + 2 bytes offset = 5 bytes)

```

---

### 5. What Are Constraints? (`DEFAULT`, `UNIQUE`)

Constraints enforce business and domain integrity rules at the schema level, preventing invalid data ingestion.

* **`DEFAULT`:** Pre-populates a column with a predefined literal value or deterministic function whenever an `INSERT` statement omits it.
* **`UNIQUE`:** Guarantees that all values across a column or composite set of columns are distinct. Unlike a Primary Key, a standard `UNIQUE` constraint allows a single `NULL` value (in SQL Server/Postgres) while blocking duplicate values.

```sql
CREATE TABLE Users (
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Email VARCHAR(255) NOT NULL CONSTRAINT UQ_Users_Email UNIQUE,
    CreatedUtc DATETIME2 NOT NULL CONSTRAINT DF_Users_Created DEFAULT SYSUTCDATETIME(),
    IsActive BIT NOT NULL CONSTRAINT DF_Users_IsActive DEFAULT 1
);

```

---

### 6. What Is a Unique Key?

A Unique Key is a candidate key constraint ensuring entity uniqueness for alternate keys outside the primary surrogate identifier (such as `NationalInsuranceNumber`, `SSN`, or `Email`).

* Enforces data distinctness across all rows.
* Automatically provisions a supporting non-clustered unique B-Tree index behind the scenes to facilitate $O(\log N)$ lookup performance during validation.
* A single table can support dozens of unique keys, whereas only one Primary Key is permitted per table.

---

### 7. Difference Between Clustered and Non-Clustered Indexes

```
  CLUSTERED INDEX (The Table IS the Index)      NON-CLUSTERED INDEX (Separate B-Tree)
               ┌──────────┐                                  ┌──────────┐
               │Root Page │                                  │Root Page │
               └────┬─────┘                                  └────┬─────┘
          ┌─────────┴─────────┐                         ┌─────────┴─────────┐
     ┌────┴────┐         ┌────┴────┐               ┌────┴────┐         ┌────┴────┐
     │ Leaf 1  │         │ Leaf 2  │               │ Leaf 1  │         │ Leaf 2  │
     └─────────┘         └─────────┘               └────┬────┘         └─────────┘
   (Stores Actual Data Rows In-Place)                   │ (Points to Clustered Key/RID)
                                                        ▼
                                               [Clustered Table Row]

```

| Dimension | Clustered Index | Non-Clustered Index |
| --- | --- | --- |
| **Physical Storage** | Sorts and stores the actual data rows physically in the leaf nodes. | Separate physical structure; leaf nodes contain pointers (Row IDs / Clustered Keys) back to data. |
| **Limit per Table** | Exactly **1** per table. | Multiple per table (up to 999 in SQL Server). |
| **Lookup Speed** | Direct traversal to data row; avoids secondary bookmark lookups. | Requires an extra step (Key Lookup / RID Lookup) unless it is a covering index. |

---

### 8. Command for Current Date

In Microsoft SQL Server / T-SQL:

```sql
-- Returns current system timestamp with server's local time zone offset:
SELECT GETDATE(); 

-- Best practice for modern systems (returns UTC timestamp):
SELECT SYSUTCDATETIME(); 

```

* *Oracle equivalent:* `SELECT SYSDATE FROM dual;`
* *PostgreSQL / MySQL equivalent:* `SELECT CURRENT_TIMESTAMP;` or `NOW();`

---

### 9. What Is an Index?

An index is an on-disk B-Tree data structure that speeds up retrieval of rows by mapping indexed column values to their underlying table locations, avoiding expensive Full Table Scans.

* **Pros:** Drastically reduces I/O consumption and execution times for `SELECT`, `JOIN`, and filtered `WHERE` predicates.
* **Cons:** Introduces storage overhead and degrades performance on data modifications (`INSERT`, `UPDATE`, `DELETE`) due to index maintenance and page splits.

---

### 10. Normalization and Its Advantages

Normalization is the systematic methodology of designing relational schema structures to eliminate redundant repeating data and protect against data modification anomalies.

**Core Advantages:**

* **Elimination of Insertion Anomalies:** You can record an entity without needing another entity present.
* **Elimination of Update Anomalies:** Data modifications occur in exactly one authoritative location, avoiding inconsistent states.
* **Elimination of Deletion Anomalies:** Deleting child### 1. Difference between DELETE and TRUNCATE

| Feature | `DELETE` | `TRUNCATE` |
| --- | --- | --- |
| **Command Type** | DML (Data Manipulation Language) | DDL (Data Definition Language) |
| **WHERE Clause** | Supported (can delete specific rows) | Not supported (removes all rows) |
| **Transaction Logging** | Logs each deleted row individually; slower for large datasets | Deallocates data pages; minimal logging, significantly faster |
| **Rollback** | Fully rollbackable within an active transaction block | Rollbackable within an active transaction block in SQL Server/PostgreSQL |
| **Triggers** | Fires `AFTER DELETE` / `INSTEAD OF DELETE` triggers | Does **not** fire DML delete triggers |
| **Identity Reset** | Preserves the current identity seed value | Resets the identity counter back to the seed |
| **Foreign Keys** | Works if child records do not violate constraints | Fails immediately if referenced by any Foreign Key (even if the child table is empty) |

```sql
-- DELETE: Logged row-by-row
BEGIN TRANSACTION;
DELETE FROM Orders WHERE OrderDate < '2023-01-01';
ROLLBACK; -- Rows restored

-- TRUNCATE: Fast page deallocation
TRUNCATE TABLE TempOrderProcessing;

```

---

### 2. Subsets of SQL (DDL, DML, DCL, TCL, DQL)

* **DDL (Data Definition Language):** Defines, alters, or destroys database structure and schema.
* *Commands:* `CREATE`, `ALTER`, `DROP`, `TRUNCATE`, `RENAME`.


* **DML (Data Manipulation Language):** Modifies data instances inside tables.
* *Commands:* `INSERT`, `UPDATE`, `DELETE`, `MERGE`.


* **DQL (Data Query Language):** Queries and extracts data without modifying state.
* *Commands:* `SELECT`.


* **DCL (Data Control Language):** Manages user rights, permissions, and security.
* *Commands:* `GRANT`, `REVOKE`.


* **TCL (Transaction Control Language):** Manages atomic database transactions.
* *Commands:* `COMMIT`, `ROLLBACK`, `SAVEPOINT`.



---

### 3. Joins in SQL (INNER, FULL, LEFT, RIGHT)

```text
Table A (Left)               Table B (Right)
+----+-------+              +----+--------+
| ID | Name  |              | ID | Detail |
+----+-------+              +----+--------+
|  1 | Alice |              |  1 | Desk   |
|  2 | Bob   |              |  3 | Chair  |
+----+-------+              +----+--------+

```

```text
       INNER JOIN                      LEFT JOIN
      ┌───────────┐                  ┌───────────┐
   ┌──┼──┐     ┌──┼──┐            ┌──┼───────────┼──┐
   │A │  │A ∩ B│  │ B│            │A │   A ∩ B   │ B│
   └──┼──┘     └──┼──┘            └──┼───────────┼──┘
      └───────────┘                  └───────────┘
   Matched keys only              All from Left + Matched Right

       RIGHT JOIN                      FULL JOIN
      ┌───────────┐                  ┌───────────┐
   ┌──┼───────────┼──┐            ┌──┼───────────┼──┐
   │A │   A ∩ B   │ B│            │A │   A ∪ B   │ B│
   └──┼───────────┼──┘            └──┼───────────┼──┘
      └───────────┘                  └───────────┘
 All from Right + Matched Left     All rows from both sides

```

* **INNER JOIN:** Returns records where the join key exists in both tables.
```sql
SELECT A.Name, B.Detail 
FROM TableA A 
INNER JOIN TableB B ON A.ID = B.ID;
-- Result: (Alice, Desk)

```


* **LEFT (OUTER) JOIN:** Returns all records from the left table; unmatched right-side columns populate as `NULL`.
```sql
SELECT A.Name, B.Detail 
FROM TableA A 
LEFT JOIN TableB B ON A.ID = B.ID;
-- Result: (Alice, Desk), (Bob, NULL)

```


* **RIGHT (OUTER) JOIN:** Returns all records from the right table; unmatched left-side columns populate as `NULL`.
```sql
SELECT A.Name, B.Detail 
FROM TableA A 
RIGHT JOIN TableB B ON A.ID = B.ID;
-- Result: (Alice, Desk), (NULL, Chair)

```


* **FULL (OUTER) JOIN:** Combines Left and Right joins, returning matching and non-matching records from both sides.
```sql
SELECT A.Name, B.Detail 
FROM TableA A 
FULL JOIN TableB B ON A.ID = B.ID;
-- Result: (Alice, Desk), (Bob, NULL), (NULL, Chair)

```



---

### 4. Difference between CHAR and VARCHAR / VARCHAR2

| Attribute | `CHAR(n)` | `VARCHAR(n)` / `VARCHAR2(n)` |
| --- | --- | --- |
| **Storage Type** | Fixed length | Variable length |
| **Space Allocation** | Always consumes $n$ bytes. Pads unused characters with trailing spaces | Consumes only actual characters + 1–2 length prefix bytes |
| **Performance** | Faster for static, known lengths (e.g., country codes `US`, GUIDs, MD5 hashes) | Minimal parsing overhead, but conserves substantial disk and memory buffer pool space |
| **Dialect Note** | Standard SQL | `VARCHAR2` is Oracle-specific (guarantees empty strings $\neq$ NULL handling differences); `VARCHAR` is standard ANSI/SQL Server |

```sql
-- Storage footprint comparison for value 'NET'
Code CHAR(10);     -- Allocates 10 bytes: 'NET       '
Code VARCHAR(10);  -- Allocates 3 bytes data + 1-2 bytes metadata: 'NET'

```

---

### 5. What are Constraints? (`DEFAULT`, `UNIQUE`)

Constraints are declarative rules enforced by the database engine on table columns to preserve data integrity and domain validity.

* **`DEFAULT`:** Injects a fallback value when an `INSERT` statement omits the column value.
* **`UNIQUE`:** Ensures that no two rows share identical values across the defined column(s), while allowing a single `NULL` (in SQL Server) or multiple `NULL`s (in Oracle/PostgreSQL ANSI standards).

```sql
CREATE TABLE Users (
    UserId INT IDENTITY(1,1) PRIMARY KEY,
    Email VARCHAR(255) NOT NULL,
    IsActive BIT CONSTRAINT DF_Users_IsActive DEFAULT 1,
    CONSTRAINT UQ_Users_Email UNIQUE (Email)
);

```

---

### 6. What is a UNIQUE Key?

A **UNIQUE Key** is an integrity constraint that guarantees uniqueness across non-primary key columns to prevent duplicate records.

* Unlike a Primary Key, a table can support multiple `UNIQUE` constraints.
* By default in SQL Server, applying a `UNIQUE` constraint creates a backing **Non-Clustered Index** (unless explicitly defined as clustered).
* Unlike Primary Keys which reject all nullability (`NOT NULL`), unique keys permit `NULL` depending on the RDBMS implementation.

```sql
ALTER TABLE Employees 
ADD CONSTRAINT UQ_Employee_NationalID UNIQUE (NationalIDNumber);

```

---

### 7. Difference between Clustered and Non-Clustered Indexes

```text
Clustered Index (B-Tree)                Non-Clustered Index (B-Tree)
        [ Root ]                                [ Root ]
        /      \                                /      \
    [Branch]  [Branch]                      [Branch]  [Branch]
     /    \    /    \                        /    \    /    \
   [Data Leaf Pages]                       [Leaf: Key + RID / PK]
(Table data IS the leaf level)                         │
                                                       ▼ Pointer Lookup
                                             [Data Pages / Heap / Clustered]

```

| Feature | Clustered Index | Non-Clustered Index |
| --- | --- | --- |
| **Physical Storage** | Dictates the physical sort order of actual data rows | Stored in a separate structure; points back to base table |
| **Quantity per Table** | Exactly **1** per table | Multiple (up to 999 in SQL Server) |
| **Leaf Level** | Contains the actual data rows | Contains index keys + a row locator (RID pointer or Clustered Key) |
| **Primary Key Default** | Creating a `PRIMARY KEY` defaults to Clustered | Creating a `UNIQUE` constraint defaults to Non-Clustered |

---

### 8. Command for Current Date: `SELECT GETDATE()`

`GETDATE()` is a deterministic/runtime-evaluated T-SQL system function that returns the current database host timestamp (`DATETIME` format):

```sql
-- SQL Server
SELECT GETDATE() AS CurrentDateTime; 
SELECT SYSDATETIMEOFFSET() AS CurrentDateTimeWithOffset; -- Recommended for UTC/TimeZone safety

-- Database Dialect Equivalents:
-- PostgreSQL: SELECT NOW();
-- Oracle:     SELECT SYSDATE FROM dual;
-- MySQL:      SELECT CURRENT_TIMESTAMP();

```

---

### 9. What is an Index?

An **Index** is an on-disk data structure (primarily a balanced B-Tree) that optimizes data retrieval speeds at the cost of additional disk space and insert/update/delete performance.

* Without an index, the query engine performs a **Table Scan** (reads every single page from disk into memory, $O(N)$).
* With an index, the query engine traverses tree nodes via binary search ($O(\log N)$), performing an **Index Seek** to directly locate target rows.

---

### 10. Normalization and its Advantages

**Normalization** is the systematic database design methodology of organizing columns and tables to eliminate data redundancy and avoid anomalies (`INSERT`, `UPDATE`, `DELETE`).

**Advantages:**

* **Eliminates Anomalies:** Prevents data inconsistencies (e.g., updating an employee address in one record while leaving old values in duplicates).
* **Storage Optimization:** Reduces duplicated string literals across millions of records.
* **Narrower Tables:** Leaner tables mean more rows fit inside an 8 KB data page, increasing buffer cache hits during scans.
* **Disadvantage / Trade-off:** High normalization requires more `JOIN` operations, which can degrade read throughput in reporting systems (often resolved by intentional denormalization or OLAP data warehousing).

---

### 11. Difference between DROP and TRUNCATE

| Attribute | `TRUNCATE` | `DROP` |
| --- | --- | --- |
| **Target** | Cleans out all rows inside the table | Destroys the table definition, metadata, and data entirely |
| **Schema Impact** | Keeps table schema, constraints, indexes intact | Removes schema, triggers, privileges, and constraints from catalog |
| **Reusability** | You can immediately `INSERT` new data | The object no longer exists; queries against it throw errors |

```sql
TRUNCATE TABLE AuditLogs; -- Table remains empty and queryable
DROP TABLE AuditLogs;     -- Table is gone; SELECT * throws "Invalid object name"

```

---

### 12. Normalization Forms (1NF, 2NF, 3NF, BCNF)

```text
Raw Table ──► [ 1NF ] ──► [ 2NF ] ──► [ 3NF ] ──► [ BCNF ]
             Atomic       Remove Partial    Remove Transitive   Strict Candidate
             Values        Dependencies       Dependencies         Key Rules

```

* **1NF (First Normal Form):**
* Columns contain atomic (indivisible) values.
* No repeating groups or arrays.
* Must have a primary identifier.


* **2NF (Second Normal Form):**
* Must meet 1NF.
* No **partial dependencies**: All non-key attributes must be fully functionally dependent on the *entire* candidate key (primarily applies to composite keys).


* **3NF (Third Normal Form):**
* Must meet 2NF.
* No **transitive dependencies**: Non-key attributes must depend *only* on the primary key, not on other non-key attributes ($A \rightarrow B \rightarrow C$).


* **BCNF (Boyce-Codd Normal Form):**
* Strict variant of 3NF. For every functional dependency ($X \rightarrow Y$), the determinant $X$ must be a **Super Key**.



---

### 13. ACID Properties in Relational Databases

* **Atomicity:** "All or nothing." Either every statement in the transaction executes successfully, or the entire transaction rolls back to the initial state.
* **Consistency:** A transaction transitions the database from one valid state to another, strictly enforcing all constraints, foreign keys, and schema rules.
* **Isolation:** Concurrent transactions execute without cross-contamination. Isolation levels determine visibility:
* *Read Uncommitted* $\rightarrow$ *Read Committed* $\rightarrow$ *Repeatable Read* $\rightarrow$ *Serializable* (alongside snapshot isolation).


* **Durability:** Once committed, transaction writes persist even during server crashes or power failures via the Write-Ahead Log (WAL / Transaction Log `LDF`).

---

### 14. Triggers in SQL

A **Trigger** is a specialized stored procedure that executes automatically in response to specific database events.

* **DML Triggers:** Fire during data modifications (`INSERT`, `UPDATE`, `DELETE`).
* `AFTER` / `FOR` Trigger: Executes after data modifications pass validation checks.
* `INSTEAD OF` Trigger: Intercepts and replaces the initiating operation (often used on complex `VIEWS` to allow updates).


* **DDL Triggers:** Fire during administrative operations like `CREATE_TABLE`, `DROP_TABLE`, or `ALTER_TABLE` for auditing and compliance.
* **Context Tables:** SQL Server exposes transient memory buffers inside triggers:
* `inserted`: Holds newly inserted or post-update values.
* `deleted`: Holds deleted or pre-update values.



```sql
CREATE TRIGGER trg_AuditSalaryChange
ON Employees
AFTER UPDATE
AS
BEGIN
    SET NOCOUNT ON;
    IF UPDATE(Salary)
    BEGIN
        INSERT INTO AuditLog (EmpId, OldSalary, NewSalary, ModifiedDate)
        SELECT d.Id, d.Salary, i.Salary, GETDATE()
        FROM deleted d
        INNER JOIN inserted i ON d.Id = i.Id;
    END
END;

```

---

### 15. Operators in SQL

* **Arithmetic Operators:** `+`, `-`, `*`, `/`, `%`
* **Comparison Operators:** `=`, `<>`, `!=`, `>`, `<`, `>=`, `<=`
* **Logical Operators:** `AND`, `OR`, `NOT`, `IN`, `BETWEEN`, `LIKE`, `EXISTS`, `ALL`, `ANY`
* **Set Operators:** `UNION`, `UNION ALL`, `INTERSECT`, `EXCEPT` (or `MINUS` in Oracle)
* **Bitwise Operators:** `&` (AND), `|` (OR), `^` (Exclusive OR)

---

### 16. CROSS JOIN vs. NATURAL JOIN

* **CROSS JOIN:** Produces a complete **Cartesian Product** of the two involved tables. If Table A has $M$ rows and Table B has $N$ rows, the result contains $M \times N$ rows. It does not require an `ON` condition.
```sql
SELECT p.ProductName, s.SizeName 
FROM Products p 
CROSS JOIN Sizes s;

```


* **NATURAL JOIN:** Automatically joins two tables based on all columns having the **same column name and data type** across both tables.
* *Caution:* It does not require an explicit `ON` clause, making it fragile in enterprise applications. Renaming or adding an identically named audit column (e.g., `CreatedDate`) unexpectedly changes the join condition.



```sql
-- Implicitly matches on common column names (e.g., DepartmentID)
SELECT * 
FROM Employees 
NATURAL JOIN Departments;

```

---

### 17. Finding the 3rd Highest Salary

#### Analysis of the Notebook's Subquery Approach

The approach written on the paper:

```sql
SELECT TOP 1 Salary 
FROM (
    SELECT TOP 3 Salary 
    FROM Employees 
    ORDER BY Salary DESC
) AS Emp 
ORDER BY Salary ASC;

```

*Critique:* This approach works in SQL Server **only if salaries are strictly unique**. If duplicate salaries exist (e.g., `100k, 100k, 90k`), `TOP 3` captures `100k, 100k, 90k`, and `TOP 1 ... ASC` returns `90k`, which is actually the **2nd distinct** highest salary.

---

#### Enterprise Solutions

**Solution A: Modern Window Functions (`DENSE_RANK`) [Recommended]**
Handles duplicates correctly without skipping ranking numbers:

```sql
WITH RankedSalaries AS (
    SELECT 
        Salary,
        DENSE_RANK() OVER (ORDER BY Salary DESC) AS RankOrder
    FROM Employees
)
SELECT DISTINCT Salary 
FROM RankedSalaries 
WHERE RankOrder = 3;

```

**Solution B: ANSI Standard `OFFSET-FETCH` (SQL Server 2012+, PostgreSQL, Oracle 12c+)**
Cleaner pagination query:

```sql
SELECT DISTINCT Salary
FROM Employees
ORDER BY Salary DESC
OFFSET 2 ROWS FETCH NEXT 1 ROWS ONLY;

```

**Solution C: Correlated Subquery (Engine-Agnostic)**

```sql
SELECT DISTINCT e1.Salary
FROM Employees e1
WHERE 2 = (
    SELECT COUNT(DISTINCT e2.Salary)
    FROM Employees e2
    WHERE e2.Salary > e1.Salary
);

```
