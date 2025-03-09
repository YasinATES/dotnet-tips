# Kapsülleme (Encapsulation)

Kapsülleme, nesne yönelimli programlamanın temel prensiplerinden biridir ve bir sınıfın iç detaylarını gizleyerek, dış dünyaya sadece gerekli arayüzü sunma prensibidir. Bu bölümde, C#'ta kapsülleme kavramını ve ilgili konuları detaylı olarak inceleyeceğiz.

## 1. Access Modifiers (Erişim Belirleyiciler)

Erişim belirleyiciler, sınıf üyelerine (alanlar, metotlar, özellikler) erişimi kontrol etmek için kullanılır.

C# ta oluşturulan her sınıf, varsayılan olarak `internal` erişim belirleyicisi ile oluşturulur.

### Temel Erişim Belirleyiciler

```csharp
public class BankAccount
{
    // private: Sadece tanımlandığı sınıf içinden erişilebilir
    private string _accountNumber;
    private decimal _balance;
    private readonly decimal _minimumBalance;
    private List<Transaction> _transactions;
    
    // public: Her yerden erişilebilir
    public string AccountHolderName { get; set; }
    
    // protected: Tanımlandığı sınıf ve alt sınıflardan erişilebilir
    protected DateTime DateOpened { get; private set; }
    
    // internal: Aynı assembly içinden erişilebilir
    internal string BranchCode { get; set; }
    
    // protected internal: Aynı assembly veya alt sınıflardan erişilebilir
    protected internal bool IsActive { get; set; }
    
    // private protected (C# 7.2+): Aynı assembly içindeki alt sınıflardan erişilebilir
    private protected void UpdateAccountStatus() 
    {
        // İç işlemler...
    }
    
    public BankAccount(string accountNumber, string accountHolderName, decimal initialDeposit)
    {
        if (string.IsNullOrWhiteSpace(accountNumber))
            throw new ArgumentException("Hesap numarası boş olamaz.");
            
        if (string.IsNullOrWhiteSpace(accountHolderName))
            throw new ArgumentException("Hesap sahibi adı boş olamaz.");
            
        if (initialDeposit < 0)
            throw new ArgumentException("Başlangıç bakiyesi negatif olamaz.");
            
        _accountNumber = accountNumber;
        AccountHolderName = accountHolderName;
        _balance = initialDeposit;
        _minimumBalance = 100; // Minimum bakiye
        _transactions = new List<Transaction>();
        DateOpened = DateTime.Now;
        IsActive = true;
        BranchCode = "HQ"; // Varsayılan şube kodu
        
        // İlk işlemi kaydet
        AddTransaction(TransactionType.Deposit, initialDeposit, "Hesap açılış bakiyesi");
    }
    
    // Public metotlar - dış dünyaya sunulan arayüz
    public bool Deposit(decimal amount, string description = "Para yatırma")
    {
        if (amount <= 0)
            throw new ArgumentException("Yatırılan miktar pozitif olmalıdır.");
            
        if (!IsActive)
            return false;
            
        _balance += amount;
        AddTransaction(TransactionType.Deposit, amount, description);
        return true;
    }
    
    public bool Withdraw(decimal amount, string description = "Para çekme")
    {
        if (amount <= 0)
            throw new ArgumentException("Çekilen miktar pozitif olmalıdır.");
            
        if (!IsActive)
            return false;
            
        if (_balance - amount < _minimumBalance)
            return false; // Minimum bakiye koruması
            
        _balance -= amount;
        AddTransaction(TransactionType.Withdrawal, amount, description);
        return true;
    }
    
    public decimal GetBalance()
    {
        return _balance;
    }
    
    public string GetAccountNumber()
    {
        // Hesap numarasının son 4 hanesini göster, diğerlerini maskele
        return "XXXX-XXXX-" + _accountNumber.Substring(_accountNumber.Length - 4);
    }
    
    public IEnumerable<Transaction> GetTransactionHistory()
    {
        // Sadece okuma amaçlı işlem geçmişi döndür
        return _transactions.AsReadOnly();
    }
    
    // Private metotlar - iç implementasyon detayları
    private void AddTransaction(TransactionType type, decimal amount, string description)
    {
        var transaction = new Transaction
        {
            Type = type,
            Amount = amount,
            Description = description,
            Date = DateTime.Now,
            Balance = _balance
        };
        
        _transactions.Add(transaction);
    }
    
    // İç sınıf - Transaction
    public class Transaction
    {
        public TransactionType Type { get; set; }
        public decimal Amount { get; set; }
        public string Description { get; set; }
        public DateTime Date { get; set; }
        public decimal Balance { get; set; } // İşlem sonrası bakiye
        
        public override string ToString()
        {
            return $"{Date:yyyy-MM-dd HH:mm:ss} | {Type} | {Amount:C} | {Description} | Bakiye: {Balance:C}";
        }
    }
    
    // Enum - TransactionType
    public enum TransactionType
    {
        Deposit,
        Withdrawal,
        Transfer,
        Fee,
        Interest
    }
}
```

