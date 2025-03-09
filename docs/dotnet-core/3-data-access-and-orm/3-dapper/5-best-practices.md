# Best Practices

Dapper, performans ve kullanım kolaylığı arasında mükemmel bir denge sağlayan güçlü bir mikro-ORM'dir. Ancak, her araçta olduğu gibi, Dapper'ı en etkili şekilde kullanmak için bazı en iyi uygulamaları takip etmek önemlidir. Bu bölümde, Dapper ile çalışırken uygulamanızın kalitesini, güvenliğini ve bakımını artıracak en iyi uygulamaları inceleyeceğiz.

## Repository Pattern ile Kullanım

Repository pattern, veri erişim katmanını soyutlaştırmak ve uygulamanın geri kalanını veritabanı detaylarından izole etmek için kullanılan yaygın bir tasarım desenidir. Dapper ile repository pattern kullanmak, kodunuzu daha modüler, test edilebilir ve bakımı kolay hale getirir.

### Temel Repository Yapısı

```csharp
// Repository arayüzü
public interface IProductRepository
{
    Product GetById(int id);
    IEnumerable<Product> GetAll();
    IEnumerable<Product> GetByCategory(int categoryId);
    int Add(Product product);
    bool Update(Product product);
    bool Delete(int id);
}

// Repository implementasyonu
public class ProductRepository : IProductRepository
{
    private readonly string _connectionString;
    
    // Bağlantı dizesini constructor üzerinden enjekte etme
    public ProductRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    // ID'ye göre ürün getirme
    public Product GetById(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.QuerySingleOrDefault<Product>(
                "SELECT * FROM Products WHERE Id = @Id", 
                new { Id = id });
        }
    }
    
    // Tüm ürünleri getirme
    public IEnumerable<Product> GetAll()
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.Query<Product>("SELECT * FROM Products");
        }
    }
    
    // Kategoriye göre ürünleri getirme
    public IEnumerable<Product> GetByCategory(int categoryId)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.Query<Product>(
                "SELECT * FROM Products WHERE CategoryId = @CategoryId", 
                new { CategoryId = categoryId });
        }
    }
    
    // Yeni ürün ekleme
    public int Add(Product product)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            var sql = @"
                INSERT INTO Products (Name, CategoryId, Price, Stock) 
                VALUES (@Name, @CategoryId, @Price, @Stock);
                SELECT CAST(SCOPE_IDENTITY() as int)";
                
            return connection.QuerySingle<int>(sql, product);
        }
    }
    
    // Ürün güncelleme
    public bool Update(Product product)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            var rowsAffected = connection.Execute(@"
                UPDATE Products 
                SET Name = @Name, 
                    CategoryId = @CategoryId, 
                    Price = @Price, 
                    Stock = @Stock 
                WHERE Id = @Id", product);
                
            return rowsAffected > 0;
        }
    }
    
    // Ürün silme
    public bool Delete(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            var rowsAffected = connection.Execute(
                "DELETE FROM Products WHERE Id = @Id", 
                new { Id = id });
                
            return rowsAffected > 0;
        }
    }
}
```

### Generic Repository

Birden fazla entity için benzer işlemleri tekrarlamaktan kaçınmak için generic repository kullanabilirsiniz:

