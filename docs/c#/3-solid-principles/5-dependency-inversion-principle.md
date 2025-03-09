# Dependency Inversion Principle (DIP)

Dependency Inversion Principle (Bağımlılık Tersine Çevirme Prensibi), SOLID prensiplerinin beşinci ve son harfi olan "D"yi temsil eder. Bu prensip, Robert C. Martin tarafından tanımlanmış ve iki temel ifadeyle açıklanmıştır:

1. Yüksek seviyeli modüller, düşük seviyeli modüllere bağımlı olmamalıdır. Her ikisi de soyutlamalara bağımlı olmalıdır.
2. Soyutlamalar detaylara bağımlı olmamalıdır. Detaylar soyutlamalara bağımlı olmalıdır.

## 1. DIP'nin Temel Kavramı

DIP'nin özü, geleneksel bağımlılık hiyerarşisini tersine çevirmektir. Geleneksel yaklaşımda, yüksek seviyeli modüller (politika, iş mantığı) düşük seviyeli modüllere (veritabanı, dosya sistemi, ağ) doğrudan bağımlıdır. DIP ile bu hiyerarşi tersine çevrilir ve her iki seviye de soyutlamalara bağımlı hale gelir.

### DIP İhlali Örneği

```csharp
// DIP ihlali - Yüksek seviyeli sınıf, düşük seviyeli sınıfa doğrudan bağımlı
public class OrderService
{
    private readonly SqlOrderRepository _orderRepository;
    private readonly EmailNotificationService _notificationService;
    
    public OrderService()
    {
        _orderRepository = new SqlOrderRepository();
        _notificationService = new EmailNotificationService();
    }
    
    public void PlaceOrder(Order order)
    {
        // Sipariş işleme mantığı
        _orderRepository.Save(order);
        _notificationService.SendOrderConfirmation(order);
    }
}

// Düşük seviyeli sınıflar
public class SqlOrderRepository
{
    public void Save(Order order)
    {
        // SQL veritabanına sipariş kaydetme kodu
        Console.WriteLine($"Sipariş #{order.Id} SQL veritabanına kaydedildi.");
    }
}

public class EmailNotificationService
{
    public void SendOrderConfirmation(Order order)
    {
        // E-posta gönderme kodu
        Console.WriteLine($"Sipariş #{order.Id} için e-posta bildirimi gönderildi.");
    }
}
```

Bu örnekte, `OrderService` sınıfı doğrudan `SqlOrderRepository` ve `EmailNotificationService` sınıflarına bağımlıdır. Bu, DIP'yi ihlal eder çünkü:

1. Yüksek seviyeli `OrderService` sınıfı, düşük seviyeli `SqlOrderRepository` ve `EmailNotificationService` sınıflarına doğrudan bağımlıdır.
2. Bağımlılıklar somut sınıflar üzerinden oluşturulmuştur, soyutlamalar üzerinden değil.
3. `OrderService` sınıfı, kendi bağımlılıklarını oluşturmaktan sorumludur (new anahtar kelimesi kullanılarak).

### DIP Uyumlu Tasarım

