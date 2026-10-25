# ASP.NET Core Conventions

## Project Structure

### Minimal API Project
```
MyProject/
├── Controllers/
├── Endpoints/
├── Services/
│   ├── Interfaces/
│   └── Implementations/
├── Models/
│   ├── Entities/
│   ├── DTOs/
│   └── ViewModels/
├── Data/
│   ├── DbContext.cs
│   └── Migrations/
├── Middleware/
├── Filters/
├── Validators/
├── Program.cs
└── appsettings.json
```

### Clean Architecture Project
```
MyProject/
├── src/
│   ├── MyProject.Api/           # Presentation layer
│   │   ├── Controllers/
│   │   ├── Endpoints/
│   │   ├── Filters/
│   │   └── Program.cs
│   ├── MyProject.Application/   # Application layer
│   │   ├── Interfaces/
│   │   ├── Services/
│   │   ├── DTOs/
│   │   └── Validators/
│   ├── MyProject.Domain/        # Domain layer
│   │   ├── Entities/
│   │   ├── Enums/
│   │   └── Exceptions/
│   └── MyProject.Infrastructure/ # Infrastructure layer
│       ├── Data/
│       ├── Repositories/
│       └── Services/
└── tests/
```

## Program.cs Setup

### Modern Minimal Hosting
```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

// Add database
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Add repositories
builder.Services.AddScoped<IUserRepository, UserRepository>();
builder.Services.AddScoped<IOrderRepository, OrderRepository>();

// Add services
builder.Services.AddScoped<IUserService, UserService>();
builder.Services.AddScoped<IOrderService, OrderService>();

// Add authentication
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options =>
    {
        options.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuer = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidAudience = builder.Configuration["Jwt:Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(
                Encoding.UTF8.GetBytes(builder.Configuration["Jwt:Key"]!))
        };
    });

// Add authorization
builder.Services.AddAuthorization();

// Add CORS
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
    {
        policy.WithOrigins("http://localhost:4200")
              .AllowAnyHeader()
              .AllowAnyMethod()
              .AllowCredentials();
    });
});

var app = builder.Build();

// Configure middleware pipeline
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseCors("AllowFrontend");
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

## Controllers

### RESTful Controller
```csharp
[ApiController]
[Route("api/v1/[controller]")]
public class UsersController : ControllerBase
{
    private readonly IUserService _userService;
    private readonly ILogger<UsersController> _logger;
    
    public UsersController(IUserService userService, ILogger<UsersController> logger)
    {
        _userService = userService;
        _logger = logger;
    }
    
    [HttpGet]
    public async Task<ActionResult<IEnumerable<UserDto>>> GetUsers(
        [FromQuery] int page = 1,
        [FromQuery] int pageSize = 20)
    {
        var users = await _userService.GetUsersAsync(page, pageSize);
        return Ok(users);
    }
    
    [HttpGet("{id}")]
    public async Task<ActionResult<UserDto>> GetUser(int id)
    {
        var user = await _userService.GetUserByIdAsync(id);
        if (user == null)
        {
            return NotFound();
        }
        return Ok(user);
    }
    
    [HttpPost]
    [Authorize]
    public async Task<ActionResult<UserDto>> CreateUser(CreateUserDto dto)
    {
        var user = await _userService.CreateUserAsync(dto);
        return CreatedAtAction(nameof(GetUser), new { id = user.Id }, user);
    }
    
    [HttpPut("{id}")]
    [Authorize]
    public async Task<IActionResult> UpdateUser(int id, UpdateUserDto dto)
    {
        try
        {
            await _userService.UpdateUserAsync(id, dto);
            return NoContent();
        }
        catch (UserNotFoundException)
        {
            return NotFound();
        }
    }
    
