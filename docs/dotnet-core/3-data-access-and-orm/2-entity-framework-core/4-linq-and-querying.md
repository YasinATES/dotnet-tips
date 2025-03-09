# LINQ ve Sorgulama

Entity Framework Core, veritabanı sorgularını oluşturmak ve çalıştırmak için LINQ (Language Integrated Query) kullanır. LINQ, C# dilinin bir parçası olarak, koleksiyonlar üzerinde sorgulama yapmanıza olanak tanır ve EF Core bu sorguları SQL sorgularına dönüştürür.

## IQueryable vs IEnumerable

EF Core'da sorgulama yaparken, `IQueryable<T>` ve `IEnumerable<T>` arasındaki farkı anlamak önemlidir.

### IQueryable

`IQueryable<T>`, sorgunun henüz çalıştırılmadığı, sadece tanımlandığı bir sorgu ifadesidir. Sorgu, veritabanına gönderilmeden önce değiştirilebilir ve optimize edilebilir.

```csharp
// IQueryable örneği
var customersQuery = context.Customers
    .Where(c => c.City == "İstanbul");
    
// Bu noktada sorgu henüz veritabanına gönderilmedi
    
// Sorguya ek filtreler eklenebilir
if (activeOnly)
{
    customersQuery = customersQuery.Where(c => c.IsActive);
}

// Sorgu, ToList() çağrıldığında veritabanına gönderilir
var customers = await customersQuery.ToListAsync();
```

### IEnumerable

`IEnumerable<T>`, sorgunun zaten çalıştırıldığı ve sonuçların belleğe alındığı bir koleksiyondur. Bu noktadan sonra yapılan filtreleme işlemleri veritabanında değil, bellekte gerçekleştirilir.

```csharp
// IEnumerable örneği
var customers = await context.Customers.ToListAsync();
// Sorgu veritabanına gönderildi ve tüm müşteriler belleğe alındı

// Bu filtreleme işlemi bellekte gerçekleştirilir
var istanbulCustomers = customers.Where(c => c.City == "İstanbul").ToList();
```

### Karşılaştırma

| Özellik | IQueryable | IEnumerable |
|---------|------------|-------------|
| Sorgu Yürütme | Ertelenmiş (deferred) | Anında |
| Sorgu Yeri | Veritabanı | Bellek |
| Performans | Veritabanında filtreleme daha verimli | Tüm veriler belleğe alınır |
| Kullanım | Veritabanı sorguları için | Bellek içi koleksiyonlar için |

### En İyi Uygulamalar

```csharp
// İyi uygulama: Veritabanında filtreleme
var activeCustomers = await context.Customers
    .Where(c => c.IsActive)
    .Where(c => c.City == "İstanbul")
    .ToListAsync();
    
// Kötü uygulama: Tüm verileri çekip bellekte filtreleme
var allCustomers = await context.Customers.ToListAsync();
var filteredCustomers = allCustomers
    .Where(c => c.IsActive)
    .Where(c => c.City == "İstanbul")
    .ToList();
```

## Eager Loading (Include, ThenInclude)

Eager loading, bir entity'yi yüklerken ilişkili entity'leri de aynı sorguda yükleme tekniğidir. Bu, N+1 sorgu problemini önlemeye yardımcı olur.

### Include Kullanımı

```csharp
// Temel Include kullanımı
var customersWithOrders = await context.Customers
    .Include(c => c.Orders)
    .ToListAsync();
    
// Birden fazla ilişkiyi Include etme
var customers = await context.Customers
    .Include(c => c.Orders)
    .Include(c => c.Addresses)
    .ToListAsync();
```

### ThenInclude Kullanımı

`ThenInclude`, iç içe ilişkileri yüklemek için kullanılır.

```csharp
// İç içe ilişkileri yükleme
var customersWithOrdersAndProducts = await context.Customers
    .Include(c => c.Orders)
        .ThenInclude(o => o.OrderItems)
            .ThenInclude(oi => oi.Product)
    .ToListAsync();
    
// Farklı ilişki yollarını yükleme
var customers = await context.Customers
    .Include(c => c.Orders)
        .ThenInclude(o => o.OrderItems)
    .Include(c => c.Orders)
        .ThenInclude(o => o.ShippingAddress)
    .ToListAsync();
```

### Filtrelenmiş Include