```csharp
// DIP uyumlu tasarım - Soyutlamalar ve dependency injection kullanımı
public interface IOrderRepository
{
    void Save(Order order);
}

public interface INotificationService
{
    void SendOrderConfirmation(Order order);
}

// Yüksek seviyeli sınıf, soyutlamalara bağımlı
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly INotificationService _notificationService;
    
    // Dependency injection ile bağımlılıkları dışarıdan alıyoruz
    public OrderService(IOrderRepository orderRepository, INotificationService notificationService)
    {
        _orderRepository = orderRepository;
        _notificationService = notificationService;
    }
    
    public void PlaceOrder(Order order)
    {
        // Sipariş işleme mantığı
        _orderRepository.Save(order);
        _notificationService.SendOrderConfirmation(order);
    }
}

// Düşük seviyeli sınıflar, soyutlamaları uygular
public class SqlOrderRepository : IOrderRepository
{
    public void Save(Order order)
    {
        // SQL veritabanına sipariş kaydetme kodu
        Console.WriteLine($"Sipariş #{order.Id} SQL veritabanına kaydedildi.");
    }
}

public class MongoOrderRepository : IOrderRepository
{
    public void Save(Order order)
    {
        // MongoDB veritabanına sipariş kaydetme kodu
        Console.WriteLine($"Sipariş #{order.Id} MongoDB veritabanına kaydedildi.");
    }
}

public class EmailNotificationService : INotificationService
{
    public void SendOrderConfirmation(Order order)
    {
        // E-posta gönderme kodu
        Console.WriteLine($"Sipariş #{order.Id} için e-posta bildirimi gönderildi.");
    }
}

public class SmsNotificationService : INotificationService
{
    public void SendOrderConfirmation(Order order)
    {
        // SMS gönderme kodu
        Console.WriteLine($"Sipariş #{order.Id} için SMS bildirimi gönderildi.");
    }
}

// Kullanım örneği
public class Program
{
    public static void Main()
    {
        // Bağımlılıkları oluştur
        IOrderRepository orderRepository = new SqlOrderRepository();
        INotificationService notificationService = new EmailNotificationService();
        
        // OrderService'i oluştur ve bağımlılıkları enjekte et
        var orderService = new OrderService(orderRepository, notificationService);
        
        // Sipariş ver
        var order = new Order { Id = 1, CustomerName = "Ahmet Yılmaz", TotalAmount = 150.75m };
        orderService.PlaceOrder(order);
        
        // Farklı implementasyonlarla tekrar dene
        orderRepository = new MongoOrderRepository();
        notificationService = new SmsNotificationService();
        
        orderService = new OrderService(orderRepository, notificationService);
        orderService.PlaceOrder(order);
    }
}
```

Bu tasarımda, DIP uygulanmıştır çünkü:

1. Yüksek seviyeli `OrderService` sınıfı, düşük seviyeli sınıflara değil, soyutlamalara (`IOrderRepository` ve `INotificationService`) bağımlıdır.
2. Düşük seviyeli sınıflar (`SqlOrderRepository`, `MongoOrderRepository`, `EmailNotificationService`, `SmsNotificationService`), soyutlamaları uygular.
3. Bağımlılıklar, constructor injection yöntemiyle dışarıdan enjekte edilir.

## 2. Dependency Injection ve Inversion of Control (IoC)

DIP'yi uygulamanın en yaygın yolu, Dependency Injection (DI) ve Inversion of Control (IoC) kullanmaktır.

### Dependency Injection (Bağımlılık Enjeksiyonu)

Dependency Injection, bir sınıfın bağımlılıklarını dışarıdan almasını sağlayan bir tekniktir. Üç ana DI yöntemi vardır:

1. **Constructor Injection**: Bağımlılıklar, sınıfın constructor'ı aracılığıyla enjekte edilir. Bu, en yaygın ve önerilen yöntemdir.

```csharp
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    
    // Constructor injection
    public OrderService(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }
}
```

2. **Property Injection**: Bağımlılıklar, sınıfın public property'leri aracılığıyla enjekte edilir.

```csharp
public class OrderService
{
    // Property injection
    public IOrderRepository OrderRepository { get; set; }
}
```

3. **Method Injection**: Bağımlılıklar, sınıfın metotları aracılığıyla enjekte edilir.

```csharp
public class OrderService
{
    public void PlaceOrder(Order order, IOrderRepository orderRepository)
    {
        // Method injection
        orderRepository.Save(order);
    }
}
```

### Inversion of Control (IoC) Containers

IoC Container'ları, bağımlılıkların oluşturulması, yönetilmesi ve enjekte edilmesi işlemlerini otomatikleştiren araçlardır. .NET ekosisteminde popüler IoC Container'ları şunlardır:

- Microsoft.Extensions.DependencyInjection (ASP.NET Core'un yerleşik DI container'ı)
- Autofac
- Ninject
- Castle Windsor
- StructureMap
- Unity

IoC Container'ları, bağımlılıkları kaydetmenize, ömürlerini (lifetime) yönetmenize ve gerektiğinde çözümlemenize (resolve) olanak tanır.

```csharp
// Microsoft.Extensions.DependencyInjection kullanımı
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Bağımlılıkları kaydet
        services.AddScoped<IOrderRepository, SqlOrderRepository>();
        services.AddScoped<INotificationService, EmailNotificationService>();
        services.AddScoped<OrderService>();
    }
}

// Autofac kullanımı
public class AutofacModule : Module
{
    protected override void Load(ContainerBuilder builder)
    {
        // Bağımlılıkları kaydet
        builder.RegisterType<SqlOrderRepository>().As<IOrderRepository>().InstancePerLifetimeScope();
        builder.RegisterType<EmailNotificationService>().As<INotificationService>().InstancePerLifetimeScope();
        builder.RegisterType<OrderService>();
    }
}
```

