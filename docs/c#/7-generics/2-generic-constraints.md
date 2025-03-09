# Generic Constraints (Generic Kısıtlamalar)

C#'ta generic kısıtlamalar, generic tip parametrelerinin belirli özelliklere sahip olmasını sağlayan güçlü bir özelliktir. Kısıtlamalar, tip parametrelerinin davranışını ve özelliklerini sınırlandırarak daha güvenli ve öngörülebilir kod yazmanıza olanak tanır. Bu bölümde, C#'taki generic kısıtlamaları detaylı olarak inceleyeceğiz.

## 1. where T : class

`where T : class` kısıtlaması, tip parametresinin yalnızca referans tipleri olabileceğini belirtir. Bu, T'nin bir sınıf, arayüz, delege veya dizi olabileceği, ancak değer tipi (struct, enum, primitive tipler) olamayacağı anlamına gelir.

```csharp
// T'nin bir referans tipi olması gerektiğini belirten kısıtlama
public class ReferenceTypeContainer<T> where T : class
{
    public T Item { get; set; }
    
    public bool IsNull()
    {
        return Item == null; // Referans tipleri null olabilir
    }
    
    public void ProcessReference(T item)
    {
        // Referans tiplerine özgü işlemler
        Console.WriteLine($"Referans tipi işleniyor: {item?.GetType().Name ?? "null"}");
    }
}
```

Kullanım örneği:

```csharp
// Geçerli kullanım - string bir referans tipidir
ReferenceTypeContainer<string> stringContainer = new ReferenceTypeContainer<string>();
stringContainer.Item = "Merhaba Dünya";
Console.WriteLine(stringContainer.IsNull()); // false

// Geçerli kullanım - Customer bir sınıftır (referans tipi)
ReferenceTypeContainer<Customer> customerContainer = new ReferenceTypeContainer<Customer>();
customerContainer.Item = new Customer { Id = 1, Name = "Ahmet" };

// Geçersiz kullanım - int bir değer tipidir
// ReferenceTypeContainer<int> intContainer = new ReferenceTypeContainer<int>(); // Derleme hatası
```

`where T : class` kısıtlamasının avantajları:
- Referans tiplerine özgü işlemleri (null kontrolü gibi) güvenle yapabilirsiniz
- Referans tiplerinin özelliklerini kullanabilirsiniz
- Derleme zamanında tip güvenliği sağlar

## 2. where T : struct

`where T : struct` kısıtlaması, tip parametresinin yalnızca değer tipleri olabileceğini belirtir. Bu, T'nin bir struct, enum veya primitive tip (int, double, bool vb.) olabileceği, ancak bir sınıf, arayüz, delege veya dizi olamayacağı anlamına gelir.

```csharp
// T'nin bir değer tipi olması gerektiğini belirten kısıtlama
public class ValueTypeContainer<T> where T : struct
{
    public T Item { get; set; }
    
    public void ProcessValue(T item)
    {
        // Değer tiplerine özgü işlemler
        Console.WriteLine($"Değer tipi işleniyor: {item.GetType().Name}");
        Console.WriteLine($"Varsayılan değer: {default(T)}");
    }
    
    // Değer tipleri null olamaz, bu nedenle her zaman bir değere sahiptir
    public T GetDefaultValue()
    {
        return default(T); // Değer tiplerinin varsayılan değeri tipine göre değişir (int için 0, bool için false vb.)
    }
}
```

Kullanım örneği:

```csharp
// Geçerli kullanım - int bir değer tipidir
ValueTypeContainer<int> intContainer = new ValueTypeContainer<int>();
intContainer.Item = 42;
intContainer.ProcessValue(100); // "Değer tipi işleniyor: Int32" ve "Varsayılan değer: 0"

// Geçerli kullanım - DateTime bir değer tipidir
ValueTypeContainer<DateTime> dateContainer = new ValueTypeContainer<DateTime>();
dateContainer.Item = DateTime.Now;

// Geçersiz kullanım - string bir referans tipidir
// ValueTypeContainer<string> stringContainer = new ValueTypeContainer<string>(); // Derleme hatası
```

`where T : struct` kısıtlamasının avantajları:
- Değer tiplerine özgü işlemleri güvenle yapabilirsiniz
- Boxing/unboxing işlemlerini azaltabilirsiniz
- Değer tiplerinin her zaman bir değere sahip olduğunu garanti edebilirsiniz (null olamazlar)
- Derleme zamanında tip güvenliği sağlar

## 3. where T : new()

