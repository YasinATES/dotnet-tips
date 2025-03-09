# Pattern Matching (Desen Eşleştirme)

C# 7.0 ile tanıtılan ve sonraki sürümlerde genişletilen desen eşleştirme (pattern matching), nesneleri belirli desenlere göre test etmenizi ve değerlerini çıkarmanızı sağlayan güçlü bir özelliktir. Bu bölümde, C#'taki çeşitli desen eşleştirme türlerini inceleyeceğiz.

## 1. Type Patterns (Tür Desenleri)

Tür desenleri, bir nesnenin belirli bir türde olup olmadığını kontrol etmek ve aynı zamanda o türe dönüştürmek için kullanılır.

```csharp
// Temel tür deseni
public void ProcessValue(object value)
{
    // Tür deseni ile kontrol ve dönüştürme
    if (value is int number)
    {
        // 'number' değişkeni burada int türünde
        Console.WriteLine($"Sayı: {number}");
    }
    else if (value is string text)
    {
        // 'text' değişkeni burada string türünde
        Console.WriteLine($"Metin: {text}");
    }
    else if (value is DateTime date)
    {
        // 'date' değişkeni burada DateTime türünde
        Console.WriteLine($"Tarih: {date:d}");
    }
    else
    {
        Console.WriteLine($"Bilinmeyen tür: {value?.GetType().Name ?? "null"}");
    }
}

// Kullanım örneği
public void TypePatternExample()
{
    ProcessValue(42);        // Sayı: 42
    ProcessValue("Merhaba"); // Metin: Merhaba
    ProcessValue(DateTime.Now); // Tarih: 01.01.2023 (günün tarihine göre)
    ProcessValue(null);      // Bilinmeyen tür: null
}
```

### Polimorfik Tür Desenleri

Tür desenleri, polimorfik sınıflarla çalışırken özellikle kullanışlıdır.

```csharp
// Şekil hiyerarşisi
public abstract class Shape
{
    public abstract double CalculateArea();
}

public class Circle : Shape
{
    public double Radius { get; }
    
    public Circle(double radius)
    {
        Radius = radius;
    }
    
    public override double CalculateArea() => Math.PI * Radius * Radius;
}

public class Rectangle : Shape
{
    public double Width { get; }
    public double Height { get; }
    
    public Rectangle(double width, double height)
    {
        Width = width;
        Height = height;
    }
    
    public override double CalculateArea() => Width * Height;
}

// Tür deseni ile şekil işleme
public void ProcessShape(Shape shape)
{
    // Tür deseni kullanarak şekil türünü kontrol etme
    if (shape is Circle circle)
    {
        Console.WriteLine($"Daire - Yarıçap: {circle.Radius}, Alan: {circle.CalculateArea():F2}");
    }
    else if (shape is Rectangle rectangle)
    {
        Console.WriteLine($"Dikdörtgen - Genişlik: {rectangle.Width}, Yükseklik: {rectangle.Height}, Alan: {rectangle.CalculateArea():F2}");
    }
    else
    {
        Console.WriteLine($"Bilinmeyen şekil - Alan: {shape.CalculateArea():F2}");
    }
}
```

## 2. Constant Patterns (Sabit Desenleri)

Sabit desenleri, bir değerin belirli bir sabite eşit olup olmadığını kontrol etmek için kullanılır.

```csharp
// Sabit deseni
public string GetDayType(DayOfWeek day)
{
    // Sabit deseni ile gün türünü kontrol etme
    if (day is DayOfWeek.Saturday or DayOfWeek.Sunday)
    {
        return "Hafta sonu";
    }
    else
    {
        return "Hafta içi";
    }
}

// Null kontrolü için sabit deseni
public string FormatName(string name)
{
    // null kontrolü için sabit deseni
    if (name is null)
    {
        return "İsimsiz";
    }
    
    return name.Trim();
}

// Kullanım örneği
public void ConstantPatternExample()
{
    Console.WriteLine(GetDayType(DayOfWeek.Monday));    // Hafta içi
    Console.WriteLine(GetDayType(DayOfWeek.Saturday));  // Hafta sonu
    
    Console.WriteLine(FormatName("John"));   // John
    Console.WriteLine(FormatName(null));     // İsimsiz
}
```

