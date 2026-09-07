## 26. What is OWIN?

**OWIN (Open Web Interface for .NET)** is an open specification (not an implementation) that decouples .NET web applications from the underlying web server (such as IIS).

Before OWIN, ASP.NET applications were tightly coupled to `System.Web.dll` and IIS worker processes (`w3wp.exe`). OWIN established a standard interface via an environment dictionary:

```csharp
using AppFunc = Func<IDictionary<string, object>, Task>;

```

**Katana** was Microsoft's implementation of OWIN for ASP.NET 4.x, paving the way for the modular, server-agnostic middleware pipeline in modern ASP.NET Core (such as Kestrel).

```
┌────────────────────────────────────────────────────────┐
│                   Application / Web API                │
├────────────────────────────────────────────────────────┤
│           Middleware Pipeline (Auth, CORS, etc.)       │
├────────────────────────────────────────────────────────┤
│                  OWIN Interface Specification          │
├────────────────────────────────────────────────────────┤
│ Host / Server (IIS, HttpListener, Self-Host, Kestrel)  │
└────────────────────────────────────────────────────────┘

```

---

## 27. What is `Program.cs` File?

`Program.cs` is the entry point of a modern .NET Core / .NET application, executing when the process starts.

Starting in .NET 6 with C# 10, ASP.NET Core adopted the **Minimal Hosting Model** (top-level statements), consolidating the older `Startup.cs` directly into `Program.cs`.

### Responsibilities of `Program.cs`

1. **Host & WebApplication Builder**: Initializes configuration, logging, and environment settings.
2. **Dependency Injection (Service Registration)**: Registers framework and application services into the `IServiceCollection` container.
3. **HTTP Middleware Pipeline**: Dictates the order in which requests and responses pass through middleware.

```csharp
var builder = WebApplication.CreateBuilder(args);

// 1. Configure Services (DI)
builder.Services.AddControllers();
builder.Services.AddScoped<IOrderService, OrderService>();

var app = builder.Build();

// 2. Configure HTTP Pipeline (Middleware Execution Order)
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();

app.MapControllers();

// 3. Start Kestrel and listen for requests
app.Run();

```

---

## 28. Can We Write PUT in POST? If Yes, What is the Need of PUT?

**Yes, technically you can.** HTTP does not stop you from performing full resource updates inside a `POST` handler. However, doing so breaks REST principles and HTTP protocol semantics.

### Key Reasons `PUT` is Essential

* **Idempotency**: `PUT` is strictly **idempotent**, while `POST` is **non-idempotent**. Calling a `PUT` request multiple times produces the exact same server state ($N$ identical updates result in the same final record). If network timeouts cause a retry, network proxies and clients can safely replay `PUT` without creating duplicate records. Replaying `POST` can result in duplicate transactions (e.g., charging a card twice).
* **Replacement vs. Subordinate Creation**:
* `POST /api/orders`: Submits data to be processed, generating a new resource with a server-assigned URI.
* `PUT /api/orders/42`: Replaces the representation of the resource explicitly addressed by the URI.


* **Network & Proxy Caching**: Upstream caches and proxies invalidate cached representations of a URI on `PUT` and `DELETE` requests conforming to RFC 9110.

| Verb | Semantics | Idempotent | Safe | Typical Status Code |
| --- | --- | --- | --- | --- |
| **POST** | Create child resource / trigger action | No | No | `201 Created` / `200 OK` |
| **PUT** | Replace entire existing resource | Yes | No | `200 OK` / `204 No Content` |
| **PATCH** | Partial update to existing resource | No (can be) | No | `200 OK` / `204 No Content` |

---

## 29. How Can We Pass an Image in Web API?

There are three primary industry patterns for uploading images to a Web API:

### 1. `multipart/form-data` using `IFormFile` (Recommended for Large Files)

Streams the file directly without memory inflation.

