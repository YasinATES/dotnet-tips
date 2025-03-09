# Advanced Entity Framework Core

Entity Framework Core, temel işlevlerin ötesinde, veritabanı uygulamalarınızı daha güçlü ve esnek hale getiren çeşitli ileri düzey özellikler sunar. Bu bölümde, bu ileri düzey özellikleri ve kullanım senaryolarını inceleyeceğiz.

## Global Query Filters

Global query filtreler, belirli bir entity tipi için otomatik olarak uygulanan sorgu filtreleridir. Bu filtreler, her sorgu için manuel olarak belirtmenize gerek kalmadan, belirli koşulları otomatik olarak ekler.

### Temel Kullanım

```csharp
// DbContext içinde global filtre tanımlama
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Tüm müşteri sorgularına IsActive filtresi uygulama
    modelBuilder.Entity<Customer>()
        .HasQueryFilter(c => c.IsActive);
        
    // Çok kiracılı (multi-tenant) uygulama için filtre
    modelBuilder.Entity<Order>()
        .HasQueryFilter(o => o.TenantId == _tenantId);
}
```

### Filtreleri Geçici Olarak Devre Dışı Bırakma

```csharp
// Global filtreyi geçici olarak devre dışı bırakma
var allCustomers = await context.Customers
    .IgnoreQueryFilters()
    .ToListAsync();
```

### Pratik Kullanım Senaryoları

```csharp
// Soft-delete (yumuşak silme) için global filtre
modelBuilder.Entity<Product>()
    .HasQueryFilter(p => !p.IsDeleted);
    
// Kullanıcıya özgü veri filtreleme
modelBuilder.Entity<Document>()
    .HasQueryFilter(d => d.UserId == _currentUserId || d.IsPublic);
    
// Zaman bazlı filtreleme
modelBuilder.Entity<Promotion>()
    .HasQueryFilter(p => p.StartDate <= DateTime.Now && p.EndDate >= DateTime.Now);
```

### Birden Fazla Filtre Birleştirme

```csharp
// Birden fazla koşulu birleştirme
modelBuilder.Entity<Invoice>()
    .HasQueryFilter(i => i.TenantId == _tenantId && !i.IsDeleted && i.IsApproved);
```

## Shadow Properties

Shadow property'ler, entity sınıflarında tanımlanmayan ancak veritabanında var olan özelliklerdir. Bu özellikler, entity'lerin C# sınıflarını değiştirmeden ek veriler saklamanıza olanak tanır.

### Shadow Property Tanımlama

```csharp
// Shadow property tanımlama
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Customer>()
        .Property<DateTime>("LastUpdated");
        
    modelBuilder.Entity<Order>()
        .Property<string>("CreatedBy")
        .HasMaxLength(100);
}
```

### Shadow Property Değerlerini Okuma ve Yazma

```csharp
// Shadow property değerini okuma
var lastUpdated = context.Entry(customer)
    .Property<DateTime>("LastUpdated")
    .CurrentValue;
    
// Shadow property değerini yazma
context.Entry(customer)
    .Property<DateTime>("LastUpdated")
    .CurrentValue = DateTime.Now;
```

### Shadow Property ile Sorgulama

```csharp
// Shadow property ile sorgulama
var recentlyUpdatedCustomers = await context.Customers
    .Where(c => EF.Property<DateTime>(c, "LastUpdated") >= DateTime.Now.AddDays(-7))
    .ToListAsync();
```

### Otomatik Değer Atama

```csharp
// SaveChanges sırasında otomatik değer atama
public override int SaveChanges()
{
    var entries = ChangeTracker.Entries()
        .Where(e => e.State == EntityState.Added || e.State == EntityState.Modified);
        
    foreach (var entry in entries)
    {
        entry.Property<DateTime>("LastUpdated").CurrentValue = DateTime.Now;
        
        if (entry.State == EntityState.Added)
        {
            entry.Property<string>("CreatedBy").CurrentValue = _currentUser;
        }
    }
    
    return base.SaveChanges();
}
```

## Owned Entity Types

Owned entity type'lar, kendi başlarına bir entity olmayan, ancak başka bir entity'nin parçası olan kompleks tipleri modellemenize olanak tanır. Bu, veritabanı şemanızı daha iyi organize etmenize ve domain modelinizi daha doğru yansıtmanıza yardımcı olur.

