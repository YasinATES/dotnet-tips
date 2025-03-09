# Open/Closed Principle (OCP)

Open/Closed Principle (Açık/Kapalı Prensibi), SOLID prensiplerinin ikinci harfi olan "O"yu temsil eder. Bu prensip, yazılım varlıklarının (sınıflar, modüller, fonksiyonlar vb.) genişletmeye açık, ancak değiştirmeye kapalı olması gerektiğini belirtir. Yani, mevcut kodu değiştirmeden yeni davranışlar ekleyebilmeliyiz.

## 1. OCP'nin Temel Kavramı

OCP'nin özü, yazılım sistemlerinin değişen gereksinimlere uyum sağlayabilmesi için, mevcut kodu değiştirmeden genişletilebilir olmasıdır. Bu, kodun daha dayanıklı olmasını sağlar ve yan etkileri azaltır.

### OCP İhlali Örneği

```csharp
// OCP ihlali - yeni ödeme yöntemi eklemek için mevcut kodu değiştirmek gerekir
public class PaymentProcessor
{
    public void ProcessPayment(string paymentMethod, decimal amount)
    {
        if (paymentMethod == "CreditCard")
        {
            // Kredi kartı ödeme işlemi
            Console.WriteLine($"Kredi kartı ile {amount:C} ödeme işleniyor...");
            // Kredi kartı işleme kodu...
        }
        else if (paymentMethod == "PayPal")
        {
            // PayPal ödeme işlemi
            Console.WriteLine($"PayPal ile {amount:C} ödeme işleniyor...");
            // PayPal işleme kodu...
        }
        else if (paymentMethod == "BankTransfer")
        {
            // Banka havalesi işlemi
            Console.WriteLine($"Banka havalesi ile {amount:C} ödeme işleniyor...");
            // Banka havalesi işleme kodu...
        }
        // Yeni bir ödeme yöntemi eklemek için bu metodu değiştirmek gerekir
        else
        {
            throw new ArgumentException($"Desteklenmeyen ödeme yöntemi: {paymentMethod}");
        }
    }
}
```

Bu tasarımda, yeni bir ödeme yöntemi eklemek için `ProcessPayment` metodunu değiştirmek gerekir. Bu, OCP'yi ihlal eder çünkü sınıf değişime açıktır.

### OCP Uyumlu Tasarım

```csharp
// OCP uyumlu tasarım - abstraction ve polymorphism kullanımı
public interface IPaymentProcessor
{
    void ProcessPayment(decimal amount);
    string PaymentMethod { get; }
}

// Kredi kartı ödeme işlemcisi
public class CreditCardProcessor : IPaymentProcessor
{
    public string PaymentMethod => "Kredi Kartı";
    
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Kredi kartı ile {amount:C} ödeme işleniyor...");
        // Kredi kartı işleme kodu...
    }
}

// PayPal ödeme işlemcisi
public class PayPalProcessor : IPaymentProcessor
{
    public string PaymentMethod => "PayPal";
    
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"PayPal ile {amount:C} ödeme işleniyor...");
        // PayPal işleme kodu...
    }
}

// Banka havalesi ödeme işlemcisi
public class BankTransferProcessor : IPaymentProcessor
{
    public string PaymentMethod => "Banka Havalesi";
    
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Banka havalesi ile {amount:C} ödeme işleniyor...");
        // Banka havalesi işleme kodu...
    }
}

// Ödeme servisi
public class PaymentService
{
    private readonly List<IPaymentProcessor> _paymentProcessors;
    
    public PaymentService(IEnumerable<IPaymentProcessor> paymentProcessors)
    {
        _paymentProcessors = new List<IPaymentProcessor>(paymentProcessors);
    }
    
    public void ProcessPayment(string paymentMethod, decimal amount)
    {
        var processor = _paymentProcessors.FirstOrDefault(p => p.PaymentMethod == paymentMethod);
        
        if (processor == null)
        {
            throw new ArgumentException($"Desteklenmeyen ödeme yöntemi: {paymentMethod}");
        }
        
        processor.ProcessPayment(amount);
    }
    
    // Yeni bir ödeme işlemcisi eklemek için
    public void AddPaymentProcessor(IPaymentProcessor processor)
    {
        _paymentProcessors.Add(processor);
    }
}

// Kullanım örneği
public class Program
{
    public static void Main()
    {
        // Ödeme işlemcilerini oluştur
        var creditCardProcessor = new CreditCardProcessor();
        var payPalProcessor = new PayPalProcessor();
        var bankTransferProcessor = new BankTransferProcessor();
        
        // Ödeme servisini oluştur
        var paymentService = new PaymentService(new IPaymentProcessor[] 
        {
            creditCardProcessor,
            payPalProcessor,
            bankTransferProcessor
        });
        
        // Ödemeleri işle
        paymentService.ProcessPayment("Kredi Kartı", 100.00m);
        paymentService.ProcessPayment("PayPal", 50.00m);
        paymentService.ProcessPayment("Banka Havalesi", 200.00m);
        
        // Yeni bir ödeme yöntemi ekle (mevcut kodu değiştirmeden)
        var cryptoProcessor = new CryptoProcessor();
        paymentService.AddPaymentProcessor(cryptoProcessor);
        
        // Yeni ödeme yöntemini kullan
        paymentService.ProcessPayment("Kripto Para", 75.00m);
    }
}

// Sonradan eklenen yeni ödeme işlemcisi
public class CryptoProcessor : IPaymentProcessor
{
    public string PaymentMethod => "Kripto Para";
    
    public void ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Kripto para ile {amount:C} ödeme işleniyor...");
        // Kripto para işleme kodu...
    }
}
```

