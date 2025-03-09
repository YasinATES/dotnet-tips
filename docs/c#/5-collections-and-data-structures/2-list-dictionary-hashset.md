# List, Dictionary ve HashSet

Bu bölümde, C#'ın en yaygın kullanılan üç koleksiyon türünü detaylı olarak inceleyeceğiz: `List<T>`, `Dictionary<TKey, TValue>` ve `HashSet<T>`. Bu koleksiyonlar, farklı senaryolarda veri depolama ve işleme ihtiyaçlarını karşılamak için tasarlanmıştır.

## 1. List<T> Operations ve Methods

`List<T>`, dinamik boyutlu bir dizi gibi davranan ve sıralı veri depolamak için kullanılan en yaygın koleksiyon türüdür. `System.Collections.Generic` namespace'inde bulunur ve `IList<T>`, `ICollection<T>` ve `IEnumerable<T>` arayüzlerini uygular.

### Temel List<T> Operasyonları

```csharp
// List oluşturma
List<string> cities = new List<string>();

// Eleman ekleme
cities.Add("İstanbul");
cities.Add("Ankara");
cities.Add("İzmir");

// Belirli bir konuma eleman ekleme
cities.Insert(1, "Bursa"); // { "İstanbul", "Bursa", "Ankara", "İzmir" }

// Eleman çıkarma
cities.Remove("Ankara"); // { "İstanbul", "Bursa", "İzmir" }
cities.RemoveAt(0); // { "Bursa", "İzmir" }

// Belirli bir aralığı çıkarma
cities.Add("Antalya");
cities.Add("Adana");
cities.RemoveRange(1, 2); // { "Bursa", "Adana" }

// Tüm elemanları temizleme
cities.Clear(); // { }
```

### Arama ve Sorgulama

```csharp
List<int> numbers = new List<int> { 5, 10, 15, 20, 25, 30, 15 };

// Eleman arama
bool contains = numbers.Contains(15); // true

// İndeks bulma
int index = numbers.IndexOf(15); // 2 (ilk bulunan)
int lastIndex = numbers.LastIndexOf(15); // 6 (son bulunan)

// Belirli bir koşula göre arama
int firstGreaterThan20 = numbers.Find(n => n > 20); // 25
List<int> allGreaterThan20 = numbers.FindAll(n => n > 20); // { 25, 30 }

// Belirli bir koşula göre indeks bulma
int indexGreaterThan20 = numbers.FindIndex(n => n > 20); // 4
int lastIndexGreaterThan20 = numbers.FindLastIndex(n => n > 20); // 5

// Tüm elemanların koşulu sağlayıp sağlamadığını kontrol etme
bool allGreaterThan0 = numbers.TrueForAll(n => n > 0); // true
bool allGreaterThan10 = numbers.TrueForAll(n => n > 10); // false
```

### Sıralama ve Dönüştürme

```csharp
List<int> numbers = new List<int> { 5, 2, 8, 1, 9 };

// Sıralama
numbers.Sort(); // { 1, 2, 5, 8, 9 }

// Ters çevirme
numbers.Reverse(); // { 9, 8, 5, 2, 1 }

// Özel sıralama
numbers.Sort((a, b) => b.CompareTo(a)); // Azalan sıralama: { 9, 8, 5, 2, 1 }

// Diziye dönüştürme
int[] array = numbers.ToArray();

// Başka bir listeye kopyalama
List<int> copy = new List<int>(numbers.Count);
numbers.ForEach(n => copy.Add(n));

// ForEach ile işlem yapma
numbers.ForEach(n => Console.WriteLine(n));
```

### Kapasite Yönetimi

```csharp
// Başlangıç kapasitesi ile liste oluşturma
List<double> values = new List<double>(10000);

// Mevcut kapasite
Console.WriteLine($"Kapasite: {values.Capacity}"); // 10000

// Kapasiteyi manuel ayarlama
values.Capacity = 20000;

// Kapasiteyi mevcut eleman sayısına düşürme
values.TrimExcess();
```

### Range Operasyonları (C# 8.0+)

```csharp
List<char> letters = new List<char> { 'a', 'b', 'c', 'd', 'e', 'f' };

// Range ile alt liste alma
List<char> subList = letters.GetRange(1, 3); // { 'b', 'c', 'd' }

// C# 8.0+ ile range operatörü kullanımı
var slice1 = letters.ToArray()[1..4]; // { 'b', 'c', 'd' }
var slice2 = letters.ToArray()[..3]; // { 'a', 'b', 'c' }
var slice3 = letters.ToArray()[3..]; // { 'd', 'e', 'f' }
```

