# Entity Framework Core Conventions

## Repository Pattern

### Interface Definition
```csharp
public interface IRepository<T> where T : class
{
    Task<T?> GetByIdAsync(int id, CancellationToken cancellationToken = default);
    Task<IEnumerable<T>> GetAllAsync(CancellationToken cancellationToken = default);
    Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate, CancellationToken cancellationToken = default);
    Task<T> AddAsync(T entity, CancellationToken cancellationToken = default);
    Task UpdateAsync(T entity, CancellationToken cancellationToken = default);
    Task DeleteAsync(T entity, CancellationToken cancellationToken = default);
    Task<int> CountAsync(Expression<Func<T, bool>>? predicate = null, CancellationToken cancellationToken = default);
    Task<bool> ExistsAsync(Expression<Func<T, bool>> predicate, CancellationToken cancellationToken = default);
}
```

### Generic Repository Implementation
```csharp
public class Repository<T> : IRepository<T> where T : class
{
    protected readonly AppDbContext _context;
    protected readonly DbSet<T> _dbSet;
    
    public Repository(AppDbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    public async Task<T?> GetByIdAsync(int id, CancellationToken cancellationToken = default)
    {
        return await _dbSet.FindAsync(new object[] { id }, cancellationToken);
    }
    
    public async Task<IEnumerable<T>> GetAllAsync(CancellationToken cancellationToken = default)
    {
        return await _dbSet.ToListAsync(cancellationToken);
    }
    
    public async Task<IEnumerable<T>> FindAsync(Expression<Func<T, bool>> predicate, CancellationToken cancellationToken = default)
    {
        return await _dbSet.Where(predicate).ToListAsync(cancellationToken);
    }
    
    public async Task<T> AddAsync(T entity, CancellationToken cancellationToken = default)
    {
        var entry = await _dbSet.AddAsync(entity, cancellationToken);
        await _context.SaveChangesAsync(cancellationToken);
        return entry.Entity;
    }
    
    public async Task UpdateAsync(T entity, CancellationToken cancellationToken = default)
    {
        _dbSet.Update(entity);
        await _context.SaveChangesAsync(cancellationToken);
    }
    
    public async Task DeleteAsync(T entity, CancellationToken cancellationToken = default)
    {
        _dbSet.Remove(entity);
        await _context.SaveChangesAsync(cancellationToken);
    }
    
    public async Task<int> CountAsync(Expression<Func<T, bool>>? predicate = null, CancellationToken cancellationToken = default)
    {
        return predicate == null 
            ? await _dbSet.CountAsync(cancellationToken)
            : await _dbSet.CountAsync(predicate, cancellationToken);
    }
    
    public async Task<bool> ExistsAsync(Expression<Func<T, bool>> predicate, CancellationToken cancellationToken = default)
    {
        return await _dbSet.AnyAsync(predicate, cancellationToken);
    }
}
```

### Specific Repository (When Needed)
```csharp
public interface IUserRepository : IRepository<User>
{
    Task<User?> GetByEmailAsync(string email, CancellationToken cancellationToken = default);
    Task<IEnumerable<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default);
}

public class UserRepository : Repository<User>, IUserRepository
{
    public UserRepository(AppDbContext context) : base(context)
    {
    }
    
    public async Task<User?> GetByEmailAsync(string email, CancellationToken cancellationToken = default)
    {
        return await _dbSet.FirstOrDefaultAsync(u => u.Email == email, cancellationToken);
    }
    
    public async Task<IEnumerable<User>> GetActiveUsersAsync(CancellationToken cancellationToken = default)
    {
        return await _dbSet.Where(u => u.IsActive).ToListAsync(cancellationToken);
    }
}
```

## Unit of Work Pattern

### Interface
```csharp
public interface IUnitOfWork : IDisposable
{
    IUserRepository Users { get; }
    IOrderRepository Orders { get; }
    Task<int> SaveChangesAsync(CancellationToken cancellationToken = default);
}
```

### Implementation
```csharp
public class UnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;
    
    public UnitOfWork(AppDbContext context)
    {
        _context = context;
        Users = new UserRepository(_context);
        Orders = new OrderRepository(_context);
    }
    
    public IUserRepository Users { get; }
    public IOrderRepository Orders { get; }
    
    public async Task<int> SaveChangesAsync(CancellationToken cancellationToken = default)
    {
        return await _context.SaveChangesAsync(cancellationToken);
    }
    
    public void Dispose()
    {
        _context.Dispose();
    }
}
```

### Usage in Service
```csharp
public class OrderService(IUnitOfWork unitOfWork) : IOrderService
{
    private readonly IUnitOfWork _unitOfWork = unitOfWork;
    
    public async Task<Order> CreateOrderAsync(CreateOrderDto dto)
    {
        var user = await _unitOfWork.Users.GetByIdAsync(dto.UserId);
        if (user == null)
        {
            throw new UserNotFoundException(dto.UserId);
        }
        
        var order = new Order
        {
            UserId = dto.UserId,
            Total = dto.Total,
            OrderDate = DateTime.UtcNow,
            Status = OrderStatus.Pending
        };
        
        var createdOrder = await _unitOfWork.Orders.AddAsync(order);
        await _unitOfWork.SaveChangesAsync();
        
        return createdOrder;
    }
}
```

