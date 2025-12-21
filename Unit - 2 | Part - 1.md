# Unit - 2 | Data Access, Caching & Logging
## Data Access Fundamentals 

## What is Data Access?

**Data Access** refers to the mechanisms and techniques used by applications to interact with data storage systems (databases). It's the bridge between your application logic and the persistent data storage.

## Core Components of Data Access

### 1. Connection Management
- **Purpose**: Establish and maintain communication with the database
- **Key Elements**: Connection strings, connection pooling, timeouts
- **Why it matters**: Proper connection management prevents resource leaks and ensures efficient database usage

### 2. Query Execution
- **Purpose**: Send commands to the database and retrieve results
- **Types**: SELECT (read), INSERT/UPDATE/DELETE (write), Stored Procedures
- **Performance Impact**: Poorly optimized queries can cripple application performance

### 3. Data Mapping
- **Purpose**: Convert database rows to application objects and vice versa
- **Challenges**: Type conversion, relationship mapping, null handling
- **Importance**: Ensures data integrity between database and application

### 4. Transaction Management
- **Purpose**: Ensure data consistency during multiple operations
- **ACID Properties**: Atomicity, Consistency, Isolation, Durability
- **Use Cases**: Financial transactions, inventory updates, complex business operations

### 5. Error Handling
- **Purpose**: Manage database-specific errors gracefully
- **Types**: Connection failures, query errors, constraint violations
- **Best Practice**: Don't expose raw database errors to end-users

## Why Dapper?

**Dapper** is a lightweight Object-Relational Mapper (micro-ORM) that:
- Provides simple SQL-to-object mapping
- Has minimal overhead (high performance)
- Works with existing database schemas
- Supports both raw SQL and stored procedures
- Easy to learn and implement

## The Repository Pattern

**Purpose**: 
- Abstracts data access logic from business logic
- Provides a clean, testable interface
- Centralizes data access code
- Enables easy swapping of data sources

**Benefits**:
- **Separation of Concerns**: Business logic doesn't know about data access details
- **Testability**: Easy to mock repositories in unit tests
- **Maintainability**: Changes to data access don't affect business logic
- **Reusability**: Same data access logic can be used across the application

## Stored Procedures vs. Raw SQL

**Stored Procedures**:
- **Pros**: 
  - Pre-compiled for performance
  - Centralized business logic
  - Enhanced security (parameterized queries)
  - Database optimization opportunities
- **Cons**:
  - Can create tight coupling to database
  - Harder to version control
  - Learning curve for complex procedures

**Raw SQL**:
- **Pros**:
  - Maximum flexibility
  - Easier to version control with application code
  - Simpler for simple queries
- **Cons**:
  - Potential for SQL injection if not parameterized
  - Logic scattered across application

## Implementation Best Practices

### 1. Connection Management
```csharp
// GOOD: Using statement ensures proper disposal
using var connection = new SqlConnection(connectionString);
await connection.OpenAsync();
```

### 2. Parameterization
```csharp
// GOOD: Prevents SQL injection
var result = await connection.QueryAsync<User>(
    "SELECT * FROM Users WHERE Email = @Email", 
    new { Email = userEmail });

// BAD: String concatenation (security risk)
var sql = $"SELECT * FROM Users WHERE Email = '{userEmail}'";
```

### 3. Async/Await Pattern
```csharp
// GOOD: Non-blocking I/O operations
public async Task<IEnumerable<User>> GetUsersAsync()
{
    using var connection = _context.CreateConnection();
    return await connection.QueryAsync<User>("sp_GetAll_Users");
}

// BAD: Synchronous calls in async methods
public async Task<IEnumerable<User>> GetUsersAsync()
{
    using var connection = _context.CreateConnection();
    return connection.Query<User>("sp_GetAll_Users"); // Blocks thread
}
```

### 4. Error Handling
```csharp
try
{
    // Data access operations
}
catch (SqlException ex) when (ex.Number == 2627) // Unique constraint violation
{
    // Handle specific database errors
}
catch (Exception ex)
{
    // Log and handle general errors
    throw; // Re-throw or handle appropriately
}
```

## Performance Considerations

1. **Connection Pooling**: Reuse connections instead of creating new ones
2. **Query Optimization**: Use appropriate indexes, avoid SELECT *
3. **Batch Operations**: Minimize round trips to the database
4. **Lazy Loading**: Load data only when needed
5. **Caching**: Reduce database hits for frequently accessed data

## Security Best Practices

1. **Parameterized Queries**: Always use parameters, never string concatenation
2. **Least Privilege**: Use database users with minimal necessary permissions
3. **Input Validation**: Validate all user input before database operations
4. **Connection Strings**: Store securely, don't hardcode in code
5. **Error Messages**: Don't expose sensitive database information in errors

## Testing Strategies

1. **Unit Testing**: Mock repositories to test business logic
2. **Integration Testing**: Test with real database (use test databases)
3. **Performance Testing**: Measure query execution times
4. **Load Testing**: Ensure system handles expected traffic

## When to Use This Approach

- **Enterprise Applications**: Where data integrity and performance are critical
- **Legacy Systems**: When working with existing database schemas
- **Complex Business Logic**: When transactions and data consistency are paramount
- **Team Environments**: Where separation of concerns improves collaboration

This fundamental approach provides a robust foundation for building scalable, maintainable, and secure data access layers in your applications.

# Dapper Fundamentals