## 2. Dictionary<TKey, TValue> Implementation

`Dictionary<TKey, TValue>`, anahtar-değer çiftlerini depolamak için kullanılan bir koleksiyon türüdür. Her anahtar benzersiz olmalıdır ve bir değere hızlı erişim sağlar. Dahili olarak hash tablosu kullanarak O(1) zaman karmaşıklığında arama, ekleme ve silme işlemleri sunar.

### Temel Dictionary Operasyonları

```csharp
// Dictionary oluşturma
Dictionary<string, int> population = new Dictionary<string, int>();

// Eleman ekleme
population.Add("İstanbul", 15462452);
population.Add("Ankara", 5503985);
population["İzmir"] = 4367251; // Alternatif ekleme yöntemi

// Değer güncelleme
population["İstanbul"] = 15519267;

// Eleman çıkarma
population.Remove("Ankara");

// Tüm elemanları temizleme
population.Clear();
```

### Arama ve Sorgulama

```csharp
Dictionary<string, int> scores = new Dictionary<string, int>
{
    { "Ali", 85 },
    { "Ayşe", 92 },
    { "Mehmet", 78 },
    { "Zeynep", 95 }
};

// Anahtar kontrolü
bool containsAli = scores.ContainsKey("Ali"); // true
bool containsEmre = scores.ContainsKey("Emre"); // false

// Değer kontrolü
bool contains92 = scores.ContainsValue(92); // true

// Güvenli değer alma
if (scores.TryGetValue("Mehmet", out int mehmetScore))
{
    Console.WriteLine($"Mehmet'in puanı: {mehmetScore}");
}

// Anahtarları ve değerleri alma
ICollection<string> names = scores.Keys;
ICollection<int> allScores = scores.Values;

// Eleman sayısı
int count = scores.Count; // 4
```

### Dictionary İterasyonu

```csharp
Dictionary<string, string> capitals = new Dictionary<string, string>
{
    { "Türkiye", "Ankara" },
    { "Almanya", "Berlin" },
    { "Fransa", "Paris" },
    { "İtalya", "Roma" }
};

// KeyValuePair ile döngü
foreach (KeyValuePair<string, string> pair in capitals)
{
    Console.WriteLine($"{pair.Key}: {pair.Value}");
}

// Deconstruction ile döngü (C# 7.0+)
foreach (var (country, capital) in capitals)
{
    Console.WriteLine($"{country}: {capital}");
}

// Sadece anahtarlar üzerinde döngü
foreach (string country in capitals.Keys)
{
    Console.WriteLine(country);
}

// Sadece değerler üzerinde döngü
foreach (string capital in capitals.Values)
{
    Console.WriteLine(capital);
}
```

### Dictionary İç Yapısı

`Dictionary<TKey, TValue>` dahili olarak bir hash tablosu kullanır. Her anahtar için bir hash kodu hesaplanır ve bu hash kodu, değerin depolanacağı "bucket" (kova) indeksini belirler. Çakışma durumunda (farklı anahtarların aynı hash kodunu üretmesi), çakışma çözme teknikleri kullanılır.

```csharp
// Dictionary'nin iç yapısını gösteren basit bir örnek
public class SimpleDictionary<TKey, TValue>
{
    private class Entry
    {
        public int HashCode;
        public TKey Key;
        public TValue Value;
        public Entry Next; // Çakışma durumunda bağlı liste için
    }
    
    private Entry[] _buckets;
    private int _count;
    private readonly IEqualityComparer<TKey> _comparer;
    
    public SimpleDictionary(IEqualityComparer<TKey> comparer = null)
    {
        _buckets = new Entry[7]; // Başlangıç kapasitesi
        _comparer = comparer ?? EqualityComparer<TKey>.Default;
    }
    
    public void Add(TKey key, TValue value)
    {
        // Hash kodu hesaplama
        int hashCode = _comparer.GetHashCode(key) & 0x7FFFFFFF;
        
        // Bucket indeksi hesaplama
        int bucketIndex = hashCode % _buckets.Length;
        
        // Çakışma kontrolü
        for (Entry entry = _buckets[bucketIndex]; entry != null; entry = entry.Next)
        {
            if (entry.HashCode == hashCode && _comparer.Equals(entry.Key, key))
                throw new ArgumentException("Anahtar zaten var.");
        }
        
        // Yeni entry oluşturma
        Entry newEntry = new Entry
        {
            HashCode = hashCode,
            Key = key,
            Value = value,
            Next = _buckets[bucketIndex]
        };
        
        // Bucket'a ekleme
        _buckets[bucketIndex] = newEntry;
        _count++;
        
        // Gerekirse yeniden boyutlandırma (rehashing)
        if (_count > _buckets.Length * 0.75)
        {
            Resize();
        }
    }
    
    private void Resize()
    {
        // Yeniden boyutlandırma mantığı...
    }
    
    // Diğer metotlar...
}
```

