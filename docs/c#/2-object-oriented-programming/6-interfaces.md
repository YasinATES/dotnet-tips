# Arayüzler (Interfaces)

Arayüzler (interfaces), C# dilinin en güçlü özelliklerinden biridir ve sürdürülebilir, genişletilebilir yazılım geliştirmenin temel yapı taşlarından biridir. Arayüzler, sınıfların "ne yapabildiğini" tanımlar, ancak "nasıl yaptığını" tanımlamaz. Bu bölümde, C#'ta arayüzleri ve ilgili konuları detaylı olarak inceleyeceğiz.

## 1. Interface Tanımlama ve Uygulama

Arayüzler, bir sınıfın uygulaması gereken metotları, özellikleri, olayları ve indeksleyicileri tanımlar.

### Temel Interface Tanımlama

```csharp
// Ödeme işlemi arayüzü
public interface IPaymentProcessor
{
    // Metotlar
    bool ProcessPayment(decimal amount);
    void RefundPayment(string transactionId, decimal amount);
    
    // Özellikler
    string PaymentMethod { get; }
    bool IsAvailable { get; }
    
    // İndeksleyici
    TransactionInfo this[string transactionId] { get; }
}

// İşlem bilgisi sınıfı
public class TransactionInfo
{
    public string TransactionId { get; set; }
    public decimal Amount { get; set; }
    public DateTime Date { get; set; }
    public string Status { get; set; }
}
```

### Interface Uygulama

```csharp
// Kredi kartı ödeme işlemcisi
public class CreditCardProcessor : IPaymentProcessor
{
    private Dictionary<string, TransactionInfo> _transactions = new Dictionary<string, TransactionInfo>();
    
    public string PaymentMethod => "Kredi Kartı";
    
    public bool IsAvailable
    {
        get
        {
            // Kredi kartı ödeme sisteminin durumunu kontrol et
            return CheckPaymentGatewayStatus();
        }
    }
    
    public TransactionInfo this[string transactionId]
    {
        get
        {
            if (_transactions.TryGetValue(transactionId, out var transaction))
                return transaction;
                
            throw new KeyNotFoundException($"İşlem bulunamadı: {transactionId}");
        }
    }
    
    public bool ProcessPayment(decimal amount)
    {
        // Kredi kartı ödeme işlemi
        Console.WriteLine($"Kredi kartı ile {amount:C} ödeme işlemi gerçekleştiriliyor...");
        
        // Ödeme işlemi simülasyonu
        bool success = true; // Gerçek uygulamada ödeme ağı yanıtına göre belirlenir
        
        if (success)
        {
            // Başarılı işlemi kaydet
            string transactionId = GenerateTransactionId();
            _transactions[transactionId] = new TransactionInfo
            {
                TransactionId = transactionId,
                Amount = amount,
                Date = DateTime.Now,
                Status = "Completed"
            };
            
            Console.WriteLine($"Ödeme başarılı. İşlem No: {transactionId}");
        }
        
        return success;
    }
    
    public void RefundPayment(string transactionId, decimal amount)
    {
        // İşlemi bul
        if (!_transactions.TryGetValue(transactionId, out var transaction))
        {
            throw new KeyNotFoundException($"İşlem bulunamadı: {transactionId}");
        }
        
        // İade işlemi
        Console.WriteLine($"Kredi kartına {amount:C} iade işlemi gerçekleştiriliyor...");
        Console.WriteLine($"İşlem No: {transactionId} için iade tamamlandı.");
        
        // İşlem durumunu güncelle
        transaction.Status = "Refunded";
    }
    
    private bool CheckPaymentGatewayStatus()
    {
        // Ödeme ağı durumunu kontrol etme simülasyonu
        return true;
    }
    
    private string GenerateTransactionId()
    {
        // Benzersiz işlem numarası oluşturma
        return "CC-" + Guid.NewGuid().ToString().Substring(0, 8).ToUpper();
    }
}

// Banka havalesi ödeme işlemcisi
public class BankTransferProcessor : IPaymentProcessor
{
    private Dictionary<string, TransactionInfo> _transactions = new Dictionary<string, TransactionInfo>();
    
    public string PaymentMethod => "Banka Havalesi";
    
    public bool IsAvailable => true; // Banka havalesi her zaman kullanılabilir
    
    public TransactionInfo this[string transactionId]
    {
        get
        {
            if (_transactions.TryGetValue(transactionId, out var transaction))
                return transaction;
                
            throw new KeyNotFoundException($"İşlem bulunamadı: {transactionId}");
        }
    }
    
    public bool ProcessPayment(decimal amount)
    {
        // Banka havalesi işlemi
        Console.WriteLine($"Banka havalesi ile {amount:C} ödeme işlemi gerçekleştiriliyor...");
        Console.WriteLine("Lütfen aşağıdaki IBAN'a ödemeyi yapın:");
        Console.WriteLine("TR12 3456 7890 1234 5678 9012 34");
        
        // Havale işlemi manuel onay gerektirir
        string transactionId = GenerateTransactionId();
        _transactions[transactionId] = new TransactionInfo
        {
            TransactionId = transactionId,
            Amount = amount,
            Date = DateTime.Now,
            Status = "Pending" // Beklemede
        };
        
        Console.WriteLine($"Ödeme beklemede. İşlem No: {transactionId}");
        
        return true; // İşlem başlatıldı
    }
    
    public void RefundPayment(string transactionId, decimal amount)
    {
        // İşlemi bul
        if (!_transactions.TryGetValue(transactionId, out var transaction))
        {
            throw new KeyNotFoundException($"İşlem bulunamadı: {transactionId}");
        }
        
        // İade işlemi
        Console.WriteLine($"Banka hesabına {amount:C} iade işlemi gerçekleştiriliyor...");
        Console.WriteLine("Lütfen IBAN bilgilerinizi girin:");
        
        // İşlem durumunu güncelle
        transaction.Status = "Refund Pending";
    }
    
    private string GenerateTransactionId()
    {
        // Benzersiz işlem numarası oluşturma
        return "BT-" + Guid.NewGuid().ToString().Substring(0, 8).ToUpper();
    }
}
```

