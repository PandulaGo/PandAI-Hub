# C# Conventions

## Modern C# Features

### File-Scoped Namespaces (C# 10+)
```csharp
namespace MyApp.Services;

public class UserService
{
    // Implementation
}
```

### Primary Constructors (C# 12+)
```csharp
// Classes
public class UserService(IUserRepository repository, ILogger<UserService> logger)
{
    private readonly IUserRepository _repository = repository;
    private readonly ILogger<UserService> _logger = logger;
    
    public async Task<User> GetUserAsync(int id)
    {
        return await _repository.GetByIdAsync(id);
    }
}

// Records
public record UserDto(int Id, string Name, string Email);
```

### Records
```csharp
// Immutable record
public record User(int Id, string Name, string Email);

// Record with validation
public record User
{
    public required int Id { get; init; }
    public required string Name { get; init; }
    public required string Email { get; init; }
}

// Record struct for value types
public record struct Point(int X, int Y);
```

### Pattern Matching
```csharp
// Type pattern
if (obj is User user)
{
    Console.WriteLine(user.Name);
}

// Property pattern
if (user is { Age: >= 18, IsActive: true })
{
    // Adult active user
}

// Switch expression
var status = user.Age switch
{
    < 18 => "Minor",
    < 65 => "Adult",
    _ => "Senior"
};
```

### Nullable Reference Types
```csharp
#nullable enable

public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = string.Empty;  // Non-nullable
    public string? MiddleName { get; set; }  // Nullable
    
    public Address? HomeAddress { get; set; }  // Nullable reference
}
```

## Async/Await Patterns

### Standard Async Method
```csharp
public async Task<User> GetUserByIdAsync(int id)
{
    var user = await _repository.GetByIdAsync(id);
    if (user == null)
    {
        throw new UserNotFoundException(id);
    }
    return user;
}
```

### Async with Cancellation
```csharp
public async Task<User> GetUserAsync(int id, CancellationToken cancellationToken = default)
{
    return await _repository.GetByIdAsync(id, cancellationToken);
}
```

### Concurrent Async Operations
```csharp
public async Task<UserProfile> GetUserProfileAsync(int userId)
{
    var userTask = _userService.GetUserAsync(userId);
    var ordersTask = _orderService.GetOrdersAsync(userId);
    var preferencesTask = _preferenceService.GetPreferencesAsync(userId);
    
    await Task.WhenAll(userTask, ordersTask, preferencesTask);
    
    return new UserProfile
    {
        User = userTask.Result,
        Orders = ordersTask.Result,
        Preferences = preferencesTask.Result
    };
}
```

### Never Use async void
```csharp
// ✅ Correct
public async Task HandleClickAsync() { }

// ❌ Wrong (except for event handlers)
public async void HandleClick() { }
```

## Error Handling

### Custom Exceptions
```csharp
public class UserNotFoundException : Exception
{
    public int UserId { get; }
    
    public UserNotFoundException(int userId)
        : base($"User with ID {userId} was not found")
    {
        UserId = userId;
    }
}

public class ValidationException : Exception
{
    public Dictionary<string, string[]> Errors { get; }
    
    public ValidationException(Dictionary<string, string[]> errors)
        : base("Validation failed")
    {
        Errors = errors;
    }
}
```

### Try-Catch Best Practices
```csharp
// ✅ Catch specific exceptions
try
{
    await _userService.CreateUserAsync(user);
}
catch (ValidationException ex)
{
    _logger.LogWarning(ex, "Validation failed for user creation");
    return BadRequest(ex.Errors);
}
catch (Exception ex)
{
    _logger.LogError(ex, "Unexpected error creating user");
    return StatusCode(500, "An unexpected error occurred");
}

// ❌ Don't catch generic Exception unless necessary
try
{
    // ...
}
catch (Exception)
{
    throw; // Only if you need to log or do something first
}
```

## LINQ Best Practices

### Method Syntax (Preferred)
```csharp
var activeUsers = users
    .Where(u => u.IsActive)
    .OrderBy(u => u.Name)
    .Select(u => new UserDto(u.Id, u.Name, u.Email))
    .ToList();
```

### Query Syntax (When More Readable)
```csharp
var result = from user in users
             where user.IsActive
             orderby user.Name
             select new UserDto(user.Id, user.Name, user.Email);
```

### Avoid Common Pitfalls
```csharp
// ✅ Use .Any() instead of .Count() > 0
if (users.Any(u => u.IsActive)) { }

// ✅ Use .FirstOrDefault() with predicate
var user = users.FirstOrDefault(u => u.Id == id);

// ✅ Use .Select() for projections
var names = users.Select(u => u.Name).ToList();

// ❌ Avoid .ToList() in middle of chain
var result = users.ToList().Where(u => u.IsActive); // Inefficient
```

## Dependency Injection

### Constructor Injection
```csharp
public class UserController(
    IUserService userService,
    ILogger<UserController> logger) : ControllerBase
{
    private readonly IUserService _userService = userService;
    private readonly ILogger<UserController> _logger = logger;
    
    [HttpGet("{id}")]
    public async Task<ActionResult<User>> GetUser(int id)
    {
        var user = await _userService.GetUserByIdAsync(id);
        if (user == null)
        {
            return NotFound();
        }
        return Ok(user);
    }
}
```

### Service Registration
```csharp
// Program.cs
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddTransient<IEmailService, EmailService>();
builder.Services.AddSingleton<ICacheService, CacheService>();
```

## Collection Initializers

### Object Initializers
```csharp
var user = new User
{
    Id = 1,
    Name = "John Doe",
    Email = "john@example.com",
    IsActive = true
};
```

### Collection Initializers
```csharp
var users = new List<User>
{
    new User { Id = 1, Name = "John" },
    new User { Id = 2, Name = "Jane" }
};

var userDict = new Dictionary<int, User>
{
    [1] = new User { Id = 1, Name = "John" },
    [2] = new User { Id = 2, Name = "Jane" }
};
```

## String Interpolation

### Use Interpolation Over Concatenation
```csharp
// ✅ Good
var message = $"User {user.Name} (ID: {user.Id}) created successfully";

// ❌ Bad
var message = "User " + user.Name + " (ID: " + user.Id + ") created successfully";
```

### Multi-line Strings
```csharp
var json = """
    {
        "name": "John Doe",
        "email": "john@example.com",
        "age": 30
    }
    """;
```

## Best Practices

### Do
- Use `var` when type is obvious from right side
- Use expression-bodied members for simple operations
- Use `string.IsNullOrEmpty()` or `string.IsNullOrWhiteSpace()`
- Use `StringBuilder` for string concatenation in loops
- Use `readonly` for fields that don't change after construction
- Use `sealed` for classes that shouldn't be inherited
- Use `record` for immutable data types

### Don't
- Don't use `this.` unless necessary (constructor ambiguity)
- Don't use regions to organize code
- Don't use public fields (use properties)
- Don't use `dynamic` unless absolutely necessary
- Don't suppress nullable warnings with `!` operator
- Don't use `async void` except for event handlers
- Don't mix synchronous and async code (avoid `.Result` or `.Wait()`)