### Servis Ömürleri (Service Lifetimes)

IoC Container'ları, servislerin ömürlerini yönetmenize olanak tanır. Yaygın servis ömürleri şunlardır:

1. **Transient**: Her istek için yeni bir örnek oluşturulur.
2. **Scoped**: Her scope (örneğin, HTTP isteği) için bir örnek oluşturulur.
3. **Singleton**: Uygulama ömrü boyunca tek bir örnek kullanılır.

```csharp
// Microsoft.Extensions.DependencyInjection ile servis ömürleri
services.AddTransient<ILogger, FileLogger>(); // Her istek için yeni örnek
services.AddScoped<IOrderRepository, SqlOrderRepository>(); // Her scope için bir örnek
services.AddSingleton<ICacheManager, MemoryCacheManager>(); // Tek örnek
```

## 3. Gerçek Hayat Örneği: E-Ticaret Uygulaması

Aşağıda, DIP'nin bir e-ticaret uygulamasında nasıl uygulanabileceğine dair kapsamlı bir örnek verilmiştir.

```csharp
// Domain modelleri
public class Order
{
    public int Id { get; set; }
    public string CustomerName { get; set; }
    public string CustomerEmail { get; set; }
    public List<OrderItem> Items { get; set; }
    public decimal TotalAmount => Items?.Sum(i => i.UnitPrice * i.Quantity) ?? 0;
}

public class OrderItem
{
    public int ProductId { get; set; }
    public string ProductName { get; set; }
    public int Quantity { get; set; }
    public decimal UnitPrice { get; set; }
}

// Soyutlamalar (Abstractions)
public interface IOrderRepository
{
    void Save(Order order);
    Order GetById(int id);
    IEnumerable<Order> GetByCustomerEmail(string email);
}

public interface INotificationService
{
    void SendOrderConfirmation(Order order);
}

public interface IInventoryService
{
    bool CheckStock(int productId, int quantity);
    void UpdateStock(int productId, int quantity);
}

public interface IPaymentService
{
    bool ProcessPayment(decimal amount, string creditCardNumber, string expiryDate, string cvv);
    void RefundPayment(string transactionId, decimal amount);
}

// Yüksek seviyeli servis (High-level service)
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly INotificationService _notificationService;
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentService _paymentService;
    
    public OrderService(
        IOrderRepository orderRepository,
        INotificationService notificationService,
        IInventoryService inventoryService,
        IPaymentService paymentService)
    {
        _orderRepository = orderRepository;
        _notificationService = notificationService;
        _inventoryService = inventoryService;
        _paymentService = paymentService;
    }
    
    public bool PlaceOrder(Order order, string creditCardNumber, string expiryDate, string cvv)
    {
        // Stok kontrolü
        foreach (var item in order.Items)
        {
            if (!_inventoryService.CheckStock(item.ProductId, item.Quantity))
            {
                Console.WriteLine($"Ürün #{item.ProductId} için yeterli stok yok.");
                return false;
            }
        }
        
        // Ödeme işlemi
        bool paymentSuccess = _paymentService.ProcessPayment(order.TotalAmount, creditCardNumber, expiryDate, cvv);
        if (!paymentSuccess)
        {
            Console.WriteLine("Ödeme işlemi başarısız.");
            return false;
        }
        
        // Stok güncelleme
        foreach (var item in order.Items)
        {
            _inventoryService.UpdateStock(item.ProductId, -item.Quantity);
        }
        
        // Sipariş kaydetme
        _orderRepository.Save(order);
        
        // Bildirim gönderme
        _notificationService.SendOrderConfirmation(order);
        
        Console.WriteLine($"Sipariş #{order.Id} başarıyla tamamlandı.");
        return true;
    }
}

// Düşük seviyeli implementasyonlar (Low-level implementations)
public class SqlOrderRepository : IOrderRepository
{
    public void Save(Order order)
    {
        // SQL veritabanına sipariş kaydetme kodu
        Console.WriteLine($"Sipariş #{order.Id} SQL veritabanına kaydedildi.");
    }
    
    public Order GetById(int id)
    {
        // SQL veritabanından sipariş getirme kodu
        Console.WriteLine($"Sipariş #{id} SQL veritabanından getirildi.");
        return new Order { Id = id };
    }
    
    public IEnumerable<Order> GetByCustomerEmail(string email)
    {
        // SQL veritabanından müşteri siparişlerini getirme kodu
        Console.WriteLine($"{email} müşterisinin siparişleri SQL veritabanından getirildi.");
        return new List<Order>();
    }
}

public class MongoOrderRepository : IOrderRepository
{
    public void Save(Order order)
    {
        // MongoDB veritabanına sipariş kaydetme kodu
        Console.WriteLine($"Sipariş #{order.Id} MongoDB veritabanına kaydedildi.");
    }
    
    public Order GetById(int id)
    {
        // MongoDB veritabanından sipariş getirme kodu
        Console.WriteLine($"Sipariş #{id} MongoDB veritabanından getirildi.");
        return new Order { Id = id };
    }
    
    public IEnumerable<Order> GetByCustomerEmail(string email)
    {
        // MongoDB veritabanından müşteri siparişlerini getirme kodu
        Console.WriteLine($"{email} müşterisinin siparişleri MongoDB veritabanından getirildi.");
        return new List<Order>();
    }
}

public class EmailNotificationService : INotificationService
{
    public void SendOrderConfirmation(Order order)
    {
        // E-posta gönderme kodu
        Console.WriteLine($"Sipariş #{order.Id} için {order.CustomerEmail} adresine e-posta bildirimi gönderildi.");
    }
}

public class SmsNotificationService : INotificationService
{
    public void SendOrderConfirmation(Order order)
    {
        // SMS gönderme kodu
        Console.WriteLine($"Sipariş #{order.Id} için SMS bildirimi gönderildi.");
    }
}

public class WarehouseInventoryService : IInventoryService
{
    public bool CheckStock(int productId, int quantity)
    {
        // Depo stok kontrolü kodu
        Console.WriteLine($"Ürün #{productId} için stok kontrolü yapıldı.");
        return true; // Örnek için her zaman true döndürüyoruz
    }
    
    public void UpdateStock(int productId, int quantity)
    {
        // Depo stok güncelleme kodu
        Console.WriteLine($"Ürün #{productId} stok miktarı {quantity} adet güncellendi.");
    }
}

public class StripePaymentService : IPaymentService
{
    public bool ProcessPayment(decimal amount, string creditCardNumber, string expiryDate, string cvv)
    {
        // Stripe ile ödeme işleme kodu
        Console.WriteLine($"Stripe ile {amount:C} tutarında ödeme işlendi.");
        return true; // Örnek için her zaman true döndürüyoruz
    }
    
    public void RefundPayment(string transactionId, decimal amount)
    {
        // Stripe ile iade işleme kodu
        Console.WriteLine($"Stripe ile {amount:C} tutarında iade işlendi.");
    }
}

public class PayPalPaymentService : IPaymentService
{
    public bool ProcessPayment(decimal amount, string creditCardNumber, string expiryDate, string cvv)
    {
        // PayPal ile ödeme işleme kodu
        Console.WriteLine($"PayPal ile {amount:C} tutarında ödeme işlendi.");
        return true; // Örnek için her zaman true döndürüyoruz
    }
    
    public void RefundPayment(string transactionId, decimal amount)
    {
        // PayPal ile iade işleme kodu
        Console.WriteLine($"PayPal ile {amount:C} tutarında iade işlendi.");
    }
}

// IoC Container kullanımı
public class Program
{
    public static void Main()
    {
        // Servis koleksiyonu oluştur
        var services = new ServiceCollection();
        
        // Bağımlılıkları kaydet
        services.AddScoped<IOrderRepository, SqlOrderRepository>();
        services.AddScoped<INotificationService, EmailNotificationService>();
        services.AddScoped<IInventoryService, WarehouseInventoryService>();
        services.AddScoped<IPaymentService, StripePaymentService>();
        services.AddScoped<OrderService>();
        
        // Service provider oluştur
        var serviceProvider = services.BuildServiceProvider();
        
        // OrderService'i al
        var orderService = serviceProvider.GetService<OrderService>();
        
        // Sipariş oluştur ve işle
        var order = new Order
        {
            Id = 1,
            CustomerName = "Ahmet Yılmaz",
            CustomerEmail = "ahmet@example.com",
            Items = new List<OrderItem>
            {
                new OrderItem { ProductId = 101, ProductName = "Laptop", Quantity = 1, UnitPrice = 5000m },
                new OrderItem { ProductId = 102, ProductName = "Mouse", Quantity = 2, UnitPrice = 100m }
            }
        };
        
        bool success = orderService.PlaceOrder(order, "1234-5678-9012-3456", "12/25", "123");
        Console.WriteLine($"Sipariş durumu: {(success ? "Başarılı" : "Başarısız")}");
    }
}
```