## 3. HashSet<T> ve Set Operations

`HashSet<T>`, benzersiz elemanları depolamak için kullanılan bir koleksiyon türüdür. Dahili olarak hash tablosu kullanarak O(1) zaman karmaşıklığında arama, ekleme ve silme işlemleri sunar. Ayrıca, küme işlemleri (birleşim, kesişim, fark) için metotlar sağlar.

### Temel HashSet Operasyonları

```csharp
// HashSet oluşturma
HashSet<int> numbers = new HashSet<int>();

// Eleman ekleme
numbers.Add(1);
numbers.Add(2);
numbers.Add(3);
numbers.Add(1); // Tekrar eden eleman eklenmez

// Eleman çıkarma
numbers.Remove(2);

// Eleman kontrolü
bool contains3 = numbers.Contains(3); // true

// Tüm elemanları temizleme
numbers.Clear();
```

### Küme İşlemleri

```csharp
HashSet<int> set1 = new HashSet<int> { 1, 2, 3, 4, 5 };
HashSet<int> set2 = new HashSet<int> { 3, 4, 5, 6, 7 };

// Birleşim (Union): A ∪ B
HashSet<int> union = new HashSet<int>(set1);
union.UnionWith(set2); // { 1, 2, 3, 4, 5, 6, 7 }

// Kesişim (Intersection): A ∩ B
HashSet<int> intersection = new HashSet<int>(set1);
intersection.IntersectWith(set2); // { 3, 4, 5 }

// Fark (Difference): A - B
HashSet<int> difference = new HashSet<int>(set1);
difference.ExceptWith(set2); // { 1, 2 }

// Simetrik Fark (Symmetric Difference): (A - B) ∪ (B - A)
HashSet<int> symmetricDifference = new HashSet<int>(set1);
symmetricDifference.SymmetricExceptWith(set2); // { 1, 2, 6, 7 }

// Alt küme kontrolü
bool isSubset = set1.IsSubsetOf(new HashSet<int> { 1, 2, 3, 4, 5, 6 }); // true

// Üst küme kontrolü
bool isSuperset = set1.IsSupersetOf(new HashSet<int> { 1, 2, 3 }); // true

// Kesişim kontrolü
bool overlaps = set1.Overlaps(set2); // true

// Eşitlik kontrolü
bool equals = set1.SetEquals(new HashSet<int> { 5, 4, 3, 2, 1 }); // true (sıra önemli değil)
```

### HashSet İterasyonu

```csharp
HashSet<string> fruits = new HashSet<string> { "Elma", "Armut", "Muz", "Portakal" };

// Foreach ile döngü
foreach (string fruit in fruits)
{
    Console.WriteLine(fruit);
}

// LINQ ile işlemler
var sortedFruits = fruits.OrderBy(f => f);
var filteredFruits = fruits.Where(f => f.Length > 4);
```

## 4. Lookup Performance

Farklı koleksiyon türleri, arama (lookup) işlemleri için farklı performans özellikleri gösterir. Aşağıda, yaygın koleksiyon türlerinin arama performansı karşılaştırılmıştır:

### Arama Performansı Karşılaştırması

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;