Bu tasarımda, yeni bir ödeme yöntemi eklemek için mevcut kodu değiştirmek gerekmez. Sadece `IPaymentProcessor` arayüzünü uygulayan yeni bir sınıf oluşturmak ve bunu `PaymentService`'e eklemek yeterlidir. Bu, OCP'ye uygundur çünkü sistem genişletmeye açık, değiştirmeye kapalıdır.

## 2. Abstraction ve Polymorphism ile OCP Uygulama

OCP'yi uygulamanın en yaygın yolu, soyutlama (abstraction) ve çok biçimlilik (polymorphism) kullanmaktır. Bu, genellikle arayüzler (interfaces) veya soyut sınıflar (abstract classes) aracılığıyla yapılır.

### Gerçek Hayat Örneği: Rapor Oluşturma Sistemi

```csharp
// OCP ihlali - yeni rapor formatı eklemek için mevcut kodu değiştirmek gerekir
public class ReportGenerator
{
    public void GenerateReport(List<SalesData> data, string format)
    {
        if (format == "PDF")
        {
            Console.WriteLine("PDF raporu oluşturuluyor...");
            // PDF rapor oluşturma kodu...
            foreach (var item in data)
            {
                Console.WriteLine($"PDF: {item.ProductName}, {item.Amount:C}");
            }
        }
        else if (format == "Excel")
        {
            Console.WriteLine("Excel raporu oluşturuluyor...");
            // Excel rapor oluşturma kodu...
            foreach (var item in data)
            {
                Console.WriteLine($"Excel: {item.ProductName}, {item.Amount:C}");
            }
        }
        else if (format == "CSV")
        {
            Console.WriteLine("CSV raporu oluşturuluyor...");
            // CSV rapor oluşturma kodu...
            foreach (var item in data)
            {
                Console.WriteLine($"CSV: {item.ProductName}, {item.Amount:C}");
            }
        }
        // Yeni bir rapor formatı eklemek için bu metodu değiştirmek gerekir
        else
        {
            throw new ArgumentException($"Desteklenmeyen rapor formatı: {format}");
        }
    }
}

public class SalesData
{
    public string ProductName { get; set; }
    public decimal Amount { get; set; }
}
```

OCP uyumlu tasarım:

```csharp
// OCP uyumlu tasarım - abstraction kullanımı
public interface IReportGenerator
{
    void GenerateReport(List<SalesData> data);
    string ReportFormat { get; }
}

// PDF rapor oluşturucu
public class PdfReportGenerator : IReportGenerator
{
    public string ReportFormat => "PDF";
    
    public void GenerateReport(List<SalesData> data)
    {
        Console.WriteLine("PDF raporu oluşturuluyor...");
        // PDF rapor oluşturma kodu...
        foreach (var item in data)
        {
            Console.WriteLine($"PDF: {item.ProductName}, {item.Amount:C}");
        }
    }
}

// Excel rapor oluşturucu
public class ExcelReportGenerator : IReportGenerator
{
    public string ReportFormat => "Excel";
    
    public void GenerateReport(List<SalesData> data)
    {
        Console.WriteLine("Excel raporu oluşturuluyor...");
        // Excel rapor oluşturma kodu...
        foreach (var item in data)
        {
            Console.WriteLine($"Excel: {item.ProductName}, {item.Amount:C}");
        }
    }
}

// CSV rapor oluşturucu
public class CsvReportGenerator : IReportGenerator
{
    public string ReportFormat => "CSV";
    
    public void GenerateReport(List<SalesData> data)
    {
        Console.WriteLine("CSV raporu oluşturuluyor...");
        // CSV rapor oluşturma kodu...
        foreach (var item in data)
        {
            Console.WriteLine($"CSV: {item.ProductName}, {item.Amount:C}");
        }
    }
}

// Rapor servisi
public class ReportService
{
    private readonly Dictionary<string, IReportGenerator> _reportGenerators;
    
    public ReportService()
    {
        _reportGenerators = new Dictionary<string, IReportGenerator>();
    }
    
    public void RegisterReportGenerator(IReportGenerator generator)
    {
        _reportGenerators[generator.ReportFormat] = generator;
    }
    
    public void GenerateReport(List<SalesData> data, string format)
    {
        if (!_reportGenerators.TryGetValue(format, out var generator))
        {
            throw new ArgumentException($"Desteklenmeyen rapor formatı: {format}");
        }
        
        generator.GenerateReport(data);
    }
}

// Kullanım örneği
public class Program
{
    public static void Main()
    {
        // Satış verilerini oluştur
        var salesData = new List<SalesData>
        {
            new SalesData { ProductName = "Laptop", Amount = 1200.00m },
            new SalesData { ProductName = "Telefon", Amount = 800.00m },
            new SalesData { ProductName = "Tablet", Amount = 400.00m }
        };
        
        // Rapor servisini oluştur
        var reportService = new ReportService();
        
        // Rapor oluşturucuları kaydet
        reportService.RegisterReportGenerator(new PdfReportGenerator());
        reportService.RegisterReportGenerator(new ExcelReportGenerator());
        reportService.RegisterReportGenerator(new CsvReportGenerator());
        
        // Raporları oluştur
        reportService.GenerateReport(salesData, "PDF");
        reportService.GenerateReport(salesData, "Excel");
        reportService.GenerateReport(salesData, "CSV");
        
        // Yeni bir rapor formatı ekle (mevcut kodu değiştirmeden)
        reportService.RegisterReportGenerator(new JsonReportGenerator());
        
        // Yeni rapor formatını kullan
        reportService.GenerateReport(salesData, "JSON");
    }
}

// Sonradan eklenen yeni rapor oluşturucu
public class JsonReportGenerator : IReportGenerator
{
    public string ReportFormat => "JSON";
    
    public void GenerateReport(List<SalesData> data)
    {
        Console.WriteLine("JSON raporu oluşturuluyor...");
        // JSON rapor oluşturma kodu...
        foreach (var item in data)
        {
            Console.WriteLine($"JSON: {item.ProductName}, {item.Amount:C}");
        }
    }
}
```

## 3. Extension Methods ve OCP

C#'ın extension methods (genişletme metotları) özelliği, OCP'yi uygulamanın başka bir yoludur. Bu, mevcut bir sınıfı değiştirmeden ona yeni davranışlar eklemenizi sağlar.

### Extension Methods Örneği

