# LinkedList ve SortedList

Bu bölümde, C#'ın önemli veri yapılarından olan `LinkedList<T>` (Bağlı Liste) ve `SortedList<TKey, TValue>` (Sıralı Liste) koleksiyonlarını inceleyeceğiz. Bu veri yapıları, belirli senaryolarda diğer koleksiyonlara göre avantajlar sunar.

## 1. LinkedList<T> Node Management

`LinkedList<T>`, çift yönlü bağlı liste (doubly linked list) olarak uygulanmış bir koleksiyon türüdür. Her eleman, önceki ve sonraki elemanlara referanslar içeren bir düğüm (node) olarak saklanır.

### LinkedList<T> Yapısı

```csharp
// LinkedList oluşturma
LinkedList<int> linkedList = new LinkedList<int>();

// Eleman ekleme
linkedList.AddLast(10); // 10
linkedList.AddLast(20); // 10 <-> 20
linkedList.AddFirst(5); // 5 <-> 10 <-> 20
linkedList.AddAfter(linkedList.Find(10), 15); // 5 <-> 10 <-> 15 <-> 20
linkedList.AddBefore(linkedList.Find(20), 17); // 5 <-> 10 <-> 15 <-> 17 <-> 20

// Düğüm (node) referanslarını alma
LinkedListNode<int> firstNode = linkedList.First; // 5
LinkedListNode<int> lastNode = linkedList.Last; // 20

// Düğümler arasında gezinme
LinkedListNode<int> currentNode = firstNode;
while (currentNode != null)
{
    Console.WriteLine(currentNode.Value);
    currentNode = currentNode.Next;
}

// Tersten gezinme
currentNode = lastNode;
while (currentNode != null)
{
    Console.WriteLine(currentNode.Value);
    currentNode = currentNode.Previous;
}
```

### LinkedListNode<T> Yönetimi

`LinkedList<T>` sınıfı, düğümleri doğrudan yönetmenize olanak tanır:

```csharp
LinkedList<string> names = new LinkedList<string>();

// Düğüm oluşturma ve ekleme
LinkedListNode<string> node1 = new LinkedListNode<string>("Ali");
LinkedListNode<string> node2 = new LinkedListNode<string>("Ayşe");
LinkedListNode<string> node3 = new LinkedListNode<string>("Mehmet");

names.AddFirst(node1); // Ali
names.AddAfter(node1, node2); // Ali <-> Ayşe
names.AddLast(node3); // Ali <-> Ayşe <-> Mehmet

// Düğüm referansı ile işlemler
LinkedListNode<string> ayseNode = names.Find("Ayşe");
if (ayseNode != null)
{
    Console.WriteLine($"Önceki: {ayseNode.Previous?.Value}"); // Ali
    Console.WriteLine($"Sonraki: {ayseNode.Next?.Value}"); // Mehmet
    
    // Düğümü çıkarma
    names.Remove(ayseNode); // Ali <-> Mehmet
}

// Düğümü taşıma
LinkedListNode<string> mehmetNode = names.FindLast("Mehmet");
names.Remove(mehmetNode);
names.AddFirst(mehmetNode); // Mehmet <-> Ali
```

### LinkedList<T> Performans Özellikleri

`LinkedList<T>`, belirli operasyonlar için diğer koleksiyonlara göre avantajlar sunar:

1. **Eleman Ekleme/Çıkarma**: O(1) - Düğüm referansı varsa
2. **Eleman Arama**: O(n) - Doğrusal arama
3. **Belirli Bir Konuma Ekleme/Çıkarma**: O(1) - Düğüm referansı varsa
4. **Bellek Kullanımı**: Her eleman için ek referans bilgisi gerektirir

## 2. Doubly Linked List Operations

Çift yönlü bağlı liste, her düğümün hem önceki hem de sonraki düğüme referans içerdiği bir veri yapısıdır. Bu yapı, her iki yönde de gezinmeyi mümkün kılar.

### Temel Operasyonlar

