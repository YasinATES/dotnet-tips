# Transactions ve Concurrency

Entity Framework Core, veritabanı işlemlerinin güvenilir ve tutarlı bir şekilde gerçekleştirilmesini sağlamak için transaction ve concurrency yönetimi mekanizmaları sunar. Bu bölümde, EF Core'da transaction kullanımını ve eşzamanlılık (concurrency) kontrolünü inceleyeceğiz.

## Transaction Scope

Transaction, bir veya daha fazla veritabanı işleminin atomik bir birim olarak gerçekleştirilmesini sağlar. Yani, transaction içindeki tüm işlemler ya tamamen başarılı olur ya da hiçbiri gerçekleşmez (ya hep ya hiç prensibi).

### System.Transactions Namespace ile Transaction Kullanımı

```csharp
// TransactionScope ile transaction yönetimi
using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    // İlk veritabanı işlemi
    using (var context1 = new OrderContext())
    {
        var order = new Order { CustomerId = 1, OrderDate = DateTime.Now };
        context1.Orders.Add(order);
        await context1.SaveChangesAsync();
    }
    
    // İkinci veritabanı işlemi (farklı context ile)
    using (var context2 = new InventoryContext())
    {
        var inventory = await context2.Inventories.FindAsync(productId);
        inventory.Stock -= quantity;
        await context2.SaveChangesAsync();
    }
    
    // Tüm işlemler başarılı ise transaction'ı tamamla
    scope.Complete();
}
// scope dispose edildiğinde, Complete() çağrılmadıysa transaction geri alınır
```

### Farklı Veritabanları ile Transaction Kullanımı

```csharp
// Farklı veritabanları arasında transaction
using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
{
    // SQL Server veritabanı işlemi
    using (var sqlContext = new SqlServerContext())
    {
        var customer = new Customer { Name = "Ahmet Yılmaz" };
        sqlContext.Customers.Add(customer);
        await sqlContext.SaveChangesAsync();
    }
    
    // PostgreSQL veritabanı işlemi
    using (var npgsqlContext = new PostgreSqlContext())
    {
        var log = new AuditLog { Action = "Müşteri Eklendi", Timestamp = DateTime.Now };
        npgsqlContext.AuditLogs.Add(log);
        await npgsqlContext.SaveChangesAsync();
    }
    
    // Her iki işlem de başarılı ise transaction'ı tamamla
    scope.Complete();
}
```

### Transaction Seçenekleri

```csharp
// Transaction izolasyon seviyesi belirleme
var options = new TransactionOptions
{
    IsolationLevel = IsolationLevel.ReadCommitted,
    Timeout = TimeSpan.FromSeconds(30)
};

using (var scope = new TransactionScope(TransactionScopeOption.Required, options))
{
    // Transaction işlemleri...
    
    scope.Complete();
}
```

## SaveChanges ve SaveChangesAsync

EF Core'da `SaveChanges` ve `SaveChangesAsync` metodları, varsayılan olarak kendi içlerinde bir transaction kullanır. Bu, tek bir context üzerinde yapılan değişikliklerin atomik olarak kaydedilmesini sağlar.

### Varsayılan Transaction Davranışı

```csharp
// SaveChanges varsayılan olarak transaction kullanır
using (var context = new ApplicationDbContext())
{
    // Birden fazla değişiklik
    var customer = new Customer { Name = "Mehmet Kaya" };
    context.Customers.Add(customer);
    
    var existingCustomer = await context.Customers.FindAsync(1);
    existingCustomer.Email = "yeni@email.com";
    
    // Tüm değişiklikler tek bir transaction içinde kaydedilir
    await context.SaveChangesAsync();
}
```

### Açık Transaction Kullanımı

```csharp
// DbContext üzerinde açık transaction başlatma
using (var context = new ApplicationDbContext())
{
    // Transaction başlat
    using (var transaction = await context.Database.BeginTransactionAsync())
    {
        try
        {
            // İlk işlem
            var order = new Order { CustomerId = 1, OrderDate = DateTime.Now };
            context.Orders.Add(order);
            await context.SaveChangesAsync();
            
            // İkinci işlem
            var payment = new Payment { OrderId = order.Id, Amount = 100.50m };
            context.Payments.Add(payment);
            await context.SaveChangesAsync();
            
            // Transaction'ı commit et
            await transaction.CommitAsync();
        }
        catch (Exception)
        {
            // Hata durumunda transaction'ı geri al
            await transaction.RollbackAsync();
            throw;
        }
    }
}
```