Bu örnekte, DIP tam olarak uygulanmıştır:

1. Yüksek seviyeli `OrderService` sınıfı, düşük seviyeli sınıflara değil, soyutlamalara bağımlıdır.
2. Tüm bağımlılıklar constructor injection ile enjekte edilir.
3. Farklı implementasyonlar (SQL, MongoDB, Email, SMS, Stripe, PayPal) kolayca değiştirilebilir.
4. IoC Container, bağımlılıkların yönetimini otomatikleştirir.

## 4. DIP ve Loosely Coupled Architecture

DIP, loosely coupled (gevşek bağlı) mimari oluşturmanın temel bir prensibidir. Loosely coupled mimaride, bileşenler birbirine minimum bağımlılıkla bağlıdır, bu da değişikliklerin izole edilmesini ve sistemin daha esnek olmasını sağlar.

### Loosely Coupled Architecture'ın Avantajları

1. **Değiştirilebilirlik**: Bileşenler, diğer bileşenleri etkilemeden değiştirilebilir.
2. **Test Edilebilirlik**: Bileşenler, bağımlılıkları mock'lanarak izole bir şekilde test edilebilir.
3. **Genişletilebilirlik**: Yeni bileşenler, mevcut kodu değiştirmeden eklenebilir.
4. **Bakım Kolaylığı**: Bileşenler arasındaki bağımlılıklar azaldığında, bakım daha kolay hale gelir.
5. **Paralel Geliştirme**: Farklı ekipler, birbirlerinin işini beklemeden paralel olarak çalışabilir.

