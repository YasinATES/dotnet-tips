# Single Responsibility Principle (SRP)

Single Responsibility Principle (Tek Sorumluluk Prensibi), SOLID prensiplerinin ilk harfi olan "S"yi temsil eder. Bu prensip, bir sınıfın yalnızca bir sorumluluğu olması gerektiğini ve bu sorumluluğun tamamen o sınıfa ait olması gerektiğini belirtir. Başka bir deyişle, bir sınıfın değişmesi için yalnızca bir nedeni olmalıdır.

## 1. SRP'nin Temel Kavramı

SRP'nin özü, bir sınıfın tek bir "aktöre" (kullanıcı, sistem, süreç) hizmet etmesi gerektiğidir. Bir sınıf birden fazla aktöre hizmet ediyorsa, bu aktörlerin talepleri çakışabilir ve sınıfın değişmesi için birden fazla neden ortaya çıkabilir.

### SRP İhlali Örneği

```csharp
// SRP ihlali - çok fazla sorumluluğa sahip sınıf
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string Address { get; set; }
    
    // Veritabanı işlemleri
    public void Save()
    {
        // Müşteriyi veritabanına kaydetme kodu
        string sql = $"INSERT INTO Customers (Name, Email, Address) VALUES ('{Name}', '{Email}', '{Address}')";
        // SQL çalıştırma kodu...
        Console.WriteLine($"Müşteri veritabanına kaydedildi: {Name}");
    }
    
    public void Delete()
    {
        // Müşteriyi veritabanından silme kodu
        string sql = $"DELETE FROM Customers WHERE Id = {Id}";
        // SQL çalıştırma kodu...
        Console.WriteLine($"Müşteri veritabanından silindi: {Name}");
    }
    
    // E-posta işlemleri
    public void SendWelcomeEmail()
    {
        // E-posta gönderme kodu
        string subject = "Hoş Geldiniz!";
        string body = $"Sayın {Name}, sitemize hoş geldiniz.";
        // SMTP kodu...
        Console.WriteLine($"Hoş geldiniz e-postası gönderildi: {Email}");
    }
    
    public void SendPromotionEmail(string promotionCode)
    {
        // Promosyon e-postası gönderme kodu
        string subject = "Özel Teklif!";
        string body = $"Sayın {Name}, size özel promosyon kodunuz: {promotionCode}";
        // SMTP kodu...
        Console.WriteLine($"Promosyon e-postası gönderildi: {Email}");
    }
    
    // Doğrulama işlemleri
    public bool ValidateEmail()
    {
        // E-posta doğrulama kodu
        return Email.Contains("@") && Email.Contains(".");
    }
    
    public bool ValidateAddress()
    {
        // Adres doğrulama kodu
        return !string.IsNullOrWhiteSpace(Address) && Address.Length > 10;
    }
}
```

Bu sınıf en az üç farklı sorumluluğa sahiptir:
1. Müşteri verilerini depolamak (veri modeli)
2. Veritabanı işlemleri (veri erişimi)
3. E-posta gönderimi (iş mantığı)
4. Veri doğrulama (validasyon)

### SRP Uyumlu Tasarım

