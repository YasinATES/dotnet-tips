# Interface Segregation Principle (ISP)

Interface Segregation Principle (Arayüz Ayrıştırma Prensibi), SOLID prensiplerinin dördüncü harfi olan "I"yı temsil eder. Bu prensip, "istemciler kullanmadıkları arayüzlere bağımlı olmamalıdır" şeklinde ifade edilir. Başka bir deyişle, büyük ve genel arayüzler yerine, daha küçük ve özelleşmiş arayüzler tercih edilmelidir.

## 1. ISP'nin Temel Kavramı

ISP'nin özü, bir sınıfın ihtiyaç duymadığı metotları içeren bir arayüzü uygulamaya zorlanmamasıdır. Bu, arayüzlerin istemci odaklı olması gerektiği anlamına gelir. Bir arayüz, onu kullanan istemcilerin ihtiyaçlarına göre tasarlanmalıdır.

### ISP İhlali Örneği

```csharp
// ISP ihlali - Çok fazla sorumluluğa sahip arayüz
public interface IMultiFunction
{
    void Print(Document document);
    void Scan(Document document);
    void Fax(Document document);
    void Copy(Document document);
    void PrintDuplex(Document document);
    void ScanColor(Document document);
    void ScanDuplex(Document document);
    int InkLevel { get; }
    bool IsOnline { get; }
}

// Basit bir yazıcı sınıfı - sadece yazdırma yapabilir
public class SimplePrinter : IMultiFunction
{
    public void Print(Document document)
    {
        Console.WriteLine("Belge yazdırılıyor...");
    }
    
    // Desteklenmeyen işlemler
    public void Scan(Document document)
    {
        throw new NotSupportedException("Bu yazıcı tarama yapamaz.");
    }
    
    public void Fax(Document document)
    {
        throw new NotSupportedException("Bu yazıcı faks gönderemez.");
    }
    
    public void Copy(Document document)
    {
        throw new NotSupportedException("Bu yazıcı fotokopi çekemez.");
    }
    
    public void PrintDuplex(Document document)
    {
        throw new NotSupportedException("Bu yazıcı çift taraflı yazdırma yapamaz.");
    }
    
    public void ScanColor(Document document)
    {
        throw new NotSupportedException("Bu yazıcı renkli tarama yapamaz.");
    }
    
    public void ScanDuplex(Document document)
    {
        throw new NotSupportedException("Bu yazıcı çift taraflı tarama yapamaz.");
    }
    
    public int InkLevel => 100; // Mürekkep seviyesi
    
    public bool IsOnline => true; // Çevrimiçi durumu
}
```

Bu örnekte, `IMultiFunction` arayüzü çok fazla sorumluluğa sahiptir ve tüm işlevleri desteklemeyen bir yazıcı için uygun değildir. `SimplePrinter` sınıfı, desteklemediği işlemler için istisna fırlatmak zorunda kalır, bu da ISP'nin ihlalidir.

### ISP Uyumlu Tasarım

