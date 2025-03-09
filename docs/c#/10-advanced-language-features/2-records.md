# Records (Kayıtlar)

C# 9.0 ile tanıtılan Records, değer semantiğine sahip referans türleridir. Özellikle değişmez (immutable) veri modelleri oluşturmak için tasarlanmışlardır. Bu bölümde, C#'taki record türlerini ve özelliklerini inceleyeceğiz.

## 1. Record Declaration (Kayıt Tanımlama)

Records, sınıflara benzer şekilde tanımlanır ancak `class` yerine `record` anahtar kelimesi kullanılır.

```csharp
// Temel record tanımı
public record Person
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
    public DateTime DateOfBirth { get; init; }
}

// Pozisyonel sözdizimi ile record tanımı
public record Employee(string FirstName, string LastName, string Department, decimal Salary);

// Kullanım örneği
public void RecordDeclarationExample()
{
    // Özellik atama sözdizimi ile oluşturma
    var person = new Person
    {
        FirstName = "Ahmet",
        LastName = "Yılmaz",
        DateOfBirth = new DateTime(1985, 5, 15)
    };
    
    // Pozisyonel sözdizimi ile oluşturma
    var employee = new Employee("Mehmet", "Kaya", "IT", 10000m);
    
    Console.WriteLine($"Kişi: {person.FirstName} {person.LastName}");
    Console.WriteLine($"Çalışan: {employee.FirstName} {employee.LastName}, Departman: {employee.Department}");
}
```

### Record Türleri

C# 10.0 ile birlikte, record sınıfları (`record class`) ve record yapıları (`record struct`) arasında ayrım yapabilirsiniz.

```csharp
// Record sınıfı (referans türü)
public record class Customer(int Id, string Name);

// Record yapısı (değer türü)
public record struct Product(int Id, string Name, decimal Price);

// Kullanım örneği
public void RecordTypesExample()
{
    var customer = new Customer(1, "ABC Ltd.");
    var product = new Product(101, "Laptop", 5000m);
    
    Console.WriteLine($"Müşteri: {customer}");
    Console.WriteLine($"Ürün: {product}");
}
```

## 2. Immutability (Değişmezlik)

Records, varsayılan olarak değişmez (immutable) olacak şekilde tasarlanmıştır. Özellikleri `init` erişimcisi ile tanımlanır.

```csharp
// Değişmez record
public record ImmutablePerson
{
    // init erişimcisi ile özellikler sadece nesne oluşturulurken ayarlanabilir
    public string FirstName { get; init; }
    public string LastName { get; init; }
    public DateTime DateOfBirth { get; init; }
    
    // Hesaplanan özellik
    public int Age => DateTime.Now.Year - DateOfBirth.Year;
}

// Kullanım örneği
public void ImmutabilityExample()
{
    var person = new ImmutablePerson
    {
        FirstName = "Ayşe",
        LastName = "Demir",
        DateOfBirth = new DateTime(1990, 3, 20)
    };
    
    // Aşağıdaki satır derleme hatası verir:
    // person.FirstName = "Fatma"; // Error CS8852: Init-only property can only be set in an object initializer
    
    Console.WriteLine($"Kişi: {person.FirstName} {person.LastName}, Yaş: {person.Age}");
}
```

### Değiştirilebilir Özellikler

Gerekirse, record içinde değiştirilebilir özellikler de tanımlayabilirsiniz, ancak bu değişmezlik prensibine aykırıdır.

```csharp
// Değiştirilebilir özelliğe sahip record
public record MutableRecord
{
    public string Id { get; init; }
    public string Name { get; init; }
    public int Counter { get; set; } // Değiştirilebilir özellik
}

// Kullanım örneği
public void MutablePropertyExample()
{
    var record = new MutableRecord
    {
        Id = "R123",
        Name = "Test Record",
        Counter = 0
    };
    
    // Değiştirilebilir özellik güncellenebilir
    record.Counter++;
    
    Console.WriteLine($"Record: {record.Name}, Sayaç: {record.Counter}");
}
```