## 3. Property Patterns (Özellik Desenleri)

Özellik desenleri, bir nesnenin özelliklerini kontrol etmek için kullanılır.

```csharp
// Müşteri sınıfı
public class Customer
{
    public string Name { get; set; }
    public string Email { get; set; }
    public CustomerType Type { get; set; }
    public DateTime RegistrationDate { get; set; }
    public Address ShippingAddress { get; set; }
}

public class Address
{
    public string Country { get; set; }
    public string City { get; set; }
    public string Street { get; set; }
}

public enum CustomerType
{
    Regular,
    Premium,
    VIP
}

// Özellik deseni kullanımı
public bool IsEligibleForDiscount(Customer customer)
{
    // Özellik deseni ile müşteri özelliklerini kontrol etme
    return customer is { Type: CustomerType.Premium or CustomerType.VIP, RegistrationDate.Year: < 2023 };
}

// İç içe özellik deseni
public bool IsInTurkey(Customer customer)
{
    // İç içe özellik deseni ile adres kontrolü
    return customer is { ShippingAddress: { Country: "Turkey" } };
}

// Kullanım örneği
public void PropertyPatternExample()
{
    var customer = new Customer
    {
        Name = "Ahmet Yılmaz",
        Email = "ahmet@example.com",
        Type = CustomerType.Premium,
        RegistrationDate = new DateTime(2022, 5, 10),
        ShippingAddress = new Address
        {
            Country = "Turkey",
            City = "Istanbul",
            Street = "Bağdat Caddesi"
        }
    };
    
    Console.WriteLine($"İndirim hakkı: {IsEligibleForDiscount(customer)}"); // True
    Console.WriteLine($"Türkiye'de: {IsInTurkey(customer)}"); // True
}
```

## 4. Tuple Patterns (Demet Desenleri)

Demet desenleri, demetlerin (tuples) içindeki değerleri kontrol etmek için kullanılır.

```csharp
// Demet deseni
public string ClassifyPoint(int x, int y)
{
    // Demet deseni ile koordinat kontrolü
    return (x, y) switch
    {
        (0, 0) => "Orijin",
        (0, _) => "Y ekseni üzerinde",
        (_, 0) => "X ekseni üzerinde",
        (var a, var b) when a == b => "Y = X doğrusu üzerinde",
        (var a, var b) when a == -b => "Y = -X doğrusu üzerinde",
        (_, _) => "Düzlemde bir nokta"
    };
}

// Kullanım örneği
public void TuplePatternExample()
{
    Console.WriteLine(ClassifyPoint(0, 0));   // Orijin
    Console.WriteLine(ClassifyPoint(0, 5));   // Y ekseni üzerinde
    Console.WriteLine(ClassifyPoint(5, 0));   // X ekseni üzerinde
    Console.WriteLine(ClassifyPoint(3, 3));   // Y = X doğrusu üzerinde
    Console.WriteLine(ClassifyPoint(3, -3));  // Y = -X doğrusu üzerinde
    Console.WriteLine(ClassifyPoint(2, 5));   // Düzlemde bir nokta
}
```

## 5. Positional Patterns (Konumsal Desenler)

Konumsal desenler, bir nesnenin yapısını açmak ve içindeki değerleri konumlarına göre kontrol etmek için kullanılır.