`where T : new()` kısıtlaması, tip parametresinin parametre almayan (default) bir constructor'a sahip olması gerektiğini belirtir. Bu kısıtlama, generic sınıf içinde T tipinde yeni nesneler oluşturmanıza olanak tanır.

```csharp
// T'nin parametre almayan bir constructor'a sahip olması gerektiğini belirten kısıtlama
public class Factory<T> where T : new()
{
    // T tipinde yeni bir nesne oluşturur
    public T Create()
    {
        return new T(); // T tipinin parametre almayan bir constructor'ı olduğu için bu işlem güvenlidir
    }
    
    // Belirtilen sayıda T nesnesi içeren bir liste oluşturur
    public List<T> CreateMany(int count)
    {
        List<T> items = new List<T>();
        for (int i = 0; i < count; i++)
        {
            items.Add(new T());
        }
        return items;
    }
}
```

Kullanım örneği:

```csharp
// Customer sınıfı parametre almayan bir constructor'a sahip
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // Parametre almayan constructor
    public Customer()
    {
        Id = 0;
        Name = "Yeni Müşteri";
    }
}

// Geçerli kullanım
Factory<Customer> customerFactory = new Factory<Customer>();
Customer newCustomer = customerFactory.Create(); // Id = 0, Name = "Yeni Müşteri"
List<Customer> customers = customerFactory.CreateMany(5); // 5 adet Customer nesnesi

// Geçerli kullanım - int gibi değer tipleri için de çalışır
Factory<int> intFactory = new Factory<int>();
int newInt = intFactory.Create(); // 0

// Geçersiz kullanım - parametre almayan constructor'ı olmayan bir sınıf
public class Product
{
    public int Id { get; }
    public string Name { get; }
    
    // Sadece parametreli constructor var
    public Product(int id, string name)
    {
        Id = id;
        Name = name;
    }
}

// Factory<Product> productFactory = new Factory<Product>(); // Derleme hatası
```

`where T : new()` kısıtlamasının avantajları:
- Generic sınıf içinde T tipinde yeni nesneler oluşturabilirsiniz
- Factory deseni gibi tasarım desenlerini generic olarak uygulayabilirsiniz
- Nesne oluşturma işlemlerini soyutlayabilirsiniz

## 4. where T : \<base class\>

`where T : <base class>` kısıtlaması, tip parametresinin belirtilen temel sınıftan türetilmiş olması gerektiğini belirtir. Bu, T'nin belirtilen temel sınıfın kendisi veya ondan türetilmiş bir sınıf olabileceği anlamına gelir.

```csharp
// Temel sınıf
public class Animal
{
    public string Name { get; set; }
    
    public virtual void MakeSound()
    {
        Console.WriteLine("Ses çıkarıyor...");
    }
}

// T'nin Animal sınıfından türetilmiş olması gerektiğini belirten kısıtlama
public class AnimalHandler<T> where T : Animal
{
    private List<T> _animals = new List<T>();
    
    public void AddAnimal(T animal)
    {
        _animals.Add(animal);
    }
    
    public void MakeAllSounds()
    {
        foreach (var animal in _animals)
        {
            Console.WriteLine($"{animal.Name}: ");
            animal.MakeSound(); // Animal sınıfının metotlarına erişebiliriz
        }
    }
    
    // T tipinin Animal'dan türetildiğini bildiğimiz için Animal özelliklerine erişebiliriz
    public List<string> GetAnimalNames()
    {
        return _animals.Select(a => a.Name).ToList();
    }
}
```

Kullanım örneği:

```csharp
// Animal'dan türetilmiş sınıflar
public class Dog : Animal
{
    public Dog()
    {
        Name = "Köpek";
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("Hav hav!");
    }
}

public class Cat : Animal
{
    public Cat()
    {
        Name = "Kedi";
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("Miyav!");
    }
}

// Geçerli kullanım
AnimalHandler<Dog> dogHandler = new AnimalHandler<Dog>();
dogHandler.AddAnimal(new Dog());
dogHandler.MakeAllSounds(); // "Köpek: Hav hav!"

AnimalHandler<Animal> animalHandler = new AnimalHandler<Animal>();
animalHandler.AddAnimal(new Dog());
animalHandler.AddAnimal(new Cat());
animalHandler.MakeAllSounds(); // "Köpek: Hav hav!" ve "Kedi: Miyav!"

// Geçersiz kullanım
// AnimalHandler<string> stringHandler = new AnimalHandler<string>(); // Derleme hatası
```