### Erişim Belirleyicilerin Kullanımı

```csharp
// BankAccount sınıfını kullanma
BankAccount account = new BankAccount("1234567890", "Ahmet Yılmaz", 1000);

// Public üyelere erişim
Console.WriteLine($"Hesap Sahibi: {account.AccountHolderName}");
account.Deposit(500, "Maaş yatırma");
Console.WriteLine($"Güncel Bakiye: {account.GetBalance():C}");

// Private üyelere doğrudan erişilemez
// account._balance = 1000000; // Derleme hatası

// Protected üyelere dışarıdan erişilemez
// DateTime opened = account.DateOpened; // Derleme hatası

// Internal üyelere aynı assembly içinden erişilebilir
account.BranchCode = "BR001";

// Hesap işlem geçmişini görüntüleme
var transactions = account.GetTransactionHistory();
foreach (var transaction in transactions)
{
    Console.WriteLine(transaction);
}
```

## 2. Properties ve Auto-Implemented Properties

Properties (özellikler), sınıf alanlarına kontrollü erişim sağlar ve veri doğrulama, hesaplama gibi işlemleri gerçekleştirebilir.

### Geleneksel Properties

```csharp
public class Customer
{
    // Private alanlar
    private string _firstName;
    private string _lastName;
    private string _email;
    private DateTime _dateOfBirth;
    private int _loyaltyPoints;
    
    // Geleneksel property - veri doğrulama ile
    public string FirstName
    {
        get { return _firstName; }
        set 
        { 
            if (string.IsNullOrWhiteSpace(value))
                throw new ArgumentException("İsim boş olamaz.");
                
            _firstName = value; 
        }
    }
    
    public string LastName
    {
        get { return _lastName; }
        set 
        { 
            if (string.IsNullOrWhiteSpace(value))
                throw new ArgumentException("Soyisim boş olamaz.");
                
            _lastName = value; 
        }
    }
    
    // Hesaplanmış property
    public string FullName
    {
        get { return $"{FirstName} {LastName}"; }
    }
    
    // Veri doğrulama ile property
    public string Email
    {
        get { return _email; }
        set
        {
            if (string.IsNullOrWhiteSpace(value))
                throw new ArgumentException("E-posta boş olamaz.");
                
            if (!value.Contains("@") || !value.Contains("."))
                throw new ArgumentException("Geçersiz e-posta formatı.");
                
            _email = value;
        }
    }
    
    // Private setter ile property
    public DateTime DateOfBirth
    {
        get { return _dateOfBirth; }
        private set { _dateOfBirth = value; }
    }
    
    // Salt okunur hesaplanmış property
    public int Age
    {
        get
        {
            var today = DateTime.Today;
            var age = today.Year - DateOfBirth.Year;
            
            if (DateOfBirth.Date > today.AddYears(-age))
                age--;
                
            return age;
        }
    }
    
    // Property ile iş mantığı
    public int LoyaltyPoints
    {
        get { return _loyaltyPoints; }
    }
    
    public void AddLoyaltyPoints(int points)
    {
        if (points <= 0)
            throw new ArgumentException("Eklenecek puan pozitif olmalıdır.");
            
        _loyaltyPoints += points;
        
        // Sadakat puanı 1000'i geçtiğinde müşteriye bildirim gönder
        if (_loyaltyPoints >= 1000 && (_loyaltyPoints - points) < 1000)
        {
            SendLoyaltyMilestoneNotification();
        }
    }
    
    private void SendLoyaltyMilestoneNotification()
    {
        Console.WriteLine($"Tebrikler {FullName}! 1000 sadakat puanını aştınız.");
    }
    
    // Constructor
    public Customer(string firstName, string lastName, string email, DateTime dateOfBirth)
    {
        FirstName = firstName; // Property setter'ı çağrılır (veri doğrulama yapılır)
        LastName = lastName;   // Property setter'ı çağrılır
        Email = email;         // Property setter'ı çağrılır
        DateOfBirth = dateOfBirth;
        _loyaltyPoints = 0;
    }
}
```

