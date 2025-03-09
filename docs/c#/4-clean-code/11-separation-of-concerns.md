# Separation of Concerns (SoC)

Separation of Concerns (Sorumlulukların Ayrılması) prensibi, bir yazılım sistemini farklı bölümlere ayırarak her bölümün ayrı bir sorumluluğa veya ilgi alanına odaklanmasını sağlayan temel bir tasarım prensibidir. Bu prensip, Edsger W. Dijkstra tarafından ortaya atılmıştır.

## 1. Separation of Concerns'in Temel Kavramı

SoC prensibinin özü, bir programı farklı bölümlere ayırmak ve her bölümün belirli bir sorumluluğa sahip olmasını sağlamaktır. Bu yaklaşım, karmaşıklığı azaltır, bakımı kolaylaştırır ve kodun yeniden kullanılabilirliğini artırır.

### SoC İhlali Örneği

```csharp
// SoC ihlali - Tek bir sınıfta çok fazla sorumluluk
public class OrderProcessor
{
    private readonly string _connectionString;
    
    public OrderProcessor(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public void ProcessOrder(Order order)
    {
        // Veri doğrulama
        if (order == null)
            throw new ArgumentNullException(nameof(order));
            
        if (order.Items == null || order.Items.Count == 0)
            throw new ArgumentException("Sipariş en az bir ürün içermelidir.");
            
        if (string.IsNullOrEmpty(order.CustomerEmail))
            throw new ArgumentException("Müşteri e-postası gereklidir.");
            
        // Veritabanı işlemleri
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            
            // Siparişi kaydet
            using (var command = new SqlCommand("INSERT INTO Orders (CustomerId, OrderDate, TotalAmount) VALUES (@CustomerId, @OrderDate, @TotalAmount); SELECT SCOPE_IDENTITY();", connection))
            {
                command.Parameters.AddWithValue("@CustomerId", order.CustomerId);
                command.Parameters.AddWithValue("@OrderDate", DateTime.Now);
                command.Parameters.AddWithValue("@TotalAmount", order.Items.Sum(i => i.Price * i.Quantity));
                
                var orderId = Convert.ToInt32(command.ExecuteScalar());
                order.Id = orderId;
            }
            
            // Sipariş öğelerini kaydet
            foreach (var item in order.Items)
            {
                using (var command = new SqlCommand("INSERT INTO OrderItems (OrderId, ProductId, Quantity, Price) VALUES (@OrderId, @ProductId, @Quantity, @Price)", connection))
                {
                    command.Parameters.AddWithValue("@OrderId", order.Id);
                    command.Parameters.AddWithValue("@ProductId", item.ProductId);
                    command.Parameters.AddWithValue("@Quantity", item.Quantity);
                    command.Parameters.AddWithValue("@Price", item.Price);
                    
                    command.ExecuteNonQuery();
                }
            }
        }
        
        // E-posta bildirimi
        var smtpClient = new SmtpClient("smtp.example.com")
        {
            Port = 587,
            Credentials = new NetworkCredential("username", "password"),
            EnableSsl = true
        };
        
        var mailMessage = new MailMessage
        {
            From = new MailAddress("orders@example.com"),
            Subject = $"Sipariş Onayı #{order.Id}",
            Body = $"Sayın Müşterimiz,\n\nSiparişiniz başarıyla alınmıştır. Sipariş numaranız: {order.Id}\n\nTeşekkür ederiz."
        };
        
        mailMessage.To.Add(order.CustomerEmail);
        smtpClient.Send(mailMessage);
        
        // Stok güncelleme
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            
            foreach (var item in order.Items)
            {
                using (var command = new SqlCommand("UPDATE Products SET StockQuantity = StockQuantity - @Quantity WHERE Id = @ProductId", connection))
                {
                    command.Parameters.AddWithValue("@Quantity", item.Quantity);
                    command.Parameters.AddWithValue("@ProductId", item.ProductId);
                    
                    command.ExecuteNonQuery();
                }
            }
        }
    }
}
```

Bu örnekte, `OrderProcessor` sınıfı birçok farklı sorumluluğu üstleniyor:
1. Veri doğrulama
2. Veritabanı işlemleri
3. E-posta bildirimi
4. Stok yönetimi

### SoC Uyumlu Tasarım

