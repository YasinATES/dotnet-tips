# Performance Optimization

Dapper, mikro-ORM'ler arasında performans açısından en iyilerinden biri olarak bilinir. Ancak, büyük ölçekli uygulamalarda veya yüksek trafik altında çalışan sistemlerde, performansı daha da artırmak için çeşitli optimizasyon teknikleri kullanılabilir. Bu bölümde, Dapper ile çalışırken uygulayabileceğiniz performans optimizasyon stratejilerini inceleyeceğiz.

## Buffered vs Unbuffered Queries

Dapper, varsayılan olarak "buffered" (tamponlanmış) sorgular kullanır. Bu, sorgu sonuçlarının tamamının belleğe alındığı ve veritabanı bağlantısının hemen kapatıldığı anlamına gelir. Ancak, büyük veri kümeleri için "unbuffered" (tamponsuz) sorgular kullanmak daha verimli olabilir.

### Buffered Queries (Varsayılan)

```csharp
// Tamponlanmış sorgu (varsayılan)
using (var connection = new SqlConnection(connectionString))
{
    // Tüm sonuçlar belleğe alınır ve bağlantı hemen kapatılır
    var products = connection.Query<Product>("SELECT * FROM Products");
    
    // Bağlantı zaten kapatılmış olabilir
    foreach (var product in products)
    {
        // İşlemler
        Console.WriteLine(product.Name);
    }
}
```

### Unbuffered Queries

```csharp
// Tamponsuz sorgu
using (var connection = new SqlConnection(connectionString))
{
    // Bağlantıyı açık tutmalıyız
    connection.Open();
    
    // buffered: false parametresi ile tamponsuz sorgu
    var products = connection.Query<Product>(
        "SELECT * FROM Products", 
        buffered: false);
    
    // Sonuçlar akış halinde işlenir, tümü belleğe alınmaz
    foreach (var product in products)
    {
        // İşlemler
        Console.WriteLine(product.Name);
    }
    
    // Bağlantı burada kapatılır
}
```

### Ne Zaman Hangisini Kullanmalı?

- **Buffered Queries (Tamponlanmış)**:
  - Küçük ve orta büyüklükteki veri kümeleri için
  - Sonuçların tamamına birden fazla kez erişmeniz gerektiğinde
  - Bağlantı havuzunu verimli kullanmak istediğinizde

- **Unbuffered Queries (Tamponsuz)**:
  - Büyük veri kümeleri için (binlerce veya milyonlarca satır)
  - Bellek kullanımını azaltmak istediğinizde
  - Sonuçları akış halinde işlemeniz gerektiğinde

### Performans Karşılaştırması

```csharp
// Performans karşılaştırması
using (var connection = new SqlConnection(connectionString))
{
    connection.Open();
    
    var stopwatch = Stopwatch.StartNew();
    
    // Tamponlanmış sorgu
    var bufferedProducts = connection.Query<Product>("SELECT * FROM Products").ToList();
    
    stopwatch.Stop();
    Console.WriteLine($"Tamponlanmış sorgu: {stopwatch.ElapsedMilliseconds}ms");
    
    stopwatch.Restart();
    
    // Tamponsuz sorgu
    var unbufferedProducts = connection.Query<Product>("SELECT * FROM Products", buffered: false).ToList();
    
    stopwatch.Stop();
    Console.WriteLine($"Tamponsuz sorgu: {stopwatch.ElapsedMilliseconds}ms");
}
```

## Multi-Result Optimization

Dapper, tek bir veritabanı çağrısında birden fazla sonuç kümesi almanıza olanak tanır. Bu, ağ trafiğini ve veritabanı yükünü azaltarak performansı artırabilir.

### Tek Tek Sorgular

```csharp
// Tek tek sorgular
using (var connection = new SqlConnection(connectionString))
{
    // Her sorgu için ayrı bir veritabanı çağrısı
    var products = connection.Query<Product>("SELECT * FROM Products").ToList();
    var categories = connection.Query<Category>("SELECT * FROM Categories").ToList();
    var suppliers = connection.Query<Supplier>("SELECT * FROM Suppliers").ToList();
    
    // Her sorgu ayrı bir round-trip gerektirir
}
```