```csharp
// Veri modeli - sadece müşteri verilerini depolar
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string Address { get; set; }
}

// Veri erişimi - veritabanı işlemlerinden sorumlu
public class CustomerRepository
{
    public void Save(Customer customer)
    {
        // Müşteriyi veritabanına kaydetme kodu
        string sql = $"INSERT INTO Customers (Name, Email, Address) VALUES ('{customer.Name}', '{customer.Email}', '{customer.Address}')";
        // SQL çalıştırma kodu...
        Console.WriteLine($"Müşteri veritabanına kaydedildi: {customer.Name}");
    }
    
    public void Delete(int customerId)
    {
        // Müşteriyi veritabanından silme kodu
        string sql = $"DELETE FROM Customers WHERE Id = {customerId}";
        // SQL çalıştırma kodu...
        Console.WriteLine($"Müşteri veritabanından silindi, ID: {customerId}");
    }
    
    public Customer GetById(int customerId)
    {
        // Müşteriyi veritabanından getirme kodu
        Console.WriteLine($"Müşteri veritabanından alındı, ID: {customerId}");
        return new Customer { Id = customerId, Name = "Test Müşteri", Email = "test@example.com", Address = "Test Adresi" };
    }
}

// E-posta servisi - e-posta işlemlerinden sorumlu
public class EmailService
{
    public void SendWelcomeEmail(Customer customer)
    {
        // E-posta gönderme kodu
        string subject = "Hoş Geldiniz!";
        string body = $"Sayın {customer.Name}, sitemize hoş geldiniz.";
        // SMTP kodu...
        Console.WriteLine($"Hoş geldiniz e-postası gönderildi: {customer.Email}");
    }
    
    public void SendPromotionEmail(Customer customer, string promotionCode)
    {
        // Promosyon e-postası gönderme kodu
        string subject = "Özel Teklif!";
        string body = $"Sayın {customer.Name}, size özel promosyon kodunuz: {promotionCode}";
        // SMTP kodu...
        Console.WriteLine($"Promosyon e-postası gönderildi: {customer.Email}");
    }
}

// Doğrulama servisi - veri doğrulama işlemlerinden sorumlu
public class CustomerValidator
{
    public bool ValidateEmail(string email)
    {
        // E-posta doğrulama kodu
        return email.Contains("@") && email.Contains(".");
    }
    
    public bool ValidateAddress(string address)
    {
        // Adres doğrulama kodu
        return !string.IsNullOrWhiteSpace(address) && address.Length > 10;
    }
    
    public bool ValidateCustomer(Customer customer)
    {
        return ValidateEmail(customer.Email) && ValidateAddress(customer.Address);
    }
}

// Müşteri servisi - müşteri işlemlerini koordine eder
public class CustomerService
{
    private readonly CustomerRepository _repository;
    private readonly EmailService _emailService;
    private readonly CustomerValidator _validator;
    
    public CustomerService(CustomerRepository repository, EmailService emailService, CustomerValidator validator)
    {
        _repository = repository;
        _emailService = emailService;
        _validator = validator;
    }
    
    public void RegisterCustomer(Customer customer)
    {
        // Müşteriyi doğrula
        if (!_validator.ValidateCustomer(customer))
        {
            throw new ArgumentException("Müşteri bilgileri geçerli değil.");
        }
        
        // Müşteriyi kaydet
        _repository.Save(customer);
        
        // Hoş geldiniz e-postası gönder
        _emailService.SendWelcomeEmail(customer);
    }
}
```

## 2. Cohesion ve Coupling Kavramları

SRP ile yakından ilişkili iki önemli kavram vardır: cohesion (bağlılık) ve coupling (bağımlılık).

### Cohesion (Bağlılık)

Cohesion, bir sınıfın veya modülün içindeki öğelerin birbiriyle ne kadar ilişkili olduğunu ifade eder. Yüksek cohesion, bir sınıfın tek bir işi iyi yapması anlamına gelir ve genellikle arzu edilen bir özelliktir.

**Düşük Cohesion Örneği:**
```csharp
// Düşük cohesion - birbiriyle ilgisiz işlevler
public class Utility
{
    public void LogError(string message) { /* ... */ }
    public decimal CalculateTax(decimal amount) { /* ... */ }
    public void SendEmail(string to, string subject) { /* ... */ }
    public bool ValidatePassword(string password) { /* ... */ }
}
```

**Yüksek Cohesion Örneği:**
```csharp
// Yüksek cohesion - ilgili işlevler bir arada
public class Logger
{
    public void LogError(string message) { /* ... */ }
    public void LogWarning(string message) { /* ... */ }
    public void LogInfo(string message) { /* ... */ }
}

public class TaxCalculator
{
    public decimal CalculateIncomeTax(decimal income) { /* ... */ }
    public decimal CalculateSalesTax(decimal amount) { /* ... */ }
}
```

### Coupling (Bağımlılık)

Coupling, bir sınıfın diğer sınıflara ne kadar bağımlı olduğunu ifade eder. Düşük coupling (gevşek bağımlılık), bir sınıfın diğer sınıflardaki değişikliklerden minimum düzeyde etkilenmesi anlamına gelir ve genellikle arzu edilen bir özelliktir.

