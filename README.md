
## NotesAPI

### Preparatory work

`dotnet new webapi -controllers -o NotesAPI`

`cd NotesAPI`

`dotnet add package Microsoft.EntityFrameworkCore.InMemory`

`dotnet dev-certs https --trust`

`dotnet run --launch-profile https`

### Correct ports in launchSettings.json

### Create Models
- Move WeatherForecast.cs in there
- Create new model for Note:
```
public class Note {
    public long Id {get; set;}
    public string? Title {get; set;}
    public string? Content {get; set;}
}
```

- Create new DbContext 
```
using Microsoft.EntityFrameworkCore;

namespace NotesAPI.Models;

public class NoteContext: DbContext {
    public NoteContext(DbContextOptions<NoteContext> options) : base(options){}
    public DbSet<Note> Notes {get; set;} = null!;
}
```

### Update Program.cs for Dependency injection

```
using Microsoft.EntityFrameworkCore;
using NotesAPI.Models;


var builder = WebApplication.CreateBuilder(args);

// Add services to the container.

builder.Services.AddControllers();
builder.Services.AddDbContext<NoteContext> (
    opt => opt.UseInMemoryDatabase("NotesList")
)
```

### Run scaffold commands to create controller

```
dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design

dotnet add package Microsoft.EntityFrameworkCore.Design

dotnet add package Microsoft.EntityFrameworkCore.SqlServer

dotnet add package Microsoft.EntityFrameworkCore.Tools

dotnet tool uninstall -g dotnet-aspnet-codegenerator

dotnet tool install -g dotnet-aspnet-codegenerator

dotnet tool update -g dotnet-aspnet-codegenerator

```


### Create controller
`dotnet aspnet-codegenerator controller -name NotesAPIController -async -api -m Note -dc NoteContext -outDir Controllers`

### Test 
- `dotnet run --launch-profile https`
- Open in browser:
    - https://localhost:7000/swagger
    - https://localhost:7000/api/NotesAPI

- Post a few sample notes
- Try all swagger urls

### Update Post method to get nameof not a hardcoded method

### Test again post method

### Test again with specific note

### Explain controller
- IDs
- Routing
- ActionResult<T>
- JSON serialization

### Explain PUT vs POST
- PUT updates content
- Needs all entity not only changes


# Optional

## Using DTO object for preventing over-posting

```
namespace TodoApi.Models
{
    public class TodoItem
    {
        public long Id { get; set; }
        public string? Name { get; set; }
        public bool IsComplete { get; set; }
        public string? Secret { get; set; }
    }
}
```

```
namespace TodoApi.Models;

public class TodoItemDTO
{
    public long Id { get; set; }
    public string? Name { get; set; }
    public bool IsComplete { get; set; }
}
```

```
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

    // GET: api/TodoItems
    [HttpGet]
    public async Task<ActionResult<IEnumerable<TodoItemDTO>>> GetTodoItems()
    {
        return await _context.TodoItems
            .Select(x => ItemToDTO(x))
            .ToListAsync();
    }

    // GET: api/TodoItems/5
    // <snippet_GetByID>
    [HttpGet("{id}")]
    public async Task<ActionResult<TodoItemDTO>> GetTodoItem(long id)
    {
        var todoItem = await _context.TodoItems.FindAsync(id);

        if (todoItem == null)
        {
            return NotFound();
        }

        return ItemToDTO(todoItem);
    }
    // </snippet_GetByID>

    // PUT: api/TodoItems/5
    // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
    // <snippet_Update>
    [HttpPut("{id}")]
    public async Task<IActionResult> PutTodoItem(long id, TodoItemDTO todoDTO)
    {
        if (id != todoDTO.Id)
        {
            return BadRequest();
        }

        var todoItem = await _context.TodoItems.FindAsync(id);
        if (todoItem == null)
        {
            return NotFound();
        }

        todoItem.Name = todoDTO.Name;
        todoItem.IsComplete = todoDTO.IsComplete;

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
    // </snippet_Update>

    // POST: api/TodoItems
    // To protect from overposting attacks, see https://go.microsoft.com/fwlink/?linkid=2123754
    // <snippet_Create>
    [HttpPost]
    public async Task<ActionResult<TodoItemDTO>> PostTodoItem(TodoItemDTO todoDTO)
    {
        var todoItem = new TodoItem
        {
            IsComplete = todoDTO.IsComplete,
            Name = todoDTO.Name
        };

        _context.TodoItems.Add(todoItem);
        await _context.SaveChangesAsync();

        return CreatedAtAction(
            nameof(GetTodoItem),
            new { id = todoItem.Id },
            ItemToDTO(todoItem));
    }
    // </snippet_Create>

    // DELETE: api/TodoItems/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteTodoItem(long id)
    {
        var todoItem = await _context.TodoItems.FindAsync(id);
        if (todoItem == null)
        {
            return NotFound();
        }

        _context.TodoItems.Remove(todoItem);
        await _context.SaveChangesAsync();

        return NoContent();
    }

    private bool TodoItemExists(long id)
    {
        return _context.TodoItems.Any(e => e.Id == id);
    }

    private static TodoItemDTO ItemToDTO(TodoItem todoItem) =>
       new TodoItemDTO
       {
           Id = todoItem.Id,
           Name = todoItem.Name,
           IsComplete = todoItem.IsComplete
       };
}
```