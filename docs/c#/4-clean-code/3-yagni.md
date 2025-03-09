# YAGNI (You Ain't Gonna Need It)

YAGNI (You Ain't Gonna Need It - İhtiyacın Olmayacak) prensibi, yazılım geliştirmede temel bir ilkedir ve "Gerçekten ihtiyaç duyuncaya kadar bir özelliği eklemeyin" fikrini savunur. Bu prensip, Extreme Programming (XP) metodolojisinin kurucularından Ron Jeffries tarafından popülerleştirilmiştir.

## 1. YAGNI Prensibinin Temel Kavramı

YAGNI prensibinin özü, gelecekte ihtiyaç duyulabilecek özellikleri önceden eklememektir. Sadece şu anda kesin olarak ihtiyaç duyulan özellikleri eklemek, kod tabanını daha basit, daha anlaşılır ve daha bakımı kolay hale getirir. Gelecekteki ihtiyaçları tahmin etmek genellikle zordur ve bu tahminler çoğu zaman yanlış çıkar.

### YAGNI İhlali Örneği

```csharp
// YAGNI ihlali - Henüz ihtiyaç duyulmayan özellikler
public class Customer
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public string Phone { get; set; }
    
    // Henüz ihtiyaç duyulmayan özellikler
    public string MiddleName { get; set; }
    public DateTime? DateOfBirth { get; set; }
    public string Address1 { get; set; }
    public string Address2 { get; set; }
    public string City { get; set; }
    public string State { get; set; }
    public string ZipCode { get; set; }
    public string Country { get; set; }
    public string TaxId { get; set; }
    public bool IsVip { get; set; }
    public decimal CreditLimit { get; set; }
    public string PreferredPaymentMethod { get; set; }
    public string Notes { get; set; }
    
    // Henüz ihtiyaç duyulmayan metotlar
    public decimal CalculateLoyaltyPoints()
    {
        // Sadece VIP müşteriler için loyalty puanları hesapla
        if (IsVip)
        {
            // Karmaşık hesaplama mantığı
            return 100.0m;
        }
        return 0;
    }
    
    public bool IsEligibleForDiscount()
    {
        // İndirim uygunluğunu kontrol et
        return IsVip || CreditLimit > 10000;
    }
    
    public string GetFullAddress()
    {
        // Tam adresi formatla
        return $"{Address1}, {(string.IsNullOrEmpty(Address2) ? "" : Address2 + ", ")}{City}, {State} {ZipCode}, {Country}";
    }
}
```

Bu örnekte, `Customer` sınıfı, mevcut iş gereksinimlerinde henüz ihtiyaç duyulmayan birçok özellik ve metot içerir. Bu, kod tabanını gereksiz yere karmaşıklaştırır ve bakımını zorlaştırır.

### YAGNI Uyumlu Tasarım

```csharp
// YAGNI uyumlu tasarım - Sadece ihtiyaç duyulan özellikler
public class Customer
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public string Phone { get; set; }
    
    // Diğer özellikler ve metotlar, ihtiyaç duyulduğunda eklenecek
}
```

Bu tasarımda, `Customer` sınıfı sadece mevcut iş gereksinimlerinde ihtiyaç duyulan özellikleri içerir. Diğer özellikler ve metotlar, gerçekten ihtiyaç duyulduğunda eklenecektir.

## 2. YAGNI Prensibinin Uygulama Alanları

YAGNI prensibi, yazılım geliştirmenin birçok alanında uygulanabilir:

### 1. Kod Düzeyinde YAGNI

Kod düzeyinde YAGNI, gereksiz kod yazmaktan kaçınmayı amaçlar. Bunu başarmak için:

- **Minimal Sınıflar**: Sınıfları, sadece mevcut iş gereksinimlerini karşılayacak şekilde tasarlayın.
- **Minimal Metotlar**: Metotları, sadece mevcut kullanım senaryolarını destekleyecek şekilde yazın.
- **Minimal Parametreler**: Metotlara, sadece mevcut işlevsellik için gerekli parametreleri ekleyin.
- **Minimal Özellikler**: Sınıflara, sadece mevcut iş gereksinimlerinde kullanılan özellikleri ekleyin.

```csharp
// YAGNI ihlali - Gereksiz parametreler
public void SendEmail(string to, string subject, string body, string cc = null, string bcc = null, bool isHtml = false, int priority = 0, bool requestReadReceipt = false)
{
    // E-posta gönderme mantığı
}

// YAGNI uyumlu - Sadece gerekli parametreler
public void SendEmail(string to, string subject, string body)
{
    // E-posta gönderme mantığı
}
```

### 2. Tasarım Düzeyinde YAGNI

Tasarım düzeyinde YAGNI, gereksiz soyutlamalardan ve karmaşık mimarilerden kaçınmayı amaçlar. Bunu başarmak için:

- **Minimal Arayüzler**: Arayüzleri, sadece mevcut kullanım senaryolarını destekleyecek şekilde tasarlayın.
- **Minimal Kalıtım**: Kalıtım hiyerarşilerini, sadece mevcut iş gereksinimlerini karşılayacak şekilde oluşturun.
- **Minimal Tasarım Desenleri**: Tasarım desenlerini, sadece mevcut problemleri çözmek için kullanın.
- **Minimal Bağımlılıklar**: Sınıflar arasındaki bağımlılıkları minimumda tutun.

```csharp
// YAGNI ihlali - Gereksiz soyutlama
public interface IRepository<T>
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
    IEnumerable<T> Find(Func<T, bool> predicate);
    void AddRange(IEnumerable<T> entities);
    void UpdateRange(IEnumerable<T> entities);
    void DeleteRange(IEnumerable<int> ids);
    int Count();
    bool Exists(int id);
}

// YAGNI uyumlu - Minimal soyutlama
public interface IRepository<T>
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
}
```

### 3. Mimari Düzeyinde YAGNI

Mimari düzeyinde YAGNI, gereksiz katmanlardan ve karmaşık yapılardan kaçınmayı amaçlar. Bunu başarmak için:

- **Minimal Katmanlar**: Sadece gerekli mimari katmanları kullanın.
- **Minimal Servisler**: Sadece mevcut iş gereksinimlerini destekleyen servisler oluşturun.
- **Minimal Veritabanı Tabloları**: Veritabanı şemasını, sadece mevcut veri ihtiyaçlarını karşılayacak şekilde tasarlayın.
- **Minimal Dış Bağımlılıklar**: Sadece gerçekten ihtiyaç duyulan dış kütüphaneleri ve servisleri kullanın.

```csharp
// YAGNI ihlali - Gereksiz karmaşık mimari
public class CustomerController
{
    private readonly ICustomerService _customerService;
    private readonly ILogger<CustomerController> _logger;
    private readonly IMapper _mapper;
    private readonly IValidator<CustomerDto> _validator;
    private readonly ICacheService _cacheService;
    private readonly IMetricsService _metricsService;
    
    public CustomerController(
        ICustomerService customerService,
        ILogger<CustomerController> logger,
        IMapper mapper,
        IValidator<CustomerDto> validator,
        ICacheService cacheService,
        IMetricsService metricsService)
    {
        _customerService = customerService;
        _logger = logger;
        _mapper = mapper;
        _validator = validator;
        _cacheService = cacheService;
        _metricsService = metricsService;
    }
    
    // Controller metotları
}

// YAGNI uyumlu - Minimal mimari
public class CustomerController
{
    private readonly ICustomerService _customerService;
    private readonly ILogger<CustomerController> _logger;
    
    public CustomerController(
        ICustomerService customerService,
        ILogger<CustomerController> logger)
    {
        _customerService = customerService;
        _logger = logger;
    }
    
    // Controller metotları
}
```

## 3. YAGNI ve Diğer Prensipler Arasındaki İlişki

YAGNI prensibi, diğer yazılım geliştirme prensipleriyle yakından ilişkilidir:

### YAGNI ve KISS

YAGNI ve KISS (Keep It Simple, Stupid) prensipleri birbirini tamamlar. YAGNI, gereksiz özellikleri eklemekten kaçınarak basitliği teşvik ederken, KISS, mevcut özellikleri basit tutmayı teşvik eder. Her iki prensip de, kod tabanını daha anlaşılır ve bakımı daha kolay hale getirmeyi amaçlar.

```csharp
// YAGNI ve KISS ihlali
public class StringHelper
{
    public static string Capitalize(string input)
    {
        if (string.IsNullOrEmpty(input))
            return input;
            
        return char.ToUpper(input[0]) + input.Substring(1).ToLower();
    }
    
    public static string CapitalizeFirstLetterOfEachWord(string input)
    {
        if (string.IsNullOrEmpty(input))
            return input;
            
        return string.Join(" ", input.Split(' ')
            .Select(word => char.ToUpper(word[0]) + word.Substring(1).ToLower()));
    }
    
    public static string Reverse(string input)
    {
        if (string.IsNullOrEmpty(input))
            return input;
            
        char[] chars = input.ToCharArray();
        Array.Reverse(chars);
        return new string(chars);
    }
    
    // Daha birçok string işleme metodu...
}

// YAGNI ve KISS uyumlu
public static class StringExtensions
{
    public static string Capitalize(this string input)
    {
        if (string.IsNullOrEmpty(input))
            return input;
            
        return char.ToUpper(input[0]) + input.Substring(1);
    }
}
```

### YAGNI ve DRY

YAGNI ve DRY (Don't Repeat Yourself) prensipleri arasında bir denge kurulmalıdır. DRY, kod tekrarını önlemeyi amaçlarken, YAGNI, gereksiz soyutlamalardan kaçınmayı amaçlar. Bazen, DRY prensibini uygulamak için oluşturulan soyutlamalar, YAGNI prensibini ihlal edebilir.

```csharp
// DRY odaklı, YAGNI ihlali
public interface IEntity
{
    int Id { get; set; }
}

public interface IRepository<T> where T : IEntity
{
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
}

public class CustomerRepository : IRepository<Customer>
{
    // Implementasyon
}

public class ProductRepository : IRepository<Product>
{
    // Implementasyon
}

public class OrderRepository : IRepository<Order>
{
    // Implementasyon
}

// YAGNI odaklı, bazı DRY ihlalleri
public class CustomerRepository
{
    public Customer GetById(int id) { /* ... */ }
    public IEnumerable<Customer> GetAll() { /* ... */ }
    public void Add(Customer customer) { /* ... */ }
}

public class ProductRepository
{
    public Product GetById(int id) { /* ... */ }
    public IEnumerable<Product> GetAll() { /* ... */ }
    // Add metodu henüz gerekli değil
}
```

### YAGNI ve SOLID

YAGNI ve SOLID prensipleri arasında da bir denge kurulmalıdır. SOLID prensipleri, kodun daha modüler, daha esnek ve daha bakımı kolay olmasını sağlarken, YAGNI, gereksiz karmaşıklıktan kaçınmayı amaçlar. Bazen, SOLID prensiplerini aşırı uygulamak, YAGNI prensibini ihlal edebilir.

```csharp
// SOLID odaklı, YAGNI ihlali
public interface ICustomerRepository
{
    Customer GetById(int id);
    IEnumerable<Customer> GetAll();
    void Add(Customer customer);
    void Update(Customer customer);
    void Delete(int id);
}

public interface ICustomerService
{
    Customer GetById(int id);
    IEnumerable<Customer> GetAll();
    void Add(Customer customer);
    void Update(Customer customer);
    void Delete(int id);
}

public class CustomerService : ICustomerService
{
    private readonly ICustomerRepository _customerRepository;
    
    public CustomerService(ICustomerRepository customerRepository)
    {
        _customerRepository = customerRepository;
    }
    
    public Customer GetById(int id) => _customerRepository.GetById(id);
    public IEnumerable<Customer> GetAll() => _customerRepository.GetAll();
    public void Add(Customer customer) => _customerRepository.Add(customer);
    public void Update(Customer customer) => _customerRepository.Update(customer);
    public void Delete(int id) => _customerRepository.Delete(id);
}

// YAGNI odaklı, bazı SOLID ihlalleri
public class CustomerRepository
{
    public Customer GetById(int id) { /* ... */ }
    public IEnumerable<Customer> GetAll() { /* ... */ }
    public void Add(Customer customer) { /* ... */ }
}

public class CustomerService
{
    private readonly CustomerRepository _customerRepository;
    
    public CustomerService(CustomerRepository customerRepository)
    {
        _customerRepository = customerRepository;
    }
    
    public Customer GetById(int id) => _customerRepository.GetById(id);
    // Diğer metotlar, ihtiyaç duyulduğunda eklenecek
}
```

## 4. Gerçek Hayattan YAGNI Örnekleri

### E-Ticaret Uygulaması Örneği

Bir e-ticaret uygulaması geliştirdiğinizi düşünün. Başlangıçta, sadece temel özelliklere ihtiyacınız var: ürün listeleme, sepete ekleme ve sipariş verme.

```csharp
// YAGNI ihlali - Gereksiz özellikler
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
    
    // Henüz ihtiyaç duyulmayan özellikler
    public string SKU { get; set; }
    public string Barcode { get; set; }
    public decimal Weight { get; set; }
    public decimal Width { get; set; }
    public decimal Height { get; set; }
    public decimal Depth { get; set; }
    public bool IsFeatured { get; set; }
    public bool IsOnSale { get; set; }
    public decimal SalePrice { get; set; }
    public DateTime? SaleStartDate { get; set; }
    public DateTime? SaleEndDate { get; set; }
    public int CategoryId { get; set; }
    public int BrandId { get; set; }
    public string Tags { get; set; }
    public string MetaTitle { get; set; }
    public string MetaDescription { get; set; }
    public string MetaKeywords { get; set; }
}

// YAGNI uyumlu - Sadece ihtiyaç duyulan özellikler
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Description { get; set; }
    public decimal Price { get; set; }
    public int StockQuantity { get; set; }
}
```

### CRM Uygulaması Örneği

Bir CRM (Customer Relationship Management) uygulaması geliştirdiğinizi düşünün. Başlangıçta, sadece müşteri bilgilerini yönetmeye ihtiyacınız var.

```csharp
// YAGNI ihlali - Gereksiz karmaşık mimari
public class CrmSystem
{
    private readonly ICustomerRepository _customerRepository;
    private readonly IContactRepository _contactRepository;
    private readonly IOpportunityRepository _opportunityRepository;
    private readonly ILeadRepository _leadRepository;
    private readonly ICampaignRepository _campaignRepository;
    private readonly IReportingService _reportingService;
    private readonly INotificationService _notificationService;
    private readonly IWorkflowEngine _workflowEngine;
    
    public CrmSystem(
        ICustomerRepository customerRepository,
        IContactRepository contactRepository,
        IOpportunityRepository opportunityRepository,
        ILeadRepository leadRepository,
        ICampaignRepository campaignRepository,
        IReportingService reportingService,
        INotificationService notificationService,
        IWorkflowEngine workflowEngine)
    {
        _customerRepository = customerRepository;
        _contactRepository = contactRepository;
        _opportunityRepository = opportunityRepository;
        _leadRepository = leadRepository;
        _campaignRepository = campaignRepository;
        _reportingService = reportingService;
        _notificationService = notificationService;
        _workflowEngine = workflowEngine;
    }
    
    // CRM işlevleri
}

// YAGNI uyumlu - Minimal mimari
public class CrmSystem
{
    private readonly ICustomerRepository _customerRepository;
    
    public CrmSystem(ICustomerRepository customerRepository)
    {
        _customerRepository = customerRepository;
    }
    
    // Temel CRM işlevleri
}
```

## 5. En İyi Pratikler

YAGNI prensibini etkili bir şekilde uygulamak için aşağıdaki en iyi pratikleri izleyin:

1. **Mevcut İhtiyaçlara Odaklanın**
   - Sadece mevcut iş gereksinimlerini karşılayan kod yazın.
   - Gelecekteki olası ihtiyaçlar için kod yazmaktan kaçının.
   - "Şu anda ihtiyacım var mı?" sorusunu sürekli sorun.

2. **Küçük Adımlarla İlerleyin**
   - Büyük, karmaşık özellikler yerine küçük, odaklanmış özellikler geliştirin.
   - Her bir özelliği, en basit şekilde implemente edin.
   - İhtiyaç duyuldukça refactoring yapın.

3. **Sürekli Geri Bildirim Alın**
   - Kullanıcılardan ve paydaşlardan sürekli geri bildirim alın.
   - Gerçek ihtiyaçları anlamak için prototip ve MVP (Minimum Viable Product) kullanın.
   - Varsayımlar yerine veriye dayalı kararlar verin.

4. **Teknik Borcu Yönetin**
   - YAGNI, teknik borç biriktirmek anlamına gelmez.
   - Kodu temiz ve bakımı kolay tutun.
   - Düzenli refactoring yaparak, kodu sürekli iyileştirin.

5. **Dengeyi Koruyun**
   - YAGNI'yi aşırıya kaçmadan uygulayın.
   - Diğer yazılım prensipleriyle (SOLID, DRY, KISS) dengeli bir şekilde kullanın.
   - Her durumda pragmatik olun.

6. **Dokümantasyon Yapın**
   - Bilinçli olarak uygulamadığınız özellikleri ve nedenlerini dokümante edin.
   - Gelecekteki geliştiricilere rehberlik etmek için tasarım kararlarınızı açıklayın.
   - Kod yorumlarını ve commit mesajlarını etkili kullanın.

7. **Test Odaklı Geliştirme Yapın**
   - Test Driven Development (TDD) kullanarak, sadece ihtiyaç duyulan kodu yazın.
   - Testler, gereksiz kod yazmanızı önlemeye yardımcı olur.
   - Testleri, gereksinimlerin doğru anlaşıldığından emin olmak için kullanın.

## 6. Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Aşırı Mühendislik**
   ```csharp
   // Kötü - aşırı mühendislik
   public interface IEntity<TKey>
   {
       TKey Id { get; set; }
   }
   
   public interface IRepository<TEntity, TKey> where TEntity : IEntity<TKey>
   {
       TEntity GetById(TKey id);
       IEnumerable<TEntity> GetAll();
       void Add(TEntity entity);
       void Update(TEntity entity);
       void Delete(TKey id);
   }
   
   public class GenericRepository<TEntity, TKey> : IRepository<TEntity, TKey> where TEntity : class, IEntity<TKey>
   {
       private readonly DbContext _context;
       private readonly DbSet<TEntity> _dbSet;
       
       public GenericRepository(DbContext context)
       {
           _context = context;
           _dbSet = context.Set<TEntity>();
       }
       
       public TEntity GetById(TKey id) => _dbSet.Find(id);
       public IEnumerable<TEntity> GetAll() => _dbSet.ToList();
       public void Add(TEntity entity) => _dbSet.Add(entity);
       public void Update(TEntity entity) => _context.Entry(entity).State = EntityState.Modified;
       public void Delete(TKey id)
       {
           var entity = GetById(id);
           if (entity != null)
               _dbSet.Remove(entity);
       }
   }
   
   // İyi - basit ve yeterli
   public class CustomerRepository
   {
       private readonly DbContext _context;
       
       public CustomerRepository(DbContext context)
       {
           _context = context;
       }
       
       public Customer GetById(int id) => _context.Customers.Find(id);
       public List<Customer> GetAll() => _context.Customers.ToList();
       public void Add(Customer customer) => _context.Customers.Add(customer);
   }
   ```

2. **Gereksiz Esneklik**
   ```csharp
   // Kötü - gereksiz esneklik
   public class ConfigurationManager
   {
       private readonly Dictionary<string, object> _settings = new Dictionary<string, object>();
       private readonly List<IConfigurationProvider> _providers = new List<IConfigurationProvider>();
       private readonly List<IConfigurationValidator> _validators = new List<IConfigurationValidator>();
       private readonly List<IConfigurationTransformer> _transformers = new List<IConfigurationTransformer>();
       
       public void AddProvider(IConfigurationProvider provider)
       {
           _providers.Add(provider);
           RefreshSettings();
       }
       
       public void AddValidator(IConfigurationValidator validator)
       {
           _validators.Add(validator);
           ValidateSettings();
       }
       
       public void AddTransformer(IConfigurationTransformer transformer)
       {
           _transformers.Add(transformer);
           TransformSettings();
       }
       
       public T GetSetting<T>(string key, T defaultValue = default)
       {
           if (_settings.TryGetValue(key, out var value) && value is T typedValue)
               return typedValue;
               
           return defaultValue;
       }
       
       private void RefreshSettings()
       {
           // Tüm sağlayıcılardan ayarları yükle
       }
       
       private void ValidateSettings()
       {
           // Tüm doğrulayıcılarla ayarları doğrula
       }
       
       private void TransformSettings()
       {
           // Tüm dönüştürücülerle ayarları dönüştür
       }
   }
   
   // İyi - basit ve doğrudan
   public class ConfigurationManager
   {
       private readonly Dictionary<string, string> _settings = new Dictionary<string, string>();
       
       public ConfigurationManager(string configFilePath)
       {
           // Yapılandırma dosyasından ayarları yükle
       }
       
       public string GetSetting(string key, string defaultValue = null)
       {
           return _settings.TryGetValue(key, out var value) ? value : defaultValue;
       }
   }
   ```

3. **Gereksiz Soyutlama**
   ```csharp
   // Kötü - gereksiz soyutlama
   public interface ILogger
   {
       void Log(string message);
       void LogError(string message, Exception ex);
       void LogWarning(string message);
       void LogInfo(string message);
       void LogDebug(string message);
       void LogVerbose(string message);
   }
   
   public interface ILoggerFactory
   {
       ILogger CreateLogger(string name);
   }
   
   public class ConsoleLoggerFactory : ILoggerFactory
   {
       public ILogger CreateLogger(string name)
       {
           return new ConsoleLogger(name);
       }
   }
   
   public class ConsoleLogger : ILogger
   {
       private readonly string _name;
       
       public ConsoleLogger(string name)
       {
           _name = name;
       }
       
       public void Log(string message) => Console.WriteLine($"[{_name}] {message}");
       public void LogError(string message, Exception ex) => Console.WriteLine($"[{_name}] ERROR: {message} - {ex}");
       public void LogWarning(string message) => Console.WriteLine($"[{_name}] WARNING: {message}");
       public void LogInfo(string message) => Console.WriteLine($"[{_name}] INFO: {message}");
       public void LogDebug(string message) => Console.WriteLine($"[{_name}] DEBUG: {message}");
       public void LogVerbose(string message) => Console.WriteLine($"[{_name}] VERBOSE: {message}");
   }
   
   // İyi - basit ve doğrudan
   public class Logger
   {
       public static void Log(string message) => Console.WriteLine(message);
       public static void LogError(string message, Exception ex) => Console.WriteLine($"ERROR: {message} - {ex}");
   }
   ```

4. **Gereksiz Genelleştirme**
   ```csharp
   // Kötü - gereksiz genelleştirme
   public class DataProcessor<TInput, TOutput, TContext>
   {
       private readonly Func<TInput, TContext, TOutput> _processFunc;
       private readonly TContext _context;
       
       public DataProcessor(Func<TInput, TContext, TOutput> processFunc, TContext context)
       {
           _processFunc = processFunc;
           _context = context;
       }
       
       public TOutput Process(TInput input)
       {
           return _processFunc(input, _context);
       }
   }
   
   // Kullanım
   var processor = new DataProcessor<string, int, Dictionary<string, int>>(
       (input, context) => {
           if (context.TryGetValue(input, out var value))
               return value;
           return 0;
       },
       new Dictionary<string, int> { { "one", 1 }, { "two", 2 }, { "three", 3 } }
   );
   
   int result = processor.Process("two"); // 2
   
   // İyi - basit ve spesifik
   public class StringToIntConverter
   {
       private readonly Dictionary<string, int> _mapping;
       
       public StringToIntConverter()
       {
           _mapping = new Dictionary<string, int> { { "one", 1 }, { "two", 2 }, { "three", 3 } };
       }
       
       public int Convert(string input)
       {
           return _mapping.TryGetValue(input, out var value) ? value : 0;
       }
   }
   

5. **Gereksiz Özellikler**
   ```csharp
   // Kötü - gereksiz özellikler
   public class User
   {
       public int Id { get; set; }
       public string Username { get; set; }
       public string Password { get; set; }
       public string Email { get; set; }
       
       // Henüz ihtiyaç olmayan özellikler
       public string ProfilePicture { get; set; }
       public string Theme { get; set; }
       public string Language { get; set; }
       public bool IsTwoFactorEnabled { get; set; }
       public string[] Roles { get; set; }
       public Dictionary<string, string> Preferences { get; set; }
       public DateTime LastLoginDate { get; set; }
       public int LoginAttempts { get; set; }
   }
   
   // İyi - sadece gerekli özellikler
   public class User
   {
       public int Id { get; set; }
       public string Username { get; set; }
       public string Password { get; set; }
       public string Email { get; set; }
   }
   ```

6. **Gereksiz Altyapı**
   ```csharp
   // Kötü - gereksiz altyapı
   public class OrderProcessor
   {
       private readonly IOrderRepository _orderRepository;
       private readonly IPaymentGateway _paymentGateway;
       private readonly IInventoryService _inventoryService;
       private readonly IEmailService _emailService;
       private readonly ILogger _logger;
       private readonly IMetricsCollector _metricsCollector;
       private readonly ICacheManager _cacheManager;
       private readonly IEventBus _eventBus;
       
       public OrderProcessor(
           IOrderRepository orderRepository,
           IPaymentGateway paymentGateway,
           IInventoryService inventoryService,
           IEmailService emailService,
           ILogger logger,
           IMetricsCollector metricsCollector,
           ICacheManager cacheManager,
           IEventBus eventBus)
       {
           _orderRepository = orderRepository;
           _paymentGateway = paymentGateway;
           _inventoryService = inventoryService;
           _emailService = emailService;
           _logger = logger;
           _metricsCollector = metricsCollector;
           _cacheManager = cacheManager;
           _eventBus = eventBus;
       }
       
       public async Task ProcessOrder(Order order)
       {
           // Karmaşık sipariş işleme mantığı
       }
   }
   
   // İyi - minimal altyapı
   public class OrderProcessor
   {
       private readonly IOrderRepository _orderRepository;
       private readonly IPaymentGateway _paymentGateway;
       
       public OrderProcessor(
           IOrderRepository orderRepository,
           IPaymentGateway paymentGateway)
       {
           _orderRepository = orderRepository;
           _paymentGateway = paymentGateway;
       }
       
       public async Task ProcessOrder(Order order)
       {
           // Basit sipariş işleme mantığı
       }
   }
   ```

7. **Gereksiz Konfigürasyon**
   ```csharp
   // Kötü - gereksiz konfigürasyon
   public class ApplicationSettings
   {
       public DatabaseSettings Database { get; set; }
       public EmailSettings Email { get; set; }
       public CacheSettings Cache { get; set; }
       public LoggingSettings Logging { get; set; }
       public SecuritySettings Security { get; set; }
       public UISettings UI { get; set; }
       public IntegrationSettings Integration { get; set; }
       public MonitoringSettings Monitoring { get; set; }
   }
   
   // İyi - minimal konfigürasyon
   public class ApplicationSettings
   {
       public string ConnectionString { get; set; }
       public string SmtpServer { get; set; }
   }
   ```

YAGNI prensibi, yazılım geliştirmede gereksiz karmaşıklığı önlemenin temel bir yoludur. Bu prensibi doğru uygulayarak, kodunuzu daha bakımı kolay, daha anlaşılır ve daha esnek hale getirebilirsiniz. Ancak, YAGNI'yi aşırıya kaçmadan, diğer prensipler ve iş gereksinimleri ile dengeli bir şekilde uygulamak önemlidir.

Önemli noktalar:
- Sadece mevcut gereksinimleri karşılayan kod yazın
- Gelecekteki olası ihtiyaçlar için şimdiden kod yazmayın
- Basit çözümlerle başlayın ve gerektiğinde genişletin
- Diğer prensiplerle (SOLID, DRY, KISS) dengeli bir şekilde uygulayın
- Teknik borç biriktirmekten kaçının
- Sürekli geri bildirim alın ve buna göre geliştirin 