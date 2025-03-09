# Advanced Dapper

Dapper'ın temel özelliklerinin ötesinde, daha karmaşık veritabanı işlemleri için kullanabileceğiniz ileri düzey özellikleri bulunmaktadır. Bu bölümde, Dapper'ın multi-mapping, dinamik parametreler, toplu işlemler, transaction yönetimi ve asenkron operasyonlar gibi ileri düzey özelliklerini inceleyeceğiz.

## Multi-Mapping

Multi-mapping, tek bir SQL sorgusu ile birden fazla ilişkili nesneyi eşleştirmenize olanak tanır. Bu, özellikle ilişkisel veritabanlarında çalışırken, JOIN işlemleri sonucunda elde edilen verileri nesne grafiğine dönüştürmek için kullanışlıdır.

### Temel Multi-Mapping

```csharp
// Temel multi-mapping örneği
using (var connection = new SqlConnection(connectionString))
{
    var sql = @"
        SELECT o.Id, o.OrderDate, o.CustomerId, 
               c.Id, c.Name, c.Email
        FROM Orders o
        INNER JOIN Customers c ON o.CustomerId = c.Id
        WHERE o.Id = @OrderId";
    
    var order = connection.Query<Order, Customer, Order>(
        sql,
        (order, customer) => {
            // Müşteri nesnesini sipariş nesnesine atama
            order.Customer = customer;
            return order;
        },
        new { OrderId = 1 },
        splitOn: "Id" // İkinci nesnenin başladığı sütun
    ).FirstOrDefault();
    
    Console.WriteLine($"Sipariş No: {order.Id}, Müşteri: {order.Customer.Name}");
}
```

### Çoklu Tablo İlişkileri

```csharp
// Üç tabloyu birleştirme
using (var connection = new SqlConnection(connectionString))
{
    var sql = @"
        SELECT o.Id, o.OrderDate, o.TotalAmount,
               c.Id, c.Name, c.Email,
               p.Id, p.Name, p.Price
        FROM Orders o
        INNER JOIN Customers c ON o.CustomerId = c.Id
        INNER JOIN OrderItems oi ON o.Id = oi.OrderId
        INNER JOIN Products p ON oi.ProductId = p.Id
        WHERE o.Id = @OrderId";
    
    var orderDictionary = new Dictionary<int, Order>();
    
    var orders = connection.Query<Order, Customer, Product, Order>(
        sql,
        (order, customer, product) => {
            // Dictionary'de sipariş var mı kontrol et
            if (!orderDictionary.TryGetValue(order.Id, out var existingOrder))
            {
                // Yeni sipariş oluştur
                existingOrder = order;
                existingOrder.Customer = customer;
                existingOrder.Products = new List<Product>();
                orderDictionary.Add(existingOrder.Id, existingOrder);
            }
            
            // Ürünü siparişe ekle
            existingOrder.Products.Add(product);
            return existingOrder;
        },
        new { OrderId = 1 },
        splitOn: "Id,Id" // Her nesnenin başladığı sütunlar
    ).Distinct().FirstOrDefault();
}
```

### Parent-Child İlişkileri

```csharp
// Parent-child ilişkisi örneği
using (var connection = new SqlConnection(connectionString))
{
    var sql = @"
        SELECT c.Id, c.Name, c.Email,
               a.Id, a.Street, a.City, a.Country, a.CustomerId
        FROM Customers c
        LEFT JOIN Addresses a ON c.Id = a.CustomerId
        WHERE c.Id = @CustomerId";
    
    var customerDictionary = new Dictionary<int, Customer>();
    
    var customers = connection.Query<Customer, Address, Customer>(
        sql,
        (customer, address) => {
            // Dictionary'de müşteri var mı kontrol et
            if (!customerDictionary.TryGetValue(customer.Id, out var existingCustomer))
            {
                // Yeni müşteri oluştur
                existingCustomer = customer;
                existingCustomer.Addresses = new List<Address>();
                customerDictionary.Add(existingCustomer.Id, existingCustomer);
            }
            
            // Adresi müşteriye ekle (null değilse)
            if (address != null)
            {
                existingCustomer.Addresses.Add(address);
            }
            
            return existingCustomer;
        },
        new { CustomerId = 1 },
        splitOn: "Id"
    ).Distinct().FirstOrDefault();
}
```