### Interface Kullanımı

```csharp
// Ödeme işlemcilerini kullanma
public class PaymentService
{
    private List<IPaymentProcessor> _paymentProcessors = new List<IPaymentProcessor>();
    
    public PaymentService()
    {
        // Kullanılabilir ödeme işlemcilerini kaydet
        _paymentProcessors.Add(new CreditCardProcessor());
        _paymentProcessors.Add(new BankTransferProcessor());
    }
    
    public List<string> GetAvailablePaymentMethods()
    {
        // Kullanılabilir ödeme yöntemlerini listele
        return _paymentProcessors
            .Where(p => p.IsAvailable)
            .Select(p => p.PaymentMethod)
            .ToList();
    }
    
    public bool ProcessPayment(string paymentMethod, decimal amount)
    {
        // Seçilen ödeme yöntemiyle ödeme işlemini gerçekleştir
        var processor = _paymentProcessors
            .FirstOrDefault(p => p.IsAvailable && p.PaymentMethod == paymentMethod);
            
        if (processor == null)
        {
            Console.WriteLine($"Ödeme yöntemi bulunamadı veya kullanılamıyor: {paymentMethod}");
            return false;
        }
        
        return processor.ProcessPayment(amount);
    }
    
    public void RefundPayment(string paymentMethod, string transactionId, decimal amount)
    {
        // Seçilen ödeme yöntemiyle iade işlemini gerçekleştir
        var processor = _paymentProcessors
            .FirstOrDefault(p => p.PaymentMethod == paymentMethod);
            
        if (processor == null)
        {
            Console.WriteLine($"Ödeme yöntemi bulunamadı: {paymentMethod}");
            return;
        }
        
        processor.RefundPayment(transactionId, amount);
    }
}
```

### Interface'lerin Önemi

Interface'ler olmadan, farklı ödeme yöntemlerini yönetmek için "if-else" veya "switch-case" yapıları kullanmak zorunda kalırdık:

```csharp
// Interface olmadan - "if programcılığı" örneği
public class PaymentServiceWithoutInterfaces
{
    public bool ProcessPayment(string paymentMethod, decimal amount)
    {
        if (paymentMethod == "CreditCard")
        {
            // Kredi kartı işlemi
            CreditCardProcessor processor = new CreditCardProcessor();
            return processor.ProcessCreditCardPayment(amount);
        }
        else if (paymentMethod == "BankTransfer")
        {
            // Banka havalesi işlemi
            BankTransferProcessor processor = new BankTransferProcessor();
            return processor.ProcessBankTransfer(amount);
        }
        else if (paymentMethod == "PayPal")
        {
            // PayPal işlemi
            PayPalProcessor processor = new PayPalProcessor();
            return processor.ProcessPayPalPayment(amount);
        }
        // Yeni bir ödeme yöntemi eklendiğinde, bu metodu değiştirmek gerekir
        // Bu, Open/Closed Principle'a aykırıdır
        
        Console.WriteLine($"Desteklenmeyen ödeme yöntemi: {paymentMethod}");
        return false;
    }
    
    // Benzer şekilde iade işlemi için de if-else yapısı gerekir
    public void RefundPayment(string paymentMethod, string transactionId, decimal amount)
    {
        // Benzer if-else yapısı...
    }
}
```

Bu yaklaşım, aşağıdaki sorunlara yol açar:
- **Spagetti Kod**: Karmaşık, bakımı zor kod yapısı oluşur.
- **Değişime Kapalı**: Yeni bir ödeme yöntemi eklemek için mevcut kodu değiştirmek gerekir.
- **Test Edilebilirlik Sorunu**: Bağımlılıkları taklit etmek (mock) zordur.
- **Tekrar Eden Kod**: Benzer mantık farklı yerlerde tekrarlanır.

Interface'ler, bu sorunları çözerek kodun daha sürdürülebilir ve genişletilebilir olmasını sağlar.

## 2. Explicit Interface Implementation

Bazen bir sınıf, aynı imzaya sahip metotları olan birden fazla interface'i uygulamak zorunda kalabilir. Bu durumda, explicit interface implementation kullanılabilir.

### Explicit Implementation Örneği

```csharp
// Dosya işlemleri arayüzü
public interface IFileOperations
{
    void Save(string data);
    string Load();
    void Delete();
}

// Veritabanı işlemleri arayüzü
public interface IDatabaseOperations
{
    void Save(string data); // IFileOperations ile aynı imza
    string Load(); // IFileOperations ile aynı imza
    void Backup();
}

// Her iki arayüzü de uygulayan sınıf
public class DataManager : IFileOperations, IDatabaseOperations
{
    private string _filePath = "data.txt";
    private string _connectionString = "Server=myServerAddress;Database=myDataBase;";
    
    // IFileOperations için explicit implementation
    void IFileOperations.Save(string data)
    {
        Console.WriteLine($"Dosyaya kaydediliyor: {_filePath}");
        File.WriteAllText(_filePath, data);
    }
    
    string IFileOperations.Load()
    {
        Console.WriteLine($"Dosyadan yükleniyor: {_filePath}");
        return File.Exists(_filePath) ? File.ReadAllText(_filePath) : string.Empty;
    }
    
    void IFileOperations.Delete()
    {
        Console.WriteLine($"Dosya siliniyor: {_filePath}");
        if (File.Exists(_filePath))
            File.Delete(_filePath);
    }
    
    // IDatabaseOperations için explicit implementation
    void IDatabaseOperations.Save(string data)
    {
        Console.WriteLine($"Veritabanına kaydediliyor: {_connectionString}");
        // Veritabanı kaydetme işlemi
    }
    
    string IDatabaseOperations.Load()
    {
        Console.WriteLine($"Veritabanından yükleniyor: {_connectionString}");
        // Veritabanından yükleme işlemi
        return "Database data";
    }
    
    void IDatabaseOperations.Backup()
    {
        Console.WriteLine("Veritabanı yedekleniyor...");
        // Veritabanı yedekleme işlemi
    }
}
```

### Explicit Implementation Kullanımı

```csharp
// DataManager kullanımı
DataManager manager = new DataManager();

// Explicit implementation'a erişmek için cast gerekir
IFileOperations fileOps = manager;
fileOps.Save("Dosya verisi");
string fileData = fileOps.Load();

IDatabaseOperations dbOps = manager;
dbOps.Save("Veritabanı verisi");
string dbData = dbOps.Load();
dbOps.Backup();

// Doğrudan erişim mümkün değildir
// manager.Save("data"); // Derleme hatası
```

