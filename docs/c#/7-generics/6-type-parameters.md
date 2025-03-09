# Type Parameters (Tip Parametreleri)

C#'ta generic tip parametreleri, farklı veri tipleriyle çalışabilen esnek ve yeniden kullanılabilir kod yazmanıza olanak tanır. Bu bölümde, generic tip parametrelerinin özelliklerini ve kullanım şekillerini inceleyeceğiz.

## 1. Default Keyword

`default` anahtar kelimesi, bir tip parametresinin varsayılan değerini elde etmek için kullanılır. Değer tipleri için sıfır, referans tipleri için `null` değerini döndürür.

### Default Keyword Kullanımı

```csharp
// Default anahtar kelimesinin kullanımı
public class DefaultValueExample<T>
{
    // Tip parametresinin varsayılan değerini döndüren metot
    public T GetDefaultValue()
    {
        return default(T);
    }
    
    // C# 7.1 ve sonrasında kısaltılmış syntax
    public T GetDefaultValueShorthand()
    {
        return default;
    }
    
    // Varsayılan değer kontrolü
    public bool IsDefault(T value)
    {
        return EqualityComparer<T>.Default.Equals(value, default(T));
    }
}
```

### Kullanım Örneği

```csharp
// Veri erişim katmanı için generic repository
public class Repository<T> where T : class
{
    private readonly DbContext _context;
    private readonly DbSet<T> _dbSet;
    
    public Repository(DbContext context)
    {
        _context = context;
        _dbSet = context.Set<T>();
    }
    
    // ID'ye göre varlık getirme
    public T GetById(int id)
    {
        var entity = _dbSet.Find(id);
        
        // Bulunamazsa varsayılan değer (null) döndür
        if (entity == null)
            return default;
            
        return entity;
    }
    
    // Belirli bir koşulu sağlayan ilk varlığı getirme
    public T FindFirst(Expression<Func<T, bool>> predicate)
    {
        // Koşulu sağlayan varlık yoksa varsayılan değer döndür
        return _dbSet.FirstOrDefault(predicate);
    }
    
    // Varlık ekleme
    public void Add(T entity)
    {
        // Null kontrolü
        if (entity == null || EqualityComparer<T>.Default.Equals(entity, default))
            throw new ArgumentNullException(nameof(entity), "Eklenecek varlık null olamaz.");
            
        _dbSet.Add(entity);
    }
    
    // Değişiklikleri kaydetme
    public int SaveChanges()
    {
        return _context.SaveChanges();
    }
}
```

## 2. Type Parameter Constraints

Tip parametresi kısıtlamaları, generic tiplerin belirli özelliklere sahip olmasını sağlar. Bu, tip parametrelerinin davranışını ve özelliklerini sınırlandırarak daha güvenli ve öngörülebilir kod yazmanıza olanak tanır.

### Kısıtlama Türleri

```csharp
// Tip parametresi kısıtlamaları
public class ConstraintsExample<T> where T : class, new()
{
    // T bir referans tipi olmalı ve parametre almayan bir constructor'a sahip olmalı
    public T CreateInstance()
    {
        return new T();
    }
}

// Birden fazla tip parametresi ve kısıtlama
public class MultipleConstraintsExample<TKey, TValue>
    where TKey : IComparable<TKey>
    where TValue : class, IDisposable
{
    // TKey, IComparable<TKey> arayüzünü uygulamalı
    // TValue, bir referans tipi olmalı ve IDisposable arayüzünü uygulamalı
    
    public void ProcessValue(TValue value)
    {
        // İşlem tamamlandığında kaynakları temizle
        value.Dispose();
    }
    
    public int CompareKeys(TKey key1, TKey key2)
    {
        return key1.CompareTo(key2);
    }
}
```

### Kullanım Örneği

