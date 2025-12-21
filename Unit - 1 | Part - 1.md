## 2501CS634 - Advanced .NET with Modern Architectures | Sem. - 6

# **Unit-1 | Fundamentals of Distributed Systems & Microservices**

# Table of Contents

1. What Are Distributed Systems?
2. Why E-Commerce Needs Distributed Systems
3. CAP Theorem 
4. Monolith vs Microservices 
5. E-Commerce Project Architecture Sample
6. Microservices Demo
7. Step-By-Step: Creating Microservices
---

# 1. What Are Distributed Systems?

A **Distributed System** is a group of computers (nodes) working together over a network to appear as a single system to users.

A distributed system has:

* Independent nodes
* No shared memory
* Communicate via REST/gRPC/messages
* Partial failures
* Clock issues
* Network delays

Examples:

* Google Search
* Amazon E-Commerce
* Netflix
* Swiggy / Zomato
* UPI Payments

---

# 2.  Why E-Commerce Needs Distributed Systems?

| Need               | Why Important   | Distributed Systems Solution |
| ------------------ | --------------- | ---------------------------- |
| Millions of users  | High traffic    | Scale horizontally           |
| Product browsing   | Must be fast    | Caching + AP systems         |
| Payment processing | Must be correct | CP databases                 |
| Zero downtime      | Sales days      | Redundancy + replicas        |
| Multiple teams     | Parallel work   | Microservices                |

---
# 3. CAP Theorem

##  3.1 Why CAP Theorem Exists?

Imagine you created a **distributed system** — which means:

* Data is stored on **multiple servers** (nodes)
* These servers may be in **different places** (cities or countries)
* Users can read/write data from **any server**

Example:
Flipkart is deployed in:

* Mumbai Data Center
* Bengaluru Data Center
* Singapore Data Center

All three must behave like **one single system**.

But what if the network between Mumbai ↔ Singapore breaks?

Now the two sides **cannot talk**.

This is called a **Network Partition**.

**CAP Theorem explains what happens when such a failure occurs.**

---

## 3.2 The CAP Theorem Statement

A distributed system can provide **ONLY TWO** of the following **at the same time**:

1. **Consistency (C)**
2. **Availability (A)**
3. **Partition Tolerance (P)**

**During a network problem (partition), you must choose:**

* **CA** is impossible because P is mandatory
* So the real choices are:

### ✔ **CP** = Consistency + Partition Tolerance

### ✔ **AP** = Availability + Partition Tolerance

**We NEVER get all three during a partition.**

---

## 3.3 Understanding the Terms

### **(C) Consistency – “Everyone sees the same data at the same time”**

Example:
Flipkart product price = ₹999.

If you update the price to ₹899:

* All users
* In all cities
* On all servers

should see **₹899 immediately**.

This is **strong consistency**.

Like:

* Teacher writes marks in register
* All students get the updated marks immediately

---

### **(A) Availability – “System should ALWAYS respond”**

No matter what happens, the system must give a response.

Example:

If you ask, *“Show me the product price”*, the server must:

* return the price
  OR
* return an error message

but **it must reply**.

It cannot stay silent.

---

### **(P) Partition Tolerance – “System keeps working even if servers cannot talk to each other”**

Example:

Mumbai server ↔ Singapore server
⚠ **Network breaks**

But:

* Mumbai users should still shop
* Singapore users should still shop

You cannot shut down the website because two data centers can't communicate.

**Partition tolerance is NON-NEGOTIABLE.**

Real cloud systems like AWS, Azure, GCP face network failures all the time.

---

## 3.4 Why We Cannot Have All Three?

Imagine your data is in two servers:

```
Server A (Mumbai)  
Server B (Singapore)
```

Network breaks ⇒ A cannot talk to B

A user updates product price on Server A:
**₹999 → ₹899**

Now:

### If you keep the system **Available (A)**

Both A and B must respond to customers.
But B cannot get the updated price (network fail).
So B will show **wrong/old price (₹999)**.

Consistency is broken ❌
So in this case, you get **AP system**.

---

### If you keep the system **Consistent (C)**

Server B will say:
“I can’t reach Server A.
So I cannot show any price until I confirm the new value.”

So it **refuses to respond**.
Availability is broken ❌
But consistency is maintained.

You get a **CP system**.

---
## 3.5 E-Commerce Mapping of CA