```csharp
// Nokta sınıfı (deconstruct metodu ile)
public class Point
{
    public int X { get; }
    public int Y { get; }
    
    public Point(int x, int y) => (X, Y) = (x, y);
    
    // Deconstruct metodu - konumsal desen için gerekli
    public void Deconstruct(out int x, out int y) => (x, y) = (X, Y);
}

// Konumsal desen kullanımı
public string ClassifyPointObject(Point point)
{
    // Konumsal desen ile Point nesnesini açma ve kontrol etme
    return point switch
    {
        (0, 0) => "Orijin",
        (0, _) => "Y ekseni üzerinde",
        (_, 0) => "X ekseni üzerinde",
        (var x, var y) when x == y => "Y = X doğrusu üzerinde",
        (var x, var y) when x == -y => "Y = -X doğrusu üzerinde",
        _ => "Düzlemde bir nokta"
    };
}

// Kullanım örneği
public void PositionalPatternExample()
{
    var points = new[]
    {
        new Point(0, 0),
        new Point(0, 5),
        new Point(5, 0),
        new Point(3, 3),
        new Point(3, -3),
        new Point(2, 5)
    };
    
    foreach (var point in points)
    {
        Console.WriteLine($"({point.X}, {point.Y}): {ClassifyPointObject(point)}");
    }
}
```

### Record Türleri ile Konumsal Desenler

C# 9.0 ile tanıtılan record türleri, konumsal desenlerle doğal olarak çalışır.

```csharp
// Record türü
public record Person(string FirstName, string LastName, int Age);

// Record ile konumsal desen kullanımı
public string GreetPerson(Person person)
{
    return person switch
    {
        ("John", "Doe", _) => "Merhaba, John Doe!",
        (_, _, var age) when age < 18 => "Merhaba, genç!",
        (var firstName, _, >= 18 and < 65) => $"Merhaba, {firstName}!",
        (_, _, >= 65) => "Merhaba, saygıdeğer büyüğümüz!",
        _ => "Merhaba!"
    };
}

// Kullanım örneği
public void RecordPositionalPatternExample()
{
    var people = new[]
    {
        new Person("John", "Doe", 30),
        new Person("Jane", "Smith", 15),
        new Person("Alice", "Johnson", 25),
        new Person("Bob", "Brown", 70)
    };
    
    foreach (var person in people)
    {
        Console.WriteLine(GreetPerson(person));
    }
}
```

## 6. Switch Expressions (Switch İfadeleri)

C# 8.0 ile tanıtılan switch ifadeleri, geleneksel switch deyimlerinin daha özlü ve ifadesel bir alternatifidir.

```csharp
// Geleneksel switch deyimi
public decimal CalculateTraditionalDiscount(CustomerType type, decimal amount)
{
    switch (type)
    {
        case CustomerType.Regular:
            return amount * 0.05m;
        case CustomerType.Premium:
            return amount * 0.10m;
        case CustomerType.VIP:
            return amount * 0.15m;
        default:
            return 0;
    }
}

// Switch ifadesi
public decimal CalculateDiscount(CustomerType type, decimal amount)
{
    return type switch
    {
        CustomerType.Regular => amount * 0.05m,
        CustomerType.Premium => amount * 0.10m,
        CustomerType.VIP => amount * 0.15m,
        _ => 0
    };
}

// Koşullu switch ifadesi
public decimal CalculateOrderDiscount(Customer customer, Order order)
{
    return (customer.Type, order.Amount) switch
    {
        (CustomerType.Regular, < 1000) => order.Amount * 0.05m,
        (CustomerType.Regular, >= 1000) => order.Amount * 0.10m,
        (CustomerType.Premium, _) => order.Amount * 0.15m,
        (CustomerType.VIP, _) => order.Amount * 0.20m,
        _ => 0
    };
}

// Sipariş sınıfı
public class Order
{
    public decimal Amount { get; set; }
    public DateTime OrderDate { get; set; }
    public OrderStatus Status { get; set; }
}

public enum OrderStatus
{
    Pending,
    Processing,
    Shipped,
    Delivered,
    Cancelled
}
```

## 7. Pattern Combinators (Desen Birleştiriciler)

C# 9.0 ile tanıtılan desen birleştiriciler, desenleri mantıksal operatörlerle birleştirmenizi sağlar.