### Var Olan Transaction'a Katılma

```csharp
// Var olan transaction'a katılma
using (var context1 = new OrderContext())
{
    using (var transaction = await context1.Database.BeginTransactionAsync())
    {
        // İlk context ile işlemler
        var order = new Order { CustomerId = 1, OrderDate = DateTime.Now };
        context1.Orders.Add(order);
        await context1.SaveChangesAsync();
        
        // İkinci context ile aynı transaction'ı kullanma
        using (var context2 = new PaymentContext())
        {
            // Var olan transaction'a katıl
            await context2.Database.UseTransactionAsync(transaction.GetDbTransaction());
            
            var payment = new Payment { OrderId = order.Id, Amount = 100.50m };
            context2.Payments.Add(payment);
            await context2.SaveChangesAsync();
        }
        
        // Transaction'ı commit et
        await transaction.CommitAsync();
    }
}
```

### Savepoint Kullanımı

```csharp
// Transaction içinde savepoint kullanımı
using (var context = new ApplicationDbContext())
{
    using (var transaction = await context.Database.BeginTransactionAsync())
    {
        try
        {
            // İlk işlem
            var customer = new Customer { Name = "Ali Veli" };
            context.Customers.Add(customer);
            await context.SaveChangesAsync();
            
            // Savepoint oluştur
            await transaction.CreateSavepointAsync("AfterCustomerCreation");
            
            try
            {
                // İkinci işlem
                var order = new Order { CustomerId = customer.Id, OrderDate = DateTime.Now };
                context.Orders.Add(order);
                await context.SaveChangesAsync();
            }
            catch (Exception)
            {
                // Sadece ikinci işlemi geri al
                await transaction.RollbackToSavepointAsync("AfterCustomerCreation");
            }
            
            // Transaction'ı commit et
            await transaction.CommitAsync();
        }
        catch (Exception)
        {
            // Tüm transaction'ı geri al
            await transaction.RollbackAsync();
            throw;
        }
    }
}
```

## Optimistic Concurrency Control

Optimistic concurrency control (iyimser eşzamanlılık kontrolü), aynı verinin aynı anda birden fazla kullanıcı tarafından değiştirilmesini engellemeye yönelik bir yaklaşımdır. Bu yaklaşımda, veri çakışmaları nadir olduğu varsayılır ve çakışmalar ancak veritabanına yazma sırasında kontrol edilir.

### Concurrency Token Kullanımı

```csharp
// Concurrency token ile model tanımlama
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int Stock { get; set; }
    
    // Concurrency token olarak kullanılacak özellik
    [ConcurrencyCheck]
    public byte[] RowVersion { get; set; }
}

// Fluent API ile concurrency token tanımlama
modelBuilder.Entity<Product>()
    .Property(p => p.RowVersion)
    .IsRowVersion();
```

### DbUpdateConcurrencyException Yakalama

```csharp
// Concurrency çakışmalarını yakalama ve çözme
try
{
    // Ürünü getir
    var product = await context.Products.FindAsync(1);
    
    // Ürünü güncelle
    product.Price = 29.99m;
    
    // Değişiklikleri kaydet
    await context.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException ex)
{
    // Çakışma durumunda
    var entry = ex.Entries.Single();
    var databaseValues = await entry.GetDatabaseValuesAsync();
    
    if (databaseValues == null)
    {
        // Kayıt silinmiş
        ModelState.AddModelError(string.Empty, "Ürün başka bir kullanıcı tarafından silinmiş.");
    }
    else
    {
        // Kayıt güncellenmiş
        var databaseProduct = (Product)databaseValues.ToObject();
        
        // Kullanıcıya çakışma bilgisi göster
        ModelState.AddModelError("Price", $"Mevcut değer: {databaseProduct.Price}");
        ModelState.AddModelError("Stock", $"Mevcut değer: {databaseProduct.Stock}");
    }
}
```

### Çakışmaları Çözme Stratejileri