class Program
{
    static void Main()
    {
        const int itemCount = 1000000;
        const int searchCount = 10000;
        Random random = new Random(42);
        
        // Test verileri oluşturma
        int[] items = Enumerable.Range(0, itemCount).ToArray();
        int[] searchItems = Enumerable.Range(0, searchCount)
            .Select(_ => random.Next(0, itemCount))
            .ToArray();
        
        // List<T> ile arama
        List<int> list = new List<int>(items);
        MeasurePerformance("List<T>.Contains", () => {
            foreach (int item in searchItems)
            {
                list.Contains(item);
            }
        });
        
        // Array.IndexOf ile arama
        MeasurePerformance("Array.IndexOf", () => {
            foreach (int item in searchItems)
            {
                Array.IndexOf(items, item);
            }
        });
        
        // HashSet<T> ile arama
        HashSet<int> hashSet = new HashSet<int>(items);
        MeasurePerformance("HashSet<T>.Contains", () => {
            foreach (int item in searchItems)
            {
                hashSet.Contains(item);
            }
        });
        
        // Dictionary<TKey, TValue> ile arama
        Dictionary<int, int> dictionary = items.ToDictionary(x => x, x => x);
        MeasurePerformance("Dictionary<K,V>.ContainsKey", () => {
            foreach (int item in searchItems)
            {
                dictionary.ContainsKey(item);
            }
        });
        
        // SortedSet<T> ile arama
        SortedSet<int> sortedSet = new SortedSet<int>(items);
        MeasurePerformance("SortedSet<T>.Contains", () => {
            foreach (int item in searchItems)
            {
                sortedSet.Contains(item);
            }
        });
        
        // SortedDictionary<TKey, TValue> ile arama
        SortedDictionary<int, int> sortedDictionary = new SortedDictionary<int, int>(dictionary);
        MeasurePerformance("SortedDictionary<K,V>.ContainsKey", () => {
            foreach (int item in searchItems)
            {
                sortedDictionary.ContainsKey(item);
            }
        });
    }
    
    static void MeasurePerformance(string name, Action action)
    {
        GC.Collect();
        GC.WaitForPendingFinalizers();
        
        Stopwatch stopwatch = Stopwatch.StartNew();
        action();
        stopwatch.Stop();
        
        Console.WriteLine($"{name}: {stopwatch.ElapsedMilliseconds} ms");
    }
}
```

### Arama Performansı Sonuçları

Tipik sonuçlar (donanıma bağlı olarak değişebilir):

```
List<T>.Contains: 3245 ms
Array.IndexOf: 3187 ms
HashSet<T>.Contains: 2 ms
Dictionary<K,V>.ContainsKey: 2 ms
SortedSet<T>.Contains: 18 ms
SortedDictionary<K,V>.ContainsKey: 19 ms
```

Bu sonuçlar, `HashSet<T>` ve `Dictionary<TKey, TValue>`'nin arama işlemleri için en iyi performansı gösterdiğini, ardından `SortedSet<T>` ve `SortedDictionary<TKey, TValue>`'nin geldiğini, en yavaş olanların ise `List<T>` ve dizi olduğunu göstermektedir.

## 5. Custom Equality Comparers

Varsayılan olarak, koleksiyonlar eşitlik karşılaştırması için `EqualityComparer<T>.Default`'u kullanır. Ancak, özel karşılaştırma mantığı gerektiğinde, `IEqualityComparer<T>` arayüzünü uygulayan özel karşılaştırıcılar oluşturabilirsiniz.

### IEqualityComparer<T> Uygulaması

```csharp
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime BirthDate { get; set; }
    
    public override string ToString()
    {
        return $"{FirstName} {LastName} ({BirthDate:d})";
    }
}

// Özel eşitlik karşılaştırıcı
public class PersonNameComparer : IEqualityComparer<Person>
{
    public bool Equals(Person x, Person y)
    {
        if (x == null && y == null)
            return true;
        if (x == null || y == null)
            return false;
            
        return string.Equals(x.FirstName, y.FirstName, StringComparison.OrdinalIgnoreCase) &&
               string.Equals(x.LastName, y.LastName, StringComparison.OrdinalIgnoreCase);
    }
    
    public int GetHashCode(Person obj)
    {
        if (obj == null)
            return 0;
            
        int hash = 17;
        hash = hash * 23 + (obj.FirstName?.ToLower().GetHashCode() ?? 0);
        hash = hash * 23 + (obj.LastName?.ToLower().GetHashCode() ?? 0);
        return hash;
    }
}
```

### Özel Karşılaştırıcı Kullanımı

```csharp
// Test verileri
var people = new List<Person>
{
    new Person { FirstName = "Ali", LastName = "Yılmaz", BirthDate = new DateTime(1980, 5, 15) },
    new Person { FirstName = "Ayşe", LastName = "Demir", BirthDate = new DateTime(1985, 8, 22) },
    new Person { FirstName = "Mehmet", LastName = "Kaya", BirthDate = new DateTime(1975, 3, 10) },
    new Person { FirstName = "ali", LastName = "YILMAZ", BirthDate = new DateTime(1990, 12, 5) }
};

