# Generic Types ve Methods

C# dilinde generics, tip güvenliği sağlayan ve kod tekrarını azaltan güçlü bir özelliktir. Generics, farklı veri tipleriyle çalışabilen sınıflar, arayüzler ve metotlar tanımlamanıza olanak tanır. Bu bölümde, C#'taki generic yapıları detaylı olarak inceleyeceğiz.

## 1. Generic Class Definition

Generic sınıflar, çalışacakları veri tipini çalışma zamanında belirleyen sınıflardır. Bu sayede, farklı veri tipleri için aynı işlevselliği sağlayan kodun tekrar yazılmasına gerek kalmaz.

```csharp
// Generic sınıf tanımlama
public class Repository<T>
{
    private List<T> _items = new List<T>();
    
    public void Add(T item)
    {
        _items.Add(item);
    }
    
    public T GetById(int id)
    {
        // Örnek amaçlı basitleştirilmiş kod
        return _items[id];
    }
    
    public List<T> GetAll()
    {
        return _items;
    }
}
```

Generic sınıfları kullanmak:

```csharp
// String tipinde bir repository
Repository<string> stringRepo = new Repository<string>();
stringRepo.Add("Merhaba");
stringRepo.Add("Dünya");
string item = stringRepo.GetById(0); // "Merhaba"

// Customer tipinde bir repository
Repository<Customer> customerRepo = new Repository<Customer>();
customerRepo.Add(new Customer { Id = 1, Name = "Ahmet" });
customerRepo.Add(new Customer { Id = 2, Name = "Mehmet" });
Customer customer = customerRepo.GetById(0); // Ahmet
```

### Generic Sınıflarda Kısıtlamalar

Generic sınıflarda, tip parametrelerinin belirli özelliklere sahip olmasını sağlamak için kısıtlamalar (constraints) kullanabilirsiniz:

```csharp
// T tipinin IComparable arayüzünü uygulaması gerekiyor
public class SortedList<T> where T : IComparable<T>
{
    private List<T> _items = new List<T>();
    
    public void Add(T item)
    {
        _items.Add(item);
        _items.Sort(); // IComparable sayesinde sıralama yapılabilir
    }
}

// T tipinin bir sınıf olması ve parametre almayan bir constructor'a sahip olması gerekiyor
public class Factory<T> where T : class, new()
{
    public T Create()
    {
        return new T();
    }
}
```

## 2. Generic Interface Definition

Generic arayüzler, farklı veri tipleriyle çalışabilen arayüzler tanımlamanıza olanak tanır. Bu, tip güvenliğini korurken kodun yeniden kullanılabilirliğini artırır.

```csharp
// Generic arayüz tanımlama
public interface IRepository<T>
{
    void Add(T item);
    T GetById(int id);
    IEnumerable<T> GetAll();
    void Update(T item);
    void Delete(T item);
}

// Generic arayüzü uygulama
public class CustomerRepository : IRepository<Customer>
{
    private List<Customer> _customers = new List<Customer>();
    
    public void Add(Customer item)
    {
        _customers.Add(item);
    }
    
    public Customer GetById(int id)
    {
        return _customers.FirstOrDefault(c => c.Id == id);
    }
    
    public IEnumerable<Customer> GetAll()
    {
        return _customers;
    }
    
    public void Update(Customer item)
    {
        // Güncelleme işlemi
    }
    
    public void Delete(Customer item)
    {
        _customers.Remove(item);
    }
}
```

Generic arayüzleri birden fazla tip parametresiyle de tanımlayabilirsiniz:

```csharp
public interface IKeyValueStore<TKey, TValue>
{
    void Add(TKey key, TValue value);
    TValue GetByKey(TKey key);
    bool TryGetValue(TKey key, out TValue value);
    bool ContainsKey(TKey key);
    IEnumerable<TKey> GetAllKeys();
}
```

## 3. Generic Method Syntax

Generic metotlar, bir sınıfın veya arayüzün generic olmasına gerek kalmadan, metot düzeyinde generic tip parametreleri tanımlamanıza olanak tanır.

```csharp
// Generic metot tanımlama
public class Utilities
{
    // Generic metot
    public T Max<T>(T first, T second) where T : IComparable<T>
    {
        return first.CompareTo(second) > 0 ? first : second;
    }
    
    // Birden fazla tip parametreli generic metot
    public void Swap<T>(ref T first, ref T second)
    {
        T temp = first;
        first = second;
        second = temp;
    }
    
    // Generic koleksiyon işleyen metot
    public List<TOutput> ConvertList<TInput, TOutput>(List<TInput> items, Func<TInput, TOutput> converter)
    {
        List<TOutput> result = new List<TOutput>();
        foreach (var item in items)
        {
            result.Add(converter(item));
        }
        return result;
    }
}
```

Generic metotları kullanmak:

```csharp
Utilities utils = new Utilities();

// Max metodu ile karşılaştırma
int maxInt = utils.Max<int>(5, 10); // 10
string maxString = utils.Max<string>("abc", "xyz"); // "xyz"

// Tip çıkarımı sayesinde tip parametresini belirtmeye gerek yok
int maxInt2 = utils.Max(5, 10); // 10

// Swap metodu ile değer değiştirme
int a = 5, b = 10;
utils.Swap(ref a, ref b); // a = 10, b = 5

// ConvertList metodu ile tip dönüşümü
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
List<string> numberStrings = utils.ConvertList(numbers, n => n.ToString());
// numberStrings: ["1", "2", "3", "4", "5"]
```

## 4. Type Parameter Naming

