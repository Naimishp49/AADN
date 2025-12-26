
### 🎯 **Task Title: "Library Management API – Mocking & Testing"**

#### 📌 **Objective**
Build and test a simple RESTful API for a Library Management System. Students will:
1. Implement core logic with proper separation of concerns (e.g., services, repositories).
2. Write **unit tests using Moq** to isolate and test business logic.
3. Write **integration/API tests** (using `HttpClient` + `TestServer` or `WebApplicationFactory`) to verify HTTP behavior.

---

## 🧱 Part 1: Setup the Project (Prerequisite)

Create a solution with:
- **`LibraryApi`** – ASP.NET Core Web API (.NET 8 recommended)
- **`LibraryApi.Tests`** – xUnit Test Project (.NET 8)
  - Includes references to:
    - `Microsoft.NET.Test.Sdk`
    - `xunit`, `xunit.runner.visualstudio`
    - `Moq`
    - `Microsoft.AspNetCore.Mvc.Testing` (for integration tests)

---

## 📚 Domain: Library System

Entities:
```csharp
public class Book
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Author { get; set; }
    public bool IsAvailable { get; set; } = true;
}

public class BorrowRecord
{
    public int Id { get; set; }
    public int BookId { get; set; }
    public string PatronName { get; set; }
    public DateTime BorrowedOn { get; set; }
    public DateTime? ReturnedOn { get; set; }
}
```

Endpoints (CRUD + Borrow/Return):
- `GET /api/books` → List all books  
- `GET /api/books/{id}` → Get book by ID  
- `POST /api/books` → Add new book  
- `POST /api/books/{id}/borrow` → Borrow a book (patron name in body)  
- `POST /api/books/{id}/return` → Return a book  

---

## 🛠️ Part 2: Implement the API

Design with clean architecture in mind:
```
LibraryApi/
├── Controllers/
│   └── BooksController.cs
├── Services/
│   ├── IBookService.cs
│   └── BookService.cs
├── Repositories/
│   ├── IBookRepository.cs
│   └── InMemoryBookRepository.cs (or use EF Core later)
```

❗ Ensure `BookService` depends on `IBookRepository` (not concrete type).

Example interface:
```csharp
public interface IBookRepository
{
    Task<Book?> GetByIdAsync(int id);
    Task<List<Book>> GetAllAsync();
    Task<Book> AddAsync(Book book);
    Task UpdateAsync(Book book);
}
```

---

## ✅ Part 3: Unit Testing with Moq (Focus: `BookService`)

### 🔹 Task: Write unit tests for `BookService`

**Scenarios to test:**
1. `BorrowBookAsync` → Should mark book as unavailable & create borrow record *(mock repo + verify calls)*  
2. `BorrowBookAsync` → Throws when book not found *(mock throws exception or returns null)*  
3. `BorrowBookAsync` → Throws when book already borrowed  
4. `ReturnBookAsync` → Marks book available & sets `ReturnedOn`  

💡 **Requirements:**
- Use `Moq` to mock `IBookRepository`.
- Verify state changes and method calls (`.Verify()`).
- Test error cases (exceptions, validation).

**Example test skeleton:**
```csharp
[Fact]
public async Task BorrowBookAsync_WithAvailableBook_ShouldMarkAsUnavailable()
{
    // Arrange
    var mockRepo = new Mock<IBookRepository>();
    var book = new Book { Id = 1, Title = "Test", IsAvailable = true };
    mockRepo.Setup(r => r.GetByIdAsync(1)).ReturnsAsync(book);
    
    var service = new BookService(mockRepo.Object);
    
    // Act
    await service.BorrowBookAsync(1, "Alice");
    
    // Assert
    Assert.False(book.IsAvailable);
    mockRepo.Verify(r => r.UpdateAsync(It.Is<Book>(b => b.Id == 1 && !b.IsAvailable)), Times.Once);
}
```

📌 **Deliverable**: 5+ well-named unit tests covering happy & sad paths.

---

## 🌐 Part 4: API / Integration Testing

Use `WebApplicationFactory<T>` to test HTTP endpoints **without mocking HTTP** (real pipeline, but in-memory).

### 🔹 Task: Write integration tests for `BooksController`

**Scenarios:**
1. `GET /api/books` → Returns 200 + list of books  
2. `POST /api/books` → Creates book, returns 201 + location header  
3. `POST /api/books/1/borrow` → 200 if successful, 400 if invalid (e.g., book not found or unavailable)  
4. `POST /api/books/1/return` → 200 when book was borrowed  

💡 **Requirements:**
- Use `WebApplicationFactory<Program>` to create test server.
- Use `HttpClient` to send real HTTP requests.
- Seed initial data (override services to inject test repo or use DB reset).
- Assert status codes, response content, and side effects.

**Example:**
```csharp
public class BooksControllerTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public BooksControllerTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
        // Optional: Reset DB or inject test data here
    }

    [Fact]
    public async Task GetBooks_ReturnsListOfBooks()
    {
        // Act
        var response = await _client.GetAsync("/api/books");
        
        // Assert
        response.EnsureSuccessStatusCode();
        var books = await response.Content.ReadFromJsonAsync<List<Book>>();
        Assert.NotNull(books);
        Assert.True(books.Count > 0);
    }
}
```

📌 **Deliverable**: 4+ integration tests covering key workflows (borrow → return flow).

---

## 🏆 Bonus Challenges (Optional)
1. Write a **test that simulates race condition** in borrowing (advanced: use `Task.WhenAll`).

---

