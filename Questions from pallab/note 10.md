## 18. Difference Between BETWEEN and IN

* **`BETWEEN`**: Filters data within a continuous, inclusive range (equivalent to `>= value1 AND <= value2`). It is optimal for sequential numeric, date, or timestamp intervals.
* **`IN`**: Filters data against a discrete set of specified values or the result of an inner subquery (equivalent to multiple `OR` conditions: `val = 1 OR val = 2`).

```sql
-- Continuous range (inclusive: 10,000 to 50,000)
SELECT * FROM Employees 
WHERE Salary BETWEEN 10000 AND 50000;

-- Discrete list match
SELECT * FROM Employees 
WHERE DepartmentID IN (1, 3, 7, 9);

```

---

## 19. Fetch Common Records from Two Tables

Common records across two datasets with compatible schemas can be retrieved using the set operator **`INTERSECT`** or an **`INNER JOIN`**.

### Using INTERSECT

Returns distinct rows that appear in both queries.

```sql
SELECT StudentID FROM Student
INTERSECT
SELECT StudentID FROM Exam;

```

### Using INNER JOIN

Provides better flexibility when you need to project different columns from both tables.

```sql
SELECT DISTINCT s.StudentID, s.StudentName
FROM Student s
INNER JOIN Exam e ON s.StudentID = e.StudentID;

```

---

## 20. Different Set Operators (UNION, INTERSECT, MINUS/EXCEPT)

Set operators combine the results of two or more queries into a single result set. Both queries must have the same number of columns with compatible data types.

```
       UNION               INTERSECT           EXCEPT / MINUS
   ┌───┐   ┌───┐         ┌───┐   ┌───┐         ┌───┐   ┌───┐
  │ A  │███│ B │        │ A │███│ B │        │███│   │ B │
  │████████████│        │   │███│   │        │███│   │   │
   └───┘   └───┘         └───┘   └───┘         └───┘   └───┘
  All elements from     Only elements in      Elements in Table A
  both tables (distinct) both Table A & B      NOT present in Table B

```

* **`UNION`**: Combines rows from both queries and removes duplicate records.
* **`UNION ALL`**: Combines rows from both queries, **retaining** all duplicates (faster because it skips the distinct sorting step).
* **`INTERSECT`**: Returns only rows present in both result sets.
* **`EXCEPT` (SQL Server) / `MINUS` (Oracle)**: Returns distinct rows from the first query that do not exist in the second query.

---

## 21. How You Can Fetch Alternate Records

To fetch alternate (odd or even) records, generate a sequential sequence number using `ROW_NUMBER()` and apply a modulo operation (`%` or `MOD`).

```sql
-- Using a Common Table Expression (CTE) in SQL Server / PostgreSQL
WITH RankedStudents AS (
    SELECT 
        StudentID, 
        StudentName, 
        ROW_NUMBER() OVER (ORDER BY StudentID ASC) AS RowNum
    FROM Student
)
-- Even records (change to = 1 for odd records)
SELECT StudentID, StudentName 
FROM RankedStudents 
WHERE RowNum % 2 = 0;

```

```sql
-- Oracle / MySQL syntax using MOD()
SELECT StudentID 
FROM (
    SELECT StudentID, ROW_NUMBER() OVER (ORDER BY StudentID) AS RowNum 
    FROM Student
) t
WHERE MOD(t.RowNum, 2) = 0;

```

---

## 22. What is a View?

A **View** is a virtual table defined by an underlying SQL query. It does not store physical data on disk (unless it is an *Indexed/Materialized View*); it dynamically runs the underlying query whenever invoked.

### Primary Advantages

* **Security & Abstraction**: Exposes only non-sensitive columns or filtered rows to specific users without giving them access to base tables.
* **Query Simplification**: Encapsulates complex joins, aggregations, and window functions into a simple `SELECT * FROM vw_Name`.
* **Consistency**: Standardizes business logic across reports and microservices.

