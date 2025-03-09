# DRY (Don't Repeat Yourself)

DRY (Don't Repeat Yourself - Kendini Tekrar Etme) prensibi, yazılım geliştirmede temel bir ilkedir ve "Her bilgi parçası, bir sistem içinde tek ve kesin bir temsile sahip olmalıdır" şeklinde ifade edilir. Bu prensip, 1999 yılında Andy Hunt ve Dave Thomas tarafından "The Pragmatic Programmer" kitabında tanıtılmıştır.

## 1. DRY Prensibinin Temel Kavramı

DRY prensibinin özü, kod tekrarını önlemektir. Aynı veya benzer kodun birden fazla yerde bulunması, bakım sorunlarına, hatalara ve tutarsızlıklara yol açabilir. DRY prensibi, kodun yeniden kullanılabilirliğini teşvik eder ve değişikliklerin tek bir yerde yapılmasını sağlar.

### DRY İhlali Örneği

```csharp
// DRY ihlali - Tekrarlanan kod
public class CustomerService
{
    public void RegisterCustomer(Customer customer)
    {
        // E-posta doğrulama
        if (string.IsNullOrEmpty(customer.Email))
        {
            throw new ArgumentException("E-posta adresi boş olamaz.");
        }
        
        if (!customer.Email.Contains("@"))
        {
            throw new ArgumentException("Geçersiz e-posta formatı.");
        }
        
        // Telefon doğrulama
        if (string.IsNullOrEmpty(customer.Phone))
        {
            throw new ArgumentException("Telefon numarası boş olamaz.");
        }
        
        if (!customer.Phone.All(c => char.IsDigit(c) || c == '+' || c == '-' || c == ' '))
        {
            throw new ArgumentException("Geçersiz telefon numarası formatı.");
        }
        
        // Müşteri kaydetme işlemi
        // ...
    }
    
    public void UpdateCustomer(Customer customer)
    {
        // E-posta doğrulama (tekrar)
        if (string.IsNullOrEmpty(customer.Email))
        {
            throw new ArgumentException("E-posta adresi boş olamaz.");
        }
        
        if (!customer.Email.Contains("@"))
        {
            throw new ArgumentException("Geçersiz e-posta formatı.");
        }
        
        // Telefon doğrulama (tekrar)
        if (string.IsNullOrEmpty(customer.Phone))
        {
            throw new ArgumentException("Telefon numarası boş olamaz.");
        }
        
        if (!customer.Phone.All(c => char.IsDigit(c) || c == '+' || c == '-' || c == ' '))
        {
            throw new ArgumentException("Geçersiz telefon numarası formatı.");
        }
        
        // Müşteri güncelleme işlemi
        // ...
    }
}
```

Bu örnekte, e-posta ve telefon doğrulama kodları hem `RegisterCustomer` hem de `UpdateCustomer` metotlarında tekrarlanmıştır. Bu, DRY prensibinin ihlalidir.

### DRY Uyumlu Tasarım

```csharp
// DRY uyumlu tasarım - Tekrarlanan kod çıkarılmış
public class CustomerService
{
    public void RegisterCustomer(Customer customer)
    {
        ValidateCustomer(customer);
        
        // Müşteri kaydetme işlemi
        // ...
    }
    
    public void UpdateCustomer(Customer customer)
    {
        ValidateCustomer(customer);
        
        // Müşteri güncelleme işlemi
        // ...
    }
    
    private void ValidateCustomer(Customer customer)
    {
        ValidateEmail(customer.Email);
        ValidatePhone(customer.Phone);
    }
    
    private void ValidateEmail(string email)
    {
        if (string.IsNullOrEmpty(email))
        {
            throw new ArgumentException("E-posta adresi boş olamaz.");
        }
        
        if (!email.Contains("@"))
        {
            throw new ArgumentException("Geçersiz e-posta formatı.");
        }
    }
    
    private void ValidatePhone(string phone)
    {
        if (string.IsNullOrEmpty(phone))
        {
            throw new ArgumentException("Telefon numarası boş olamaz.");
        }
        
        if (!phone.All(c => char.IsDigit(c) || c == '+' || c == '-' || c == ' '))
        {
            throw new ArgumentException("Geçersiz telefon numarası formatı.");
        }
    }
}
```

Bu tasarımda, doğrulama kodları ayrı metotlara çıkarılmıştır, böylece kod tekrarı önlenmiştir. Değişiklikler tek bir yerde yapılabilir ve tüm kullanım yerlerinde otomatik olarak uygulanır.

## 2. DRY Prensibinin Uygulama Alanları

DRY prensibi, yazılım geliştirmenin birçok alanında uygulanabilir:

### 1. Kod Düzeyinde DRY

Kod düzeyinde DRY, aynı veya benzer kod bloklarının tekrarlanmasını önlemeyi amaçlar. Bunu başarmak için:

- **Metotlar ve Fonksiyonlar**: Tekrarlanan kod bloklarını metotlara çıkarın.
- **Sınıflar ve Modüller**: İlgili işlevselliği sınıflar ve modüller halinde gruplandırın.
- **Kalıtım ve Kompozisyon**: Ortak davranışları paylaşmak için kalıtım ve kompozisyon kullanın.
- **Genişletme Metotları**: Mevcut sınıfları genişletmek için extension metotları kullanın.

```csharp
// Extension metot örneği
public static class StringExtensions
{
    public static bool IsValidEmail(this string email)
    {
        return !string.IsNullOrEmpty(email) && email.Contains("@");
    }
    
    public static bool IsValidPhone(this string phone)
    {
        return !string.IsNullOrEmpty(phone) && 
               phone.All(c => char.IsDigit(c) || c == '+' || c == '-' || c == ' ');
    }
}

// Kullanım
public class CustomerService
{
    public void RegisterCustomer(Customer customer)
    {
        if (!customer.Email.IsValidEmail())
        {
            throw new ArgumentException("Geçersiz e-posta adresi.");
        }
        
        if (!customer.Phone.IsValidPhone())
        {
            throw new ArgumentException("Geçersiz telefon numarası.");
        }
        
        // Müşteri kaydetme işlemi
        // ...
    }
}
```

### 2. Veri Düzeyinde DRY

Veri düzeyinde DRY, veri tekrarını önlemeyi amaçlar. Bunu başarmak için:

- **Veritabanı Normalizasyonu**: Veritabanı tablolarını normalize ederek veri tekrarını azaltın.
- **Merkezi Yapılandırma**: Yapılandırma ayarlarını merkezi bir yerde saklayın.
- **Sabitler ve Enum'lar**: Tekrarlanan değerleri sabitler ve enum'lar olarak tanımlayın.

```csharp
// Sabitler ve enum'lar örneği
public enum CustomerStatus
{
    Active,
    Inactive,
    Suspended,
    Deleted
}

public static class ValidationMessages
{
    public const string EmailRequired = "E-posta adresi boş olamaz.";
    public const string InvalidEmailFormat = "Geçersiz e-posta formatı.";
    public const string PhoneRequired = "Telefon numarası boş olamaz.";
    public const string InvalidPhoneFormat = "Geçersiz telefon numarası formatı.";
}

// Kullanım
public class CustomerService
{
    public void ValidateEmail(string email)
    {
        if (string.IsNullOrEmpty(email))
        {
            throw new ArgumentException(ValidationMessages.EmailRequired);
        }
        
        if (!email.Contains("@"))
        {
            throw new ArgumentException(ValidationMessages.InvalidEmailFormat);
        }
    }
}
```

### 3. Tasarım Düzeyinde DRY

Tasarım düzeyinde DRY, mimari tekrarını önlemeyi amaçlar. Bunu başarmak için:

- **Tasarım Desenleri**: Ortak problemleri çözmek için tasarım desenleri kullanın.
- **Katmanlı Mimari**: İşlevselliği mantıksal katmanlara ayırın.
- **Servis Odaklı Mimari (SOA)**: Ortak işlevselliği servisler olarak sunun.
- **Mikroservisler**: Bağımsız, yeniden kullanılabilir servisler oluşturun.

```csharp
// Repository pattern örneği
public interface IRepository<T>
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
```

## 3. DRY ve Diğer Prensipler Arasındaki İlişki

DRY prensibi, diğer yazılım geliştirme prensipleriyle yakından ilişkilidir:

### DRY ve SOLID

- **Single Responsibility Principle (SRP)**: Her iki prensip de kodun modülerliğini teşvik eder. SRP, bir sınıfın tek bir sorumluluğu olması gerektiğini söylerken, DRY, kod tekrarını önlemeyi amaçlar.
- **Open/Closed Principle (OCP)**: DRY, kodun yeniden kullanılabilirliğini artırarak, mevcut kodu değiştirmeden genişletmeyi kolaylaştırır.
- **Dependency Inversion Principle (DIP)**: Soyutlamalar kullanarak, DRY prensibini uygulamak daha kolay hale gelir.

### DRY ve KISS (Keep It Simple, Stupid)

DRY ve KISS prensipleri genellikle birbirini tamamlar. DRY, kod tekrarını azaltarak karmaşıklığı azaltır, bu da KISS prensibine uygundur. Ancak, bazen aşırı DRY uygulaması, kodun anlaşılmasını zorlaştırabilir ve KISS prensibini ihlal edebilir.

### DRY ve YAGNI (You Aren't Gonna Need It)

DRY, mevcut kod tekrarını azaltmayı amaçlarken, YAGNI, gelecekteki potansiyel kullanım için kod yazılmaması gerektiğini söyler. Bu iki prensip arasında bir denge kurmak önemlidir.

## 4. Gerçek Hayattan DRY Örnekleri

### Örnek 1: Doğrulama Mantığı

```csharp
// Doğrulama mantığı için FluentValidation kütüphanesi kullanımı
public class CustomerValidator : AbstractValidator<Customer>
{
    public CustomerValidator()
    {
        RuleFor(x => x.Email)
            .NotEmpty().WithMessage("E-posta adresi boş olamaz.")
            .EmailAddress().WithMessage("Geçersiz e-posta formatı.");
            
        RuleFor(x => x.Phone)
            .NotEmpty().WithMessage("Telefon numarası boş olamaz.")
            .Matches(@"^[\d\+\-\s]+$").WithMessage("Geçersiz telefon numarası formatı.");
    }
}

// Kullanım
public class CustomerService
{
    private readonly CustomerValidator _validator;
    
    public CustomerService(CustomerValidator validator)
    {
        _validator = validator;
    }
    
    public void RegisterCustomer(Customer customer)
    {
        var result = _validator.Validate(customer);
        if (!result.IsValid)
        {
            throw new ValidationException(result.Errors);
        }
        
        // Müşteri kaydetme işlemi
        // ...
    }
    
    public void UpdateCustomer(Customer customer)
    {
        var result = _validator.Validate(customer);
        if (!result.IsValid)
        {
            throw new ValidationException(result.Errors);
        }
        
        // Müşteri güncelleme işlemi
        // ...
    }
}
```

### Örnek 2: İş Mantığı

```csharp
// İş mantığı için servis katmanı kullanımı
public interface IDiscountService
{
    decimal CalculateDiscount(Order order);
}

public class DiscountService : IDiscountService
{
    public decimal CalculateDiscount(Order order)
    {
        decimal discount = 0;
        
        // Sipariş tutarına göre indirim
        if (order.TotalAmount > 1000)
        {
            discount += order.TotalAmount * 0.1m; // %10 indirim
        }
        else if (order.TotalAmount > 500)
        {
            discount += order.TotalAmount * 0.05m; // %5 indirim
        }
        
        // Sadakat programı indirimi
        if (order.Customer.IsMember && order.Customer.MembershipYears > 2)
        {
            discount += order.TotalAmount * 0.03m; // %3 ek indirim
        }
        
        return discount;
    }
}

// Kullanım
public class OrderService
{
    private readonly IDiscountService _discountService;
    
    public OrderService(IDiscountService discountService)
    {
        _discountService = discountService;
    }
    
    public void PlaceOrder(Order order)
    {
        // İndirim hesaplama
        decimal discount = _discountService.CalculateDiscount(order);
        order.Discount = discount;
        order.FinalAmount = order.TotalAmount - discount;
        
        // Sipariş işleme
        // ...
    }
    
    public decimal CalculateOrderPreview(Order order)
    {
        // İndirim hesaplama (aynı mantık tekrar kullanılıyor)
        decimal discount = _discountService.CalculateDiscount(order);
        return order.TotalAmount - discount;
    }
}
```

### Örnek 3: Kullanıcı Arayüzü

```csharp
// Kullanıcı arayüzü için partial view kullanımı (ASP.NET MVC)
// _CustomerForm.cshtml (Partial View)
@model CustomerViewModel

<div class="form-group">
    <label asp-for="Name"></label>
    <input asp-for="Name" class="form-control" />
    <span asp-validation-for="Name" class="text-danger"></span>
</div>

<div class="form-group">
    <label asp-for="Email"></label>
    <input asp-for="Email" class="form-control" />
    <span asp-validation-for="Email" class="text-danger"></span>
</div>

<div class="form-group">
    <label asp-for="Phone"></label>
    <input asp-for="Phone" class="form-control" />
    <span asp-validation-for="Phone" class="text-danger"></span>
</div>

// Create.cshtml
@model CustomerViewModel

<h2>Yeni Müşteri Ekle</h2>

<form asp-action="Create" method="post">
    <partial name="_CustomerForm" model="Model" />
    
    <button type="submit" class="btn btn-primary">Kaydet</button>
</form>

// Edit.cshtml
@model CustomerViewModel

<h2>Müşteri Düzenle</h2>

<form asp-action="Edit" method="post">
    <input type="hidden" asp-for="Id" />
    
    <partial name="_CustomerForm" model="Model" />
    
    <button type="submit" class="btn btn-primary">Güncelle</button>
</form>
```

## 5. En İyi Pratikler

DRY prensibini etkili bir şekilde uygulamak için aşağıdaki en iyi pratikleri izleyin:

1. **Tekrarlanan Kodu Tanımlayın**
   - Kod tabanınızı düzenli olarak gözden geçirin ve tekrarlanan kod bloklarını belirleyin.
   - Kod klonlama tespit araçları kullanın (örn. SonarQube, NDepend).
   - Code review süreçlerinde tekrarlanan kodu tespit etmeye özen gösterin.

2. **Anlamlı Soyutlamalar Oluşturun**
   - Tekrarlanan kodu çıkarırken, anlamlı ve tutarlı soyutlamalar oluşturun.
   - Metot ve sınıf isimlerini, işlevselliği açıkça yansıtacak şekilde seçin.
   - Soyutlamaları, tek bir sorumluluğa sahip olacak şekilde tasarlayın (SRP).

3. **Uygun Seviyede Soyutlama Yapın**
   - Aşırı soyutlamadan kaçının, bu kodun anlaşılmasını zorlaştırabilir.
   - Soyutlama seviyesini, projenin karmaşıklığına ve ekibin deneyimine göre ayarlayın.
   - "Üç kez tekrarlanana kadar soyutlama yapma" kuralını göz önünde bulundurun.

4. **Kütüphaneler ve Çerçeveler Kullanın**
   - Yaygın problemleri çözmek için mevcut kütüphaneleri ve çerçeveleri kullanın.
   - Kendi çözümünüzü oluşturmadan önce, mevcut çözümleri araştırın.
   - Şirket içi ortak kütüphaneler oluşturun ve kullanın.

5. **Kod Üretimi ve Şablonlar Kullanın**
   - Tekrarlanan kod yapıları için kod üretimi ve şablonlar kullanın.
   - T4 şablonları, code generators veya source generators gibi araçlardan yararlanın.
   - Tekrarlanan kod yapıları için IDE snippet'leri oluşturun.

6. **Tasarım Desenleri Uygulayın**
   - Yaygın problemleri çözmek için uygun tasarım desenleri kullanın.
   - Factory, Strategy, Template Method gibi desenler, kod tekrarını azaltmaya yardımcı olabilir.
   - Desenler hakkında bilgi edinin ve ne zaman kullanılacaklarını öğrenin.

7. **Sürekli Refactoring Yapın**
   - Kod tabanını sürekli olarak iyileştirin ve tekrarlanan kodu çıkarın.
   - "Boy Scout Rule" uygulayın: Kodu bulduğunuzdan daha temiz bırakın.
   - Teknik borcu azaltmak için düzenli refactoring seansları planlayın.

## 6. Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Aşırı DRY Uygulaması**
   ```csharp
   // Kötü - aşırı genelleştirme
   public class GenericProcessor<T, TResult>
   {
       public TResult Process(T input, Func<T, TResult> processor, 
                             Action<T> preProcess = null, 
                             Action<TResult> postProcess = null)
       {
           preProcess?.Invoke(input);
           var result = processor(input);
           postProcess?.Invoke(result);
           return result;
       }
   }
   
   // İyi - daha spesifik ve anlaşılır
   public class OrderProcessor
   {
       public decimal CalculateTotal(Order order)
       {
           ValidateOrder(order);
           var total = order.Items.Sum(item => item.Price * item.Quantity);
           ApplyDiscounts(order, ref total);
           return total;
       }
       
       private void ValidateOrder(Order order)
       {
           // Doğrulama mantığı
       }
       
       private void ApplyDiscounts(Order order, ref decimal total)
       {
           // İndirim mantığı
       }
   }
   ```

2. **Prematüre Soyutlama**
   ```csharp
   // Kötü - henüz gerekmeyen soyutlama
   public interface IUserRepository
   {
       User GetById(int id);
       IEnumerable<User> GetAll();
       void Add(User user);
       void Update(User user);
       void Delete(int id);
       IEnumerable<User> FindByName(string name);
       IEnumerable<User> FindByEmail(string email);
       IEnumerable<User> FindByRole(string role);
       // ... ve daha birçok metot
   }
   
   // İyi - başlangıçta daha basit
   public interface IUserRepository
   {
       User GetById(int id);
       IEnumerable<User> GetAll();
       void Add(User user);
       void Update(User user);
       void Delete(int id);
   }
   ```

3. **Bağlam Göz Ardı Etme**
   ```csharp
   // Kötü - farklı bağlamlar için aynı soyutlama
   public class ValidationHelper
   {
       public static bool IsValid(string value, string type)
       {
           switch (type)
           {
               case "email":
                   return !string.IsNullOrEmpty(value) && value.Contains("@");
               case "phone":
                   return !string.IsNullOrEmpty(value) && value.All(c => char.IsDigit(c) || c == '+' || c == '-' || c == ' ');
               case "zipcode":
                   return !string.IsNullOrEmpty(value) && value.Length == 5 && value.All(char.IsDigit);
               default:
                   return true;
           }
       }
   }
   
   // İyi - bağlama özgü doğrulama
   public class EmailValidator
   {
       public bool IsValid(string email)
       {
           return !string.IsNullOrEmpty(email) && email.Contains("@");
       }
   }
   
   public class PhoneValidator
   {
       public bool IsValid(string phone)
       {
           return !string.IsNullOrEmpty(phone) && phone.All(c => char.IsDigit(c) || c == '+' || c == '-' || c == ' ');
       }
   }
   
   public class ZipCodeValidator
   {
       public bool IsValid(string zipCode)
       {
           return !string.IsNullOrEmpty(zipCode) && zipCode.Length == 5 && zipCode.All(char.IsDigit);
       }
   }
   ```

4. **Yanlış Soyutlama Seviyesi**
   ```csharp
   // Kötü - çok düşük seviyede soyutlama
   public class StringHelper
   {
       public static string Capitalize(string input)
       {
           if (string.IsNullOrEmpty(input))
               return input;
               
           return char.ToUpper(input[0]) + input.Substring(1);
       }
   }
   
   // Kötü - çok yüksek seviyede soyutlama
   public interface IProcessor
   {
       object Process(object input);
   }
   
   // İyi - uygun seviyede soyutlama
   public interface IOrderProcessor
   {
       OrderResult Process(Order order);
   }
   ```

5. **WET (Write Everything Twice) Kodu Göz Ardı Etme**
   ```csharp
   // Kötü - tekrarlanan kod göz ardı edilmiş
   public class CustomerController
   {
       public IActionResult Create(CustomerViewModel model)
       {
           if (!ModelState.IsValid)
               return View(model);
               
           // Müşteri oluşturma mantığı
           // ...
           
           return RedirectToAction("Index");
       }
       
       public IActionResult Edit(int id, CustomerViewModel model)
       {
           if (!ModelState.IsValid)
               return View(model);
               
           // Müşteri güncelleme mantığı (benzer kod)
           // ...
           
           return RedirectToAction("Index");
       }
   }
   
   // İyi - ortak kod çıkarılmış
   public class CustomerController
   {
       private IActionResult HandleCustomerForm(CustomerViewModel model, Func<Customer, Task> action)
       {
           if (!ModelState.IsValid)
               return View(model);
               
           var customer = MapToCustomer(model);
           await action(customer);
           
           return RedirectToAction("Index");
       }
       
       public async Task<IActionResult> Create(CustomerViewModel model)
       {
           return await HandleCustomerForm(model, async customer => 
           {
               // Müşteri oluşturma mantığı
               await _customerService.CreateAsync(customer);
           });
       }
       
       public async Task<IActionResult> Edit(int id, CustomerViewModel model)
       {
           return await HandleCustomerForm(model, async customer => 
           {
               customer.Id = id;
               // Müşteri güncelleme mantığı
               await _customerService.UpdateAsync(customer);
           });
       }
       
       private Customer MapToCustomer(CustomerViewModel model)
       {
           // Mapping mantığı
           // ...
       }
   }
   ```

DRY prensibi, yazılım geliştirmede kod kalitesini artırmanın temel bir yoludur. Bu prensibi doğru uygulayarak, kodunuzu daha bakımı kolay, daha az hata içeren ve daha esnek hale getirebilirsiniz. Ancak, DRY'ı aşırıya kaçmadan, diğer prensipler ve bağlam göz önünde bulundurularak uygulamak önemlidir. 