```csharp
// ISP uyumlu tasarım - Ayrıştırılmış arayüzler
public interface IPrinter
{
    void Print(Document document);
    int InkLevel { get; }
    bool IsOnline { get; }
}

public interface IScanner
{
    void Scan(Document document);
}

public interface IFax
{
    void Fax(Document document);
}

public interface ICopier
{
    void Copy(Document document);
}

public interface IDuplexPrinter : IPrinter
{
    void PrintDuplex(Document document);
}

public interface IColorScanner : IScanner
{
    void ScanColor(Document document);
}

public interface IDuplexScanner : IScanner
{
    void ScanDuplex(Document document);
}

// Basit bir yazıcı sınıfı - sadece yazdırma yapabilir
public class SimplePrinter : IPrinter
{
    public void Print(Document document)
    {
        Console.WriteLine("Belge yazdırılıyor...");
    }
    
    public int InkLevel => 100; // Mürekkep seviyesi
    
    public bool IsOnline => true; // Çevrimiçi durumu
}

// Çok fonksiyonlu bir yazıcı sınıfı
public class MultiFunctionPrinter : IPrinter, IScanner, IFax, ICopier, IDuplexPrinter, IColorScanner
{
    public void Print(Document document)
    {
        Console.WriteLine("Belge yazdırılıyor...");
    }
    
    public void Scan(Document document)
    {
        Console.WriteLine("Belge taranıyor...");
    }
    
    public void Fax(Document document)
    {
        Console.WriteLine("Belge fakslanıyor...");
    }
    
    public void Copy(Document document)
    {
        Console.WriteLine("Belge kopyalanıyor...");
    }
    
    public void PrintDuplex(Document document)
    {
        Console.WriteLine("Belge çift taraflı yazdırılıyor...");
    }
    
    public void ScanColor(Document document)
    {
        Console.WriteLine("Belge renkli taranıyor...");
    }
    
    public int InkLevel => 100; // Mürekkep seviyesi
    
    public bool IsOnline => true; // Çevrimiçi durumu
}

// Kullanım örneği
public class PrintingService
{
    public void PrintDocument(IPrinter printer, Document document)
    {
        if (!printer.IsOnline)
        {
            Console.WriteLine("Yazıcı çevrimdışı!");
            return;
        }
        
        if (printer.InkLevel < 10)
        {
            Console.WriteLine("Mürekkep seviyesi düşük!");
        }
        
        printer.Print(document);
    }
}
```

Bu tasarımda, büyük `IMultiFunction` arayüzü daha küçük ve özelleşmiş arayüzlere bölünmüştür. Her sınıf, yalnızca desteklediği işlevleri içeren arayüzleri uygular. Bu, ISP'ye uygundur çünkü sınıflar kullanmadıkları metotları uygulamak zorunda kalmazlar.

## 2. Role Interfaces ve Header Interfaces

ISP'yi uygulamanın iki yaygın yaklaşımı vardır: Role Interfaces ve Header Interfaces.

### Role Interfaces (Rol Arayüzleri)

Rol arayüzleri, bir sınıfın belirli bir bağlamda oynayabileceği rolü tanımlar. Bu arayüzler genellikle küçüktür ve tek bir sorumluluğa odaklanır.

```csharp
// Role Interface örneği
public interface IPayable
{
    decimal CalculatePayment();
}

public interface ITaxable
{
    decimal CalculateTax();
}

public interface IShippable
{
    decimal CalculateShippingCost();
}

// Bir ürün sınıfı, birden fazla rolü uygulayabilir
public class Product : IPayable, ITaxable, IShippable
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public decimal Weight { get; set; }
    
    public decimal CalculatePayment()
    {
        return Price;
    }
    
    public decimal CalculateTax()
    {
        return Price * 0.18m; // %18 KDV
    }
    
    public decimal CalculateShippingCost()
    {
        return Weight * 0.5m; // Kilo başına 0.5 TL
    }
}

// Bir hizmet sınıfı, sadece ilgili rolleri uygular
public class Service : IPayable, ITaxable
{
    public string Name { get; set; }
    public decimal HourlyRate { get; set; }
    public int Hours { get; set; }
    
    public decimal CalculatePayment()
    {
        return HourlyRate * Hours;
    }
    
    public decimal CalculateTax()
    {
        return CalculatePayment() * 0.18m; // %18 KDV
    }
}

// Kullanım örneği
public class PaymentProcessor
{
    public decimal ProcessPayment(IPayable payable)
    {
        return payable.CalculatePayment();
    }
}

public class TaxCalculator
{
    public decimal CalculateTax(ITaxable taxable)
    {
        return taxable.CalculateTax();
    }
}

public class ShippingCalculator
{
    public decimal CalculateShippingCost(IShippable shippable)
    {
        return shippable.CalculateShippingCost();
    }
}
```

Bu örnekte, her arayüz belirli bir rolü temsil eder ve sınıflar yalnızca ilgili rolleri uygular. Bu, ISP'ye uygundur çünkü sınıflar kullanmadıkları metotları uygulamak zorunda kalmazlar.

### Header Interfaces (Başlık Arayüzleri)

