# Database-First Yaklaşımı

Database-First yaklaşımı, mevcut bir veritabanından Entity Framework Core model sınıflarını otomatik olarak oluşturmanıza olanak tanıyan bir geliştirme modelidir. Bu yaklaşım, özellikle var olan veritabanlarıyla çalışırken veya veritabanı tasarımının uygulama kodundan önce geldiği durumlarda kullanışlıdır.

## Scaffold-DbContext Komutu

Database-First yaklaşımında, `Scaffold-DbContext` komutu (Package Manager Console) veya `dotnet ef dbcontext scaffold` komutu (.NET CLI) kullanılarak mevcut bir veritabanından model sınıfları oluşturulur.

### Package Manager Console Kullanımı

```powershell
# Basic scaffolding
Scaffold-DbContext "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models

# Scaffolding with additional options
Scaffold-DbContext "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -ContextDir Data -Context MyDbContext -Tables Customer, Order -Force -NoOnConfiguring
```

### .NET CLI Kullanımı

```bash
# Basic scaffolding
dotnet ef dbcontext scaffold "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;" Microsoft.EntityFrameworkCore.SqlServer -o Models

# Scaffolding with additional options
dotnet ef dbcontext scaffold "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;" Microsoft.EntityFrameworkCore.SqlServer -o Models -c MyDbContext -t Customer -t Order --context-dir Data --force --no-onconfiguring
```

### Komut Parametreleri

- **Connection String**: Veritabanı bağlantı dizesi
- **Provider**: Veritabanı sağlayıcısı (örn. Microsoft.EntityFrameworkCore.SqlServer)
- **-OutputDir (-o)**: Model sınıflarının oluşturulacağı dizin
- **-ContextDir**: DbContext sınıfının oluşturulacağı dizin
- **-Context (-c)**: Oluşturulacak DbContext sınıfının adı
- **-Tables (-t)**: Sadece belirli tabloları dahil etmek için
- **-Force**: Var olan dosyaların üzerine yazmak için
- **-NoOnConfiguring**: OnConfiguring metodunu oluşturmamak için

## Reverse Engineering

Reverse engineering, mevcut bir veritabanı şemasını analiz ederek C# sınıflarına dönüştürme sürecidir. EF Core, bu süreci otomatikleştirerek veritabanı tablolarını entity sınıflarına, ilişkileri navigation property'lere ve veritabanı kısıtlamalarını validation attribute'lerine dönüştürür.

### Desteklenen Veritabanı Özellikleri

EF Core reverse engineering, aşağıdaki veritabanı özelliklerini destekler:

- Tablolar ve sütunlar
- Birincil anahtarlar
- Yabancı anahtar ilişkileri
- İndeksler
- Varsayılan değerler
- Bazı check kısıtlamaları
- Stored procedure'ler (sınırlı destek)
- View'lar

### Örnek Çıktı

Aşağıda, bir veritabanından scaffold edilmiş örnek sınıflar gösterilmektedir:

```csharp
// Generated entity class
public partial class Customer
{
    public Customer()
    {
        Orders = new HashSet<Order>();
    }

    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime RegisterDate { get; set; }
    public bool IsActive { get; set; }

    public virtual ICollection<Order> Orders { get; set; }
}

// Generated DbContext class
public partial class MyDbContext : DbContext
{
    public MyDbContext()
    {
    }

    public MyDbContext(DbContextOptions<MyDbContext> options)
        : base(options)
    {
    }

    public virtual DbSet<Customer> Customers { get; set; }
    public virtual DbSet<Order> Orders { get; set; }
    public virtual DbSet<Product> Products { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Customer>(entity =>
        {
            entity.ToTable("Customers");

            entity.HasIndex(e => e.Email)
                .HasName("IX_Customers_Email")
                .IsUnique();

            entity.Property(e => e.Email)
                .IsRequired()
                .HasMaxLength(100);

            entity.Property(e => e.Name)
                .IsRequired()
                .HasMaxLength(50);

            entity.Property(e => e.RegisterDate)
                .HasColumnType("datetime")
                .HasDefaultValueSql("(getdate())");
        });

        // More configuration...

        OnModelCreatingPartial(modelBuilder);
    }

    partial void OnModelCreatingPartial(ModelBuilder modelBuilder);
}
```

## Mevcut Veritabanı Şemasını Modelleme

Database-First yaklaşımında, veritabanı şeması entity model sınıflarına dönüştürülür. Bu süreçte dikkat edilmesi gereken bazı noktalar vardır.

### Tablo ve Sütun Adlandırma Kuralları

EF Core, veritabanındaki tablo ve sütun adlarını C# adlandırma kurallarına uygun hale getirmeye çalışır:

- Tablo adları entity sınıflarına dönüştürülürken tekil hale getirilir (örn. "Customers" -> "Customer")
- Sütun adları property adlarına dönüştürülür (örn. "customer_id" -> "CustomerId")
- Özel karakterler ve boşluklar kaldırılır