## 3. IDisposable ve using Statement

`IDisposable` interface'i, yönetilmeyen kaynakları (dosya tanıtıcıları, veritabanı bağlantıları, ağ soketleri vb.) düzgün bir şekilde serbest bırakmak için kullanılır.

### IDisposable Implementasyonu

```csharp
// Veritabanı bağlantı yöneticisi
public class DatabaseConnection : IDisposable
{
    private SqlConnection _connection;
    private bool _disposed = false;
    
    public DatabaseConnection(string connectionString)
    {
        _connection = new SqlConnection(connectionString);
        _connection.Open();
        Console.WriteLine("Veritabanı bağlantısı açıldı.");
    }
    
    public void ExecuteQuery(string query)
    {
        if (_disposed)
            throw new ObjectDisposedException(nameof(DatabaseConnection));
            
        Console.WriteLine($"Sorgu çalıştırılıyor: {query}");
        // Sorgu çalıştırma işlemi
    }
    
    // IDisposable implementasyonu
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    // Dispose pattern
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Yönetilen kaynakları temizle
                if (_connection != null)
                {
                    _connection.Close();
                    _connection.Dispose();
                    _connection = null;
                    Console.WriteLine("Veritabanı bağlantısı kapatıldı.");
                }
            }
            
            // Yönetilmeyen kaynakları temizle (bu örnekte yok)
            
            _disposed = true;
        }
    }
    
    // Finalizer
    ~DatabaseConnection()
    {
        Dispose(false);
    }
}
```

### using Statement Kullanımı

```csharp
// using bloğu ile otomatik Dispose çağrısı
public void QueryDatabase()
{
    using (var connection = new DatabaseConnection("connection_string"))
    {
        connection.ExecuteQuery("SELECT * FROM Users");
        // using bloğu sonunda Dispose otomatik çağrılır
    }
    
    // C# 8.0+ using ifadesi
    using var connection2 = new DatabaseConnection("connection_string");
    connection2.ExecuteQuery("SELECT * FROM Products");
    // Metot sonunda Dispose otomatik çağrılır
}
```

## 4. Default Interface Methods (C# 8.0+)

C# 8.0 ile birlikte, interface'lerde varsayılan implementasyon sağlama özelliği gelmiştir. Bu, mevcut interface'lere geriye dönük uyumluluk sağlayarak yeni metotlar eklemeyi kolaylaştırır.

### Default Interface Methods Örneği

```csharp
// Loglama arayüzü
public interface ILogger
{
    void Log(string message);
    
    // Default implementasyon
    void LogError(string message)
    {
        Log($"ERROR: {message}");
    }
    
    void LogWarning(string message)
    {
        Log($"WARNING: {message}");
    }
    
    void LogInfo(string message)
    {
        Log($"INFO: {message}");
    }
}

// Basit implementasyon - sadece Log metodu uygulanır
public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine($"[{DateTime.Now}] {message}");
    }
    
    // LogError, LogWarning ve LogInfo için default implementasyon kullanılır
}

// Özel implementasyon - bazı metotlar override edilir
public class FileLogger : ILogger
{
    private string _logFilePath;
    
    public FileLogger(string logFilePath)
    {
        _logFilePath = logFilePath;
    }
    
    public void Log(string message)
    {
        File.AppendAllText(_logFilePath, $"[{DateTime.Now}] {message}\n");
    }
    
    // LogError özelleştirildi
    public void LogError(string message)
    {
        Log($"CRITICAL ERROR: {message}");
        // Ek işlemler (e-posta gönderme vb.)
    }
    
    // Diğer metotlar için default implementasyon kullanılır
}
```

### Default Interface Methods Kullanımı

```csharp
// Default metotları kullanma
ILogger consoleLogger = new ConsoleLogger();
consoleLogger.Log("Normal log mesajı");
consoleLogger.LogError("Hata oluştu!"); // Default implementasyon
consoleLogger.LogWarning("Dikkat!"); // Default implementasyon

ILogger fileLogger = new FileLogger("app.log");
fileLogger.Log("Normal log mesajı");
fileLogger.LogError("Hata oluştu!"); // Özelleştirilmiş implementasyon
fileLogger.LogWarning("Dikkat!"); // Default implementasyon
```

## 5. Interface Inheritance

