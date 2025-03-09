# Demeter Yasası (Law of Demeter)

Demeter Yasası (LoD) veya "En Az Bilgi" prensibi, yazılım geliştirmede nesneler arasındaki etkileşimi minimize etmeyi amaçlayan bir tasarım ilkesidir. Bu prensip, bir nesnenin yalnızca "yakın arkadaşları" ile konuşması gerektiğini savunur.

## 1. Demeter Yasası'nın Temel Kavramı

Demeter Yasası'na göre, bir metot yalnızca şunları çağırabilir:

1. Kendi sınıfının metotlarını
2. Metoda parametre olarak gelen nesnelerin metotlarını
3. Metot içinde oluşturulan nesnelerin metotlarını
4. Sınıfın doğrudan bağımlı olduğu nesnelerin metotlarını (instance değişkenleri)

Basitçe ifade etmek gerekirse: "Yalnızca doğrudan arkadaşlarınla konuş, yabancılarla konuşma."

### Demeter Yasası İhlali Örneği

```csharp
// Demeter Yasası ihlali - Zincirleme çağrılar
public class Customer
{
    public Address GetAddress()
    {
        return new Address();
    }
}

public class Address
{
    public City GetCity()
    {
        return new City();
    }
}

public class City
{
    public string GetZipCode()
    {
        return "34000";
    }
}

public class OrderProcessor
{
    public void ProcessOrder(Customer customer)
    {
        // Demeter Yasası ihlali - zincirleme çağrılar
        string zipCode = customer.GetAddress().GetCity().GetZipCode();
        
        // Sipariş işleme mantığı...
    }
}
```

Bu örnekte, `OrderProcessor` sınıfı, `Customer` nesnesinin iç yapısı hakkında çok fazla bilgiye sahiptir. `Address` ve `City` sınıflarının varlığını ve bu sınıfların metotlarını bilmektedir.

### Demeter Yasası Uyumlu Tasarım

```csharp
// Demeter Yasası uyumlu tasarım
public class Customer
{
    private Address _address;
    
    public string GetZipCode()
    {
        return _address.GetZipCode();
    }
}

public class Address
{
    private City _city;
    
    public string GetZipCode()
    {
        return _city.GetZipCode();
    }
}

public class City
{
    private string _zipCode;
    
    public string GetZipCode()
    {
        return _zipCode;
    }
}

public class OrderProcessor
{
    public void ProcessOrder(Customer customer)
    {
        // Demeter Yasası uyumlu - doğrudan müşteriden posta kodu alınıyor
        string zipCode = customer.GetZipCode();
        
        // Sipariş işleme mantığı...
    }
}
```

Bu tasarımda, her sınıf yalnızca kendi doğrudan bağımlılıklarıyla etkileşime girer. `OrderProcessor` sınıfı, `Customer` sınıfının iç yapısı hakkında bilgi sahibi değildir.

## 2. Demeter Yasası'nın Faydaları

1. **Düşük Bağımlılık**
   - Sınıflar arasındaki bağımlılık azalır
   - Değişiklikler daha az etki yaratır
   - Kodun bakımı kolaylaşır

2. **Daha İyi Kapsülleme**
   - İç yapı dışarıya kapatılır
   - Uygulama detayları gizlenir
   - Arayüzler daha temiz olur

3. **Daha Kolay Test Edilebilirlik**
   - Mock nesneleri daha kolay oluşturulur
   - Test kapsamı daha net olur
   - Bağımlılıklar daha kolay izole edilir

## 3. Demeter Yasası'nın Uygulanması

### 1. Tell, Don't Ask Prensibi

"Tell, Don't Ask" (Sor Değil, Söyle) prensibi, Demeter Yasası'nı uygulamanın bir yoludur. Bu prensibe göre, bir nesneye veri sorgulamak yerine, bir eylem gerçekleştirmesini söylemelisiniz.

```csharp
// Kötü - Ask yaklaşımı
public void ProcessPayment(Order order)
{
    if (order.GetCustomer().GetPaymentMethod().IsValid())
    {
        decimal amount = order.GetTotal();
        order.GetCustomer().GetPaymentMethod().Charge(amount);
    }
}

// İyi - Tell yaklaşımı
public void ProcessPayment(Order order)
{
    order.ProcessPayment();
}
```

### 2. Facade Pattern Kullanımı

Facade (Ön Yüz) tasarım deseni, karmaşık alt sistemleri basit bir arayüz arkasında gizleyerek Demeter Yasası'na uyumu kolaylaştırır.