```csharp
// SoC uyumlu tasarım - Her sınıf tek bir sorumluluğa sahip

// 1. Veri doğrulama
public class OrderValidator
{
    public void Validate(Order order)
    {
        if (order == null)
            throw new ArgumentNullException(nameof(order));
            
        if (order.Items == null || order.Items.Count == 0)
            throw new ArgumentException("Sipariş en az bir ürün içermelidir.");
            
        if (string.IsNullOrEmpty(order.CustomerEmail))
            throw new ArgumentException("Müşteri e-postası gereklidir.");
    }
}

// 2. Veritabanı işlemleri
public class OrderRepository
{
    private readonly string _connectionString;
    
    public OrderRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public void SaveOrder(Order order)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            
            // Siparişi kaydet
            using (var command = new SqlCommand("INSERT INTO Orders (CustomerId, OrderDate, TotalAmount) VALUES (@CustomerId, @OrderDate, @TotalAmount); SELECT SCOPE_IDENTITY();", connection))
            {
                command.Parameters.AddWithValue("@CustomerId", order.CustomerId);
                command.Parameters.AddWithValue("@OrderDate", DateTime.Now);
                command.Parameters.AddWithValue("@TotalAmount", order.Items.Sum(i => i.Price * i.Quantity));
                
                var orderId = Convert.ToInt32(command.ExecuteScalar());
                order.Id = orderId;
            }
            
            // Sipariş öğelerini kaydet
            foreach (var item in order.Items)
            {
                using (var command = new SqlCommand("INSERT INTO OrderItems (OrderId, ProductId, Quantity, Price) VALUES (@OrderId, @ProductId, @Quantity, @Price)", connection))
                {
                    command.Parameters.AddWithValue("@OrderId", order.Id);
                    command.Parameters.AddWithValue("@ProductId", item.ProductId);
                    command.Parameters.AddWithValue("@Quantity", item.Quantity);
                    command.Parameters.AddWithValue("@Price", item.Price);
                    
                    command.ExecuteNonQuery();
                }
            }
        }
    }
}

// 3. E-posta bildirimi
public class OrderNotificationService
{
    private readonly SmtpClient _smtpClient;
    
    public OrderNotificationService(string smtpServer, int port, string username, string password)
    {
        _smtpClient = new SmtpClient(smtpServer)
        {
            Port = port,
            Credentials = new NetworkCredential(username, password),
            EnableSsl = true
        };
    }
    
    public void SendOrderConfirmation(Order order)
    {
        var mailMessage = new MailMessage
        {
            From = new MailAddress("orders@example.com"),
            Subject = $"Sipariş Onayı #{order.Id}",
            Body = $"Sayın Müşterimiz,\n\nSiparişiniz başarıyla alınmıştır. Sipariş numaranız: {order.Id}\n\nTeşekkür ederiz."
        };
        
        mailMessage.To.Add(order.CustomerEmail);
        _smtpClient.Send(mailMessage);
    }
}

// 4. Stok yönetimi
public class InventoryService
{
    private readonly string _connectionString;
    
    public InventoryService(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public void UpdateStock(IEnumerable<OrderItem> items)
    {
        using (var connection = new SqlConnection(_connectionString))
        {
            connection.Open();
            
            foreach (var item in items)
            {
                using (var command = new SqlCommand("UPDATE Products SET StockQuantity = StockQuantity - @Quantity WHERE Id = @ProductId", connection))
                {
                    command.Parameters.AddWithValue("@Quantity", item.Quantity);
                    command.Parameters.AddWithValue("@ProductId", item.ProductId);
                    
                    command.ExecuteNonQuery();
                }
            }
        }
    }
}

// Orchestrator - Tüm bileşenleri koordine eder
public class OrderProcessor
{
    private readonly OrderValidator _validator;
    private readonly OrderRepository _repository;
    private readonly OrderNotificationService _notificationService;
    private readonly InventoryService _inventoryService;
    
    public OrderProcessor(
        OrderValidator validator,
        OrderRepository repository,
        OrderNotificationService notificationService,
        InventoryService inventoryService)
    {
        _validator = validator;
        _repository = repository;
        _notificationService = notificationService;
        _inventoryService = inventoryService;
    }
    
    public void ProcessOrder(Order order)
    {
        // 1. Doğrulama
        _validator.Validate(order);
        
        // 2. Veritabanına kaydetme
        _repository.SaveOrder(order);
        
        // 3. Bildirim gönderme
        _notificationService.SendOrderConfirmation(order);
        
        // 4. Stok güncelleme
        _inventoryService.UpdateStock(order.Items);
    }
}
```

Bu tasarımda, her sınıf tek bir sorumluluğa sahiptir ve `OrderProcessor` sınıfı sadece bu bileşenleri koordine eder.

## 2. SoC'nin Uygulama Alanları

### 1. Katmanlı Mimari