### Owned Entity Tanımlama

```csharp
// Owned entity sınıfı
public class Address
{
    public string Street { get; set; }
    public string City { get; set; }
    public string PostalCode { get; set; }
    public string Country { get; set; }
}

// Ana entity sınıfı
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Owned entity
    public Address ShippingAddress { get; set; }
    public Address BillingAddress { get; set; }
}
```

### DbContext Yapılandırması

```csharp
// Owned entity yapılandırması
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Customer>()
        .OwnsOne(c => c.ShippingAddress, sa =>
        {
            // Özellik adlarını özelleştirme
            sa.Property(a => a.Street).HasColumnName("ShippingStreet");
            sa.Property(a => a.City).HasColumnName("ShippingCity");
            sa.Property(a => a.PostalCode).HasColumnName("ShippingPostalCode");
            sa.Property(a => a.Country).HasColumnName("ShippingCountry");
        });
        
    modelBuilder.Entity<Customer>()
        .OwnsOne(c => c.BillingAddress, ba =>
        {
            // Özellik adlarını özelleştirme
            ba.Property(a => a.Street).HasColumnName("BillingStreet");
            ba.Property(a => a.City).HasColumnName("BillingCity");
            ba.Property(a => a.PostalCode).HasColumnName("BillingPostalCode");
            ba.Property(a => a.Country).HasColumnName("BillingCountry");
        });
}
```

### İç İçe Owned Entity'ler

```csharp
// İç içe owned entity'ler
public class ContactInfo
{
    public string Email { get; set; }
    public string Phone { get; set; }
    public Address Address { get; set; }
}

public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public ContactInfo Contact { get; set; }
}

// Yapılandırma
modelBuilder.Entity<Customer>()
    .OwnsOne(c => c.Contact, ci =>
    {
        ci.Property(c => c.Email).IsRequired();
        ci.Property(c => c.Phone);
        
        // İç içe owned entity
        ci.OwnsOne(c => c.Address);
    });
```

### Owned Entity Koleksiyonları

```csharp
// Owned entity koleksiyonu
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public List<Address> Addresses { get; set; }
}

// Yapılandırma
modelBuilder.Entity<Customer>()
    .OwnsMany(c => c.Addresses, a =>
    {
        a.WithOwner().HasForeignKey("CustomerId");
        a.Property<int>("Id").ValueGeneratedOnAdd();
        a.HasKey("Id");
    });
```

## Table Splitting

Table splitting, birden fazla entity tipini tek bir veritabanı tablosuna eşlemenize olanak tanır. Bu, ilişkili entity'leri birleştirerek veritabanı şemanızı basitleştirmenize yardımcı olabilir.

### Table Splitting Tanımlama

```csharp
// Entity sınıfları
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public string CustomerName { get; set; }
    
    public OrderDetails Details { get; set; }
}

public class OrderDetails
{
    public int Id { get; set; }  // Order.Id ile aynı değere sahip olmalı
    public string ShippingAddress { get; set; }
    public decimal TotalAmount { get; set; }
    public string Notes { get; set; }
    
    public Order Order { get; set; }
}

// Yapılandırma
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Order>(order =>
    {
        order.ToTable("Orders");
        order.HasKey(o => o.Id);
        
        order.HasOne(o => o.Details)
            .WithOne(d => d.Order)
            .HasForeignKey<OrderDetails>(d => d.Id);
    });
    
    modelBuilder.Entity<OrderDetails>(details =>
    {
        details.ToTable("Orders");  // Aynı tablo adı
        details.HasKey(d => d.Id);
    });
}
```

### Table Splitting Kullanımı

```csharp
// Table splitting ile veri ekleme
var order = new Order
{
    OrderDate = DateTime.Now,
    CustomerName = "Ahmet Yılmaz",
    Details = new OrderDetails
    {
        ShippingAddress = "İstanbul, Türkiye",
        TotalAmount = 1250.50m,
        Notes = "Lütfen dikkatli paketleyin"
    }
};

context.Orders.Add(order);
await context.SaveChangesAsync();

// Table splitting ile veri sorgulama
var ordersWithDetails = await context.Orders
    .Include(o => o.Details)
    .ToListAsync();
```

## Entity State Management

Entity Framework Core, entity'lerin durumunu (state) izler ve bu durumlara göre veritabanı işlemlerini gerçekleştirir. Entity durumlarını manuel olarak yönetmek, karmaşık senaryolarda daha fazla kontrol sağlar.