C#'ta generic tip parametreleri için yaygın olarak kullanılan isimlendirme kuralları vardır. Bu kurallar, kodun okunabilirliğini artırır ve diğer geliştiricilerin kodunuzu daha kolay anlamasını sağlar.

### Yaygın Tip Parametre İsimleri

- `T`: Tek tip parametresi olan generic sınıflar veya metotlar için kullanılır.
- `TKey`, `TValue`: Anahtar-değer çiftleri için kullanılır.
- `TInput`, `TOutput`: Giriş ve çıkış tipleri için kullanılır.
- `TSource`, `TResult`: Kaynak ve sonuç tipleri için kullanılır.
- `TElement`: Koleksiyon elemanları için kullanılır.

```csharp
// Tek tip parametresi
public class List<T> { }

// Anahtar-değer çiftleri
public class Dictionary<TKey, TValue> { }

// Dönüşüm işlemleri
public TOutput Convert<TInput, TOutput>(TInput input) { }

// LINQ sorguları
public IEnumerable<TResult> Select<TSource, TResult>(
    this IEnumerable<TSource> source, 
    Func<TSource, TResult> selector) { }
```

### İsimlendirme Kuralları

- Tip parametreleri genellikle "T" harfi ile başlar.
- PascalCase kullanılır (ilk harf büyük, sonraki kelimelerin ilk harfleri büyük).
- Anlamlı isimler kullanılmalıdır, özellikle birden fazla tip parametresi olduğunda.
- Tek harfli isimler (T, K, V gibi) basit durumlarda tercih edilebilir.

## 5. Generic Type Inference

C# derleyicisi, çoğu durumda generic metotlar için tip parametrelerini otomatik olarak çıkarabilir. Bu özelliğe "tip çıkarımı" (type inference) denir ve kodunuzu daha temiz ve okunabilir hale getirir.

```csharp
public class TypeInference
{
    // Generic metot
    public T Identity<T>(T value)
    {
        return value;
    }
    
    // Generic koleksiyon metodu
    public List<T> CreateList<T>(params T[] items)
    {
        return new List<T>(items);
    }
}

// Kullanım
TypeInference ti = new TypeInference();

// Açık tip belirtme
int result1 = ti.Identity<int>(42);

// Tip çıkarımı - derleyici int tipini otomatik olarak çıkarır
int result2 = ti.Identity(42);

// Açık tip belirtme
List<string> list1 = ti.CreateList<string>("a", "b", "c");

// Tip çıkarımı - derleyici string tipini otomatik olarak çıkarır
List<string> list2 = ti.CreateList("a", "b", "c");
```

Tip çıkarımı, LINQ sorgularında özellikle kullanışlıdır:

```csharp
// Tip parametrelerini açıkça belirtme
var numbers = Enumerable.Range(1, 10).Where<int>(n => n % 2 == 0).ToList<int>();

// Tip çıkarımı ile daha temiz kod
var numbers2 = Enumerable.Range(1, 10).Where(n => n % 2 == 0).ToList();
```

## 6. Nested Generic Types

C#'ta generic tipler iç içe (nested) olabilir. Bu, karmaşık veri yapıları oluşturmanıza olanak tanır.

```csharp
// İç içe generic sınıf
public class Outer<T>
{
    public T OuterValue { get; set; }
    
    // İç içe generic sınıf
    public class Inner<U>
    {
        public U InnerValue { get; set; }
        public T OuterTypeValue { get; set; } // Dış sınıfın tip parametresine erişim
        
        public Inner(U innerValue, T outerValue)
        {
            InnerValue = innerValue;
            OuterTypeValue = outerValue;
        }
    }
    
    // İç sınıfı kullanan metot
    public Inner<U> CreateInner<U>(U innerValue)
    {
        return new Inner<U>(innerValue, OuterValue);
    }
}
```

İç içe generic tipleri kullanmak:

```csharp
// Dış sınıf örneği
Outer<string> outer = new Outer<string> { OuterValue = "Dış değer" };

// İç sınıf örneği
Outer<string>.Inner<int> inner = outer.CreateInner(42);

Console.WriteLine(inner.InnerValue); // 42
Console.WriteLine(inner.OuterTypeValue); // "Dış değer"

// Doğrudan iç sınıf örneği oluşturma
var inner2 = new Outer<string>.Inner<double>(3.14, "Başka bir dış değer");
```

İç içe generic tipler, veri yapıları ve algoritmaları uygulamak için güçlü bir araçtır. Örneğin, bir ağaç veri yapısı şu şekilde uygulanabilir:

```csharp
public class Tree<T>
{
    public T Value { get; set; }
    public List<Node<T>> Children { get; } = new List<Node<T>>();
    
    public class Node<U> where U : T
    {
        public U Value { get; set; }
        public List<Node<U>> Children { get; } = new List<Node<U>>();
        
        public Node(U value)
        {
            Value = value;
        }
        
        public void AddChild(U childValue)
        {
            Children.Add(new Node<U>(childValue));
        }
    }
}
```

## Özet

Generic tipler ve metotlar, C#'ta tip güvenliğini korurken kodun yeniden kullanılabilirliğini artıran güçlü özelliklerdir. Bu bölümde, generic sınıflar, arayüzler, metotlar, tip parametresi isimlendirme kuralları, tip çıkarımı ve iç içe generic tipler hakkında bilgi edindik.

Generics kullanarak:
- Tip güvenliğini koruyabilirsiniz
- Kod tekrarını azaltabilirsiniz
- Performansı artırabilirsiniz (boxing/unboxing işlemlerini önleyerek)
- Daha esnek ve yeniden kullanılabilir kod yazabilirsiniz

Bir sonraki bölümde, generic kısıtlamaları ve generic koleksiyonları daha detaylı inceleyeceğiz. 