```csharp
// Filtrelenmiş Include (EF Core 5.0+)
var customersWithRecentOrders = await context.Customers
    .Include(c => c.Orders.Where(o => o.OrderDate >= DateTime.Now.AddMonths(-1)))
    .ToListAsync();
```

### AutoInclude

```csharp
// DbContext yapılandırmasında AutoInclude
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Order>()
        .Navigation(o => o.Customer)
        .AutoInclude();
}

// Kullanım
var orders = await context.Orders.ToListAsync();
// Customer otomatik olarak yüklenir
```

## Lazy Loading ve Explicit Loading

### Lazy Loading

Lazy loading, ilişkili entity'lerin sadece erişildiğinde yüklenmesi tekniğidir. Bu, gereksiz veri yüklemeyi önler ancak N+1 sorgu problemine neden olabilir.

#### Lazy Loading Yapılandırması

```csharp
// 1. Proxy'leri etkinleştirme
services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(connectionString)
           .UseLazyLoadingProxies());
           
// 2. Navigation property'leri virtual olarak işaretleme
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Lazy loading için virtual
    public virtual ICollection<Order> Orders { get; set; }
}
```

#### Lazy Loading Kullanımı

```csharp
// Lazy loading örneği
var customer = await context.Customers.FirstOrDefaultAsync(c => c.Id == 1);
// Bu noktada sadece müşteri yüklendi

// Orders'a erişildiğinde otomatik olarak yüklenir
var orderCount = customer.Orders.Count;
// Bu noktada ek bir sorgu çalıştırıldı
```

### Explicit Loading

Explicit loading, ilişkili entity'leri ihtiyaç duyulduğunda manuel olarak yükleme tekniğidir.

```csharp
// Explicit loading örneği
var customer = await context.Customers.FirstOrDefaultAsync(c => c.Id == 1);
// Bu noktada sadece müşteri yüklendi

// Orders'ı açıkça yükleme
await context.Entry(customer)
    .Collection(c => c.Orders)
    .LoadAsync();
    
// Filtrelenmiş explicit loading
await context.Entry(customer)
    .Collection(c => c.Orders)
    .Query()
    .Where(o => o.OrderDate >= DateTime.Now.AddMonths(-1))
    .LoadAsync();
    
// Referans ilişkisini yükleme
var order = await context.Orders.FirstOrDefaultAsync(o => o.Id == 1);
await context.Entry(order)
    .Reference(o => o.Customer)
    .LoadAsync();
```

## Projection (Select, SelectMany)

Projection, veritabanından sadece ihtiyaç duyulan alanları seçme tekniğidir. Bu, performansı artırır ve bellek kullanımını azaltır.

### Select Kullanımı

```csharp
// Basit projection
var customerNames = await context.Customers
    .Select(c => c.Name)
    .ToListAsync();
    
// Anonim tip projection
var customerSummaries = await context.Customers
    .Select(c => new
    {
        c.Id,
        c.Name,
        OrderCount = c.Orders.Count,
        TotalSpent = c.Orders.Sum(o => o.Total)
    })
    .ToListAsync();
    
// DTO projection
var customerDtos = await context.Customers
    .Select(c => new CustomerDto
    {
        Id = c.Id,
        FullName = c.Name,
        Email = c.Email,
        OrderCount = c.Orders.Count
    })
    .ToListAsync();
```

### SelectMany Kullanımı

`SelectMany`, bir koleksiyonu düzleştirmek için kullanılır.

```csharp
// Tüm sipariş öğelerini müşterilerden getirme
var allOrderItems = await context.Customers
    .SelectMany(c => c.Orders)
    .SelectMany(o => o.OrderItems)
    .ToListAsync();
    
// Belirli bir şehirdeki müşterilerin tüm siparişlerini getirme
var ordersFromIstanbul = await context.Customers
    .Where(c => c.City == "İstanbul")
    .SelectMany(c => c.Orders, (customer, order) => new
    {
        CustomerName = customer.Name,
        OrderId = order.Id,
        OrderDate = order.OrderDate,
        OrderTotal = order.Total
    })
    .ToListAsync();
```

## Filtreleme, Sıralama ve Gruplama

### Filtreleme (Where)