### Çoklu Sonuç Kümeleri

```csharp
// Çoklu sonuç kümeleri örneği
using (var connection = new SqlConnection(connectionString))
{
    var sql = @"
        SELECT * FROM Customers WHERE Id = @CustomerId;
        SELECT * FROM Orders WHERE CustomerId = @CustomerId;
        SELECT p.* 
        FROM Products p
        INNER JOIN OrderItems oi ON p.Id = oi.ProductId
        INNER JOIN Orders o ON oi.OrderId = o.Id
        WHERE o.CustomerId = @CustomerId";
    
    using (var multi = connection.QueryMultiple(sql, new { CustomerId = 1 }))
    {
        // İlk sonuç kümesi: Müşteri
        var customer = multi.ReadFirstOrDefault<Customer>();
        
        // İkinci sonuç kümesi: Siparişler
        var orders = multi.Read<Order>().ToList();
        
        // Üçüncü sonuç kümesi: Ürünler
        var products = multi.Read<Product>().ToList();
        
        // Nesneleri ilişkilendir
        customer.Orders = orders;
        
        Console.WriteLine($"Müşteri: {customer.Name}");
        Console.WriteLine($"Sipariş Sayısı: {customer.Orders.Count}");
        Console.WriteLine($"Satın Alınan Ürün Sayısı: {products.Count}");
    }
}
```

## Dynamic Parameters

Dynamic parameters, SQL sorgularında kullanılacak parametreleri dinamik olarak oluşturmanıza ve yönetmenize olanak tanır. Bu, özellikle stored procedure'ler ve karmaşık sorgular için kullanışlıdır.

### Temel DynamicParameters Kullanımı

```csharp
// Temel DynamicParameters kullanımı
using (var connection = new SqlConnection(connectionString))
{
    var parameters = new DynamicParameters();
    
    // Parametre ekleme
    parameters.Add("@CustomerId", 1);
    parameters.Add("@StartDate", new DateTime(2023, 1, 1));
    parameters.Add("@EndDate", DateTime.Now);
    
    // Sorgu çalıştırma
    var orders = connection.Query<Order>(
        @"SELECT * FROM Orders 
          WHERE CustomerId = @CustomerId 
          AND OrderDate BETWEEN @StartDate AND @EndDate",
        parameters);
}
```

### Output Parametreleri

```csharp
// Output parametreleri örneği
using (var connection = new SqlConnection(connectionString))
{
    var parameters = new DynamicParameters();
    
    // Input parametreleri
    parameters.Add("@ProductName", "Yeni Ürün");
    parameters.Add("@CategoryId", 1);
    parameters.Add("@Price", 99.99m);
    
    // Output parametresi
    parameters.Add("@ProductId", dbType: DbType.Int32, direction: ParameterDirection.Output);
    
    // Stored procedure çağırma
    connection.Execute(
        "CreateProduct",
        parameters,
        commandType: CommandType.StoredProcedure);
    
    // Output parametresini okuma
    int newProductId = parameters.Get<int>("@ProductId");
    Console.WriteLine($"Yeni ürün ID: {newProductId}");
}
```

### Stored Procedure Return Value

```csharp
// Return değeri örneği
using (var connection = new SqlConnection(connectionString))
{
    var parameters = new DynamicParameters();
    
    // Input parametresi
    parameters.Add("@OrderId", 1);
    
    // Return değeri için parametre
    parameters.Add("@ReturnValue", dbType: DbType.Int32, direction: ParameterDirection.ReturnValue);
    
    // Stored procedure çağırma
    connection.Execute(
        "CancelOrder",
        parameters,
        commandType: CommandType.StoredProcedure);
    
    // Return değerini okuma
    int returnValue = parameters.Get<int>("@ReturnValue");
    
    // Return değerine göre işlem yapma
    if (returnValue == 0)
        Console.WriteLine("Sipariş başarıyla iptal edildi.");
    else if (returnValue == 1)
        Console.WriteLine("Sipariş zaten gönderilmiş, iptal edilemez.");
    else
        Console.WriteLine("Sipariş iptal edilemedi.");
}
```

