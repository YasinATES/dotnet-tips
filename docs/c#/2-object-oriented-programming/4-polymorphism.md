# Polimorfizm (Polymorphism)

Polimorfizm, nesne yönelimli programlamanın temel prensiplerinden biridir ve "çok biçimlilik" anlamına gelir. Bu kavram, farklı sınıfların aynı arayüzü paylaşabilmesini ve her sınıfın bu arayüzü kendi şekilde uygulayabilmesini sağlar. Bu bölümde, C#'ta polimorfizm kavramını ve ilgili konuları detaylı olarak inceleyeceğiz.

## 1. Metot Overriding

Metot overriding, bir alt sınıfın üst sınıftan miras aldığı bir metodu yeniden tanımlamasıdır. Bu, polimorfizmin en temel formudur.

### Temel Metot Overriding

```csharp
// Ödeme işlemi temel sınıfı
public class PaymentProcessor
{
    protected decimal _commissionRate;
    
    public PaymentProcessor(decimal commissionRate)
    {
        _commissionRate = commissionRate;
    }
    
    public virtual decimal CalculateFee(decimal amount)
    {
        // Temel komisyon hesaplama
        return amount * _commissionRate;
    }
    
    public virtual bool ProcessPayment(decimal amount)
    {
        decimal fee = CalculateFee(amount);
        decimal totalAmount = amount + fee;
        
        Console.WriteLine($"İşlem tutarı: {amount:C}");
        Console.WriteLine($"Komisyon: {fee:C}");
        Console.WriteLine($"Toplam tutar: {totalAmount:C}");
        
        // Ödeme işlemi gerçekleştirme
        return true;
    }
}

// Kredi kartı ödeme işlemcisi
public class CreditCardProcessor : PaymentProcessor
{
    private readonly decimal _additionalFeeRate;
    
    public CreditCardProcessor(decimal commissionRate, decimal additionalFeeRate) 
        : base(commissionRate)
    {
        _additionalFeeRate = additionalFeeRate;
    }
    
    // Üst sınıfın metodu override edildi
    public override decimal CalculateFee(decimal amount)
    {
        // Temel komisyon + ek kredi kartı komisyonu
        decimal baseFee = base.CalculateFee(amount);
        decimal additionalFee = amount * _additionalFeeRate;
        
        return baseFee + additionalFee;
    }
    
    public override bool ProcessPayment(decimal amount)
    {
        Console.WriteLine("Kredi kartı ile ödeme işlemi başlatılıyor...");
        
        // Kredi kartı doğrulama işlemi
        if (!ValidateCard())
        {
            Console.WriteLine("Kredi kartı doğrulanamadı!");
            return false;
        }
        
        // Üst sınıfın metodu çağrılıyor
        return base.ProcessPayment(amount);
    }
    
    private bool ValidateCard()
    {
        // Kredi kartı doğrulama işlemi
        Console.WriteLine("Kredi kartı doğrulanıyor...");
        return true; // Gerçek uygulamada kart doğrulama mantığı olacaktır
    }
}

// Banka havalesi ödeme işlemcisi
public class BankTransferProcessor : PaymentProcessor
{
    private readonly decimal _discountRate;
    
    public BankTransferProcessor(decimal commissionRate, decimal discountRate) 
        : base(commissionRate)
    {
        _discountRate = discountRate;
    }
    
    // Üst sınıfın metodu override edildi
    public override decimal CalculateFee(decimal amount)
    {
        // Temel komisyon - indirim
        decimal baseFee = base.CalculateFee(amount);
        decimal discount = baseFee * _discountRate;
        
        return Math.Max(0, baseFee - discount);
    }
    
    public override bool ProcessPayment(decimal amount)
    {
        Console.WriteLine("Banka havalesi ile ödeme işlemi başlatılıyor...");
        Console.WriteLine("IBAN doğrulanıyor...");
        
        // Üst sınıfın metodu çağrılıyor
        return base.ProcessPayment(amount);
    }
}
```

### Polimorfik Davranış