| Module             | Should Choose | Reason                            |
| ------------------ | ------------- | --------------------------------- |
| Product Catalog    | **AP**        | Can show slightly stale data      |
| Search Suggestions | **AP**        | Delay acceptable                  |
| Shopping Cart      | **AP**        | Performance > perfect consistency |
| Order Placement    | **CP**        | Must not lose order               |
| Payment Processing | **CP**        | Money cannot be inconsistent      |

Thus:

* Product Service → **AP**
* Order Service → **CP**
* Payment Service → **CP**
---

## Remember

**During a network partition, you must sacrifice either:**

* **Availability (A)**
  or
* **Consistency (C)**

There is no magic to keep both.

This is the core message of CAP.

---

# 4. Monolith vs Microservices

### 1. Monolith (Monolithic)

### What it is?

A **monolithic application** is built as **one single project**.
All features run inside the same codebase and use **one shared database**.

### How it works?

Example: An **e-commerce monolith**
The same project contains:

* Product management
* User login
* Cart
* Orders
* Payments
* Inventory
* Notifications

All deployed together in one package.

### Advantages

* Easy to start and develop
* Simple deployment
* Best for small teams and simple projects

### Disadvantages

* Hard to scale specific modules (e.g., only Orders)
* One error may crash the entire app
* Codebase grows too large over time
* Slow deployment because everything is packaged together
* Updating one feature requires redeploying the entire system

### Real-Time Example

Imagine you built a single app called **MegaShop**.
If the **payment feature fails**, customers cannot browse products or place orders because the entire application is one unit.
Also, if you modify only the cart module, you still must redeploy the **entire monolith**.

---

### 2. Microservices

### What it is?

In microservices, the application is **divided into small, independent services**.
Each service has **its own codebase**, and often **its own database**.
Services communicate using **APIs**.

Example microservices in e-commerce:

* Product Service
* Order Service
* Payment Service
* Inventory Service
* User/Authentication Service
* Notification/Email Service

Each service is like a mini-application.

### Advantages

* Each service can scale independently
* Teams can work independently
* Easy to deploy updates for one service
* Technology freedom (each service can use different tech)

### Disadvantages

* More setup (API gateway, communication handling)
* Debugging across services becomes harder
* Requires strong DevOps practices (Docker, CI/CD)

### Real-Time Example

In Amazon, when a user places an order:

* Product Service checks availability
* Order Service creates the order
* Payment Service processes payment
* Notification Service sends email

If Payment Service fails, only payments stop working—
but browsing products or adding items to the cart still works,
because each service is independent.

---

## Difference (Monolithic v/s Microservices)

| Feature              | Monolithic                        | Microservices                                  |
| -------------------- | --------------------------------- | ---------------------------------------------- |
| **Structure**        | One big application               | Many small applications                        |
| **Modules**          | All modules are connected tightly | Each module is a separate service              |
| **Database**         | One database for the whole system | Each service can have its own database         |
| **Development**      | One team works on entire code     | Different teams can work on different services |
| **Technology**       | Must use one backend language     | Each service can use any language              |
| **Deployment**       | Everything deployed together      | Each service deployed separately               |
| **Scaling**          | Must scale entire application     | Only scale the service that needs it           |
| **Failure**          | One error can crash the whole app | Only one service fails, others keep working    |
| **Debugging**        | Easier (because all-in-one)       | Harder (services communicate)                  |
| **Speed of Updates** | Slow (big build + deploy)         | Fast (small service deploys quickly)           |
| **Flexibility**      | Low                               | Very high                                      |


---
# 5. E-Commerce Architecture [Sample]


### **Diagram 1 — Monolith Architecture (Startup Stage)**
<img width="930" height="503" alt="Screenshot 2025-11-21 103204" src="https://github.com/user-attachments/assets/cb3accb5-4786-486e-a103-35b31f7d28ac" />

### **Explanation**

* A single application handles all business logic.
* One database stores everything: products, orders, payments, users.
* Easy to develop in the early stage.
* Becomes slow and harder to update as the app grows.

---

### **Diagram 2 — Microservices Architecture (Scale Stage)**

<img width="874" height="478" alt="Screenshot 2025-11-21 103146" src="https://github.com/user-attachments/assets/081f5740-79db-4003-81ee-c020bcb29aab" />

### **Explanation of Microservices Architecture**

* **API Gateway** routes all external requests to the correct service.
* Each microservice has:

  * Its own codebase
  * Its own database
  * Its own deployment