```csharp
// Varlık temel sınıfı
public abstract class Entity
{
    public int Id { get; set; }
    public DateTime CreatedDate { get; set; }
    public DateTime? ModifiedDate { get; set; }
}

// Denetim (audit) işlemleri için generic servis
public class AuditService<T> where T : Entity
{
    private readonly ILogger<AuditService<T>> _logger;
    
    public AuditService(ILogger<AuditService<T>> logger)
    {
        _logger = logger;
    }
    
    // Varlık oluşturma işlemini denetle
    public void AuditCreation(T entity, string username)
    {
        if (entity == null)
            throw new ArgumentNullException(nameof(entity));
            
        entity.CreatedDate = DateTime.UtcNow;
        
        _logger.LogInformation($"Entity created: {typeof(T).Name}, ID: {entity.Id}, User: {username}");
    }
    
    // Varlık güncelleme işlemini denetle
    public void AuditModification(T entity, string username)
    {
        if (entity == null)
            throw new ArgumentNullException(nameof(entity));
            
        entity.ModifiedDate = DateTime.UtcNow;
        
        _logger.LogInformation($"Entity modified: {typeof(T).Name}, ID: {entity.Id}, User: {username}");
    }
}

// Müşteri varlığı
public class Customer : Entity
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
}

// Hesap varlığı
public class Account : Entity
{
    public string AccountNumber { get; set; }
    public decimal Balance { get; set; }
    public int CustomerId { get; set; }
}

// Müşteri servisi
public class CustomerService
{
    private readonly Repository<Customer> _repository;
    private readonly AuditService<Customer> _auditService;
    
    public CustomerService(Repository<Customer> repository, AuditService<Customer> auditService)
    {
        _repository = repository;
        _auditService = auditService;
    }
    
    // Yeni müşteri oluşturma
    public void CreateCustomer(Customer customer, string username)
    {
        _auditService.AuditCreation(customer, username);
        _repository.Add(customer);
        _repository.SaveChanges();
    }
}
```

## 3. Type Parameter Inference

Tip parametresi çıkarımı, C# derleyicisinin metot çağrılarında tip parametrelerini otomatik olarak belirlemesidir. Bu, kodunuzu daha temiz ve okunabilir hale getirir.

### Tip Çıkarımı Örnekleri

```csharp
// Tip parametresi çıkarımı
public class InferenceExample
{
    // Generic metot
    public static T Identity<T>(T value)
    {
        return value;
    }
    
    // Birden fazla tip parametreli generic metot
    public static TResult Convert<TInput, TResult>(TInput input, Func<TInput, TResult> converter)
    {
        return converter(input);
    }
    
    // Kullanım örneği
    public void DemonstrateInference()
    {
        // Açık tip belirtme
        int result1 = Identity<int>(42);
        
        // Tip çıkarımı - derleyici int tipini otomatik olarak çıkarır
        int result2 = Identity(42);
        
        // Açık tip belirtme
        string result3 = Convert<int, string>(42, x => x.ToString());
        
        // Tip çıkarımı - derleyici tipleri otomatik olarak çıkarır
        string result4 = Convert(42, x => x.ToString());
    }
}
```

### Kullanım Örneği

```csharp
// Veri dönüştürme servisi
public class DataConversionService
{
    // Generic dönüştürme metodu
    public TOutput ConvertData<TInput, TOutput>(TInput input, Func<TInput, TOutput> converter)
    {
        if (input == null)
            return default;
            
        try
        {
            return converter(input);
        }
        catch (Exception ex)
        {
            // Dönüştürme hatası
            Console.WriteLine($"Dönüştürme hatası: {ex.Message}");
            return default;
        }
    }
    
    // Koleksiyon dönüştürme metodu
    public IEnumerable<TOutput> ConvertCollection<TInput, TOutput>(
        IEnumerable<TInput> inputCollection,
        Func<TInput, TOutput> converter)
    {
        if (inputCollection == null)
            yield break;
            
        foreach (var item in inputCollection)
        {
            TOutput result = ConvertData(item, converter);
            yield return result;
        }
    }
}

// Kullanım örneği
public class DataProcessingService
{
    private readonly DataConversionService _conversionService;
    
    public DataProcessingService(DataConversionService conversionService)
    {
        _conversionService = conversionService;
    }
    
    public void ProcessCustomerData(List<CustomerDto> customerDtos)
    {
        // Tip çıkarımı ile dönüştürme
        var customers = _conversionService.ConvertCollection(customerDtos, dto => new Customer
        {
            Id = dto.Id,
            FirstName = dto.FirstName,
            LastName = dto.LastName,
            Email = dto.Email
        });
        
        foreach (var customer in customers)
        {
            // Müşteri işlemleri
            Console.WriteLine($"Müşteri işleniyor: {customer.FirstName} {customer.LastName}");
        }
    }
}

// DTO sınıfı
public class CustomerDto
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
}
```

## 4. Generic Method Resolution

Generic metot çözümleme, C# derleyicisinin hangi generic metot uygulamasının çağrılacağını belirleme sürecidir. Bu, özellikle aşırı yüklenmiş (overloaded) metotlar ve tip çıkarımı ile birlikte kullanıldığında önemlidir.

### Metot Çözümleme Örnekleri