// Varsayılan karşılaştırıcı ile HashSet
HashSet<Person> uniquePeople = new HashSet<Person>();
foreach (var person in people)
{
    bool added = uniquePeople.Add(person);
    Console.WriteLine($"Eklendi: {added} - {person}");
}
// Tüm kişiler eklenir (4 kişi)

// Özel karşılaştırıcı ile HashSet
HashSet<Person> uniqueByName = new HashSet<Person>(new PersonNameComparer());
foreach (var person in people)
{
    bool added = uniqueByName.Add(person);
    Console.WriteLine($"Eklendi: {added} - {person}");
}
// Sadece benzersiz isimler eklenir (3 kişi, "Ali Yılmaz" tekrar etmez)

// Özel karşılaştırıcı ile Dictionary
Dictionary<Person, string> personNotes = new Dictionary<Person, string>(new PersonNameComparer());
personNotes.Add(people[0], "İlk kişi");
// personNotes.Add(people[3], "Hata!"); // Anahtar zaten var hatası
```

### Lambda ile Karşılaştırıcı Oluşturma

C# 10.0 ve sonrasında, lambda ifadeleri kullanarak daha kısa bir şekilde karşılaştırıcılar oluşturabilirsiniz:

```csharp
// C# 10.0+ ile lambda kullanarak karşılaştırıcı oluşturma
IEqualityComparer<Person> nameComparer = 
    EqualityComparer<Person>.Create(
        (x, y) => string.Equals(x.FirstName, y.FirstName, StringComparison.OrdinalIgnoreCase) &&
                  string.Equals(x.LastName, y.LastName, StringComparison.OrdinalIgnoreCase),
        obj => {
            int hash = 17;
            hash = hash * 23 + (obj.FirstName?.ToLower().GetHashCode() ?? 0);
            hash = hash * 23 + (obj.LastName?.ToLower().GetHashCode() ?? 0);
            return hash;
        }
    );

// Kullanım
HashSet<Person> uniqueByNameLambda = new HashSet<Person>(nameComparer);
```

## 6. Collection Capacity Management

Koleksiyonların kapasitesini doğru yönetmek, performans açısından önemlidir. Özellikle büyük miktarda veri eklerken, kapasite yönetimi bellek kullanımını ve performansı etkileyebilir.

### List<T> Kapasite Yönetimi

`List<T>`, dahili olarak bir dizi kullanır ve eleman sayısı kapasiteyi aştığında, kapasiteyi artırır (genellikle mevcut kapasitenin 2 katına çıkarır).

```csharp
// Kapasite izleme örneği
List<int> list = new List<int>();
Console.WriteLine($"Başlangıç kapasitesi: {list.Capacity}"); // Genellikle 0

for (int i = 0; i < 10; i++)
{
    list.Add(i);
    Console.WriteLine($"Eleman sayısı: {list.Count}, Kapasite: {list.Capacity}");
}
// Tipik çıktı:
// Eleman sayısı: 1, Kapasite: 4
// Eleman sayısı: 2, Kapasite: 4
// ...
// Eleman sayısı: 5, Kapasite: 8
// ...
// Eleman sayısı: 9, Kapasite: 16
// Eleman sayısı: 10, Kapasite: 16

// Kapasiteyi manuel ayarlama
list.Capacity = 100;

// Kapasiteyi mevcut eleman sayısına düşürme
list.TrimExcess();
Console.WriteLine($"TrimExcess sonrası kapasite: {list.Capacity}"); // 10
```

### Dictionary<TKey, TValue> Kapasite Yönetimi

`Dictionary<TKey, TValue>` de benzer şekilde, eleman sayısı belirli bir eşiği aştığında kapasitesini artırır. Başlangıç kapasitesini belirtmek, performansı artırabilir.

```csharp
// Başlangıç kapasitesi ile Dictionary oluşturma
Dictionary<int, string> dictionary = new Dictionary<int, string>(10000);

// Çok sayıda eleman ekleme
for (int i = 0; i < 10000; i++)
{
    dictionary.Add(i, $"Item {i}");
}
```

### HashSet<T> Kapasite Yönetimi

`HashSet<T>` için de başlangıç kapasitesini belirtmek mümkündür:

```csharp
// Başlangıç kapasitesi ile HashSet oluşturma
HashSet<int> hashSet = new HashSet<int>(10000);