```csharp
// Generic repository arayüzü
public interface IRepository<T> where T : class
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    int Add(T entity);
    bool Update(T entity);
    bool Delete(int id);
}

// Generic repository implementasyonu
public class Repository<T> : IRepository<T> where T : class
{
    private readonly string _connectionString;
    private readonly string _tableName;
    
    public Repository(string connectionString, string tableName)
    {
        _connectionString = connectionString;
        _tableName = tableName;
    }
    
    public T GetById(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.QuerySingleOrDefault<T>(
                $"SELECT * FROM {_tableName} WHERE Id = @Id", 
                new { Id = id });
        }
    }
    
    public IEnumerable<T> GetAll()
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.Query<T>($"SELECT * FROM {_tableName}");
        }
    }
    
    public int Add(T entity)
    {
        // Bu metot, entity özelliklerine göre dinamik SQL oluşturmalıdır
        // Basitlik için burada gösterilmemiştir
        throw new NotImplementedException();
    }
    
    public bool Update(T entity)
    {
        // Bu metot, entity özelliklerine göre dinamik SQL oluşturmalıdır
        // Basitlik için burada gösterilmemiştir
        throw new NotImplementedException();
    }
    
    public bool Delete(int id)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            var rowsAffected = connection.Execute(
                $"DELETE FROM {_tableName} WHERE Id = @Id", 
                new { Id = id });
                
            return rowsAffected > 0;
        }
    }
}
```

### Unit of Work Pattern

Repository pattern ile birlikte Unit of Work pattern kullanarak, birden fazla repository işlemini tek bir transaction içinde yönetebilirsiniz:

```csharp
// Unit of Work arayüzü
public interface IUnitOfWork : IDisposable
{
    IProductRepository Products { get; }
    ICategoryRepository Categories { get; }
    
    void BeginTransaction();
    void Commit();
    void Rollback();
}

// Unit of Work implementasyonu
public class UnitOfWork : IUnitOfWork
{
    private readonly string _connectionString;
    private SqlConnection _connection;
    private SqlTransaction _transaction;
    
    private IProductRepository _productRepository;
    private ICategoryRepository _categoryRepository;
    
    public UnitOfWork(string connectionString)
    {
        _connectionString = connectionString;
        _connection = new SqlConnection(_connectionString);
        _connection.Open();
    }
    
    public IProductRepository Products => 
        _productRepository ??= new ProductRepository(_connection, _transaction);
        
    public ICategoryRepository Categories => 
        _categoryRepository ??= new CategoryRepository(_connection, _transaction);
    
    public void BeginTransaction()
    {
        _transaction = _connection.BeginTransaction();
    }
    
    public void Commit()
    {
        try
        {
            _transaction.Commit();
        }
        catch
        {
            _transaction.Rollback();
            throw;
        }
        finally
        {
            _transaction.Dispose();
            _transaction = null;
        }
    }
    
    public void Rollback()
    {
        _transaction.Rollback();
        _transaction.Dispose();
        _transaction = null;
    }
    
    public void Dispose()
    {
        if (_transaction != null)
        {
            _transaction.Dispose();
            _transaction = null;
        }
        
        if (_connection != null)
        {
            _connection.Close();
            _connection.Dispose();
            _connection = null;
        }
    }
}

// Transaction ile çalışan repository
public class ProductRepository : IProductRepository
{
    private readonly SqlConnection _connection;
    private readonly SqlTransaction _transaction;
    
    public ProductRepository(SqlConnection connection, SqlTransaction transaction = null)
    {
        _connection = connection;
        _transaction = transaction;
    }
    
    public Product GetById(int id)
    {
        return _connection.QuerySingleOrDefault<Product>(
            "SELECT * FROM Products WHERE Id = @Id", 
            new { Id = id }, 
            _transaction);
    }
    
    // Diğer metotlar...
}
```

## Unit Testing

Dapper ile yazılan kodun test edilebilirliği, özellikle repository pattern kullanıldığında oldukça yüksektir. Ancak, veritabanı bağımlılıklarını yönetmek için bazı stratejiler uygulamanız gerekir.

### In-Memory Veritabanı Kullanma

Test için gerçek bir veritabanı yerine in-memory veritabanı kullanabilirsiniz:

```csharp
// SQLite in-memory veritabanı ile test
public class ProductRepositoryTests
{
    private string _connectionString = "Data Source=:memory:";
    private SqliteConnection _connection;
    
    [SetUp]
    public void Setup()
    {
        // In-memory SQLite veritabanı oluşturma
        _connection = new SqliteConnection(_connectionString);
        _connection.Open();
        
        // Test veritabanı şemasını oluşturma
        _connection.Execute(@"
            CREATE TABLE Products (
                Id INTEGER PRIMARY KEY AUTOINCREMENT,
                Name TEXT NOT NULL,
                CategoryId INTEGER NOT NULL,
                Price REAL NOT NULL,
                Stock INTEGER NOT NULL
            )");
    }
    
    [TearDown]
    public void TearDown()
    {
        _connection.Close();
    }
    
    [Test]
    public void GetById_ExistingProduct_ReturnsProduct()
    {
        // Arrange
        var repository = new ProductRepository(_connection);
        var expectedProduct = new Product 
        { 
            Name = "Test Product", 
            CategoryId = 1, 
            Price = 99.99m, 
            Stock = 10 
        };
        
        var id = repository.Add(expectedProduct);
        
        // Act
        var actualProduct = repository.GetById(id);
        
        // Assert
        Assert.IsNotNull(actualProduct);
        Assert.AreEqual(expectedProduct.Name, actualProduct.Name);
        Assert.AreEqual(expectedProduct.CategoryId, actualProduct.CategoryId);
        Assert.AreEqual(expectedProduct.Price, actualProduct.Price);
        Assert.AreEqual(expectedProduct.Stock, actualProduct.Stock);
    }
}
```

### Mock Kullanma

Repository'leri mock'layarak, veritabanı bağımlılığı olmadan servis katmanını test edebilirsiniz:

```csharp
// Repository'leri mock'lama
public class ProductServiceTests
{
    [Test]
    public void GetProductsInStock_ReturnsOnlyProductsInStock()
    {
        // Arrange
        var mockRepository = new Mock<IProductRepository>();
        mockRepository.Setup(repo => repo.GetAll())
            .Returns(new List<Product>
            {
                new Product { Id = 1, Name = "Product 1", Stock = 10 },
                new Product { Id = 2, Name = "Product 2", Stock = 0 },
                new Product { Id = 3, Name = "Product 3", Stock = 5 }
            });
            
        var service = new ProductService(mockRepository.Object);
        
        // Act
        var productsInStock = service.GetProductsInStock();
        
        // Assert
        Assert.AreEqual(2, productsInStock.Count());
        Assert.IsTrue(productsInStock.All(p => p.Stock > 0));
    }
}

// Test edilen servis
public class ProductService
{
    private readonly IProductRepository _productRepository;
    
    public ProductService(IProductRepository productRepository)
    {
        _productRepository = productRepository;
    }
    
    public IEnumerable<Product> GetProductsInStock()
    {
        return _productRepository.GetAll().Where(p => p.Stock > 0);
    }
}
```

### Integration Testing

Gerçek veritabanı ile integration test yapmak için test veritabanı kullanabilirsiniz:

```csharp
// Integration test
public class ProductRepositoryIntegrationTests
{
    private string _connectionString = "Server=localhost;Database=TestDb;User Id=sa;Password=P@ssw0rd;";
    private IProductRepository _repository;
    
    [SetUp]
    public void Setup()
    {
        // Test veritabanını temizleme
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Execute("DELETE FROM Products");
        }
        
        _repository = new ProductRepository(_connectionString);
    }
    
    [Test]
    public void Add_ValidProduct_ReturnsId()
    {
        // Arrange
        var product = new Product 
        { 
            Name = "Test Product", 
            CategoryId = 1, 
            Price = 99.99m, 
            Stock = 10 
        };
        
        // Act
        var id = _repository.Add(product);
        
        // Assert
        Assert.Greater(id, 0);
        
        // Verify
        var savedProduct = _repository.GetById(id);
        Assert.IsNotNull(savedProduct);
        Assert.AreEqual(product.Name, savedProduct.Name);
    }
}
```

