# Senior .NET / Full-Stack Engineer Interview Preparation Guide (Part 4)
**Candidate Persona:** Senior Software Engineer (8+ Years Experience in ASP.NET Core / MVC, Entity Framework Core, SQL Server, and CLR Internals)

---

## 26. Different Components in .NET Framework
**Interview Answer:**
> "The .NET architecture comprises two foundational runtime pillars along with higher-level application models:
>
> 1. **Common Language Runtime (CLR):** The execution engine that handles managed code. Key sub-components include:
>    - **JIT Compiler (Just-In-Time):** Converts Common Intermediate Language (CIL/MSIL) into native CPU machine instructions.
>    - **Garbage Collector (GC):** Automates memory allocation and reclamation across generations (Gen 0, 1, 2, and LOH/POH).
>    - **Type System & Security Engine:** Enforces the Common Type System (CTS), Common Language Specification (CLS), type safety, and thread synchronization.
> 2. **Base Class Library (BCL) / Framework Class Library (FCL):** A comprehensive collection of reusable, type-safe APIs for primitive types (`System`), collections, threading, I/O, cryptography, and networking.
> 3. **Application Stacks:** Framework layers tailored for specific workloads—such as ASP.NET Core for web APIs/applications, EF Core for ORM data persistence, and Windows presentation technologies (WPF, WinUI)."

---

