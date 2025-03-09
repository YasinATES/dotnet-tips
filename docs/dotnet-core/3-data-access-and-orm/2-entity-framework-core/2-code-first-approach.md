# Code-First Yaklaşımı

Code-First yaklaşımı, veritabanı şemasını C# sınıfları üzerinden tanımlamanıza olanak tanıyan bir Entity Framework Core geliştirme modelidir. Bu yaklaşımda, önce model sınıflarınızı oluşturur, ardından EF Core bu sınıfları kullanarak veritabanı şemasını oluşturur.

## Model Sınıfları Oluşturma

Code-First yaklaşımında, veritabanı tablolarını temsil eden POCO (Plain Old CLR Object) sınıfları oluşturarak başlarsınız.

### Temel Model Sınıfı

```csharp
// Basic entity class
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public bool IsAvailable { get; set; }
    public DateTime CreatedDate { get; set; }
}
```

### İlişkili Model Sınıfları

```csharp
// Related entity classes
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    
    // Navigation property for one-to-many relationship
    public List<Product> Products { get; set; }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    
    // Foreign key property
    public int CategoryId { get; set; }
    
    // Navigation property for many-to-one relationship
    public Category Category { get; set; }
    
    // Navigation property for many-to-many relationship
    public List<ProductTag> ProductTags { get; set; }
}

public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property for many-to-many relationship
    public List<ProductTag> ProductTags { get; set; }
}

// Join entity for many-to-many relationship
public class ProductTag
{
    public int ProductId { get; set; }
    public Product Product { get; set; }
    
    public int TagId { get; set; }
    public Tag Tag { get; set; }
}
```

### DbContext Sınıfı

Model sınıflarınızı tanımladıktan sonra, bunları DbContext sınıfınıza eklersiniz:

```csharp
// DbContext with entity sets
public class ShopContext : DbContext
{
    public ShopContext(DbContextOptions<ShopContext> options)
        : base(options)
    {
    }
    
    public DbSet<Product> Products { get; set; }
    public DbSet<Category> Categories { get; set; }
    public DbSet<Tag> Tags { get; set; }
    public DbSet<ProductTag> ProductTags { get; set; }
    
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Configure entities using Fluent API
        // (will be covered later in this document)
    }
}
```

## Data Annotations Kullanımı

Data Annotations, model sınıflarınızı yapılandırmak için kullanabileceğiniz özniteliklerdir. Bu öznitelikler, sütun adları, veri tipleri, zorunlu alanlar ve diğer kısıtlamalar gibi meta verileri tanımlamanıza olanak tanır.

### Temel Data Annotations

```csharp
// Using Data Annotations
public class Product
{
    [Key] // Primary key
    public int Id { get; set; }
    
    [Required] // NOT NULL constraint
    [StringLength(100)] // VARCHAR(100)
    public string Name { get; set; }
    
    [Column("UnitPrice")] // Custom column name
    [Range(0, 1000)] // Check constraint
    [Precision(10, 2)] // DECIMAL(10,2)
    public decimal Price { get; set; }
    
    [Required]
    [DefaultValue(true)]
    public bool IsAvailable { get; set; }
    
    [DataType(DataType.Date)]
    [DisplayFormat(DataFormatString = "{0:yyyy-MM-dd}")]
    public DateTime CreatedDate { get; set; }
    
    [NotMapped] // Exclude from database mapping
    public decimal PriceWithTax => Price * 1.18m;
}
```

### İlişki Data Annotations

```csharp
// Relationship Data Annotations
public class Product
{
    [Key]
    public int Id { get; set; }
    
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [ForeignKey("Category")]
    public int CategoryId { get; set; }
    
    [Required]
    public Category Category { get; set; }
}

public class Category
{
    public int Id { get; set; }
    
    [Required]
    [StringLength(50)]
    public string Name { get; set; }
    
    [InverseProperty("Category")]
    public List<Product> Products { get; set; }
}
```

### Tablo ve İndeks Annotations

```csharp
// Table and Index Annotations
[Table("Products", Schema = "inventory")]
[Index(nameof(Name), IsUnique = true)]
public class Product
{
    [Key]
    public int Id { get; set; }
    
    [Required]
    [StringLength(100)]
    public string Name { get; set; }
    
    [Column("UnitPrice")]
    public decimal Price { get; set; }
    
    [Required]
    public bool IsAvailable { get; set; }
}
```

## Fluent API ile Model Yapılandırma

Fluent API, Data Annotations'a göre daha güçlü ve esnek bir yapılandırma yöntemidir. DbContext sınıfındaki OnModelCreating metodunda kullanılır.

### Temel Fluent API Yapılandırması