`where T : <base class>` kısıtlamasının avantajları:
- Temel sınıfın özelliklerine ve metotlarına erişebilirsiniz
- Polimorfizmi generic sınıflarda kullanabilirsiniz
- Belirli bir sınıf hiyerarşisine ait tipleri kabul eden generic sınıflar oluşturabilirsiniz

## 5. where T : \<interface\>

`where T : <interface>` kısıtlaması, tip parametresinin belirtilen arayüzü uygulaması gerektiğini belirtir. Bu, T'nin belirtilen arayüzü uygulayan herhangi bir sınıf veya struct olabileceği anlamına gelir.

```csharp
// Arayüz tanımı
public interface ILogger
{
    void Log(string message);
    LogLevel Level { get; set; }
}

public enum LogLevel
{
    Debug,
    Info,
    Warning,
    Error
}

// T'nin ILogger arayüzünü uygulaması gerektiğini belirten kısıtlama
public class LoggingService<T> where T : ILogger
{
    private T _logger;
    
    public LoggingService(T logger)
    {
        _logger = logger;
    }
    
    public void LogMessage(string message)
    {
        _logger.Log(message); // ILogger arayüzünün metotlarına erişebiliriz
    }
    
    public void SetLogLevel(LogLevel level)
    {
        _logger.Level = level; // ILogger arayüzünün özelliklerine erişebiliriz
    }
    
    public T GetLogger()
    {
        return _logger;
    }
}
```

Kullanım örneği:

```csharp
// ILogger arayüzünü uygulayan sınıflar
public class ConsoleLogger : ILogger
{
    public LogLevel Level { get; set; } = LogLevel.Info;
    
    public void Log(string message)
    {
        Console.WriteLine($"[{Level}] {message}");
    }
}

public class FileLogger : ILogger
{
    public LogLevel Level { get; set; } = LogLevel.Warning;
    
    public void Log(string message)
    {
        // Dosyaya yazma işlemi
        Console.WriteLine($"Dosyaya yazılıyor: [{Level}] {message}");
    }
}

// Geçerli kullanım
LoggingService<ConsoleLogger> consoleLoggingService = new LoggingService<ConsoleLogger>(new ConsoleLogger());
consoleLoggingService.LogMessage("Bu bir test mesajıdır"); // "[Info] Bu bir test mesajıdır"

LoggingService<FileLogger> fileLoggingService = new LoggingService<FileLogger>(new FileLogger());
fileLoggingService.LogMessage("Hata oluştu"); // "Dosyaya yazılıyor: [Warning] Hata oluştu"

// Geçersiz kullanım
// LoggingService<string> stringLoggingService = new LoggingService<string>("logger"); // Derleme hatası
```

`where T : <interface>` kısıtlamasının avantajları:
- Arayüzün metotlarına ve özelliklerine erişebilirsiniz
- Farklı sınıfların aynı arayüzü uygulamasını sağlayarak polimorfizmi kullanabilirsiniz
- Bağımlılık enjeksiyonu ve strateji deseni gibi tasarım desenlerini generic olarak uygulayabilirsiniz

## 6. Multiple Constraints (Çoklu Kısıtlamalar)

C#'ta bir tip parametresi için birden fazla kısıtlama belirtebilirsiniz. Bu, tip parametresinin tüm belirtilen kısıtlamaları karşılaması gerektiği anlamına gelir.

```csharp
// Çoklu kısıtlama örneği
// T: 
// 1. IComparable<T> arayüzünü uygulamalı
// 2. IDisposable arayüzünü uygulamalı
// 3. Bir referans tipi olmalı
// 4. Parametre almayan bir constructor'a sahip olmalı
public class MultipleConstraintsExample<T> where T : class, IComparable<T>, IDisposable, new()
{
    private T _item;
    
    public MultipleConstraintsExample()
    {
        _item = new T(); // new() kısıtlaması sayesinde
    }
    
    public void SetItem(T item)
    {
        if (item.CompareTo(_item) > 0) // IComparable<T> kısıtlaması sayesinde
        {
            _item.Dispose(); // IDisposable kısıtlaması sayesinde
            _item = item;
        }
    }
    
    public T GetItem()
    {
        return _item;
    }
    
    public void Dispose()
    {
        _item.Dispose(); // IDisposable kısıtlaması sayesinde
    }
}
```

Kullanım örneği:

```csharp
// Tüm kısıtlamaları karşılayan bir sınıf
public class ComplexResource : IComparable<ComplexResource>, IDisposable
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // new() kısıtlaması için parametre almayan constructor
    public ComplexResource()
    {
        Id = 0;
        Name = "Yeni Kaynak";
    }
    
    // IComparable<T> için CompareTo metodu
    public int CompareTo(ComplexResource other)
    {
        if (other == null) return 1;
        return Id.CompareTo(other.Id);
    }
    
    // IDisposable için Dispose metodu
    public void Dispose()
    {
        Console.WriteLine($"Kaynak temizleniyor: {Name}");
        // Kaynakları temizleme işlemleri
    }
}

// Geçerli kullanım
MultipleConstraintsExample<ComplexResource> example = new MultipleConstraintsExample<ComplexResource>();
example.SetItem(new ComplexResource { Id = 1, Name = "Önemli Kaynak" });
```

Çoklu kısıtlamaların sıralaması önemlidir:
1. `where T : struct` veya `where T : class` kısıtlaması (varsa) ilk olmalıdır
2. Temel sınıf kısıtlaması (varsa) ikinci olmalıdır
3. Arayüz kısıtlamaları herhangi bir sırada olabilir
4. `where T : new()` kısıtlaması (varsa) son olmalıdır

```csharp
// Doğru sıralama
public class CorrectOrder<T> where T : class, BaseClass, IInterface1, IInterface2, new()
{
    // Sınıf içeriği
}

// Yanlış sıralama - derleme hatası
// public class WrongOrder<T> where T : new(), BaseClass, IInterface1, class
// {
//     // Sınıf içeriği
// }
```

## 7. Generic Kısıtlamaların Pratik Kullanımları

Generic kısıtlamalar, gerçek dünya uygulamalarında çeşitli senaryolarda kullanılabilir:

### Repository Deseni

```csharp
// Temel varlık sınıfı
public abstract class Entity
{
    public int Id { get; set; }
}

// Generic repository
public class Repository<T> where T : Entity, new()
{
    private List<T> _items = new List<T>();
    
    public void Add(T item)
    {
        _items.Add(item);
    }
    
    public T GetById(int id)
    {
        return _items.FirstOrDefault(item => item.Id == id);
    }
    
    public T CreateNew()
    {
        return new T();
    }
}
```

### Serileştirme İşlemleri

```csharp
public class JsonSerializer<T> where T : class, new()
{
    public string Serialize(T obj)
    {
        // JSON serileştirme işlemi
        return $"{{\"type\":\"{typeof(T).Name}\"}}"; // Örnek amaçlı basitleştirilmiş
    }
    
    public T Deserialize(string json)
    {
        // JSON deserileştirme işlemi
        return new T(); // Örnek amaçlı basitleştirilmiş
    }
}
```

### Matematiksel İşlemler

```csharp
public static class MathOperations<T> where T : struct, IComparable<T>
{
    public static T Max(T a, T b)
    {
        return a.CompareTo(b) > 0 ? a : b;
    }
    
    public static T Min(T a, T b)
    {
        return a.CompareTo(b) < 0 ? a : b;
    }
}
```

## Özet

Generic kısıtlamalar, C#'ta tip parametrelerinin davranışını ve özelliklerini sınırlandırarak daha güvenli ve öngörülebilir kod yazmanıza olanak tanır. Bu bölümde, çeşitli generic kısıtlamaları ve bunların kullanım senaryolarını inceledik:

- `where T : class` - Tip parametresinin bir referans tipi olması gerektiğini belirtir
- `where T : struct` - Tip parametresinin bir değer tipi olması gerektiğini belirtir
- `where T : new()` - Tip parametresinin parametre almayan bir constructor'a sahip olması gerektiğini belirtir
- `where T : <base class>` - Tip parametresinin belirtilen temel sınıftan türetilmiş olması gerektiğini belirtir
- `where T : <interface>` - Tip parametresinin belirtilen arayüzü uygulaması gerektiğini belirtir
- Çoklu kısıtlamalar - Tip parametresinin birden fazla kısıtlamayı karşılaması gerektiğini belirtir

Generic kısıtlamalar kullanarak:
- Derleme zamanında tip güvenliği sağlayabilirsiniz
- Tip parametrelerinin belirli özelliklere sahip olmasını garanti edebilirsiniz
- Tip parametrelerinin metotlarına ve özelliklerine erişebilirsiniz
- Daha esnek ve yeniden kullanılabilir kod yazabilirsiniz

Bir sonraki bölümde, generic koleksiyonları ve bunların kullanım senaryolarını inceleyeceğiz. 