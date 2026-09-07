### 1. Difference between .NET 4.x, .NET Core, and .NET 5+

* **.NET Framework (up to 4.8.x):** Windows-only, monolithic framework tightly coupled to the Windows OS and IIS. Updates are system-wide (machine-wide GAC).
* **.NET Core (1.0 – 3.1):** A complete, high-performance, cross-platform (Windows, Linux, macOS) rewrite. Modular (delivered via NuGet packages), supports side-by-side installations, and built ground-up with dependency injection and asynchronous request pipelines.
* **.NET 5 / 6 / 7 / 8+:** The unified successor that dropped the "Core" branding. It brings Mono/Xamarin and .NET Core together into a single BCL (Base Class Library), offering unified runtime capabilities across cloud, desktop, mobile, IoT, and WebAssembly (Blazor).

```
.NET Framework 4.x  ──> [ Windows Only, Legacy / Maintenance ]
.NET Core 1.x - 3.x ──> [ Cross-Platform Rewrite ] ───┐
Xamarin / Mono      ──> [ Mobile / Embedded ]     ───┴──> .NET 5 / 6 / 7 / 8+ (Unified Platform)

```

---

### 2. What is IL and what is JIT?

* **Intermediate Language (IL / CIL / MSIL):** A CPU-independent, intermediate instruction set into which high-level .NET languages (C#, VB.NET, F#) are compiled by Roslyn.
* **Just-In-Time (JIT) Compiler:** The runtime component inside the CLR that compiles IL into machine-specific native code (`x86`, `x64`, `ARM`) right before execution.

```
+-----------+      Roslyn Compiler       +-------------------+
|  C# Code  |  ───────────────────────>  | Intermediate (IL) |
+-----------+                            +-------------------+
                                                   │
                                            JIT Compiler (CLR)
                                                   ▼
                                         +-------------------+
                                         | Native Machine    |
                                         | Code (x86/x64)    |
                                         +-------------------+

```

Types of JIT compilers historically include:

1. **Normal JIT:** Compiles methods on demand the first time they are called and caches them in memory.
2. **Econo JIT:** Fast compilation with no optimization; clears memory when code is not in use (obsolete in modern runtimes).
3. **Pre-JIT (NGen / Native AOT):** Compiles entire assemblies ahead-of-time into native code before execution. Modern .NET relies heavily on Tiered Compilation and Native AOT.

---

### 3. What is CLR?

The **Common Language Runtime (CLR)** is the execution engine of .NET that manages executing applications. In modern .NET, it is called **CoreCLR**.

**Key Responsibilities:**

* Memory management via the Garbage Collector (GC).
* JIT compilation of IL to machine code.
* Thread management and execution safety.
* Code access security and type verification.
* Structured exception handling.

---

### 4. What is CTS (Common Type System)?

The **Common Type System (CTS)** defines standard data types, object models, and rules enforced across all .NET-supported languages to ensure cross-language interoperability.

* A C# `int` maps to `System.Int32`.
* A VB.NET `Integer` maps to `System.Int32`.
* Because both compile to the identical CTS type, code written in VB.NET can consume libraries written in C# without data-type mismatches.

---

### 5. What is CLS (Common Language Specification)?

The **Common Language Specification (CLS)** is a subset of CTS rules that library authors must follow to make their code consumable by *any* .NET language.

* **Example:** C# is case-sensitive, but VB.NET is case-insensitive. If a public API exposes two methods named `GetData()` and `getdata()`, it violates CLS compliance because a VB.NET project cannot differentiate between them. Marking an assembly with `[assembly: CLSCompliant(true)]` triggers compiler warnings for non-compliant exposed members.

---

### 6. What is MVC?

**Model-View-Controller (MVC)** is an architectural design pattern that separates application concerns into three core components:

* **Model:** Contains application data, business logic, domain entities, and data validation rules.
* **View:** Handles UI rendering and presentation logic (HTML, Razor markup).
* **Controller:** Directs user requests, interacts with models/services, and selects the corresponding view or JSON payload to return.

```
       1. Request
Browser  ───────> [ Controller ] ──> 2. Fetch/Update ──> [ Model / Database ]
   ▲                     │
   │                     │ 3. Passes Model
   │ 4. HTML Render      ▼
   └──────────────── [ View ]

```

---

### 7. Difference between Design Pattern and Architectural Pattern

* **Architectural Pattern:** High-level strategic blueprint that defines the overall structure, subsystems, and global flow of the entire application.
* *Scope:* System-wide / Macro level.
* *Examples:* MVC, Microservices, Event-Driven, Clean/Hexagonal Architecture.


* **Design Pattern:** Low-level tactical solution to recurring localized software design problems.
* *Scope:* Component / Class / Micro level.
* *Examples:* Singleton, Factory, Repository, Strategy, Observer.



---

### 8. Advantages of ASP.NET MVC

* **Separation of Concerns (SoC):** Distinct separation of UI, business logic, and routing makes large projects maintainable.
* **Full Control Over HTML, CSS, & JS:** Clean semantic HTML generation without hidden `__VIEWSTATE` bloat or generated control IDs.
* **Testability:** Controllers are decoupled from the HTTP context via interfaces (`HttpContextBase` or standard ASP.NET Core interfaces), enabling unit testing with mocking frameworks.
* **SEO-Friendly Clean URLs:** Powerful pattern-based routing provides clean URLs by default.
* **Extensible & Pluggable Architecture:** Custom filters, action results, custom model binders, and view engines can be swapped out easily.

---

### 9. What is Model and View?

* **Model:** Encapsulates the domain state and business operations. It can be a domain model (e.g., `Customer`) or a targeted presentation model (`CustomerViewModel`). It carries no knowledge of how it will be rendered.
* **View:** The template layer responsible for turning model data into presentation output for the client (typically `.cshtml` using Razor syntax). It should contain only presentation logic, avoiding data-access calls or heavy business computations.

---

### 10. What are the different types of Views?

1. **Standard View (`.cshtml`):** Regular full-page view bound to an action method.
2. **Partial View (`_Partial.cshtml`):** Reusable component view rendered inside other views; does not use a master layout by default.
3. **Layout View (`_Layout.cshtml`):** Serves as the master template containing boilerplate HTML, navigation, headers, and footers, with placeholder calls like `@RenderBody()` and `@RenderSection()`.
4. **Strongly-Typed View:** Uses the `@model YourNamespace.YourModel` directive, providing compile-time type checking and IntelliSense.
5. **Dynamic / Weakly-Typed View:** Relies on dynamic dictionaries like `ViewBag` or `ViewData` without compile-time safety.

---

### 11. Different types of Partial Views

Partial views can be categorized by how they are invoked and rendered:

1. **Synchronous Static Inclusions:** Rendered at initial page compile time:
* ASP.NET Core: `<partial name="_UserCard" model="Model.User" />` or `@await Html.PartialAsync("_UserCard", Model.User)`.
* Classic MVC: `@Html.Partial("_UserCard", Model.User)`.


2. **Child Action / ViewComponent Partials:** Partials that require their own independent data-fetching logic:
* Classic MVC: `@Html.Action("RecentComments", "Comment")`.
* ASP.NET Core: **View Components** replaced Child Actions for reusable, independent logic: `@await Component.InvokeAsync("RecentComments")`.


3. **AJAX-Loaded Partials:** Loaded asynchronously via JavaScript fetches/XHR to dynamically replace DOM nodes without a full page refresh:
```javascript
fetch('/Orders/GetOrderSummary/42')
    .then(res => res.text())
    .then(html => document.getElementById('summary-container').innerHTML = html);

```



---

### 12. What is a View Engine?

A **View Engine** is the runtime module responsible for parsing template markup, processing embedded server-side instructions, and generating raw HTML strings sent back in the HTTP response body.

* In earlier ASP.NET MVC versions, two primary view engines existed: the **ASPX View Engine** (`WebFormViewEngine`) and the **Razor View Engine** (`RazorViewEngine`).
* In modern .NET Core / .NET 8, the **Razor Engine** is the primary standard.

---

### 13. What is Razor?

**Razor** is a lightweight, view-templating markup syntax that embeds C# directly into HTML using the `@` symbol.

**Key Features:**

* Seamless context switching between markup and C# without requiring explicit closing tags (e.g., `<% %>` from classic ASPX).
* Automatic HTML encoding on dynamic output (mitigating Cross-Site Scripting / XSS attacks).
* Supports blocks, control structures, and inline expressions:
```razor
@if (Model.IsActive)
{
    <p class="status">Welcome, @Model.Name</p>
}

```



---

### 14. Difference between ASP.NET Web Forms vs. ASP.NET MVC 5 vs. ASP.NET Core MVC

| Feature | ASP.NET Web Forms | ASP.NET MVC 5 | ASP.NET Core MVC |
| --- | --- | --- | --- |
| **Architecture** | Event-driven, Page Controller | Model-View-Controller | Model-View-Controller |
| **OS Support** | Windows only (IIS) | Windows only (IIS) | Cross-platform (Win/Linux/Mac) |
| **State Management** | Heavy (`ViewState`, ControlState) | Stateless by default | Stateless by default |
| **Performance** | Slow (large HTML payloads) | High | Highest (optimized Kestrel server) |
| **Dependency Injection** | Not built-in | Third-party containers (Unity, Autofac) | First-class, built-in IoC container |
| **Configuration** | `web.config` (XML) | `web.config` (XML) | `appsettings.json`, environment vars |

---

### 15. Explain MVC Architecture (Execution Flow)

```
[ HTTP Request ]
       │
       ▼
[ Routing Engine ] ────────> Resolves Route (Controller/Action)
       │
       ▼
[ Controller Factory ] ────> Controller Instantiation (resolves DI)
       │
       ▼
[ Action Invoker ] ────────> Runs Action Filters -> Model Binding -> Validation
       │
       ▼
[ Controller Action Method ]
       │
       ▼
[ Returns ActionResult ]
       │
       ▼
[ View Engine ] ───────────> Locates .cshtml -> Compiles Razor -> Evaluates Model
       │
       ▼
[ HTTP 200 HTML Output ] ──> Sent back to Client

```

---

### 16. What is Dependency Injection (DI)?

**Dependency Injection** is a technique that implements the Inversion of Control (IoC) principle. Instead of an object constructing its own dependencies internally using `new`, the dependencies are "injected" from an external consumer or IoC container (typically through interfaces).

```csharp
// Violates DI (Tightly Coupled):
public class OrderService {
    private SqlRepository _repo = new SqlRepository();
}

// Implements DI (Loosely Coupled):
public class OrderService {
    private readonly IRepository _repo;
    public OrderService(IRepository repo) => _repo = repo;
}

```

---

### 17. Benefits of Dependency Injection

* **Decoupling:** High-level policy modules do not depend on low-level implementation details.
* **Testability:** Simplifies unit testing by allowing developers to swap real implementations (databases, payment gateways) with test mocks or stubs.
* **Maintainability & Flexibility:** Swapping implementations (e.g., switching from SQL Server to PostgreSQL) requires re-registering the service type in the composition root without altering client classes.
* **Controlled Lifetimes:** Centralized control over object life cycles (Transient, Scoped, Singleton).

---

### 18. How do we implement Dependency Injection?

**Three Primary Injection Patterns:**

1. **Constructor Injection (Most common & recommended):** Dependencies are supplied via class constructors. Guarantees the object is initialized in a valid state.
2. **Property / Setter Injection:** Dependencies are set through public properties. Useful for optional dependencies.
3. **Method Injection:** Dependency is supplied directly into a specific method signature via `[FromServices]` in ASP.NET controllers without storing it as a class field.

**Lifetimes in .NET Core / Modern .NET:**

```csharp
// Program.cs / Startup.cs
builder.Services.AddTransient<ITransientService, TransientService>(); // New instance per request
builder.Services.AddScoped<IScopedService, ScopedService>();          // One instance per HTTP request scope
builder.Services.AddSingleton<ISingletonService, SingletonService>(); // One instance across the entire application

```

---

### 19. Middleware in MVC

In modern ASP.NET Core, **Middleware** consists of software components assembled into an application's HTTP pipeline to handle requests and responses.

Each component:

* Decides whether to pass the request to the next component in the pipeline.
* Can perform work before and after the next component is invoked.

```
Request ──> [Middleware 1] ──> [Middleware 2] ──> [Routing / Endpoint]
                                                          │
Response <── [Middleware 1] <── [Middleware 2] <──────────┘

```

---

### 20. How do we add Middleware?

Middleware is registered in the HTTP request pipeline via the `WebApplication` instance in modern `Program.cs` (or `Configure` in legacy `Startup.cs`) using `Use*` extensions:

```csharp
var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

// 1. Built-in Middleware
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthentication();
app.UseAuthorization();

// 2. Custom inline middleware using app.Use
app.Use(async (context, next) =>
{
    // Logic before next component
    await next.Invoke();
    // Logic after next component
});

// 3. Custom Middleware Class registration
app.UseMiddleware<CustomLoggingMiddleware>();

app.MapControllers();
app.Run();

```

---

### 21. Use of Startup.cs File

In ASP.NET Core (up to .NET 5, and optional in .NET 6+), `Startup.cs` serves as the application's configuration and composition root, containing two key methods:

1. **`ConfigureServices(IServiceCollection services)`:** Configures and registers dependencies with the built-in IoC container (e.g., Entity Framework DbContext, Identity, CORS, MVC controllers).
2. **`Configure(IApplicationBuilder app, IWebHostEnvironment env)`:** Configures the HTTP request pipeline by defining the order in which middleware runs.

*(Note: In .NET 6, 7, and 8+, `Startup.cs` was consolidated into a single unified `Program.cs` file using Top-Level Statements).*

---

### 22. What is a ViewModel?

A **ViewModel** is a design pattern representing a specialized model tailored strictly for a specific View's presentation requirements.

* **Separation:** Keeps persistent Domain/Entity models (e.g., containing password hashes, soft-delete flags, internal IDs) separated from the UI.
* **Aggregation:** Combines data from multiple domain entities into a single strongly-typed object (e.g., combining `Customer`, `List<Order>`, and an `AddressDropdownOptions` list into `CustomerDashboardViewModel`).
* **Validation:** Contains UI-specific data annotations (`[Required]`, `[Compare]`, `[EmailAddress]`).

---

### 23. What are Cookies?

A **Cookie** is a small text file (up to 4KB) stored on the client's web browser, transmitted back and forth with every subsequent HTTP request to the matching domain inside the `Cookie` header.

* **Usage:** Session tracking, persistent user preferences, personalization, and authentication tokens.
* **Key Security Flags:**
* `HttpOnly`: Prevents client-side scripts (JavaScript) from accessing the cookie, mitigating XSS risks.
* `Secure`: Forces the cookie to be transmitted exclusively over encrypted HTTPS connections.
* `SameSite (Strict/Lax/None)`: Defends against Cross-Site Request Forgery (CSRF) attacks by restricting cross-site transmission.



---

### 24. What is Session Management?

**Session Management** is the mechanism used to maintain state across stateless HTTP requests for an individual user during their visit.

* **In-Process (InProc):** Stores session state inside the web server's memory (`w3wp.exe`). Fast, but fails in load-balanced web farms without sticky sessions, and state is lost whenever the AppPool restarts.
* **Out-of-Process (State Server / Redis / Distributed SQL):** Stores session externally in a distributed cache or database. Enables reliable multi-server load balancing and horizontal scaling without state drops.
* **Cookie-based State / JWT Tokens:** Keeps signed/encrypted state inside cookies or tokens carried with the request, moving storage overhead away from server memory.

---

### 25. What is .NET Framework?

The **.NET Framework** is Microsoft’s original comprehensive development platform for building and running Windows applications, first released in 2002.

It consists of:

1. **Common Language Runtime (CLR):** The managed execution environment providing memory management, JIT compilation, exception handling, and threading.
2. **Framework Class Library (FCL):** A comprehensive collection of reusable types, algorithms, and utilities (`System.*`, `System.IO`, `System.Net`).
3. **Application Stacks:** Built-in application frameworks including ASP.NET (Web Forms, MVC, Web API), Windows Forms, WPF (Windows Presentation Foundation), and WCF (Windows Communication Foundation).