    [HttpDelete("{id}")]
    [Authorize(Roles = "Admin")]
    public async Task<IActionResult> DeleteUser(int id)
    {
        try
        {
            await _userService.DeleteUserAsync(id);
            return NoContent();
        }
        catch (UserNotFoundException)
        {
            return NotFound();
        }
    }
}
```

## Minimal APIs

### Endpoint Definition
```csharp
app.MapGet("/api/v1/users", async (IUserService userService, int page = 1, int pageSize = 20) =>
{
    var users = await userService.GetUsersAsync(page, pageSize);
    return Results.Ok(users);
})
.WithName("GetUsers")
.WithTags("Users")
.Produces<IEnumerable<UserDto>>()
.AllowAnonymous();

app.MapGet("/api/v1/users/{id}", async (int id, IUserService userService) =>
{
    var user = await userService.GetUserByIdAsync(id);
    return user is not null ? Results.Ok(user) : Results.NotFound();
})
.WithName("GetUser")
.WithTags("Users")
.Produces<UserDto>()
.Produces(StatusCodes.Status404NotFound)
.RequireAuthorization();

app.MapPost("/api/v1/users", async (CreateUserDto dto, IUserService userService) =>
{
    var user = await userService.CreateUserAsync(dto);
    return Results.Created($"/api/v1/users/{user.Id}", user);
})
.WithName("CreateUser")
.WithTags("Users")
.Produces<UserDto>(StatusCodes.Status201Created)
.RequireAuthorization();

app.MapPut("/api/v1/users/{id}", async (int id, UpdateUserDto dto, IUserService userService) =>
{
    try
    {
        await userService.UpdateUserAsync(id, dto);
        return Results.NoContent();
    }
    catch (UserNotFoundException)
    {
        return Results.NotFound();
    }
})
.WithName("UpdateUser")
.WithTags("Users")
.Produces(StatusCodes.Status204NoContent)
.Produces(StatusCodes.Status404NotFound)
.RequireAuthorization();

app.MapDelete("/api/v1/users/{id}", async (int id, IUserService userService) =>
{
    try
    {
        await userService.DeleteUserAsync(id);
        return Results.NoContent();
    }
    catch (UserNotFoundException)
    {
        return Results.NotFound();
    }
})
.WithName("DeleteUser")
.WithTags("Users")
.Produces(StatusCodes.Status204NoContent)
.Produces(StatusCodes.Status404NotFound)
.RequireAuthorization(Roles = "Admin");
```

### Grouped Endpoints
```csharp
var usersGroup = app.MapGroup("/api/v1/users")
    .WithTags("Users");

usersGroup.MapGet("/", GetUsers);
usersGroup.MapGet("/{id}", GetUser);
usersGroup.MapPost("/", CreateUser).RequireAuthorization();
usersGroup.MapPut("/{id}", UpdateUser).RequireAuthorization();
usersGroup.MapDelete("/{id}", DeleteUser).RequireAuthorization("Admin");

static async Task<IResult> GetUsers(IUserService userService, int page = 1, int pageSize = 20)
{
    var users = await userService.GetUsersAsync(page, pageSize);
    return Results.Ok(users);
}

static async Task<IResult> GetUser(int id, IUserService userService)
{
    var user = await userService.GetUserByIdAsync(id);
    return user is not null ? Results.Ok(user) : Results.NotFound();
}
```

## DTOs (Data Transfer Objects)

### Request DTOs
```csharp
public record CreateUserDto
{
    [Required]
    [EmailAddress]
    public string Email { get; init; } = string.Empty;
    
    [Required]
    [StringLength(100, MinimumLength = 2)]
    public string Name { get; init; } = string.Empty;
    
    [Required]
    [StringLength(100, MinimumLength = 8)]
    public string Password { get; init; } = string.Empty;
}

public record UpdateUserDto
{
    [StringLength(100, MinimumLength = 2)]
    public string? Name { get; init; }
    