**Yüksek Coupling Örneği:**
```csharp
// Yüksek coupling - doğrudan bağımlılık
public class OrderProcessor
{
    public void ProcessOrder(Order order)
    {
        // Doğrudan bağımlılık
        SqlConnection connection = new SqlConnection("connection_string");
        connection.Open();
        
        // Veritabanı işlemleri...
        
        // E-posta gönderme
        SmtpClient smtpClient = new SmtpClient("smtp.example.com");
        smtpClient.Send(new MailMessage("orders@example.com", order.CustomerEmail, "Sipariş Onayı", "Siparişiniz alındı."));
        
        // Ödeme işlemi
        PaymentGateway gateway = new PaymentGateway("api_key");
        gateway.ProcessPayment(order.PaymentDetails);
    }
}
```

**Düşük Coupling Örneği:**
```csharp
// Düşük coupling - bağımlılık enjeksiyonu
public class OrderProcessor
{
    private readonly IOrderRepository _orderRepository;
    private readonly IEmailService _emailService;
    private readonly IPaymentService _paymentService;
    
    public OrderProcessor(
        IOrderRepository orderRepository,
        IEmailService emailService,
        IPaymentService paymentService)
    {
        _orderRepository = orderRepository;
        _emailService = emailService;
        _paymentService = paymentService;
    }
    
    public void ProcessOrder(Order order)
    {
        // Veritabanı işlemleri
        _orderRepository.Save(order);
        
        // E-posta gönderme
        _emailService.SendOrderConfirmation(order);
        
        // Ödeme işlemi
        _paymentService.ProcessPayment(order.PaymentDetails);
    }
}
```

## 3. SRP İhlallerini Tespit Etme ve Düzeltme

### SRP İhlallerini Tespit Etme

Bir sınıfın SRP'yi ihlal edip etmediğini anlamak için şu soruları sorabilirsiniz:

1. Sınıfın birden fazla nedenden dolayı değişmesi gerekebilir mi?
2. Sınıf birden fazla aktöre (kullanıcı, sistem, süreç) hizmet ediyor mu?
3. Sınıfın sorumlulukları tek bir cümlede özetlenebilir mi?
4. Sınıf çok sayıda metot ve özelliğe sahip mi?
5. Sınıfın bazı metotları, sınıfın diğer özelliklerini veya metotlarını kullanmıyor mu?

### SRP İhlallerini Düzeltme

SRP ihlallerini düzeltmek için genellikle şu adımları izleriz:

1. Sınıfın sorumluluklarını belirleyin
2. Her sorumluluk için ayrı bir sınıf oluşturun
3. Gerekirse, bu sınıfları koordine eden bir servis sınıfı ekleyin

**Örnek: Rapor Oluşturma Sınıfı**

SRP ihlali:
```csharp
// SRP ihlali
public class SalesReport
{
    public void GenerateReport(DateTime startDate, DateTime endDate)
    {
        // Veritabanından satış verilerini çekme
        List<Sale> sales = GetSalesData(startDate, endDate);
        
        // Verileri analiz etme
        decimal totalSales = CalculateTotalSales(sales);
        decimal averageSale = CalculateAverageSale(sales);
        
        // Rapor dosyası oluşturma
        string reportContent = $"Satış Raporu ({startDate:d} - {endDate:d})\n";
        reportContent += $"Toplam Satış: {totalSales:C}\n";
        reportContent += $"Ortalama Satış: {averageSale:C}\n";
        reportContent += "Satış Detayları:\n";
        
        foreach (var sale in sales)
        {
            reportContent += $"{sale.Date:d} - {sale.ProductName} - {sale.Amount:C}\n";
        }
        
        // Dosyaya yazma
        File.WriteAllText("SalesReport.txt", reportContent);
        
        // E-posta gönderme
        SendReportByEmail("manager@example.com", "Satış Raporu", reportContent);
    }
    
    private List<Sale> GetSalesData(DateTime startDate, DateTime endDate)
    {
        // Veritabanı sorgusu...
        return new List<Sale>(); // Örnek veri
    }
    
    private decimal CalculateTotalSales(List<Sale> sales)
    {
        return sales.Sum(s => s.Amount);
    }
    
    private decimal CalculateAverageSale(List<Sale> sales)
    {
        return sales.Count > 0 ? sales.Average(s => s.Amount) : 0;
    }
    
    private void SendReportByEmail(string to, string subject, string body)
    {
        // E-posta gönderme kodu...
        Console.WriteLine($"E-posta gönderildi: {to}");
    }
}

public class Sale
{
    public DateTime Date { get; set; }
    public string ProductName { get; set; }
    public decimal Amount { get; set; }
}
```