## 3. With-expressions (With İfadeleri)

Records, mevcut bir kaydın değiştirilmiş bir kopyasını oluşturmak için "with" ifadelerini destekler.

```csharp
// With ifadesi kullanımı
public void WithExpressionExample()
{
    var originalEmployee = new Employee("Ali", "Veli", "Finans", 8000m);
    
    // With ifadesi ile yeni bir kayıt oluşturma (sadece Salary değişiyor)
    var promotedEmployee = originalEmployee with { Salary = 9000m };
    
    // With ifadesi ile birden fazla özelliği değiştirme
    var transferredEmployee = originalEmployee with 
    { 
        Department = "Pazarlama", 
        Salary = 8500m 
    };
    
    Console.WriteLine($"Orijinal: {originalEmployee}");
    Console.WriteLine($"Terfi alan: {promotedEmployee}");
    Console.WriteLine($"Transfer olan: {transferredEmployee}");
}
```

### Karmaşık Nesnelerle With İfadeleri

İç içe record'lar ile with ifadelerini kullanırken dikkatli olmalısınız.

```csharp
// İç içe record'lar
public record Address(string Street, string City, string Country);
public record PersonWithAddress(string Name, Address HomeAddress);

// İç içe record'larla with ifadesi kullanımı
public void NestedWithExpressionExample()
{
    var address = new Address("Bağdat Caddesi", "İstanbul", "Türkiye");
    var person = new PersonWithAddress("Zeynep", address);
    
    // Sadece şehri değiştirme
    var personNewCity = person with 
    { 
        HomeAddress = person.HomeAddress with { City = "Ankara" } 
    };
    
    Console.WriteLine($"Orijinal: {person}");
    Console.WriteLine($"Yeni şehir: {personNewCity}");
}
```

## 4. Inheritance (Kalıtım)

Records, diğer record'lardan türetilebilir, ancak sınıflardan türetilemez veya sınıflar record'lardan türetilemez.

```csharp
// Temel record
public record Person2(string FirstName, string LastName);

// Türetilmiş record
public record Student(string FirstName, string LastName, string StudentId, string Major) 
    : Person2(FirstName, LastName);

// Türetilmiş record - ek özelliklerle
public record Teacher(string FirstName, string LastName, string EmployeeId, string[] Subjects) 
    : Person2(FirstName, LastName);

// Kullanım örneği
public void InheritanceExample()
{
    var student = new Student("Emre", "Şahin", "S12345", "Bilgisayar Mühendisliği");
    var teacher = new Teacher("Hakan", "Öztürk", "T98765", new[] { "Matematik", "Fizik" });
    
    Console.WriteLine($"Öğrenci: {student}");
    Console.WriteLine($"Öğretmen: {teacher}");
    
    // Polimorfizm
    Person2 person = student;
    Console.WriteLine($"Kişi olarak öğrenci: {person}");
}
```

### Sealed (Mühürlü) Records

Varsayılan olarak, record'lar mühürlü (sealed) değildir. Ancak, `sealed` anahtar kelimesi ile mühürlenebilirler.

```csharp
// Mühürlü record
public sealed record SealedRecord(int Id, string Name);

// Aşağıdaki satır derleme hatası verir:
// public record DerivedFromSealed(int Id, string Name, string Description) : SealedRecord(Id, Name);
```

## 5. Equality (Eşitlik)

Records, değer semantiğine sahiptir, yani içerikleri aynı olan iki record eşit kabul edilir.