### DIP ile Loosely Coupled Architecture Oluşturma

DIP, loosely coupled architecture oluşturmanın temel bir yoludur. Bunu başarmak için:

1. **Soyutlamalar Tanımlayın**: Sistemdeki temel bileşenler için soyutlamalar (interface'ler veya abstract class'lar) tanımlayın.
2. **Bağımlılıkları Soyutlamalara Yönlendirin**: Somut sınıflar yerine soyutlamalara bağımlı olun.
3. **Dependency Injection Kullanın**: Bağımlılıkları dışarıdan enjekte edin, içeride oluşturmayın.
4. **IoC Container Kullanın**: Bağımlılıkların yönetimini otomatikleştirin.
5. **Modüler Tasarım Yapın**: Sistemi, birbirine gevşek bağlı modüllere bölün.

## 5. En İyi Pratikler

DIP'yi etkili bir şekilde uygulamak için aşağıdaki en iyi pratikleri izleyin:

1. **Soyutlamaları Doğru Tasarlayın**
   - Soyutlamalar, implementasyon detaylarını değil, kavramsal işlevselliği yansıtmalıdır.
   - Soyutlamalar, kullanıcıların ihtiyaçlarına göre tasarlanmalıdır, implementasyonlara göre değil.
   - Interface Segregation Principle'ı uygulayarak, soyutlamaları küçük ve odaklanmış tutun.

2. **Constructor Injection'ı Tercih Edin**
   - Bağımlılıkları constructor aracılığıyla enjekte edin, bu şekilde sınıfın tüm bağımlılıkları açıkça görünür olur.
   - Constructor injection, bağımlılıkların zorunlu olduğunu ve sınıfın oluşturulması için gerekli olduğunu belirtir.
   - Property injection'ı, opsiyonel bağımlılıklar için kullanın.

3. **IoC Container Kullanın**
   - Büyük projelerde, bağımlılıkların yönetimini kolaylaştırmak için bir IoC Container kullanın.
   - Container'ı, uygulamanın composition root'unda (başlangıç noktası) yapılandırın.
   - Servis ömürlerini (transient, scoped, singleton) doğru şekilde yapılandırın.