Başlık arayüzleri, bir sınıfın tüm yeteneklerini tek bir arayüzde tanımlar. Bu yaklaşım, sınıfın tüm işlevlerini açıkça belirtir, ancak ISP'yi ihlal edebilir.

```csharp
// Header Interface örneği
public interface IProduct
{
    string Name { get; set; }
    decimal Price { get; set; }
    decimal Weight { get; set; }
    decimal CalculatePayment();
    decimal CalculateTax();
    decimal CalculateShippingCost();
}

public interface IService
{
    string Name { get; set; }
    decimal HourlyRate { get; set; }
    int Hours { get; set; }
    decimal CalculatePayment();
    decimal CalculateTax();
}

// Ürün sınıfı
public class Product : IProduct
{
    public string Name { get; set; }
    public decimal Price { get; set; }
    public decimal Weight { get; set; }
    
    public decimal CalculatePayment()
    {
        return Price;
    }
    
    public decimal CalculateTax()
    {
        return Price * 0.18m; // %18 KDV
    }
    
    public decimal CalculateShippingCost()
    {
        return Weight * 0.5m; // Kilo başına 0.5 TL
    }
}

// Hizmet sınıfı
public class Service : IService
{
    public string Name { get; set; }
    public decimal HourlyRate { get; set; }
    public int Hours { get; set; }
    
    public decimal CalculatePayment()
    {
        return HourlyRate * Hours;
    }
    
    public decimal CalculateTax()
    {
        return CalculatePayment() * 0.18m; // %18 KDV
    }
}
```

Bu yaklaşımda, her sınıf için ayrı bir arayüz tanımlanır. Bu, sınıfın tüm yeteneklerini açıkça belirtir, ancak istemcilerin kullanmadıkları metotlara bağımlı olmasına neden olabilir. Genellikle, rol arayüzleri başlık arayüzlerinden daha esnek ve ISP'ye daha uygundur.

## 3. ISP ve .NET Framework Design

.NET Framework, ISP'yi yaygın olarak uygular. Örneğin, koleksiyon arayüzleri hiyerarşisi:

```csharp
// .NET koleksiyon arayüzleri
public interface IEnumerable<T>
{
    IEnumerator<T> GetEnumerator();
}

public interface ICollection<T> : IEnumerable<T>
{
    int Count { get; }
    bool IsReadOnly { get; }
    void Add(T item);
    void Clear();
    bool Contains(T item);
    void CopyTo(T[] array, int arrayIndex);
    bool Remove(T item);
}

public interface IList<T> : ICollection<T>
{
    T this[int index] { get; set; }
    int IndexOf(T item);
    void Insert(int index, T item);
    void RemoveAt(int index);
}

public interface IDictionary<TKey, TValue> : ICollection<KeyValuePair<TKey, TValue>>
{
    TValue this[TKey key] { get; set; }
    ICollection<TKey> Keys { get; }
    ICollection<TValue> Values { get; }
    bool ContainsKey(TKey key);
    bool TryGetValue(TKey key, out TValue value);
    void Add(TKey key, TValue value);
    bool Remove(TKey key);
}
```

Bu tasarımda, her arayüz belirli bir sorumluluğa odaklanır ve daha spesifik arayüzler daha genel arayüzleri genişletir. Bu, ISP'ye uygundur çünkü istemciler yalnızca ihtiyaç duydukları arayüzlere bağımlı olabilirler.

## 4. Gerçek Hayat Örneği: E-Ticaret Sistemi

E-ticaret sistemleri, ISP'yi uygulamak için iyi bir örnektir. Bir e-ticaret sisteminde, sipariş işleme, ödeme, kargo ve bildirim gibi çeşitli işlevler vardır.

### ISP İhlali