### QueryMultiple ile Optimizasyon

```csharp
// QueryMultiple ile optimizasyon
using (var connection = new SqlConnection(connectionString))
{
    // Tek bir SQL komutu ile birden fazla sorgu
    var sql = @"
        SELECT * FROM Products;
        SELECT * FROM Categories;
        SELECT * FROM Suppliers;";
    
    // Tek bir veritabanı çağrısı
    using (var multi = connection.QueryMultiple(sql))
    {
        // Sonuç kümelerini sırayla okuma
        var products = multi.Read<Product>().ToList();
        var categories = multi.Read<Category>().ToList();
        var suppliers = multi.Read<Supplier>().ToList();
        
        // Tek bir round-trip ile üç sonuç kümesi
    }
}
```

### İlişkisel Veriler için Optimizasyon

```csharp
// İlişkisel veriler için optimizasyon
using (var connection = new SqlConnection(connectionString))
{
    // Tek bir SQL komutu ile ilişkili verileri getirme
    var sql = @"
        SELECT * FROM Orders WHERE CustomerId = @CustomerId;
        SELECT * FROM OrderItems WHERE OrderId IN (SELECT Id FROM Orders WHERE CustomerId = @CustomerId);";
    
    var customerId = 1;
    
    using (var multi = connection.QueryMultiple(sql, new { CustomerId = customerId }))
    {
        var orders = multi.Read<Order>().ToList();
        var orderItems = multi.Read<OrderItem>().ToList();
        
        // İlişkileri kurma
        foreach (var order in orders)
        {
            order.Items = orderItems.Where(item => item.OrderId == order.Id).ToList();
        }
        
        // Tek bir veritabanı çağrısı ile ilişkili veriler
    }
}
```

## Statement Caching

Dapper, SQL ifadelerini otomatik olarak önbelleğe alır, ancak bu davranışı daha da optimize edebilirsiniz.

### Otomatik Önbelleğe Alma

```csharp
// Dapper otomatik olarak SQL ifadelerini önbelleğe alır
using (var connection = new SqlConnection(connectionString))
{
    // Bu sorgu ilk çalıştırıldığında ayrıştırılır ve önbelleğe alınır
    var products = connection.Query<Product>("SELECT * FROM Products WHERE CategoryId = @CategoryId", 
        new { CategoryId = 1 });
    
    // Aynı sorgu tekrar çalıştırıldığında, önbellekten alınır
    var moreProducts = connection.Query<Product>("SELECT * FROM Products WHERE CategoryId = @CategoryId", 
        new { CategoryId = 2 });
}
```

### Parametreli Sorgular Kullanma

```csharp
// Parametreli sorgular önbellek kullanımını artırır
using (var connection = new SqlConnection(connectionString))
{
    // YANLIŞ: Her seferinde farklı bir sorgu olarak kabul edilir
    for (int i = 1; i <= 10; i++)
    {
        var products = connection.Query<Product>($"SELECT * FROM Products WHERE CategoryId = {i}");
    }
    
    // DOĞRU: Aynı sorgu tekrar kullanılır, sadece parametreler değişir
    for (int i = 1; i <= 10; i++)
    {
        var products = connection.Query<Product>("SELECT * FROM Products WHERE CategoryId = @CategoryId", 
            new { CategoryId = i });
    }
}
```

### Sorgu Şablonları

```csharp
// Sorgu şablonları tanımlama
public class ProductRepository
{
    private readonly string _connectionString;
    private readonly string _getProductsByCategoryQuery;
    private readonly string _getProductByIdQuery;
    
    public ProductRepository(string connectionString)
    {
        _connectionString = connectionString;
        
        // Sorgu şablonlarını bir kez tanımlama
        _getProductsByCategoryQuery = "SELECT * FROM Products WHERE CategoryId = @CategoryId";
        _getProductByIdQuery = "SELECT * FROM Products WHERE Id = @Id";
    }
    
    public IEnumerable<Product> GetProductsByCategory(int categoryId)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            // Önceden tanımlanmış sorgu şablonunu kullanma
            return connection.Query<Product>(_getProductsByCategoryQuery, new { CategoryId = categoryId });
        }
    }
    
    public Product GetProductById(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            // Önceden tanımlanmış sorgu şablonunu kullanma
            return connection.QuerySingleOrDefault<Product>(_getProductByIdQuery, new { Id = id });
        }
    }
}
```