### Auto-Implemented Properties

```csharp
public class Product
{
    // Auto-implemented property
    public int Id { get; set; }
    
    // Auto-implemented property with default value (C# 6.0+)
    public string Category { get; set; } = "Uncategorized";
    
    // Auto-implemented property with private setter
    public string Name { get; private set; }
    
    // Auto-implemented property with init accessor (C# 9.0+)
    public string SKU { get; init; }
    
    // Read-only auto-implemented property
    public DateTime CreatedDate { get; } = DateTime.Now;
    
    // Constructor
    public Product(string name, string sku)
    {
        Name = name;
        SKU = sku;
    }
    
    // Name property'sini değiştiren metot
    public void UpdateName(string newName)
    {
        if (string.IsNullOrWhiteSpace(newName))
            throw new ArgumentException("Ürün adı boş olamaz.");
            
        Name = newName;
    }
}
```

## 3. Indexers

Indexer'lar, bir sınıfın dizi veya koleksiyon gibi indekslenebilir olmasını sağlar.

### Temel Indexer Kullanımı

```csharp
public class Inventory
{
    private Dictionary<string, Product> _products = new Dictionary<string, Product>();
    
    // String key ile indexer
    public Product this[string sku]
    {
        get 
        {
            if (_products.TryGetValue(sku, out var product))
                return product;
                
            throw new KeyNotFoundException($"SKU: {sku} bulunamadı.");
        }
        set
        {
            if (string.IsNullOrWhiteSpace(sku))
                throw new ArgumentException("SKU boş olamaz.");
                
            _products[sku] = value;
        }
    }
    
    // Int index ile indexer
    public Product this[int index]
    {
        get
        {
            if (index < 0 || index >= _products.Count)
                throw new IndexOutOfRangeException("Geçersiz indeks.");
                
            return _products.Values.ElementAt(index);
        }
    }
    
    // Çoklu parametre ile indexer
    public Product this[string category, int index]
    {
        get
        {
            var categoryProducts = _products.Values
                .Where(p => p.Category == category)
                .ToList();
                
            if (index < 0 || index >= categoryProducts.Count)
                throw new IndexOutOfRangeException("Geçersiz indeks.");
                
            return categoryProducts[index];
        }
    }
    
    // Ürün sayısı
    public int Count => _products.Count;
    
    // Ürün ekleme metodu
    public void AddProduct(Product product)
    {
        if (product == null)
            throw new ArgumentNullException(nameof(product));
            
        _products[product.SKU] = product;
    }
}
```

### Indexer Kullanımı

```csharp
// Inventory sınıfını kullanma
Inventory inventory = new Inventory();

// Ürünler oluşturma
Product laptop = new Product("Laptop", "TECH001") { Category = "Electronics", Id = 1 };
Product phone = new Product("Smartphone", "TECH002") { Category = "Electronics", Id = 2 };
Product book = new Product("C# Programming", "BOOK001") { Category = "Books", Id = 3 };

// Ürünleri envantere ekleme
inventory.AddProduct(laptop);
inventory.AddProduct(phone);
inventory.AddProduct(book);

// Indexer ile ürünlere erişim
Product product1 = inventory["TECH001"]; // SKU ile erişim
Console.WriteLine($"Ürün: {product1.Name}, Kategori: {product1.Category}");

Product product2 = inventory[1]; // İndeks ile erişim
Console.WriteLine($"Ürün: {product2.Name}, Kategori: {product2.Category}");

Product product3 = inventory["Electronics", 0]; // Kategori ve indeks ile erişim
Console.WriteLine($"Ürün: {product3.Name}, SKU: {product3.SKU}");

// Indexer ile ürün güncelleme
inventory["TECH001"] = new Product("Gaming Laptop", "TECH001") { Category = "Electronics", Id = 1 };
```

## 4. Explicit Interface Implementation

Bir sınıf, birden fazla interface'i uygulayabilir ve aynı isimde metotları olabilir. Bu durumda, explicit interface implementation kullanılabilir.

### Explicit Interface Implementation Örneği