```csharp
// Ödeme işlemcilerini kullanma
public class PaymentService
{
    public void ProcessPayment(PaymentProcessor processor, decimal amount)
    {
        Console.WriteLine("Ödeme işlemi başlatılıyor...");
        
        bool result = processor.ProcessPayment(amount);
        
        if (result)
        {
            Console.WriteLine("Ödeme işlemi başarılı!");
        }
        else
        {
            Console.WriteLine("Ödeme işlemi başarısız!");
        }
    }
}

// Kullanım
PaymentService paymentService = new PaymentService();

// Farklı ödeme işlemcileri oluşturma
PaymentProcessor creditCardProcessor = new CreditCardProcessor(0.02m, 0.01m);
PaymentProcessor bankTransferProcessor = new BankTransferProcessor(0.02m, 0.5m);

// Aynı metot, farklı davranışlar (polimorfizm)
paymentService.ProcessPayment(creditCardProcessor, 100);
paymentService.ProcessPayment(bankTransferProcessor, 100);
```

## 2. Operatör Overloading

C#, operatörlerin davranışını özelleştirmenize olanak tanır. Bu, sınıflarınızın yerleşik türler gibi davranmasını sağlar.

### Temel Operatör Overloading

```csharp
// Para birimi sınıfı
public class Money
{
    public decimal Amount { get; }
    public string Currency { get; }
    
    public Money(decimal amount, string currency)
    {
        Amount = amount;
        Currency = currency;
    }
    
    // + operatörü overload
    public static Money operator +(Money left, Money right)
    {
        if (left.Currency != right.Currency)
        {
            throw new InvalidOperationException("Farklı para birimleri toplanamaz!");
        }
        
        return new Money(left.Amount + right.Amount, left.Currency);
    }
    
    // - operatörü overload
    public static Money operator -(Money left, Money right)
    {
        if (left.Currency != right.Currency)
        {
            throw new InvalidOperationException("Farklı para birimleri çıkarılamaz!");
        }
        
        return new Money(left.Amount - right.Amount, left.Currency);
    }
    
    // * operatörü overload (para birimini bir sayı ile çarpma)
    public static Money operator *(Money money, decimal multiplier)
    {
        return new Money(money.Amount * multiplier, money.Currency);
    }
    
    // * operatörü overload (sayıyı para birimi ile çarpma - sıra değişimi)
    public static Money operator *(decimal multiplier, Money money)
    {
        return money * multiplier; // Yukarıdaki overload'ı çağırır
    }
    
    // == operatörü overload
    public static bool operator ==(Money left, Money right)
    {
        if (ReferenceEquals(left, null) && ReferenceEquals(right, null))
            return true;
            
        if (ReferenceEquals(left, null) || ReferenceEquals(right, null))
            return false;
            
        return left.Currency == right.Currency && left.Amount == right.Amount;
    }
    
    // != operatörü overload
    public static bool operator !=(Money left, Money right)
    {
        return !(left == right);
    }
    
    // > operatörü overload
    public static bool operator >(Money left, Money right)
    {
        if (left.Currency != right.Currency)
        {
            throw new InvalidOperationException("Farklı para birimleri karşılaştırılamaz!");
        }
        
        return left.Amount > right.Amount;
    }
    
    // < operatörü overload
    public static bool operator <(Money left, Money right)
    {
        if (left.Currency != right.Currency)
        {
            throw new InvalidOperationException("Farklı para birimleri karşılaştırılamaz!");
        }
        
        return left.Amount < right.Amount;
    }
    
    // Equals ve GetHashCode metotlarını override etmek iyi bir pratiktir
    public override bool Equals(object obj)
    {
        if (obj is Money other)
        {
            return this == other;
        }
        
        return false;
    }
    
    public override int GetHashCode()
    {
        return HashCode.Combine(Amount, Currency);
    }
    
    public override string ToString()
    {
        return $"{Amount:N2} {Currency}";
    }
}
```

### Operatör Overloading Kullanımı