### Dinamik Sorgu Oluşturma

```csharp
// Dinamik sorgu oluşturma
public IEnumerable<Product> SearchProducts(string name = null, int? categoryId = null, decimal? minPrice = null, decimal? maxPrice = null)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        var parameters = new DynamicParameters();
        var conditions = new List<string>();
        var sql = "SELECT * FROM Products WHERE 1=1";
        
        // Dinamik koşullar ekleme
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
        
        if (maxPrice.HasValue)
        {
            conditions.Add("Price <= @MaxPrice");
            parameters.Add("@MaxPrice", maxPrice.Value);
        }
        
        // Koşulları SQL sorgusuna ekleme
        if (conditions.Any())
        {
            sql += " AND " + string.Join(" AND ", conditions);
        }
        
        // Sorguyu çalıştırma
        return connection.Query<Product>(sql, parameters);
    }
}
```

## Bulk Operations

Dapper, toplu veri işlemleri için optimize edilmiş yöntemler sunar. Bu, büyük miktarda veriyi veritabanına eklemek, güncellemek veya silmek için kullanışlıdır.

### Toplu Veri Ekleme

```csharp
// Toplu veri ekleme
public void BulkInsertProducts(IEnumerable<Product> products)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        // Tek bir SQL sorgusu ile çoklu kayıt ekleme
        var rowsAffected = connection.Execute(
            @"INSERT INTO Products (Name, CategoryId, Price, Stock) 
              VALUES (@Name, @CategoryId, @Price, @Stock)",
            products);
        
        Console.WriteLine($"Eklenen ürün sayısı: {rowsAffected}");
    }
}
```

### Toplu Veri Güncelleme

```csharp
// Toplu veri güncelleme
public void BulkUpdateProductPrices(IEnumerable<ProductPriceUpdate> updates)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        // Tek bir SQL sorgusu ile çoklu kayıt güncelleme
        var rowsAffected = connection.Execute(
            "UPDATE Products SET Price = @NewPrice WHERE Id = @ProductId",
            updates);
        
        Console.WriteLine($"Güncellenen ürün sayısı: {rowsAffected}");
    }
}

// Güncelleme için yardımcı sınıf
public class ProductPriceUpdate
{
    public int ProductId { get; set; }
    public decimal NewPrice { get; set; }
}
```

### Toplu Veri Silme

```csharp
// Toplu veri silme
public void BulkDeleteProducts(IEnumerable<int> productIds)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        // IN operatörü ile çoklu kayıt silme
        var rowsAffected = connection.Execute(
            "DELETE FROM Products WHERE Id IN @Ids",
            new { Ids = productIds });
        
        Console.WriteLine($"Silinen ürün sayısı: {rowsAffected}");
    }
}
```

### SqlBulkCopy ile Entegrasyon

```csharp
// SqlBulkCopy ile toplu veri ekleme (sadece SQL Server)
public void BulkInsertWithSqlBulkCopy(DataTable productsTable)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        using (var bulkCopy = new SqlBulkCopy(connection))
        {
            // Hedef tablo adını ayarlama
            bulkCopy.DestinationTableName = "Products";
            
            // Sütun eşleştirmelerini ayarlama
            bulkCopy.ColumnMappings.Add("Name", "Name");
            bulkCopy.ColumnMappings.Add("CategoryId", "CategoryId");
            bulkCopy.ColumnMappings.Add("Price", "Price");
            bulkCopy.ColumnMappings.Add("Stock", "Stock");
            
            // Toplu veri yazma
            bulkCopy.WriteToServer(productsTable);
        }
    }
}

// DataTable oluşturma yardımcı metodu
private DataTable CreateProductsDataTable(IEnumerable<Product> products)
{
    var table = new DataTable();
    
    // Sütunları tanımlama
    table.Columns.Add("Name", typeof(string));
    table.Columns.Add("CategoryId", typeof(int));
    table.Columns.Add("Price", typeof(decimal));
    table.Columns.Add("Stock", typeof(int));
    
    // Satırları ekleme
    foreach (var product in products)
    {
        table.Rows.Add(product.Name, product.CategoryId, product.Price, product.Stock);
    }
    
    return table;
}
```