```csharp
// Mevcut sınıf (değiştiremeyeceğimizi varsayalım)
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime DateOfBirth { get; set; }
}

// Extension methods ile genişletme
public static class CustomerExtensions
{
    // Müşterinin yaşını hesaplayan extension method
    public static int GetAge(this Customer customer)
    {
        return DateTime.Today.Year - customer.DateOfBirth.Year - 
               (DateTime.Today.DayOfYear < customer.DateOfBirth.DayOfYear ? 1 : 0);
    }
    
    // Müşterinin reşit olup olmadığını kontrol eden extension method
    public static bool IsAdult(this Customer customer)
    {
        return customer.GetAge() >= 18;
    }
    
    // Müşterinin e-posta adresinin geçerli olup olmadığını kontrol eden extension method
    public static bool HasValidEmail(this Customer customer)
    {
        return !string.IsNullOrWhiteSpace(customer.Email) && 
               customer.Email.Contains("@") && 
               customer.Email.Contains(".");
    }
}

// Kullanım örneği
public class Program
{
    public static void Main()
    {
        var customer = new Customer
        {
            Id = 1,
            Name = "Ahmet Yılmaz",
            Email = "ahmet@example.com",
            DateOfBirth = new DateTime(1990, 5, 15)
        };
        
        // Extension methods kullanımı
        int age = customer.GetAge();
        bool isAdult = customer.IsAdult();
        bool hasValidEmail = customer.HasValidEmail();
        
        Console.WriteLine($"Müşteri: {customer.Name}");
        Console.WriteLine($"Yaş: {age}");
        Console.WriteLine($"Reşit mi: {isAdult}");
        Console.WriteLine($"Geçerli e-posta: {hasValidEmail}");
    }
}
```

Bu örnekte, `Customer` sınıfını değiştirmeden ona yeni davranışlar ekledik. Bu, OCP'ye uygundur çünkü mevcut kodu değiştirmeden genişlettik.

## En İyi Pratikler

1. **Soyutlama Kullanın**
   - Somut sınıflar yerine arayüzlere (interfaces) veya soyut sınıflara (abstract classes) bağımlı olun.
   - Bu, kodunuzu değiştirmeden yeni davranışlar eklemenizi sağlar.
   - Arayüzler, sınıfların "ne" yapması gerektiğini tanımlar, "nasıl" yapacağını değil.

2. **Strateji Deseni Uygulayın**
   - Farklı algoritmaları veya davranışları kapsüllemek için Strateji (Strategy) tasarım desenini kullanın.
   - Bu, çalışma zamanında algoritmaları değiştirmenizi sağlar.
   - Örneğin, farklı ödeme işleme stratejileri veya farklı indirim hesaplama stratejileri.

3. **Fabrika Deseni Kullanın**
   - Nesne oluşturma mantığını merkezi bir yerde toplamak için Fabrika (Factory) tasarım desenini kullanın.
   - Bu, yeni nesne türleri eklerken istemci kodunu değiştirmemenizi sağlar.
   - Örneğin, farklı rapor oluşturucuları veya farklı ödeme işlemcileri için fabrikalar.

4. **Kompozisyon Tercih Edin**
   - Kalıtım yerine kompozisyonu tercih edin.
   - Kompozisyon, daha esnek ve değiştirilebilir tasarımlar sağlar.
   - "is-a" ilişkisi yerine "has-a" ilişkisini kullanın.

5. **Dependency Injection Kullanın**
   - Bağımlılıkları dışarıdan enjekte edin, içeride oluşturmayın.
   - Bu, bağımlılıkları değiştirmenizi ve test etmenizi kolaylaştırır.
   - Constructor injection, property injection veya method injection kullanabilirsiniz.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Switch/If-Else Blokları**
   ```csharp
   // Kötü - switch/if-else blokları
   public decimal CalculateDiscount(string customerType, decimal amount)
   {
       switch (customerType)
       {
           case "Regular":
               return amount * 0.05m;
           case "Premium":
               return amount * 0.10m;
           case "VIP":
               return amount * 0.15m;
           default:
               return 0;
       }
   }
   
   // İyi - polymorphism kullanımı
   public interface IDiscountStrategy
   {
       decimal CalculateDiscount(decimal amount);
   }
   
   public class RegularCustomerDiscount : IDiscountStrategy
   {
       public decimal CalculateDiscount(decimal amount)
       {
           return amount * 0.05m;
       }
   }
   
   public class PremiumCustomerDiscount : IDiscountStrategy
   {
       public decimal CalculateDiscount(decimal amount)
       {
           return amount * 0.10m;
       }
   }
   
   public class VipCustomerDiscount : IDiscountStrategy
   {
       public decimal CalculateDiscount(decimal amount)
       {
           return amount * 0.15m;
       }
   }
   ```