```csharp
// Money sınıfını kullanma
Money wallet1 = new Money(100, "TRY");
Money wallet2 = new Money(50, "TRY");
Money wallet3 = new Money(200, "USD");

// Operatör kullanımı
Money sum = wallet1 + wallet2; // 150 TRY
Money difference = wallet1 - wallet2; // 50 TRY
Money doubled = wallet1 * 2; // 200 TRY
Money tripled = 3 * wallet1; // 300 TRY

// Karşılaştırma operatörleri
bool isEqual = wallet1 == wallet2; // false
bool isGreater = wallet1 > wallet2; // true

try
{
    // Farklı para birimleri
    Money invalid = wallet1 + wallet3; // Exception fırlatır
}
catch (InvalidOperationException ex)
{
    Console.WriteLine(ex.Message); // "Farklı para birimleri toplanamaz!"
}

Console.WriteLine($"Cüzdan 1: {wallet1}"); // "Cüzdan 1: 100.00 TRY"
Console.WriteLine($"Cüzdan 1 + Cüzdan 2: {sum}"); // "Cüzdan 1 + Cüzdan 2: 150.00 TRY"
Console.WriteLine($"Cüzdan 1 * 2: {doubled}"); // "Cüzdan 1 * 2: 200.00 TRY"
```

## 3. is ve as Operatörleri

`is` ve `as` operatörleri, tip dönüşümlerini güvenli bir şekilde gerçekleştirmenize yardımcı olur ve polimorfik davranışı destekler.

### is Operatörü

```csharp
// Ödeme işlemcisi hiyerarşisi
public void ProcessPayment(PaymentProcessor processor, decimal amount)
{
    // is operatörü ile tip kontrolü
    if (processor is CreditCardProcessor)
    {
        Console.WriteLine("Kredi kartı işlemi tespit edildi.");
    }
    else if (processor is BankTransferProcessor)
    {
        Console.WriteLine("Banka havalesi işlemi tespit edildi.");
    }
    
    processor.ProcessPayment(amount);
}
```

### as Operatörü

```csharp
// as operatörü ile güvenli tip dönüşümü
public void ApplyDiscount(PaymentProcessor processor, decimal discountRate)
{
    // as operatörü ile dönüşüm (başarısız olursa null döner)
    CreditCardProcessor creditCardProcessor = processor as CreditCardProcessor;
    
    if (creditCardProcessor != null)
    {
        // CreditCardProcessor'a özgü işlemler
        Console.WriteLine($"Kredi kartı işlemine %{discountRate * 100} indirim uygulanıyor.");
        // ...
    }
    else
    {
        Console.WriteLine("Bu işlemci türüne indirim uygulanamaz.");
    }
}
```

## 4. Pattern Matching (C# 7.0+)

Pattern matching, C# 7.0 ile tanıtılan ve sonraki sürümlerde genişletilen güçlü bir özelliktir. Tip kontrolü ve dönüşümünü daha zarif bir şekilde yapmanıza olanak tanır.

### Type Pattern

```csharp
// Type pattern ile tip kontrolü ve dönüşümü
public decimal CalculateDiscountedFee(PaymentProcessor processor, decimal amount)
{
    decimal fee = processor.CalculateFee(amount);
    
    // Type pattern
    switch (processor)
    {
        case CreditCardProcessor creditCard:
            // CreditCardProcessor'a özgü işlemler
            return fee * 0.9m; // %10 indirim
            
        case BankTransferProcessor bankTransfer:
            // BankTransferProcessor'a özgü işlemler
            return fee * 0.8m; // %20 indirim
            
        case PayPalProcessor paypal when paypal.IsPreferred:
            // PayPalProcessor ve IsPreferred true ise
            return fee * 0.7m; // %30 indirim
            
        default:
            return fee; // İndirim yok
    }
}
```

### Property Pattern (C# 8.0+)

