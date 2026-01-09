# Serilog + Seq in ASP.NET Core (.NET 6+):
## Serilog vs Built-in Microsoft.Extensions.Logging (ILogger):

| Aspect                      | Microsoft.Extensions.Logging (Built-in)                  | Serilog                                              |
| :-------------------------- | :------------------------------------------------------ | :--------------------------------------------------- |
| **Primary Focus**           | Lightweight abstraction for logging; provider-based (Console, Debug, EventLog, etc.). | Full-featured structured logging library.            |
| **Structured Logging**      | Limited (mostly string messages; properties via scopes). | First-class (message templates with named properties, destructuring with `@` and `$`). |
| **Configuration**           | Simple, via `appsettings.json` or code; basic filters.  | Rich fluent API, JSON config, enrichers, filters, async support. |
| **Sinks/Outputs**           | Basic built-ins; extend via providers.                  | Vast ecosystem (Seq, files, ELK, cloud services, etc.). |
| **Request Logging**         | Noisy (multiple events per request).                    | Clean single-event middleware with timing/status.    |
| **Integration in ASP.NET Core** | Native; use `ILogger<T>` directly.                      | Plugs in via `UseSerilog()`; all logs (app + framework) route through Serilog. |
| **Best For**                | Simple apps, minimal dependencies.                      | Production apps needing queryable logs, correlation, dashboards. |
| **How They Work Together**  | Serilog is a *provider* for the built-in abstraction — inject `ILogger<T>` as usual, but configure Serilog as the backend. | N/A                                                  |

Serilog is a **structured** logging library for .NET that turns logs into queryable data instead of plain text, and Seq is a log server UI that stores and lets you search those structured logs in real time. Used together in ASP.NET Core, they give you rich, centralized, production‑ready logging with minimal code changes.


---

## 1. Serilog Basics

Serilog replaces or extends the default .NET logging with:

- Structured events: Named properties instead of string concatenation, e.g. `OrderId`, `UserId`, `ElapsedMs`.
- Multiple sinks: Console, files, Seq, Application Insights, Elasticsearch, etc., configured independently.
- Rich configuration: JSON configuration, enrichers, filters, async sinks, request logging middleware.

**Why Serilog vs default .NET logging**

- Default logging writes mostly text; Serilog makes every log a data record with fields, improving search and dashboards.
- Serilog has a large ecosystem (Seq sink, enrichers, async sinks, cloud sinks) and a consistent message‑template syntax.

**Serilog vs NLog vs log4net**

| Feature                   | Serilog                                     | NLog                                            | log4net                           |
| :------------------------ | :------------------------------------------ | :---------------------------------------------- | :-------------------------------- |
| Structured logging        | First‑class, message templates, properties. | Supported, but Serilog is more idiomatic.       | Primarily text‑oriented.          |
| Configuration style       | Strong JSON/config, fluent API.             | XML / JSON / fluent.                            | XML‑heavy configuration.          |
| Ecosystem/sinks           | Many sinks (Seq, ELK, AI, etc.).            | Many targets, quite mature.                     | Mature but slower evolving.       |
| ASP.NET Core support      | Dedicated `Serilog.AspNetCore` package.     | Integrates via logging providers.               | Less idiomatic with minimal host. |
| Popularity in modern .NET | Very high for new projects.                 | Still popular, especially in legacy/mixed apps. | Mostly legacy.                    |

---

## 2. What Seq Is

Seq is a log server and UI designed for structured logs. It:

- Accepts structured events over HTTP (default port 5341) from Serilog and other clients.
- Stores logs and exposes a web UI where you can query with a SQL‑like language, filter by properties, build dashboards, and set alerts.
  **Seq UI Tips** (once logged in):

- Filter: `OrderId = 42` OR `@Level = 'Error'`
- Group by `CorrelationId` → see full request trace
- Signals → alert on `@Exception LIKE '%SqlException%'`
- Time range → last 1h, 24h, custom

---

## 3. Real‑World Use Cases

- Web APIs: Trace each HTTP request with correlation ID, user, latency, and response code to quickly diagnose failures.
- Background services: Monitor queue processing, retries, and exception rates across multiple worker instances.
- Production troubleshooting: Filter by `CorrelationId`, `UserName`, or feature flag to reconstruct what happened for a single user or tenant.

---

## 4. Required NuGet Packages