```csharp
// Eşitlik karşılaştırması
public void EqualityExample()
{
    var person1 = new Person { FirstName = "Ahmet", LastName = "Yılmaz", DateOfBirth = new DateTime(1985, 5, 15) };
    var person2 = new Person { FirstName = "Ahmet", LastName = "Yılmaz", DateOfBirth = new DateTime(1985, 5, 15) };
    var person3 = new Person { FirstName = "Mehmet", LastName = "Yılmaz", DateOfBirth = new DateTime(1985, 5, 15) };
    
    // İçerik eşitliği kontrolü
    Console.WriteLine($"person1 == person2: {person1 == person2}"); // True
    Console.WriteLine($"person1 == person3: {person1 == person3}"); // False
    
    // Equals metodu
    Console.WriteLine($"person1.Equals(person2): {person1.Equals(person2)}"); // True
    
    // GetHashCode
    Console.WriteLine($"person1 hash: {person1.GetHashCode()}");
    Console.WriteLine($"person2 hash: {person2.GetHashCode()}");
    Console.WriteLine($"person3 hash: {person3.GetHashCode()}");
}
```

### Özel Eşitlik Karşılaştırması

Eşitlik karşılaştırmasını özelleştirmek için, `Equals` metodunu ve `==` operatörünü geçersiz kılabilirsiniz.

```csharp
// Özel eşitlik karşılaştırması
public record CustomEqualityPerson
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
    public DateTime DateOfBirth { get; init; }
    
    // Sadece ad ve soyadı karşılaştırma
    public virtual bool Equals(CustomEqualityPerson other)
    {
        return other != null && 
               FirstName == other.FirstName && 
               LastName == other.LastName;
    }
    
    public override int GetHashCode()
    {
        return HashCode.Combine(FirstName, LastName);
    }
}

// Kullanım örneği
public void CustomEqualityExample()
{
    var person1 = new CustomEqualityPerson 
    { 
        FirstName = "Ahmet", 
        LastName = "Yılmaz", 
        DateOfBirth = new DateTime(1985, 5, 15) 
    };
    
    var person2 = new CustomEqualityPerson 
    { 
        FirstName = "Ahmet", 
        LastName = "Yılmaz", 
        DateOfBirth = new DateTime(1990, 10, 20) // Farklı doğum tarihi
    };
    
    // Özel eşitlik karşılaştırması - sadece ad ve soyad karşılaştırılır
    Console.WriteLine($"person1 == person2: {person1 == person2}"); // True
}
```

## 6. Deconstruction (Parçalara Ayırma)

Pozisyonel sözdizimi ile tanımlanan record'lar, otomatik olarak bir Deconstruct metodu içerir.

```csharp
// Deconstruction kullanımı
public void DeconstructionExample()
{
    var employee = new Employee("Mehmet", "Kaya", "IT", 10000m);
    
    // Deconstruct metodu ile parçalara ayırma
    var (firstName, lastName, department, salary) = employee;
    
    Console.WriteLine($"Ad: {firstName}");
    Console.WriteLine($"Soyad: {lastName}");
    Console.WriteLine($"Departman: {department}");
    Console.WriteLine($"Maaş: {salary:C}");
}
```

### Özel Deconstruct Metodu

Özellik atama sözdizimi ile tanımlanan record'lar için özel bir Deconstruct metodu tanımlayabilirsiniz.

```csharp
// Özel Deconstruct metodu
public record PersonWithDeconstruct
{
    public string FirstName { get; init; }
    public string LastName { get; init; }
    public DateTime DateOfBirth { get; init; }
    
    // Özel Deconstruct metodu
    public void Deconstruct(out string firstName, out string lastName, out int age)
    {
        firstName = FirstName;
        lastName = LastName;
        age = DateTime.Now.Year - DateOfBirth.Year;
    }
}

// Kullanım örneği
public void CustomDeconstructExample()
{
    var person = new PersonWithDeconstruct
    {
        FirstName = "Ayşe",
        LastName = "Demir",
        DateOfBirth = new DateTime(1990, 3, 20)
    };
    
    // Özel Deconstruct metodu ile parçalara ayırma
    var (firstName, lastName, age) = person;
    
    Console.WriteLine($"Ad: {firstName}");
    Console.WriteLine($"Soyad: {lastName}");
    Console.WriteLine($"Yaş: {age}");
}
```

## 7. JSON Serialization (JSON Serileştirme)