### Entity Durumları

```csharp
// Entity durumları
public enum EntityState
{
    Detached,   // Context tarafından izlenmiyor
    Unchanged,  // Yüklenmiş ve değişmemiş
    Added,      // Yeni eklenmiş, henüz veritabanında yok
    Modified,   // Değiştirilmiş, güncelleme gerekiyor
    Deleted     // Silinmek üzere işaretlenmiş
}
```

### Entity Durumunu Kontrol Etme

```csharp
// Entity durumunu kontrol etme
var customer = await context.Customers.FindAsync(1);
var state = context.Entry(customer).State;  // Unchanged

customer.Name = "Yeni İsim";
state = context.Entry(customer).State;  // Modified
```

### Entity Durumunu Manuel Olarak Ayarlama

```csharp
// Entity durumunu manuel olarak ayarlama
var newCustomer = new Customer { Name = "Yeni Müşteri" };
context.Entry(newCustomer).State = EntityState.Added;

var existingCustomer = new Customer { Id = 1, Name = "Güncellenmiş İsim" };
context.Entry(existingCustomer).State = EntityState.Modified;

var customerToDelete = new Customer { Id = 2 };
context.Entry(customerToDelete).State = EntityState.Deleted;

await context.SaveChangesAsync();
```

### Belirli Özellikleri İşaretleme

```csharp
// Sadece belirli özellikleri değişmiş olarak işaretleme
var customer = new Customer { Id = 1, Name = "Yeni İsim", Email = "yeni@email.com" };

var entry = context.Entry(customer);
entry.State = EntityState.Unchanged;  // Başlangıçta unchanged olarak ayarla

// Sadece Name özelliğini modified olarak işaretle
entry.Property(c => c.Name).IsModified = true;

await context.SaveChangesAsync();  // Sadece Name güncellenir
```

### Toplu Durum Değişiklikleri

```csharp
// Toplu durum değişiklikleri
var customers = GetCustomersFromExternalSource();  // Dış kaynaktan müşteri listesi

foreach (var customer in customers)
{
    var entry = context.Entry(customer);
    
    if (customer.Id == 0)
    {
        entry.State = EntityState.Added;
    }
    else
    {
        entry.State = EntityState.Modified;
    }
}

await context.SaveChangesAsync();
```

### Değişiklikleri İzleme

```csharp
// Değişiklikleri izleme
var changes = context.ChangeTracker.Entries()
    .Where(e => e.State != EntityState.Unchanged)
    .ToList();
    
foreach (var change in changes)
{
    Console.WriteLine($"Entity: {change.Entity.GetType().Name}, State: {change.State}");
    
    if (change.State == EntityState.Modified)
    {
        foreach (var property in change.Properties)
        {
            if (property.IsModified)
            {
                Console.WriteLine($"  Property: {property.Metadata.Name}");
                Console.WriteLine($"  Original: {property.OriginalValue}");
                Console.WriteLine($"  Current: {property.CurrentValue}");
            }
        }
    }
}
```

### No-Tracking Sorgular

```csharp
// No-tracking sorgu
var customers = await context.Customers
    .AsNoTracking()
    .ToListAsync();
    
// Bu müşteriler context tarafından izlenmez
var state = context.Entry(customers[0]).State;  // Detached
```

## Pratik Örnekler

### Audit Log Uygulaması

