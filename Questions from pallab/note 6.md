## 1. Return Types in ASP.NET Web API

In ASP.NET Web API (covering both classic .NET Framework Web API 2 and modern ASP.NET Core), action methods can return four primary types:

* **`void`**: Returns an empty response with HTTP status code `204 No Content`. Common for fire-and-forget or update actions where no payload is needed back.
* **Primitive or Complex Types (e.g., `string`, `int`, `CustomerDTO`)**: The framework serializes the returned CLR object directly into the response body (typically JSON or XML via content negotiation) and returns HTTP `200 OK`. However, you cannot directly customize status codes or response headers without injecting context.
* **`HttpResponseMessage`**: Gives total, low-level control over the raw HTTP response, including headers, status code, and raw body stream.
* **`IHttpActionResult` (or `IActionResult` / `ActionResult<T>` in ASP.NET Core)**: Acts as a factory for `HttpResponseMessage`. It encapsulates the response-generation logic, promotes unit testability, and standardizes helper responses like `Ok()`, `NotFound()`, and `BadRequest()`.

```csharp
// 1. Primitive / Complex Type
[HttpGet]
public Customer GetCustomer(int id) => _service.Find(id); // Returns 200 OK or null (204)

// 2. HttpResponseMessage (Direct control)
[HttpGet]
public HttpResponseMessage GetRaw()
{
    var response = Request.CreateResponse(HttpStatusCode.OK, "Direct Payload");
    response.Headers.Add("X-Custom-Header", "Value");
    return response;
}

// 3. IHttpActionResult (Web API 2) / IActionResult (.NET Core)
[HttpGet]
public IHttpActionResult GetById(int id)
{
    var item = _service.Find(id);
    if (item == null) return NotFound(); // 404
    return Ok(item);                    // 200 with payload
}

```

---

## 2. Web API vs. WCF

| Feature | ASP.NET Web API | WCF (Windows Communication Foundation) |
| --- | --- | --- |
| **Protocol Focus** | Exclusively HTTP / HTTPS | Protocol-agnostic (HTTP, TCP, Named Pipes, MSMQ) |
| **Architecture** | RESTful architectural style | Primarily SOAP / RPC (WS-* specifications) |
| **Configuration** | Convention over configuration; minimal setup | Heavy, XML-driven configuration (`web.config`) |
| **Data Formats** | Native JSON, XML, Form-Data, BSON | Primarily XML/SOAP envelope; binary via TCP |
| **Client Reach** | Universal: Browsers, mobile, IoT, cross-platform | Ideal for enterprise Windows-to-Windows communication |
| **Performance** | Lightweight, low overhead over HTTP | High performance over NetTCP, heavy overhead over HTTP/SOAP |

---

## 3. Difference Between REST API vs. RESTful API

While frequently used as synonyms in the industry, there is a technical and semantic distinction:

* **REST API**: An API that adopts some principles of Fielding's Representational State Transfer (such as using HTTP verbs or exposing resource-based URIs), but might omit formal constraints (like complete statelessness, hypermedia controls, or standard caching).
* **RESTful API**: An API that strictly satisfies all core architectural constraints defined by Roy Fielding:
1. **Statelessness**: Every request contains all context needed to execute.
2. **Client-Server Separation**: Independent evolution of UI and data store.
3. **Uniform Interface**: Resource identification via URIs, resource manipulation via representations, self-descriptive messages, and HATEOAS (Hypermedia As The Engine Of Application State).
4. **Cacheability**: Responses explicitly mark themselves cacheable or non-cacheable.
5. **Layered System**: Intermediaries (load balancers, proxies) are transparent to the client.



```
Client ----(Self-Descriptive HTTP Request: GET /orders/123)----> Server
Client <---(200 OK + Payload + HATEOAS links to /cancel, /pay)--- Server

```

---

## 4. Advantages of Using REST API