```csharp
// 'and' desen birleştirici
public bool IsValidUsername(string username)
{
    // Kullanıcı adı null olmamalı, 3-20 karakter arasında olmalı ve alfanümerik olmalı
    return username is not null and { Length: >= 3 and <= 20 } and string name && IsAlphanumeric(name);
}

private bool IsAlphanumeric(string text)
{
    return text.All(char.IsLetterOrDigit);
}

// 'or' desen birleştirici
public bool IsWeekend(DateTime date)
{
    return date.DayOfWeek is DayOfWeek.Saturday or DayOfWeek.Sunday;
}

// 'not' desen birleştirici
public bool IsWorkingHour(DateTime time)
{
    // Çalışma saatleri: 9:00 - 18:00
    return time.Hour is >= 9 and < 18 and not 13; // 13:00 öğle molası
}

// Karmaşık desen birleştiriciler
public string ClassifyTemperature(double temperature)
{
    return temperature switch
    {
        < 0 => "Dondurucu",
        >= 0 and < 15 => "Soğuk",
        >= 15 and < 25 => "Ilık",
        >= 25 and < 35 => "Sıcak",
        >= 35 => "Çok sıcak"
    };
}
```

### Gelişmiş Desen Birleştirici Örnekleri

```csharp
// Sipariş durumu kontrolü
public string GetOrderStatusMessage(Order order)
{
    return order.Status switch
    {
        OrderStatus.Pending or OrderStatus.Processing => "Siparişiniz hazırlanıyor",
        OrderStatus.Shipped => "Siparişiniz kargoya verildi",
        OrderStatus.Delivered => "Siparişiniz teslim edildi",
        OrderStatus.Cancelled => "Siparişiniz iptal edildi",
        _ => "Bilinmeyen sipariş durumu"
    };
}

// Tarih ve sipariş durumu kontrolü
public bool IsRecentDelivery(Order order)
{
    return order is { Status: OrderStatus.Delivered, OrderDate: var date } 
           && date >= DateTime.Now.AddDays(-7);
}

// Müşteri ve sipariş kontrolü
public string GetCustomerOrderSummary(Customer customer, Order order)
{
    return (customer, order) switch
    {
        (null, _) => "Müşteri bulunamadı",
        (_, null) => "Sipariş bulunamadı",
        ({ Type: CustomerType.VIP }, { Status: OrderStatus.Delivered }) => "VIP müşteri siparişi teslim edildi",
        ({ Type: CustomerType.VIP }, _) => "VIP müşteri siparişi işleniyor",
        (_, { Status: OrderStatus.Cancelled }) => "Sipariş iptal edildi",
        (_, { Status: OrderStatus.Delivered }) => "Sipariş teslim edildi",
        _ => "Sipariş işleniyor"
    };
}
```

## Gerçek Dünya Uygulamaları

### Ödeme İşleme Sistemi

```csharp
// Ödeme türleri
public abstract class Payment
{
    public decimal Amount { get; set; }
    public DateTime Date { get; set; }
}

public class CreditCardPayment : Payment
{
    public string CardNumber { get; set; }
    public string CardHolderName { get; set; }
    public DateTime ExpiryDate { get; set; }
}

public class BankTransferPayment : Payment
{
    public string AccountNumber { get; set; }
    public string BankName { get; set; }
    public string ReferenceNumber { get; set; }
}

public class CashPayment : Payment
{
    public string ReceivedBy { get; set; }
}

// Desen eşleştirme ile ödeme işleme
public string ProcessPayment(Payment payment)
{
    return payment switch
    {
        CreditCardPayment { Amount: > 10000 } => "Büyük miktarlı kredi kartı ödemesi için ek doğrulama gerekli",
        CreditCardPayment { ExpiryDate: var expiry } when expiry < DateTime.Now => "Kredi kartı süresi dolmuş",
        CreditCardPayment card => $"Kredi kartı ödemesi işlendi: {card.CardNumber.Substring(card.CardNumber.Length - 4)}",
        
        BankTransferPayment { BankName: "XYZ Bank" } bank => $"XYZ Bank transferi işlendi: {bank.ReferenceNumber}",
        BankTransferPayment bank => $"Banka transferi işlendi: {bank.BankName}",
        
        CashPayment cash => $"Nakit ödeme alındı, alan kişi: {cash.ReceivedBy}",
        
        null => "Ödeme bilgisi bulunamadı",
        _ => "Bilinmeyen ödeme türü"
    };
}
```

