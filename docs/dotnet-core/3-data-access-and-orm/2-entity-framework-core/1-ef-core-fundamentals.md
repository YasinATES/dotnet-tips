# EF Core Temelleri

Entity Framework Core (EF Core), Microsoft tarafından geliştirilen modern, hafif, genişletilebilir ve çapraz platform destekli bir ORM (Object-Relational Mapping) framework'üdür. Bu bölümde, EF Core'un temel bileşenlerini ve kullanımını inceleyeceğiz.

## DbContext ve DbSet Yapısı

EF Core'da veritabanı etkileşimleri `DbContext` sınıfı üzerinden gerçekleştirilir. `DbContext`, veritabanı oturumunu temsil eder ve veritabanı sorguları yapmak, değişiklikleri izlemek ve veritabanına kaydetmek için kullanılır.

### DbContext Sınıfı

`DbContext` sınıfı, uygulamanız ile veritabanı arasındaki köprüdür. Kendi veritabanı bağlamınızı oluşturmak için `DbContext` sınıfından türetmeniz gerekir.

```csharp
// Basic DbContext implementation
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }
    
    // DbSet properties for your entities
    public DbSet<Customer> Customers { get; set; }
    public DbSet<Order> Orders { get; set; }
    
    // Optional: Configure the model using Fluent API
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure entities using Fluent API
        modelBuilder.Entity<Customer>()
            .HasKey(c => c.Id);
            
        modelBuilder.Entity<Customer>()
            .Property(c => c.Name)
            .IsRequired()
            .HasMaxLength(100);
            
        modelBuilder.Entity<Customer>()
            .HasMany(c => c.Orders)
            .WithOne(o => o.Customer)
            .HasForeignKey(o => o.CustomerId);
    }
}
```

### DbSet Sınıfı

`DbSet<TEntity>` sınıfı, belirli bir entity türü için veritabanı sorguları yapmak ve değişiklikler gerçekleştirmek için kullanılır. Her `DbSet` özelliği, veritabanındaki bir tabloyu temsil eder.

```csharp
// Using DbSet for CRUD operations
public async Task<List<Customer>> GetCustomersAsync()
{
    using var context = new ApplicationDbContext(_options);
    return await context.Customers.ToListAsync();
}

public async Task AddCustomerAsync(Customer customer)
{
    using var context = new ApplicationDbContext(_options);
    context.Customers.Add(customer);
    await context.SaveChangesAsync();
}

public async Task UpdateCustomerAsync(Customer customer)
{
    using var context = new ApplicationDbContext(_options);
    context.Customers.Update(customer);
    await context.SaveChangesAsync();
}

public async Task DeleteCustomerAsync(int customerId)
{
    using var context = new ApplicationDbContext(_options);
    var customer = await context.Customers.FindAsync(customerId);
    if (customer != null)
    {
        context.Customers.Remove(customer);
        await context.SaveChangesAsync();
    }
}
```

## Connection String Yapılandırması

EF Core, veritabanına bağlanmak için connection string'leri kullanır. Connection string'ler genellikle yapılandırma dosyalarında (appsettings.json) saklanır ve uygulama başlatılırken yüklenir.

### appsettings.json Dosyasında Connection String

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyAppDb;User Id=sa;Password=YourPassword;TrustServerCertificate=True;"
  }
}
```

### DbContext Yapılandırması

Connection string'i DbContext'e bağlamak için `DbContextOptionsBuilder` kullanılır. Bu, genellikle uygulama başlatma sırasında yapılır.

```csharp
// Configure DbContext with connection string in Program.cs or Startup.cs
public void ConfigureServices(IServiceCollection services)
{
    // Read connection string from configuration
    var connectionString = Configuration.GetConnectionString("DefaultConnection");
    
    // Configure DbContext with SQL Server
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(connectionString));
}
```

### Farklı Veritabanı Sağlayıcıları

EF Core, çeşitli veritabanı sağlayıcılarını destekler. İşte bazı yaygın sağlayıcılar ve yapılandırmaları:

```csharp
// SQL Server
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString));

// SQLite
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlite("Data Source=myapp.db"));

