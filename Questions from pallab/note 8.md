## 1. What is .NET Core?

.NET Core (now unified under .NET 6/7/8+) is an open-source, cross-platform, modular, and cloud-optimized framework created by Microsoft. It runs on Windows, Linux, and macOS, removing the historical dependency on the Windows OS and IIS (`System.Web`).

* **Cross-Platform & Architecture**: Runs on x64, x86, and ARM architectures.
* **Modular Runtime**: Ships via standalone runtimes and lightweight NuGet packages rather than monolithic machine-wide GAC (Global Assembly Cache) installations.
* **Modern CLI**: Standardized project manipulation and deployment via the `dotnet` CLI toolchain.

---

## 2. What are the Features Provided by ASP.NET Core?

* **Kestrel Web Server**: High-performance, event-driven internal web server built on libuv and managed sockets.
* **Built-in Dependency Injection (DI)**: Inversion of control is a first-class citizen embedded across the framework.
* **Asynchronous Execution Pipeline**: Non-blocking request handling optimized via `Task` and `ValueTask`.
* **Unified Programming Model**: Merged MVC and Web API into a single controller and routing infrastructure.
* **Flexible Configuration Engine**: Hierarchical, multi-provider configuration system (`appsettings.json`, environment variables, command-line arguments, Azure Key Vault).

---

## 3. What are the Advantages of .NET Core Over ASP.NET?

| Capability | ASP.NET Core | Classic ASP.NET (Framework) |
| --- | --- | --- |
| **Platform** | Windows, Linux, macOS, Docker containers | Windows-only dependency |
| **Performance** | Top-tier TechEmpower benchmarks (Kestrel) | Heavier footprint via `System.Web` |
| **Hosting Model** | Self-hosted (out-of-process/in-process), Kestrel, Docker, IIS | Tightly bound to IIS worker process (`w3wp.exe`) |
| **Side-by-Side Deployment** | App-local runtime versions run concurrently | Machine-wide Framework updates risk breaking apps |
| **Code Footprint** | Pay-as-you-go modular packages | Monolithic assembly references |

---

## 4. What are Metapackages?

A **Metapackage** is a package specification that contains no assemblies of its own; instead, it aggregates a curated list of dependent NuGet packages.

* In earlier .NET Core versions (2.x), `Microsoft.AspNetCore.All` and `Microsoft.AspNetCore.App` were used so developers didn't have to manage dozens of separate package versions.
* In modern .NET (3.1 through .NET 8+), metapackages were largely replaced by the **Shared Framework (`Microsoft.AspNetCore.App`)**, referenced via the project SDK (`<Project Sdk="Microsoft.NET.Sdk.Web">`), shipping pre-compiled binaries with the runtime itself to reduce deployment sizes.

---

## 5. What is the Startup Class in ASP.NET Core?

In classic ASP.NET Core architectures (prior to .NET 6 minimal hosting), the `Startup` class served as the application's configuration hub. It conventionally includes two core methods:

* `ConfigureServices(IServiceCollection services)`: Registers dependencies into the IoC container.
* `Configure(IApplicationBuilder app, IWebHostEnvironment env)`: Sets up the HTTP request pipeline (middleware chain).

```csharp
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddControllers();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        app.UseRouting();
        app.UseEndpoints(endpoints => endpoints.MapControllers());
    }
}

```

---

## 6. What is the Use of `ConfigureServices` Method of the Startup Class?

The `ConfigureServices` method configures Dependency Injection (DI). It takes an `IServiceCollection` parameter and is invoked by the host runtime before the middleware pipeline is assembled.

```csharp
public void ConfigureServices(IServiceCollection services)
{
    // Framework services
    services.AddControllersWithViews();
    services.AddDbContext<AppDbContext>(opts => opts.UseSqlServer("conn_str"));

    // Custom Application services with defined lifecycles
    services.AddScoped<IOrderRepository, OrderRepository>();
    services.AddTransient<IEmailSender, SmtpEmailSender>();
}

```

---

## 7. What is the Use of the `Configure` Method of the Startup Class?

The `Configure` method defines the HTTP request pipeline by specifying how the application responds to incoming requests. It chains **Middleware** components sequentially using the `IApplicationBuilder` instance.

* **Order of Execution**: Middleware runs strictly in the order it is registered within this method.

```csharp
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment()) app.UseDeveloperExceptionPage();

    app.UseHttpsRedirection();
    app.UseStaticFiles();
    app.UseRouting();
    app.UseAuthorization();
    app.UseEndpoints(endpoints => endpoints.MapControllers());
}

```

---

## 8. What is Middleware?

**Middleware** is software assembled into an application pipeline to handle requests and responses. Each component:

* Decides whether to pass the request to the next component in the pipeline.
* Can perform work before and after the next component is invoked.

```
       HTTP Request
            │
            ▼
┌───────────────────────┐
│     Middleware 1      │
│  (e.g., Exception)    │
│  ┌─────────────────┐  │
│  │ Pre-processing  │  │
│  └────────┬────────┘  │
│           ▼           │
│  ┌─────────────────┐  │
│  │  Middleware 2   │  │
│  │  (e.g., Auth)   │  │
│  │  ┌───────────┐  │  │
│  │  │ Endpoint  │  │  │
│  │  └─────┬─────┘  │  │
│  │  ┌─────▼─────┐  │  │
│  │  │Post-action│  │  │
│  │  └───────────┘  │  │
│  └────────┬────────┘  │
│  ┌────────▼────────┐  │
│  │ Post-processing │  │
│  └─────────────────┘  │
└───────────┬───────────┘
            ▼
       HTTP Response

```

---

## 9. Difference Between `IApplicationBuilder.Use()` and `IApplicationBuilder.Run()`

* **`Use()`**: Chains middleware. It receives a pointer/delegate to the next middleware (`Func<Task> next`) and can pass execution downstream via `await next()`.
* **`Run()`**: A **terminal middleware**. It short-circuits the pipeline; it does not receive a `next` delegate and prevents subsequent middleware from running.

```csharp
// Use() can continue the chain
app.Use(async (context, next) =>
{
    // Pre-processing
    await next(); 
    // Post-processing
});

// Run() terminates the chain
app.Run(async context =>
{
    await context.Response.WriteAsync("Terminal Response: Execution ends here.");
});

```

---

## 10. What is the Use of the "Map" Extension While Adding Middleware?

`Map` extensions (`Map`, `MapWhen`) branch the pipeline based on request paths or boolean predicates.

* **`app.Map("/branch")`**: Diverts execution down an isolated sub-pipeline if the path matches.
* **`app.MapWhen(predicate)`**: Diverts execution based on custom logic (e.g., checking headers, query params).

```csharp
app.Map("/webhooks", webhookApp =>
{
    webhookApp.Use(async (context, next) =>
    {
        // Custom logic exclusively executed for /webhooks/*
        await next();
    });
    webhookApp.Run(async ctx => await ctx.Response.WriteAsync("Webhook Processed"));
});

```

---

## 11. What is Routing in ASP.NET Core?

Routing inspects incoming HTTP request URIs and verbs and dispatches them to matching executable endpoints. ASP.NET Core provides **Endpoint Routing** split across two middleware:

1. **`UseRouting()`**: Matches the incoming request path against route definitions and assigns an endpoint to `HttpContext`.
2. **`UseEndpoints()`**: Executes the matched endpoint delegate or controller action.

```csharp
// Conventional Routing
app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

// Attribute Routing
[Route("api/[controller]")]
public class UsersController : ControllerBase
{
    [HttpGet("{id:int}")]
    public IActionResult GetUser(int id) => Ok();
}

```

---

## 12. How to Enable Session in ASP.NET Core?

Sessions require an in-memory or distributed backing store plus the session middleware:

```csharp
// 1. Register Session services
builder.Services.AddDistributedMemoryCache(); // Backing store
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(20);
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
});

// 2. Add Middleware to pipeline (Must be between UseRouting and UseEndpoints)
app.UseRouting();
app.UseSession();
app.MapControllers();

// 3. Set/Get Session in a Controller
HttpContext.Session.SetString("UserRole", "Admin");
string role = HttpContext.Session.GetString("UserRole");

```

---

## 13. What is Tag Helper in ASP.NET Core?

**Tag Helpers** enable server-Assuming you want concise, interview-ready answers for each question written in the notebook:

**1. What is .NET Core?**

A free, open-source, cross-platform successor to .NET Framework designed for modern, cloud-enabled, and internet-connected applications across Windows, Linux, and macOS.

**2. What are the features provided by ASP.NET Core?**

Cross-platform support, built-in dependency injection (DI), lightweight modular HTTP request pipeline, high-performance asynchronous execution, unified programming model for MVC and Web APIs, and environment-based configuration.

**3. What are the advantages of Core over ASP.NET?**

Cross-platform deployment (Docker/Linux friendly), significantly higher performance and throughput (Kestrel), modular architecture via NuGet packages (no heavy `System.Web`), side-by-side versioning, and built-in DI.

**4. What are Metapackages?**

Packages that do not contain actual libraries themselves, but rather aggregate a set of dependent packages with specific versions (e.g., historical `Microsoft.AspNetCore.App` before it became an implicit shared framework reference).