```csharp
// ISP ihlali - Çok fazla sorumluluğa sahip arayüz
public interface IOrderService
{
    // Sipariş işleme
    Order CreateOrder(Customer customer, List<OrderItem> items);
    void UpdateOrder(Order order);
    void CancelOrder(int orderId);
    Order GetOrderById(int orderId);
    List<Order> GetOrdersByCustomer(int customerId);
    
    // Ödeme işleme
    void ProcessPayment(int orderId, PaymentMethod paymentMethod, decimal amount);
    void RefundPayment(int orderId);
    PaymentStatus GetPaymentStatus(int orderId);
    
    // Kargo işleme
    void ShipOrder(int orderId, ShippingMethod shippingMethod);
    ShippingStatus GetShippingStatus(int orderId);
    void UpdateShippingAddress(int orderId, Address address);
    
    // Bildirimler
    void SendOrderConfirmation(int orderId);
    void SendShippingNotification(int orderId);
    void SendPaymentReceipt(int orderId);
}

// Kullanım örneği
public class OrderProcessor
{
    private readonly IOrderService _orderService;
    
    public OrderProcessor(IOrderService orderService)
    {
        _orderService = orderService;
    }
    
    public void ProcessOrder(Customer customer, List<OrderItem> items, PaymentMethod paymentMethod, ShippingMethod shippingMethod)
    {
        // Sipariş oluşturma
        var order = _orderService.CreateOrder(customer, items);
        
        // Ödeme işlemi
        _orderService.ProcessPayment(order.Id, paymentMethod, order.TotalAmount);
        
        // Kargo işlemi
        _orderService.ShipOrder(order.Id, shippingMethod);
        
        // Bildirimler
        _orderService.SendOrderConfirmation(order.Id);
        _orderService.SendShippingNotification(order.Id);
        _orderService.SendPaymentReceipt(order.Id);
    }
}
```

Bu tasarımda, `IOrderService` arayüzü çok fazla sorumluluğa sahiptir ve farklı istemcilerin ihtiyaçlarına göre ayrıştırılmamıştır. Bu, ISP'nin ihlalidir.

### ISP Uyumlu Tasarım

```csharp
// ISP uyumlu tasarım - Ayrıştırılmış arayüzler
public interface IOrderService
{
    Order CreateOrder(Customer customer, List<OrderItem> items);
    void UpdateOrder(Order order);
    void CancelOrder(int orderId);
    Order GetOrderById(int orderId);
    List<Order> GetOrdersByCustomer(int customerId);
}

public interface IPaymentService
{
    void ProcessPayment(int orderId, PaymentMethod paymentMethod, decimal amount);
    void RefundPayment(int orderId);
    PaymentStatus GetPaymentStatus(int orderId);
}

public interface IShippingService
{
    void ShipOrder(int orderId, ShippingMethod shippingMethod);
    ShippingStatus GetShippingStatus(int orderId);
    void UpdateShippingAddress(int orderId, Address address);
}

public interface INotificationService
{
    void SendOrderConfirmation(int orderId);
    void SendShippingNotification(int orderId);
    void SendPaymentReceipt(int orderId);
}

// Kullanım örneği
public class OrderProcessor
{
    private readonly IOrderService _orderService;
    private readonly IPaymentService _paymentService;
    private readonly IShippingService _shippingService;
    private readonly INotificationService _notificationService;
    
    public OrderProcessor(
        IOrderService orderService,
        IPaymentService paymentService,
        IShippingService shippingService,
        INotificationService notificationService)
    {
        _orderService = orderService;
        _paymentService = paymentService;
        _shippingService = shippingService;
        _notificationService = notificationService;
    }
    
    public void ProcessOrder(Customer customer, List<OrderItem> items, PaymentMethod paymentMethod, ShippingMethod shippingMethod)
    {
        // Sipariş oluşturma
        var order = _orderService.CreateOrder(customer, items);
        
        // Ödeme işlemi
        _paymentService.ProcessPayment(order.Id, paymentMethod, order.TotalAmount);
        
        // Kargo işlemi
        _shippingService.ShipOrder(order.Id, shippingMethod);
        
        // Bildirimler
        _notificationService.SendOrderConfirmation(order.Id);
        _notificationService.SendShippingNotification(order.Id);
        _notificationService.SendPaymentReceipt(order.Id);
    }
}
```

Bu tasarımda, büyük `IOrderService` arayüzü daha küçük ve özelleşmiş arayüzlere bölünmüştür. Her arayüz, belirli bir sorumluluğa odaklanır ve istemciler yalnızca ihtiyaç duydukları arayüzlere bağımlı olurlar. Bu, ISP'ye uygundur.