```csharp
// Temel filtreleme
var activeCustomers = await context.Customers
    .Where(c => c.IsActive)
    .ToListAsync();
    
// Çoklu koşullar
var premiumCustomers = await context.Customers
    .Where(c => c.IsActive && c.CustomerType == CustomerType.Premium)
    .ToListAsync();
    
// String filtreleme
var customersWithEmail = await context.Customers
    .Where(c => c.Email.Contains("@example.com"))
    .ToListAsync();
    
// Tarih filtreleme
var recentOrders = await context.Orders
    .Where(o => o.OrderDate >= DateTime.Now.AddDays(-30))
    .ToListAsync();
    
// İlişkili entity filtreleme
var customersWithPendingOrders = await context.Customers
    .Where(c => c.Orders.Any(o => o.Status == OrderStatus.Pending))
    .ToListAsync();
```

### Sıralama (OrderBy, ThenBy)

```csharp
// Temel sıralama
var customersByName = await context.Customers
    .OrderBy(c => c.Name)
    .ToListAsync();
    
// Azalan sıralama
var customersByRegistrationDateDesc = await context.Customers
    .OrderByDescending(c => c.RegistrationDate)
    .ToListAsync();
    
// Çoklu sıralama
var customers = await context.Customers
    .OrderBy(c => c.City)
    .ThenBy(c => c.Name)
    .ToListAsync();
    
// İlişkili entity'ye göre sıralama
var customersByOrderCount = await context.Customers
    .OrderByDescending(c => c.Orders.Count)
    .ToListAsync();
```

### Gruplama (GroupBy)

```csharp
// Şehre göre müşterileri gruplama
var customersByCity = await context.Customers
    .GroupBy(c => c.City)
    .Select(g => new
    {
        City = g.Key,
        CustomerCount = g.Count(),
        Customers = g.ToList()
    })
    .ToListAsync();
    
// Sipariş sayısına göre müşterileri gruplama
var customersByOrderCount = await context.Customers
    .GroupBy(c => c.Orders.Count)
    .Select(g => new
    {
        OrderCount = g.Key,
        CustomerCount = g.Count(),
        TotalRevenue = g.Sum(c => c.Orders.Sum(o => o.Total))
    })
    .OrderBy(x => x.OrderCount)
    .ToListAsync();
    
// Tarih aralığına göre siparişleri gruplama
var ordersByMonth = await context.Orders
    .GroupBy(o => new { o.OrderDate.Year, o.OrderDate.Month })
    .Select(g => new
    {
        Year = g.Key.Year,
        Month = g.Key.Month,
        OrderCount = g.Count(),
        TotalRevenue = g.Sum(o => o.Total)
    })
    .OrderBy(x => x.Year)
    .ThenBy(x => x.Month)
    .ToListAsync();
```

## İleri Düzey Sorgulama Teknikleri

### Sayfalama (Skip, Take)

```csharp
// Sayfalama örneği
int pageSize = 10;
int pageNumber = 2; // 1-tabanlı sayfa numarası

var pagedCustomers = await context.Customers
    .OrderBy(c => c.Name)
    .Skip((pageNumber - 1) * pageSize)
    .Take(pageSize)
    .ToListAsync();
```

### Aggregate Fonksiyonlar

```csharp
// Count
int customerCount = await context.Customers.CountAsync();
int activeCustomerCount = await context.Customers.CountAsync(c => c.IsActive);

// Sum
decimal totalRevenue = await context.Orders.SumAsync(o => o.Total);

// Average
decimal averageOrderValue = await context.Orders.AverageAsync(o => o.Total);

// Min/Max
decimal lowestOrderValue = await context.Orders.MinAsync(o => o.Total);
decimal highestOrderValue = await context.Orders.MaxAsync(o => o.Total);

// Kombinasyon
var orderStats = await context.Orders
    .GroupBy(o => o.CustomerId)
    .Select(g => new
    {
        CustomerId = g.Key,
        OrderCount = g.Count(),
        TotalSpent = g.Sum(o => o.Total),
        AverageOrderValue = g.Average(o => o.Total),
        MinOrderValue = g.Min(o => o.Total),
        MaxOrderValue = g.Max(o => o.Total)
    })
    .ToListAsync();
```

### Raw SQL Sorguları