```csharp
[HttpPost("upload")]
public async Task<IActionResult> UploadImage([FromForm] ImageUploadDto dto)
{
    if (dto.File == null || dto.File.Length == 0)
        return BadRequest("No file uploaded.");

    var allowedExtensions = new[] { ".jpg", ".jpeg", ".png" };
    var extension = Path.GetExtension(dto.File.FileName).ToLowerInvariant();
    if (!allowedExtensions.Contains(extension))
        return BadRequest("Invalid image format.");

    var filePath = Path.Combine("Uploads", $"{Guid.NewGuid()}{extension}");
    using (var stream = new FileStream(filePath, FileMode.Create))
    {
        await dto.File.CopyToAsync(stream);
    }

    return Ok(new { FilePath = filePath });
}

public class ImageUploadDto
{
    public string Title { get; set; }
    public IFormFile File { get; set; }
}

```

### 2. Base64 Encoded String (Best for Small Icons / Inline Payloads)

The image is converted to a base64 string and passed in a JSON payload.

* *Drawback*: Base64 increases the file payload size by roughly **33%** and incurs CPU overhead for encoding/decoding.

### 3. Direct Binary Stream (`application/octet-stream`)

Ideal for multi-gigabyte or raw block data streaming directly to blob storage (such as Azure Blob Storage or AWS S3) via `Request.BodyReader`.

---

## 30. Content Negotiation

**Content Negotiation** is the mechanism where client and server agree on the representation format of the payload exchanged over HTTP.

1. The client sends an `Accept` header indicating preferred MIME types (e.g., `Accept: application/json, text/xml;q=0.9`).
2. The server evaluates the registered formatters in order.
3. The server chooses the best matching formatter to serialize the object and sets the response `Content-Type` header accordingly.

```
Client                                                  API Gateway / Controller
  │                                                                 │
  ├──── GET /api/products ─────────────────────────────────────────►│
  │     Accept: application/xml                                     │
  │                                                                 │
  │                                                Inspects formatters:
  │                                                - JsonFormatter (Skip)
  │                                                - XmlFormatter  (Match!)
  │                                                                 │
  │◄─── 200 OK ─────────────────────────────────────────────────────┤
  │     Content-Type: application/xml                               │
  │     <Product><Id>1</Id><Name>Widget</Name></Product>            │

```

In ASP.NET Core, enable XML support alongside default JSON via:

```csharp
builder.Services.AddControllers()
    .AddXmlSerializerFormatters();

```

---

## 31. SAGA Pattern

In distributed microservice architectures, managing transactions across independent service databases violates the 2-Phase Commit (2PC) scalability model. The **SAGA Pattern** solves this by breaking a distributed transaction into a sequence of local transactions:

1. Each service executes its local transaction and updates its private database.
2. The service publishes a domain event or message.
3. The next service handles the event and executes its local transaction.
4. **Compensating Transactions**: If any step fails, the system executes a series of compensating (rollback) transactions backward to restore state consistency.

```
Happy Path:
[Order Service] ──► [Payment Service] ──► [Inventory Service]
 (Create Order)       (Deduct Funds)         (Reserve Stock)

Failure Path:
[Order Service] ──► [Payment Service] ──► [Inventory Service (FAILS: Out of stock)]
 (Cancel Order) ◄── (Refund Payment)  ◄── (Trigger Compensation)

```

### Two Coordination Approaches

* **Choreography**: Each service produces and listens to events without a central coordinator (event-driven via message brokers like RabbitMQ or Azure Service Bus).
* **Orchestration**: A central coordinator (Orchestrator) manages command execution, waits for responses, and triggers compensations on failure (e.g., using frameworks like MassTransit Courier or Temporal).

---

## 32. gRPC

**gRPC (Google Remote Procedure Call)** is a high-performance, contract-first, open-source RPC framework.

