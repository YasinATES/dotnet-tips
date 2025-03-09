# Dapper Temelleri

Dapper, .NET için geliştirilmiş hafif, hızlı ve basit bir mikro-ORM (Object-Relational Mapper) kütüphanesidir. "King of Micro ORM" olarak da bilinen Dapper, Entity Framework gibi tam kapsamlı ORM'lere göre daha az özellik sunar, ancak performans açısından çok daha hızlıdır.

## Kurulum ve Yapılandırma

Dapper, NuGet üzerinden kolayca projenize eklenebilir. Herhangi bir yapılandırma gerektirmez ve doğrudan kullanıma hazırdır.

### NuGet ile Kurulum

```csharp
// Package Manager Console ile kurulum
Install-Package Dapper

// .NET CLI ile kurulum
dotnet add package Dapper
```

### Temel Kullanım için Gerekli Namespace'ler

```csharp
// Gerekli namespace'ler
using Dapper;
using System.Data;
using System.Data.SqlClient; // SQL Server için
// veya
using Npgsql; // PostgreSQL için
// veya
using MySql.Data.MySqlClient; // MySQL için
// veya
using Microsoft.Data.Sqlite; // SQLite için
```

### Veritabanı Bağlantısı Oluşturma

```csharp
// SQL Server bağlantısı
string connectionString = "Server=localhost;Database=TestDb;User Id=sa;Password=YourPassword;";
using (var connection = new SqlConnection(connectionString))
{
    connection.Open();
    
    // Dapper sorguları burada...
}

// PostgreSQL bağlantısı
string npgsqlConnectionString = "Host=localhost;Database=TestDb;Username=postgres;Password=YourPassword;";
using (var connection = new NpgsqlConnection(npgsqlConnectionString))
{
    connection.Open();
    
    // Dapper sorguları burada...
}
```

### Bağlantı Yönetimi

```csharp
// Bağlantı havuzu kullanımı (varsayılan olarak etkin)
public class ProductRepository
{
    private readonly string _connectionString;
    
    public ProductRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public IEnumerable<Product> GetAllProducts()
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            // Bağlantı otomatik olarak açılır ve havuza geri döner
            return connection.Query<Product>("SELECT * FROM Products");
        }
    }
}
```

## Basit Sorgu Çalıştırma

Dapper, ADO.NET'in `IDbConnection` arayüzünü genişleterek, veritabanı sorgularını çalıştırmak ve sonuçları C# nesnelerine dönüştürmek için basit metodlar sunar.

### Tekli Kayıt Sorgulama

```csharp
// Tek bir kayıt sorgulama
using (var connection = new SqlConnection(connectionString))
{
    var product = connection.QuerySingleOrDefault<Product>(
        "SELECT * FROM Products WHERE Id = 1");
    
    // Eğer ürün bulunursa
    if (product != null)
    {
        Console.WriteLine($"Ürün: {product.Name}, Fiyat: {product.Price}");
    }
}
```

### Çoklu Kayıt Sorgulama

```csharp
// Birden fazla kayıt sorgulama
using (var connection = new SqlConnection(connectionString))
{
    var products = connection.Query<Product>("SELECT * FROM Products");
    
    // Sonuçları işleme
    foreach (var product in products)
    {
        Console.WriteLine($"Ürün: {product.Name}, Fiyat: {product.Price}");
    }
}
```

### Dinamik Tip Kullanımı

```csharp
// Dinamik tip ile sorgulama
using (var connection = new SqlConnection(connectionString))
{
    var products = connection.Query("SELECT Name, Price FROM Products");
    
    // Dinamik sonuçları işleme
    foreach (var product in products)
    {
        Console.WriteLine($"Ürün: {product.Name}, Fiyat: {product.Price}");
    }
}
```

### Skaler Değer Sorgulama

```csharp
// Tek bir değer sorgulama
using (var connection = new SqlConnection(connectionString))
{
    int productCount = connection.ExecuteScalar<int>("SELECT COUNT(*) FROM Products");
    Console.WriteLine($"Toplam ürün sayısı: {productCount}");
    
    string productName = connection.ExecuteScalar<string>(
        "SELECT Name FROM Products WHERE Id = 1");
    Console.WriteLine($"Ürün adı: {productName}");
}
```

## Parametre Kullanımı

Dapper, SQL enjeksiyon saldırılarını önlemek ve sorgu performansını artırmak için parametreli sorgular destekler.

### Anonim Nesne ile Parametre Kullanımı