## Hybrid Yaklaşımlar (EF Core + Dapper)

Entity Framework Core ve Dapper'ı birlikte kullanarak, her ikisinin de avantajlarından yararlanabilirsiniz. EF Core, CRUD işlemleri ve ilişki yönetimi için kullanılırken, Dapper karmaşık sorgular ve performans kritik işlemler için kullanılabilir.

### DbContext ile Dapper Entegrasyonu

```csharp
// EF Core ve Dapper'ı birlikte kullanma
public class HybridRepository : IProductRepository
{
    private readonly AppDbContext _dbContext;
    
    public HybridRepository(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    // EF Core ile basit CRUD işlemleri
    public Product GetById(int id)
    {
        return _dbContext.Products.Find(id);
    }
    
    public int Add(Product product)
    {
        _dbContext.Products.Add(product);
        _dbContext.SaveChanges();
        return product.Id;
    }
    
    public bool Update(Product product)
    {
        _dbContext.Products.Update(product);
        return _dbContext.SaveChanges() > 0;
    }
    
    public bool Delete(int id)
    {
        var product = _dbContext.Products.Find(id);
        if (product == null) return false;
        
        _dbContext.Products.Remove(product);
        return _dbContext.SaveChanges() > 0;
    }
    
    // Dapper ile karmaşık sorgular
    public IEnumerable<ProductSalesDto> GetTopSellingProducts(int count)
    {
        var connection = _dbContext.Database.GetDbConnection();
        
        var sql = @"
            SELECT p.Id, p.Name, p.Price, SUM(oi.Quantity) as TotalSold
            FROM Products p
            JOIN OrderItems oi ON p.Id = oi.ProductId
            GROUP BY p.Id, p.Name, p.Price
            ORDER BY TotalSold DESC
            LIMIT @Count";
            
        return connection.Query<ProductSalesDto>(sql, new { Count = count });
    }
    
    public IEnumerable<ProductInventoryDto> GetLowStockProducts(int threshold)
    {
        var connection = _dbContext.Database.GetDbConnection();
        
        var sql = @"
            SELECT p.Id, p.Name, p.Stock, c.Name as CategoryName
            FROM Products p
            JOIN Categories c ON p.CategoryId = c.Id
            WHERE p.Stock <= @Threshold
            ORDER BY p.Stock";
            
        return connection.Query<ProductInventoryDto>(sql, new { Threshold = threshold });
    }
}

// DTO sınıfları
public class ProductSalesDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int TotalSold { get; set; }
}

public class ProductInventoryDto
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int Stock { get; set; }
    public string CategoryName { get; set; }
}
```

### Transaction Paylaşımı

EF Core ve Dapper arasında transaction paylaşarak, atomik işlemler gerçekleştirebilirsiniz:

```csharp
// EF Core ve Dapper arasında transaction paylaşımı
public class OrderService
{
    private readonly AppDbContext _dbContext;
    
    public OrderService(AppDbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    public async Task<bool> ProcessOrderAsync(Order order)
    {
        // Transaction başlatma
        using var transaction = await _dbContext.Database.BeginTransactionAsync();
        try
        {
            // EF Core ile sipariş ekleme
            _dbContext.Orders.Add(order);
            await _dbContext.SaveChangesAsync();
            
            // Dapper ile stok güncelleme
            var connection = _dbContext.Database.GetDbConnection();
            foreach (var item in order.Items)
            {
                var rowsAffected = await connection.ExecuteAsync(
                    "UPDATE Products SET Stock = Stock - @Quantity WHERE Id = @ProductId AND Stock >= @Quantity",
                    new { item.ProductId, item.Quantity },
                    transaction: transaction.GetDbTransaction());
                    
                if (rowsAffected == 0)
                {
                    // Stok yetersiz, transaction'ı geri al
                    await transaction.RollbackAsync();
                    return false;
                }
            }
            
            // Transaction'ı commit et
            await transaction.CommitAsync();
            return true;
        }
        catch
        {
            // Hata durumunda transaction'ı geri al
            await transaction.RollbackAsync();
            throw;
        }
    }
}
```