- `Serilog.AspNetCore` – ASP.NET Core integration and request logging middleware.
- `Serilog.Sinks.Seq` – Sink that sends events to Seq.
- `Serilog.Sinks.Console` – Console logging (local dev, containers).
- `Serilog.Sinks.Async` – Async wrapper for performance.
- `Serilog.Enrichers.Environment` – Machine name, process, environment name.

Optional: `Swashbuckle.AspNetCore` for Swagger.

---

## 5. Architecture Diagram

```
+--------------------+        HTTP Requests        +-------------------+
|   Client (SPA /    |  ------------------------>  | ASP.NET Core API  |
|  Mobile / Other)   |                             |  (Serilog)        |
+--------------------+                             +-------------------+
                                                          |
                                                          | Structured events
                                                          v
                                                    +-------------+
                                                    |   Seq       |
                                                    | (Docker)    |
                                                    +-------------+
                                                          |
                                                          v
                                                 Web UI / Dashboards
```

---

## 6. Docker Setup for Seq

### docker-compose.yml

```yaml
version: "3.9"

services:
  seq:
    image: datalust/seq:latest
    container_name: seq
    restart: unless-stopped
    environment:
      - ACCEPT_EULA=Y
      - SEQ_FIRSTRUN_ADMINPASSWORD=changeme
    ports:
      - "5341:80"
    volumes:
      - ./seq-data:/data
```

Run: `docker compose up -d`
Access: `http://localhost:5341` (admin/changeme)

**Login reset** (if password forgotten):

```bash
docker stop seq
docker run --rm -it -v ./seq-data:/data datalust/seq auth reset -u admin -p ADMINADMIN
docker start seq
```

---

## 7. Complete Configuration

### appsettings.json

```json
{
  "Serilog": {
    "Using": [
      "Serilog.Sinks.Console",
      "Serilog.Sinks.Seq",
      "Serilog.Sinks.Async",
      "Serilog.Enrichers.Environment"
    ],
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning",
        "Microsoft.Hosting.Lifetime": "Information"
      }
    },
    "Enrich": ["FromLogContext", "WithMachineName"],
    "WriteTo": [
      {
        "Name": "Async",
        "Args": {
          "configure": [
            {
              "Name": "Console",
              "Args": {
                "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] {CorrelationId} {SourceContext} {Message:lj}{NewLine}{Exception}"
              }
            }
          ]
        }
      },
      {
        "Name": "Async",
        "Args": {
          "configure": [
            {
              "Name": "Seq",
              "Args": {
                "serverUrl": "http://localhost:5341"
              }
            }
          ]
        }
      }
    ],
    "Properties": {
      "Application": "Sample.Api"
    }
  }
}
```

### appsettings.Development.json (override)

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Debug"
    }
  }
}
```

---

## 8. Program.cs (Complete)

```csharp
using Serilog;

Log.Logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateLogger();

try
{
    Log.Information("Starting web host");

    var builder = WebApplication.CreateBuilder(args);

    // Serilog from appsettings
    builder.Host.UseSerilog((ctx, services, loggerConfig) => loggerConfig
        .ReadFrom.Configuration(ctx.Configuration)
        .ReadFrom.Services(services)
        .Enrich.FromLogContext());

    // API + Swagger
    builder.Services.AddControllers();
    builder.Services.AddEndpointsApiExplorer();
    builder.Services.AddSwaggerGen();

    var app = builder.Build();

    // Custom middleware → request logging
    app.UseMiddleware<CorrelationIdMiddleware>();
    app.UseSerilogRequestLogging();

    // Swagger
     if (app.Environment.IsDevelopment())
    {
        app.UseSwagger();
        app.UseSwaggerUI();  // Access at https://localhost:<port>/swagger
    }

    app.MapControllers();

    app.Run();
}
catch (Exception ex)
{
    Log.Fatal(ex, "Host terminated unexpectedly");
}
finally
{
    Log.CloseAndFlush();
}

// Correlation ID middleware
public class CorrelationIdMiddleware
{
    private const string HeaderName = "X-Correlation-Id";
    private readonly RequestDelegate _next;

    public CorrelationIdMiddleware(RequestDelegate next) => _next = next;