## Transaction Yönetimi

Dapper, ADO.NET transaction'larını kullanarak veritabanı işlemlerinin atomik olarak gerçekleştirilmesini sağlar. Bu, birden fazla veritabanı işleminin ya hep birlikte başarılı olmasını ya da hiçbirinin gerçekleşmemesini garanti eder.

### Temel Transaction Kullanımı

```csharp
// Temel transaction kullanımı
using (var connection = new SqlConnection(_connectionString))
{
    connection.Open();
    
    using (var transaction = connection.BeginTransaction())
    {
        try
        {
            // İlk işlem
            connection.Execute(
                "INSERT INTO Categories (Name) VALUES (@Name)",
                new { Name = "Yeni Kategori" },
                transaction);
            
            // İkinci işlem
            connection.Execute(
                "INSERT INTO Products (Name, CategoryId, Price) VALUES (@Name, @CategoryId, @Price)",
                new { Name = "Yeni Ürün", CategoryId = 1, Price = 99.99m },
                transaction);
            
            // Transaction'ı commit et
            transaction.Commit();
            Console.WriteLine("İşlemler başarıyla tamamlandı.");
        }
        catch (Exception ex)
        {
            // Hata durumunda transaction'ı geri al
            transaction.Rollback();
            Console.WriteLine($"Hata: {ex.Message}");
        }
    }
}
```

### Savepoint Kullanımı

```csharp
// Savepoint kullanımı
using (var connection = new SqlConnection(_connectionString))
{
    connection.Open();
    
    using (var transaction = connection.BeginTransaction())
    {
        try
        {
            // İlk işlem
            connection.Execute(
                "INSERT INTO Categories (Name) VALUES (@Name)",
                new { Name = "Yeni Kategori" },
                transaction);
            
            // Savepoint oluştur
            transaction.Save("AfterCategoryInsert");
            
            try
            {
                // İkinci işlem
                connection.Execute(
                    "INSERT INTO Products (Name, CategoryId, Price) VALUES (@Name, @CategoryId, @Price)",
                    new { Name = "Yeni Ürün", CategoryId = 999, Price = 99.99m }, // Olmayan bir CategoryId
                    transaction);
            }
            catch
            {
                // Sadece ikinci işlemi geri al
                transaction.Rollback("AfterCategoryInsert");
                
                // Alternatif ürün ekle
                connection.Execute(
                    "INSERT INTO Products (Name, CategoryId, Price) VALUES (@Name, @CategoryId, @Price)",
                    new { Name = "Alternatif Ürün", CategoryId = 1, Price = 79.99m },
                    transaction);
            }
            
            // Transaction'ı commit et
            transaction.Commit();
            Console.WriteLine("İşlemler başarıyla tamamlandı.");
        }
        catch (Exception ex)
        {
            // Hata durumunda transaction'ı geri al
            transaction.Rollback();
            Console.WriteLine($"Hata: {ex.Message}");
        }
    }
}
```

### Distributed Transaction

```csharp
// Distributed transaction örneği
using (var scope = new TransactionScope())
{
    try
    {
        // SQL Server veritabanı işlemi
        using (var sqlConnection = new SqlConnection(_sqlConnectionString))
        {
            sqlConnection.Execute(
                "INSERT INTO Logs (Message, Timestamp) VALUES (@Message, @Timestamp)",
                new { Message = "İşlem başladı", Timestamp = DateTime.Now });
        }
        
        // PostgreSQL veritabanı işlemi
        using (var npgsqlConnection = new NpgsqlConnection(_npgsqlConnectionString))
        {
            npgsqlConnection.Execute(
                "INSERT INTO audit_logs (message, created_at) VALUES (@Message, @CreatedAt)",
                new { Message = "İşlem başladı", CreatedAt = DateTime.Now });
        }
        
        // Transaction'ı tamamla
        scope.Complete();
        Console.WriteLine("Distributed transaction başarıyla tamamlandı.");
    }
    catch (Exception ex)
    {
        // Hata durumunda transaction otomatik olarak geri alınır
        Console.WriteLine($"Hata: {ex.Message}");
    }
}
```