## SQL Injection Önleme

SQL Injection, en yaygın ve tehlikeli güvenlik açıklarından biridir. Dapper, parametreli sorgular kullanarak SQL Injection saldırılarına karşı koruma sağlar, ancak bu korumanın etkili olması için doğru kullanılması gerekir.

### Parametreli Sorgular Kullanma

```csharp
// DOĞRU: Parametreli sorgu kullanımı
public Product GetByName(string name)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        return connection.QuerySingleOrDefault<Product>(
            "SELECT * FROM Products WHERE Name = @Name", 
            new { Name = name });
    }
}

// YANLIŞ: String birleştirme (SQL Injection riski)
public Product GetByNameUnsafe(string name)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        // Bu kod SQL Injection'a açıktır!
        return connection.QuerySingleOrDefault<Product>(
            $"SELECT * FROM Products WHERE Name = '{name}'");
    }
}
```

### Dinamik Sorgularda Güvenlik

```csharp
// Dinamik sorgularda güvenlik
public IEnumerable<Product> SearchProducts(string name = null, int? categoryId = null)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        var conditions = new List<string>();
        var parameters = new DynamicParameters();
        
        if (!string.IsNullOrEmpty(name))
        {
            // DOĞRU: Parametre kullanma
            conditions.Add("Name LIKE @Name");
            parameters.Add("@Name", $"%{name}%");
        }
        
        if (categoryId.HasValue)
        {
            // DOĞRU: Parametre kullanma
            conditions.Add("CategoryId = @CategoryId");
            parameters.Add("@CategoryId", categoryId.Value);
        }
        
        var whereClause = conditions.Any() ? " WHERE " + string.Join(" AND ", conditions) : "";
        var sql = "SELECT * FROM Products" + whereClause;
        
        return connection.Query<Product>(sql, parameters);
    }
}
```

### Stored Procedure Kullanma

```csharp
// Stored procedure kullanma
public IEnumerable<Product> GetProductsByCategory(int categoryId)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        return connection.Query<Product>(
            "GetProductsByCategory", 
            new { CategoryId = categoryId }, 
            commandType: CommandType.StoredProcedure);
    }
}
```

## Error Handling

Dapper ile çalışırken, veritabanı işlemlerinde oluşabilecek hataları düzgün bir şekilde yönetmek önemlidir. İyi bir hata yönetimi, uygulamanızın güvenilirliğini ve bakımını kolaylaştırır.

### Try-Catch Blokları

```csharp
// Temel hata yönetimi
public Product GetById(int id)
{
    try
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            return connection.QuerySingleOrDefault<Product>(
                "SELECT * FROM Products WHERE Id = @Id", 
                new { Id = id });
        }
    }
    catch (SqlException ex)
    {
        // SQL hatalarını loglama
        _logger.LogError(ex, "Ürün getirme hatası: {Id}", id);
        throw new DatabaseException("Ürün veritabanından getirilemedi", ex);
    }
    catch (Exception ex)
    {
        // Diğer hataları loglama
        _logger.LogError(ex, "Beklenmeyen hata: {Id}", id);
        throw;
    }
}
```

### Özel Exception Sınıfları

```csharp
// Özel exception sınıfları
public class DatabaseException : Exception
{
    public DatabaseException(string message) : base(message)
    {
    }
    
    public DatabaseException(string message, Exception innerException) 
        : base(message, innerException)
    {
    }
}

public class EntityNotFoundException : Exception
{
    public EntityNotFoundException(string entityName, object id) 
        : base($"{entityName} with ID {id} not found")
    {
        EntityName = entityName;
        Id = id;
    }
    
    public string EntityName { get; }
    public object Id { get; }
}
```