```csharp
LinkedList<int> numbers = new LinkedList<int>(new[] { 10, 20, 30, 40, 50 });

// Listenin başından ve sonundan eleman ekleme/çıkarma
numbers.AddFirst(5); // 5 <-> 10 <-> 20 <-> 30 <-> 40 <-> 50
numbers.AddLast(60); // 5 <-> 10 <-> 20 <-> 30 <-> 40 <-> 50 <-> 60
numbers.RemoveFirst(); // 10 <-> 20 <-> 30 <-> 40 <-> 50 <-> 60
numbers.RemoveLast(); // 10 <-> 20 <-> 30 <-> 40 <-> 50

// Belirli bir değeri bulma ve çıkarma
bool removed = numbers.Remove(30); // 10 <-> 20 <-> 40 <-> 50

// Belirli bir değeri arama
LinkedListNode<int> node = numbers.Find(40);
if (node != null)
{
    Console.WriteLine($"Bulunan değer: {node.Value}");
}

// Tüm elemanları temizleme
numbers.Clear();
```

### Düğüm Manipülasyonu

```csharp
LinkedList<char> chars = new LinkedList<char>();

// Düğümleri ekleme
LinkedListNode<char> nodeA = chars.AddLast('A');
LinkedListNode<char> nodeC = chars.AddLast('C');
LinkedListNode<char> nodeD = chars.AddLast('D');

// Düğümler arasına ekleme
chars.AddAfter(nodeA, 'B'); // A <-> B <-> C <-> D

// Düğüm değerini değiştirme
nodeC.Value = 'X'; // A <-> B <-> X <-> D

// Düğümü taşıma
chars.Remove(nodeD);
chars.AddFirst(nodeD); // D <-> A <-> B <-> X
```

### Özel LinkedList Uygulamaları

1. **LRU Cache (Least Recently Used)**
   ```csharp
   public class LRUCache<TKey, TValue>
   {
       private readonly int _capacity;
       private readonly Dictionary<TKey, LinkedListNode<KeyValuePair<TKey, TValue>>> _cache;
       private readonly LinkedList<KeyValuePair<TKey, TValue>> _lruList;
       
       public LRUCache(int capacity)
       {
           _capacity = capacity;
           _cache = new Dictionary<TKey, LinkedListNode<KeyValuePair<TKey, TValue>>>(capacity);
           _lruList = new LinkedList<KeyValuePair<TKey, TValue>>();
       }
       
       public TValue Get(TKey key)
       {
           if (!_cache.TryGetValue(key, out LinkedListNode<KeyValuePair<TKey, TValue>> node))
           {
               return default;
           }
           
           // Kullanılan elemanı listenin başına taşı (en son kullanılan)
           _lruList.Remove(node);
           _lruList.AddFirst(node);
           
           return node.Value.Value;
       }
       
       public void Put(TKey key, TValue value)
       {
           // Eğer anahtar zaten varsa, güncelle ve başa taşı
           if (_cache.TryGetValue(key, out LinkedListNode<KeyValuePair<TKey, TValue>> existingNode))
           {
               _lruList.Remove(existingNode);
               _lruList.AddFirst(new KeyValuePair<TKey, TValue>(key, value));
               _cache[key] = _lruList.First;
               return;
           }
           
           // Kapasite doluysa, en az kullanılanı (listenin sonundakini) çıkar
           if (_cache.Count >= _capacity)
           {
               _cache.Remove(_lruList.Last.Value.Key);
               _lruList.RemoveLast();
           }
           
           // Yeni elemanı ekle
           _lruList.AddFirst(new KeyValuePair<TKey, TValue>(key, value));
           _cache.Add(key, _lruList.First);
       }
   }
   ```

2. **Çift Yönlü Kuyruk (Deque)**
   ```csharp
   public class Deque<T>
   {
       private readonly LinkedList<T> _list = new LinkedList<T>();
       
       public int Count => _list.Count;
       
       public void AddFirst(T item) => _list.AddFirst(item);
       
       public void AddLast(T item) => _list.AddLast(item);
       
       public T RemoveFirst()
       {
           if (_list.Count == 0)
               throw new InvalidOperationException("Deque is empty");
               
           T value = _list.First.Value;
           _list.RemoveFirst();
           return value;
       }
       
       public T RemoveLast()
       {
           if (_list.Count == 0)
               throw new InvalidOperationException("Deque is empty");
               
           T value = _list.Last.Value;
           _list.RemoveLast();
           return value;
       }
       
       public T PeekFirst() => _list.Count > 0 ? _list.First.Value : default;
       
       public T PeekLast() => _list.Count > 0 ? _list.Last.Value : default;
   }
   ```