```csharp
// Anonim nesne ile parametre kullanımı
using (var connection = new SqlConnection(connectionString))
{
    var product = connection.QuerySingleOrDefault<Product>(
        "SELECT * FROM Products WHERE Id = @Id",
        new { Id = 1 });
    
    // Birden fazla parametre
    var filteredProducts = connection.Query<Product>(
        "SELECT * FROM Products WHERE CategoryId = @CategoryId AND Price > @MinPrice",
        new { CategoryId = 2, MinPrice = 50.0m });
}
```

### DynamicParameters Kullanımı

```csharp
// DynamicParameters ile parametre kullanımı
using (var connection = new SqlConnection(connectionString))
{
    var parameters = new DynamicParameters();
    parameters.Add("@Id", 1);
    parameters.Add("@Name", "Yeni Ürün", DbType.String, ParameterDirection.Input, 100);
    parameters.Add("@CategoryId", dbType: DbType.Int32, direction: ParameterDirection.Output);
    
    var product = connection.QuerySingleOrDefault<Product>(
        "SELECT * FROM Products WHERE Id = @Id OR Name = @Name",
        parameters);
    
    // Output parametresini okuma
    int categoryId = parameters.Get<int>("@CategoryId");
}
```

### Liste Parametresi Kullanımı

```csharp
// Liste parametresi kullanımı (IN operatörü)
using (var connection = new SqlConnection(connectionString))
{
    var productIds = new[] { 1, 2, 3, 4, 5 };
    
    var products = connection.Query<Product>(
        "SELECT * FROM Products WHERE Id IN @Ids",
        new { Ids = productIds });
}
```

## Çoklu Sonuç Döndürme

Dapper, tek bir sorgu ile birden fazla sonuç kümesi döndürmeyi destekler.

### QueryMultiple ile Çoklu Sonuç

```csharp
// Tek sorguda birden fazla sonuç kümesi alma
using (var connection = new SqlConnection(connectionString))
{
    using (var multi = connection.QueryMultiple(
        @"SELECT * FROM Products WHERE CategoryId = @CategoryId;
          SELECT * FROM Categories WHERE Id = @CategoryId",
        new { CategoryId = 1 }))
    {
        // İlk sonuç kümesi: Ürünler
        var products = multi.Read<Product>().ToList();
        
        // İkinci sonuç kümesi: Kategori
        var category = multi.ReadSingleOrDefault<Category>();
        
        Console.WriteLine($"Kategori: {category.Name}");
        Console.WriteLine($"Ürün sayısı: {products.Count}");
    }
}
```

### İlişkili Verileri Sorgulama

```csharp
// İlişkili verileri tek sorguda alma
using (var connection = new SqlConnection(connectionString))
{
    var orderDictionary = new Dictionary<int, Order>();
    
    var orders = connection.Query<Order, OrderDetail, Order>(
        @"SELECT o.*, od.*
          FROM Orders o
          JOIN OrderDetails od ON o.Id = od.OrderId
          WHERE o.CustomerId = @CustomerId",
        (order, orderDetail) =>
        {
            // Aynı siparişi tekrar oluşturmamak için dictionary kullan
            if (!orderDictionary.TryGetValue(order.Id, out var existingOrder))
            {
                existingOrder = order;
                existingOrder.OrderDetails = new List<OrderDetail>();
                orderDictionary.Add(existingOrder.Id, existingOrder);
            }
            
            // Sipariş detayını ekle
            existingOrder.OrderDetails.Add(orderDetail);
            return existingOrder;
        },
        new { CustomerId = 1 },
        splitOn: "Id" // İkinci nesnenin başladığı sütun
    ).Distinct().ToList();
}
```

### Çoklu Tablo Sorgulama

```csharp
// Üç tabloyu birleştirme
using (var connection = new SqlConnection(connectionString))
{
    var orders = connection.Query<Order, Customer, Employee, Order>(
        @"SELECT o.*, c.*, e.*
          FROM Orders o
          JOIN Customers c ON o.CustomerId = c.Id
          JOIN Employees e ON o.EmployeeId = e.Id
          WHERE o.OrderDate >= @StartDate",
        (order, customer, employee) =>
        {
            order.Customer = customer;
            order.Employee = employee;
            return order;
        },
        new { StartDate = new DateTime(2023, 1, 1) },
        splitOn: "Id,Id" // Her nesnenin başladığı sütunlar
    ).ToList();
}
```