## Async Operasyonlar

Dapper, tüm temel işlemler için asenkron versiyonlar sunar. Bu, özellikle web uygulamaları ve yüksek eşzamanlılık gerektiren senaryolarda performansı artırır.

### Temel Async Sorgular

```csharp
// Temel async sorgu
public async Task<IEnumerable<Product>> GetProductsAsync()
{
    using (var connection = new SqlConnection(_connectionString))
    {
        // Bağlantıyı asenkron olarak aç
        await connection.OpenAsync();
        
        // Sorguyu asenkron olarak çalıştır
        return await connection.QueryAsync<Product>("SELECT * FROM Products");
    }
}

// Tek kayıt getirme
public async Task<Product> GetProductByIdAsync(int id)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        return await connection.QuerySingleOrDefaultAsync<Product>(
            "SELECT * FROM Products WHERE Id = @Id",
            new { Id = id });
    }
}
```

### Async Multi-Mapping

```csharp
// Async multi-mapping
public async Task<Order> GetOrderWithDetailsAsync(int orderId)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        var sql = @"
            SELECT o.*, c.*, oi.*, p.*
            FROM Orders o
            INNER JOIN Customers c ON o.CustomerId = c.Id
            INNER JOIN OrderItems oi ON o.Id = oi.OrderId
            INNER JOIN Products p ON oi.ProductId = p.Id
            WHERE o.Id = @OrderId";
        
        var orderDictionary = new Dictionary<int, Order>();
        var orderItemDictionary = new Dictionary<int, OrderItem>();
        
        var orders = await connection.QueryAsync<Order, Customer, OrderItem, Product, Order>(
            sql,
            (order, customer, orderItem, product) => {
                // Order nesnesini al veya oluştur
                if (!orderDictionary.TryGetValue(order.Id, out var existingOrder))
                {
                    existingOrder = order;
                    existingOrder.Customer = customer;
                    existingOrder.OrderItems = new List<OrderItem>();
                    orderDictionary.Add(existingOrder.Id, existingOrder);
                }
                
                // OrderItem nesnesini al veya oluştur
                if (!orderItemDictionary.TryGetValue(orderItem.Id, out var existingOrderItem))
                {
                    existingOrderItem = orderItem;
                    existingOrderItem.Product = product;
                    orderItemDictionary.Add(existingOrderItem.Id, existingOrderItem);
                    existingOrder.OrderItems.Add(existingOrderItem);
                }
                
                return existingOrder;
            },
            new { OrderId = orderId },
            splitOn: "Id,Id,Id");
        
        return orderDictionary.Values.FirstOrDefault();
    }
}
```

### Async Transaction

```csharp
// Async transaction
public async Task<bool> CreateOrderAsync(Order order)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        await connection.OpenAsync();
        
        using (var transaction = await connection.BeginTransactionAsync())
        {
            try
            {
                // Sipariş başlığını ekle
                var orderId = await connection.QuerySingleAsync<int>(
                    @"INSERT INTO Orders (CustomerId, OrderDate, TotalAmount)
                      VALUES (@CustomerId, @OrderDate, @TotalAmount);
                      SELECT SCOPE_IDENTITY()",
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
                    
                    await connection.ExecuteAsync(
                        @"INSERT INTO OrderItems (OrderId, ProductId, Quantity, UnitPrice)
                          VALUES (@OrderId, @ProductId, @Quantity, @UnitPrice)",
                        item,
                        transaction);
                    
                    // Ürün stoğunu güncelle
                    await connection.ExecuteAsync(
                        @"UPDATE Products
                          SET Stock = Stock - @Quantity
                          WHERE Id = @ProductId",
                        new { item.ProductId, item.Quantity },
                        transaction);
                }
                
                // Transaction'ı commit et
                await transaction.CommitAsync();
                return true;
            }
            catch (Exception)
            {
                // Hata durumunda transaction'ı geri al
                await transaction.RollbackAsync();
                return false;
            }
        }
    }
}
```

### Async Stored Procedure