SRP uyumlu tasarım:
```csharp
// Veri erişim sınıfı
public class SalesRepository
{
    public List<Sale> GetSalesData(DateTime startDate, DateTime endDate)
    {
        // Veritabanı sorgusu...
        return new List<Sale>(); // Örnek veri
    }
}

// Analiz sınıfı
public class SalesAnalyzer
{
    public decimal CalculateTotalSales(List<Sale> sales)
    {
        return sales.Sum(s => s.Amount);
    }
    
    public decimal CalculateAverageSale(List<Sale> sales)
    {
        return sales.Count > 0 ? sales.Average(s => s.Amount) : 0;
    }
}

// Rapor oluşturma sınıfı
public class ReportGenerator
{
    public string GenerateReportContent(DateTime startDate, DateTime endDate, List<Sale> sales, decimal totalSales, decimal averageSale)
    {
        string reportContent = $"Satış Raporu ({startDate:d} - {endDate:d})\n";
        reportContent += $"Toplam Satış: {totalSales:C}\n";
        reportContent += $"Ortalama Satış: {averageSale:C}\n";
        reportContent += "Satış Detayları:\n";
        
        foreach (var sale in sales)
        {
            reportContent += $"{sale.Date:d} - {sale.ProductName} - {sale.Amount:C}\n";
        }
        
        return reportContent;
    }
}

// Dosya işlemleri sınıfı
public class FileManager
{
    public void SaveToFile(string filePath, string content)
    {
        File.WriteAllText(filePath, content);
    }
}

// E-posta servisi
public class EmailService
{
    public void SendEmail(string to, string subject, string body)
    {
        // E-posta gönderme kodu...
        Console.WriteLine($"E-posta gönderildi: {to}");
    }
}

// Koordinasyon sınıfı
public class SalesReportService
{
    private readonly SalesRepository _repository;
    private readonly SalesAnalyzer _analyzer;
    private readonly ReportGenerator _reportGenerator;
    private readonly FileManager _fileManager;
    private readonly EmailService _emailService;
    
    public SalesReportService(
        SalesRepository repository,
        SalesAnalyzer analyzer,
        ReportGenerator reportGenerator,
        FileManager fileManager,
        EmailService emailService)
    {
        _repository = repository;
        _analyzer = analyzer;
        _reportGenerator = reportGenerator;
        _fileManager = fileManager;
        _emailService = emailService;
    }
    
    public void GenerateAndSendReport(DateTime startDate, DateTime endDate, string filePath, string emailTo)
    {
        // Veritabanından satış verilerini çekme
        List<Sale> sales = _repository.GetSalesData(startDate, endDate);
        
        // Verileri analiz etme
        decimal totalSales = _analyzer.CalculateTotalSales(sales);
        decimal averageSale = _analyzer.CalculateAverageSale(sales);
        
        // Rapor içeriği oluşturma
        string reportContent = _reportGenerator.GenerateReportContent(startDate, endDate, sales, totalSales, averageSale);
        
        // Dosyaya kaydetme
        _fileManager.SaveToFile(filePath, reportContent);
        
        // E-posta gönderme
        _emailService.SendEmail(emailTo, "Satış Raporu", reportContent);
    }
}
```

## En İyi Pratikler

1. **Sınıfları Küçük ve Odaklanmış Tutun**
   - Her sınıfın tek bir sorumluluğu olmalıdır.
   - Bir sınıfın sorumluluğunu tek bir cümlede ifade edebilmelisiniz.
   - Eğer bir sınıfın açıklaması "ve" içeriyorsa, muhtemelen birden fazla sorumluluğu vardır.

2. **Sorumlulukları Doğru Tanımlayın**
   - Sorumlulukları "değişim nedenleri" açısından düşünün.
   - Bir sınıfın değişmesi için yalnızca bir neden olmalıdır.
   - Farklı aktörlere (kullanıcı, sistem, süreç) hizmet eden sorumlulukları ayırın.

3. **Bağımlılık Enjeksiyonu Kullanın**
   - Sınıflar arasındaki bağımlılıkları azaltmak için bağımlılık enjeksiyonu kullanın.
   - Somut sınıflar yerine arayüzlere (interface) bağımlı olun.
   - Bu, kodunuzu daha test edilebilir ve değiştirilebilir hale getirir.

