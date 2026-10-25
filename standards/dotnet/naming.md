# .NET Naming Conventions

## Classes, Records, and Structs
- PascalCase: `UserService`, `OrderRepository`, `UserDto`
- Use nouns for class names
- One class per file (except small helper classes)
- File name matches class name: `UserService.cs`

## Interfaces
- PascalCase with "I" prefix: `IUserService`, `IOrderRepository`
- Interface file name: `IUserService.cs`
- Use for dependency injection and abstraction

## Methods
- PascalCase: `GetUserById`, `CalculateTotal`, `ValidateInput`
- Use verb-noun pattern
- Async methods end with "Async": `GetUserByIdAsync`
- Boolean methods start with "Is", "Has", "Can": `IsValid()`, `HasPermission()`

## Properties
- PascalCase: `FirstName`, `LastName`, `IsActive`
- Use auto-properties when possible
- Boolean properties use "Is", "Has", "Can": `IsActive`, `HasPermission`

## Variables and Parameters
- camelCase: `firstName`, `lastName`, `userCount`
- Use descriptive names (no single letters except in loops)
- Private fields: `_fieldName` (with underscore prefix)
- Loop variables: `i`, `j`, `k` (short names acceptable)

## Constants
- PascalCase: `MaxRetryCount`, `DefaultTimeout`
- Use `const` for compile-time constants
- Use `readonly` for runtime constants
- Static constants: `public const int MaxItems = 100;`

## Enums
- PascalCase for enum name: `UserStatus`, `OrderType`
- PascalCase for enum values: `Active`, `Inactive`, `Pending`
- Use singular names for flags, plural for non-flags
- Define in separate file or group related enums together

## Namespaces
- PascalCase: `CompanyName.ProjectName.Module`
- Match folder structure: `MyApp.Services` → `Services/` folder
- Avoid deeply nested namespaces (max 3-4 levels)

## Files and Folders
- PascalCase for file names: `UserService.cs`, `OrderController.cs`
- PascalCase for folder names: `Controllers`, `Services`, `Models`
- One class per file (except partial classes)
- Group by feature or by layer (consistent within project)

## Generic Types
- Use single uppercase letter: `T`, `TResult`, `TEntity`
- Descriptive names for multiple generics: `TKey`, `TValue`
- Constraint naming: `where T : class, IEntity`

## Async/Await
- Always suffix async methods with "Async": `GetUserAsync()`
- Use `Task` or `Task<T>` return type
- Never use `async void` except for event handlers
- Use `ConfigureAwait(false)` in library code

## Dependency Injection
- Interface names: `IUserService`
- Implementation names: `UserService`
- Register with clear lifetime: `AddScoped`, `AddTransient`, `AddSingleton`

## Database and API
- Entity class names: PascalCase singular (`User`, `Order`)
- DbContext names: `AppDbContext`, `MyProjectDbContext`
- Table names: PascalCase (EF Core default) or snake_case (configure in DbContext)
- Column names: PascalCase (EF Core default) or snake_case
- JSON properties: camelCase (configure with `JsonNamingPolicy`)

## Events
- PascalCase: `UserCreated`, `OrderPlaced`
- Use past tense for completed actions
- EventHandler suffix for event handler methods

## Exceptions
- End with "Exception": `UserNotFoundException`, `OrderValidationException`
- Use built-in exceptions when appropriate: `ArgumentException`, `InvalidOperationException`
- Create custom exceptions only when adding meaningful context

## Best Practices

### Do
- Follow .NET naming conventions consistently
- Use meaningful, descriptive names
- Avoid abbreviations (except well-known ones: `id`, `url`, `db`, `api`)
- Use plural names for collections: `Users`, `Orders`
- Prefix boolean variables/properties with `is`, `has`, `can`

### Don't
- Don't use Hungarian notation (`strName`, `intCount`)
- Don't use underscores in public members
- Don't use all-caps for constants (use PascalCase)
- Don't abbreviate unless widely understood
- Don't use generic names like `Data`, `Info`, `Manager`
