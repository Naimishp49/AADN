## 🔹 1. What is **Serilog**?

**Serilog** is a **logging library** for .NET.

### What it does

* Captures logs from your application
* Structures logs as **key–value data**, not just text
* Sends logs to different **destinations (sinks)**

### Examples of what Serilog handles

* `Information`, `Warning`, `Error`, `Fatal` logs
* Enriching logs with:

  * RequestId
  * UserId
  * Environment
  * Exception details

### Example

```csharp
Log.Information("User {UserId} logged in at {Time}", userId, DateTime.Now);
```

👉 Serilog **creates and sends logs**

---

## 🔹 2. What is **Seq**?

**Seq** is a **log server + UI**.

### What it does

* **Receives logs** from applications
* Stores logs centrally
* Provides a **web dashboard**
* Allows:

  * Searching
  * Filtering
  * Querying
  * Visualizing logs

### Example UI features

* Search: `@Level = 'Error'`
* Filter by `UserId`
* Timeline view
* Exception stack trace viewer

👉 Seq **stores and shows logs**

---

## 🔁 3. How They Work Together

```text
ASP.NET Core App
     |
     |  (logs)
     v
 Serilog
     |
     |  (sink)
     v
   Seq
```

* Serilog **writes logs**
* Seq **displays logs**

---

## 🔍 4. Key Differences (Quick Table)

| Feature            | Serilog           | Seq                |
| ------------------ | ----------------- | ------------------ |
| Type               | Logging framework | Log server + UI    |
| Runs inside app    | ✅ Yes             | ❌ No               |
| Stores logs        | ❌ No              | ✅ Yes              |
| Visual dashboard   | ❌ No              | ✅ Yes              |
| Structured logging | ✅ Yes             | ✅ Yes              |
| Needs installation | ❌ NuGet           | ✅ Separate service |

---

## ⚙️ 5. Do You Need Both?

### ✔ Recommended in real projects

* **Serilog** → Capture & structure logs
* **Seq** → Centralize & analyze logs

### Alternatives to Seq

* Elasticsearch + Kibana
* Azure Application Insights
* Grafana Loki

Serilog can send logs to **many sinks**, not only Seq.

---

## 🧠 6. Real-World Analogy

| Role    | Example              |
| ------- | -------------------- |
| Serilog | CCTV camera          |
| Seq     | Control room monitor |

Camera records → Monitor shows

---

## 🚀 7. Minimal Setup Example

### Install packages

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Seq
```

### Configure

```csharp
Log.Logger = new LoggerConfiguration()
    .WriteTo.Seq("http://localhost:5341")
    .CreateLogger();
```

---

## 🎯 Bottom Line

> **Serilog = logging engine**
> **Seq = log storage + viewer**
> **They are different, but best used together**

---
## Why Serilog? 

- **Performance**: Handles high-volume logs without slowing your app (e.g., async logging).
- **Structured Logs**: Logs aren't just text—they can include key-value pairs (e.g., `{ "UserId": 123, "Action": "Login" }`), perfect for querying.
- **Sinks (Outputs)**: Write logs to files, console, databases (e.g., SQL Server, MongoDB), cloud services (e.g., Azure), and more. Over 90+ options!
- **Easy Integration**: Plugs right into ASP.NET Core's logging system.
- **Community**: Active GitHub repo with tons of examples.

Create a new ASP.NET Core Web API project:
1. Open Visual Studio → Create a new project → Search for "ASP.NET Core Web API" → Name it (e.g., `SerilogDemo`) → Select .NET 6.0+ → Create.
2. Run it (F5) to see Swagger UI at `https://localhost:<port>/swagger`. This confirms your setup.

## Step-by-Step Implementation

We'll use the default `WeatherForecastController` for examples. Logs will appear when you call the API endpoint.

### Install Serilog Packages

Serilog isn't built-in, so add it via NuGet. Two ways:

#### Package Manager Console
1. Tools → NuGet Package Manager → Package Manager Console.
2. Run these commands one by one:
   ```
   Install-Package Serilog.AspNetCore
   Install-Package Serilog.Sinks.Console
   Install-Package Serilog.Sinks.File
   ```
   
### Example 1: Basic Logging to a File (Using appsettings.json)

This logs to a daily rolling text file in a `logs` folder.

#### Step 1: Configure in `Program.cs`
Replace your `Program.cs` with this (for .NET 6+ minimal hosting model):

```csharp
using Serilog;

var builder = WebApplication.CreateBuilder(args);

// Configure Serilog logger
var logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)  // Reads from appsettings.json
    .Enrich.FromLogContext()  // Adds context like request IDs
    .CreateLogger();

builder.Logging.ClearProviders();  // Remove default loggers
builder.Logging.AddSerilog(logger);  // Add Serilog

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

**Explanation**:
- `LoggerConfiguration()`: Starts building your logger.
- `ReadFrom.Configuration()`: Pulls settings from `appsettings.json`.
- `ClearProviders()` + `AddSerilog()`: Replaces default logging with Serilog.

#### Step 2: Update `appsettings.json`
Add Serilog section:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  },
  "Serilog": {
    "Using": [ "Serilog.Sinks.File" ],
    "MinimumLevel": {
      "Default": "Information"
    },
    "WriteTo": [
      {
        "Name": "File",
        "Args": {
          "path": "logs/webapi-.log",
          "rollingInterval": "Day",
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} {CorrelationId} {Level:u3} {Username} {Message:lj}{Exception}{NewLine}"
        }
      }
    ]
  }
}
```