// Çok sayıda eleman ekleme
for (int i = 0; i < 10000; i++)
{
    hashSet.Add(i);
}
```

### Kapasite Yönetimi Performans Testi

```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;

class Program
{
    static void Main()
    {
        const int itemCount = 1000000;
        
        // Kapasite belirtmeden
        MeasurePerformance("List<T> (kapasite belirtilmemiş)", () => {
            List<int> list = new List<int>();
            for (int i = 0; i < itemCount; i++)
            {
                list.Add(i);
            }
        });
        
        // Kapasite belirterek
        MeasurePerformance("List<T> (kapasite belirtilmiş)", () => {
            List<int> list = new List<int>(itemCount);
            for (int i = 0; i < itemCount; i++)
            {
                list.Add(i);
            }
        });
        
        // Dictionary için benzer test
        MeasurePerformance("Dictionary<K,V> (kapasite belirtilmemiş)", () => {
            Dictionary<int, int> dict = new Dictionary<int, int>();
            for (int i = 0; i < itemCount; i++)
            {
                dict.Add(i, i);
            }
        });
        
        MeasurePerformance("Dictionary<K,V> (kapasite belirtilmiş)", () => {
            Dictionary<int, int> dict = new Dictionary<int, int>(itemCount);
            for (int i = 0; i < itemCount; i++)
            {
                dict.Add(i, i);
            }
        });
    }
    
    static void MeasurePerformance(string name, Action action)
    {
        GC.Collect();
        GC.WaitForPendingFinalizers();
        
        Stopwatch stopwatch = Stopwatch.StartNew();
        action();
        stopwatch.Stop();
        
        Console.WriteLine($"{name}: {stopwatch.ElapsedMilliseconds} ms");
    }
}
```

### Kapasite Yönetimi Sonuçları

Tipik sonuçlar (donanıma bağlı olarak değişebilir):

```
List<T> (kapasite belirtilmemiş): 85 ms
List<T> (kapasite belirtilmiş): 42 ms
Dictionary<K,V> (kapasite belirtilmemiş): 312 ms
Dictionary<K,V> (kapasite belirtilmiş): 187 ms
```

Bu sonuçlar, başlangıç kapasitesini belirtmenin, özellikle büyük miktarda veri eklerken, önemli bir performans artışı sağlayabileceğini göstermektedir.

## 7. En İyi Pratikler

1. **Doğru Koleksiyon Türünü Seçin**
   - Sıralı veriler ve indeks tabanlı erişim için `List<T>`
   - Anahtar-değer çiftleri için `Dictionary<TKey, TValue>`
   - Benzersiz elemanlar ve küme işlemleri için `HashSet<T>`

2. **Kapasite Yönetimine Dikkat Edin**
   - Eleman sayısı biliniyorsa, başlangıç kapasitesini belirtin
   - Gereksiz bellek kullanımını önlemek için `TrimExcess` kullanın

3. **Arama Performansını Optimize Edin**
   - Sık arama gerektiren işlemler için `Dictionary<TKey, TValue>` veya `HashSet<T>` kullanın
   - Özel arama gereksinimleri için uygun karşılaştırıcılar oluşturun

4. **Koleksiyonları Değiştirirken Dikkatli Olun**
   - Bir koleksiyon üzerinde döngü yaparken, koleksiyonu değiştirmekten kaçının
   - Değiştirme gerekiyorsa, koleksiyonun bir kopyasını oluşturun veya tersten döngü kullanın
   - `RemoveAll` gibi güvenli metotları tercih edin

5. **Thread Safety'ye Dikkat Edin**
   - Çoklu iş parçacığı ortamında, thread-safe koleksiyonlar kullanın (`ConcurrentDictionary<TKey, TValue>` gibi)
   - Gerekirse koleksiyonları kilitleyin (`lock` kullanarak)
   - İmmutable koleksiyonları düşünün

6. **LINQ ile Koleksiyonları Etkili Kullanın**
   - Gereksiz LINQ zincirleri oluşturmaktan kaçının
   - Büyük koleksiyonlarda `ToList()` veya `ToArray()` çağrılarını minimize edin
   - Lazy evaluation'ı anlaşın ve kullanın

7. **Bellek Kullanımını Optimize Edin**
   - Kullanılmayan koleksiyonları temizleyin
   - Büyük koleksiyonları parçalara ayırın
   - Gerektiğinde `struct` tabanlı koleksiyonları düşünün