## 3. SortedList<TKey, TValue> Implementation

`SortedList<TKey, TValue>`, anahtarlara göre sıralanmış bir anahtar-değer koleksiyonudur. Dahili olarak iki ayrı dizi kullanır: biri anahtarlar için, diğeri değerler için.

### Temel SortedList Operasyonları

```csharp
// SortedList oluşturma
SortedList<string, int> scores = new SortedList<string, int>();

// Eleman ekleme (anahtara göre otomatik sıralanır)
scores.Add("Zeynep", 95);
scores.Add("Ali", 85);
scores.Add("Mehmet", 78);
scores.Add("Ayşe", 92);

// İndeks ile erişim
string firstKey = scores.Keys[0]; // "Ali"
int firstValue = scores.Values[0]; // 85

// Anahtar ile erişim
int mehmetScore = scores["Mehmet"]; // 78

// Güvenli erişim
if (scores.TryGetValue("Emre", out int emreScore))
{
    Console.WriteLine($"Emre'nin puanı: {emreScore}");
}
else
{
    Console.WriteLine("Emre bulunamadı.");
}

// Eleman çıkarma
scores.Remove("Ali");

// Belirli bir indeksteki elemanı çıkarma
scores.RemoveAt(0);

// Anahtarın indeksini bulma
int index = scores.IndexOfKey("Zeynep");

// Değerin indeksini bulma
int valueIndex = scores.IndexOfValue(92);

// Anahtar ve değer kontrolü
bool containsKey = scores.ContainsKey("Ayşe"); // true
bool containsValue = scores.ContainsValue(100); // false
```

### SortedList İterasyonu

```csharp
SortedList<int, string> employees = new SortedList<int, string>
{
    { 103, "Ahmet" },
    { 101, "Fatma" },
    { 105, "Kemal" },
    { 102, "Zeynep" }
};

// KeyValuePair ile döngü (anahtara göre sıralı)
foreach (KeyValuePair<int, string> employee in employees)
{
    Console.WriteLine($"ID: {employee.Key}, İsim: {employee.Value}");
}
// Çıktı:
// ID: 101, İsim: Fatma
// ID: 102, İsim: Zeynep
// ID: 103, İsim: Ahmet
// ID: 105, İsim: Kemal

// Sadece anahtarlar üzerinde döngü
foreach (int id in employees.Keys)
{
    Console.WriteLine(id);
}

// Sadece değerler üzerinde döngü
foreach (string name in employees.Values)
{
    Console.WriteLine(name);
}
```

## 4. Binary Search in SortedList

`SortedList<TKey, TValue>`, anahtarları sıralı tuttuğu için ikili arama (binary search) algoritmasını kullanarak hızlı arama yapabilir.

### İkili Arama Prensibi

```csharp
SortedList<int, string> sortedData = new SortedList<int, string>();
for (int i = 0; i < 1000; i++)
{
    sortedData.Add(i, $"Value-{i}");
}

// IndexOfKey metodu ikili arama kullanır - O(log n)
int keyIndex = sortedData.IndexOfKey(500); // Hızlı arama

// ContainsKey de ikili arama kullanır - O(log n)
bool hasKey = sortedData.ContainsKey(750); // Hızlı arama
```

### Manuel İkili Arama Uygulaması

```csharp
public static int BinarySearch<TKey, TValue>(SortedList<TKey, TValue> list, TKey key) where TKey : IComparable<TKey>
{
    int left = 0;
    int right = list.Count - 1;
    
    while (left <= right)
    {
        int middle = left + (right - left) / 2;
        TKey middleKey = list.Keys[middle];
        
        int comparison = key.CompareTo(middleKey);
        
        if (comparison == 0)
            return middle; // Anahtar bulundu
        
        if (comparison < 0)
            right = middle - 1; // Sol yarıda ara
        else
            left = middle + 1; // Sağ yarıda ara
    }
    
    return ~left; // Anahtar bulunamadı, eklenebilecek konum
}
```

## 5. Custom Sorting Rules

`SortedList<TKey, TValue>`, varsayılan olarak anahtarların doğal sıralamasını kullanır. Ancak, özel sıralama kuralları da tanımlayabilirsiniz.

### IComparer<T> Kullanımı

