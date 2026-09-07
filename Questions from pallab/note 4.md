Here are the professional, interview-ready answers covering the full list of .NET, MVC, and database architecture questions.

---

### 26. Different Components in .NET Framework

The .NET platform is composed of distinct execution, runtime, and development layers:

* **Common Language Runtime (CLR):** The core execution engine handling thread management, garbage collection (GC), JIT compilation, code verification, and exception handling.
* **Common Type System (CTS):** Defines data types and operations supported across all languages (C#, VB.NET, F#) ensuring cross-language interoperability.
* **Common Language Specification (CLS):** A set of rules defining language features so that libraries authored in one language can be consumed seamlessly by another.
* **Framework Class Library (FCL / BCL):** The comprehensive library providing reusable classes, interfaces, and namespaces (e.g., `System.IO`, `System.Threading`, `System.Collections`).

```
+-------------------------------------------------------------------+
|               Languages: C#  |  F#  |  VB.NET                     |
+-------------------------------------------------------------------+
|               Common Language Specification (CLS)                 |
+-------------------------------------------------------------------+
|               Framework Class Library (FCL / BCL)                 |
+-------------------------------------------------------------------+
|                  Common Type System (CTS)                         |
+-------------------------------------------------------------------+
|             Common Language Runtime (CLR) Engine                  |
|  [ Garbage Collector ]   [ JIT Compiler ]   [ Security Engine ]   |
+-------------------------------------------------------------------+
|                         Operating System                          |
+-------------------------------------------------------------------+

```

---

### 27. Managed vs. Unmanaged Code

* **Managed Code:** Code compiled to Intermediate Language (IL) that runs under the direct supervision of the CLR. The runtime provides automated memory management, type safety checking, buffer-overflow bounds checks, and exception management.
* **Unmanaged Code:** Code compiled directly into machine architecture binaries (e.g., C/C++, Win32 APIs, COM objects) that runs straight on the OS. Developers bear full responsibility for manual allocation/deallocation (`malloc`/`free`) and lifecycle cleanup.

---

### 28. DbContext and DbSet (Entity Framework)

* **`DbContext`:** The central unit-of-work and repository abstraction in Entity Framework. It orchestrates the database connection, builds the domain model via `OnModelCreating`, tracks entity state mutations via the Change Tracker, and translates queries into SQL.
* **`DbSet<TEntity>`:** A collection property representing an individual database table or view within the `DbContext`. It provides LINQ query semantics (`IQueryable<TEntity>`) and exposes CRUD operations (`Add`, `Remove`, `Update`).

```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    public DbSet<Product> Products { get; set; } // Maps to Products table
}

```

---

### 29. AsNoTracking

`AsNoTracking()` is an Entity Framework extension method for queries executed solely for **read-only** workloads.

By default, EF stores entity snapshots in its internal `ChangeTracker` cache to monitor mutations. Calling `.AsNoTracking()` bypasses change-tracking bookkeeping entirely, reducing memory footprints and CPU cycles during data ingestion and API queries.

```csharp
// High-performance read: No ChangeTracker overhead
var activeProducts = await context.Products
    .AsNoTracking()
    .Where(p => p.IsActive)
    .ToListAsync();

```

---

### 30. Custom Attribute

Custom attributes are classes inheriting from `System.Attribute` that append declarative metadata to code artifacts (assemblies, classes, methods, or properties). This metadata is inspected at runtime using Reflection.

```csharp
[AttributeUsage(AttributeTargets.Method, Inherited = false)]
public class AuditLogAttribute : Attribute
{
    public string Operation { get; }
    public AuditLogAttribute(string operation) => Operation = operation;
}

// Consuming custom attribute
public class OrderService
{
    [AuditLog("ProcessOrder")]
    public void PlaceOrder(int orderId) { /* Execution */ }
}

```

---

### 31. Difference Between IActionResult and HttpResponseMessage

* **`IActionResult` (ASP.NET Core / Modern MVC):** An abstraction interface returning action results. It decouples the payload from HTTP status formatting, deferring HTTP generation to specialized result types (e.g., `OkObjectResult`, `NotFoundResult`). Facilitates unit testing without mocking HTTP pipelines.
* **`HttpResponseMessage` (ASP.NET Web API 2 / Legacy):** A raw HTTP response message holding low-level properties (`StatusCode`, `Headers`, `HttpContent`). Requires manual assembly of HTTP transport constructs, making automated unit testing complex.

---

### 32. Difference Between Stored Procedure and Function

| Feature | Stored Procedure (SP) | User-Defined Function (UDF) |
| --- | --- | --- |
| **Return Value** | Returns zero, single values, or multiple result sets. | Must return a single scalar value or a table result. |
| **Usage Context** | Invoked independently via `EXECUTE`. Cannot be joined. | Can be used inline inside `SELECT`, `WHERE`, and `JOIN` clauses. |
| **Data Manipulation** | Fully supports DML (`INSERT`, `UPDATE`, `DELETE`). | Strictly read-only; cannot alter database state or mutate tables. |
| **Transactions** | Allows `BEGIN TRAN`, `COMMIT`, and `ROLLBACK`. | Disallows transaction blocks. |

---

### 33. Attribute Routing

Attribute routing uses annotations directly on controllers and action methods to map URI endpoints, offering finer control than convention-based route tables (`MapRoute`).

```csharp
[ApiController]
[Route("api/v1/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet("{id:int}")] // GET api/v1/orders/42
    public IActionResult GetById(int id) => Ok();

    [HttpGet("customer/{customerId:guid}/history")] // GET api/v1/orders/customer/{guid}/history
    public IActionResult GetHistory(Guid customerId) => Ok();
}

```

---

### 34. Difference Between ActionResult and ViewResult

* **`ActionResult`:** The abstract base class (and generic implementation `ActionResult<T>`) representing any HTTP outcome produced by an action method (e.g., `JsonResult`, `ViewResult`, `RedirectToActionResult`, `StatusCodeResult`).
* **`ViewResult`:** A specialized derivative class of `ActionResult` that renders an HTML view (`.cshtml`) back to the client using the Razor View Engine.

---

### 35. Self Join

A **Self Join** is a regular join operation where a table is joined with itself. It is used when an entity maintains a hierarchical or recursive foreign key relationship pointing to its own primary key (such as an organizational chart or nested category tree).

```sql
SELECT 
    Emp.FirstName AS Employee, 
    Mgr.FirstName AS Manager
FROM Employees Emp
LEFT JOIN Employees Mgr ON Emp.ManagerId = Mgr.EmployeeId;

```

---

### 36. Decorators (Decorator Pattern)

The **Decorator Pattern** is a structural design pattern that dynamically attaches additional responsibilities and behaviors to an object without modifying its source code or relying on inheritance explosions.

```text
[ Client Call ] ──> [ LoggingDecorator ] ──> [ CachingDecorator ] ──> [ ConcreteService ]

```

```csharp
public interface IOrderService { void Place(); }

public class OrderService : IOrderService 
{
    public void Place() => Console.WriteLine("Order placed.");
}

public class LoggingOrderDecorator : IOrderService
{
    private readonly IOrderService _inner;
    public LoggingOrderDecorator(IOrderService inner) => _inner = inner;

    public void Place()
    {
        Console.WriteLine("Log: Initiating order placement...");
        _inner.Place();
    }
}

```

---

### 37. Indexers

An **indexer** allows instances of a class or struct to be accessed using array-indexing syntax (`[]`). It is declared using the `this` keyword with parameters.

```csharp
public class DataStore<T>
{
    private readonly T[] _store = new T[10];

    // Indexer declaration
    public T this[int index]
    {
        get => _store[index];
        set => _store[index] = value;
    }
}

// Usage
var cache = new DataStore<string>();
cache[0] = "Cached Item";

```

---

### 38. Lifecycle of ASP.NET MVC

The MVC execution pipeline processes an incoming HTTP request through these stages:

```text
  [ Incoming HTTP Request ]
              │
              ▼
    1. Routing (UrlRoutingModule matches route handlers)
              │
              ▼
    2. Controller Initialization (IControllerFactory / DI creates controller)
              │
              ▼
    3. Action Execution:
       ├── Model Binding & Data Validation
       ├── Action Filters (OnActionExecuting)
       ├── Action Method Execution
       └── Action Filters (OnActionExecuted)
              │
              ▼
    4. Result Execution:
       ├── Result Filters (OnResultExecuting)
       ├── View Engine Selection & Razor Rendering
       └── Result Filters (OnResultExecuted)
              │
              ▼
  [ HTTP Response Generated ]

```

---

### 39. Exception Handling in SQL Functions

User-Defined Functions (UDFs) in SQL Server **do not support `TRY...CATCH` blocks**. If an unhandled runtime error occurs inside a function, execution terminates and aborts the enclosing query statement.

To handle errors robustly in database logic, either:

1. Validate inputs proactively within the function using conditional validation (`IF...ELSE`, `ISNULL`, `COALESCE`, `NULLIF`).
2. Move complex transactional and error handling logic into a **Stored Procedure** where full `BEGIN TRY ... BEGIN CATCH` blocks are supported.

---

### 40. How Many Non-Clustered Indexes Can Be Supported?

In modern SQL Server architectures (SQL Server 2008 and all subsequent versions):

* **Clustered Index:** Maximum of **1** per table (determines the physical sort order of data pages).
* **Non-Clustered Indexes:** Maximum of **999** per table.
*(Note: Older legacy engines such as SQL Server 2005 supported up to 249 non-clustered indexes).*

---

### 41. Copy Data from Two Identical Tables Without Using Any Loop

Data migration between tables with matching schemas can be handled using set-based operations:

```sql
-- Direct Set-Based Bulk Insert
INSERT INTO TableB
SELECT * FROM TableA;

```

For conditional operations, data synchronization, or reconciliation, use `MERGE`:

```sql
MERGE INTO TableB AS Target
USING TableA AS Source
ON Target.Id = Source.Id
WHEN NOT MATCHED THEN
    INSERT VALUES (Source.Id, Source.Name, Source.CreatedAt);

```

---

### 42. Eager Loading and Lazy Loading

* **Eager Loading:** Related child entities are loaded from the database alongside the parent entity in a single query using `.Include()` / `.ThenInclude()`. Avoids iterative roundtrips.
* **Lazy Loading:** Child entity data is deferred until explicitly referenced in code. Uses database queries on demand (requires `virtual` navigation properties or proxies).

```text
Eager Loading:  [ 1 SQL Query (JOIN) ] ───────> All Parent & Child Entities Loaded

Lazy Loading:   [ 1 SQL Query for Parent ] ───> Iteration triggers N extra queries (N+1 Problem!)

```

```csharp
// Eager Loading: Explicit JOIN via Include()
var orders = context.Orders.Include(o => o.OrderLines).ToList();

// Lazy Loading: Accessing property triggers database roundtrip
var order = context.Orders.Find(1);
var lines = order.OrderLines; // Triggers secondary query at runtime

```

---

### 43. web.config

`web.config` is an XML-based configuration file used by legacy ASP.NET Framework applications running under IIS. It centralized:

* Database connection strings (`<connectionStrings>`)
* Session state management, authentication, and authorization settings
* Custom error rules, HTTP modules, and handlers
* Assembly bindings and compilation parameters

*(In modern ASP.NET Core, `web.config` is replaced by cross-platform JSON configurations like `appsettings.json` and code-based configuration pipelines, though a minimal `web.config` can still be used for IIS reverse proxy hosting).*

---

### 44. How .NET Code Gets Compiled to Machine Code

Compilation in the .NET CLR is a two-step process:

```text
[ C# / VB Source Code ]
          │
          ▼  Roslyn Compiler (csc)
[ Assembly DLL / EXE (CIL / MSIL + Metadata) ]
          │
          ▼  Execution on Target Machine
[ CLR Class Loader & JIT Compiler (RyuJIT) ]
          │
          ▼
[ Native Machine Code (x86 / x64 / ARM) ]

```

1. **Build Time:** The language compiler (Roslyn) translates source code into Common Intermediate Language (CIL/IL) and packages it with structural metadata into portable assemblies (`.dll` or `.exe`).
2. **Runtime Execution:** When the program runs, the CLR's **Just-In-Time (JIT) Compiler** compiles the intermediate IL instructions into native machine code tailored to the host processor architecture.

---

### 45. Generics

Generics introduce parameterized types to the .NET framework (`List<T>`, `Dictionary<TKey, TValue>`). They allow classes, interfaces, and methods to be defined with placeholders for data types until instantiated by consuming code.

* **Compile-Time Type Safety:** Prevents runtime casting errors by validating types during compilation.
* **Performance Optimization:** Eliminates boxing and unboxing allocations for value types.
* **Code Reusability:** Provides a single, maintainable code implementation for multiple types.

```csharp
public class Result<T>
{
    public bool IsSuccess { get; set; }
    public T Data { get; set; }
}

```

---

### 46. How Allocation and Deallocation Happens in Memory

```text
MANAGED HEAP:
┌────────────────────┬────────────────────┬────────────────────┬────────────────────┐
│    Generation 0    │    Generation 1    │    Generation 2    │ Large Object Heap  │
│ (Ephemeral Objects)│ (Survivors of Gen0)│ (Long-Lived State) │   (LOH >= 85KB)    │
└────────────────────┴────────────────────┴────────────────────┴────────────────────┘

```

#### 1. Allocation

* **Stack:** Allocated instantaneously using a moving stack pointer. Used for method frames, local primitive types, and object references. Deallocated immediately as execution exits scope.
* **Heap:** Allocated via the CLR using a Next-Object-Pointer. Fast, contiguous allocation for reference types. Objects $\ge$ 85,000 bytes go directly to the Large Object Heap (LOH).

#### 2. Deallocation (Garbage Collection)

The GC runs periodically under memory pressure, when allocations exceed generation thresholds, or via `GC.Collect()`:

1. **Marking:** Traverses active roots (CPU registers, local references, static pointers) and marks all reachable objects in the dependency graph.
2. **Sweeping:** Identifies unmarked objects as dead memory.
3. **Compaction:** Shifts surviving live objects contiguously to eliminate memory fragmentation and adjusts internal pointer references.
4. **Aging:** Objects surviving collections promote through generational tiers ($Gen\ 0 \rightarrow Gen\ 1 \rightarrow Gen\ 2$). Gen 2 collections represent complete, full-heap GC sweeps.

---

Would you like to walk through standard follow-up interview questions on any of these topics, such as debugging high GC overhead or optimizing EF Core query plans?