## En İyi Pratikler

1. **Arayüzleri İstemci Odaklı Tasarlayın**
   - Arayüzleri, onları kullanacak istemcilerin ihtiyaçlarına göre tasarlayın.
   - Bir istemcinin kullanmayacağı metotları içeren arayüzleri uygulamaya zorlamayın.
   - "Bir arayüz, bir sorumluluk" prensibini benimseyin.

2. **Arayüzleri Küçük ve Odaklanmış Tutun**
   - Büyük arayüzleri daha küçük ve özelleşmiş arayüzlere bölün.
   - Bir arayüz, tek bir kavramsal sorumluluğa odaklanmalıdır.
   - Arayüzlerin adları, sorumluluklarını açıkça yansıtmalıdır.

3. **Rol Arayüzlerini Tercih Edin**
   - Başlık arayüzleri yerine rol arayüzlerini tercih edin.
   - Bir sınıfın farklı bağlamlarda oynayabileceği rolleri ayrı arayüzlerle tanımlayın.
   - Bu, daha modüler ve esnek bir tasarım sağlar.

4. **Arayüz Hiyerarşilerini Dikkatli Tasarlayın**
   - Arayüz hiyerarşileri oluştururken, alt arayüzlerin üst arayüzlerin sorumluluklarını genişlettiğinden emin olun.
   - Çok derin hiyerarşilerden kaçının, çünkü bu, arayüzlerin odaklanmış kalmasını zorlaştırır.
   - Kompozisyon (birden fazla arayüzü uygulama) genellikle kalıtımdan daha esnektir.

5. **Dependency Injection Kullanın**
   - Bağımlılıkları arayüzler üzerinden enjekte edin, somut sınıflar üzerinden değil.
   - Bu, kodunuzu daha test edilebilir ve değiştirilebilir hale getirir.
   - Dependency Injection Container'ları, bağımlılıkları yönetmeyi kolaylaştırır.

6. **Arayüzleri Evrimleştirin, Değiştirmeyin**
   - Mevcut arayüzleri değiştirmek yerine, yeni arayüzler ekleyin.
   - Bu, geriye dönük uyumluluğu korur ve mevcut kodu bozmaz.
   - Adapter Pattern, eski arayüzleri yeni arayüzlerle uyumlu hale getirmenize yardımcı olabilir.

7. **Marker Arayüzlerinden Kaçının**
   - Metot içermeyen marker arayüzleri yerine, attribute'ları veya generic type constraint'leri kullanın.
   - Marker arayüzleri, ISP'yi ihlal etmez ancak genellikle daha iyi alternatifler vardır.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Şişman Arayüzler**
   ```csharp
   // Kötü - şişman arayüz
   public interface IRepository
   {
       void Add(object entity);
       void Update(object entity);
       void Delete(object entity);
       object GetById(int id);
       IEnumerable<object> GetAll();
       IEnumerable<object> Find(Func<object, bool> predicate);
       void SaveChanges();
       void BeginTransaction();
       void CommitTransaction();
       void RollbackTransaction();
       void ExecuteSql(string sql);
       int Count();
       bool Exists(int id);
       // ... ve daha fazlası
   }
   
   // İyi - ayrıştırılmış arayüzler
   public interface IRepository<T>
   {
       void Add(T entity);
       void Update(T entity);
       void Delete(T entity);
       T GetById(int id);
       IEnumerable<T> GetAll();
   }
   
   public interface ISearchableRepository<T>
   {
       IEnumerable<T> Find(Func<T, bool> predicate);
   }
   
   public interface IUnitOfWork
   {
       void SaveChanges();
   }
   
   public interface ITransactionManager
   {
       void BeginTransaction();
       void CommitTransaction();
       void RollbackTransaction();
   }
   
   public interface ISqlExecutor
   {
       void ExecuteSql(string sql);
   }
   
   public interface ICountable<T>
   {
       int Count();
       bool Exists(int id);
   }
   ```