```csharp
// Özel karşılaştırıcı
public class ReverseComparer<T> : IComparer<T> where T : IComparable<T>
{
    public int Compare(T x, T y)
    {
        // Ters sıralama için karşılaştırma sonucunu tersine çevir
        return y.CompareTo(x);
    }
}

// Özel karşılaştırıcı ile SortedList oluşturma
SortedList<int, string> descendingList = new SortedList<int, string>(new ReverseComparer<int>());

// Eleman ekleme
descendingList.Add(5, "Beş");
descendingList.Add(3, "Üç");
descendingList.Add(8, "Sekiz");
descendingList.Add(1, "Bir");

// Elemanları yazdırma (azalan sırada)
foreach (var item in descendingList)
{
    Console.WriteLine($"{item.Key}: {item.Value}");
}
// Çıktı:
// 8: Sekiz
// 5: Beş
// 3: Üç
// 1: Bir
```

### String Karşılaştırma Özelleştirme

```csharp
// Büyük/küçük harf duyarsız sıralama
SortedList<string, int> caseInsensitiveList = new SortedList<string, int>(StringComparer.OrdinalIgnoreCase);

caseInsensitiveList.Add("apple", 1);
caseInsensitiveList.Add("Banana", 2);
caseInsensitiveList.Add("cherry", 3);
caseInsensitiveList.Add("APPLE", 4); // Hata! "apple" anahtarı zaten var (büyük/küçük harf duyarsız)

// Kültüre özgü sıralama
SortedList<string, int> cultureSensitiveList = new SortedList<string, int>(StringComparer.CurrentCulture);

// Özel string karşılaştırıcı
SortedList<string, int> customStringList = new SortedList<string, int>(
    new Comparison<string>((x, y) => x.Length.CompareTo(y.Length)).ToComparer());

customStringList.Add("a", 1);
customStringList.Add("ccc", 3);
customStringList.Add("bb", 2);
customStringList.Add("dddd", 4);

// Uzunluğa göre sıralanmış anahtarlar: "a", "bb", "ccc", "dddd"
```

### Özel Nesne Sıralaması

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    
    public override string ToString() => $"{Name} ({Age})";
}

// Yaşa göre sıralama
public class PersonAgeComparer : IComparer<Person>
{
    public int Compare(Person x, Person y)
    {
        return x.Age.CompareTo(y.Age);
    }
}

// İsme göre sıralama
public class PersonNameComparer : IComparer<Person>
{
    public int Compare(Person x, Person y)
    {
        return string.Compare(x.Name, y.Name, StringComparison.OrdinalIgnoreCase);
    }
}

// Kullanım
SortedList<Person, string> peopleByAge = new SortedList<Person, string>(new PersonAgeComparer());
SortedList<Person, string> peopleByName = new SortedList<Person, string>(new PersonNameComparer());

var ali = new Person { Name = "Ali", Age = 30 };
var ayse = new Person { Name = "Ayşe", Age = 25 };
var mehmet = new Person { Name = "Mehmet", Age = 35 };

peopleByAge.Add(ali, "Mühendis");
peopleByAge.Add(ayse, "Doktor");
peopleByAge.Add(mehmet, "Öğretmen");

// Yaşa göre sıralı: Ayşe (25), Ali (30), Mehmet (35)
foreach (var person in peopleByAge.Keys)
{
    Console.WriteLine(person);
}
```

## 6. Performance Characteristics

`LinkedList<T>` ve `SortedList<TKey, TValue>` koleksiyonlarının performans özellikleri, kullanım senaryonuza göre avantaj veya dezavantaj olabilir.

### LinkedList<T> Performans Özellikleri

| İşlem | Zaman Karmaşıklığı | Açıklama |
|-------|-------------------|----------|
| Başa/Sona Ekleme | O(1) | Sabit zaman, düğüm referansları güncellenebilir |
| Ortaya Ekleme | O(1) | Düğüm referansı varsa sabit zaman |
| Eleman Arama | O(n) | Doğrusal arama gerektirir |
| Eleman Çıkarma | O(1) | Düğüm referansı varsa sabit zaman |
| Bellek Kullanımı | Yüksek | Her eleman için ek referans bilgisi gerektirir |

```csharp
// LinkedList performans testi
LinkedList<int> linkedList = new LinkedList<int>();
Stopwatch sw = Stopwatch.StartNew();