**5. What is the Startup class in ASP.NET Core?**

The class where services required by the app are registered and the HTTP request handling pipeline is configured. It typically contains `ConfigureServices` and `Configure` methods (or is consolidated in `Program.cs` in modern .NET).

**6. What is the use of ConfigureServices method of the Startup class?**

Used to register dependencies into the built-in IoC container via `IServiceCollection` so they can be injected across controllers, middleware, or services.

**7. What is the use of the Configure method of the Startup class?**

Used to specify how the application responds to individual HTTP requests by configuring the middleware pipeline using `IApplicationBuilder`.

**8. What is Middleware?**

Software assembled into an application pipeline to handle requests and responses. Each component chooses whether to pass the request to the next component or short-circuit it.

**9. What is the difference between IApplicationBuilder.Use() and IApplicationBuilder.Run()?**

`Use()` can invoke the next middleware in the pipeline using the `next` delegate, whereas `Run()` terminates/short-circuits the pipeline without calling any subsequent middleware.

**10. What is the use of the "Map" extension while adding middleware to the ASP.NET Core pipeline?**

Branches the pipeline based on the request path (e.g., `app.Map("/api", ...)` routes requests starting with `/api` through a dedicated branch).

**11. What is Routing in ASP.NET Core?**

The mechanism that inspects incoming HTTP request URLs and maps them to specific executable endpoints, such as controller actions, Razor Pages, or minimal API handlers.

**12. How to enable Session in ASP.NET Core?**

Register the session service in DI via `builder.Services.AddDistributedMemoryCache()` and `builder.Services.AddSession()`, then add `app.UseSession()` into the middleware pipeline before endpoint execution.

**13. What is Tag Helper in ASP.NET Core?**

A feature in Razor views enabling server-side code to participate in creating and rendering HTML elements using HTML-friendly syntax (e.g., `<a asp-action="Index">`).

**14. What is Singleton, Transient, and Scoped?**

* **Transient:** Created each time they are requested.
* **Scoped:** Created once per client request (connection/scope).
* **Singleton:** Created once on initial request and shared application-wide across all subsequent requests.

**15. Describe Dependency Injection.**

A design pattern where an object receives its dependencies from an external source (IoC container) rather than instantiating them internally, promoting loose coupling and easier testing.

**16. Explain the request processing pipeline in ASP.NET Core.**

A sequence of middleware components arranged in a two-way pipeline: requests travel down through each middleware layer to the endpoint handler, and responses travel back up through the same layers in reverse order.

**17. Difference between App.Use and App.Run.**

Identical to question 9: `App.Use()` can pass execution to the next middleware via `next.Invoke()`, while `App.Run()` is a terminal middleware that never calls `next()`.

**18. How to use multiple environments in ASP.NET Core?**

Using the `ASPNETCORE_ENVIRONMENT` environment variable (e.g., `Development`, `Staging`, `Production`), environment-specific configuration files (`appsettings.Development.json`), and checking `IWebHostEnvironment.IsDevelopment()`.

**19. How to handle errors in ASP.NET Core?**

Using the built-in exception handler middleware (`app.UseExceptionHandler()`), developer exception page (`app.UseDeveloperExceptionPage()`), status code pages, or custom exception filters and middleware.

**20. How does ASP.NET Core serve static files?**

By placing files in the `wwwroot` folder and adding `app.UseStaticFiles()` to the request pipeline.

**21. Explain Session and State Management in ASP.NET Core.**

Techniques to persist data across user requests, including:

* **Client-side:** Cookies, Query Strings, Hidden form fields.
* **Server-side:** In-memory or distributed Session (`HttpContext.Session`), Cache (`IMemoryCache`, `IDistributedCache`), and `TempData`.

**22. Explain Model Binding in ASP.NET Core.**

The automatic process of extracting data from HTTP requests (route data, query strings, form fields, headers, request bodies) and converting it into .NET action method parameters or objects.

**23. Describe Model Validation.**

Evaluating incoming model properties against validation attributes (e.g., `[Required]`, `[StringLength]`) and checking `ModelState.IsValid` before executing business logic.

**24. How to write custom ASP.NET Core middleware?**

Create a class with a constructor accepting `RequestDelegate next` and a public method `public async Task InvokeAsync(HttpContext context)`, then expose it via an `IApplicationBuilder` extension method.

**25. How to access HttpContext in ASP.NET Core?**

In controllers via the base property `ControllerBase.HttpContext`, or in arbitrary classes by registering `builder.Services.AddHttpContextAccessor()` and injecting `IHttpContextAccessor`.

---

Would you like code snippets or deep dives for any specific questions from this list?