2. **Arayüz Kirliliği**
   ```csharp
   // Kötü - arayüz kirliliği
   public class Customer : IEquatable<Customer>, IComparable<Customer>, ICloneable, ISerializable, IDisposable, INotifyPropertyChanged, IDataErrorInfo, IEditableObject, INotifyDataErrorInfo
   {
       // Çok fazla arayüz uygulaması
   }
   
   // İyi - odaklanmış arayüz uygulamaları
   public class Customer : IEquatable<Customer>, IComparable<Customer>
   {
       // Temel işlevsellik için gerekli arayüzler
   }
   
   public class EditableCustomer : Customer, INotifyPropertyChanged, IDataErrorInfo
   {
       // UI bağlama için gerekli arayüzler
   }
   
   public class SerializableCustomer : Customer, ISerializable
   {
       // Serileştirme için gerekli arayüzler
   }
   ```

3. **Arayüz Sızıntısı**
   ```csharp
   // Kötü - arayüz sızıntısı
   public interface IUserService
   {
       User GetUserById(int id);
       void CreateUser(User user);
       void UpdateUser(User user);
       void DeleteUser(int id);
       
       // Implementasyon detayları sızdırılıyor
       SqlConnection GetDatabaseConnection();
       void CloseConnection(SqlConnection connection);
       DataTable ExecuteQuery(string sql);
   }
   
   // İyi - soyutlama seviyesi korunuyor
   public interface IUserService
   {
       User GetUserById(int id);
       void CreateUser(User user);
       void UpdateUser(User user);
       void DeleteUser(int id);
   }
   
   // Implementasyon detayları ayrı bir arayüzde
   internal interface IDatabaseAccess
   {
       SqlConnection GetConnection();
       void CloseConnection(SqlConnection connection);
       DataTable ExecuteQuery(string sql);
   }
   ```

4. **Aşırı Parçalama**
   ```csharp
   // Kötü - aşırı parçalama
   public interface IAddable<T>
   {
       void Add(T item);
   }
   
   public interface IRemovable<T>
   {
       void Remove(T item);
   }
   
   public interface IClearable
   {
       void Clear();
   }
   
   public interface ICountable
   {
       int Count { get; }
   }
   
   // İyi - dengeli parçalama
   public interface ICollection<T>
   {
       void Add(T item);
       void Remove(T item);
       void Clear();
       int Count { get; }
   }
   ```

5. **Arayüz Hiyerarşisini Kötüye Kullanmak**
   ```csharp
   // Kötü - arayüz hiyerarşisini kötüye kullanmak
   public interface IBaseRepository
   {
       void Add(object entity);
       void Update(object entity);
       void Delete(object entity);
       object GetById(int id);
   }
   
   public interface IAdvancedRepository : IBaseRepository
   {
       IEnumerable<object> GetAll();
       IEnumerable<object> Find(Func<object, bool> predicate);
       void SaveChanges();
       void BeginTransaction();
       void CommitTransaction();
       void RollbackTransaction();
   }
   
   // İyi - mantıklı arayüz hiyerarşisi
   public interface IRepository<T>
   {
       void Add(T entity);
       void Update(T entity);
       void Delete(T entity);
       T GetById(int id);
   }
   
   public interface IReadableRepository<T> : IRepository<T>
   {
       IEnumerable<T> GetAll();
   }
   
   public interface ISearchableRepository<T> : IReadableRepository<T>
   {
       IEnumerable<T> Find(Func<T, bool> predicate);
   }
   
   public interface IUnitOfWorkRepository<T> : IRepository<T>
   {
       void SaveChanges();
   }
   
   public interface ITransactionalRepository<T> : IUnitOfWorkRepository<T>
   {
       void BeginTransaction();
       void CommitTransaction();
       void RollbackTransaction();
   }
   ```

Interface Segregation Principle, yazılım tasarımında modülerlik ve esneklik sağlamanın temel bir yoludur. Bu prensibi doğru uygulayarak, kodunuzu daha bakımı kolay, test edilebilir ve genişletilebilir hale getirebilirsiniz. ISP, diğer SOLID prensipleriyle birlikte, daha sağlam ve sürdürülebilir yazılım sistemleri oluşturmanıza yardımcı olur. 