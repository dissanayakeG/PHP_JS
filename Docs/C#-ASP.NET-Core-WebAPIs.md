# ASP.NET Core minimal api setup

```cs
dotnet new webapi -n MyMinimalApi --no-https
dotnet new web -o TodoApi?
```

- install rest-client vs-code extension
- Create directory called Dtos
- Create new Dto class here
- Notice the record before class keyword
- this is perfect for Dtos, because they are not going to change.
- Dto use for data transfer between client and server

```cs
public record class WineItem
{
    public Guid Id { get; init; }
    public string Name { get; init; } = string.Empty;
    public string Description { get; init; } = string.Empty;
    public decimal Price { get; init; }
    public DateTimeOffset CreatedDate { get; init; }

}
```

- init: Allows setting the value of Id only during object initialization (e.g., when creating the object or using an object initializer). After initialization, the property becomes read-only.
- This is useful for immutable objects, like records, where you want properties to be set only once when the object is created and not changed afterward

- Create a List of WineItem in the main class and return them from /wines api

```cs
List<WineItem> wineItems = new()
{
    new WineItem { 
        Id = Guid.NewGuid(), 
        Name = "Chateau Margaux", 
        Description = "A full-bodied red wine from Bordeaux, France.", 
        Price = 299.99M,
        CreatedDate = DateTimeOffset.UtcNow 
    },
    ...
};

or
var wineItems = new List<WineItem>
{
    new WineItem { 
        Id = Guid.NewGuid(), 
        Name = "Chateau Margaux", 
        Description = "A full-bodied red wine from Bordeaux, France.", 
        Price = 299.99M, 
        CreatedDate = DateTimeOffset.UtcNow 
    },
    ...
};

app.MapGet("/wines", () =>
{
    return Results.Ok(wineItems);
});
```

- Create request.http file in the root path and add /wine request
- GET http://localhost:5177/wines
- run the application by
```cs
dotnet build
dotnet run
```

- Now let's move endpoints into separate namespace
- Create Endpoints directory in the root path and create WineItems.cs class there
- This class is a static, so all the members here will be readonly and static

```cs
using System;
using WineStore.Api.Dtos;

namespace WineStore.Api.Endpoints;

public static class WineItems
{
    private static readonly List<WineItem> wineItems = new()
    {
        new WineItem { Id = Guid.NewGuid(), Name = "Chateau Margaux", Description = "A full-bodied red wine from Bordeaux, France.", Price = 299.99M, CreatedDate = DateTimeOffset.UtcNow },
        new WineItem { Id = Guid.NewGuid(), Name = "Screaming Eagle", Description = "A cult Cabernet Sauvignon from Napa Valley, California.", Price = 999.99M, CreatedDate = DateTimeOffset.UtcNow },
        new WineItem { Id = Guid.NewGuid(), Name = "Penfolds Grange", Description = "An iconic Australian Shiraz with rich flavors.", Price = 849.99M, CreatedDate = DateTimeOffset.UtcNow },
        new WineItem { Id = Guid.NewGuid(), Name = "Vega Sicilia Unico", Description = "A prestigious Spanish red wine with complex aromas.", Price = 499.99M, CreatedDate = DateTimeOffset.UtcNow },
        new WineItem { Id = Guid.NewGuid(), Name = "Dom Pérignon", Description = "A luxurious Champagne known for its elegance and finesse.", Price = 199.99M, CreatedDate = DateTimeOffset.UtcNow }
    };
    public static WebApplication MapWineItemsEndpoints(this WebApplication app)
    {
        app.MapGet("/wines", () =>
        {
            return Results.Ok(wineItems);
        });
        return app;
    }
}
```
- Now cleanup the main file and just add below code, the /wines api should still be working

```cs
app.MapWineItemsEndpoints();
```

# ASP.NET Core Web API Setup: TodoApi

## Create the Project

```bash
dotnet new webapi --use-controllers -o TodoApi
cd TodoApi
```

## Add Required Packages

### In-Memory Database (for quick prototyping)

```bash
dotnet add package Microsoft.EntityFrameworkCore.InMemory
```

### MySQL Support

```bash
dotnet add package Pomelo.EntityFrameworkCore.MySql
```

### Swagger (OpenAPI UI)

```bash
dotnet add package NSwag.AspNetCore
```

## Enable HTTPS for Local Development (Optional)

```bash
dotnet dev-certs https --trust
# Select "Yes" when prompted
dotnet run --launch-profile https
```

> **Note:** If you enable HTTPS, make sure to use `https://localhost` for local API calls.

##  Define the Model

`Models/TodoItem.cs`

```csharp
namespace TodoApi.Models;

public class TodoItem
{
    public long Id { get; set; }
    public string? Name { get; set; }
    public bool IsComplete { get; set; }
}
```

## Create the Database Context

`Models/TodoContext.cs`

```csharp
using Microsoft.EntityFrameworkCore;

namespace TodoApi.Models;

public class TodoContext : DbContext
{
    public TodoContext(DbContextOptions<TodoContext> options): base(options) { }

    public DbSet<TodoItem> TodoItems { get; set; } = null!;
    
    //DbSet<T> represents a table in the database.
    //Here, TodoItems maps to a table that stores TodoItem entities.
    //EF Core lets you query, insert, update, and delete from this DbSet.
    //DbSet<T> is a class in Entity Framework Core.
    //It represents a collection of entities of type T (usually a table in the database).
}
```

## Register Services in `Program.cs`