## Stored Procedure Çağırma

Dapper, stored procedure'leri çağırmak için basit ve etkili bir yol sunar.

### Basit Stored Procedure Çağrısı

```csharp
// Basit stored procedure çağrısı
using (var connection = new SqlConnection(connectionString))
{
    var products = connection.Query<Product>(
        "GetProductsByCategory",
        new { CategoryId = 1 },
        commandType: CommandType.StoredProcedure);
}
```

### Output Parametreli Stored Procedure

```csharp
// Output parametreli stored procedure
using (var connection = new SqlConnection(connectionString))
{
    var parameters = new DynamicParameters();
    parameters.Add("@ProductName", "Yeni Ürün");
    parameters.Add("@CategoryId", 1);
    parameters.Add("@Price", 99.99m);
    parameters.Add("@ProductId", dbType: DbType.Int32, direction: ParameterDirection.Output);
    
    connection.Execute(
        "CreateProduct",
        parameters,
        commandType: CommandType.StoredProcedure);
    
    // Output parametresini okuma
    int newProductId = parameters.Get<int>("@ProductId");
    Console.WriteLine($"Yeni ürün ID: {newProductId}");
}
```

### Return Value Kullanımı

```csharp
// Return değeri olan stored procedure
using (var connection = new SqlConnection(connectionString))
{
    var parameters = new DynamicParameters();
    parameters.Add("@ProductId", 1);
    parameters.Add("@ReturnValue", dbType: DbType.Int32, direction: ParameterDirection.ReturnValue);
    
    connection.Execute(
        "DeleteProduct",
        parameters,
        commandType: CommandType.StoredProcedure);
    
    // Return değerini okuma
    int returnValue = parameters.Get<int>("@ReturnValue");
    
    if (returnValue == 0)
        Console.WriteLine("Ürün başarıyla silindi.");
    else
        Console.WriteLine("Ürün silinemedi.");
}
```

## Veri Değiştirme İşlemleri

Dapper, veri ekleme, güncelleme ve silme işlemleri için `Execute` metodunu kullanır.

### Veri Ekleme

```csharp
// Tek kayıt ekleme
using (var connection = new SqlConnection(connectionString))
{
    int rowsAffected = connection.Execute(
        "INSERT INTO Products (Name, CategoryId, Price) VALUES (@Name, @CategoryId, @Price)",
        new { Name = "Yeni Ürün", CategoryId = 1, Price = 99.99m });
    
    Console.WriteLine($"Etkilenen satır sayısı: {rowsAffected}");
}
```

### Toplu Veri Ekleme

```csharp
// Çoklu kayıt ekleme
using (var connection = new SqlConnection(connectionString))
{
    var products = new List<Product>
    {
        new Product { Name = "Ürün 1", CategoryId = 1, Price = 19.99m },
        new Product { Name = "Ürün 2", CategoryId = 1, Price = 29.99m },
        new Product { Name = "Ürün 3", CategoryId = 2, Price = 39.99m }
    };
    
    int rowsAffected = connection.Execute(
        "INSERT INTO Products (Name, CategoryId, Price) VALUES (@Name, @CategoryId, @Price)",
        products);
    
    Console.WriteLine($"Eklenen ürün sayısı: {rowsAffected}");
}
```

### Veri Güncelleme

```csharp
// Kayıt güncelleme
using (var connection = new SqlConnection(connectionString))
{
    int rowsAffected = connection.Execute(
        "UPDATE Products SET Price = @Price WHERE Id = @Id",
        new { Id = 1, Price = 149.99m });
    
    Console.WriteLine($"Güncellenen ürün sayısı: {rowsAffected}");
}
```

### Veri Silme

```csharp
// Kayıt silme
using (var connection = new SqlConnection(connectionString))
{
    int rowsAffected = connection.Execute(
        "DELETE FROM Products WHERE Id = @Id",
        new { Id = 1 });
    
    Console.WriteLine($"Silinen ürün sayısı: {rowsAffected}");
}
```

## Pratik Örnekler

### Repository Pattern ile Dapper Kullanımı