Records, JSON serileştirme ve ters serileştirme için çok uygundur.

```csharp
// JSON serileştirme
public void JsonSerializationExample()
{
    var person = new Person
    {
        FirstName = "Ahmet",
        LastName = "Yılmaz",
        DateOfBirth = new DateTime(1985, 5, 15)
    };
    
    // JSON'a serileştirme
    string json = JsonSerializer.Serialize(person, new JsonSerializerOptions
    {
        WriteIndented = true
    });
    
    Console.WriteLine("JSON:");
    Console.WriteLine(json);
    
    // JSON'dan ters serileştirme
    var deserializedPerson = JsonSerializer.Deserialize<Person>(json);
    
    Console.WriteLine($"Ters serileştirilen: {deserializedPerson}");
}
```

### Özel JSON Serileştirme

JSON serileştirme davranışını özelleştirmek için, `System.Text.Json.Serialization` özniteliklerini kullanabilirsiniz.

```csharp
// Özel JSON serileştirme
public record PersonWithJsonAttributes
{
    [JsonPropertyName("first_name")]
    public string FirstName { get; init; }
    
    [JsonPropertyName("last_name")]
    public string LastName { get; init; }
    
    [JsonPropertyName("birth_date")]
    [JsonConverter(typeof(DateOnlyJsonConverter))]
    public DateTime DateOfBirth { get; init; }
    
    [JsonIgnore]
    public int Age => DateTime.Now.Year - DateOfBirth.Year;
}

// DateTime için özel JSON dönüştürücü
public class DateOnlyJsonConverter : JsonConverter<DateTime>
{
    public override DateTime Read(ref Utf8JsonReader reader, Type typeToConvert, JsonSerializerOptions options)
    {
        return DateTime.Parse(reader.GetString());
    }
    
    public override void Write(Utf8JsonWriter writer, DateTime value, JsonSerializerOptions options)
    {
        writer.WriteStringValue(value.ToString("yyyy-MM-dd"));
    }
}

// Kullanım örneği
public void CustomJsonSerializationExample()
{
    var person = new PersonWithJsonAttributes
    {
        FirstName = "Ahmet",
        LastName = "Yılmaz",
        DateOfBirth = new DateTime(1985, 5, 15)
    };
    
    // JSON'a serileştirme
    string json = JsonSerializer.Serialize(person, new JsonSerializerOptions
    {
        WriteIndented = true
    });
    
    Console.WriteLine("Özel JSON:");
    Console.WriteLine(json);
}
```

## Gerçek Dünya Uygulamaları

### Veri Transfer Nesneleri (DTO)

Records, özellikle Veri Transfer Nesneleri (DTO) için idealdir.

```csharp
// API yanıt modelleri
public record UserDto(int Id, string Username, string Email);
public record ProductDto(int Id, string Name, decimal Price, string Category);
public record OrderDto(int Id, DateTime OrderDate, UserDto User, List<ProductDto> Products);

// API servisi
public class ApiService
{
    public async Task<OrderDto> GetOrderAsync(int orderId)
    {
        // Gerçek uygulamada, bu veriler bir veritabanından veya başka bir API'den gelir
        await Task.Delay(100); // Simüle edilmiş gecikme
        
        return new OrderDto(
            orderId,
            DateTime.Now.AddDays(-5),
            new UserDto(1, "ahmet", "ahmet@example.com"),
            new List<ProductDto>
            {
                new ProductDto(101, "Laptop", 5000m, "Elektronik"),
                new ProductDto(102, "Mouse", 200m, "Elektronik")
            }
        );
    }
}

// Kullanım örneği
public async Task DtoExample()
{
    var apiService = new ApiService();
    var order = await apiService.GetOrderAsync(12345);
    
    Console.WriteLine($"Sipariş #{order.Id}, Tarih: {order.OrderDate:d}");
    Console.WriteLine($"Müşteri: {order.User.Username} ({order.User.Email})");
    Console.WriteLine("Ürünler:");
    
    foreach (var product in order.Products)
    {
        Console.WriteLine($"- {product.Name}: {product.Price:C} ({product.Category})");
    }
}
```

