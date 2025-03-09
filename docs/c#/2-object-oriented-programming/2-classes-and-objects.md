# Sınıflar ve Nesneler

Sınıflar ve nesneler, C# dilinin ve nesne yönelimli programlamanın temel yapı taşlarıdır. Bu bölümde, C#'ta sınıf tanımlama, nesne oluşturma ve sınıflarla ilgili çeşitli özellikleri detaylı olarak inceleyeceğiz.

## 1. Sınıf Tanımlama ve Nesne Oluşturma

Sınıflar, nesnelerin şablonlarıdır ve nesneler bu şablonlardan oluşturulan somut örneklerdir.

### Temel Sınıf Tanımlama

```csharp
// Temel bir ürün sınıfı
public class Product
{
    // Alanlar (Fields)
    private int _id;
    private string _name;
    private decimal _price;
    
    // Özellikler (Properties)
    public int Id 
    { 
        get { return _id; } 
        set { _id = value; } 
    }
    
    public string Name 
    { 
        get { return _name; } 
        set { _name = value; } 
    }
    
    public decimal Price 
    { 
        get { return _price; } 
        set 
        { 
            if (value < 0)
                throw new ArgumentException("Fiyat negatif olamaz");
            _price = value; 
        } 
    }
    
    // Auto-implemented property
    public string Category { get; set; }
    
    // Metotlar
    public decimal CalculateDiscount(decimal discountRate)
    {
        return Price * discountRate;
    }
    
    public string GetProductInfo()
    {
        return $"Ürün: {Name}, Fiyat: {Price:C}, Kategori: {Category}";
    }
}
```

### Nesne Oluşturma ve Kullanma

```csharp
// Nesne oluşturma
Product laptop = new Product();
laptop.Id = 1;
laptop.Name = "Laptop";
laptop.Price = 5000;
laptop.Category = "Elektronik";

// Nesne metotlarını çağırma
decimal discount = laptop.CalculateDiscount(0.1m); // %10 indirim
Console.WriteLine($"İndirim miktarı: {discount:C}");
Console.WriteLine(laptop.GetProductInfo());

// Alternatif nesne oluşturma (object initializer)
Product phone = new Product
{
    Id = 2,
    Name = "Akıllı Telefon",
    Price = 3000,
    Category = "Elektronik"
};

Console.WriteLine(phone.GetProductInfo());
```

## 2. Constructor ve Destructor

Constructor (yapıcı metot), bir sınıfın nesnesi oluşturulduğunda otomatik olarak çağrılan özel bir metottur. Destructor (yıkıcı metot) ise nesne bellekten temizlendiğinde çağrılır.

### Constructor Tanımlama

```csharp
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    
    // Parametresiz constructor
    public Customer()
    {
        Id = 0;
        Name = "Misafir";
        Email = "misafir@example.com";
        Console.WriteLine("Parametresiz constructor çağrıldı");
    }
    
    // Parametreli constructor
    public Customer(int id, string name, string email)
    {
        Id = id;
        Name = name;
        Email = email;
        Console.WriteLine("Parametreli constructor çağrıldı");
    }
    
    // Kısmi parametreli constructor - constructor chaining
    public Customer(int id, string name) : this(id, name, "varsayilan@example.com")
    {
        Console.WriteLine("Kısmi parametreli constructor çağrıldı");
    }
}
```

### Constructor Kullanımı

```csharp
// Parametresiz constructor
Customer guest = new Customer();
Console.WriteLine($"Misafir: {guest.Name}, {guest.Email}");

// Parametreli constructor
Customer customer1 = new Customer(1, "Ahmet Yılmaz", "ahmet@example.com");
Console.WriteLine($"Müşteri: {customer1.Name}, {customer1.Email}");

// Kısmi parametreli constructor
Customer customer2 = new Customer(2, "Ayşe Demir");
Console.WriteLine($"Müşteri: {customer2.Name}, {customer2.Email}");
```

### Destructor Tanımlama

```csharp
public class ResourceManager
{
    private string _resourceName;
    
    public ResourceManager(string resourceName)
    {
        _resourceName = resourceName;
        Console.WriteLine($"{_resourceName} kaynağı oluşturuldu");
    }
    
    // Destructor
    ~ResourceManager()
    {
        Console.WriteLine($"{_resourceName} kaynağı temizlendi");
    }
}
```