```csharp
// Generic metot çözümleme
public class MethodResolutionExample
{
    // Tek tip parametreli metot
    public static void Process<T>(T value)
    {
        Console.WriteLine($"Process<T>(T value) çağrıldı. Tip: {typeof(T).Name}, Değer: {value}");
    }
    
    // İki tip parametreli metot
    public static void Process<T, U>(T first, U second)
    {
        Console.WriteLine($"Process<T, U>(T first, U second) çağrıldı. Tipler: {typeof(T).Name}, {typeof(U).Name}");
    }
    
    // Belirli bir tip için aşırı yüklenmiş metot
    public static void Process(int value)
    {
        Console.WriteLine($"Process(int value) çağrıldı. Değer: {value}");
    }
    
    // Belirli bir tip için generic metot
    public static void Process<T>(List<T> items)
    {
        Console.WriteLine($"Process<T>(List<T> items) çağrıldı. Eleman tipi: {typeof(T).Name}, Eleman sayısı: {items.Count}");
    }
}
```

### Kullanım Örneği

```csharp
// Veri doğrulama servisi
public class ValidationService
{
    // Temel doğrulama metodu
    public ValidationResult Validate<T>(T entity) where T : class
    {
        Console.WriteLine($"Genel doğrulama: {typeof(T).Name}");
        return new ValidationResult { IsValid = true };
    }
    
    // Müşteri için özel doğrulama
    public ValidationResult Validate(Customer customer)
    {
        if (customer == null)
            return new ValidationResult { IsValid = false, ErrorMessage = "Müşteri null olamaz." };
            
        var result = new ValidationResult { IsValid = true };
        
        // E-posta doğrulama
        if (string.IsNullOrEmpty(customer.Email) || !customer.Email.Contains("@"))
        {
            result.IsValid = false;
            result.ErrorMessage = "Geçersiz e-posta adresi.";
        }
        
        Console.WriteLine("Müşteri özel doğrulaması yapıldı.");
        return result;
    }
    
    // Hesap için özel doğrulama
    public ValidationResult Validate(Account account)
    {
        if (account == null)
            return new ValidationResult { IsValid = false, ErrorMessage = "Hesap null olamaz." };
            
        var result = new ValidationResult { IsValid = true };
        
        // Bakiye kontrolü
        if (account.Balance < 0)
        {
            result.IsValid = false;
            result.ErrorMessage = "Hesap bakiyesi negatif olamaz.";
        }
        
        Console.WriteLine("Hesap özel doğrulaması yapıldı.");
        return result;
    }
    
    // Koleksiyon doğrulama
    public ValidationResult Validate<T>(IEnumerable<T> entities) where T : class
    {
        if (entities == null)
            return new ValidationResult { IsValid = false, ErrorMessage = "Koleksiyon null olamaz." };
            
        var result = new ValidationResult { IsValid = true };
        
        foreach (var entity in entities)
        {
            var itemResult = Validate(entity);
            if (!itemResult.IsValid)
            {
                result.IsValid = false;
                result.ErrorMessage = itemResult.ErrorMessage;
                break;
            }
        }
        
        Console.WriteLine($"Koleksiyon doğrulaması yapıldı. Tip: {typeof(T).Name}");
        return result;
    }
}

// Doğrulama sonucu
public class ValidationResult
{
    public bool IsValid { get; set; }
    public string ErrorMessage { get; set; }
}
```

## 5. Type Parameter Performance

Tip parametreleri, doğru kullanıldığında performans avantajları sağlar. Özellikle değer tipleriyle çalışırken, boxing ve unboxing işlemlerini önleyerek performansı artırabilirsiniz.

### Performans Karşılaştırması

```csharp
// Tip parametresi performans karşılaştırması
public class PerformanceComparison
{
    // Generic metot - boxing/unboxing yok
    public static T FindMaxGeneric<T>(T[] items) where T : IComparable<T>
    {
        if (items == null || items.Length == 0)
            throw new ArgumentException("Dizi boş olamaz.");
            
        T max = items[0];
        
        for (int i = 1; i < items.Length; i++)
        {
            if (items[i].CompareTo(max) > 0)
                max = items[i];
        }
        
        return max;
    }
    
    // Non-generic metot - boxing/unboxing var
    public static object FindMaxNonGeneric(IComparable[] items)
    {
        if (items == null || items.Length == 0)
            throw new ArgumentException("Dizi boş olamaz.");
            
        IComparable max = items[0];
        
        for (int i = 1; i < items.Length; i++)
        {
            if (items[i].CompareTo(max) > 0)
                max = items[i];
        }
        
        return max;
    }
    
    // Performans testi
    public static void ComparePerformance(int size)
    {
        // Test verileri
        int[] numbers = new int[size];
        IComparable[] comparables = new IComparable[size];
        
        Random random = new Random();
        for (int i = 0; i < size; i++)
        {
            numbers[i] = random.Next(1000000);
            comparables[i] = numbers[i];
        }
        
        // Generic metot testi
        var sw1 = System.Diagnostics.Stopwatch.StartNew();
        var maxGeneric = FindMaxGeneric(numbers);
        sw1.Stop();
        
        // Non-generic metot testi
        var sw2 = System.Diagnostics.Stopwatch.StartNew();
        var maxNonGeneric = FindMaxNonGeneric(comparables);
        sw2.Stop();
        
        Console.WriteLine($"Generic metot: {sw1.ElapsedMilliseconds} ms");
        Console.WriteLine($"Non-generic metot: {sw2.ElapsedMilliseconds} ms");
    }
}
```