```csharp
// FromSqlRaw kullanımı
var customers = await context.Customers
    .FromSqlRaw("SELECT * FROM Customers WHERE City = {0}", "İstanbul")
    .ToListAsync();
    
// Stored procedure çağırma
var customersByRegion = await context.Customers
    .FromSqlRaw("EXEC GetCustomersByRegion @Region", 
        new SqlParameter("@Region", "Marmara"))
    .ToListAsync();
    
// SQL sorgusunu LINQ ile birleştirme
var activeCustomersFromSql = await context.Customers
    .FromSqlRaw("SELECT * FROM Customers")
    .Where(c => c.IsActive)
    .OrderBy(c => c.Name)
    .ToListAsync();
```

### Sorgu Performansı İyileştirme

```csharp
// AsNoTracking ile izlemeyi devre dışı bırakma
var customers = await context.Customers
    .AsNoTracking()
    .ToListAsync();
    
// Belirli özellikleri izleme
var orders = await context.Orders
    .AsTracking()
    .Include(o => o.Customer.AsNoTracking())
    .ToListAsync();
    
// Sorgu filtreleri
var filteredCustomers = await context.Customers
    .TagWith("GetActiveCustomers") // SQL profiler için etiket
    .Where(c => c.IsActive)
    .ToListAsync();
```

## Pratik Örnekler

### Kompleks Bir Sorgu Örneği

```csharp
// Aktif müşterilerin son 30 gündeki siparişlerini getirme
var customerOrderSummaries = await context.Customers
    .Where(c => c.IsActive)
    .Select(c => new CustomerOrderSummary
    {
        CustomerId = c.Id,
        CustomerName = c.Name,
        Email = c.Email,
        RecentOrders = c.Orders
            .Where(o => o.OrderDate >= DateTime.Now.AddDays(-30))
            .Select(o => new OrderSummary
            {
                OrderId = o.Id,
                OrderDate = o.OrderDate,
                Total = o.Total,
                Status = o.Status,
                ItemCount = o.OrderItems.Count
            })
            .OrderByDescending(o => o.OrderDate)
            .ToList(),
        TotalOrderCount = c.Orders.Count,
        RecentOrderCount = c.Orders.Count(o => o.OrderDate >= DateTime.Now.AddDays(-30)),
        TotalSpent = c.Orders.Sum(o => o.Total),
        RecentSpent = c.Orders
            .Where(o => o.OrderDate >= DateTime.Now.AddDays(-30))
            .Sum(o => o.Total)
    })
    .ToListAsync();
```

### Repository Pattern ile LINQ Kullanımı

```csharp
// Repository sınıfı
public class CustomerRepository : ICustomerRepository
{
    private readonly ApplicationDbContext _context;
    
    public CustomerRepository(ApplicationDbContext context)
    {
        _context = context;
    }
    
    public async Task<IEnumerable<Customer>> GetActiveCustomersAsync()
    {
        return await _context.Customers
            .Where(c => c.IsActive)
            .OrderBy(c => c.Name)
            .ToListAsync();
    }
    
    public async Task<Customer> GetCustomerWithOrdersAsync(int customerId)
    {
        return await _context.Customers
            .Include(c => c.Orders)
            .FirstOrDefaultAsync(c => c.Id == customerId);
    }
    
    public async Task<IEnumerable<CustomerDto>> GetCustomerSummariesAsync()
    {
        return await _context.Customers
            .Select(c => new CustomerDto
            {
                Id = c.Id,
                Name = c.Name,
                Email = c.Email,
                OrderCount = c.Orders.Count,
                TotalSpent = c.Orders.Sum(o => o.Total)
            })
            .ToListAsync();
    }
}
```

## Özet

Bu bölümde, Entity Framework Core'da LINQ ve sorgulama tekniklerini inceledik:

- **IQueryable vs IEnumerable**: Sorguların ne zaman ve nerede çalıştırıldığını anlamak
- **Eager Loading**: İlişkili entity'leri aynı sorguda yükleme (Include, ThenInclude)
- **Lazy Loading ve Explicit Loading**: İlişkili entity'leri ihtiyaç duyulduğunda yükleme
- **Projection**: Sadece ihtiyaç duyulan alanları seçme (Select, SelectMany)
- **Filtreleme, Sıralama ve Gruplama**: Verileri filtreleme, sıralama ve gruplama teknikleri

LINQ, Entity Framework Core ile veritabanı sorgularını oluşturmak ve çalıştırmak için güçlü bir araçtır. Doğru kullanıldığında, performanslı ve okunabilir sorgular yazmanıza olanak tanır. 