## Connection Pooling

Veritabanı bağlantıları, oluşturulması ve kapatılması maliyetli işlemlerdir. Bağlantı havuzlama (connection pooling), bu maliyeti azaltmak için kullanılan bir tekniktir.

### Bağlantı Havuzlama Nasıl Çalışır?

.NET'in SqlConnection sınıfı, varsayılan olarak bağlantı havuzlamayı kullanır. Bağlantı dizesinde `Pooling=true` (varsayılan) olduğunda, bağlantılar kapatıldıktan sonra havuza geri döner ve tekrar kullanılabilir hale gelir.

```csharp
// Bağlantı havuzlama varsayılan olarak etkindir
var connectionString = "Server=myserver;Database=mydb;User Id=myuser;Password=mypassword;";

// Bu bağlantı havuzdan alınır
using (var connection = new SqlConnection(connectionString))
{
    connection.Open();
    // İşlemler
}
// Bağlantı kapatıldığında havuza geri döner

// Bu bağlantı da havuzdan alınır (muhtemelen aynı fiziksel bağlantı)
using (var connection = new SqlConnection(connectionString))
{
    connection.Open();
    // İşlemler
}
```

### Bağlantı Havuzlama Optimizasyonu

```csharp
// Bağlantı havuzlama optimizasyonu
public class DatabaseService
{
    private readonly string _connectionString;
    
    public DatabaseService(string connectionString)
    {
        // Aynı bağlantı dizesini kullanmak, havuzlama verimliliğini artırır
        _connectionString = connectionString;
    }
    
    public IEnumerable<Product> GetProducts()
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.Query<Product>("SELECT * FROM Products");
        }
        // Bağlantı havuza geri döner
    }
    
    public Product GetProductById(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.QuerySingleOrDefault<Product>("SELECT * FROM Products WHERE Id = @Id", new { Id = id });
        }
        // Aynı havuzdan bağlantı alınır
    }
}
```

### Bağlantı Havuzlama Ayarları

```csharp
// Bağlantı havuzlama ayarları
var connectionString = "Server=myserver;Database=mydb;User Id=myuser;Password=mypassword;" +
                       "Min Pool Size=5;Max Pool Size=100;Connection Timeout=30;";

// Min Pool Size: Havuzda her zaman hazır bulunacak minimum bağlantı sayısı
// Max Pool Size: Havuzda bulunabilecek maksimum bağlantı sayısı
// Connection Timeout: Bağlantı kurulması için beklenecek maksimum süre (saniye)
```

### Bağlantıları Açık Tutma

```csharp
// Bağlantıları açık tutma (dikkatli kullanın)
public class LongRunningProcess
{
    private readonly SqlConnection _connection;
    
    public LongRunningProcess(string connectionString)
    {
        _connection = new SqlConnection(connectionString);
        _connection.Open();
    }
    
    public void ProcessBatch(IEnumerable<int> ids)
    {
        // Aynı bağlantıyı tekrar tekrar kullanma
        foreach (var id in ids)
        {
            var product = _connection.QuerySingleOrDefault<Product>(
                "SELECT * FROM Products WHERE Id = @Id", new { Id = id });
            
            // İşlemler
        }
    }
    
    public void Dispose()
    {
        // İşlem bittiğinde bağlantıyı kapatma
        _connection.Dispose();
    }
}

// Kullanım
using (var process = new LongRunningProcess(connectionString))
{
    process.ProcessBatch(new[] { 1, 2, 3, 4, 5 });
}
```

## Execution Plan Reuse

