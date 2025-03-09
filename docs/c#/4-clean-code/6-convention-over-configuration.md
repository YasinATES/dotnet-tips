# Convention over Configuration

Convention over Configuration (Konfigürasyon Yerine Konvansiyon) prensibi, yazılım çerçevelerinde yaygın olarak kullanılan ve açık yapılandırma yerine yaygın kabul görmüş kuralları tercih eden bir yaklaşımdır. Bu prensip, Ruby on Rails framework'ü ile popülerlik kazanmıştır.

## 1. Convention over Configuration'ın Temel Kavramı

Bu prensibin özü, geliştiricilerin sadece standart yaklaşımdan sapan durumları yapılandırması gerektiğidir. Eğer bir uygulama yaygın kabul görmüş kurallara uyuyorsa, minimum yapılandırma ile çalışabilmelidir.

### Konvansiyon İhlali Örneği

```csharp
// Konvansiyon ihlali - Aşırı yapılandırma
public class OrderMapping : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.ToTable("Orders");
        builder.HasKey(o => o.Id);
        builder.Property(o => o.Id).HasColumnName("Id");
        builder.Property(o => o.OrderDate).HasColumnName("OrderDate");
        builder.Property(o => o.CustomerId).HasColumnName("CustomerId");
        builder.Property(o => o.TotalAmount).HasColumnName("TotalAmount");
        
        builder.HasOne(o => o.Customer)
            .WithMany(c => c.Orders)
            .HasForeignKey(o => o.CustomerId)
            .HasConstraintName("FK_Orders_Customers");
    }
}
```

### Konvansiyon Uyumlu Tasarım

```csharp
// Konvansiyon uyumlu - Minimum yapılandırma
public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    public int CustomerId { get; set; }
    public decimal TotalAmount { get; set; }
    
    public Customer Customer { get; set; }
    public ICollection<OrderItem> Items { get; set; }
}

// Sadece standarttan sapan durumlar için yapılandırma
public class OrderMapping : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        // Özel durumlar için yapılandırma
        builder.Property(o => o.TotalAmount)
            .HasPrecision(18, 2);
    }
}
```

## 2. Yaygın Konvansiyonlar

### 1. İsimlendirme Konvansiyonları

```csharp
// Sınıf isimleri - PascalCase
public class CustomerService { }

// Metot isimleri - PascalCase
public void ProcessOrder() { }

// Değişken ve parametre isimleri - camelCase
private string firstName;
public void SetName(string newName) { }

// Interface isimleri - I prefix
public interface IRepository { }

// Özel durumlar için attribute
[Table("TBL_CUSTOMERS")]
public class Customer { }
```

### 2. Dosya Organizasyonu

```
/src
    /Domain
        /Entities
            Customer.cs
            Order.cs
        /Interfaces
            ICustomerRepository.cs
    /Infrastructure
        /Data
            ApplicationDbContext.cs
        /Repositories
            CustomerRepository.cs
    /Application
        /Services
            CustomerService.cs
        /DTOs
            CustomerDto.cs
    /Web
        /Controllers
            CustomerController.cs
        /Views
            /Customer
                Index.cshtml
```

## 3. En İyi Pratikler

1. **Varsayılan Davranışları Kullanın**
   - Framework'ün sunduğu varsayılan davranışları tercih edin
   - Sadece gerektiğinde özelleştirme yapın
   - Konvansiyonları dokümante edin

2. **Tutarlı Yapı Kullanın**
   - Proje genelinde tutarlı bir yapı benimseyin
   - Özel durumları açıkça belirtin
   - Takım içinde konvansiyonları standardize edin

3. **Otomatik Yapılandırma Kullanın**
   - Attribute'ları etkili kullanın
   - Fluent API'yi sadece gerektiğinde kullanın
   - Convention-based mapping tercih edin

4. **İstisnaları Yönetin**
   - Konvansiyonlardan sapmaları dokümante edin
   - Özel durumları açıkça gerekçelendirin
   - İstisnaları minimize edin

5. **Takım Standartları Oluşturun**
   - Proje konvansiyonlarını belirleyin
   - Style guide oluşturun
   - Code review sürecinde kontrol edin

## 4. Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Aşırı Yapılandırma**
   ```csharp
   // Kötü - gereksiz yapılandırma
   public class UserConfiguration : IEntityTypeConfiguration<User>
   {
       public void Configure(EntityTypeBuilder<User> builder)
       {
           builder.ToTable("Users");
           builder.HasKey(u => u.Id);
           builder.Property(u => u.Id).HasColumnName("Id");
           builder.Property(u => u.FirstName).HasColumnName("FirstName");
           builder.Property(u => u.LastName).HasColumnName("LastName");
           builder.Property(u => u.Email).HasColumnName("Email");
       }
   }
   
   // İyi - sadece gerekli yapılandırma
   public class UserConfiguration : IEntityTypeConfiguration<User>
   {
       public void Configure(EntityTypeBuilder<User> builder)
       {
           builder.Property(u => u.Email).IsRequired().HasMaxLength(256);
       }
   }
   ```

2. **Konvansiyon Dışı İsimlendirme**
   ```csharp
   // Kötü - tutarsız isimlendirme
   public class userService
   {
       private readonly IUserREPO _UserRepository;
       
       public user GetUserbyId(int ID)
       {
           return _UserRepository.GETBYID(ID);
       }
   }
   
   // İyi - konvansiyon uyumlu isimlendirme
   public class UserService
   {
       private readonly IUserRepository _userRepository;
       
       public User GetUserById(int id)
       {
           return _userRepository.GetById(id);
       }
   }
   ```

3. **Yapısal Tutarsızlık**
   ```csharp
   // Kötü - tutarsız yapı
   /Controllers
       HomeController.cs
       /Admin
           UserManagement.cs
       products.controller.cs
       /Security
           /Auth
               LoginController.cs
   
   // İyi - tutarlı yapı
   /Controllers
       /Admin
           UserManagementController.cs
       /Security
           AuthController.cs
       HomeController.cs
       ProductController.cs
   ```

Convention over Configuration prensibi, yazılım geliştirmede verimliliği artıran ve bakım maliyetlerini düşüren önemli bir yaklaşımdır. Bu prensibi doğru uygulayarak, daha tutarlı ve yönetilebilir sistemler geliştirebilirsiniz. 