* Services scale independently (e.g., more servers only for Order Service during Diwali sale).
* Failure in one service does not break others.

---
# 6. Microservices Demo

### **Project Overview**

We are building a **mini E-Commerce system** using **Microservices Architecture**.
The system will contain **three independent services**:

1. **Product Service** – Shows products
2. **Order Service** – Places orders
3. **Payment Service** – Simulates payment gateway

Each service is fully independent with its own:

* API project
* Business logic layer
* Domain models
* Infrastructure layer

This is exactly how real companies such as **Amazon, Flipkart, Meesho, Myntra** build their scalable backend systems.


# 7. Step-By-Step Microservices Creation

You will be able to run all 3 services at once and test the full flow:

```
1. GET http://localhost:5001/api/products          → Returns products
2. POST http://localhost:5002/api/orders/place      → Creates order (calls Product Service internally)
3. POST http://localhost:5003/api/payment/pay       → Simulates payment
```

Let’s start!

### Folder Structure

```
ECommerce.Microservices
     ├── ProductService
     │     ├── ProductService.Api
     │     ├── ProductService.Application
     │     ├── ProductService.Domain
     │     └── ProductService.Infrastructure
     ├── OrderService
     │     ├── OrderService.Api
     │     ├── Application
     │     ├── Domain
     │     └── Infrastructure
     └── PaymentService
           ├── Api
           ├── Application
           ├── Domain
           └── Infrastructure
```
---

### Step 1: Create Blank Solution

1. Open **Visual Studio**
2. Click **Create a new project**
3. Search → **Blank Solution**
4. Click **Next**
5. Solution Name: `ECommerce.Microservices`
6. Location: Choose your folder
7. Click **Create**

---

### Step 2: Create All Projects (3 Services × 4 Layers = 12 Projects)

Right-click on Solution → **Add** → **New Project**

#### Product Service

| Project Name                        | Template                    |
|-------------------------------------|-----------------------------|
| `ProductService.Api`                | ASP.NET Core Web API        |
| `ProductService.Application`        | Class Library (.NET)        |
| `ProductService.Domain`             | Class Library (.NET)        |
| `ProductService.Infrastructure`     | Class Library (.NET)        |

#### Order Service

| Project Name                        | Template                    |
|-------------------------------------|-----------------------------|
| `OrderService.Api`                  | ASP.NET Core Web API        |
| `OrderService.Application`          | Class Library (.NET)        |
| `OrderService.Domain`               | Class Library (.NET)        |
| `OrderService.Infrastructure`       | Class Library (.NET)        |

#### Payment Service

| Project Name                        | Template                    |
|-------------------------------------|-----------------------------|
| `PaymentService.Api`                | ASP.NET Core Web API        |
| `PaymentService.Application`        | Class Library (.NET)        |
| `PaymentService.Domain`             | Class Library (.NET)        |
| `PaymentService.Infrastructure`     | Class Library (.NET)        |

**Tip**: After creating each project, drag it into a folder (right-click solution → Add → New Solution Folder) named `ProductService`, `OrderService`, `PaymentService` for clean structure.

<img width="589" height="500" alt="image" src="https://github.com/user-attachments/assets/53b6607e-8d07-4652-a3ea-d51556545351" />

---

### Step 3: Set Correct Project References (Very Important!)

Right-click on each project → **Add** → **Project Reference**
<img width="1920" height="1020" alt="image" src="https://github.com/user-attachments/assets/272f5fc6-f294-4e48-8dfc-54b0a40b9e5b" />

#### ProductService References
| Project                          | References To                     |
|----------------------------------|-----------------------------------|
| ProductService.Api               | → Application                     |
| ProductService.Application       | → Domain + Infrastructure        |
| ProductService.Infrastructure    | → Domain                          |

#### OrderService References
| Project                          | References To                     |
|----------------------------------|-----------------------------------|
| OrderService.Api                 | → Application                     |
| OrderService.Application         | → Domain + Infrastructure        |
| OrderService.Infrastructure     | → Domain                          |

#### PaymentService References
| Project                          | References To                     |
|----------------------------------|-----------------------------------|
| PaymentService.Api               | → Application                     |
| PaymentService.Application       | → Domain + Infrastructure        |
| PaymentService.Infrastructure   | → Domain                          |

---

### Step 4: Set Multiple Startup Projects (Run All 3 APIs Together)

1. Right-click Solution → **Properties**
2. Configuration Properties → **Multiple startup projects**
3. Set all three `.Api` projects to **Start**:

| Project                  | Action  |
|--------------------------|---------|
| ProductService.Api       | Start   |
| OrderService.Api         | Start   |
| PaymentService.Api       | Start   |

4. Click OK

---

### Step 5: Configure Different Ports (Avoid Port Conflict) -- OPTIONAL

Open each `Properties/launchSettings.json` inside `.Api` projects and change `applicationUrl`:

#### ProductService.Api → launchSettings.json
```json
"applicationUrl": "http://localhost:5001",
"ASPNETCORE_ENVIRONMENT": "Development"
```

#### OrderService.Api → launchSettings.json
```json
"applicationUrl": "http://localhost:5002",
"ASPNETCORE_ENVIRONMENT": "Development"
```

#### PaymentService.Api → launchSettings.json
```json
"applicationUrl": "http://localhost:5003",
"ASPNETCORE_ENVIRONMENT": "Development"
```

---

### Step 6: Add Full Working Code 

#### 1. ProductService – Fully Working

**ProductService.Domain/Entities/Product.cs**
```csharp
namespace ProductService.Domain.Entities;
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;
    public decimal Price { get; set; }
    public int Stock { get; set; }
}
```

**ProductService.Application/DTOs/ProductDto.cs**
```csharp
namespace ProductService.Application.DTOs;
public record ProductDto(int Id, string Name, decimal Price, int Stock);
```

**ProductService.Application/Services/  .cs**
```csharp
namespace ProductService.Application.Services;
public interface IProductService
{
    Task<IEnumerable<ProductDto>> GetAllAsync();
}
```

**ProductService.Application/Services/ProductService.cs**
```csharp
using ProductService.Application.DTOs;
using ProductService.Domain.Entities;

namespace ProductService.Application.Services;

public class ProductService : IProductService
{
    private static readonly List<Product> _products = new()
    {
        new() { Id = 1, Name = "iPhone 16 Pro", Price = 129900, Stock = 10 },
        new() { Id = 2, Name = "MacBook Air M3", Price = 114900, Stock = 5 },
        new() { Id = 3, Name = "Samsung S24 Ultra", Price = 119999, Stock = 8 }
    };

    public Task<IEnumerable<ProductDto>> GetAllAsync()
    {
        var dtos = _products.Select(p => new ProductDto(p.Id, p.Name, p.Price, p.Stock));
        return Task.FromResult(dtos);
    }
}
```

**ProductService.Api/Controllers/ProductsController.cs** (Replace default)
```csharp
using Microsoft.AspNetCore.Mvc;
using ProductService.Application.Services;

namespace ProductService.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpGet]
    public async Task<IActionResult> GetProducts()
    {
        var products = await _productService.GetAllAsync();
        return Ok(new
        {
            Message = "Product Service is LIVE!",
            Timestamp = DateTime.UtcNow,
            Data = products
        });
    }
}
```

**ProductService.Api/Program.cs** (Add DI)
```csharp
using ProductService.Application.Services;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Register Application Services
builder.Services.AddScoped<IProductService, ProductService>();

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

---

#### 2. OrderService – Calls Product Service Internally

**OrderService.Domain/Entities/Order.cs**
```csharp
namespace OrderService.Domain.Entities;
public class Order
{
    public Guid Id { get; set; }
    public int ProductId { get; set; }
    public int Quantity { get; set; }
    public decimal TotalAmount { get; set; }
    public string Status { get; set; } = "Pending Payment";
}
```

**OrderService.Application/DTOs/PlaceOrderRequest.cs**
```csharp
namespace OrderService.Application.DTOs;
public record PlaceOrderRequest(int ProductId, int Quantity);
```

**OrderService.Application/DTOs/ProductDto.cs** (same as ProductService)
```csharp
namespace OrderService.Application.DTOs;
public record ProductDto(int Id, string Name, decimal Price, int Stock);
```

**OrderService.Application/Services/IOrderService.cs**
```csharp
using OrderService.Application.DTOs;

namespace OrderService.Application.Services;
public interface IOrderService
{
    Task<OrderResponse> PlaceOrderAsync(PlaceOrderRequest request);
}

public record OrderResponse(Guid OrderId, string Status, string Message);
```

**OrderService.Application/Services/OrderService.cs**
```csharp
using OrderService.Application.DTOs;
using System.Net.Http.Json;
using System.Text.Json;

namespace OrderService.Application.Services;