## 27. Managed and Unmanaged Code
**Interview Answer:**
> "The core distinction lies in who manages execution, memory, and lifecycle:
>
> - **Managed Code:** Code written in high-level CLI-compliant languages (C#, F#) that compiles to MSIL and executes directly under the CLR. The runtime oversees memory allocation, garbage collection, bounds checking, and structured exception handling.
> - **Unmanaged Code:** Code that compiles directly into platform-specific machine code and executes outside the CLR runtime umbrella—such as native C/C++ libraries, Win32 APIs, or COM objects. Developers must manually manage memory allocation (`malloc`/`free`) and native resource handles.
> - **Interoperability:** In .NET, we bridge the boundary using **Platform Invocation Services (P/Invoke)** with `[DllImport]` or COM Interop, safely wrapping raw OS pointers inside `SafeHandle` instances."

---

## 28. `DbContext` and `DbSet` (Entity Framework / EF Core)
**Interview Answer:**
> "In Entity Framework Core, these two classes represent the core patterns of enterprise data access:
>
> - **`DbContext` (Unit of Work & Gateway):** Coordinates database connectivity, manages transactions, caches query pipelines, and tracks changes across entity instances via its internal `ChangeTracker`. When `.SaveChangesAsync()` is called, it packages all dirty tracked entities into a single optimized transactional database round-trip.
> - **`DbSet<TEntity>` (Repository Pattern):** Represents the in-memory collection mapping directly to a database table or view. It implements `IQueryable<T>`, enabling developers to write strongly typed LINQ queries that the EF Core query engine translates into parameter-driven SQL queries.
>
> In modern ASP.NET Core architectures, `DbContext` is registered with a **Scoped lifetime** so each HTTP request gets an isolated Unit of Work that is cleanly disposed of at the end of the pipeline."

---

## 29. `AsNoTracking` in Entity Framework
**Interview Answer:**
> "`AsNoTracking()` is an `IQueryable` extension method that instructs the EF Core `ChangeTracker` not to monitor the retrieved entity instances for mutations.
>
> **Why it is critical for performance:**
> - When change tracking is enabled (the default), EF Core takes snapshot copies of property values in memory and attaches entities to the context. This adds memory overhead and increases GC pressure.
> - By applying `.AsNoTracking()`, EF Core skips creating identity map snapshots and tracking dictionaries.
>
> **Best Practice:** Apply `.AsNoTracking()` (or `.AsNoTrackingWithIdentityResolution()`) to all read-only queries, reporting dashboards, and GET endpoints. Enable tracking exclusively on entities you intend to modify, update, or delete during that transaction."

---

## 30. Custom Attributes
**Interview Answer:**
> "Custom attributes provide a declarative way to associate metadata with program elements (classes, methods, properties, assemblies).
>
> To create one:
> 1. Inherit from `System.Attribute`.
> 2. Annotate the class with `[AttributeUsage]` to specify valid targets (e.g., `AttributeTargets.Method`) and whether multiple instances are permitted (`AllowMultiple`).
> 3. Inspect metadata at runtime using Reflection (`MemberInfo.GetCustomAttributes()`) or compile-time Roslyn Source Generators.
>
> ```csharp
> [AttributeUsage(AttributeTargets.Method, Inherited = false)]
> public sealed class AuditLogAttribute : Attribute
> {
>     public string ActionName { get; }
>     public AuditLogAttribute(string actionName) => ActionName = actionName;
> }
> ```
>
> In enterprise systems, custom attributes drive cross-cutting concerns like custom authorization filters, validation decorators, and audit logging."

---

## 31. Difference between `IActionResult` and `HttpResponseMessage`
**Interview Answer:**
> "This comparison highlights the evolution between legacy WCF/ASP.NET Web API and modern unified ASP.NET Core:
>
> - **`HttpResponseMessage` (Legacy Web API / HttpClient):** A low-level HTTP transport object belonging to `System.Net.Http`. It explicitly wraps raw HTTP response headers, status codes, and message bodies. While powerful, it tightly couples controller actions to raw HTTP primitives and complicates unit testing.
> - **`IActionResult` (ASP.NET Core / MVC):** A high-level contract defining an action result. It encapsulates content negotiation, formatting, and status code generation into concrete types (`OkObjectResult`, `NotFoundResult`, `ViewResult`). 
>
> When returning `IActionResult` (or generic `ActionResult<T>`), the framework defers serialization to formatters (`System.Text.Json`), keeping actions easy to unit-test without needing to mock low-level HTTP headers."

---

## 32. Difference between Stored Procedure and Function in SQL
**Interview Answer:**
> "Both are precompiled database objects, but they have distinct usage patterns and transactional capabilities:
>
> | Feature | Stored Procedure (`PROCEDURE`) | User-Defined Function (`FUNCTION`) |
> |---|---|---|
> | **Execution Context** | Invoked independently using `EXEC` / `EXECUTE` | Invoked directly inline inside `SELECT`, `WHERE`, or `JOIN` clauses |
> | **Return Values** | Returns 0, single/multiple result sets, or status integers | **Must** return a single scalar value or a table result (`TVF`) |
> | **DML / State Changes** | Fully permitted (`INSERT`, `UPDATE`, `DELETE`) | Strictly **read-only**; cannot modify persistent database state |
> | **Transactions** | Full transaction control (`BEGIN TRAN`, `COMMIT`, `ROLLBACK`) | Cannot manage transactions |
> | **Performance Tip** | Excellent for batch writes and complex business logic | Scalar functions can cause performance degradation on large tables by forcing row-by-row execution (RBAR); prefer Inline Table-Valued Functions (iTVFs) |"

---

## 33. Attribute Routing
**Interview Answer:**
> "Attribute routing uses C# attributes placed directly on controller classes and action methods to map incoming HTTP requests to specific endpoints.
>
> ```csharp
> [ApiController]
> [Route("api/v{version:apiVersion}/[controller]")]
> public class OrdersController : ControllerBase
> {
>     [HttpGet("{id:int}")]
>     public async Task<IActionResult> GetById(int id) { ... }
> }
> ```
>
> **Advantages over Convention-Based Routing:**
> 1. **Co-location:** Route definitions live directly alongside the action logic.
> 2. **Inline Route Constraints:** Enforces constraints directly in the route template (e.g., `:int`, `:guid`, `:min(1)`), rejecting invalid requests before model binding occurs.
> 3. **API Versioning:** Simplifies REST API design, hierarchical sub-resources (`orders/{orderId}/items/{itemId}`), and URI versioning schemes."

---

## 34. Difference between `ActionResult` and `ViewResult`
**Interview Answer:**
> "The relationship between these two is built on inheritance and polymorphism:
>
> - **`ActionResult`:** An abstract base class (implementing `IActionResult`) representing the result of an action method. It serves as the general return type for various outcomes—including redirects (`RedirectToActionResult`), file downloads (`FileContentResult`), JSON payloads (`JsonResult`), and HTML views.
> - **`ViewResult`:** A concrete subclass of `ActionResult` designed specifically for server-side HTML rendering. It looks up a Razor `.cshtml` view file, binds a model to it, renders the view engine pipeline, and writes an HTML response back to the client.
>
> In modern REST APIs, we generally return `IActionResult` or `ActionResult<T>` rather than `ViewResult`."

---

## 35. Self Join
**Interview Answer:**
> "A Self Join is an operation where a database table is joined with itself. It is primarily used to query hierarchical, recursive, or graph-like relationships stored within a single table.
>
> The query requires using different table aliases for the same table:
>
> ```sql
> SELECT 
>     E.EmployeeId, 
>     E.FullName AS EmployeeName, 
>     M.FullName AS ManagerName
> FROM Employees E
> LEFT JOIN Employees M ON E.ReportsTo = M.EmployeeId;
> ```
>
> For multi-level hierarchies (such as full organizational charts or category trees), self joins are often paired with **Recursive Common Table Expressions (Recursive CTEs)** to traverse relationships without hardcoded nesting."

---

## 36. Decorator Pattern
**Interview Answer:**
> "The Decorator Pattern is a structural Gang of Four (GoF) design pattern that dynamically attaches additional responsibilities and behavior to an object without modifying its source code or resorting to class inheritance.
>
> **Implementation in C#:**
> Both the concrete service and the decorator implement the same interface. The decorator wraps the concrete instance and delegates method calls to it, adding logic before or after:
>
> ```csharp
> public interface IOrderService { Task ProcessAsync(Order order); }
>
> public class LoggingOrderServiceDecorator : IOrderService
> {
>     private readonly IOrderService _inner;
>     private readonly ILogger<LoggingOrderServiceDecorator> _logger;
>
>     public LoggingOrderServiceDecorator(IOrderService inner, ILogger<LoggingOrderServiceDecorator> logger)
>     {
>         _inner = inner;
>         _logger = logger;
>     }
>
>     public async Task ProcessAsync(Order order)
>     {
>         _logger.LogInformation("Starting order processing: {Id}", order.Id);
>         await _inner.ProcessAsync(order);
>         _logger.LogInformation("Order completed: {Id}", order.Id);
>     }
> }
> ```
>
> This pattern is ideal for cross-cutting concerns (logging, caching, metrics, circuit breakers) and is natively supported in .NET 8 via `services.Decorate<TInterface, TDecorator>()` (via Scrutor) or standard factory delegates."

---

## 37. Indexers in C#
**Interview Answer:**
> "An indexer allows instances of a class or struct to be indexed just like an array, using array bracket notation (`instance[index]`).
>
> It is declared using the `this` keyword with parameter signatures:
>
> ```csharp
> public class CacheStore<T>
> {
>     private readonly Dictionary<string, T> _store = new();
>     
>     public T this[string key]
>     {
>         get => _store.TryGetValue(key, out var val) ? val : default!;
>         set => _store[key] = value;
>     }
> }
> ```
>
> Under the hood, the C# compiler emits an `Item` property with `get_Item` and `set_Item` accessor methods. Indexers can be overloaded by type (e.g., accessing an element via integer ID or string key)."

---

## 38. ASP.NET MVC Request Lifecycle
**Interview Answer:**
> "The ASP.NET MVC request lifecycle follows an ordered pipeline from raw network socket to client response:
>
> 1. **Routing:** Incoming HTTP requests pass through the routing middleware/engine (`EndpointRoutingMiddleware`), matching the URL template to a target Controller and Action.
> 2. **Controller Initialization:** The `DefaultControllerFactory` activates the controller instance, resolving its dependencies via the DI container (`IServiceProvider`).
> 3. **Action Execution Pipeline:**
>    - **Authorization Filters:** Confirms credentials and role claims (short-circuits if unauthorized).
>    - **Model Binding & Validation:** Maps query strings, route parameters, headers, and JSON/form payloads onto strongly typed method parameters, populating `ModelState`.
>    - **Action Filters:** Executes pre-action logic (`OnActionExecuting`).
>    - **Action Invocation:** The action method executes and produces an `IActionResult`.
>    - **Action Filters Post:** Executes post-action logic (`OnActionExecuted`).
> 4. **Result Execution:**
>    - **Result Filters:** Pre/post interceptors around result processing.
>    - **View Engine / Formatter:** If returning a `ViewResult`, the Razor View Engine locates, compiles, and renders the view. If returning an API model, an Output Formatter (e.g., `System.Text.Json`) serializes the object to the response stream.
> 5. **Response Return:** The completed response stream is transmitted back down the middleware pipeline to the client."

---

## 39. Exception Handling in SQL Functions
**Interview Answer:**
> "In SQL Server, **`TRY...CATCH` blocks are strictly prohibited inside User-Defined Functions (both Scalar and Table-Valued Functions)**.
>
> The SQL Server engine enforces this restriction because functions must be deterministic, non-state-altering, and safe to execute repeatedly within query projections and parallelized query plans.
>
> **How to handle errors in functions:**
> 1. **Defensive Logic:** Validate inputs using guard clauses with `CASE`, `ISNULL()`, or `NULLIF()` (e.g., avoiding division-by-zero via `SELECT a / NULLIF(b, 0)`).
> 2. **Refactoring to Stored Procedures:** If execution requires robust `BEGIN TRY...BEGIN CATCH` blocks and `THROW`/`RAISERROR` mechanisms, move the logic to a Stored Procedure instead."

---

## 40. How Many Non-Clustered Indexes Can a Table Support?
**Interview Answer:**
> "In modern SQL Server (from SQL Server 2008 through SQL Server 2022):
>
> - A single table supports up to **999 Non-Clustered Indexes** (along with at most **1 Clustered Index**).
> - (In legacy SQL Server 2000/2005, the limit was 249 non-clustered indexes).
>
> **Senior Performance Warning:**
> Even though the engine supports 999 non-clustered indexes, creating dozens on a single production table is generally an anti-pattern. Every write operation (`INSERT`, `UPDATE`, `DELETE`) must synchronously update and write to all associated non-clustered B-Trees, which can significantly degrade write throughput and increase storage footprint. Aim for lean, covering indexes with `INCLUDE` clauses tailored to your primary query paths."

---

## 41. Copy Data from Two Identical Tables Without Using Any Loop
**Interview Answer:**
> "In relational databases, row-by-row looping (`WHILE` loops or cursors) is an anti-pattern (RBAR - Row By Agonizing Row). We leverage set-based SQL operations instead:
>
> ```sql
> INSERT INTO TableB (Id, Name, Balance, CreatedAt)
> SELECT Id, Name, Balance, CreatedAt
> FROM TableA;
> ```
>
> **High-Performance Variations for Large Datasets:**
> - If `TableB` doesn't exist yet, use `SELECT * INTO TableB FROM TableA;` to execute a minimally logged bulk operation.
> - If transferring millions of records between tables in a .NET application, use `SqlBulkCopy` rather than issuing row-by-row EF Core inserts."

---

## 42. Eager Loading vs. Lazy Loading
**Interview Answer:**
> "These are two data-loading strategies in Entity Framework Core for retrieving related navigation entities:
>
> - **Eager Loading:** Related entities are queried upfront in a single SQL query using the `.Include()` and `.ThenInclude()` operators:
>   ```csharp
>   var orders = context.Orders.Include(o => o.OrderItems).ToList();
>   ```
>   *Pros:* Predictable performance; eliminates the N+1 query problem.
>
> - **Lazy Loading:** Related entities are deferred and loaded on the fly only when the navigation property is first accessed in code (requires `virtual` navigation properties and `Microsoft.EntityFrameworkCore.Proxies`):
>   ```csharp
>   var order = context.Orders.First();
>   var count = order.OrderItems.Count; // Fires an implicit, separate SQL query behind the scenes
>   ```
>   *Cons:* Can cause severe performance issues by triggering the **N+1 query problem** inside loops, creating dozens or hundreds of unexpected database round-trips.
>
> **Best Practice:** Default to **Eager Loading** or explicit **Projection** (`.Select(x => new Dto { ... })`) for clean, predictable database interactions."

---

## 43. `web.config`
**Interview Answer:**
> "`web.config` is an XML-based configuration file historically used in the legacy .NET Framework and IIS web server ecosystem.
>
> Key responsibilities included:
> - Application settings (`<appSettings>`) and database connection strings (`<connectionStrings>`).
> - IIS pipeline handler and module registrations (`<system.webServer>`).
> - Authentication, session state, and diagnostic tracing.
>
> **Evolution to Modern .NET:**
> In modern ASP.NET Core, `web.config` has been largely superseded by **`appsettings.json`** and the unified, extensible `Microsoft.Extensions.Configuration` provider system (supporting JSON, Environment Variables, Azure Key Vault, and Command Line arguments). In modern deployments, a minimal `web.config` is used only when reverse-proxying behind Windows IIS using the `AspNetCoreModuleV2` (ANCM)."

---

## 44. How .NET Code Gets Compiled in the Machine
**Interview Answer:**
> "The .NET compilation lifecycle operates across two primary stages:
>
> 1. **Compilation Phase (Build Time):**
>    - The Roslyn compiler (`csc`) checks syntax, analyzes types, and compiles `.cs` source files into **Common Intermediate Language (CIL/MSIL)** bytecodes along with type metadata.
>    - The output is packaged into a managed portable executable (`.dll` or `.exe`).
>
> 2. **Execution Phase (Runtime via the CLR):**
>    - When the application starts, the operating system launches the CLR runtime engine.
>    - As methods are called, the **JIT (Just-In-Time) Compiler** dynamically translates the machine-agnostic CIL instructions into native assembly instructions for the host CPU architecture (x64, ARM64).
>    - Modern .NET uses **Tiered Compilation**:
>      - *Tier 0 (Quick JIT):* Compiles code rapidly without heavy optimizations to minimize initial application startup time.
>      - *Tier 1:* Hot-path methods are identified and recompiled using Profile-Guided Optimization (PGO), vectorization (SIMD), and loop unrolling for maximum runtime throughput.
>    - Modern .NET also supports **Native AOT (Ahead-of-Time)**, which compiles C# code directly into platform-specific native machine code ahead of deployment, skipping CIL and JIT entirely."

---

## 45. Generics in C#
**Interview Answer:**
> "Generics (introduced in C# 2.0) allow developers to author classes, interfaces, and methods parameterized by type placeholders (`<T>`).
>
> **Core Advantages:**
> 1. **Compile-Time Type Safety:** The compiler verifies data types, preventing runtime `InvalidCastException` errors.
> 2. **Zero Boxing/Unboxing:** Value types stored inside generic containers (`List<int>`) remain on the stack without being boxed to the heap.
> 3. **Code Reuse:** A single generic implementation (`Repository<T>`) can support hundreds of entity types.
>
> **CLR Internals & Generic Constraints:**
> The CLR generates specialized native machine code for each unique value type (e.g., `List<int>` vs. `List<double>`), while sharing a single canonical native code implementation across all reference types (`List<string>`, `List<Order>`) because pointers share identical machine representations. Generics also support compile-time constraints: `where T : class`, `where T : new()`, `where T : struct`, or `where T : IEntity`."

---

## 46. How Memory Allocation and Deallocation Happen in Memory
**Interview Answer:**
> "Memory management in .NET is divided between the **Thread Stack** and the **Managed Heap**:
>
> 1. **Stack Allocation & Deallocation:**
>    - Stores local variables, parameter values, and value types (`struct`, `int`).
>    - Operates on a Last-In, First-Out (LIFO) model tied to execution scope.
>    - Allocation is fast—the CPU merely increments the stack pointer register. Deallocation is automatic and immediate as the method returns and the stack frame unwinds.
>
> 2. **Heap Allocation:**
>    - Stores reference types (`class`, `string`, boxed objects).
>    - Managed by the CLR. When an object is instantiated via `new`, the memory allocator finds contiguous space on the managed heap and increments an allocation pointer.
>    - Small objects are allocated in **Generation 0**. Large objects ($\ge 85,000$ bytes) bypass Gen 0 and go directly to the **Large Object Heap (LOH)** to avoid high memory compaction costs.
>
> 3. **Heap Deallocation (Garbage Collection):**
>    - Deallocation is **non-deterministic** and automated by the Garbage Collector.
>    - **Phase 1: Mark:** The GC pauses application threads (during full sweeps) and traverses the object graph starting from GC Roots (CPU registers, active local variables, static fields). Objects that are reachable are marked as live.
>    - **Phase 2: Plan & Sweep:** Unreachable objects are identified for reclamation.
>    - **Phase 3: Compact:** Fragmented free spaces are consolidated by sliding surviving live objects together in memory, and object references are updated to reflect the new memory addresses. Surviving objects are promoted to older generations (Gen 0 $\rightarrow$ Gen 1 $\rightarrow$ Gen 2)."