4. **Soyutlamaları Doğru Yerde Tanımlayın**
   - Soyutlamalar, onları kullanan modüllerde tanımlanmalıdır, onları uygulayan modüllerde değil.
   - Bu, "Dependency Inversion" prensibinin "Inversion" kısmını yansıtır.
   - Örneğin, bir iş mantığı katmanı bir veritabanı soyutlamasına ihtiyaç duyuyorsa, soyutlama iş mantığı katmanında tanımlanmalıdır.

5. **Bağımlılık Çemberlerinden Kaçının**
   - A sınıfı B'ye, B sınıfı C'ye ve C sınıfı A'ya bağımlı olduğunda bağımlılık çemberleri oluşur.
   - Bu tür çemberler, sistemin anlaşılmasını ve test edilmesini zorlaştırır.
   - Bağımlılık çemberlerini kırmak için, soyutlamalar ve mediator pattern gibi tasarım desenleri kullanın.

6. **Somut Sınıfları Factory'ler Aracılığıyla Oluşturun**
   - Somut sınıfların doğrudan oluşturulması gerektiğinde, factory pattern kullanın.
   - Bu, somut sınıf oluşturma mantığını izole eder ve değiştirilebilirliği artırır.
   - Factory'leri de soyutlamalar üzerinden kullanın.

7. **Test Edilebilirliği Ön Planda Tutun**
   - DIP'nin en büyük faydalarından biri, test edilebilirliği artırmasıdır.
   - Bağımlılıkları mock'layarak, sınıfları izole bir şekilde test edin.
   - Test Driven Development (TDD) yaklaşımı, DIP'yi doğal olarak teşvik eder.

## 6. Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Service Locator Pattern'i Kötüye Kullanmak**
   ```csharp
   // Kötü - Service Locator kullanımı
   public class OrderService
   {
       private readonly IOrderRepository _orderRepository;
       
       public OrderService()
       {
           _orderRepository = ServiceLocator.Current.GetInstance<IOrderRepository>();
       }
   }
   
   // İyi - Dependency Injection kullanımı
   public class OrderService
   {
       private readonly IOrderRepository _orderRepository;
       
       public OrderService(IOrderRepository orderRepository)
       {
           _orderRepository = orderRepository;
       }
   }
   ```

2. **Somut Sınıfları Doğrudan Oluşturmak**
   ```csharp
   // Kötü - somut sınıfları doğrudan oluşturmak
   public class OrderService
   {
       private readonly SqlOrderRepository _orderRepository;
       
       public OrderService()
       {
           _orderRepository = new SqlOrderRepository();
       }
   }
   
   // İyi - soyutlamalara bağımlı olmak
   public class OrderService
   {
       private readonly IOrderRepository _orderRepository;
       
       public OrderService(IOrderRepository orderRepository)
       {
           _orderRepository = orderRepository;
       }
   }
   ```

3. **Aşırı Soyutlama**
   ```csharp
   // Kötü - aşırı soyutlama
   public interface IEntity { }
   public interface IRepository<T> where T : IEntity { }
   public interface IService<T> where T : IEntity { }
   public interface IValidator<T> where T : IEntity { }
   public interface IFactory<T> where T : IEntity { }
   
   // İyi - dengeli soyutlama
   public interface IOrderRepository
   {
       void Save(Order order);
       Order GetById(int id);
   }
   
   public interface IProductRepository
   {
       Product GetById(int id);
       IEnumerable<Product> GetAll();
   }
   ```