2. **Tip Kontrolü**
   ```csharp
   // Kötü - tip kontrolü
   public void ProcessShape(object shape)
   {
       if (shape is Circle)
       {
           Circle circle = (Circle)shape;
           // Daire işleme kodu...
       }
       else if (shape is Rectangle)
       {
           Rectangle rectangle = (Rectangle)shape;
           // Dikdörtgen işleme kodu...
       }
       else if (shape is Triangle)
       {
           Triangle triangle = (Triangle)shape;
           // Üçgen işleme kodu...
       }
   }
   
   // İyi - polymorphism kullanımı
   public interface IShape
   {
       void Process();
   }
   
   public class Circle : IShape
   {
       public void Process()
       {
           // Daire işleme kodu...
       }
   }
   
   public class Rectangle : IShape
   {
       public void Process()
       {
           // Dikdörtgen işleme kodu...
       }
   }
   
   public class Triangle : IShape
   {
       public void Process()
       {
           // Üçgen işleme kodu...
       }
   }
   
   public void ProcessShape(IShape shape)
   {
       shape.Process();
   }
   ```

3. **Sınıf İçinde Nesne Oluşturma**
   ```csharp
   // Kötü - sınıf içinde nesne oluşturma
   public class OrderProcessor
   {
       private readonly PaymentProcessor _paymentProcessor;
       
       public OrderProcessor()
       {
           _paymentProcessor = new CreditCardProcessor(); // Sıkı bağımlılık
       }
       
       public void ProcessOrder(Order order)
       {
           // Sipariş işleme kodu...
           _paymentProcessor.ProcessPayment(order.Amount);
       }
   }
   
   // İyi - dependency injection
   public class OrderProcessor
   {
       private readonly IPaymentProcessor _paymentProcessor;
       
       public OrderProcessor(IPaymentProcessor paymentProcessor)
       {
           _paymentProcessor = paymentProcessor;
       }
       
       public void ProcessOrder(Order order)
       {
           // Sipariş işleme kodu...
           _paymentProcessor.ProcessPayment(order.Amount);
       }
   }
   ```

4. **Aşırı Genişletme**
   ```csharp
   // Kötü - aşırı genişletme
   public interface IRepository
   {
       void Add(object entity);
       void Update(object entity);
       void Delete(object entity);
       object GetById(int id);
       IEnumerable<object> GetAll();
       IEnumerable<object> Find(Func<object, bool> predicate);
       void SaveChanges();
       // ... ve daha fazlası
   }
   
   // İyi - odaklanmış arayüzler
   public interface IRepository<T>
   {
       void Add(T entity);
       void Update(T entity);
       void Delete(T entity);
       T GetById(int id);
       IEnumerable<T> GetAll();
   }
   
   public interface ISearchableRepository<T> : IRepository<T>
   {
       IEnumerable<T> Find(Func<T, bool> predicate);
   }
   
   public interface IUnitOfWork
   {
       void SaveChanges();
   }
   ```

5. **Kalıtımı Kötüye Kullanmak**
   ```csharp
   // Kötü - kalıtımı kötüye kullanmak
   public class Rectangle
   {
       public virtual int Width { get; set; }
       public virtual int Height { get; set; }
       
       public int CalculateArea()
       {
           return Width * Height;
       }
   }
   
   public class Square : Rectangle
   {
       public override int Width
       {
           get { return base.Width; }
           set
           {
               base.Width = value;
               base.Height = value;
           }
       }
       
       public override int Height
       {
           get { return base.Height; }
           set
           {
               base.Width = value;
               base.Height = value;
           }
       }
   }
   
   // İyi - kompozisyon kullanmak
   public interface IShape
   {
       int CalculateArea();
   }
   
   public class Rectangle : IShape
   {
       public int Width { get; set; }
       public int Height { get; set; }
       
       public int CalculateArea()
       {
           return Width * Height;
       }
   }
   
   public class Square : IShape
   {
       public int Side { get; set; }
       
       public int CalculateArea()
       {
           return Side * Side;
       }
   }
   ```

Open/Closed Principle, yazılım geliştirmede esneklik ve dayanıklılık sağlamanın temel bir yoludur. Bu prensibi doğru uygulayarak, mevcut kodu değiştirmeden yeni davranışlar ekleyebilir, değişen gereksinimlere daha kolay uyum sağlayabilir ve kodunuzun genel kalitesini artırabilirsiniz. 