### İlişkilerin Modellenmesi

EF Core, veritabanındaki yabancı anahtar kısıtlamalarını analiz ederek entity sınıfları arasındaki ilişkileri modelleyebilir:

```csharp
// One-to-Many relationship from database
public partial class Order
{
    public Order()
    {
        OrderItems = new HashSet<OrderItem>();
    }

    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public int CustomerId { get; set; }

    // Navigation property for the Customer
    public virtual Customer Customer { get; set; }
    
    // Navigation collection for OrderItems
    public virtual ICollection<OrderItem> OrderItems { get; set; }
}
```

### Özel Veri Tipleri

Veritabanındaki özel veri tipleri, uygun C# veri tiplerine dönüştürülür:

- `decimal`, `money` -> `decimal`
- `varchar`, `nvarchar`, `text` -> `string`
- `datetime`, `date` -> `DateTime`
- `bit` -> `bool`
- `varbinary`, `image` -> `byte[]`

### Şema Adları

Veritabanı şema adları, varsayılan olarak entity sınıflarında korunur:

```csharp
// Entity with schema name
modelBuilder.Entity<Product>(entity =>
{
    entity.ToTable("Products", "inventory");
    
    // More configuration...
});
```

## Model Güncelleme Stratejileri

Database-First yaklaşımında, veritabanı şeması değiştiğinde model sınıflarını güncel tutmak için çeşitli stratejiler kullanılabilir.

### Yeniden Scaffold Etme

En basit yaklaşım, model sınıflarını tamamen yeniden oluşturmaktır:

```powershell
# Regenerate all models
Scaffold-DbContext "Connection String" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Force
```

Ancak bu yaklaşım, model sınıflarına yapılan özel değişiklikleri kaybetme riskini taşır.

### Seçici Yeniden Scaffold Etme

Sadece belirli tabloları yeniden scaffold etmek, değişikliklerin etkisini sınırlandırabilir:

```powershell
# Regenerate only specific tables
Scaffold-DbContext "Connection String" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Tables Customer, Order -Force
```

### Partial Sınıflar Kullanma

Scaffold edilen sınıflar `partial` olarak tanımlanır, bu da özel kodunuzu ayrı dosyalarda tutmanıza olanak tanır:

```csharp
// Generated code in Customer.cs
public partial class Customer
{
    // Generated properties and navigation properties
}

// Custom code in Customer.Custom.cs
public partial class Customer
{
    // Custom methods and properties
    public string FullName => $"{FirstName} {LastName}";
    
    public bool IsVip()
    {
        return TotalPurchases > 10000;
    }
}
```

### T4 Şablonları Kullanma

Daha gelişmiş senaryolarda, T4 şablonları kullanarak model oluşturma sürecini özelleştirebilirsiniz:

1. EF Core Power Tools gibi araçlar kullanarak T4 şablonları oluşturun
2. Şablonları ihtiyaçlarınıza göre düzenleyin
3. Şablonları kullanarak modelleri oluşturun

## Stored Procedure ve View Kullanımı

EF Core, veritabanındaki stored procedure'leri ve view'ları da modelleyebilir.

### View'ları Modelleme

View'lar, normal tablolar gibi DbSet olarak modellenebilir:

```csharp
// Scaffold a view
public partial class CustomerOrderSummary
{
    public int CustomerId { get; set; }
    public string CustomerName { get; set; }
    public int OrderCount { get; set; }
    public decimal TotalAmount { get; set; }
}

// In DbContext
public virtual DbSet<CustomerOrderSummary> CustomerOrderSummaries { get; set; }

// Configuration
modelBuilder.Entity<CustomerOrderSummary>(entity =>
{
    entity.HasNoKey();
    entity.ToView("vw_CustomerOrderSummary");
});
```

### Stored Procedure Kullanımı

EF Core, stored procedure'leri doğrudan DbContext üzerinden çağırmanıza olanak tanır:

```csharp
// Call a stored procedure
var customers = await context.Customers
    .FromSqlRaw("EXEC GetCustomersByRegion @Region", new SqlParameter("@Region", "North"))
    .ToListAsync();
```

### Stored Procedure Sonuçlarını Modelleme

Stored procedure sonuçları için özel entity sınıfları oluşturabilirsiniz:

```csharp
// Entity for stored procedure result
public class CustomerSalesReport
{
    public int CustomerId { get; set; }
    public string CustomerName { get; set; }
    public decimal TotalSales { get; set; }
    public int ProductCount { get; set; }
    public DateTime LastOrderDate { get; set; }
}

// Method to call stored procedure
public async Task<List<CustomerSalesReport>> GetCustomerSalesReportAsync(DateTime startDate, DateTime endDate)
{
    var startDateParam = new SqlParameter("@StartDate", startDate);
    var endDateParam = new SqlParameter("@EndDate", endDate);
    
    return await context.Set<CustomerSalesReport>()
        .FromSqlRaw("EXEC GetCustomerSalesReport @StartDate, @EndDate", startDateParam, endDateParam)
        .ToListAsync();
}
```