```csharp
// Çakışmaları çözme: Client Wins
try
{
    await context.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException ex)
{
    foreach (var entry in ex.Entries)
    {
        // Veritabanındaki mevcut değerleri al
        var databaseValues = await entry.GetDatabaseValuesAsync();
        
        // Mevcut değerleri entry'ye yükle
        entry.OriginalValues.SetValues(databaseValues);
        
        // Tekrar kaydetmeyi dene (client değerleri kazanır)
        await context.SaveChangesAsync();
    }
}

// Çakışmaları çözme: Database Wins
try
{
    await context.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException ex)
{
    foreach (var entry in ex.Entries)
    {
        // Veritabanındaki mevcut değerleri al
        var databaseValues = await entry.GetDatabaseValuesAsync();
        
        // Client değerlerini geçici olarak sakla
        var proposedValues = entry.CurrentValues.Clone();
        
        // Entry'yi veritabanı değerleriyle güncelle
        entry.CurrentValues.SetValues(databaseValues);
        
        // Sadece belirli özellikleri client değerleriyle güncelle
        entry.CurrentValues["Name"] = proposedValues["Name"];
        
        // Tekrar kaydetmeyi dene
        await context.SaveChangesAsync();
    }
}
```

### Timestamp Kullanımı

```csharp
// Timestamp ile concurrency kontrolü
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    
    // SQL Server'da timestamp/rowversion olarak kullanılır
    [Timestamp]
    public byte[] RowVersion { get; set; }
}

// Fluent API ile timestamp tanımlama
modelBuilder.Entity<Customer>()
    .Property(c => c.RowVersion)
    .IsRowVersion();
```

## Pessimistic Concurrency

Pessimistic concurrency control (kötümser eşzamanlılık kontrolü), verilerin okunduğu anda kilitlenmesini sağlayarak çakışmaları önleyen bir yaklaşımdır. Bu yaklaşım, çakışmaların sık olduğu durumlarda tercih edilir.

### Kilit Tipleri

```csharp
// Okuma kilidi (shared lock)
using (var transaction = await context.Database.BeginTransactionAsync())
{
    // HOLDLOCK ile okuma kilidi
    var customers = await context.Customers
        .FromSqlRaw("SELECT * FROM Customers WITH (HOLDLOCK)")
        .ToListAsync();
    
    // İşlemler...
    
    await transaction.CommitAsync();
}

// Yazma kilidi (exclusive lock)
using (var transaction = await context.Database.BeginTransactionAsync())
{
    // UPDLOCK ile yazma kilidi
    var product = await context.Products
        .FromSqlRaw("SELECT * FROM Products WITH (UPDLOCK) WHERE Id = {0}", productId)
        .FirstOrDefaultAsync();
    
    // Ürünü güncelle
    product.Stock -= quantity;
    await context.SaveChangesAsync();
    
    await transaction.CommitAsync();
}
```

### LINQ ile Kilit Kullanımı

```csharp
// LINQ sorgularında kilit kullanımı
using (var transaction = await context.Database.BeginTransactionAsync())
{
    // WithTableHints extension metodu ile kilit belirtme
    var products = await context.Products
        .Where(p => p.CategoryId == categoryId)
        .WithTableHints(TableHints.UpdLock, TableHints.HoldLock)
        .ToListAsync();
    
    // İşlemler...
    
    await transaction.CommitAsync();
}
```

### Deadlock Önleme

```csharp
// Deadlock önleme için timeout ayarlama
using (var transaction = await context.Database.BeginTransactionAsync())
{
    try
    {
        // READCOMMITTED izolasyon seviyesi ve timeout ayarı
        await context.Database.ExecuteSqlRawAsync(
            "SET LOCK_TIMEOUT 5000; SET TRANSACTION ISOLATION LEVEL READ COMMITTED;");
        
        // Kilitleme ile sorgu
        var product = await context.Products
            .FromSqlRaw("SELECT * FROM Products WITH (UPDLOCK) WHERE Id = {0}", productId)
            .FirstOrDefaultAsync();
        
        // İşlemler...
        
        await transaction.CommitAsync();
    }
    catch (SqlException ex) when (ex.Number == 1222) // Lock request timeout
    {
        // Timeout durumunda
        await transaction.RollbackAsync();
        throw new TimeoutException("İşlem zaman aşımına uğradı. Lütfen daha sonra tekrar deneyin.", ex);
    }
}
```

## Concurrency Tokens ve Timestamps

Concurrency token'lar, optimistic concurrency control için kullanılan ve bir kaydın son değiştirilme durumunu izleyen özelliklerdir. Timestamp'ler ise SQL Server'da özel olarak desteklenen ve otomatik olarak güncellenen binary concurrency token'lardır.

### Farklı Concurrency Token Tipleri