    public async Task Invoke(HttpContext context)
    {
        var correlationId = context.Request.Headers.TryGetValue(HeaderName, out var existing)
            ? existing.ToString()
            : Guid.NewGuid().ToString();

        context.Response.Headers[HeaderName] = correlationId;

        using (Serilog.Context.LogContext.PushProperty("CorrelationId", correlationId))
        {
            await _next(context);
        }
    }
}
```

---

## 9. OrdersController.cs (Structured Logging)

```csharp
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly ILogger<OrdersController> _logger;

    public OrdersController(ILogger<OrdersController> logger) => _logger = logger;

    [HttpGet("{id:int}")]
    public IActionResult Get(int id)
    {
        _logger.LogInformation("Fetching order {OrderId}", id);

        if (id <= 0)
        {
            _logger.LogWarning("Invalid order id {OrderId}", id);
            return BadRequest(new { message = "Invalid id" });
        }

        try
        {
            var order = new { Id = id, Total = 123.45m, Status = "Paid" };
            _logger.LogInformation("Order retrieved {@Order}", order);
            return Ok(order);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Error while fetching order {OrderId}", id);
            throw;
        }
    }
}
```

---

## 10. Verification Checklist
<img width="1894" height="1010" alt="image" src="https://github.com/user-attachments/assets/569723da-f6a9-471b-bd10-bff25332ea34" />

<img width="1907" height="1012" alt="image" src="https://github.com/user-attachments/assets/95c73f33-98c9-4791-9dff-5862aa21c3b2" />


After `dotnet run` and `docker compose up -d`:


```
✅ docker ps → Seq running on 5341
✅ http://localhost:5341 → Seq UI opens (admin/changeme)
✅ https://localhost:5001/swagger → API + Swagger
✅ GET /api/orders/1 → API responds with X-Correlation-Id header
✅ Seq UI → "Tail" → see live events:
   • "Fetching order 1" (OrderId=1)
   • "Order retrieved {Id=1,Total=123.45,Status=Paid}"
   • HTTP request log (StatusCode=200, ElapsedMs)
✅ Filter: OrderId = 1 → only matching events
```

**Health check Seq**:

```bash
curl -X POST http://localhost:5341/api/events/raw \
  -H "Content-Type: application/json" \
  -d '{"Timestamp":"2026-01-08T10:00:00Z","Level":"Information","MessageTemplate":"Test {Property}","Properties":{"Property":"Hello Seq"},"RenderedMessage":"Test Hello Seq"}'
```

---

## 11. Advanced: Masking Sensitive Data

**SensitiveDataPolicy.cs**:

```csharp
public class SensitiveDataPolicy : IDestructuringPolicy
{
    private static readonly string[] SensitiveKeys = { "Password", "CreditCard", "SSN" };

    public bool TryDestructure(object value, ILogEventPropertyValueFactory propertyValueFactory, out LogEventPropertyValue result)
    {
        if (value is IDictionary<string, object> dict)
        {
            var masked = new Dictionary<string, object>();
            foreach (var kvp in dict)
            {
                masked[kvp.Key] = SensitiveKeys.Contains(kvp.Key, StringComparer.OrdinalIgnoreCase)
                    ? "[REDACTED]"
                    : kvp.Value;
            }
            result = propertyValueFactory.CreatePropertyValue(masked);
            return true;
        }
        result = null!;
        return false;
    }
}
```

```csharp
// In Program.cs Serilog config:
.Destructure.ByTransforming<IDictionary<string, object>>(new SensitiveDataPolicy())
```

---

## 12. Docker Compose: API + Seq

```yaml
services:
  seq:
    # ... (as above)

  api:
    build: .
    environment:
      - Serilog__WriteTo__1__Args__serverUrl=http://seq:80
    ports:
      - "5001:80"
    depends_on:
      - seq
```

---

## 13. Best Practices

**Log Levels:** Dev=Debug, Prod=Information, Microsoft/System=Warning
**Performance:** Async sinks, no large objects in logs
**Security:** Mask PII/secrets (above policy)
**Troubleshooting:** `Serilog.Debugging.SelfLog.Enable()`

**Production Strategy:**

- Dev: Console + Seq, Debug level
- Prod: Seq only, Information level
- Alerts: Seq Signals for `@Level = 'Error'` spikes

---

## 14. Common Mistakes

| Problem            | Cause                        | Fix                            |
| :----------------- | :--------------------------- | :----------------------------- |
| No logs in Seq     | Wrong URL (`5340` vs `5341`) | Use `http://localhost:5341`    |
| Seq login fails    | Wrong initial password       | `docker run ... auth reset`    |
| No Debug logs      | Default=Information          | `appsettings.Development.json` |
| Performance issues | Sync sinks                   | `.WriteTo.Async(...)`          |