## DbContext Configuration

### Basic DbContext
```csharp
public class AppDbContext : DbContext
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options)
    {
    }
    
    public DbSet<User> Users => Set<User>();
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<Product> Products => Set<Product>();
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        // Apply configurations from separate files
        modelBuilder.ApplyConfiguration(new UserConfiguration());
        modelBuilder.ApplyConfiguration(new OrderConfiguration());
        modelBuilder.ApplyConfiguration(new ProductConfiguration());
    }
}
```

### Entity Configuration (Fluent API)
```csharp
public class UserConfiguration : IEntityTypeConfiguration<User>
{
    public void Configure(EntityTypeBuilder<User> builder)
    {
        builder.ToTable("Users");
        
        builder.HasKey(u => u.Id);
        
        builder.Property(u => u.Id)
            .ValueGeneratedOnAdd();
        
        builder.Property(u => u.Email)
            .IsRequired()
            .HasMaxLength(255);
        
        builder.Property(u => u.Name)
            .IsRequired()
            .HasMaxLength(100);
        
        builder.HasIndex(u => u.Email)
            .IsUnique();
        
        builder.Property(u => u.CreatedAt)
            .HasDefaultValueSql("GETUTCDATE()");
        
        // Relationships
        builder.HasMany(u => u.Orders)
            .WithOne(o => o.User)
            .HasForeignKey(o => o.UserId)
            .OnDelete(DeleteBehavior.Cascade);
    }
}
```

## Entity Design

### Base Entity
```csharp
public abstract class BaseEntity
{
    public int Id { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
    public bool IsDeleted { get; set; } = false;
}
```

### Entity with Relationships
```csharp
public class User : BaseEntity
{
    public string Email { get; set; } = string.Empty;
    public string Name { get; set; } = string.Empty;
    public string PasswordHash { get; set; } = string.Empty;
    public bool IsActive { get; set; } = true;
    
    // Navigation properties
    public ICollection<Order> Orders { get; set; } = new List<Order>();
}

public class Order : BaseEntity
{
    public int UserId { get; set; }
    public decimal Total { get; set; }
    public OrderStatus Status { get; set; }
    public DateTime OrderDate { get; set; }
    
    // Navigation properties
    public User User { get; set; } = null!;
    public ICollection<OrderItem> Items { get; set; } = new List<OrderItem>();
}
```

## Migrations

### Create Migration
```bash
dotnet ef migrations add InitialCreate
dotnet ef migrations add AddOrderStatus
```

### Apply Migration
```bash
dotnet ef database update
```

### Seed Data
```csharp
public class AppDbContext : DbContext
{
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        base.OnModelCreating(modelBuilder);
        
        modelBuilder.Entity<User>().HasData(
            new User { Id = 1, Email = "admin@example.com", Name = "Admin", CreatedAt = DateTime.UtcNow }
        );
    }
}
```

## Query Best Practices

### Use AsNoTracking for Read-Only Queries
```csharp
var users = await _context.Users
    .AsNoTracking()
    .Where(u => u.IsActive)
    .ToListAsync();
```

### Eager Loading
```csharp
var orders = await _context.Orders
    .Include(o => o.User)
    .Include(o => o.Items)
    .ThenInclude(i => i.Product)
    .Where(o => o.Status == OrderStatus.Pending)
    .ToListAsync();
```

### Explicit Loading
```csharp
var user = await _context.Users.FindAsync(userId);
await _context.Entry(user)
    .Collection(u => u.Orders)
    .LoadAsync();
```

### Pagination
```csharp
public async Task<PagedResult<User>> GetUsersAsync(int page, int pageSize)
{
    var query = _context.Users.AsQueryable();
    
    var total = await query.CountAsync();
    var items = await query
        .OrderBy(u => u.Name)
        .Skip((page - 1) * pageSize)
        .Take(pageSize)
        .ToListAsync();
    
    return new PagedResult<User>
    {
        Items = items,
        Total = total,
        Page = page,
        PageSize = pageSize
    };
}
```

## Transaction Management

### Manual Transaction
```csharp
public async Task TransferFundsAsync(int fromUserId, int toUserId, decimal amount)
{
    using var transaction = await _context.Database.BeginTransactionAsync();
    
    try
    {
        var fromUser = await _context.Users.FindAsync(fromUserId);
        var toUser = await _context.Users.FindAsync(toUserId);
        
        fromUser.Balance -= amount;
        toUser.Balance += amount;
        
        await _context.SaveChangesAsync();
        await transaction.CommitAsync();
    }
    catch
    {
        await transaction.RollbackAsync();
        throw;
    }
}
```

## Common Pitfalls

### Don't
- Don't use `_context.Users.ToList()` - loads all users into memory
- Don't forget `AsNoTracking()` for read-only queries
- Don't use navigation properties without `Include()` (causes lazy loading issues)
- Don't call `SaveChanges()` multiple times in a single operation
- Don't use `FirstOrDefault()` without checking for null

### Do
- Always use async methods (`ToListAsync()`, `FirstOrDefaultAsync()`)
- Always use `CancellationToken` for long-running queries
- Always handle null returns from `FirstOrDefault()`
- Always use transactions for multiple related operations
- Always dispose DbContext (let DI handle it)