4. **Soyutlamaları Yanlış Yerde Tanımlamak**
   ```csharp
   // Kötü - soyutlama yanlış yerde
   // DataAccess katmanında tanımlanmış
   namespace MyApp.DataAccess
   {
       public interface IOrderRepository
       {
           void Save(Order order);
           Order GetById(int id);
       }
       
       public class SqlOrderRepository : IOrderRepository
       {
           // Implementasyon
       }
   }
   
   // İyi - soyutlama doğru yerde

   // İyi - soyutlama doğru yerde
// Domain katmanında tanımlanmış
namespace MyApp.Domain
{
    public interface IOrderRepository
    {
        void Save(Order order);
        Order GetById(int id);
    }
}

// DataAccess katmanında implementasyon
namespace MyApp.DataAccess
{
    public class SqlOrderRepository : IOrderRepository
    {
        // Implementasyon
    }
}

5. **Bağımlılık Enjeksiyonunu Aşırı Kullanmak**
```csharp
// Kötü - aşırı bağımlılık enjeksiyonu
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly ICustomerRepository _customerRepository;
    private readonly IProductRepository _productRepository;
    private readonly IInventoryService _inventoryService;
    private readonly IPaymentService _paymentService;
    private readonly INotificationService _notificationService;
    private readonly ILoggingService _loggingService;
    private readonly IMetricsService _metricsService;
    private readonly ICacheService _cacheService;
    private readonly IValidationService _validationService;
    
    public OrderService(
        IOrderRepository orderRepository,
        ICustomerRepository customerRepository,
        IProductRepository productRepository,
        IInventoryService inventoryService,
        IPaymentService paymentService,
        INotificationService notificationService,
        ILoggingService loggingService,
        IMetricsService metricsService,
        ICacheService cacheService,
        IValidationService validationService)
    {
        _orderRepository = orderRepository;
        _customerRepository = customerRepository;
        _productRepository = productRepository;
        _inventoryService = inventoryService;
        _paymentService = paymentService;
        _notificationService = notificationService;
        _loggingService = loggingService;
        _metricsService = metricsService;
        _cacheService = cacheService;
        _validationService = validationService;
    }
}

// İyi - daha az bağımlılık, daha iyi sorumluluk ayrımı
public class OrderService
{
    private readonly IOrderRepository _orderRepository;
    private readonly IPaymentService _paymentService;
    private readonly INotificationService _notificationService;
    
    public OrderService(
        IOrderRepository orderRepository,
        IPaymentService paymentService,
        INotificationService notificationService)
    {
        _orderRepository = orderRepository;
        _paymentService = paymentService;
        _notificationService = notificationService;
    }
}

public class InventoryService
{
    private readonly IProductRepository _productRepository;
    private readonly IInventoryRepository _inventoryRepository;
    
    public InventoryService(
        IProductRepository productRepository,
        IInventoryRepository inventoryRepository)
    {
        _productRepository = productRepository;
        _inventoryRepository = inventoryRepository;
    }
}
```

6. **Circular Dependency (Döngüsel Bağımlılık)**
```csharp
// Kötü - döngüsel bağımlılık
public class OrderService
{
    private readonly ICustomerService _customerService;
    
    public OrderService(ICustomerService customerService)
    {
        _customerService = customerService;
    }
}

public class CustomerService
{
    private readonly IOrderService _orderService;
    
    public CustomerService(IOrderService orderService)
    {
        _orderService = orderService;
    }
}

// İyi - döngüsel bağımlılığı kırmak
public class OrderService
{
    private readonly ICustomerRepository _customerRepository;
    
    public OrderService(ICustomerRepository customerRepository)
    {
        _customerRepository = customerRepository;
    }
}

public class CustomerService
{
    private readonly IOrderRepository _orderRepository;
    
    public CustomerService(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }
}
```

7. **Bağımlılıkları Gizlemek**
```csharp
// Kötü - gizli bağımlılıklar
public class OrderService
{
    public void PlaceOrder(Order order)
    {
        // Gizli bağımlılık
        var emailService = new EmailService();
        emailService.SendOrderConfirmation(order);
        
        // Başka bir gizli bağımlılık
        var logger = Logger.GetInstance();
        logger.Log($"Sipariş oluşturuldu: {order.Id}");
    }
}

// İyi - açık bağımlılıklar
public class OrderService
{
    private readonly INotificationService _notificationService;
    private readonly ILogger _logger;
    
    public OrderService(INotificationService notificationService, ILogger logger)
    {
        _notificationService = notificationService;
        _logger = logger;
    }
    
    public void PlaceOrder(Order order)
    {
        _notificationService.SendOrderConfirmation(order);
        _logger.Log($"Sipariş oluşturuldu: {order.Id}");
    }
}
```

Dependency Inversion Principle, yazılım tasarımında esneklik, test edilebilirlik ve bakım kolaylığı sağlamanın temel bir yoludur. Bu prensibi doğru uygulayarak, kodunuzu daha modüler, daha dayanıklı ve değişikliklere daha açık hale getirebilirsiniz. DIP, diğer SOLID prensipleriyle birlikte, daha sağlam ve sürdürülebilir yazılım sistemleri oluşturmanıza yardımcı olur. 