```csharp
using Microsoft.EntityFrameworkCore;
using TodoApi.Models;

builder.Services.AddControllers();
builder.Services.AddOpenApi();

// If you want to use In-Memory DB
builder.Services.AddDbContext<TodoContext>(opt =>
    opt.UseInMemoryDatabase("TodoList"));

// If you want to use MySQL DB
builder.Services.AddDbContext<TodoContext>(opt =>
    opt.UseMySql(builder.Configuration.GetConnectionString("TodoContext"),
        new MySqlServerVersion(new Version(8, 0, 36))));
```

## Configure Connection String

`appsettings.json`

```json
"ConnectionStrings": {
  "TodoContext": "server=localhost;database=TodoList;user=root;password=yourpassword"
}
```

> ✅ Create the `TodoList` database using MySQL CLI or Workbench.

## Build and Run the App

```bash
dotnet build
dotnet run
```

> dotnet build → 
>  IL (portable, not tied to a CPU).
>  It compiles your C# source code (.cs files) into Intermediate Language (IL) code,
>  not machine code.
>  The IL is stored in a .dll (library) or .exe (application) file inside the bin/ folder.
>  Alongside that, the build process also creates a .deps.json and .runtimeconfig.json that tell >  the runtime what dependencies and framework versions are needed.

> dotnet run → 
> The Just-In-Time (JIT) compiler translates IL into machine code for your CPU.
> That machine code runs directly on your processor.

## Entity Framework Core Setup

### Install the required Entity Framework Core Design package for migrations to work.

```bash
dotnet add package Microsoft.EntityFrameworkCore.Design
```

### Install EF Core CLI Tools

```bash
dotnet tool install --global dotnet-ef
```

### Create Initial Migration

```bash
dotnet ef migrations add InitialCreate
#Build and Run the App if needed
dotnet ef database update
```

> This will generate the `TodoItems` table based on your model.

## Add Controller

> You can manually create controller if you wish

## Scaffold the Controller (Optional)

### Add Required Packages

```bash
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Tools
dotnet tool uninstall -g dotnet-aspnet-codegenerator
dotnet tool install -g dotnet-aspnet-codegenerator
dotnet tool update -g dotnet-aspnet-codegenerator
```

### Generate Controller

```bash
dotnet aspnet-codegenerator controller -name TodoItemsController -async -api -m TodoItem -dc TodoContext -outDir Controllers
```

> This scaffolds a fully functional `TodoItemsController`.

## Enable Swagger UI

Update `Program.cs`:

```csharp
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.UseSwaggerUi(options =>
    {
        options.DocumentPath = "/openapi/v1.json";
    });
}
```

Visit: [http://localhost:5241/swagger](http://localhost:5241/swagger)

## DTO vs Model: Which to Use?

### Why Use DTOs?

- **Security:** Hide sensitive fields.
- **Decoupling:** Separate API contract from DB schema.
- **Validation:** Add custom validation logic.
- **Shaping Data:** Rename or format fields for clients.

### When Is It OK to Use Models Directly?

- For simple apps or internal tools.
- When the model matches the API contract exactly.

> **Best Practice:** Use DTOs for public APIs or production apps.

## Define the DTO

`Models/TodoItemDTO.cs`

```csharp
namespace TodoApi.Models;

public class TodoItemDTO
{
    public long Id { get; set; }
    public string? Name { get; set; }
    public bool IsComplete { get; set; }
}
```

## TodoItemsController Example

`Controllers/TodoItemsController.cs`

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using TodoApi.Models;

namespace TodoApi.Controllers;

[Route("api/[controller]")]
[ApiController]
public class TodoItemsController : ControllerBase
{
    private readonly TodoContext _context;

    public TodoItemsController(TodoContext context)
    {
        _context = context;
    }

    [HttpGet("test")]
    public ActionResult<string> Test() => "This is a static test response.";

    [HttpGet]
    public async Task<ActionResult<IEnumerable<TodoItemDTO>>> GetTodoItems()
    {
        return await _context.TodoItems
             .Select(item => ItemToDTO(item))
             .ToListAsync();
    }

    [HttpGet("{id}")]
    public async Task<ActionResult<TodoItemDTO>> GetTodoItem(long id)
    {
        var item = await _context.TodoItems.FindAsync(id);
        return item == null ? NotFound() : ItemToDTO(item);
    }

    [HttpPut("{id}")]
    public async Task<IActionResult> PutTodoItem(long id, TodoItemDTO dto)
    {
        if (id != dto.Id) return BadRequest();

        var item = await _context.TodoItems.FindAsync(id);
        if (item == null) return NotFound();

        item.Name = dto.Name;
        item.IsComplete = dto.IsComplete;

        try
        {
            await _context.SaveChangesAsync();
        }
        catch (DbUpdateConcurrencyException) when (!TodoItemExists(id))
        {
            return NotFound();
        }

        return NoContent();
    }

    [HttpPost]
    public async Task<ActionResult<TodoItemDTO>> PostTodoItem(TodoItemDTO dto)
    {
        var item = new TodoItem { Name = dto.Name, IsComplete = dto.IsComplete };
        _context.TodoItems.Add(item);
        await _context.SaveChangesAsync();

        return CreatedAtAction(nameof(GetTodoItem), new { id = item.Id }, ItemToDTO(item));
    }

    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteTodoItem(long id)
    {
        var item = await _context.TodoItems.FindAsync(id);
        if (item == null) return NotFound();

        _context.TodoItems.Remove(item);
        await _context.SaveChangesAsync();

        return NoContent();
    }

    private bool TodoItemExists(long id) =>
        _context.TodoItems.Any(e => e.Id == id);

    private static TodoItemDTO ItemToDTO(TodoItem item) => new()
    {
        Id = item.Id,
        Name = item.Name,
        IsComplete = item.IsComplete
    };
}
```