```csharp
// Sayısal concurrency token
public class Article
{
    public int Id { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    
    // Sayısal concurrency token
    [ConcurrencyCheck]
    public int Version { get; set; }
}

// Tarih bazlı concurrency token
public class Comment
{
    public int Id { get; set; }
    public string Text { get; set; }
    
    // Tarih bazlı concurrency token
    [ConcurrencyCheck]
    public DateTime LastUpdated { get; set; }
}

// Çoklu özellik concurrency token
modelBuilder.Entity<User>()
    .Property(u => u.Email)
    .IsConcurrencyToken();
    
modelBuilder.Entity<User>()
    .Property(u => u.LastLoginDate)
    .IsConcurrencyToken();
```

### Concurrency Token Güncelleme

```csharp
// Concurrency token manuel güncelleme
public override int SaveChanges()
{
    // Değişen entity'leri bul
    var entries = ChangeTracker.Entries()
        .Where(e => e.State == EntityState.Modified);
    
    foreach (var entry in entries)
    {
        // Sayısal concurrency token güncelleme
        if (entry.Entity is Article article)
        {
            article.Version++;
        }
        
        // Tarih bazlı concurrency token güncelleme
        if (entry.Entity is Comment comment)
        {
            comment.LastUpdated = DateTime.UtcNow;
        }
    }
    
    return base.SaveChanges();
}
```

### Timestamp Kullanım Senaryoları

```csharp
// Timestamp ile entity tanımlama
[Table("Employees")]
public class Employee
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Department { get; set; }
    public decimal Salary { get; set; }
    
    [Timestamp]
    public byte[] RowVersion { get; set; }
}

// Timestamp değerini form'da saklama
@model Employee

<form asp-action="Edit">
    <input type="hidden" asp-for="Id" />
    <input type="hidden" asp-for="RowVersion" />
    
    <div class="form-group">
        <label asp-for="Name"></label>
        <input asp-for="Name" class="form-control" />
    </div>
    
    <!-- Diğer form alanları -->
    
    <button type="submit" class="btn btn-primary">Kaydet</button>
</form>
```

### Concurrency Çakışmalarını Loglama

```csharp
// Concurrency çakışmalarını loglama
try
{
    await context.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException ex)
{
    foreach (var entry in ex.Entries)
    {
        var proposedValues = entry.CurrentValues;
        var databaseValues = await entry.GetDatabaseValuesAsync();
        
        foreach (var property in proposedValues.Properties)
        {
            var proposedValue = proposedValues[property];
            var databaseValue = databaseValues[property];
            
            // Değerler farklı ise log'la
            if (!Equals(proposedValue, databaseValue))
            {
                _logger.LogWarning(
                    "Concurrency çakışması: {EntityType} (ID: {Id}), Özellik: {Property}, " +
                    "Önerilen değer: {ProposedValue}, Veritabanı değeri: {DatabaseValue}",
                    entry.Entity.GetType().Name,
                    entry.Property("Id").CurrentValue,
                    property.Name,
                    proposedValue,
                    databaseValue);
            }
        }
    }
    
    // Çakışmayı kullanıcıya bildir
    throw new ConcurrencyException("Düzenlediğiniz kayıt başka bir kullanıcı tarafından değiştirilmiş.");
}
```

## Pratik Örnekler

### Para Transferi Örneği

```csharp
// Banka hesapları arası para transferi
public async Task<bool> TransferMoneyAsync(int fromAccountId, int toAccountId, decimal amount)
{
    using (var scope = new TransactionScope(TransactionScopeAsyncFlowOption.Enabled))
    {
        try
        {
            // Gönderen hesabı güncelle
            using (var context1 = new BankContext())
            {
                var fromAccount = await context1.Accounts
                    .FindAsync(fromAccountId);
                
                if (fromAccount.Balance < amount)
                    throw new InvalidOperationException("Yetersiz bakiye");
                
                fromAccount.Balance -= amount;
                await context1.SaveChangesAsync();
            }
            
            // Alıcı hesabı güncelle
            using (var context2 = new BankContext())
            {
                var toAccount = await context2.Accounts
                    .FindAsync(toAccountId);
                
                toAccount.Balance += amount;
                await context2.SaveChangesAsync();
            }
            
            // İşlem kaydı oluştur
            using (var context3 = new BankContext())
            {
                var transaction = new BankTransaction
                {
                    FromAccountId = fromAccountId,
                    ToAccountId = toAccountId,
                    Amount = amount,
                    TransactionDate = DateTime.Now
                };
                
                context3.Transactions.Add(transaction);
                await context3.SaveChangesAsync();
            }
            
            // Tüm işlemler başarılı ise transaction'ı tamamla
            scope.Complete();
            return true;
        }
        catch (Exception)
        {
            // Herhangi bir hata durumunda transaction otomatik olarak geri alınır
            return false;
        }
    }
}
```