Interface'ler, diğer interface'lerden miras alabilir. Bu, daha özelleştirilmiş interface'ler oluşturmanıza olanak tanır.

### Interface Inheritance Örneği

```csharp
// Temel repository arayüzü
public interface IRepository<T>
{
    void Add(T entity);
    void Update(T entity);
    void Delete(int id);
    T GetById(int id);
    IEnumerable<T> GetAll();
}

// Arama yetenekleri ekleyen arayüz
public interface ISearchableRepository<T> : IRepository<T>
{
    IEnumerable<T> Search(string term);
}

// Sayfalama yetenekleri ekleyen arayüz
public interface IPagedRepository<T> : IRepository<T>
{
    IEnumerable<T> GetPage(int pageNumber, int pageSize);
    int GetTotalPages(int pageSize);
}

// Gelişmiş repository arayüzü - hem arama hem sayfalama yetenekleri
public interface IAdvancedRepository<T> : ISearchableRepository<T>, IPagedRepository<T>
{
    IEnumerable<T> GetPagedAndFiltered(int pageNumber, int pageSize, Func<T, bool> predicate);
}

// Müşteri sınıfı
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string Phone { get; set; }
}

// Gelişmiş müşteri repository implementasyonu
public class CustomerRepository : IAdvancedRepository<Customer>
{
    private List<Customer> _customers = new List<Customer>();
    
    // IRepository<Customer> implementasyonu
    public void Add(Customer entity)
    {
        if (entity.Id == 0)
        {
            entity.Id = _customers.Count > 0 ? _customers.Max(c => c.Id) + 1 : 1;
        }
        
        _customers.Add(entity);
    }
    
    public void Update(Customer entity)
    {
        var index = _customers.FindIndex(c => c.Id == entity.Id);
        if (index >= 0)
        {
            _customers[index] = entity;
        }
    }
    
    public void Delete(int id)
    {
        _customers.RemoveAll(c => c.Id == id);
    }
    
    public Customer GetById(int id)
    {
        return _customers.FirstOrDefault(c => c.Id == id);
    }
    
    public IEnumerable<Customer> GetAll()
    {
        return _customers;
    }
    
    // ISearchableRepository<Customer> implementasyonu
    public IEnumerable<Customer> Search(string term)
    {
        if (string.IsNullOrWhiteSpace(term))
            return _customers;
            
        term = term.ToLower();
        
        return _customers.Where(c => 
            c.Name.ToLower().Contains(term) || 
            c.Email.ToLower().Contains(term) || 
            c.Phone.Contains(term));
    }
    
    // IPagedRepository<Customer> implementasyonu
    public IEnumerable<Customer> GetPage(int pageNumber, int pageSize)
    {
        return _customers
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize);
    }
    
    public int GetTotalPages(int pageSize)
    {
        return (int)Math.Ceiling(_customers.Count / (double)pageSize);
    }
    
    // IAdvancedRepository<Customer> implementasyonu
    public IEnumerable<Customer> GetPagedAndFiltered(int pageNumber, int pageSize, Func<Customer, bool> predicate)
    {
        return _customers
            .Where(predicate)
            .Skip((pageNumber - 1) * pageSize)
            .Take(pageSize);
    }
}
```

### Interface Inheritance Kullanımı

```csharp
// CustomerRepository kullanımı
CustomerRepository repository = new CustomerRepository();

// Müşteri ekle
repository.Add(new Customer { Id = 1, Name = "Ahmet Yılmaz", Email = "ahmet@example.com", Phone = "555-1234" });
repository.Add(new Customer { Id = 2, Name = "Ayşe Demir", Email = "ayse@example.com", Phone = "555-5678" });
repository.Add(new Customer { Id = 3, Name = "Mehmet Kaya", Email = "mehmet@example.com", Phone = "555-9012" });

// Farklı arayüz yeteneklerini kullanma
IRepository<Customer> basicRepo = repository;
Customer customer = basicRepo.GetById(1);
Console.WriteLine($"Müşteri: {customer.Name}");

ISearchableRepository<Customer> searchableRepo = repository;
var searchResults = searchableRepo.Search("demir");
Console.WriteLine($"Arama sonuçları: {searchResults.Count()} müşteri bulundu.");

IPagedRepository<Customer> pagedRepo = repository;
var page1 = pagedRepo.GetPage(1, 2);
Console.WriteLine($"Sayfa 1: {page1.Count()} müşteri gösteriliyor.");

IAdvancedRepository<Customer> advancedRepo = repository;
var filteredPage = advancedRepo.GetPagedAndFiltered(1, 10, c => c.Email.Contains("example.com"));
Console.WriteLine($"Filtrelenmiş sayfa: {filteredPage.Count()} müşteri gösteriliyor.");
```