Destructorlar genellikle unmanaged kaynakları temizlemek için kullanılır, ancak C#'ta Garbage Collector olduğu için çoğu durumda gerekli değildir. Unmanaged kaynaklar için `IDisposable` pattern kullanmak daha iyidir.

## 3. Static Sınıflar ve Üyeler

Static üyeler, sınıfın kendisine ait olup nesne örneğine ait olmayan üyelerdir. Static sınıflar ise sadece static üyeler içerebilir ve örnekleri oluşturulamaz.

### Static Üyeler

```csharp
public class MathHelper
{
    // Static alan
    public static readonly double PI = 3.14159265359;
    
    // Static özellik
    public static int OperationCount { get; private set; }
    
    // Static metot
    public static double CalculateCircleArea(double radius)
    {
        OperationCount++;
        return PI * radius * radius;
    }
    
    // Static metot
    public static double CalculateCirclePerimeter(double radius)
    {
        OperationCount++;
        return 2 * PI * radius;
    }
    
    // Instance metot
    public void DisplayInfo()
    {
        Console.WriteLine($"Bu bir yardımcı matematik sınıfıdır. Toplam işlem sayısı: {OperationCount}");
    }
}
```

### Static Sınıf

```csharp
// Static sınıf - örnekleri oluşturulamaz
public static class Logger
{
    // Static alan
    private static int _logCount;
    
    // Static constructor
    static Logger()
    {
        _logCount = 0;
        Console.WriteLine("Logger sınıfı başlatıldı");
    }
    
    // Static metot
    public static void LogInfo(string message)
    {
        _logCount++;
        Console.WriteLine($"INFO [{DateTime.Now}]: {message}");
    }
    
    // Static metot
    public static void LogError(string message)
    {
        _logCount++;
        Console.WriteLine($"ERROR [{DateTime.Now}]: {message}");
    }
    
    // Static özellik
    public static int LogCount => _logCount;
}
```

### Static Üyelerin Kullanımı

```csharp
// Static metotları çağırma
double area = MathHelper.CalculateCircleArea(5);
double perimeter = MathHelper.CalculateCirclePerimeter(5);

Console.WriteLine($"Daire alanı: {area}");
Console.WriteLine($"Daire çevresi: {perimeter}");
Console.WriteLine($"Toplam işlem sayısı: {MathHelper.OperationCount}");

// Instance metodu çağırmak için nesne oluşturma
MathHelper helper = new MathHelper();
helper.DisplayInfo();

// Static sınıf kullanımı
Logger.LogInfo("Uygulama başlatıldı");
Logger.LogError("Bağlantı hatası");
Console.WriteLine($"Toplam log sayısı: {Logger.LogCount}");
```

## 4. Partial Sınıflar ve Metotlar

Partial sınıflar, bir sınıfın tanımını birden fazla dosyaya bölmenize olanak tanır. Partial metotlar ise bir kısmı tanımlanmış, diğer kısmı ise başka bir partial sınıf içinde uygulanabilen metotlardır.

### Partial Sınıf Tanımlama

```csharp
// Dosya: Customer.cs
public partial class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    public string GetBasicInfo()
    {
        return $"Müşteri ID: {Id}, İsim: {Name}";
    }
    
    // Partial metot bildirimi
    partial void ValidateCustomer();
}

// Dosya: Customer.Address.cs
public partial class Customer
{
    public string Address { get; set; }
    public string City { get; set; }
    public string Country { get; set; }
    
    public string GetAddressInfo()
    {
        return $"Adres: {Address}, {City}, {Country}";
    }
    
    // Partial metot uygulaması
    partial void ValidateCustomer()
    {
        if (string.IsNullOrEmpty(Name))
        {
            throw new InvalidOperationException("Müşteri adı boş olamaz");
        }
        
        if (string.IsNullOrEmpty(Address))
        {
            throw new InvalidOperationException("Adres boş olamaz");
        }
    }
}
```

### Partial Sınıf Kullanımı