```csharp
// Loglama interface'i
public interface ILogger
{
    void Log(string message);
    void LogError(string error);
}

// Dosya işlemleri interface'i
public interface IFileHandler
{
    void Save(string content);
    string Load();
    void Log(string operation); // ILogger.Log ile aynı isimde
}

// Her iki interface'i de uygulayan sınıf
public class FileLogger : ILogger, IFileHandler
{
    private string _logFilePath;
    private string _dataFilePath;
    
    public FileLogger(string logFilePath, string dataFilePath)
    {
        _logFilePath = logFilePath;
        _dataFilePath = dataFilePath;
    }
    
    // ILogger.Log için explicit implementation
    void ILogger.Log(string message)
    {
        File.AppendAllText(_logFilePath, $"[INFO] {DateTime.Now}: {message}{Environment.NewLine}");
    }
    
    // ILogger.LogError için normal implementation
    public void LogError(string error)
    {
        File.AppendAllText(_logFilePath, $"[ERROR] {DateTime.Now}: {error}{Environment.NewLine}");
    }
    
    // IFileHandler.Log için explicit implementation
    void IFileHandler.Log(string operation)
    {
        File.AppendAllText(_logFilePath, $"[OPERATION] {DateTime.Now}: {operation}{Environment.NewLine}");
    }
    
    // IFileHandler.Save için normal implementation
    public void Save(string content)
    {
        File.WriteAllText(_dataFilePath, content);
        ((IFileHandler)this).Log($"Saved content to {_dataFilePath}");
    }
    
    // IFileHandler.Load için normal implementation
    public string Load()
    {
        if (!File.Exists(_dataFilePath))
            return string.Empty;
            
        string content = File.ReadAllText(_dataFilePath);
        ((IFileHandler)this).Log($"Loaded content from {_dataFilePath}");
        return content;
    }
    
    // Her iki Log metodu için genel bir metot
    public void LogMessage(string message, bool isError = false)
    {
        if (isError)
        {
            LogError(message);
        }
        else
        {
            ((ILogger)this).Log(message);
        }
    }
}
```

### Explicit Interface Implementation Kullanımı

```csharp
// FileLogger sınıfını kullanma
FileLogger fileLogger = new FileLogger("app.log", "data.txt");

// Normal metotlara erişim
fileLogger.LogError("Bir hata oluştu!");
fileLogger.Save("Örnek içerik");
string content = fileLogger.Load();

// Explicit implementation metotlarına erişim için cast gerekir
((ILogger)fileLogger).Log("Bilgi mesajı");
((IFileHandler)fileLogger).Log("Dosya işlemi");

// Interface referansı ile kullanım
ILogger logger = fileLogger;
logger.Log("Interface referansı ile log"); // ILogger.Log çağrılır

IFileHandler fileHandler = fileLogger;
fileHandler.Log("Interface referansı ile dosya log"); // IFileHandler.Log çağrılır

// Genel metot kullanımı
fileLogger.LogMessage("Normal mesaj");
fileLogger.LogMessage("Hata mesajı", true);
```

## En İyi Pratikler

1. **Erişim Belirleyicileri**
   - Sınıf üyelerini mümkün olan en kısıtlayıcı erişim belirleyicisiyle tanımlayın.
   - Sınıfın iç durumunu koruyan alanları `private` olarak işaretleyin.
   - Sadece gerçekten gerekli olan üyeleri `public` yapın.
   - Alt sınıfların erişmesi gereken ama dış dünyadan gizlenmesi gereken üyeler için `protected` kullanın.

2. **Properties**
   - Veri doğrulama ve iş kurallarını property'lerin setter'larında uygulayın.
   - Hesaplanmış değerler için salt okunur property'ler kullanın.
   - Sadece sınıf içinden değiştirilebilecek property'ler için `private set` kullanın.
   - Basit property'ler için auto-implemented property'leri tercih edin.

3. **Indexers**
   - Koleksiyon benzeri sınıflar için indexer'ları kullanın.
   - Farklı parametre tipleriyle overload edilmiş indexer'lar sağlayın.
   - Indexer'larda geçersiz indeks kontrolü yapın.