4. **Facade Desenini Kullanın**
   - Karmaşık alt sistemleri basit bir arayüz arkasında gizlemek için Facade desenini kullanın.
   - Bu, istemci kodunun karmaşıklıkla başa çıkmasına gerek kalmadan alt sistemlerle etkileşim kurmasını sağlar.

5. **Refactoring İçin İşaretleri Tanıyın**
   - Büyük sınıflar genellikle SRP ihlalinin işaretidir.
   - Çok sayıda import/using ifadesi, sınıfın çok fazla bağımlılığı olduğunu gösterebilir.
   - Sınıf adının çok genel olması (örn. "Manager", "Processor") genellikle çok fazla sorumluluğa işaret eder.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **God Sınıflar Oluşturmak**
   ```csharp
   // Kötü - "God Class" (her şeyi yapan sınıf)
   public class OrderManager
   {
       public void CreateOrder() { /* ... */ }
       public void UpdateOrder() { /* ... */ }
       public void CancelOrder() { /* ... */ }
       public void ProcessPayment() { /* ... */ }
       public void GenerateInvoice() { /* ... */ }
       public void SendOrderConfirmation() { /* ... */ }
       public void UpdateInventory() { /* ... */ }
       public void CalculateShipping() { /* ... */ }
       public void ApplyDiscounts() { /* ... */ }
       // ... ve daha fazlası
   }
   
   // İyi - ayrılmış sorumluluklar
   public class OrderService { /* ... */ }
   public class PaymentService { /* ... */ }
   public class InvoiceService { /* ... */ }
   public class NotificationService { /* ... */ }
   public class InventoryService { /* ... */ }
   public class ShippingCalculator { /* ... */ }
   public class DiscountService { /* ... */ }
   ```

2. **Utility Sınıfları Kötüye Kullanmak**
   ```csharp
   // Kötü - her şeyi içeren utility sınıfı
   public static class Utilities
   {
       public static void LogError(string message) { /* ... */ }
       public static decimal CalculateTax(decimal amount) { /* ... */ }
       public static bool ValidateEmail(string email) { /* ... */ }
       public static void SendEmail(string to, string subject) { /* ... */ }
       public static string FormatCurrency(decimal amount) { /* ... */ }
       // ... ve daha fazlası
   }
   
   // İyi - ayrılmış utility sınıfları
   public static class LoggingUtilities { /* ... */ }
   public static class TaxCalculationUtilities { /* ... */ }
   public static class ValidationUtilities { /* ... */ }
   public static class EmailUtilities { /* ... */ }
   public static class FormattingUtilities { /* ... */ }
   ```

3. **Veri ve Davranışı Karıştırmak**
   ```csharp
   // Kötü - veri ve davranış karışımı
   public class Invoice
   {
       public int Id { get; set; }
       public decimal Amount { get; set; }
       public DateTime Date { get; set; }
       
       public void SaveToDatabase() { /* ... */ }
       public void SendByEmail(string email) { /* ... */ }
       public void Print() { /* ... */ }
   }
   
   // İyi - ayrılmış veri ve davranış
   public class Invoice
   {
       public int Id { get; set; }
       public decimal Amount { get; set; }
       public DateTime Date { get; set; }
   }
   
   public class InvoiceRepository
   {
       public void Save(Invoice invoice) { /* ... */ }
   }
   
   public class InvoiceEmailService
   {
       public void SendInvoice(Invoice invoice, string email) { /* ... */ }
   }
   
   public class InvoicePrinter
   {
       public void Print(Invoice invoice) { /* ... */ }
   }
   ```

4. **Aşırı Parçalama**
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
       public bool ValidateCustomer(Customer customer) { /* ... */ }
   }
   ```

5. **Sınıf İsimlerini Belirsiz Tutmak**
   ```csharp
   // Kötü - belirsiz sınıf isimleri
   public class Manager { /* ... */ }
   public class Processor { /* ... */ }
   public class Handler { /* ... */ }
   
   // İyi - açıklayıcı sınıf isimleri
   public class OrderManager { /* ... */ }
   public class PaymentProcessor { /* ... */ }
   public class ExceptionHandler { /* ... */ }
   ```

Single Responsibility Principle, kodunuzu daha modüler, bakımı kolay ve genişletilebilir hale getirmenin temel bir yoludur. Bu prensibi doğru uygulayarak, değişikliklerin etkisini sınırlayabilir, test edilebilirliği artırabilir ve kodunuzun genel kalitesini iyileştirebilirsiniz. 