### Stok Yönetimi Örneği

```csharp
// Stok yönetimi ile ürün satışı
public async Task<bool> SellProductAsync(int productId, int quantity, int customerId)
{
    using (var context = new ShopContext())
    {
        // Transaction başlat
        using (var transaction = await context.Database.BeginTransactionAsync(IsolationLevel.RepeatableRead))
        {
            try
            {
                // Ürünü kilit ile getir
                var product = await context.Products
                    .FromSqlRaw("SELECT * FROM Products WITH (UPDLOCK) WHERE Id = {0}", productId)
                    .FirstOrDefaultAsync();
                
                if (product == null)
                    throw new NotFoundException("Ürün bulunamadı");
                
                if (product.Stock < quantity)
                    throw new InvalidOperationException("Yetersiz stok");
                
                // Stok güncelle
                product.Stock -= quantity;
                
                // Sipariş oluştur
                var order = new Order
                {
                    CustomerId = customerId,
                    OrderDate = DateTime.Now,
                    OrderItems = new List<OrderItem>
                    {
                        new OrderItem
                        {
                            ProductId = productId,
                            Quantity = quantity,
                            UnitPrice = product.Price
                        }
                    },
                    TotalAmount = product.Price * quantity
                };
                
                context.Orders.Add(order);
                
                // Değişiklikleri kaydet
                await context.SaveChangesAsync();
                
                // Transaction'ı commit et
                await transaction.CommitAsync();
                return true;
            }
            catch (Exception)
            {
                // Hata durumunda transaction'ı geri al
                await transaction.RollbackAsync();
                throw;
            }
        }
    }
}
```

### Concurrency Yönetimi ile Rezervasyon Sistemi

```csharp
// Otel rezervasyon sistemi
public async Task<bool> BookRoomAsync(int roomId, DateTime checkIn, DateTime checkOut, int guestId)
{
    using (var context = new HotelContext())
    {
        // Odayı concurrency token ile getir
        var room = await context.Rooms
            .Include(r => r.Reservations)
            .FirstOrDefaultAsync(r => r.Id == roomId);
        
        if (room == null)
            throw new NotFoundException("Oda bulunamadı");
        
        // Tarih çakışması kontrolü
        bool isAvailable = !room.Reservations.Any(r =>
            (checkIn >= r.CheckIn && checkIn < r.CheckOut) ||
            (checkOut > r.CheckIn && checkOut <= r.CheckOut) ||
            (checkIn <= r.CheckIn && checkOut >= r.CheckOut));
        
        if (!isAvailable)
            throw new InvalidOperationException("Seçilen tarihler için oda müsait değil");
        
        // Yeni rezervasyon oluştur
        var reservation = new Reservation
        {
            RoomId = roomId,
            GuestId = guestId,
            CheckIn = checkIn,
            CheckOut = checkOut,
            ReservationDate = DateTime.Now,
            Status = ReservationStatus.Confirmed
        };
        
        context.Reservations.Add(reservation);
        
        try
        {
            // Değişiklikleri kaydet (concurrency kontrolü burada gerçekleşir)
            await context.SaveChangesAsync();
            return true;
        }
        catch (DbUpdateConcurrencyException)
        {
            // Başka bir kullanıcı aynı odayı rezerve etmiş olabilir
            return false;
        }
    }
}
```

## Özet

Bu bölümde, Entity Framework Core'da transaction ve concurrency yönetimini inceledik:

- **Transaction Scope**: System.Transactions namespace ile transaction yönetimi
- **SaveChanges ve SaveChangesAsync**: DbContext'in transaction davranışı
- **Optimistic Concurrency Control**: Çakışmaları yazma sırasında kontrol etme
- **Pessimistic Concurrency**: Verileri okuma sırasında kilitleme
- **Concurrency Tokens ve Timestamps**: Veri tutarlılığını sağlama mekanizmaları

Transaction ve concurrency yönetimi, veri bütünlüğünü korumak ve çoklu kullanıcı senaryolarında güvenilir uygulamalar geliştirmek için kritik öneme sahiptir. Entity Framework Core, bu konularda esnek ve güçlü çözümler sunarak, karmaşık veritabanı işlemlerini yönetmeyi kolaylaştırır. 