```csharp
// Basic Fluent API configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Product>(entity =>
    {
        // Table configuration
        entity.ToTable("Products", "inventory");
        
        // Primary key
        entity.HasKey(e => e.Id);
        
        // Auto-increment
        entity.Property(e => e.Id)
              .UseIdentityColumn();
        
        // Property configuration
        entity.Property(e => e.Name)
              .IsRequired()
              .HasMaxLength(100);
              
        entity.Property(e => e.Price)
              .HasColumnName("UnitPrice")
              .HasPrecision(10, 2);
              
        entity.Property(e => e.IsAvailable)
              .IsRequired()
              .HasDefaultValue(true);
              
        // Ignore property
        entity.Ignore(e => e.PriceWithTax);
        
        // Index
        entity.HasIndex(e => e.Name)
              .IsUnique();
    });
}
```

### İlişki Yapılandırması

```csharp
// Relationship configuration with Fluent API
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // One-to-Many relationship
    modelBuilder.Entity<Product>()
        .HasOne(p => p.Category)
        .WithMany(c => c.Products)
        .HasForeignKey(p => p.CategoryId)
        .OnDelete(DeleteBehavior.Restrict);
    
    // Many-to-Many relationship
    modelBuilder.Entity<ProductTag>()
        .HasKey(pt => new { pt.ProductId, pt.TagId });
        
    modelBuilder.Entity<ProductTag>()
        .HasOne(pt => pt.Product)
        .WithMany(p => p.ProductTags)
        .HasForeignKey(pt => pt.ProductId);
        
    modelBuilder.Entity<ProductTag>()
        .HasOne(pt => pt.Tag)
        .WithMany(t => t.ProductTags)
        .HasForeignKey(pt => pt.TagId);
}
```

### Seed Data Yapılandırması

```csharp
// Seed data configuration
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    // Other configurations...
    
    // Seed data
    modelBuilder.Entity<Category>().HasData(
        new Category { Id = 1, Name = "Elektronik", Description = "Elektronik ürünler" },
        new Category { Id = 2, Name = "Giyim", Description = "Giyim ürünleri" }
    );
    
    modelBuilder.Entity<Product>().HasData(
        new Product { Id = 1, Name = "Laptop", Price = 5000m, CategoryId = 1, IsAvailable = true },
        new Product { Id = 2, Name = "Telefon", Price = 3000m, CategoryId = 1, IsAvailable = true },
        new Product { Id = 3, Name = "T-Shirt", Price = 100m, CategoryId = 2, IsAvailable = true }
    );
}
```

## Relationships (İlişkiler)

EF Core, veritabanındaki tablolar arasındaki ilişkileri modellemek için çeşitli ilişki türlerini destekler.

### One-to-One (Bire-Bir) İlişki

Bir varlığın başka bir varlıkla bire-bir ilişkisi olduğunda kullanılır.

```csharp
// One-to-One relationship
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property
    public CustomerAddress Address { get; set; }
}

public class CustomerAddress
{
    public int Id { get; set; }
    public string Street { get; set; }
    public string City { get; set; }
    
    // Foreign key
    public int CustomerId { get; set; }
    
    // Navigation property
    public Customer Customer { get; set; }
}

// Configuration in OnModelCreating
modelBuilder.Entity<Customer>()
    .HasOne(c => c.Address)
    .WithOne(a => a.Customer)
    .HasForeignKey<CustomerAddress>(a => a.CustomerId);
```

### One-to-Many (Bire-Çok) İlişki

Bir varlığın başka bir varlıkla bire-çok ilişkisi olduğunda kullanılır.

```csharp
// One-to-Many relationship
public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property
    public List<Product> Products { get; set; }
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Foreign key
    public int CategoryId { get; set; }
    
    // Navigation property
    public Category Category { get; set; }
}

// Configuration in OnModelCreating
modelBuilder.Entity<Product>()
    .HasOne(p => p.Category)
    .WithMany(c => c.Products)
    .HasForeignKey(p => p.CategoryId);
```

### Many-to-Many (Çoka-Çok) İlişki

İki varlık arasında çoka-çok ilişki olduğunda kullanılır.

#### Açık Join Entity ile

```csharp
// Many-to-Many relationship with explicit join entity
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property
    public List<ProductTag> ProductTags { get; set; }
}

public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property
    public List<ProductTag> ProductTags { get; set; }
}

public class ProductTag
{
    public int ProductId { get; set; }
    public Product Product { get; set; }
    
    public int TagId { get; set; }
    public Tag Tag { get; set; }
}

// Configuration in OnModelCreating
modelBuilder.Entity<ProductTag>()
    .HasKey(pt => new { pt.ProductId, pt.TagId });
    
modelBuilder.Entity<ProductTag>()
    .HasOne(pt => pt.Product)
    .WithMany(p => p.ProductTags)
    .HasForeignKey(pt => pt.ProductId);
    
modelBuilder.Entity<ProductTag>()
    .HasOne(pt => pt.Tag)
    .WithMany(t => t.ProductTags)
    .HasForeignKey(pt => pt.TagId);
```