```csharp
// Async stored procedure
public async Task<IEnumerable<Product>> GetProductsByCategoryAsync(int categoryId)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        return await connection.QueryAsync<Product>(
            "GetProductsByCategory",
            new { CategoryId = categoryId },
            commandType: CommandType.StoredProcedure);
    }
}

// Async output parametreli stored procedure
public async Task<int> CreateProductAsync(Product product)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        var parameters = new DynamicParameters();
        parameters.Add("@Name", product.Name);
        parameters.Add("@CategoryId", product.CategoryId);
        parameters.Add("@Price", product.Price);
        parameters.Add("@Stock", product.Stock);
        parameters.Add("@ProductId", dbType: DbType.Int32, direction: ParameterDirection.Output);
        
        await connection.ExecuteAsync(
            "CreateProduct",
            parameters,
            commandType: CommandType.StoredProcedure);
        
        return parameters.Get<int>("@ProductId");
    }
}
```

## Pratik Örnekler

### Repository Pattern ile Async Dapper

```csharp
// Async repository pattern
public class ProductRepository : IProductRepository
{
    private readonly string _connectionString;
    
    public ProductRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public async Task<Product> GetByIdAsync(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return await connection.QuerySingleOrDefaultAsync<Product>(
                "SELECT * FROM Products WHERE Id = @Id",
                new { Id = id });
        }
    }
    
    public async Task<IEnumerable<Product>> GetAllAsync()
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return await connection.QueryAsync<Product>("SELECT * FROM Products");
        }
    }
    
    public async Task<int> CreateAsync(Product product)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            var sql = @"
                INSERT INTO Products (Name, CategoryId, Price, Stock)
                VALUES (@Name, @CategoryId, @Price, @Stock);
                SELECT CAST(SCOPE_IDENTITY() as int)";
            
            return await connection.QuerySingleAsync<int>(sql, product);
        }
    }
    
    public async Task<bool> UpdateAsync(Product product)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            int rowsAffected = await connection.ExecuteAsync(
                @"UPDATE Products 
                  SET Name = @Name, 
                      CategoryId = @CategoryId, 
                      Price = @Price,
                      Stock = @Stock
                  WHERE Id = @Id",
                product);
            
            return rowsAffected > 0;
        }
    }
    
    public async Task<bool> DeleteAsync(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            int rowsAffected = await connection.ExecuteAsync(
                "DELETE FROM Products WHERE Id = @Id",
                new { Id = id });
            
            return rowsAffected > 0;
        }
    }
}
```

### Performans Optimizasyonu

```csharp
// Performans optimizasyonu örneği
public async Task<IEnumerable<Product>> GetProductsOptimizedAsync()
{
    using (var connection = new SqlConnection(_connectionString))
    {
        // CommandBehavior.SequentialAccess ile performans optimizasyonu
        using (var reader = await connection.ExecuteReaderAsync(
            "SELECT * FROM Products",
            commandBehavior: CommandBehavior.SequentialAccess))
        {
            var products = new List<Product>();
            
            while (await reader.ReadAsync())
            {
                products.Add(new Product
                {
                    Id = reader.GetInt32(0),
                    Name = reader.GetString(1),
                    CategoryId = reader.GetInt32(2),
                    Price = reader.GetDecimal(3),
                    Stock = reader.GetInt32(4)
                });
            }
            
            return products;
        }
    }
}
```

## Özet

Bu bölümde, Dapper'ın ileri düzey özelliklerini inceledik:

- **Multi-Mapping**: İlişkili nesneleri tek bir sorgu ile eşleştirme
- **Dynamic Parameters**: Parametreleri dinamik olarak oluşturma ve yönetme
- **Bulk Operations**: Büyük miktarda veriyi verimli bir şekilde işleme
- **Transaction Yönetimi**: Veritabanı işlemlerinin atomikliğini sağlama
- **Async Operasyonlar**: Asenkron programlama ile performansı artırma

Dapper'ın bu ileri düzey özellikleri, karmaşık veritabanı işlemlerini daha verimli ve etkili bir şekilde gerçekleştirmenize olanak tanır. Dapper'ın hem basitliği hem de güçlü özellikleri, onu performans odaklı uygulamalar için mükemmel bir seçim haline getirir. 