```csharp
// Presentation Layer (UI)
public class CustomerController : Controller
{
    private readonly ICustomerService _customerService;
    
    public CustomerController(ICustomerService customerService)
    {
        _customerService = customerService;
    }
    
    public IActionResult Index()
    {
        var customers = _customerService.GetAllCustomers();
        return View(customers);
    }
    
    [HttpPost]
    public IActionResult Create(CustomerViewModel model)
    {
        if (!ModelState.IsValid)
            return View(model);
            
        var customer = new Customer
        {
            Name = model.Name,
            Email = model.Email,
            Phone = model.Phone
        };
        
        _customerService.CreateCustomer(customer);
        
        return RedirectToAction("Index");
    }
}

// Business Layer (Service)
public interface ICustomerService
{
    IEnumerable<Customer> GetAllCustomers();
    void CreateCustomer(Customer customer);
}

public class CustomerService : ICustomerService
{
    private readonly ICustomerRepository _customerRepository;
    private readonly IValidator<Customer> _validator;
    
    public CustomerService(ICustomerRepository customerRepository, IValidator<Customer> validator)
    {
        _customerRepository = customerRepository;
        _validator = validator;
    }
    
    public IEnumerable<Customer> GetAllCustomers()
    {
        return _customerRepository.GetAll();
    }
    
    public void CreateCustomer(Customer customer)
    {
        _validator.ValidateAndThrow(customer);
        _customerRepository.Add(customer);
    }
}

// Data Access Layer (Repository)
public interface ICustomerRepository
{
    IEnumerable<Customer> GetAll();
    void Add(Customer customer);
}

public class CustomerRepository : ICustomerRepository
{
    private readonly DbContext _dbContext;
    
    public CustomerRepository(DbContext dbContext)
    {
        _dbContext = dbContext;
    }
    
    public IEnumerable<Customer> GetAll()
    {
        return _dbContext.Customers.ToList();
    }
    
    public void Add(Customer customer)
    {
        _dbContext.Customers.Add(customer);
        _dbContext.SaveChanges();
    }
}
```

### 2. MVC (Model-View-Controller) Deseni

```csharp
// Model - Veri ve iş mantığı
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    
    public bool IsInStock()
    {
        return StockQuantity > 0;
    }
    
    public void ReduceStock(int quantity)
    {
        if (quantity <= 0)
            throw new ArgumentException("Miktar pozitif olmalıdır.");
            
        if (quantity > StockQuantity)
            throw new InvalidOperationException("Yeterli stok yok.");
            
        StockQuantity -= quantity;
    }
}

// View - Kullanıcı arayüzü
@model Product

<div class="product-details">
    <h2>@Model.Name</h2>
    <p>Fiyat: @Model.Price.ToString("C")</p>
    
    @if (Model.IsInStock())
    {
        <p class="in-stock">Stokta var</p>
        <form asp-action="AddToCart" method="post">
            <input type="hidden" name="productId" value="@Model.Id" />
            <button type="submit">Sepete Ekle</button>
        </form>
    }
    else
    {
        <p class="out-of-stock">Stokta yok</p>
    }
</div>

// Controller - Kullanıcı etkileşimlerini yönetir
public class ProductController : Controller
{
    private readonly IProductService _productService;
    private readonly ICartService _cartService;
    
    public ProductController(IProductService productService, ICartService cartService)
    {
        _productService = productService;
        _cartService = cartService;
    }
    
    public IActionResult Details(int id)
    {
        var product = _productService.GetProductById(id);
        
        if (product == null)
            return NotFound();
            
        return View(product);
    }
    
    [HttpPost]
    public IActionResult AddToCart(int productId)
    {
        var product = _productService.GetProductById(productId);
        
        if (product == null)
            return NotFound();
            
        if (!product.IsInStock())
            return BadRequest("Ürün stokta yok.");
            
        _cartService.AddToCart(User.Identity.Name, product, 1);
        
        return RedirectToAction("Index", "Cart");
    }
}
```

## 3. SoC'nin Faydaları

1. **Daha İyi Bakım Yapılabilirlik**
   - Her bileşen bağımsız olarak değiştirilebilir
   - Değişiklikler izole edilebilir
   - Hata ayıklama kolaylaşır

2. **Daha Kolay Test Edilebilirlik**
   - Bileşenler ayrı ayrı test edilebilir
   - Mock nesneleri daha kolay oluşturulabilir
   - Test kapsamı daha net olur

3. **Daha İyi Yeniden Kullanılabilirlik**
   - Bileşenler farklı bağlamlarda kullanılabilir
   - Bağımlılıklar azalır
   - Kod tekrarı önlenir

4. **Daha Kolay Geliştirme**
   - Ekip üyeleri paralel çalışabilir
   - Sorumluluklar net bir şekilde tanımlanır
   - Öğrenme eğrisi azalır

## 4. En İyi Pratikler

1. **Sorumlulukları Net Bir Şekilde Tanımlayın**
   - Her sınıfın tek bir sorumluluğu olmalı
   - Her metot tek bir iş yapmalı
   - Sorumluluklar mantıklı bir şekilde gruplandırılmalı