```csharp
// Sipariş sınıfı
public class Order
{
    public decimal TotalAmount { get; set; }
    public string CustomerType { get; set; }
    public bool IsRecurring { get; set; }
    public DateTime OrderDate { get; set; }
    public PaymentMethod PaymentMethod { get; set; }
}

public enum PaymentMethod
{
    CreditCard,
    BankTransfer,
    PayPal,
    Cash
}

// Property pattern kullanımı
public decimal CalculateDiscount(Order order)
{
    // Property pattern
    return order switch
    {
        { TotalAmount: > 1000, CustomerType: "Premium" } => order.TotalAmount * 0.2m, // %20 indirim
        { TotalAmount: > 500, IsRecurring: true } => order.TotalAmount * 0.15m, // %15 indirim
        { PaymentMethod: PaymentMethod.BankTransfer } => order.TotalAmount * 0.05m, // %5 indirim
        { OrderDate.Month: 12 } => order.TotalAmount * 0.1m, // Aralık ayında %10 indirim
        _ => 0 // İndirim yok
    };
}
```

### Tuple Pattern (C# 8.0+)

```csharp
// Tuple pattern kullanımı
public string GetShippingCost(string country, decimal orderTotal)
{
    return (country, orderTotal) switch
    {
        ("USA", > 100) => "Ücretsiz Kargo",
        ("USA", _) => "$5.99",
        ("Canada", > 100) => "Ücretsiz Kargo",
        ("Canada", _) => "$10.99",
        ("Turkey", > 200) => "Ücretsiz Kargo",
        ("Turkey", > 100) => "₺15.00",
        ("Turkey", _) => "₺25.00",
        (_, > 300) => "Ücretsiz Kargo", // Diğer ülkeler için 300 üzeri ücretsiz
        _ => "Standart Kargo Ücreti Uygulanır"
    };
}
```

### Positional Pattern (C# 8.0+)

```csharp
// Nokta sınıfı
public class Point
{
    public int X { get; }
    public int Y { get; }
    
    public Point(int x, int y) => (X, Y) = (x, y);
    
    // Deconstruct metodu
    public void Deconstruct(out int x, out int y) => (x, y) = (X, Y);
}

// Positional pattern kullanımı
public string ClassifyPoint(Point point)
{
    return point switch
    {
        (0, 0) => "Orijin",
        (0, _) => "Y Ekseni",
        (_, 0) => "X Ekseni",
        (var x, var y) when x == y => "X = Y Doğrusu",
        (var x, var y) when x == -y => "X = -Y Doğrusu",
        (> 0, > 0) => "1. Bölge",
        (< 0, > 0) => "2. Bölge",
        (< 0, < 0) => "3. Bölge",
        (> 0, < 0) => "4. Bölge",
        _ => "Belirsiz"
    };
}
```

## 5. Dynamic Binding ve dynamic Keyword

C#, `dynamic` anahtar kelimesi ile statik tip kontrolünü çalışma zamanına erteleyebilir. Bu, özellikle dinamik diller veya COM nesneleriyle çalışırken faydalıdır.

### dynamic Keyword Kullanımı

```csharp
// dynamic kullanımı
public void ProcessPaymentDynamically(dynamic processor, decimal amount)
{
    try
    {
        // Çalışma zamanında metot çağrısı çözümlenir
        processor.ProcessPayment(amount);
    }
    catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException)
    {
        Console.WriteLine("Bu nesne ProcessPayment metodunu desteklemiyor!");
    }
}

// Farklı nesnelerle kullanım
public void DynamicExample()
{
    dynamic creditCard = new CreditCardProcessor(0.02m, 0.01m);
    dynamic bankTransfer = new BankTransferProcessor(0.02m, 0.5m);
    dynamic invalidObject = new { Name = "Test" }; // ProcessPayment metodu yok
    
    ProcessPaymentDynamically(creditCard, 100); // Çalışır
    ProcessPaymentDynamically(bankTransfer, 100); // Çalışır
    ProcessPaymentDynamically(invalidObject, 100); // RuntimeBinderException fırlatır
}
```

### Reflection ile Karşılaştırma