```csharp
// Repository sınıfı
public class ProductRepository : IProductRepository
{
    private readonly string _connectionString;
    
    public ProductRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public Product GetById(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.QuerySingleOrDefault<Product>(
                "SELECT * FROM Products WHERE Id = @Id",
                new { Id = id });
        }
    }
    
    public IEnumerable<Product> GetAll()
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.Query<Product>("SELECT * FROM Products");
        }
    }
    
    public IEnumerable<Product> GetByCategoryId(int categoryId)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.Query<Product>(
                "SELECT * FROM Products WHERE CategoryId = @CategoryId",
                new { CategoryId = categoryId });
        }
    }
    
    public int Create(Product product)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            var sql = @"
                INSERT INTO Products (Name, CategoryId, Price)
                VALUES (@Name, @CategoryId, @Price);
                SELECT CAST(SCOPE_IDENTITY() as int)";
            
            return connection.QuerySingle<int>(sql, product);
        }
    }
    
    public bool Update(Product product)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            int rowsAffected = connection.Execute(
                @"UPDATE Products 
                  SET Name = @Name, 
                      CategoryId = @CategoryId, 
                      Price = @Price
                  WHERE Id = @Id",
                product);
            
            return rowsAffected > 0;
        }
    }
    
    public bool Delete(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            int rowsAffected = connection.Execute(
                "DELETE FROM Products WHERE Id = @Id",
                new { Id = id });
            
            return rowsAffected > 0;
        }
    }
}
```

### Dependency Injection ile Kullanım

```csharp
// Program.cs veya Startup.cs içinde
public void ConfigureServices(IServiceCollection services)
{
    // Bağlantı dizesini yapılandırmadan alın
    var connectionString = Configuration.GetConnectionString("DefaultConnection");
    
    // Repository'yi DI container'a kaydedin
    services.AddScoped<IProductRepository>(provider => 
        new ProductRepository(connectionString));
}

// Controller veya servis içinde kullanım
public class ProductController : ControllerBase
{
    private readonly IProductRepository _productRepository;
    
    public ProductController(IProductRepository productRepository)
    {
        _productRepository = productRepository;
    }
    
    [HttpGet("{id}")]
    public ActionResult<Product> GetProduct(int id)
    {
        var product = _productRepository.GetById(id);
        
        if (product == null)
            return NotFound();
            
        return product;
    }
    
    // Diğer action metodları...
}
```

### Transaction Kullanımı

```csharp
// Transaction ile sipariş oluşturma
public bool CreateOrder(Order order)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        using (var transaction = connection.BeginTransaction())
        {
            try
            {
                // Sipariş başlığını ekle
                var orderId = connection.QuerySingle<int>(
                    @"INSERT INTO Orders (CustomerId, OrderDate, TotalAmount)
                      VALUES (@CustomerId, @OrderDate, @TotalAmount);
                      SELECT CAST(SCOPE_IDENTITY() as int)",
                    new { 
                        order.CustomerId, 
                        OrderDate = DateTime.Now, 
                        order.TotalAmount 
                    },
                    transaction);
                
                // Sipariş detaylarını ekle
                foreach (var item in order.OrderItems)
                {
                    item.OrderId = orderId;
                    
                    connection.Execute(
                        @"INSERT INTO OrderItems (OrderId, ProductId, Quantity, UnitPrice)
                          VALUES (@OrderId, @ProductId, @Quantity, @UnitPrice)",
                        item,
                        transaction);
                    
                    // Ürün stoğunu güncelle
                    connection.Execute(
                        @"UPDATE Products
                          SET Stock = Stock - @Quantity
                          WHERE Id = @ProductId",
                        new { item.ProductId, item.Quantity },
                        transaction);
                }
                
                // Transaction'ı commit et
                transaction.Commit();
                return true;
            }
            catch (Exception)
            {
                // Hata durumunda transaction'ı geri al
                transaction.Rollback();
                return false;
            }
        }
    }
}
```

## Özet

Bu bölümde, Dapper'ın temel özelliklerini ve kullanımını inceledik:

- **Kurulum ve Yapılandırma**: NuGet ile kurulum ve temel yapılandırma
- **Basit Sorgu Çalıştırma**: Tekli ve çoklu kayıt sorgulama
- **Parametre Kullanımı**: Anonim nesneler ve DynamicParameters ile parametre kullanımı
- **Çoklu Sonuç Döndürme**: QueryMultiple ile birden fazla sonuç kümesi alma
- **Stored Procedure Çağırma**: Stored procedure'leri parametrelerle çağırma

Dapper, performans odaklı ve SQL kontrolü gerektiren uygulamalar için mükemmel bir seçimdir. Entity Framework Core gibi tam kapsamlı ORM'lere göre daha az soyutlama sunar, ancak bu sayede daha yüksek performans ve daha fazla SQL kontrolü sağlar. 