## En İyi Pratikler

Interface'ler, sürdürülebilir ve genişletilebilir kod yazmanın temel yapı taşlarıdır. Aşağıdaki en iyi pratikler, interface'leri etkili bir şekilde kullanmanıza yardımcı olacaktır:

1. **Interface Tasarımı**
   - Interface'leri küçük ve odaklanmış tutun (Interface Segregation Principle).
   - Bir interface, tek bir sorumluluğa veya ilgili sorumluluklar grubuna odaklanmalıdır.
   - Interface'leri isimlerini "I" ile başlatın (örn. `IRepository`).
   - Interface metotlarını açıklayıcı isimlerle adlandırın.

2. **Dependency Inversion**
   - Somut sınıflar yerine interface'lere bağımlı olun (Dependency Inversion Principle).
   - Bu, kodunuzu test edilebilir ve değiştirilebilir hale getirir.
   - Dependency Injection kullanarak interface bağımlılıklarını yönetin.

   ```csharp
   // Kötü - somut sınıfa bağımlılık
   public class OrderService
   {
       private SqlOrderRepository _repository = new SqlOrderRepository();
       
       public void ProcessOrder(Order order)
       {
           _repository.Save(order);
       }
   }
   
   // İyi - interface'e bağımlılık
   public class OrderService
   {
       private readonly IOrderRepository _repository;
       
       public OrderService(IOrderRepository repository)
       {
           _repository = repository;
       }
       
       public void ProcessOrder(Order order)
       {
           _repository.Save(order);
       }
   }
   ```

3. **Interface Inheritance**
   - Interface'leri mantıklı bir şekilde hiyerarşik olarak düzenleyin.
   - Çok derin interface hiyerarşilerinden kaçının.
   - Ortak işlevselliği temel interface'lerde tanımlayın.

4. **Explicit Implementation**
   - Aynı imzaya sahip metotları olan birden fazla interface'i uygularken explicit implementation kullanın.
   - Explicit implementation, interface'lerin çakışmasını önler ve kodun daha net olmasını sağlar.

5. **IDisposable Kullanımı**
   - Yönetilmeyen kaynakları kullanan sınıflar için `IDisposable` uygulayın.
   - `IDisposable` sınıflarını her zaman `using` bloğu içinde kullanın.
   - Standart dispose pattern'i uygulayın (Dispose(bool) metodu ve finalizer).

6. **Default Interface Methods**
   - Default interface metotlarını, geriye dönük uyumluluk sağlamak veya utility metotları eklemek için kullanın.
   - Default implementasyonları basit tutun ve karmaşık iş mantığı içermemesine dikkat edin.

7. **Interface vs Abstract Class**
   - Çoklu kalıtım gerektiğinde interface'leri tercih edin.
   - Ortak implementasyon paylaşımı gerektiğinde abstract sınıfları düşünün.
   - Bazen ikisinin kombinasyonu en iyi çözüm olabilir.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Şişman Interface'ler**
   ```csharp
   // Kötü - çok fazla sorumluluğa sahip interface
   public interface IRepository
   {
       void Save(object entity);
       object GetById(int id);
       IEnumerable<object> GetAll();
       void Delete(int id);
       void SendEmail(string to, string subject);
       void GenerateReport();
       void ProcessPayment(decimal amount);
   }
   
   // İyi - ayrılmış sorumluluklar
   public interface IRepository
   {
       void Save(object entity);
       object GetById(int id);
       IEnumerable<object> GetAll();
       void Delete(int id);
   }
   
   public interface IEmailSender
   {
       void SendEmail(string to, string subject);
   }
   
   public interface IReportGenerator
   {
       void GenerateReport();
   }
   
   public interface IPaymentProcessor
   {
       void ProcessPayment(decimal amount);
   }
   ```