**What is Dapper?**
- Micro-ORM (Object-Relational Mapper)
- Lightweight, high-performance
- Simple to use with raw SQL
- Works with existing database schemas

## Dapper CRUD Operations with Stored Procedures

### Basic Setup
```csharp
using Dapper;
using System.Data.SqlClient;

public class DataContext
{
    private readonly string _connectionString;
    
    public DataContext(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public IDbConnection CreateConnection() => new SqlConnection(_connectionString);
}
```

### Repository Pattern Implementation

```csharp
public interface IRepository<T>
{
    Task<IEnumerable<T>> GetAllAsync();
    Task<T> GetByIdAsync(int id);
    Task<int> InsertAsync(T entity);
    Task<bool> UpdateAsync(T entity);
    Task<bool> DeleteAsync(int id);
}

public class Repository<T> : IRepository<T> where T : class
{
    private readonly DataContext _context;
    private readonly string _tableName;
    
    public Repository(DataContext context, string tableName)
    {
        _context = context;
        _tableName = tableName;
    }
    
    public async Task<IEnumerable<T>> GetAllAsync()
    {
        using var connection = _context.CreateConnection();
        return await connection.QueryAsync<T>($"sp_GetAll_{_tableName}");
    }
    
    public async Task<T> GetByIdAsync(int id)
    {
        using var connection = _context.CreateConnection();
        var parameters = new { Id = id };
        return await connection.QueryFirstOrDefaultAsync<T>(
            $"sp_GetById_{_tableName}", parameters);
    }
    
    public async Task<int> InsertAsync(T entity)
    {
        using var connection = _context.CreateConnection();
        var parameters = entity;
        return await connection.ExecuteAsync(
            $"sp_Insert_{_tableName}", parameters, commandType: CommandType.StoredProcedure);
    }
    
    public async Task<bool> UpdateAsync(T entity)
    {
        using var connection = _context.CreateConnection();
        var parameters = entity;
        var affectedRows = await connection.ExecuteAsync(
            $"sp_Update_{_tableName}", parameters, commandType: CommandType.StoredProcedure);
        return affectedRows > 0;
    }
    
    public async Task<bool> DeleteAsync(int id)
    {
        using var connection = _context.CreateConnection();
        var parameters = new { Id = id };
        var affectedRows = await connection.ExecuteAsync(
            $"sp_Delete_{_tableName}", parameters, commandType: CommandType.StoredProcedure);
        return affectedRows > 0;
    }
}
```

### Stored Procedure Examples (SQL Server)

```sql
-- Get All
CREATE PROCEDURE sp_GetAll_Users
AS
BEGIN
    SELECT * FROM Users
END

-- Get By Id
CREATE PROCEDURE sp_GetById_Users
    @Id INT
AS
BEGIN
    SELECT * FROM Users WHERE Id = @Id
END

-- Insert
CREATE PROCEDURE sp_Insert_Users
    @Name NVARCHAR(100),
    @Email NVARCHAR(100),
    @CreatedDate DATETIME
AS
BEGIN
    INSERT INTO Users (Name, Email, CreatedDate)
    VALUES (@Name, @Email, @CreatedDate)
    
    SELECT CAST(SCOPE_IDENTITY() as int) as NewId
END

-- Update
CREATE PROCEDURE sp_Update_Users
    @Id INT,
    @Name NVARCHAR(100),
    @Email NVARCHAR(100)
AS
BEGIN
    UPDATE Users 
    SET Name = @Name, Email = @Email
    WHERE Id = @Id
END

-- Delete
CREATE PROCEDURE sp_Delete_Users
    @Id INT
AS
BEGIN
    DELETE FROM Users WHERE Id = @Id
END
```

## Best Practices

1. **Connection Management**
   - Use `using` statements for connections
   - Consider connection pooling

2. **Error Handling**
   ```csharp
   try
   {
       // Dapper operations
   }
   catch (SqlException ex)
   {
       // Handle specific database errors
   }
   catch (Exception ex)
   {
       // Handle general errors
   }
   ```

3. **Transaction Support**
   ```csharp
   using var connection = _context.CreateConnection();
   using var transaction = connection.BeginTransaction();
   
   try
   {
       // Multiple Dapper operations
       transaction.Commit();
   }
   catch
   {
       transaction.Rollback();
       throw;
   }
   ```

4. **Async Operations**
   - Always use async/await patterns
   - Avoid mixing sync and async code

5. **Mapping Complex Types**
   ```csharp
   // One-to-many relationship
   var sql = @"SELECT * FROM Orders o
               LEFT JOIN OrderItems oi ON o.Id = oi.OrderId";
   
   var orderDictionary = new Dictionary<int, Order>();
   
   var orders = connection.Query<Order, OrderItem, Order>(
       sql,
       (order, orderItem) =>
       {
           if (!orderDictionary.TryGetValue(order.Id, out var currentOrder))
           {
               currentOrder = order;
               orderDictionary.Add(currentOrder.Id, currentOrder);
           }
           
           if (orderItem != null)
               currentOrder.OrderItems.Add(orderItem);
               
           return currentOrder;
       },
       splitOn: "OrderItemId"
   ).Distinct().ToList();
   ```

## Advantages of This Approach

1. **Performance**: Dapper is very fast compared to full ORMs
2. **Control**: Full control over SQL and stored procedures
3. **Testability**: Easy to mock and unit test
4. **Maintainability**: Repository pattern provides clean separation
5. **Scalability**: Works well with existing database schemas