* **Transport**: Operates over **HTTP/2** (and HTTP/3), enabling multiplexed bidirectional streaming and header compression on a single TCP connection.
* **Payload Serialization**: Uses **Protocol Buffers (`.proto`)**, a binary serialization mechanism far smaller and faster to parse than JSON or XML.
* **Strict Contracts**: Services and data contracts are defined in `.proto` files, which auto-generate strongly typed client stubs and server bases across multiple languages.

```protobuf
// greet.proto
syntax = "proto3";

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply);
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}

```

### Typical Use Case

Ideal for high-throughput internal microservice-to-microservice communication where serialization speed and minimal network footprint are critical.

---

## 33. Open API Gateway (API Gateway Pattern)

An **API Gateway** acts as a single entry point reverse-proxy for all external clients, abstracting the internal microservice topology.

```
                                          ┌────────────────────────┐
                                    ┌────►│  Order Microservice    │
┌──────────────┐    ┌─────────────┐ │     └────────────────────────┘
│ Mobile App   ├───►│             │ │     ┌────────────────────────┐
└──────────────┘    │ API Gateway ├─┼────►│  Payment Microservice  │
┌──────────────┐    │  (Ocelot /  │ │     └────────────────────────┘
│ SPA (React)  ├───►│   YARP)     │ │     ┌────────────────────────┐
└──────────────┘    └─────────────┘ └────►│  Catalog Microservice  │
                                          └────────────────────────┘

```

### Core Cross-Cutting Concerns Handled by the Gateway

* **Routing & Aggregation**: Directs requests to downstream services and can fan out/aggregate multiple microservice responses into a single client payload.
* **Security**: Enforces SSL offloading, global token validation (JWT/OAuth2), and API key enforcement.
* **Resiliency & Traffic Control**: Rate limiting, throttling, circuit breakers, and load balancing.
* **Telemetry**: Unified logging, metrics aggregation, and distributed request tracing (`TraceId` injection).
* **Popular .NET Implementations**: **YARP** (Microsoft's Yet Another Reverse Proxy) and **Ocelot**.

---

## 34. Difference Between Microservice Architecture vs. Service-Oriented Architecture (SOA)

| Dimension | Microservices Architecture | Service-Oriented Architecture (SOA) |
| --- | --- | --- |
| **Service Scope** | Small, focused on a single bounded context (Domain-Driven Design) | Enterprise-wide, coarse-grained business services |
| **Communication Infrastructure** | Dumb pipes, smart endpoints (REST, gRPC, lightweight brokers) | Smart pipes, dumb endpoints (Enterprise Service Bus / ESB) |
| **Data Storage** | **Database-per-service**: Independent data isolation | Often shares large centralized enterprise databases |
| **Coupling & Governance** | Highly decoupled; decentralized governance (polyglot stacks allowed) | Tightly coordinated; centralized governance and enterprise data standards |
| **Deployment** | Independently containerized (Docker/Kubernetes); rapid CI/CD cycles | Monolithic deployments or large co-dependent service releases |

---

## 35. API Versioning

API versioning prevents breaking existing client integrations when data contracts, route definitions, or business workflows change.

### The 4 Major Versioning Strategies in ASP.NET Core

* **URI Path Versioning (Most Popular)**:
```http
GET /api/v1/orders
GET /api/v2/orders

```


* **Query String Parameter**:
```http
GET /api/orders?api-version=2.0

```


* **Custom HTTP Header**:
```http
GET /api/orders
X-Version: 2.0

```


* **Content Negotiation / Accept Header (Media Type Versioning)**:
```http
GET /api/orders
Accept: application/vnd.company.orders.v2+json

```



### Implementation with `Asp.Versioning.Mvc`

```csharp
// Program.cs
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true; // Returns api-supported-versions in headers
});

// Controllers
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/orders")]
public class OrdersV1Controller : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok("V1 Contract");
}

[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/orders")]
public class OrdersV2Controller : ControllerBase
{
    [HttpGet]
    public IActionResult Get() => Ok("V2 Contract with breaking changes added");
}

```