```csharp
Customer customer = new Customer
{
    Id = 1,
    Name = "Ali Yılmaz",
    Address = "Atatürk Cad. No:123",
    City = "İstanbul",
    Country = "Türkiye"
};

Console.WriteLine(customer.GetBasicInfo());
Console.WriteLine(customer.GetAddressInfo());
```

Partial metotlar, özellikle kod üretimi yapan araçlarla çalışırken kullanışlıdır. Örneğin, Entity Framework gibi ORM araçları, veritabanı modellerini partial sınıflar olarak üretir ve bu sayede siz kendi kodunuzu ayrı dosyalarda ekleyebilirsiniz.

## 5. Sealed Sınıflar

Sealed sınıflar, başka sınıflar tarafından miras alınamayan sınıflardır. Bir sınıfı sealed olarak işaretlemek, o sınıfın davranışının değiştirilmesini engeller.

### Sealed Sınıf Tanımlama

```csharp
// Sealed sınıf - miras alınamaz
public sealed class CreditCard
{
    public string CardNumber { get; }
    public string CardHolderName { get; }
    public DateTime ExpiryDate { get; }
    private string _cvv;
    
    public CreditCard(string cardNumber, string cardHolderName, DateTime expiryDate, string cvv)
    {
        // Basit doğrulama
        if (string.IsNullOrEmpty(cardNumber) || cardNumber.Length != 16)
            throw new ArgumentException("Kart numarası 16 haneli olmalıdır");
            
        if (string.IsNullOrEmpty(cvv) || cvv.Length != 3)
            throw new ArgumentException("CVV 3 haneli olmalıdır");
            
        if (expiryDate < DateTime.Now)
            throw new ArgumentException("Kart süresi dolmuş");
            
        CardNumber = cardNumber;
        CardHolderName = cardHolderName;
        ExpiryDate = expiryDate;
        _cvv = cvv;
    }
    
    public bool ValidateCard()
    {
        // Kart doğrulama mantığı
        return !string.IsNullOrEmpty(CardNumber) && 
               !string.IsNullOrEmpty(CardHolderName) && 
               ExpiryDate > DateTime.Now;
    }
    
    public string GetMaskedCardNumber()
    {
        // Kart numarasının son 4 hanesini göster, diğerlerini maskele
        return $"**** **** **** {CardNumber.Substring(12, 4)}";
    }
}
```

Sealed sınıflar, güvenlik açısından kritik olan veya davranışının değiştirilmemesi gereken sınıflar için kullanılır. Örneğin, `System.String` sınıfı sealed'dır.

## 6. Nested Classes (İç İçe Sınıflar)

Nested sınıflar, bir sınıfın içinde tanımlanan sınıflardır. Bu, ilgili sınıfları mantıksal olarak gruplandırmanıza ve kapsüllemeyi artırmanıza olanak tanır.

### Nested Sınıf Tanımlama

```csharp
public class Order
{
    public int OrderId { get; set; }
    public DateTime OrderDate { get; set; }
    public Customer Customer { get; set; }
    private List<OrderItem> _items;
    
    public Order(int orderId, Customer customer)
    {
        OrderId = orderId;
        OrderDate = DateTime.Now;
        Customer = customer;
        _items = new List<OrderItem>();
    }
    
    public void AddItem(int productId, string productName, decimal price, int quantity)
    {
        var item = new OrderItem(productId, productName, price, quantity);
        _items.Add(item);
    }
    
    public decimal CalculateTotal()
    {
        decimal total = 0;
        foreach (var item in _items)
        {
            total += item.CalculateSubtotal();
        }
        return total;
    }
    
    public void DisplayOrderDetails()
    {
        Console.WriteLine($"Sipariş No: {OrderId}");
        Console.WriteLine($"Tarih: {OrderDate}");
        Console.WriteLine($"Müşteri: {Customer.Name}");
        Console.WriteLine("Sipariş Kalemleri:");
        
        foreach (var item in _items)
        {
            Console.WriteLine($"  - {item.ProductName} x {item.Quantity} = {item.CalculateSubtotal():C}");
        }
        
        Console.WriteLine($"Toplam: {CalculateTotal():C}");
    }
    
    // Nested sınıf - Order sınıfının içinde tanımlanmış
    public class OrderItem
    {
        public int ProductId { get; }
        public string ProductName { get; }
        public decimal Price { get; }
        public int Quantity { get; }
        
        public OrderItem(int productId, string productName, decimal price, int quantity)
        {
            ProductId = productId;
            ProductName = productName;
            Price = price;
            Quantity = quantity;
        }
        
        public decimal CalculateSubtotal()
        {
            return Price * Quantity;
        }
    }
}
```