### Function Mapping

Veritabanı fonksiyonlarını da modelleyebilirsiniz:

```csharp
// Map a database function
modelBuilder.HasDbFunction(
    typeof(MyDbContext).GetMethod(nameof(CalculateDiscount), new[] { typeof(decimal), typeof(int) }))
    .HasName("CalculateOrderDiscount");

// Method to call the function
public decimal CalculateDiscount(decimal orderTotal, int customerLoyaltyPoints)
    => throw new NotImplementedException(); // This will be replaced by the database function

// Usage
var discountedTotal = context.Orders
    .Where(o => o.Id == orderId)
    .Select(o => o.Total - context.CalculateDiscount(o.Total, o.Customer.LoyaltyPoints))
    .FirstOrDefault();
```

## Pratik Örnekler

### Northwind Veritabanını Scaffold Etme

```powershell
# Scaffold Northwind database
Scaffold-DbContext "Server=.;Database=Northwind;Trusted_Connection=True;" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -ContextDir Data -Context NorthwindContext
```

### Scaffold Edilen Modeli Dependency Injection ile Kullanma

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);

// Add DbContext to the container
builder.Services.AddDbContext<NorthwindContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("NorthwindConnection")));

// appsettings.json
{
  "ConnectionStrings": {
    "NorthwindConnection": "Server=.;Database=Northwind;Trusted_Connection=True;"
  }
}
```

### Scaffold Edilen Modeli Özelleştirme

```csharp
// Custom extension methods for the scaffolded model
public static class CustomerExtensions
{
    public static IQueryable<Customer> ActiveCustomers(this IQueryable<Customer> customers)
    {
        return customers.Where(c => c.IsActive);
    }
    
    public static IQueryable<Customer> WithRecentOrders(this IQueryable<Customer> customers, int days)
    {
        var cutoffDate = DateTime.Now.AddDays(-days);
        return customers.Where(c => c.Orders.Any(o => o.OrderDate >= cutoffDate));
    }
}

// Usage
var activeCustomersWithRecentOrders = await context.Customers
    .ActiveCustomers()
    .WithRecentOrders(30)
    .ToListAsync();
```

## Sık Karşılaşılan Sorunlar ve Çözümleri

### Çoklu Şema Desteği

Birden fazla şema içeren veritabanlarında, tüm şemaları dahil etmek için:

```powershell
# Include all schemas
Scaffold-DbContext "Connection String" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Schemas dbo,sales,inventory
```

### Tablo İsimleri ve C# Adlandırma Kuralları

Tablo isimleri C# adlandırma kurallarına uymuyorsa, özelleştirme yapabilirsiniz:

```csharp
// Custom entity name
modelBuilder.Entity<Customer>().ToTable("TBL_CUSTOMERS");
```

### Veritabanı İlişkilerinin Doğru Modellenmemesi

Bazen EF Core, veritabanı ilişkilerini doğru şekilde algılayamayabilir. Bu durumda manuel olarak yapılandırabilirsiniz:

```csharp
// Manually configure relationships
modelBuilder.Entity<Order>()
    .HasOne(o => o.Customer)
    .WithMany(c => c.Orders)
    .HasForeignKey(o => o.CustomerId)
    .OnDelete(DeleteBehavior.Restrict);
```

### Performans İyileştirmeleri

Büyük veritabanlarında scaffold işlemi yavaş olabilir. Sadece ihtiyaç duyulan tabloları dahil ederek performansı artırabilirsiniz:

```powershell
# Include only necessary tables
Scaffold-DbContext "Connection String" Microsoft.EntityFrameworkCore.SqlServer -OutputDir Models -Tables Customer, Order, Product, Category
```

## Özet

Bu bölümde, Entity Framework Core'un Database-First yaklaşımını inceledik:

- **Scaffold-DbContext Komutu**: Mevcut veritabanından model sınıfları oluşturma
- **Reverse Engineering**: Veritabanı şemasını C# sınıflarına dönüştürme
- **Mevcut Veritabanı Şemasını Modelleme**: Tablo, sütun ve ilişkilerin modellenmesi
- **Model Güncelleme Stratejileri**: Veritabanı değişikliklerini modele yansıtma
- **Stored Procedure ve View Kullanımı**: Veritabanı nesnelerini EF Core ile kullanma

Database-First yaklaşımı, özellikle mevcut veritabanlarıyla çalışırken veya veritabanı tasarımının uygulama kodundan önce geldiği durumlarda değerli bir araçtır. Bu yaklaşım, veritabanı şemasını hızlı bir şekilde C# sınıflarına dönüştürerek geliştirme sürecini hızlandırır. 