### Domain Modelleri

Records, Domain-Driven Design (DDD) için değer nesneleri olarak da kullanılabilir.

```csharp
// Değer nesneleri
public record Money(decimal Amount, string Currency);
public record Address(string Street, string City, string PostalCode, string Country);
public record PersonName(string FirstName, string LastName)
{
    public string FullName => $"{FirstName} {LastName}";
}

// Domain modeli
public record Customer
{
    public int Id { get; init; }
    public PersonName Name { get; init; }
    public Address ShippingAddress { get; init; }
    public List<Order> Orders { get; init; } = new List<Order>();
}

public record Order
{
    public int Id { get; init; }
    public DateTime OrderDate { get; init; }
    public Money TotalAmount { get; init; }
    public List<OrderLine> Lines { get; init; } = new List<OrderLine>();
}

public record OrderLine(int ProductId, string ProductName, int Quantity, Money UnitPrice)
{
    public Money TotalPrice => new Money(UnitPrice.Amount * Quantity, UnitPrice.Currency);
}

// Kullanım örneği
public void DomainModelExample()
{
    var customer = new Customer
    {
        Id = 1,
        Name = new PersonName("Ahmet", "Yılmaz"),
        ShippingAddress = new Address("Bağdat Caddesi", "İstanbul", "34000", "Türkiye"),
        Orders = new List<Order>
        {
            new Order
            {
                Id = 1001,
                OrderDate = DateTime.Now.AddDays(-10),
                TotalAmount = new Money(5200m, "TRY"),
                Lines = new List<OrderLine>
                {
                    new OrderLine(101, "Laptop", 1, new Money(5000m, "TRY")),
                    new OrderLine(102, "Mouse", 1, new Money(200m, "TRY"))
                }
            }
        }
    };
    
    Console.WriteLine($"Müşteri: {customer.Name.FullName}");
    Console.WriteLine($"Adres: {customer.ShippingAddress.Street}, {customer.ShippingAddress.City}");
    
    foreach (var order in customer.Orders)
    {
        Console.WriteLine($"Sipariş #{order.Id}, Tarih: {order.OrderDate:d}, Toplam: {order.TotalAmount.Amount} {order.TotalAmount.Currency}");
        
        foreach (var line in order.Lines)
        {
            Console.WriteLine($"- {line.ProductName} x {line.Quantity}: {line.TotalPrice.Amount} {line.TotalPrice.Currency}");
        }
    }
}
```

## Özet

Bu bölümde, C# 9.0 ile tanıtılan record türlerini inceledik:

1. **Record Declaration**: Record'lar, değer semantiğine sahip referans türleridir ve `record` anahtar kelimesi ile tanımlanır.

2. **Immutability**: Record'lar, varsayılan olarak değişmez (immutable) olacak şekilde tasarlanmıştır ve özellikleri `init` erişimcisi ile tanımlanır.

3. **With-expressions**: "With" ifadeleri, mevcut bir kaydın değiştirilmiş bir kopyasını oluşturmak için kullanılır.

4. **Inheritance**: Record'lar, diğer record'lardan türetilebilir, ancak sınıflardan türetilemez veya sınıflar record'lardan türetilemez.

5. **Equality**: Record'lar, değer semantiğine sahiptir, yani içerikleri aynı olan iki record eşit kabul edilir.

6. **Deconstruction**: Pozisyonel sözdizimi ile tanımlanan record'lar, otomatik olarak bir Deconstruct metodu içerir.

7. **JSON Serialization**: Record'lar, JSON serileştirme ve ters serileştirme için çok uygundur.

Record'lar, özellikle değişmez veri modelleri, DTO'lar ve değer nesneleri için idealdir. Kodunuzu daha özlü ve daha az hataya açık hale getirmenize yardımcı olurlar. 