// PostgreSQL
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseNpgsql("Host=localhost;Database=myapp;Username=postgres;Password=password"));

// MySQL
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseMySql(connectionString, ServerVersion.AutoDetect(connectionString)));

// In-Memory (for testing)
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseInMemoryDatabase("TestDatabase"));
```

## Dependency Injection ile DbContext Kullanımı

Modern .NET uygulamalarında, DbContext genellikle Dependency Injection (DI) kullanılarak yönetilir. Bu, DbContext'in yaşam döngüsünü kontrol etmeyi ve test edilebilirliği artırmayı sağlar.

### Servis Kaydı

DbContext'i DI container'a kaydetmek için `AddDbContext` metodu kullanılır:

```csharp
// In Program.cs (for .NET 6+)
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// In Startup.cs (for older .NET versions)
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>(options =>
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
}
```

### Constructor Injection

DbContext'i bir servise enjekte etmek için constructor injection kullanılır:

```csharp
// Using DbContext with Dependency Injection
public class CustomerService
{
    private readonly ApplicationDbContext _context;
    
    // Constructor injection
    public CustomerService(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<List<Customer>> GetAllCustomersAsync()
    {
        return await _context.Customers.ToListAsync();
    }
    
    public async Task<Customer> GetCustomerByIdAsync(int id)
    {
        return await _context.Customers.FindAsync(id);
    }
    
    public async Task AddCustomerAsync(Customer customer)
    {
        _context.Customers.Add(customer);
        await _context.SaveChangesAsync();
    }
}
```

### DbContext Yaşam Döngüsü

DbContext'in yaşam döngüsü, DI container tarafından yönetilir. Varsayılan olarak, DbContext "scoped" bir servis olarak kaydedilir, yani her HTTP isteği için yeni bir örnek oluşturulur.

```csharp
// Explicitly setting the service lifetime (usually not necessary)
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString),
    contextLifetime: ServiceLifetime.Scoped);
```

## Migrations ve Database Update

EF Core Migrations, veritabanı şemasını kod üzerinden yönetmenizi sağlar. Model sınıflarınızda yapılan değişiklikleri izler ve bu değişiklikleri veritabanına uygulamak için gerekli SQL komutlarını oluşturur.

### Migration Oluşturma

Migration oluşturmak için .NET CLI veya Package Manager Console kullanabilirsiniz:

```bash
# .NET CLI
dotnet ef migrations add InitialCreate

# Package Manager Console
Add-Migration InitialCreate
```

Bu komut, aşağıdaki dosyaları içeren bir Migrations klasörü oluşturur:
- `YYYYMMDDHHMMSS_InitialCreate.cs`: Migration'ın ana dosyası
- `YYYYMMDDHHMMSS_InitialCreate.Designer.cs`: Migration meta verileri
- `ApplicationDbContextModelSnapshot.cs`: Veritabanı modelinin anlık görüntüsü

### Veritabanını Güncelleme

Migration'ları veritabanına uygulamak için:

```bash
# .NET CLI
dotnet ef database update