```csharp
// Facade örneği
public class ShippingFacade
{
    private readonly AddressValidator _addressValidator;
    private readonly ShippingCalculator _shippingCalculator;
    private readonly LabelGenerator _labelGenerator;
    private readonly ShippingProvider _shippingProvider;
    
    public ShippingFacade(
        AddressValidator addressValidator,
        ShippingCalculator shippingCalculator,
        LabelGenerator labelGenerator,
        ShippingProvider shippingProvider)
    {
        _addressValidator = addressValidator;
        _shippingCalculator = shippingCalculator;
        _labelGenerator = labelGenerator;
        _shippingProvider = shippingProvider;
    }
    
    public ShippingResult ProcessShipping(Order order)
    {
        // Karmaşık alt sistemleri basit bir arayüz arkasında gizler
        bool isValid = _addressValidator.Validate(order.ShippingAddress);
        if (!isValid) return ShippingResult.InvalidAddress();
        
        decimal cost = _shippingCalculator.Calculate(order);
        string label = _labelGenerator.GenerateLabel(order);
        string trackingNumber = _shippingProvider.Ship(order, label);
        
        return ShippingResult.Success(cost, trackingNumber);
    }
}

// Kullanım
public class OrderService
{
    private readonly ShippingFacade _shippingFacade;
    
    public OrderService(ShippingFacade shippingFacade)
    {
        _shippingFacade = shippingFacade;
    }
    
    public void CompleteOrder(Order order)
    {
        // Demeter Yasası'na uygun - karmaşık alt sistemlerle doğrudan etkileşim yok
        ShippingResult result = _shippingFacade.ProcessShipping(order);
        
        if (result.Success)
        {
            order.SetShippingInfo(result.Cost, result.TrackingNumber);
        }
    }
}
```

## 4. En İyi Pratikler

1. **Zincirleme Çağrıları Azaltın**
   - `a.GetB().GetC().DoSomething()` gibi zincirleme çağrılardan kaçının
   - Delegasyon metotları oluşturun
   - Facade pattern kullanın

2. **Arayüzleri Basit Tutun**
   - Sınıfların dış dünyaya açık arayüzlerini minimize edin
   - İç yapıyı gizleyin
   - "Ihtiyacın kadar bilgi" prensibini uygulayın

3. **Fluent Interface'leri Dikkatli Kullanın**
   - Builder pattern gibi fluent interface'ler Demeter Yasası'nın istisnası olabilir
   - Bu tür durumlarda, zincirleme çağrılar aynı nesne üzerinde olmalıdır
   - Farklı nesneler arasında zincirleme çağrılardan kaçının

4. **Dependency Injection Kullanın**
   - Bağımlılıkları constructor üzerinden enjekte edin
   - Servis lokasyonu yerine DI tercih edin
   - Bağımlılıkları açıkça belirtin

## 5. Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Getter Zincirleri**
   ```csharp
   // Kötü - getter zinciri
   public void ProcessOrder(Customer customer)
   {
       var paymentMethod = customer.GetAccount().GetPaymentProfile().GetPreferredMethod();
       paymentMethod.Process();
   }
   
   // İyi - delegasyon metodu
   public void ProcessOrder(Customer customer)
   {
       customer.ProcessPayment();
   }
   ```

2. **Aşırı Delegasyon**
   ```csharp
   // Kötü - aşırı delegasyon
   public class Customer
   {
       private Address _address;
       
       public string GetStreet() { return _address.GetStreet(); }
       public string GetCity() { return _address.GetCity(); }
       public string GetState() { return _address.GetState(); }
       public string GetZipCode() { return _address.GetZipCode(); }
       public string GetCountry() { return _address.GetCountry(); }
   }
   
   // İyi - anlamlı delegasyon
   public class Customer
   {
       private Address _address;
       
       public Address GetAddress() { return _address; }
       public bool IsInState(string state) { return _address.IsInState(state); }
       public bool IsInternational() { return _address.IsInternational(); }
   }
   ```

3. **Demeter Yasası'nı Aşırı Uygulama**
   ```csharp
   // Kötü - aşırı uygulama
   public class ShoppingCart
   {
       private List<Item> _items;
       
       // Her işlem için ayrı metot
       public void AddItem(Item item) { _items.Add(item); }
       public void RemoveItem(Item item) { _items.Remove(item); }
       public int GetItemCount() { return _items.Count; }
       public decimal GetTotalPrice() { /* ... */ }
       public bool ContainsItem(Item item) { return _items.Contains(item); }
       public void ClearItems() { _items.Clear(); }
   }
   
   // İyi - dengeli yaklaşım
   public class ShoppingCart
   {
       private List<Item> _items;
       
       // Temel işlemler için metotlar
       public void AddItem(Item item) { _items.Add(item); }
       public void RemoveItem(Item item) { _items.Remove(item); }
       public IReadOnlyCollection<Item> GetItems() { return _items.AsReadOnly(); }
   }
   ```

4. **Veri Sınıflarında Aşırı Kapsülleme**
   ```csharp
   // Kötü - veri sınıfında aşırı kapsülleme
   public class OrderDto
   {
       private int _id;
       private string _customerName;
       private decimal _total;
       
       public int GetId() { return _id; }
       public string GetCustomerName() { return _customerName; }
       public decimal GetTotal() { return _total; }
   }
   
   // İyi - veri sınıfı için uygun tasarım
   public class OrderDto
   {
       public int Id { get; set; }
       public string CustomerName { get; set; }
       public decimal Total { get; set; }
   }
   ```

Demeter Yasası, yazılım tasarımında bağımlılıkları azaltmanın ve kapsüllemeyi artırmanın etkili bir yoludur. Bu prensibi doğru uygulayarak, daha bakımı kolay ve değişime daha açık sistemler geliştirebilirsiniz. 