4. **Interface Implementation**
   - Aynı isimde metotları olan birden fazla interface'i uygularken explicit implementation kullanın.
   - Interface metotlarını explicit olarak uyguladığınızda, gerekirse bu metotlara erişim sağlayan public metotlar ekleyin.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Yetersiz Kapsülleme**
   ```csharp
   // Kötü - public alanlar
   public class User
   {
       public string Username;
       public string Password;
       public int LoginAttempts;
   }
   
   // İyi - kapsüllenmiş veriler
   public class User
   {
       private string _username;
       private string _password;
       private int _loginAttempts;
       
       public string Username
       {
           get { return _username; }
           set
           {
               if (string.IsNullOrWhiteSpace(value))
                   throw new ArgumentException("Kullanıcı adı boş olamaz.");
                   
               _username = value;
           }
       }
       
       // Password için sadece set property - güvenlik için
       public string Password
       {
           set
           {
               if (string.IsNullOrWhiteSpace(value))
                   throw new ArgumentException("Şifre boş olamaz.");
                   
               if (value.Length < 8)
                   throw new ArgumentException("Şifre en az 8 karakter olmalıdır.");
                   
               _password = HashPassword(value);
           }
       }
       
       // Salt okunur property
       public bool IsLocked => _loginAttempts >= 3;
       
       private string HashPassword(string password)
       {
           // Şifre hash'leme işlemi...
           return "hashed_" + password;
       }
       
       public bool ValidatePassword(string password)
       {
           if (HashPassword(password) == _password)
           {
               _loginAttempts = 0;
               return true;
           }
           
           _loginAttempts++;
           return false;
       }
   }
   ```

2. **Property'lerde Yan Etkiler**
   ```csharp
   // Kötü - property'de yan etki
   public class Order
   {
       private List<OrderItem> _items = new List<OrderItem>();
       
       public decimal TotalAmount
       {
           get
           {
               SendOrderUpdateNotification(); // Yan etki!
               return _items.Sum(i => i.LineTotal);
           }
       }
       
       private void SendOrderUpdateNotification()
       {
           // E-posta gönderme, log kaydetme vb.
       }
   }
   
   // İyi - yan etkisiz property
   public class Order
   {
       private List<OrderItem> _items = new List<OrderItem>();
       
       public decimal TotalAmount => _items.Sum(i => i.LineTotal);
       
       public void UpdateOrder()
       {
           // İş mantığı...
           SendOrderUpdateNotification();
       }
       
       private void SendOrderUpdateNotification()
       {
           // E-posta gönderme, log kaydetme vb.
       }
   }
   ```

3. **Gereksiz Explicit Interface Implementation**
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
   
   // İyi - normal implementation
   public class FileLogger : ILogger
   {
       public void Log(string message)
       {
           File.AppendAllText("log.txt", message + Environment.NewLine);
       }
   }
   ```

4. **Indexer'ların Yanlış Kullanımı**
   ```csharp
   // Kötü - indexer yerine metot kullanımı
   public class DataStore
   {
       private Dictionary<string, object> _data = new Dictionary<string, object>();
       
       public object GetItem(string key)
       {
           return _data.ContainsKey(key) ? _data[key] : null;
       }
       
       public void SetItem(string key, object value)
       {
           _data[key] = value;
       }
   }
   
   // İyi - indexer kullanımı
   public class DataStore
   {
       private Dictionary<string, object> _data = new Dictionary<string, object>();
       
       public object this[string key]
       {
           get { return _data.ContainsKey(key) ? _data[key] : null; }
           set { _data[key] = value; }
       }
   }
   ```

5. **Auto-Implemented Property'lerde Veri Doğrulama Eksikliği**
   ```csharp
   // Kötü - veri doğrulama eksikliği
   public class Product
   {
       public string Name { get; set; }
       public decimal Price { get; set; }
   }
   
   // İyi - veri doğrulama ile
   public class Product
   {
       private string _name;
       private decimal _price;
       
       public string Name
       {
           get => _name;
           set
           {
               if (string.IsNullOrWhiteSpace(value))
                   throw new ArgumentException("Ürün adı boş olamaz.");
               
               _name = value;
           }
       }
       
       public decimal Price
       {
           get => _price;
           set
           {
               if (value < 0)
                   throw new ArgumentException("Fiyat negatif olamaz.");
               
               _price = value;
           }
       }
   }
   ```

Kapsülleme, C# programlamanın temel prensiplerinden biridir ve doğru kullanıldığında, kodunuzu daha güvenli, bakımı kolay ve genişletilebilir hale getirir. Erişim belirleyicileri, property'ler, indexer'lar ve explicit interface implementation gibi mekanizmalar, etkili kapsülleme sağlamanıza yardımcı olur. 