```csharp
// Reflection kullanımı
public void ProcessPaymentWithReflection(object processor, decimal amount)
{
    Type type = processor.GetType();
    MethodInfo method = type.GetMethod("ProcessPayment", new[] { typeof(decimal) });
    
    if (method != null)
    {
        method.Invoke(processor, new object[] { amount });
    }
    else
    {
        Console.WriteLine("Bu nesne ProcessPayment metodunu desteklemiyor!");
    }
}

// dynamic ile aynı işlem
public void ProcessPaymentWithDynamic(dynamic processor, decimal amount)
{
    try
    {
        processor.ProcessPayment(amount);
    }
    catch (Microsoft.CSharp.RuntimeBinder.RuntimeBinderException)
    {
        Console.WriteLine("Bu nesne ProcessPayment metodunu desteklemiyor!");
    }
}
```

### ExpandoObject ile Dinamik Nesneler

```csharp
// ExpandoObject ile dinamik nesne oluşturma
public void ExpandoObjectExample()
{
    dynamic customer = new System.Dynamic.ExpandoObject();
    
    // Dinamik olarak özellikler ekleme
    customer.Name = "Ahmet Yılmaz";
    customer.Email = "ahmet@example.com";
    customer.Balance = 1000m;
    
    // Dinamik olarak metot ekleme
    customer.UpdateBalance = (Action<decimal>)(amount => {
        customer.Balance += amount;
        Console.WriteLine($"Yeni bakiye: {customer.Balance:C}");
    });
    
    // Kullanım
    Console.WriteLine($"Müşteri: {customer.Name}, Bakiye: {customer.Balance:C}");
    customer.UpdateBalance(500);
}
```

## En İyi Pratikler

1. **Metot Overriding**
   - `virtual`, `override` ve `new` anahtar kelimelerini doğru kullanın.
   - Base sınıf metotlarını override ederken, gerektiğinde `base` anahtar kelimesi ile üst sınıf implementasyonunu çağırın.
   - Çok derin kalıtım hiyerarşilerinden kaçının, bu tür hiyerarşilerde override edilen metotların davranışını takip etmek zorlaşır.

2. **Operatör Overloading**
   - Operatörleri, sınıfınızın doğal semantiğine uygun şekilde overload edin.
   - Operatör overloading'i aşırı kullanmaktan kaçının, kodun okunabilirliğini azaltabilir.
   - İlgili operatörleri birlikte overload edin (örn. `==` ve `!=`, `<` ve `>`).
   - Operatör overloading kullanıyorsanız, `Equals` ve `GetHashCode` metotlarını da override edin.

3. **is ve as Operatörleri**
   - Tip dönüşümleri için `as` operatörünü tercih edin, başarısız olduğunda exception yerine `null` döndürür.
   - Sadece tip kontrolü yapacaksanız `is` operatörünü kullanın.
   - C# 7.0+ ile gelen pattern matching sözdizimini kullanmayı düşünün.

4. **Pattern Matching**
   - Karmaşık tip kontrolleri ve dönüşümleri için pattern matching kullanın.
   - Switch expression'ları, geleneksel switch statement'lara tercih edin.
   - Property pattern'leri, nesne özelliklerine göre karmaşık koşullar oluşturmak için kullanın.

5. **dynamic Keyword**
   - `dynamic` anahtar kelimesini sadece gerektiğinde kullanın (örn. COM nesneleri, dinamik JSON işleme).
   - `dynamic` kullanırken, çalışma zamanı hatalarına karşı savunmacı programlama yapın.
   - Performans kritik uygulamalarda `dynamic` kullanımından kaçının, statik tiplere göre daha yavaştır.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Gereksiz Tip Dönüşümleri**
   ```csharp
   // Kötü - gereksiz tip dönüşümleri
   public void ProcessPayment(PaymentProcessor processor, decimal amount)
   {
       CreditCardProcessor creditCard = (CreditCardProcessor)processor; // Tehlikeli
       creditCard.ProcessPayment(amount);
   }
   
   // İyi - polimorfizm kullanımı
   public void ProcessPayment(PaymentProcessor processor, decimal amount)
   {
       processor.ProcessPayment(amount); // Polimorfik çağrı
   }
   ```