// Başa ekleme - O(1)
for (int i = 0; i < 100000; i++)
{
    linkedList.AddFirst(i);
}
sw.Stop();
Console.WriteLine($"LinkedList başa ekleme: {sw.ElapsedMilliseconds} ms");

// Ortaya ekleme - Düğüm referansı olmadan O(n)
sw.Restart();
LinkedListNode<int> middle = linkedList.Find(50000);
for (int i = 0; i < 1000; i++)
{
    linkedList.AddAfter(middle, i);
}
sw.Stop();
Console.WriteLine($"LinkedList ortaya ekleme: {sw.ElapsedMilliseconds} ms");

// Arama - O(n)
sw.Restart();
LinkedListNode<int> node = linkedList.Find(75000);
sw.Stop();
Console.WriteLine($"LinkedList arama: {sw.ElapsedMilliseconds} ms");
```

### SortedList<TKey, TValue> Performans Özellikleri

| İşlem | Zaman Karmaşıklığı | Açıklama |
|-------|-------------------|----------|
| Ekleme | O(n) | Sırayı korumak için elemanları kaydırma gerekebilir |
| Anahtar ile Arama | O(log n) | İkili arama kullanır |
| İndeks ile Erişim | O(1) | Diziye doğrudan erişim |
| Çıkarma | O(n) | Elemanları kaydırma gerekebilir |
| Bellek Kullanımı | Orta | İki ayrı dizi kullanır (anahtarlar ve değerler için) |

```csharp
// SortedList performans testi
SortedList<int, string> sortedList = new SortedList<int, string>();
Stopwatch sw = Stopwatch.StartNew();

// Ekleme - O(n)
for (int i = 0; i < 100000; i++)
{
    sortedList.Add(i, $"Value-{i}");
}
sw.Stop();
Console.WriteLine($"SortedList ekleme: {sw.ElapsedMilliseconds} ms");

// Anahtar ile arama - O(log n)
sw.Restart();
bool containsKey = sortedList.ContainsKey(75000);
sw.Stop();
Console.WriteLine($"SortedList anahtar arama: {sw.ElapsedMilliseconds} ms");

// İndeks ile erişim - O(1)
sw.Restart();
string value = sortedList.Values[50000];
sw.Stop();
Console.WriteLine($"SortedList indeks erişimi: {sw.ElapsedMilliseconds} ms");
```

### Karşılaştırmalı Performans

```csharp
// Farklı koleksiyonların karşılaştırması
const int itemCount = 100000;

// LinkedList
LinkedList<int> linkedList = new LinkedList<int>();
Stopwatch sw = Stopwatch.StartNew();
for (int i = 0; i < itemCount; i++)
{
    linkedList.AddLast(i);
}
sw.Stop();
Console.WriteLine($"LinkedList ekleme: {sw.ElapsedMilliseconds} ms");

// List
List<int> list = new List<int>(itemCount);
sw.Restart();
for (int i = 0; i < itemCount; i++)
{
    list.Add(i);
}
sw.Stop();
Console.WriteLine($"List ekleme: {sw.ElapsedMilliseconds} ms");

// SortedList
SortedList<int, int> sortedList = new SortedList<int, int>();
sw.Restart();
for (int i = 0; i < itemCount; i++)
{
    sortedList.Add(i, i);
}
sw.Stop();
Console.WriteLine($"SortedList ekleme: {sw.ElapsedMilliseconds} ms");

// Dictionary
Dictionary<int, int> dictionary = new Dictionary<int, int>(itemCount);
sw.Restart();
for (int i = 0; i < itemCount; i++)
{
    dictionary.Add(i, i);
}
sw.Stop();
Console.WriteLine($"Dictionary ekleme: {sw.ElapsedMilliseconds} ms");

// SortedDictionary
SortedDictionary<int, int> sortedDictionary = new SortedDictionary<int, int>();
sw.Restart();
for (int i = 0; i < itemCount; i++)
{
    sortedDictionary.Add(i, i);
}
sw.Stop();
Console.WriteLine($"SortedDictionary ekleme: {sw.ElapsedMilliseconds} ms");
```

LinkedList ve SortedList, belirli senaryolarda diğer koleksiyonlara göre avantajlar sunar. Doğru veri yapısını seçmek, uygulamanızın performansını ve kullanılabilirliğini önemli ölçüde etkileyebilir. 