```csharp
// Audit log için DbContext
public class AuditableDbContext : DbContext
{
    private readonly string _currentUser;
    
    public AuditableDbContext(DbContextOptions options, ICurrentUserService userService)
        : base(options)
    {
        _currentUser = userService.GetCurrentUser();
    }
    
    public override int SaveChanges()
    {
        var auditEntries = new List<AuditEntry>();
        
        // Değişiklikleri topla
        foreach (var entry in ChangeTracker.Entries())
        {
            if (entry.Entity is IAuditable && entry.State is EntityState.Added or EntityState.Modified or EntityState.Deleted)
            {
                var auditEntry = new AuditEntry
                {
                    EntityType = entry.Entity.GetType().Name,
                    EntityId = entry.Properties.Single(p => p.Metadata.IsPrimaryKey()).CurrentValue.ToString(),
                    Action = entry.State.ToString(),
                    Timestamp = DateTime.Now,
                    Username = _currentUser
                };
                
                // Değişen özellikleri kaydet
                if (entry.State == EntityState.Modified)
                {
                    foreach (var property in entry.Properties.Where(p => p.IsModified && !p.Metadata.IsPrimaryKey()))
                    {
                        auditEntry.Changes.Add(new PropertyChange
                        {
                            PropertyName = property.Metadata.Name,
                            OldValue = property.OriginalValue?.ToString(),
                            NewValue = property.CurrentValue?.ToString()
                        });
                    }
                }
                
                auditEntries.Add(auditEntry);
            }
        }
        
        // Önce normal değişiklikleri kaydet
        var result = base.SaveChanges();
        
        // Sonra audit log'ları kaydet
        if (auditEntries.Any())
        {
            AuditLogs.AddRange(auditEntries);
            base.SaveChanges();
        }
        
        return result;
    }
    
    public DbSet<AuditEntry> AuditLogs { get; set; }
}
```

### Multi-Tenant Uygulama

```csharp
// Multi-tenant DbContext
public class MultiTenantDbContext : DbContext
{
    private readonly Guid _tenantId;
    
    public MultiTenantDbContext(DbContextOptions options, ITenantService tenantService)
        : base(options)
    {
        _tenantId = tenantService.GetCurrentTenant();
    }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Tüm tenant-specific entity'lere filtre uygula
        foreach (var entityType in modelBuilder.Model.GetEntityTypes())
        {
            if (typeof(ITenantEntity).IsAssignableFrom(entityType.ClrType))
            {
                var parameter = Expression.Parameter(entityType.ClrType, "e");
                var property = Expression.Property(parameter, "TenantId");
                var tenantId = Expression.Constant(_tenantId);
                var filter = Expression.Lambda(Expression.Equal(property, tenantId), parameter);
                
                entityType.SetQueryFilter(filter);
            }
        }
    }
    
    // Tenant ID'yi otomatik ayarla
    public override int SaveChanges()
    {
        foreach (var entry in ChangeTracker.Entries<ITenantEntity>())
        {
            if (entry.State == EntityState.Added)
            {
                entry.Entity.TenantId = _tenantId;
            }
            else if (entry.State == EntityState.Modified)
            {
                entry.Property(x => x.TenantId).IsModified = false;
            }
        }
        
        return base.SaveChanges();
    }
}
```

### Soft Delete Uygulaması

```csharp
// Soft delete DbContext
public class SoftDeleteDbContext : DbContext
{
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Tüm soft-delete entity'lere filtre uygula
        foreach (var entityType in modelBuilder.Model.GetEntityTypes())
        {
            if (typeof(ISoftDelete).IsAssignableFrom(entityType.ClrType))
            {
                var parameter = Expression.Parameter(entityType.ClrType, "e");
                var property = Expression.Property(parameter, "IsDeleted");
                var filter = Expression.Lambda(Expression.Equal(property, Expression.Constant(false)), parameter);
                
                entityType.SetQueryFilter(filter);
            }
        }
    }
    
    // Silme işlemini soft delete olarak gerçekleştir
    public override int SaveChanges()
    {
        foreach (var entry in ChangeTracker.Entries<ISoftDelete>())
        {
            if (entry.State == EntityState.Deleted)
            {
                entry.State = EntityState.Modified;
                entry.Entity.IsDeleted = true;
                entry.Entity.DeletedAt = DateTime.Now;
            }
        }
        
        return base.SaveChanges();
    }
}
```

## Özet

Bu bölümde, Entity Framework Core'un ileri düzey özelliklerini inceledik:

- **Global Query Filters**: Otomatik olarak uygulanan sorgu filtreleri
- **Shadow Properties**: Entity sınıflarında tanımlanmayan veritabanı sütunları
- **Owned Entity Types**: Başka bir entity'nin parçası olan kompleks tipler
- **Table Splitting**: Birden fazla entity'yi tek bir tabloya eşleme
- **Entity State Management**: Entity durumlarını manuel olarak yönetme

Bu ileri düzey özellikler, Entity Framework Core ile daha karmaşık ve özelleştirilmiş veritabanı uygulamaları geliştirmenize olanak tanır. Doğru kullanıldığında, bu özellikler kodunuzu daha temiz, daha bakımı kolay ve daha performanslı hale getirebilir. 