SQL Server, sorguları çalıştırmadan önce bir yürütme planı oluşturur. Bu planların yeniden kullanılması, performansı önemli ölçüde artırabilir.

### Parametreli Sorgular

```csharp
// Parametreli sorgular yürütme planı yeniden kullanımını sağlar
using (var connection = new SqlConnection(connectionString))
{
    // YANLIŞ: Her seferinde farklı bir yürütme planı oluşturulur
    var product1 = connection.QuerySingleOrDefault<Product>("SELECT * FROM Products WHERE Id = 1");
    var product2 = connection.QuerySingleOrDefault<Product>("SELECT * FROM Products WHERE Id = 2");
    
    // DOĞRU: Aynı yürütme planı tekrar kullanılır
    var product1 = connection.QuerySingleOrDefault<Product>("SELECT * FROM Products WHERE Id = @Id", new { Id = 1 });
    var product2 = connection.QuerySingleOrDefault<Product>("SELECT * FROM Products WHERE Id = @Id", new { Id = 2 });
}
```

### Stored Procedure Kullanımı

```csharp
// Stored procedure kullanımı
using (var connection = new SqlConnection(connectionString))
{
    // Stored procedure çağrısı
    var product = connection.QuerySingleOrDefault<Product>(
        "GetProductById", 
        new { Id = 1 }, 
        commandType: CommandType.StoredProcedure);
    
    // Stored procedure'ler önceden derlenmiş yürütme planlarına sahiptir
}
```

### Dinamik SQL'den Kaçınma

```csharp
// Dinamik SQL'den kaçınma
public IEnumerable<Product> GetFilteredProducts(string name = null, int? categoryId = null, decimal? minPrice = null)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        // YANLIŞ: Dinamik SQL, yürütme planı yeniden kullanımını engeller
        var sql = "SELECT * FROM Products WHERE 1=1";
        
        if (!string.IsNullOrEmpty(name))
            sql += " AND Name LIKE '%" + name + "%'";
        
        if (categoryId.HasValue)
            sql += " AND CategoryId = " + categoryId.Value;
        
        if (minPrice.HasValue)
            sql += " AND Price >= " + minPrice.Value;
        
        return connection.Query<Product>(sql);
        
        // DOĞRU: Parametreli sorgu, yürütme planı yeniden kullanımını sağlar
        var parameters = new DynamicParameters();
        var conditions = new List<string>();
        
        if (!string.IsNullOrEmpty(name))
        {
            conditions.Add("Name LIKE @Name");
            parameters.Add("@Name", $"%{name}%");
        }
        
        if (categoryId.HasValue)
        {
            conditions.Add("CategoryId = @CategoryId");
            parameters.Add("@CategoryId", categoryId.Value);
        }
        
        if (minPrice.HasValue)
        {
            conditions.Add("Price >= @MinPrice");
            parameters.Add("@MinPrice", minPrice.Value);
        }
        
        var whereClause = conditions.Any() ? " WHERE " + string.Join(" AND ", conditions) : "";
        var sql = "SELECT * FROM Products" + whereClause;
        
        return connection.Query<Product>(sql, parameters);
    }
}
```

## Pratik Performans İpuçları

### 1. Sadece İhtiyaç Duyulan Verileri Getirme