public class OrderService : IOrderService
{
    private readonly HttpClient _httpClient;

    public OrderService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<OrderResponse> PlaceOrderAsync(PlaceOrderRequest request)
    {
        // Call Product Service to validate product
        var response = await _httpClient.GetAsync($"http://localhost:5001/api/Products");
        if (!response.IsSuccessStatusCode)
        {
            return new OrderResponse(Guid.Empty, "Failed",
                $"Product Service returned {response.StatusCode}");
        }

        var jsonContent = await response.Content.ReadAsStringAsync();
        using var doc = JsonDocument.Parse(jsonContent);
        var dataArray = doc.RootElement
                           .GetProperty("data")
                           .EnumerateArray();

        var products = dataArray.Select(item => new ProductDto(
            Id: item.GetProperty("id").GetInt32(),
            Name: item.GetProperty("name").GetString() ?? "Unknown",
            Price: item.GetProperty("price").GetDecimal(),
            Stock: item.GetProperty("stock").GetInt32()
        )).ToList();

        var product = products.FirstOrDefault(p => p.Id == request.ProductId);

        if (product == null)
            return new OrderResponse(Guid.Empty, "Failed", "Product not found");

        if (product.Stock < request.Quantity)
            return new OrderResponse(Guid.Empty, "Failed", "Insufficient stock");

        var total = product.Price * request.Quantity;
        var orderId = Guid.NewGuid();

        return new OrderResponse(orderId, "Pending Payment",
            $"Order placed successfully! Total: ₹{total:N0}");
    }
}
```

**OrderService.Api/Program.cs**
```csharp
using OrderService.Application.Services;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
// Works with your current running ports + HTTPS
var handler = new HttpClientHandler()
{
    ServerCertificateCustomValidationCallback =
        HttpClientHandler.DangerousAcceptAnyServerCertificateValidator
};
// Register HttpClient for Product Service
builder.Services.AddHttpClient<IOrderService, OrderService>(client =>
{
    client.BaseAddress = new Uri("http://localhost:5001");
});
.ConfigurePrimaryHttpMessageHandler(() => handler);

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.MapControllers();
app.Run();
```

**OrderService.Api/Controllers/OrdersController.cs**
```csharp
using Microsoft.AspNetCore.Mvc;
using OrderService.Application.DTOs;
using OrderService.Application.Services;

namespace OrderService.Api.Controllers;

[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _orderService;

    public OrdersController(IOrderService orderService)
    {
        _orderService = orderService;
    }

    [HttpPost("place")]
    public async Task<IActionResult> PlaceOrder([FromBody] PlaceOrderRequest request)
    {
        var result = await _orderService.PlaceOrderAsync(request);
        if (result.OrderId == Guid.Empty)
            return BadRequest(result);

        return Ok(new
        {
            Message = result.Message,
            OrderId = result.OrderId,
            Status = result.Status,
            Timestamp = DateTime.UtcNow
        });
    }
}
```

---

#### 3. PaymentService – Simple Simulation

**PaymentService.Api/Controllers/PaymentController.cs**
```csharp
using Microsoft.AspNetCore.Mvc;

namespace PaymentService.Api.Controllers;

[ApiController]
[Route("api/payment")]
public class PaymentController : ControllerBase
{
    [HttpPost("pay")]
    public IActionResult Pay([FromBody] PaymentRequest request)
    {
        // Simulate payment success
        return Ok(new
        {
            PaymentId = Guid.NewGuid().ToString().Substring(0, 8),
            Status = "Success",
            Amount = request.Amount,
            Message = "Payment successful! Thank you for shopping."
        });
    }
}

public record PaymentRequest(decimal Amount);
```

**PaymentService.Api/Program.cs**
```csharp
var builder = WebApplication.CreateBuilder(args);
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
app.MapControllers();
app.Run();
```

---

### Final Step: Run & Test

1. Press **Ctrl + F5** (Start Without Debugging)
2. All 3 Swagger pages will open:

- http://localhost:5001/swagger → Product Service
- http://localhost:5002/swagger → Order Service
- http://localhost:5003/swagger → Payment Service

### Test Full Flow

1. GET → `/api/products` → See products
2. POST → `/api/orders/place`
   Body:
   ```json
   { "productId": 1, "quantity": 2 }
   ```
   → Returns order with "Pending Payment"
3. POST → `/api/payment/pay`
   ```json
   { "amount": 259800 }
   ```
   → Payment success!