### Belge İşleme Sistemi

```csharp
// Belge türleri
public abstract class Document
{
    public string Title { get; set; }
    public DateTime CreationDate { get; set; }
    public string Author { get; set; }
}

public class TextDocument : Document
{
    public string Content { get; set; }
    public int WordCount { get; set; }
}

public class SpreadsheetDocument : Document
{
    public int RowCount { get; set; }
    public int ColumnCount { get; set; }
    public bool HasFormulas { get; set; }
}

public class PresentationDocument : Document
{
    public int SlideCount { get; set; }
    public bool HasAnimations { get; set; }
}

// Desen eşleştirme ile belge işleme
public string AnalyzeDocument(Document document)
{
    return document switch
    {
        TextDocument { WordCount: > 1000 } text => 
            $"Uzun metin belgesi: {text.Title}, {text.WordCount} kelime",
        
        TextDocument text => 
            $"Metin belgesi: {text.Title}, {text.WordCount} kelime",
        
        SpreadsheetDocument { RowCount: > 100, HasFormulas: true } spreadsheet => 
            $"Karmaşık hesap tablosu: {spreadsheet.Title}, {spreadsheet.RowCount}x{spreadsheet.ColumnCount}",
        
        SpreadsheetDocument spreadsheet => 
            $"Hesap tablosu: {spreadsheet.Title}, {spreadsheet.RowCount}x{spreadsheet.ColumnCount}",
        
        PresentationDocument { SlideCount: > 20, HasAnimations: true } presentation => 
            $"Animasyonlu uzun sunum: {presentation.Title}, {presentation.SlideCount} slayt",
        
        PresentationDocument presentation => 
            $"Sunum: {presentation.Title}, {presentation.SlideCount} slayt",
        
        { CreationDate: var date } when date.Year < 2010 => 
            $"Eski belge: {document.Title} ({date.Year})",
        
        null => "Belge bulunamadı",
        
        _ => $"Bilinmeyen belge türü: {document.Title}"
    };
}
```

## Özet

Bu bölümde, C#'taki desen eşleştirme özelliklerini inceledik:

1. **Tür Desenleri**: Bir nesnenin belirli bir türde olup olmadığını kontrol etmek ve aynı zamanda o türe dönüştürmek için kullanılır.

2. **Sabit Desenleri**: Bir değerin belirli bir sabite eşit olup olmadığını kontrol etmek için kullanılır.

3. **Özellik Desenleri**: Bir nesnenin özelliklerini kontrol etmek için kullanılır.

4. **Demet Desenleri**: Demetlerin (tuples) içindeki değerleri kontrol etmek için kullanılır.

5. **Konumsal Desenler**: Bir nesnenin yapısını açmak ve içindeki değerleri konumlarına göre kontrol etmek için kullanılır.

6. **Switch İfadeleri**: Geleneksel switch deyimlerinin daha özlü ve ifadesel bir alternatifidir.

7. **Desen Birleştiriciler**: Desenleri mantıksal operatörlerle birleştirmenizi sağlar.

Desen eşleştirme, kodunuzu daha okunabilir, daha özlü ve daha az hataya açık hale getirmenize yardımcı olur. Özellikle karmaşık nesne hiyerarşileri ve veri yapıları ile çalışırken, desen eşleştirme kodunuzu önemli ölçüde basitleştirebilir. 