    [StringLength(100, MinimumLength = 8)]
    public string? Password { get; init; }
}
```

### Response DTOs
```csharp
public record UserDto
{
    public int Id { get; init; }
    public string Email { get; init; } = string.Empty;
    public string Name { get; init; } = string.Empty;
    public bool IsActive { get; init; }
    public DateTime CreatedAt { get; init; }
}
```

## Validation

### FluentValidation
```csharp
public class CreateUserDtoValidator : AbstractValidator<CreateUserDto>
{
    public CreateUserDtoValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("Email is required")
            .EmailAddress().WithMessage("Invalid email format");
        
        RuleFor(x => x.Name)
            .NotEmpty().WithMessage("Name is required")
            .Length(2, 100).WithMessage("Name must be between 2 and 100 characters");
        
        RuleFor(x => x.Password)
            .NotEmpty().WithMessage("Password is required")
            .MinimumLength(8).WithMessage("Password must be at least 8 characters");
    }
}
```

### Register Validator
```csharp
builder.Services.AddValidatorsFromAssemblyContaining<CreateUserDtoValidator>();
```

## Middleware

### Custom Middleware
```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;
    
    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        _logger.LogInformation("Request: {Method} {Path}", context.Request.Method, context.Request.Path);
        
        var stopwatch = Stopwatch.StartNew();
        await _next(context);
        stopwatch.Stop();
        
        _logger.LogInformation("Response: {StatusCode} in {ElapsedMs}ms", 
            context.Response.StatusCode, stopwatch.ElapsedMilliseconds);
    }
}

// Register in Program.cs
app.UseMiddleware<RequestLoggingMiddleware>();
```

## Filters

### Exception Filter
```csharp
public class CustomExceptionFilter : IExceptionFilter
{
    private readonly ILogger<CustomExceptionFilter> _logger;
    
    public void OnException(ExceptionContext context)
    {
        _logger.LogError(context.Exception, "Unhandled exception");
        
        var response = new
        {
            success = false,
            error = new
            {
                code = "INTERNAL_ERROR",
                message = "An unexpected error occurred"
            }
        };
        
        context.Result = new ObjectResult(response)
        {
            StatusCode = StatusCodes.Status500InternalServerError
        };
        
        context.ExceptionHandled = true;
    }
}

// Register
builder.Services.AddControllers(options =>
{
    options.Filters.Add<CustomExceptionFilter>();
});
```

## Authentication & Authorization

### JWT Token Generation
```csharp
public string GenerateJwtToken(User user)
{
    var claims = new[]
    {
        new Claim(ClaimTypes.NameIdentifier, user.Id.ToString()),
        new Claim(ClaimTypes.Email, user.Email),
        new Claim(ClaimTypes.Name, user.Name),
        new Claim(ClaimTypes.Role, user.Role)
    };
    
    var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["Jwt:Key"]!));
    var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);
    
    var token = new JwtSecurityToken(
        issuer: _configuration["Jwt:Issuer"],
        audience: _configuration["Jwt:Audience"],
        claims: claims,
        expires: DateTime.Now.AddHours(1),
        signingCredentials: creds);
    
    return new JwtSecurityTokenHandler().WriteToken(token);
}
```

### Authorization Policies
```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminOnly", policy =>
        policy.RequireRole("Admin"));
    
    options.AddPolicy("CanEditUsers", policy =>
        policy.RequireClaim("Permission", "EditUsers"));
});

// Use in controller
[Authorize(Policy = "AdminOnly")]
public async Task<IActionResult> DeleteUser(int id) { }
```

## Best Practices

### Do
- Always use async/await in controllers
- Always return appropriate HTTP status codes
- Always validate input data
- Always use DTOs (don't expose entities directly)
- Always use dependency injection
- Always log errors and important events
- Always handle exceptions globally

### Don't
- Don't put business logic in controllers
- Don't expose sensitive data in responses
- Don't use synchronous methods (`.Result`, `.Wait()`)
- Don't hardcode configuration values
- Don't mix concerns in controllers
- Don't forget to dispose resources
- Don't use `dynamic` in API responses