2. **Interface'leri Gereksiz Yere Kullanmak**
   ```csharp
   // Kötü - gereksiz interface
   public interface ILogger
   {
       void Log(string message);
   }
   
   public class Logger : ILogger
   {
       public void Log(string message)
       {
           Console.WriteLine(message);
       }
   }
   
   // Tek bir implementasyon varsa ve test/genişletme planı yoksa,
   // interface gereksiz olabilir
   ```

3. **Interface'leri Sürekli Değiştirmek**
   ```csharp
   // Kötü - sürekli değişen interface
   public interface IUserService
   {
       User GetUser(int id);
       void CreateUser(User user);
       // Sonradan eklenen metotlar - mevcut implementasyonları bozar
       void UpdateUser(User user);
       void DeleteUser(int id);
       IEnumerable<User> SearchUsers(string term);
   }
   
   // İyi - genişletilebilir tasarım
   public interface IUserService
   {
       User GetUser(int id);
       void CreateUser(User user);
   }
   
   // Yeni yetenekler için yeni interface'ler
   public interface IUserManagementService : IUserService
   {
       void UpdateUser(User user);
       void DeleteUser(int id);
   }
   
   public interface IUserSearchService
   {
       IEnumerable<User> SearchUsers(string term);
   }
   ```

4. **Explicit Implementation'ı Yanlış Kullanmak**
   ```csharp
   // Kötü - gereksiz explicit implementation
   public interface ILogger
   {
       void Log(string message);
   }
   
   public class FileLogger : ILogger
   {
       // Gereksiz explicit implementation
       void ILogger.Log(string message)
       {
           File.AppendAllText("log.txt", message + Environment.NewLine);
       }
   }
   
   // İyi - çakışma olmadığında normal implementation
   public class FileLogger : ILogger
   {
       public void Log(string message)
       {
           File.AppendAllText("log.txt", message + Environment.NewLine);
       }
   }
   
   // İyi - çakışma olduğunda explicit implementation
   public interface IFileLogger
   {
       void Log(string message);
   }
   
   public interface IConsoleLogger
   {
       void Log(string message);
   }
   
   public class CombinedLogger : IFileLogger, IConsoleLogger
   {
       // Çakışan metotlar için explicit implementation
       void IFileLogger.Log(string message)
       {
           File.AppendAllText("log.txt", message + Environment.NewLine);
       }
       
       void IConsoleLogger.Log(string message)
       {
           Console.WriteLine(message);
       }
       
       // Genel log metodu
       public void Log(string message)
       {
           ((IFileLogger)this).Log(message);
           ((IConsoleLogger)this).Log(message);
       }
   }
   ```

5. **IDisposable'ı Yanlış Uygulamak**
   ```csharp
   // Kötü - eksik IDisposable implementasyonu
   public class ResourceManager : IDisposable
   {
       private FileStream _fileStream;
       
       public ResourceManager(string filePath)
       {
           _fileStream = File.OpenRead(filePath);
       }
       
       public void Dispose()
       {
           _fileStream.Dispose(); // Sadece bu yeterli değil
       }
   }
   
   // İyi - tam IDisposable implementasyonu
   public class ResourceManager : IDisposable
   {
       private FileStream _fileStream;
       private bool _disposed = false;
       
       public ResourceManager(string filePath)
       {
           _fileStream = File.OpenRead(filePath);
       }
       
       public void Dispose()
       {
           Dispose(true);
           GC.SuppressFinalize(this);
       }
       
       protected virtual void Dispose(bool disposing)
       {
           if (!_disposed)
           {
               if (disposing)
               {
                   // Yönetilen kaynakları temizle
                   _fileStream?.Dispose();
               }
               
               // Yönetilmeyen kaynakları temizle (varsa)
               
               _disposed = true;
           }
       }
       
       ~ResourceManager()
       {
           Dispose(false);
       }
   }
   ```

Interface'ler, C# programlamanın en güçlü özelliklerinden biridir. Doğru kullanıldığında, kodunuzu daha modüler, test edilebilir, bakımı kolay ve genişletilebilir hale getirir. Interface'lerin sağladığı soyutlama ve gevşek bağlantı (loose coupling), yazılım geliştirme sürecinde esneklik ve dayanıklılık sağlar.