### Nested Sınıf Kullanımı

```csharp
Customer customer = new Customer(1, "Mehmet Yılmaz", "mehmet@example.com");
Order order = new Order(1001, customer);

order.AddItem(1, "Laptop", 5000, 1);
order.AddItem(2, "Mouse", 100, 2);
order.AddItem(3, "Klavye", 200, 1);

order.DisplayOrderDetails();

// Nested sınıfa dışarıdan erişim
Order.OrderItem newItem = new Order.OrderItem(4, "Monitör", 1500, 1);
```

Nested sınıflar, ana sınıfla yakından ilişkili ve genellikle sadece ana sınıf tarafından kullanılan yardımcı sınıflar için idealdir.

## 7. Record Types (C# 9.0+)

Record'lar, C# 9.0 ile tanıtılan, değer eşitliği semantiğine sahip, değiştirilemez (immutable) veri taşıyıcı sınıflardır.

### Record Tanımlama

```csharp
// Temel record tanımı
public record Person(string FirstName, string LastName, DateTime DateOfBirth);

// Daha kapsamlı record
public record Employee(
    int Id, 
    string FirstName, 
    string LastName, 
    string Department, 
    decimal Salary) 
{
    // Ek özellikler ve metotlar eklenebilir
    public string FullName => $"{FirstName} {LastName}";
    
    public decimal CalculateAnnualSalary() => Salary * 12;
    
    // Init-only property
    public string Email { get; init; } = string.Empty;
}
```

### Record Kullanımı

```csharp
// Record oluşturma
Person person1 = new Person("Ali", "Yılmaz", new DateTime(1985, 5, 15));
Person person2 = new Person("Ali", "Yılmaz", new DateTime(1985, 5, 15));

// Değer eşitliği kontrolü
Console.WriteLine(person1 == person2); // True - aynı değerlere sahipler

// Değiştirilemez (immutable) özellik
// person1.FirstName = "Ahmet"; // Derleme hatası

// With ifadesi ile yeni record oluşturma
Person person3 = person1 with { FirstName = "Ahmet" };
Console.WriteLine(person1 == person3); // False - farklı değerlere sahipler

// Deconstruction
var (firstName, lastName, dob) = person1;
Console.WriteLine($"{firstName} {lastName}, Doğum Tarihi: {dob.ToShortDateString()}");

// Daha kapsamlı record kullanımı
Employee emp1 = new Employee(1, "Zeynep", "Demir", "IT", 10000)
{
    Email = "zeynep@example.com"
};

Console.WriteLine($"Çalışan: {emp1.FullName}");
Console.WriteLine($"Yıllık Maaş: {emp1.CalculateAnnualSalary():C}");

// Init-only property ile yeni record oluşturma
Employee emp2 = emp1 with { Email = "zeynep.demir@example.com" };
```

Record'lar, özellikle veri taşıma nesneleri (DTO'lar), değer nesneleri (value objects) ve değiştirilemez veri yapıları için idealdir. Otomatik olarak `Equals`, `GetHashCode`, `ToString` ve deconstruction metotları sağlarlar.

## En İyi Pratikler

1. **Sınıf Tasarımı**
   - Sınıflar tek bir sorumluluğa sahip olmalıdır (Single Responsibility Principle).
   - Sınıf isimleri açıklayıcı ve anlamlı olmalıdır.
   - Sınıflar, gerektiğinde genişletilebilir ancak değiştirilmesi zor olmalıdır (Open/Closed Principle).

2. **Constructor Kullanımı**
   - Sınıfın geçerli bir durumda oluşturulmasını sağlamak için constructor'ları kullanın.
   - Çok sayıda parametre gerektiren constructor'lar yerine builder pattern veya factory metotları düşünün.

3. **Static Üyeler**
   - Static üyeleri, durumu olmayan ve global erişim gerektiren işlevler için kullanın.
   - Static sınıfları, yardımcı metotlar ve extension metotlar için kullanın.