```csharp
// YANLIŞ: Tüm sütunları getirme
var products = connection.Query<Product>("SELECT * FROM Products");

// DOĞRU: Sadece ihtiyaç duyulan sütunları getirme
var products = connection.Query<ProductSummary>("SELECT Id, Name, Price FROM Products");

public class ProductSummary
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

### 2. Sayfalama Kullanma

```csharp
// Sayfalama kullanma
public IEnumerable<Product> GetPagedProducts(int pageNumber, int pageSize)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        // SQL Server için OFFSET-FETCH kullanımı
        var sql = @"
            SELECT Id, Name, Price 
            FROM Products 
            ORDER BY Id 
            OFFSET @Offset ROWS 
            FETCH NEXT @PageSize ROWS ONLY";
        
        var offset = (pageNumber - 1) * pageSize;
        
        return connection.Query<Product>(sql, new { Offset = offset, PageSize = pageSize });
    }
}
```

### 3. Asenkron Metotları Kullanma

```csharp
// Asenkron metotları kullanma
public async Task<IEnumerable<Product>> GetProductsAsync()
{
    using (var connection = new SqlConnection(_connectionString))
    {
        await connection.OpenAsync();
        return await connection.QueryAsync<Product>("SELECT * FROM Products");
    }
}
```

### 4. Bulk İşlemler için SqlBulkCopy Kullanma

```csharp
// Bulk işlemler için SqlBulkCopy kullanma
public void BulkInsertProducts(IEnumerable<Product> products)
{
    // DataTable oluşturma
    var table = new DataTable();
    table.Columns.Add("Name", typeof(string));
    table.Columns.Add("CategoryId", typeof(int));
    table.Columns.Add("Price", typeof(decimal));
    
    foreach (var product in products)
    {
        table.Rows.Add(product.Name, product.CategoryId, product.Price);
    }
    
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        // SqlBulkCopy kullanma
        using (var bulkCopy = new SqlBulkCopy(connection))
        {
            bulkCopy.DestinationTableName = "Products";
            bulkCopy.ColumnMappings.Add("Name", "Name");
            bulkCopy.ColumnMappings.Add("CategoryId", "CategoryId");
            bulkCopy.ColumnMappings.Add("Price", "Price");
            
            bulkCopy.WriteToServer(table);
        }
    }
}
```

### 5. CommandBehavior Kullanma

```csharp
// CommandBehavior kullanma
public IEnumerable<Product> GetProductsOptimized()
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        using (var command = new SqlCommand("SELECT * FROM Products", connection))
        using (var reader = command.ExecuteReader(CommandBehavior.SequentialAccess))
        {
            while (reader.Read())
            {
                yield return new Product
                {
                    Id = reader.GetInt32(0),
                    Name = reader.GetString(1),
                    CategoryId = reader.GetInt32(2),
                    Price = reader.GetDecimal(3)
                };
            }
        }
    }
}
```

## Performans Testi ve İzleme

### Basit Performans Testi

```csharp
// Basit performans testi
public void PerformanceTest()
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        var stopwatch = Stopwatch.StartNew();
        
        // Test edilecek sorgu
        var products = connection.Query<Product>("SELECT * FROM Products").ToList();
        
        stopwatch.Stop();
        
        Console.WriteLine($"Sorgu süresi: {stopwatch.ElapsedMilliseconds}ms");
        Console.WriteLine($"Ürün sayısı: {products.Count}");
    }
}
```

### Sorgu İzleme

```csharp
// Sorgu izleme
public void QueryProfiling()
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        // SQL Server için sorgu istatistiklerini etkinleştirme
        connection.Execute("SET STATISTICS TIME ON; SET STATISTICS IO ON;");
        
        // Test edilecek sorgu
        var products = connection.Query<Product>("SELECT * FROM Products").ToList();
        
        // İstatistikleri kapatma
        connection.Execute("SET STATISTICS TIME OFF; SET STATISTICS IO OFF;");
    }
}
```

## Özet

Bu bölümde, Dapper ile çalışırken uygulayabileceğiniz performans optimizasyon stratejilerini inceledik:

- **Buffered vs Unbuffered Queries**: Veri boyutuna göre uygun sorgu tipini seçme
- **Multi-Result Optimization**: Tek bir veritabanı çağrısında birden fazla sonuç kümesi alma
- **Statement Caching**: SQL ifadelerinin önbelleğe alınmasını optimize etme
- **Connection Pooling**: Veritabanı bağlantılarını verimli kullanma
- **Execution Plan Reuse**: SQL yürütme planlarının yeniden kullanımını sağlama

Bu optimizasyon teknikleri, Dapper uygulamalarınızın performansını önemli ölçüde artırabilir. Ancak, her zaman gerçek dünya senaryolarında test etmek ve uygulamanızın özel gereksinimlerine göre ayarlamak önemlidir. 