* **Lightweight & High Performance**: Minimal transport overhead compared to envelope-heavy protocols like SOAP.
* **Stateless Scalability**: Since the server preserves no client session state between requests, horizontal scaling behind load balancers requires no complex sticky-session infrastructure.
* **Platform & Language Agnostic**: Any client capable of issuing HTTP requests and parsing JSON/XML (Python, Go, Node.js, mobile SDKs) can interact seamlessly.
* **Built-in HTTP Caching**: Uses native HTTP caching semantics (`ETag`, `Cache-Control`, `Last-Modified`) to offload server traffic directly via CDNs and client caches.
* **Clear Resource Separation**: Standard HTTP verbs (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`) establish an intuitive domain-driven interface.

---

## 5. Media Type Formatter in Web API

A **Media Type Formatter** handles serialization (writing objects into the HTTP response body) and deserialization (reading request body streams into CLR objects) based on the `Content-Type` and `Accept` headers.

### Built-in Formatters

* `JsonMediaTypeFormatter` (uses `System.Text.Json` or `Newtonsoft.Json`)
* `XmlMediaTypeFormatter` (`DataContractSerializer` / `XmlSerializer`)
* `FormUrlEncodedMediaTypeFormatter`

### Pipeline Flow

```
[Request Body] ---> [ReadStream] ---> [MediaTypeFormatter] ---> [C# Model Object]
                                                                        |
                                                                 (Action Execution)
                                                                        |
[Response Body] <-- [WriteStream] <-- [MediaTypeFormatter] <--- [Action Result]

```

### Custom Formatter Registration Example

To handle custom payloads (e.g., CSV formatting):

```csharp
public static class WebApiConfig
{
    public static void Register(HttpConfiguration config)
    {
        // Enforce JSON serialization and remove XML support globally
        config.Formatters.Remove(config.Formatters.XmlFormatter);
        
        // Add custom formatter
        config.Formatters.Add(new CsvMediaTypeFormatter());
    }
}

```

---

## 6. Web API Filters

Filters allow you to inject cross-cutting concerns (logging, authentication, validation) declaratively or programmatically across actions, controllers, or globally.

### Filter Execution Hierarchy

```
       HTTP Request
            │
            ▼
┌───────────────────────┐
│ Authentication Filter │  Validates client credentials (IAuthenticationFilter)
└───────────┬───────────┘
            ▼
┌───────────────────────┐
│ Authorization Filter  │  Validates permissions/roles (IAuthorizationFilter / [Authorize])
└───────────┬───────────┘
            ▼
┌───────────────────────┐
│     Action Filter     │  Executes OnActionExecuting (pre-action logic)
└───────────┬───────────┘
            ▼
┌───────────────────────┐
│    Action Method      │  Your business/controller logic runs here
└───────────┬───────────┘
            ▼
┌───────────────────────┐
│     Action Filter     │  Executes OnActionExecuted (post-action logic)
└───────────┬───────────┘
            ▼
┌───────────────────────┐
│    Exception Filter   │  Executes OnException (if unhandled exception occurs)
└───────────┬───────────┘
            │
            ▼
      HTTP Response

```

### Example Custom Action Filter

```csharp
public class ValidateModelAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(HttpActionContext actionContext)
    {
        if (!actionContext.ModelState.IsValid)
        {
            actionContext.Response = actionContext.Request.CreateErrorResponse(
                HttpStatusCode.BadRequest, actionContext.ModelState);
        }
    }
}

```

---

## 7. How to Handle Errors in Web API

In enterprise .NET development, error handling is implemented across three tiers:

### 1. `HttpResponseException`

Used to terminate action execution early and return an explicit HTTP status code directly:

```csharp
public Product GetProduct(int id)
{
    var item = _repo.Get(id);
    if (item == null)
        throw new HttpResponseException(HttpStatusCode.NotFound);
    return item;
}

```

### 2. Exception Filters (`ExceptionFilterAttribute`)

Catches unhandled exceptions specifically thrown from controller actions:

```csharp
public class GlobalExceptionFilter : ExceptionFilterAttribute
{
    public override void OnException(HttpActionExecutedContext context)
    {
        _logger.LogError(context.Exception, "Unhandled Web API Exception");
        context.Response = context.Request.CreateResponse(
            HttpStatusCode.InternalServerError, 
            new { message = "An error occurred while processing your request." }
        );
    }
}

```

### 3. Global `IExceptionLogger` and `IExceptionHandler` (Web API 2) or Custom Middleware (.NET Core)

Exception filters do not catch errors outside of controllers (e.g., inside message handlers, routing resolution, or serialization). A central exception handler handles all failures:

```csharp
// ASP.NET Core Middleware pattern
app.UseExceptionHandler(appError =>
{
    appError.Run(async context =>
    {
        context.Response.StatusCode = (int)HttpStatusCode.InternalServerError;
        context.Response.ContentType = "application/json";
        await context.Response.WriteAsync(JsonSerializer.Serialize(new { Error = "Internal Server Error" }));
    });
});

```

---

## 8. What is the Use of `HttpResponseMessage`?

`HttpResponseMessage` represents the complete HTTP response message sent back to the client. It provides direct access to mutate:

* **Status Code**: Set explicit HTTP status codes (`200 OK`, `201 Created`, `409 Conflict`).
* **Headers**: Append caching controls, custom trace identifiers (`X-Correlation-ID`), or pagination metadata (`X-Total-Count`).
* **Content/Payload**: Wrap raw streams, byte arrays, file downloads, or serialized strings.

```csharp
[HttpGet]
[Route("api/files/download")]
public HttpResponseMessage DownloadFile()
{
    byte[] fileBytes = System.IO.File.ReadAllBytes("report.pdf");
    var response = new HttpResponseMessage(HttpStatusCode.OK)
    {
        Content = new ByteArrayContent(fileBytes)
    };
    response.Content.Headers.ContentType = new MediaTypeHeaderValue("application/pdf");
    response.Content.Headers.ContentDisposition = new ContentDispositionHeaderValue("attachment")
    {
        FileName = "Report.pdf"
    };
    return response;
}

```

---

## 9. Difference Between `ApiController` and `Controller`

| Metric | `ApiController` (Web API) | `Controller` (MVC) |
| --- | --- | --- |
| **Namespace** | `System.Web.Http` | `System.Web.Mvc` (or `Microsoft.AspNetCore.Mvc`) |
| **Target Output** | Raw data models (JSON, XML) | HTML Views (`ViewResult`, `PartialView`) |
| **Action Resolution** | Matches based on HTTP verbs (`Get()`, `Post()`) | Matches based on action method names |
| **Serialization** | Automatic via `MediaTypeFormatters` | Manual via `Json(model)` helper or Razor rendering |
| **Lifecycle Hooks** | Web API Filter Pipeline | MVC Filter Pipeline |

> *Note on ASP.NET Core:* Both base classes were unified under `Microsoft.AspNetCore.Mvc.ControllerBase` (for headless APIs) and `Controller` (for Razor views with UI support).

---

## 10. ASP.NET Web API Routing

Routing maps incoming HTTP request URIs and verbs to specific controller actions.

### 1. Convention-Based Routing

Configured centrally in `WebApiConfig.cs`. Routes rely on parameter placeholders:

```csharp
config.Routes.MapHttpRoute(
    name: "DefaultApi",
    routeTemplate: "api/{controller}/{id}",
    defaults: new { id = RouteParameter.Optional }
);

```

### 2. Attribute Routing (Preferred)

Applied directly at the controller or action level using attributes. Ideal for RESTful hierarchical resources:

```csharp
[RoutePrefix("api/departments/{deptId}/employees")]
public class EmployeesController : ApiController
{
    [HttpGet]
    [Route("{empId:int}")] // Route constraint
    public IHttpActionResult GetEmployee(int deptId, int empId)
    {
        return Ok(_service.Get(deptId, empId));
    }
}

```

---

## 11. Content Negotiation in ASP.NET Web API

Content Negotiation (ConNeg) is the process defined by RFC 2616 where client and server negotiate which representation format to use for an HTTP exchange.

```
Client                              Server
  │                                   │
  ├─── Accept: application/json ─────►│ 1. Inspects 'Accept' header
  │                                   │ 2. Scans config.Formatters
  │                                   │ 3. Selects JsonMediaTypeFormatter
  │◄── Content-Type: application/json─┼ 4. Serializes object

```

### How the Default Negotiator Works

The `DefaultContentNegotiator` evaluates factors in the following order:

1. **MediaTypeMapping**: Matches explicit URL mappings (e.g., query string `?format=json`).
2. **`Accept` Header**: Checks the MIME type requested by the client (`application/json`, `text/xml`).
3. **Request Body `Content-Type**`: Matches the incoming format if no `Accept` is declared.
4. **Formatter Capabilities**: Selects the first registered formatter capable of writing the object type.

---

## 12. What is CORS in Web API?

**CORS (Cross-Origin Resource Sharing)** is a W3C standard and browser security mechanism that relaxes the **Same-Origin Policy (SOP)**.

A request is considered cross-origin when the **Protocol**, **Domain**, or **Port** of the client application differs from the target API server.

```
Client Origin: https://dashboard.example.com
Target API:    https://api.example.com:443
               └──── Different subdomains: Browser triggers CORS!

```

### The Preflight Check (`OPTIONS`)

For non-simple HTTP requests (e.g., requests carrying custom headers like `Authorization` or using verbs like `PUT`/`DELETE`), the browser dispatches an initial preflight request:

```
Browser                                  Web API Server
   │                                           │
   ├── OPTIONS /api/orders ───────────────────►│ (Validates Origin & Methods)
   │   Origin: https://dashboard.example.com   │
   │   Access-Control-Request-Method: PUT      │
   │                                           │
   │◄─ 200 OK ─────────────────────────────────┤
   │   Access-Control-Allow-Origin: *          │
   │   Access-Control-Allow-Methods: GET, PUT  │
   │                                           │
   ├── PUT /api/orders (Actual Request) ──────►│
   │◄─ 200 OK (With Payload) ──────────────────┤

```

### Enabling CORS in ASP.NET Web API

Install NuGet package `Microsoft.AspNet.WebApi.Cors` and register:

```csharp
// Global configuration (WebApiConfig.cs)
public static void Register(HttpConfiguration config)
{
    // Origins, Headers, Methods
    var cors = new EnableCorsAttribute("https://dashboard.example.com", "*", "GET,POST,PUT");
    config.EnableCors(cors);
}

// Or at Controller / Action scope:
[EnableCors(origins: "https://dashboard.example.com", headers: "*", methods: "*")]
public class OrdersController : ApiController { }

```