### Repository'de Hata Yönetimi

```csharp
// Repository'de hata yönetimi
public class ProductRepository : IProductRepository
{
    private readonly string _connectionString;
    private readonly ILogger<ProductRepository> _logger;
    
    public ProductRepository(string connectionString, ILogger<ProductRepository> logger)
    {
        _connectionString = connectionString;
        _logger = logger;
    }
    
    public Product GetById(int id)
    {
        try
        {
            using (var connection = new SqlConnection(_connectionString))
            {
                var product = connection.QuerySingleOrDefault<Product>(
                    "SELECT * FROM Products WHERE Id = @Id", 
                    new { Id = id });
                    
                if (product == null)
                {
                    throw new EntityNotFoundException("Product", id);
                }
                
                return product;
            }
        }
        catch (EntityNotFoundException)
        {
            // EntityNotFoundException'ı yeniden fırlatma
            throw;
        }
        catch (SqlException ex)
        {
            _logger.LogError(ex, "SQL hatası: {Id}", id);
            throw new DatabaseException("Veritabanı hatası oluştu", ex);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Beklenmeyen hata: {Id}", id);
            throw;
        }
    }
    
    // Diğer metotlar...
}
```

### Transaction ile Hata Yönetimi

```csharp
// Transaction ile hata yönetimi
public bool TransferFunds(int fromAccountId, int toAccountId, decimal amount)
{
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        using (var transaction = connection.BeginTransaction())
        {
            try
            {
                // Para çekme
                var withdrawResult = connection.Execute(
                    "UPDATE Accounts SET Balance = Balance - @Amount WHERE Id = @AccountId AND Balance >= @Amount",
                    new { AccountId = fromAccountId, Amount = amount },
                    transaction);
                    
                if (withdrawResult == 0)
                {
                    // Yetersiz bakiye
                    transaction.Rollback();
                    return false;
                }
                
                // Para yatırma
                connection.Execute(
                    "UPDATE Accounts SET Balance = Balance + @Amount WHERE Id = @AccountId",
                    new { AccountId = toAccountId, Amount = amount },
                    transaction);
                
                // İşlem kaydı
                connection.Execute(
                    "INSERT INTO Transactions (FromAccountId, ToAccountId, Amount, Date) VALUES (@FromAccountId, @ToAccountId, @Amount, @Date)",
                    new { FromAccountId = fromAccountId, ToAccountId = toAccountId, Amount = amount, Date = DateTime.Now },
                    transaction);
                
                // Transaction'ı commit et
                transaction.Commit();
                return true;
            }
            catch (Exception ex)
            {
                // Hata durumunda transaction'ı geri al
                transaction.Rollback();
                _logger.LogError(ex, "Para transferi hatası: {FromAccountId} -> {ToAccountId}, {Amount}", 
                    fromAccountId, toAccountId, amount);
                throw;
            }
        }
    }
}
```

## Özet

Bu bölümde, Dapper ile çalışırken uygulamanızın kalitesini, güvenliğini ve bakımını artıracak en iyi uygulamaları inceledik:

- **Repository Pattern ile Kullanım**: Veri erişim katmanını soyutlaştırarak kodunuzu daha modüler ve test edilebilir hale getirme
- **Unit Testing**: Dapper ile yazılan kodun test edilebilirliğini artırma stratejileri
- **Hybrid Yaklaşımlar**: EF Core ve Dapper'ı birlikte kullanarak her ikisinin de avantajlarından yararlanma
- **SQL Injection Önleme**: Parametreli sorgular ve diğer güvenlik önlemleri ile SQL Injection saldırılarına karşı koruma
- **Error Handling**: Veritabanı işlemlerinde oluşabilecek hataları düzgün bir şekilde yönetme

Bu en iyi uygulamaları takip ederek, Dapper ile daha güvenli, bakımı kolay ve performanslı uygulamalar geliştirebilirsiniz. 