2. **Operatör Overloading'in Aşırı Kullanımı**
   ```csharp
   // Kötü - beklenmeyen operatör davranışı
   public class User
   {
       public string Name { get; set; }
       
       // Kafa karıştırıcı operatör overloading
       public static User operator +(User user1, User user2)
       {
           return new User { Name = user1.Name + user2.Name };
       }
   }
   
   // İyi - anlamlı metot isimleri
   public class User
   {
       public string Name { get; set; }
       
       public User CombineWith(User other)
       {
           return new User { Name = this.Name + other.Name };
       }
   }
   ```

3. **Pattern Matching'in Karmaşık Kullanımı**
   ```csharp
   // Kötü - aşırı karmaşık pattern matching
   public decimal CalculateDiscount(Order order)
   {
       return order switch
       {
           { TotalAmount: > 1000, CustomerType: "Premium", IsRecurring: true, PaymentMethod: PaymentMethod.CreditCard } => order.TotalAmount * 0.25m,
           { TotalAmount: > 1000, CustomerType: "Premium", IsRecurring: true } => order.TotalAmount * 0.22m,
           { TotalAmount: > 1000, CustomerType: "Premium" } => order.TotalAmount * 0.2m,
           // ... onlarca benzer durum
           _ => 0
       };
   }
   
   // İyi - daha modüler yaklaşım
   public decimal CalculateDiscount(Order order)
   {
       decimal baseDiscount = GetBaseDiscount(order);
       decimal loyaltyDiscount = GetLoyaltyDiscount(order);
       decimal paymentMethodDiscount = GetPaymentMethodDiscount(order);
       
       return baseDiscount + loyaltyDiscount + paymentMethodDiscount;
   }
   
   private decimal GetBaseDiscount(Order order) => order.TotalAmount switch
   {
       > 1000 => order.TotalAmount * 0.1m,
       > 500 => order.TotalAmount * 0.05m,
       _ => 0
   };
   
   // Diğer indirim metotları...
   ```

4. **dynamic Anahtar Kelimesinin Gereksiz Kullanımı**
   ```csharp
   // Kötü - gereksiz dynamic kullanımı
   public void ProcessCustomer(dynamic customer)
   {
       Console.WriteLine($"Müşteri: {customer.Name}");
       customer.UpdateBalance(100);
   }
   
   // İyi - interface kullanımı
   public interface ICustomer
   {
       string Name { get; }
       void UpdateBalance(decimal amount);
   }
   
   public void ProcessCustomer(ICustomer customer)
   {
       Console.WriteLine($"Müşteri: {customer.Name}");
       customer.UpdateBalance(100);
   }
   ```

5. **Tip Kontrolü Yerine Polimorfizm Kullanmamak**
   ```csharp
   // Kötü - tip kontrolüne dayalı davranış
   public void ProcessShape(object shape)
   {
       if (shape is Circle)
       {
           Circle circle = (Circle)shape;
           Console.WriteLine($"Daire alanı: {Math.PI * circle.Radius * circle.Radius}");
       }
       else if (shape is Rectangle)
       {
           Rectangle rectangle = (Rectangle)shape;
           Console.WriteLine($"Dikdörtgen alanı: {rectangle.Width * rectangle.Height}");
       }
   }
   
   // İyi - polimorfizm kullanımı
   public interface IShape
   {
       double CalculateArea();
   }
   
   public class Circle : IShape
   {
       public double Radius { get; set; }
       
       public double CalculateArea()
       {
           return Math.PI * Radius * Radius;
       }
   }
   
   public class Rectangle : IShape
   {
       public double Width { get; set; }
       public double Height { get; set; }
       
       public double CalculateArea()
       {
           return Width * Height;
       }
   }
   
   public void ProcessShape(IShape shape)
   {
       Console.WriteLine($"Şekil alanı: {shape.CalculateArea()}");
   }
   ```

Polimorfizm, C# programlamanın en güçlü özelliklerinden biridir. Doğru kullanıldığında, kodunuzu daha esnek, genişletilebilir ve bakımı kolay hale getirir. Farklı polimorfizm tekniklerini anlamak ve uygun yerlerde kullanmak, daha iyi nesne yönelimli tasarımlar oluşturmanıza yardımcı olacaktır. 