# Package Manager Console
Update-Database
```

### Migration Kod Örneği

Tipik bir migration dosyası şuna benzer:

```csharp
// Migration class
public partial class InitialCreate : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.CreateTable(
            name: "Customers",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                Name = table.Column<string>(maxLength: 100, nullable: false),
                Email = table.Column<string>(maxLength: 150, nullable: true)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Customers", x => x.Id);
            });
            
        migrationBuilder.CreateTable(
            name: "Orders",
            columns: table => new
            {
                Id = table.Column<int>(nullable: false)
                    .Annotation("SqlServer:Identity", "1, 1"),
                OrderDate = table.Column<DateTime>(nullable: false),
                CustomerId = table.Column<int>(nullable: false)
            },
            constraints: table =>
            {
                table.PrimaryKey("PK_Orders", x => x.Id);
                table.ForeignKey(
                    name: "FK_Orders_Customers_CustomerId",
                    column: x => x.CustomerId,
                    principalTable: "Customers",
                    principalColumn: "Id",
                    onDelete: ReferentialAction.Cascade);
            });
    }
    
    protected override void Down(MigrationBuilder migrationBuilder)
    {
        migrationBuilder.DropTable(name: "Orders");
        migrationBuilder.DropTable(name: "Customers");
    }
}
```

### Seed Data

Migration'lar ayrıca başlangıç verilerini (seed data) eklemek için de kullanılabilir:

```csharp
// Adding seed data in a migration
protected override void Up(MigrationBuilder migrationBuilder)
{
    // Create tables first...
    
    migrationBuilder.InsertData(
        table: "Customers",
        columns: new[] { "Name", "Email" },
        values: new object[] { "John Doe", "john@example.com" });
        
    migrationBuilder.InsertData(
        table: "Customers",
        columns: new[] { "Name", "Email" },
        values: new object[] { "Jane Smith", "jane@example.com" });
}
```

## Logging ve Debugging

EF Core, kapsamlı bir logging sistemi sunar. Bu, EF Core'un arka planda neler yaptığını anlamak ve sorunları gidermek için çok yararlıdır.

### Logging Yapılandırması

EF Core logging'i yapılandırmak için `DbContextOptionsBuilder`'da `LogTo` ve `EnableSensitiveDataLogging` metodlarını kullanabilirsiniz:

```csharp
// Configure logging in DbContext
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder
        .UseSqlServer(_connectionString)
        .LogTo(Console.WriteLine) // Log to console
        .EnableSensitiveDataLogging() // Include parameter values in logs (not recommended for production)
        .EnableDetailedErrors(); // Show detailed error messages
}
```

### ILoggerFactory Kullanımı

Daha gelişmiş logging için, .NET'in yerleşik `ILoggerFactory` sistemini kullanabilirsiniz:

```csharp
// Using ILoggerFactory for logging
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<ApplicationDbContext>((provider, options) =>
    {
        var loggerFactory = provider.GetRequiredService<ILoggerFactory>();
        
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"))
               .UseLoggerFactory(loggerFactory)
               .EnableSensitiveDataLogging(Environment.IsDevelopment());
    });
}
```

### SQL Sorgularını İzleme

Oluşturulan SQL sorgularını izlemek, performans sorunlarını teşhis etmek için çok yararlıdır:

```csharp
// Log generated SQL queries
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString)
           .LogTo(Console.WriteLine, new[] { DbLoggerCategory.Database.Command.Name },
                  LogLevel.Information)
           .EnableSensitiveDataLogging(Environment.IsDevelopment()));
```

### Debugging İpuçları

1. **SQL Profiler**: SQL Server Profiler veya benzeri araçları kullanarak, EF Core tarafından oluşturulan gerçek SQL sorgularını izleyebilirsiniz.

2. **ToQueryString()**: Bir sorgunun oluşturacağı SQL'i görmek için `ToQueryString()` metodunu kullanabilirsiniz:

```csharp
// Get the SQL for a query without executing it
var query = context.Customers
    .Where(c => c.Orders.Count > 5)
    .OrderBy(c => c.Name);
    
string sql = query.ToQueryString();
Console.WriteLine(sql);
```

3. **Database.EnsureCreated() vs Migrations**: Geliştirme sırasında hızlı prototipleme için `Database.EnsureCreated()` kullanabilirsiniz, ancak üretim için her zaman Migrations kullanın:

```csharp
// Quick database creation for development/testing
if (env.IsDevelopment())
{
    context.Database.EnsureCreated();
}
```

## Özet

Bu bölümde, Entity Framework Core'un temel bileşenlerini ve kullanımını inceledik:

- **DbContext ve DbSet**: Veritabanı etkileşimlerinin temel yapı taşları
- **Connection String Yapılandırması**: Farklı veritabanı sağlayıcıları için bağlantı ayarları
- **Dependency Injection**: Modern .NET uygulamalarında DbContext kullanımı
- **Migrations**: Veritabanı şemasını kod üzerinden yönetme
- **Logging ve Debugging**: EF Core uygulamalarında sorun giderme

Entity Framework Core, .NET uygulamalarında veritabanı işlemlerini kolaylaştıran güçlü bir ORM framework'üdür. Doğru yapılandırıldığında, geliştirme sürecini hızlandırır ve bakımı kolaylaştırır. 