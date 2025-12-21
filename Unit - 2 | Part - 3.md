
# **Unit - 2. Data Access, Caching & Logging**

## **1. Caching Fundamentals**

### Why Cache?
- Reduce DB load & latency.
- Improve scalability & user experience.

### Caching Strategies

| Strategy | When to Use | .NET Tools |
|---------|-------------|------------|
| **In-Memory Cache** | Single-instance apps, short-lived data | `IMemoryCache` |
| **Distributed Cache** | Multi-instance (e.g., cloud), shared state | `IDistributedCache` (Redis, SQL Server, NCache) |
| **Cache-Aside (Lazy Loading)** | Most common: load on miss | Manual check → DB → store |
| **Write-Through** | Update cache & DB together (strong consistency) | Custom logic or Redis modules |
| **Cache-Aside with Expiry** | Prevent stale data | Sliding/Absolute expiration |

### Example: Cache-Aside in Repository
```csharp
public class CachedProductRepository : IProductRepository
{
    private readonly IProductRepository _repo;
    private readonly IMemoryCache _cache;

    public CachedProductRepository(IProductRepository repo, IMemoryCache cache)
    {
        _repo = repo;
        _cache = cache;
    }

    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _cache.GetOrCreateAsync($"product_{id}", async entry =>
        {
            entry.SlidingExpiration = TimeSpan.FromMinutes(10);
            return await _repo.GetByIdAsync(id);
        });
    }
}
```

> ✅ Best Practices:
> - Use **cache keys** with namespace (e.g., `"prod:123"`).
> - Set reasonable **expiration**.
> - **Invalidate** on writes (e.g., clear cache in `UpdateAsync`/`DeleteAsync`).

---

## **2. Logging Essentials**

### Goals
- Diagnose issues (errors, performance).
- Audit data access (who queried what).
- Monitor system health.

### .NET Logging Abstraction: `ILogger<T>`

#### Setup (ASP.NET Core)
```csharp
// Inject ILogger<ProductRepository>
public class ProductRepository : IProductRepository
{
    private readonly ILogger<ProductRepository> _logger;

    public ProductRepository(ILogger<ProductRepository> logger, /* ... */)
        => _logger = logger;
}
```

#### Logging Levels & Use Cases
| Level | Use Case |
|------|----------|
| `Trace` | Detailed dev/debug info (e.g., raw SQL, params) — *disable in prod*. |
| `Debug` | Diagnostics (e.g., cache miss/hit). |
| `Information` | Key events (e.g., “Product 5 updated”). |
| `Warning` | Recoverable issues (e.g., retryable DB timeout). |
| `Error` | Failures (e.g., exception in query). |
| `Critical` | Catastrophic (e.g., DB connection pool exhausted). |

#### Example in Repository
```csharp
public async Task<Product?> GetByIdAsync(int id)
{
    _logger.LogDebug("Fetching product with ID {ProductId}", id);
    
    try
    {
        var product = await _connection.QuerySingleOrDefaultAsync<Product>(
            "SELECT * FROM Products WHERE Id = @Id", new { Id = id });

        if (product == null)
            _logger.LogInformation("Product with ID {ProductId} not found", id);
        
        return product;
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Failed to retrieve product with ID {ProductId}", id);
        throw; // or wrap & rethrow
    }
}
```

### 🔐 Sensitive Data
- **Never log passwords, tokens, PII(Personally Identifiable Information like Full name, Social Security Number, driver's license number, email address, physical address, financial information (credit card numbers, bank accounts), passport numbers, and biometric data)**.
- Use `{@Object}` for structured logging (Serilog), but sanitize first.

### Tools
- **Built-in**: `Microsoft.Extensions.Logging`
- **Structured Logging**: Serilog (with sinks: Seq, Elasticsearch, etc.)
- **Diagnostics**: Application Insights (Azure), OpenTelemetry

---

## ✅ Summary Checklist

| Area | Key Action |
|------|------------|
| **Dapper CRUD** | Use parameters, async methods, `QuerySingle` vs `QuerySingleOrDefault`. |
| **Stored Procedures** | Use `CommandType.StoredProcedure`; `DynamicParameters` for outputs. |
| **Repository** | Interface-driven, DI-registered, keep logic testable. |
| **Caching** | Cache-aside + expiry + invalidation; prefer distributed for scale. |
| **Logging** | Use `ILogger<T>`, structured logs, appropriate levels, no secrets. |

---