```sql
CREATE VIEW vw_ActiveEmployees AS
SELECT EmployeeID, FirstName, LastName, DepartmentID
FROM Employees
WHERE IsActive = 1;

-- Usage
SELECT * FROM vw_ActiveEmployees WHERE DepartmentID = 10;

```

---

## 23. User-Defined Functions (Scalar, Inline Table-Valued, Multi-Statement Table-Valued)

SQL Server provides three core categories of User-Defined Functions (UDFs):

### 1. Scalar Function

Accepts input parameters and returns a **single scalar value** (e.g., `int`, `varchar`).

* *Warning*: Can cause performance issues when used in `WHERE` or `SELECT` clauses over large tables due to row-by-row (RBAR) execution.

```sql
CREATE FUNCTION dbo.CalculateTax (@Salary DECIMAL(18,2))
RETURNS DECIMAL(18,2)
AS
BEGIN
    RETURN (@Salary * 0.18);
END;

```

### 2. Inline Table-Valued Function (ITVF)

Returns a table data type defined by a single inline `SELECT` statement. SQL Server treats this like a parameterized view and inlines it directly into the execution plan.

```sql
CREATE FUNCTION dbo.GetEmployeesByDept (@DeptId INT)
RETURNS TABLE
AS
RETURN (
    SELECT EmployeeID, FirstName, Salary 
    FROM Employees 
    WHERE DepartmentID = @DeptId
);

```

### 3. Multi-Statement Table-Valued Function (MSTVF)

Declares a table structure explicitly, populates it using multiple procedural logic statements (`IF/ELSE`, `WHILE`, inserts), and returns the final table variable.

```sql
CREATE FUNCTION dbo.GetHighEarners()
RETURNS @HighEarners TABLE (EmpId INT, AnnualComp DECIMAL(18,2))
AS
BEGIN
    INSERT INTO @HighEarners
    SELECT EmployeeID, (Salary * 12) FROM Employees WHERE Salary > 100000;
    RETURN;
END;

```

---

## 24. Collation

**Collation** refers to a configuration set in relational databases (server, database, or column level) that defines:

1. **Character Encoding Rules**: Defines how characters are physically represented in storage (code pages).
2. **Comparison & Sorting Rules**: Dictates how text data is sorted and compared.

### Collation Sensitivity Flags

* **CS / CI**: Case Sensitive vs. Case Insensitive (`'A' = 'a'` is TRUE in CI, FALSE in CS).
* **AS / AI**: Accent Sensitive vs. Accent Insensitive (`'e' = 'é'` is TRUE in AI, FALSE in AS).
* **WS / WI**: Width Sensitive vs. Width Insensitive (half-width vs. full-width characters).

```sql
-- Overriding collation directly inside a comparison query
SELECT * FROM Customers 
WHERE LastName COLLATE Latin1_General_CS_AS = 'smith'; 
-- Will NOT match 'Smith' or 'SMITH' due to Case Sensitivity (CS)

```

---

## 25. STUFF vs. REPLACE

| Feature | `STUFF()` | `REPLACE()` |
| --- | --- | --- |
| **Operation Type** | **Position-based** modification | **Value-based** search and substitution |
| **Mechanics** | Deletes a fixed length of characters from a start index and inserts new characters | Scans the entire string for every occurrence of a pattern and replaces it |
| **Syntax** | `STUFF(string, start, length, new_string)` | `REPLACE(string, old_string, new_string)` |

```sql
-- 1. STUFF Example: Mask a phone number based on string position
-- Starts at index 1, deletes 5 characters, inserts '*****'
SELECT STUFF('9876543210', 1, 5, '*****');
-- Output: '*****43210'

-- 2. REPLACE Example: Find every instance of a target substring
SELECT REPLACE('dotnet core and dotnet framework', 'dotnet', '.NET');
-- Output: '.NET core and .NET framework'

```

---

Would you like execution plans or performance optimization tips for any of these SQL questions?