### Kullanım Örneği

```csharp
// Yüksek performanslı veri işleme servisi
public class DataProcessingService<T>
{
    private readonly List<T> _data;
    
    public DataProcessingService()
    {
        _data = new List<T>();
    }
    
    // Veri ekleme
    public void AddData(T item)
    {
        _data.Add(item);
    }
    
    // Toplu veri ekleme
    public void AddBulkData(IEnumerable<T> items)
    {
        _data.AddRange(items);
    }
    
    // Veri filtreleme
    public IEnumerable<T> Filter(Predicate<T> predicate)
    {
        for (int i = 0; i < _data.Count; i++)
        {
            if (predicate(_data[i]))
                yield return _data[i];
        }
    }
    
    // Veri dönüştürme
    public IEnumerable<TResult> Transform<TResult>(Func<T, TResult> transformer)
    {
        for (int i = 0; i < _data.Count; i++)
        {
            yield return transformer(_data[i]);
        }
    }
    
    // Veri toplama (sayısal veriler için)
    public TResult Aggregate<TResult>(TResult seed, Func<TResult, T, TResult> func)
    {
        TResult result = seed;
        
        for (int i = 0; i < _data.Count; i++)
        {
            result = func(result, _data[i]);
        }
        
        return result;
    }
}

// Kullanım örneği
public class FinancialAnalysisService
{
    private readonly DataProcessingService<Transaction> _transactionProcessor;
    
    public FinancialAnalysisService()
    {
        _transactionProcessor = new DataProcessingService<Transaction>();
    }
    
    // İşlemleri yükleme
    public void LoadTransactions(IEnumerable<Transaction> transactions)
    {
        _transactionProcessor.AddBulkData(transactions);
    }
    
    // Toplam işlem tutarını hesaplama
    public decimal CalculateTotalAmount()
    {
        return _transactionProcessor.Aggregate(0m, (total, transaction) => total + transaction.Amount);
    }
    
    // Belirli bir kategorideki işlemleri filtreleme
    public IEnumerable<Transaction> GetTransactionsByCategory(string category)
    {
        return _transactionProcessor.Filter(t => t.Category == category);
    }
    
    // İşlem özetleri oluşturma
    public IEnumerable<TransactionSummary> CreateTransactionSummaries()
    {
        return _transactionProcessor.Transform(t => new TransactionSummary
        {
            Id = t.Id,
            Date = t.Date,
            Amount = t.Amount,
            Description = $"{t.Category}: {t.Description}"
        });
    }
}

// İşlem sınıfı
public class Transaction
{
    public int Id { get; set; }
    public DateTime Date { get; set; }
    public decimal Amount { get; set; }
    public string Category { get; set; }
    public string Description { get; set; }
}

// İşlem özeti sınıfı
public class TransactionSummary
{
    public int Id { get; set; }
    public DateTime Date { get; set; }
    public decimal Amount { get; set; }
    public string Description { get; set; }
}
```

## 6. Boxing Considerations

Boxing, değer tiplerinin referans tiplerine dönüştürülmesi işlemidir. Generic tipler, doğru kullanıldığında boxing işlemlerini önleyerek performansı artırabilir.

### Boxing ve Unboxing Örnekleri

```csharp
// Boxing ve unboxing örnekleri
public class BoxingExample
{
    // Non-generic metot - boxing gerektirir
    public static void ProcessValueNonGeneric(object value)
    {
        // value bir değer tipi ise boxing gerçekleşir
        Console.WriteLine($"İşlenen değer: {value}");
    }
    
    // Generic metot - boxing gerektirmez
    public static void ProcessValueGeneric<T>(T value)
    {
        // T bir değer tipi olsa bile boxing gerçekleşmez
        Console.WriteLine($"İşlenen değer: {value}");
    }
    
    // Boxing ve unboxing karşılaştırması
    public static void CompareBoxing()
    {
        int number = 42;
        
        // Boxing gerçekleşir
        ProcessValueNonGeneric(number);
        
        // Boxing gerçekleşmez
        ProcessValueGeneric(number);
        
        // Manuel boxing
        object boxed = number;
        
        // Unboxing
        int unboxed = (int)boxed;
    }
}
```