**Explanation**:
- `Using`: Lists sink packages (e.g., File).
- `MinimumLevel`: Logs "Information" and above (see levels below).
- `WriteTo`: Defines outputs. Here, a file sink with:
  - `path`: Log file location (creates `logs` folder).
  - `rollingInterval`: New file daily.
  - `outputTemplate`: Custom format (timestamp, level, message, etc.).

#### Step 3: Log in Your Controller
Update `Controllers/WeatherForecastController.cs`:

```csharp
using Microsoft.AspNetCore.Mvc;

namespace SerilogDemo.Controllers
{
    [ApiController]
    [Route("[controller]")]
    public class WeatherForecastController : ControllerBase
    {
        private static readonly string[] Summaries = new[]
        {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };

        private readonly ILogger<WeatherForecastController> _logger;

        public WeatherForecastController(ILogger<WeatherForecastController> logger)
        {
            _logger = logger;
        }

        [HttpGet(Name = "GetWeatherForecast")]
        public IEnumerable<WeatherForecast> Get()
        {
            _logger.LogInformation("Serilog is working! Endpoint called at {Timestamp}", DateTime.UtcNow);  // Structured log

            return Enumerable.Range(1, 5).Select(index => new WeatherForecast
            {
                Date = DateTime.Now.AddDays(index),
                TemperatureC = Random.Shared.Next(-20, 55),
                Summary = Summaries[Random.Shared.Next(Summaries.Length)]
            })
            .ToArray();
        }
    }
}
```

**Explanation**: Inject `ILogger<T>` in the constructor. Use `_logger.LogInformation()` (or `LogWarning`, `LogError`). Add `{property}` for structured data.

#### Step 4: Run and Test
- Press F5 → Swagger opens.
- Click **GET /WeatherForecast** → Execute.
- Check `bin/Debug/net6.0/logs/` for `webapi-YYYYMMDD.log`. It should show something like:
  ```
  2023-10-01 12:00:00.000 +00:00  {CorrelationId} INF  Serilog is working! Endpoint called at 2023-10-01T12:00:00Z
  ```

### Example 2: Advanced Config with Separate JSON File (Console + File)

For bigger apps, separate configs keep things clean. This logs to both console and file.

#### Step 1: Add `serilog.config.json`
Right-click project → Add → New Item → JSON File → Name it `serilog.config.json`. Add:

```json
{
  "Serilog": {
    "Using": [ "Serilog.Sinks.File", "Serilog.Sinks.Console" ],
    "MinimumLevel": {
      "Default": "Information"
    },
    "WriteTo": [
      {
        "Name": "File",
        "Args": {
          "path": "logs/webapi-.log",
          "rollingInterval": "Day",
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} {CorrelationId} {Level:u3} {Username} {Message:lj}{Exception}{NewLine}"
        }
      },
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss} [{Level:u3}] {Message}{NewLine}{Exception}"
        }
      }
    ]
  }
}
```

**Explanation**: Same as before, but adds a `Console` sink for real-time output.

#### Step 2: Update `Program.cs`
Change logger config to read the new file:

```csharp
using Serilog;
using Microsoft.Extensions.Configuration;

var builder = WebApplication.CreateBuilder(args);

// Custom config for Serilog
var config = new ConfigurationBuilder()
    .AddJsonFile("serilog.config.json")
    .Build();

var logger = new LoggerConfiguration()
    .ReadFrom.Configuration(config)
    .Enrich.FromLogContext()
    .CreateLogger();

builder.Logging.ClearProviders();
builder.Logging.AddSerilog(logger);

// Rest of your services...
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Pipeline...
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

#### Step 3: Run and Test
- F5 → Call the endpoint.
- See logs in Visual Studio's console **and** the file.

## Key Concepts and Properties

| Property/Method | Description | Example Use |
|-----------------|-------------|-------------|
| **LoggerConfiguration()** | Starts logger setup. | `new LoggerConfiguration()` |
| **ReadFrom.Configuration()** | Loads from JSON/config. | Reads `appsettings.json` or custom file. |
| **Enrich.FromLogContext()** | Adds auto-properties (e.g., request ID). | Great for web apps! |
| **CreateLogger()** | Builds the final logger. | Pass to `AddSerilog()`. |
| **MinimumLevel** | Sets log threshold (Verbose < Debug < Information < Warning < Error < Fatal). | `"Default": "Information"` logs Info+ levels. |
| **WriteTo** | Array of sinks (outputs). | File, Console, Database. |
| **outputTemplate** | Custom log format string. | Includes `{Timestamp}`, `{Level}`, `{Message}`. |

**Log Levels Explained** (From Noisiest to Quietest):
- **Verbose/Debug**: Dev-only details.
- **Information**: Normal flow (e.g., "User logged in").
- **Warning**: Potential issues (e.g., "Deprecated API used").
- **Error/Fatal**: Bugs/crashes—fix ASAP!