2. **Arayüzleri Kullanın**
   - Bileşenler arasındaki bağımlılıkları azaltmak için arayüzler kullanın
   - Somut sınıflar yerine soyutlamalara bağımlı olun
   - Dependency Injection kullanın

3. **Katmanlı Mimari Kullanın**
   - Uygulamayı mantıklı katmanlara ayırın
   - Katmanlar arasındaki bağımlılık yönünü kontrol edin
   - Her katmanın belirli bir sorumluluğu olsun

4. **Cross-Cutting Concerns'i Ayırın**
   - Loglama, güvenlik, önbellekleme gibi kesişen ilgileri ayırın
   - Aspect-Oriented Programming (AOP) tekniklerini kullanın
   - Dekoratör deseni veya davranış zincirleri kullanın

5. **Uygun Tasarım Desenleri Kullanın**
   - MVC, MVVM, Repository, Unit of Work gibi desenleri kullanın
   - Her desen belirli bir sorumluluğu ayırmaya yardımcı olur
   - Desenleri doğru bağlamda kullanın

## 5. Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Aşırı Parçalama**
   ```csharp
   // Kötü - aşırı parçalama
   public class CustomerNameValidator
   {
       public bool ValidateName(string name) { /* ... */ }
   }
   
   public class CustomerEmailValidator
   {
       public bool ValidateEmail(string email) { /* ... */ }
   }
   
   public class CustomerPhoneValidator
   {
       public bool ValidatePhone(string phone) { /* ... */ }
   }
   
   // İyi - dengeli parçalama
   public class CustomerValidator
   {
       public bool ValidateName(string name) { /* ... */ }
       public bool ValidateEmail(string email) { /* ... */ }
       public bool ValidatePhone(string phone) { /* ... */ }
       
       public bool ValidateCustomer(Customer customer)
       {
           return ValidateName(customer.Name) && 
                  ValidateEmail(customer.Email) && 
                  ValidatePhone(customer.Phone);
       }
   }
   ```

2. **Yetersiz Ayrım**
   ```csharp
   // Kötü - yetersiz ayrım
   public class OrderService
   {
       public void ProcessOrder(Order order)
       {
           // Doğrulama, veritabanı işlemleri, e-posta gönderme, stok güncelleme...
       }
   }
   
   // İyi - yeterli ayrım
   public class OrderService
   {
       private readonly IOrderValidator _validator;
       private readonly IOrderRepository _repository;
       private readonly INotificationService _notificationService;
       private readonly IInventoryService _inventoryService;
       
       // Constructor ve ProcessOrder metodu...
   }
   ```

3. **Yanlış Sorumluluk Ataması**
   ```csharp
   // Kötü - yanlış sorumluluk ataması
   public class Customer
   {
       public int Id { get; set; }
       public string Name { get; set; }
       public string Email { get; set; }
       
       // Müşteri sınıfı veritabanı işlemleri yapmamalı
       public void Save()
       {
           using (var connection = new SqlConnection("connection_string"))
           {
               // Veritabanına kaydetme kodu...
           }
       }
       
       // Müşteri sınıfı e-posta gönderme işlemi yapmamalı
       public void SendWelcomeEmail()
       {
           var smtpClient = new SmtpClient();
           // E-posta gönderme kodu...
       }
   }
   
   // İyi - doğru sorumluluk ataması
   public class Customer
   {
       public int Id { get; set; }
       public string Name { get; set; }
       public string Email { get; set; }
   }
   
   public class CustomerRepository
   {
       public void Save(Customer customer)
       {
           // Veritabanına kaydetme kodu...
       }
   }
   
   public class CustomerNotificationService
   {
       public void SendWelcomeEmail(Customer customer)
       {
           // E-posta gönderme kodu...
       }
   }
   ```

4. **Katmanlar Arası Sızıntı**
   ```csharp
   // Kötü - katmanlar arası sızıntı
   public class CustomerController : Controller
   {
       private readonly DbContext _dbContext;
       
       public CustomerController(DbContext dbContext)
       {
           _dbContext = dbContext;
       }
       
       public IActionResult Index()
       {
           // Controller doğrudan veritabanına erişmemeli
           var customers = _dbContext.Customers.ToList();
           return View(customers);
       }
   }
   
   // İyi - katmanlar arası temiz ayrım
   public class CustomerController : Controller
   {
       private readonly ICustomerService _customerService;
       
       public CustomerController(ICustomerService customerService)
       {
           _customerService = customerService;
       }
       
       public IActionResult Index()
       {
           var customers = _customerService.GetAllCustomers();
           return View(customers);
       }
   }
   ```

Separation of Concerns prensibi, yazılım tasarımında karmaşıklığı yönetmenin ve bakımı kolaylaştırmanın temel bir yoludur. Bu prensibi doğru uygulayarak, daha modüler, test edilebilir ve sürdürülebilir sistemler geliştirebilirsiniz. 