### Kullanım Örneği

```csharp
// Yüksek performanslı önbellek servisi
public class CacheService<TKey, TValue>
{
    private readonly Dictionary<TKey, TValue> _cache;
    
    public CacheService()
    {
        _cache = new Dictionary<TKey, TValue>();
    }
    
    // Önbelleğe veri ekleme
    public void Add(TKey key, TValue value)
    {
        if (_cache.ContainsKey(key))
            _cache[key] = value;
        else
            _cache.Add(key, value);
    }
    
    // Önbellekten veri alma
    public bool TryGetValue(TKey key, out TValue value)
    {
        return _cache.TryGetValue(key, out value);
    }
    
    // Önbellekten veri silme
    public bool Remove(TKey key)
    {
        return _cache.Remove(key);
    }
    
    // Önbelleği temizleme
    public void Clear()
    {
        _cache.Clear();
    }
}

// Performans ölçüm servisi
public class PerformanceMonitoringService
{
    // Değer tipleri için önbellek - boxing yok
    private readonly CacheService<string, int> _intCache;
    
    // Referans tipleri için önbellek
    private readonly CacheService<string, object> _objectCache;
    
    public PerformanceMonitoringService()
    {
        _intCache = new CacheService<string, int>();
        _objectCache = new CacheService<string, object>();
    }
    
    // Değer tipini önbelleğe ekleme - boxing yok
    public void CacheIntValue(string key, int value)
    {
        _intCache.Add(key, value);
    }
    
    // Değer tipini önbellekten alma - unboxing yok
    public bool TryGetIntValue(string key, out int value)
    {
        return _intCache.TryGetValue(key, out value);
    }
    
    // Değer tipini object önbelleğine ekleme - boxing var
    public void CacheIntAsObject(string key, int value)
    {
        _objectCache.Add(key, value); // Boxing gerçekleşir
    }
    
    // Değer tipini object önbellekten alma - unboxing var
    public bool TryGetIntFromObject(string key, out int value)
    {
        if (_objectCache.TryGetValue(key, out object objValue))
        {
            value = (int)objValue; // Unboxing gerçekleşir
            return true;
        }
        
        value = default;
        return false;
    }
    
    // Performans karşılaştırması
    public void ComparePerformance(int iterations)
    {
        var sw1 = System.Diagnostics.Stopwatch.StartNew();
        
        for (int i = 0; i < iterations; i++)
        {
            CacheIntValue($"key{i}", i);
            TryGetIntValue($"key{i}", out int value1);
        }
        
        sw1.Stop();
        
        var sw2 = System.Diagnostics.Stopwatch.StartNew();
        
        for (int i = 0; i < iterations; i++)
        {
            CacheIntAsObject($"key{i}", i);
            TryGetIntFromObject($"key{i}", out int value2);
        }
        
        sw2.Stop();
        
        Console.WriteLine($"Generic önbellek (boxing yok): {sw1.ElapsedMilliseconds} ms");
        Console.WriteLine($"Object önbellek (boxing var): {sw2.ElapsedMilliseconds} ms");
    }
}
```

## Özet

Bu bölümde, C#'taki generic tip parametrelerinin özelliklerini ve kullanım şekillerini inceledik:

- **Default Keyword**: Tip parametrelerinin varsayılan değerlerini elde etmek için kullanılır.
- **Type Parameter Constraints**: Tip parametrelerinin belirli özelliklere sahip olmasını sağlar.
- **Type Parameter Inference**: Derleyicinin metot çağrılarında tip parametrelerini otomatik olarak belirlemesidir.
- **Generic Method Resolution**: Derleyicinin hangi generic metot uygulamasının çağrılacağını belirleme sürecidir.
- **Type Parameter Performance**: Tip parametreleri, doğru kullanıldığında performans avantajları sağlar.
- **Boxing Considerations**: Generic tipler, değer tipleriyle çalışırken boxing ve unboxing işlemlerini önleyerek performansı artırabilir.

Generic tip parametreleri, tip güvenliği sağlayarak, kod tekrarını azaltarak ve performansı artırarak daha güvenilir ve bakımı kolay uygulamalar geliştirmenize yardımcı olur. Özellikle performans kritik uygulamalarda, generic tiplerin doğru kullanımı önemli avantajlar sağlar. 