#### Örtük Join Entity ile

```csharp
// Many-to-Many relationship with implicit join entity
public class Student
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property
    public List<Course> Courses { get; set; }
}

public class Course
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Navigation property
    public List<Student> Students { get; set; }
}

// Configuration in OnModelCreating
modelBuilder.Entity<Student>()
    .HasMany(s => s.Courses)
    .WithMany(c => c.Students)
    .UsingEntity(j => j.ToTable("StudentCourses"));
```

## Inheritance Mapping (Kalıtım Eşleştirme)

EF Core, nesne yönelimli programlamadaki kalıtım ilişkilerini veritabanına eşleştirmek için üç farklı strateji sunar.

### Table-per-Hierarchy (TPH)

TPH stratejisinde, tüm kalıtım hiyerarşisi tek bir tabloda saklanır ve bir ayrıştırıcı sütun kullanılarak hangi türe ait olduğu belirlenir.

```csharp
// Table-per-Hierarchy (TPH) inheritance
public abstract class Payment
{
    public int Id { get; set; }
    public decimal Amount { get; set; }
    public DateTime PaymentDate { get; set; }
}

public class CashPayment : Payment
{
    public string ReceivedBy { get; set; }
}

public class CreditCardPayment : Payment
{
    public string CardNumber { get; set; }
    public string CardHolderName { get; set; }
    public string ExpiryDate { get; set; }
}

// Configuration in OnModelCreating
modelBuilder.Entity<Payment>()
    .HasDiscriminator<string>("PaymentType")
    .HasValue<CashPayment>("Cash")
    .HasValue<CreditCardPayment>("CreditCard");
```

### Table-per-Type (TPT)

TPT stratejisinde, her türe ait özellikler ayrı tablolarda saklanır ve birincil anahtar ile ilişkilendirilir.

```csharp
// Table-per-Type (TPT) inheritance
public abstract class Vehicle
{
    public int Id { get; set; }
    public string Make { get; set; }
    public string Model { get; set; }
    public int Year { get; set; }
}

public class Car : Vehicle
{
    public int Doors { get; set; }
    public string BodyType { get; set; }
}

public class Motorcycle : Vehicle
{
    public int EngineCapacity { get; set; }
    public bool HasSideCar { get; set; }
}

// Configuration in OnModelCreating
modelBuilder.Entity<Vehicle>().ToTable("Vehicles");
modelBuilder.Entity<Car>().ToTable("Cars");
modelBuilder.Entity<Motorcycle>().ToTable("Motorcycles");
```

### Table-per-Concrete-Type (TPC)

TPC stratejisinde, her somut türe ait tüm özellikler (kalıtım alınan özellikler dahil) ayrı tablolarda saklanır.

```csharp
// Table-per-Concrete-Type (TPC) inheritance
[Table("People")]
public abstract class Person
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}

[Table("Employees")]
public class Employee : Person
{
    public string Department { get; set; }
    public decimal Salary { get; set; }
}

[Table("Customers")]
public class Customer : Person
{
    public string CompanyName { get; set; }
    public string Phone { get; set; }
}

// Configuration in OnModelCreating
modelBuilder.Entity<Employee>().ToTable("Employees");
modelBuilder.Entity<Customer>().ToTable("Customers");

// Explicitly map all properties from the base type
modelBuilder.Entity<Employee>(entity =>
{
    entity.Property(e => e.Id);
    entity.Property(e => e.Name);
    entity.Property(e => e.Email);
});

modelBuilder.Entity<Customer>(entity =>
{
    entity.Property(e => e.Id);
    entity.Property(e => e.Name);
    entity.Property(e => e.Email);
});
```

## Özet

Bu bölümde, Entity Framework Core'un Code-First yaklaşımını inceledik:

- **Model Sınıfları Oluşturma**: Veritabanı tablolarını temsil eden POCO sınıfları
- **Data Annotations**: Model sınıflarını özniteliklerle yapılandırma
- **Fluent API**: Daha güçlü ve esnek model yapılandırması
- **İlişkiler**: One-to-One, One-to-Many ve Many-to-Many ilişkileri
- **Kalıtım Eşleştirme**: TPH, TPT ve TPC stratejileri

Code-First yaklaşımı, veritabanı şemasını C# kodu üzerinden tanımlamanıza ve yönetmenize olanak tanır. Bu, veritabanı şemasını kaynak kontrolü altında tutmanızı, değişiklikleri izlemenizi ve uygulamanızı kolaylaştırır. 