4. **Partial Sınıflar**
   - Partial sınıfları, otomatik üretilen kod ile manuel yazılan kodu ayırmak için kullanın.
   - Çok büyük sınıfları bölmek için partial sınıfları kullanmaktan kaçının, bunun yerine sınıfı daha küçük sınıflara ayırmayı düşünün.

5. **Sealed Sınıflar**
   - Güvenlik açısından kritik sınıfları veya davranışının değiştirilmemesi gereken sınıfları sealed olarak işaretleyin.
   - Performans optimizasyonu için sık kullanılan ve miras alınması gerekmeyen sınıfları sealed olarak işaretlemeyi düşünün.

6. **Record Types**
   - Değiştirilemez veri yapıları için record'ları tercih edin.
   - DTO'lar ve value objects için record'ları kullanın.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Aşırı Karmaşık Sınıflar**
   ```csharp
   // Kötü - çok fazla sorumluluğa sahip sınıf
   public class UserManager
   {
       public void RegisterUser(User user) { /* ... */ }
       public void ValidateUser(User user) { /* ... */ }
       public void SendEmail(string to, string subject, string body) { /* ... */ }
       public void GenerateReport() { /* ... */ }
       public void UpdateDatabase() { /* ... */ }
   }
   
   // İyi - tek sorumluluğa sahip sınıflar
   public class UserRegistrationService
   {
       private readonly EmailService _emailService;
       private readonly UserValidator _validator;
       
       public UserRegistrationService(EmailService emailService, UserValidator validator)
       {
           _emailService = emailService;
           _validator = validator;
       }
       
       public void RegisterUser(User user)
       {
           if (_validator.ValidateUser(user))
           {
               // Kullanıcı kayıt işlemleri
               _emailService.SendWelcomeEmail(user.Email);
           }
       }
   }
   ```

2. **Gereksiz Static Kullanımı**
   ```csharp
   // Kötü - her şeyi static yapmak
   public static class UserHelper
   {
       public static List<User> Users { get; set; } = new List<User>();
       
       public static void AddUser(User user)
       {
           Users.Add(user);
       }
       
       public static User GetUser(int id)
       {
           return Users.FirstOrDefault(u => u.Id == id);
       }
   }
   
   // İyi - instance tabanlı yaklaşım
   public class UserRepository
   {
       private readonly List<User> _users = new List<User>();
       
       public void AddUser(User user)
       {
           _users.Add(user);
       }
       
       public User GetUser(int id)
       {
           return _users.FirstOrDefault(u => u.Id == id);
       }
   }
   ```

3. **Destructor Yerine IDisposable Kullanmamak**
   ```csharp
   // Kötü - destructor kullanımı
   public class FileProcessor
   {
       private FileStream _fileStream;
       
       public FileProcessor(string filePath)
       {
           _fileStream = File.OpenRead(filePath);
       }
       
       ~FileProcessor()
       {
           _fileStream?.Dispose();
       }
   }
   
   // İyi - IDisposable pattern
   public class FileProcessor : IDisposable
   {
       private FileStream _fileStream;
       private bool _disposed = false;
       
       public FileProcessor(string filePath)
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
                   _fileStream?.Dispose();
               }
               
               _disposed = true;
           }
       }
   }
   ```

4. **Record'ları Yanlış Kullanmak**
   ```csharp
   // Kötü - mutable state içeren record
   public record User
   {
       public int Id { get; set; }
       public string Name { get; set; }
       public List<Order> Orders { get; set; } = new List<Order>();
       
       public void AddOrder(Order order)
       {
           Orders.Add(order);
       }
   }
   
   // İyi - immutable record
   public record User(int Id, string Name, IReadOnlyList<Order> Orders)
   {
       // Factory metot ile yeni sipariş eklemek
       public User AddOrder(Order order)
       {
           var newOrders = Orders.ToList();
           newOrders.Add(order);
           return this with { Orders = newOrders };
       }
   }
   ```

Sınıflar ve nesneler, C# programlamanın temel yapı taşlarıdır. Bu kavramları doğru ve etkili bir şekilde kullanmak, daha modüler, bakımı kolay ve genişletilebilir kod yazmanıza yardımcı olacaktır. 