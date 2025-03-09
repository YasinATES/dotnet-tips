# Arrays ve Collections Framework

C# programlama dilinde, veri koleksiyonlarını yönetmek için çeşitli veri yapıları bulunmaktadır. Bu yapılar, verileri organize etmek, depolamak ve işlemek için farklı özellikler sunar. Bu bölümde, diziler (arrays) ve .NET Collections Framework'ün temel bileşenlerini inceleyeceğiz.

## 1. Arrays (Diziler)

Diziler, aynı türdeki verileri sabit boyutlu bir yapıda depolamak için kullanılır. C#'ta diziler referans türleridir ve `System.Array` sınıfından türetilir.

### Single-Dimensional Arrays (Tek Boyutlu Diziler)

Tek boyutlu diziler, en basit dizi türüdür ve verileri doğrusal bir sırada depolar.

```csharp
// Tek boyutlu dizi tanımlama
int[] numbers = new int[5]; // 5 elemanlı bir int dizisi

// Dizi elemanlarına değer atama
numbers[0] = 10;
numbers[1] = 20;
numbers[2] = 30;
numbers[3] = 40;
numbers[4] = 50;

// Dizi başlatma (initialization)
int[] numbers2 = new int[] { 10, 20, 30, 40, 50 };
// veya daha kısa şekilde
int[] numbers3 = { 10, 20, 30, 40, 50 };

// Dizi elemanlarına erişim
Console.WriteLine(numbers[2]); // 30 yazdırır

// Dizi üzerinde döngü
foreach (int number in numbers)
{
    Console.WriteLine(number);
}
```

### Multi-Dimensional Arrays (Çok Boyutlu Diziler)

Çok boyutlu diziler, verileri matris formunda depolamak için kullanılır. C#'ta iki tür çok boyutlu dizi vardır: rectangular (dikdörtgen) ve jagged (düzensiz) diziler.

#### Rectangular Arrays (Dikdörtgen Diziler)

Dikdörtgen dizilerde, her bir boyut aynı uzunluktadır.

```csharp
// 2 boyutlu dikdörtgen dizi tanımlama (3x4 matris)
int[,] matrix = new int[3, 4];

// Değer atama
matrix[0, 0] = 1;
matrix[0, 1] = 2;
matrix[1, 0] = 3;
matrix[2, 2] = 4;

// Dizi başlatma
int[,] matrix2 = new int[,] {
    { 1, 2, 3, 4 },
    { 5, 6, 7, 8 },
    { 9, 10, 11, 12 }
};

// Elemanlara erişim
Console.WriteLine(matrix2[1, 2]); // 7 yazdırır

// Çok boyutlu dizi üzerinde döngü
for (int i = 0; i < matrix2.GetLength(0); i++)
{
    for (int j = 0; j < matrix2.GetLength(1); j++)
    {
        Console.Write($"{matrix2[i, j]} ");
    }
    Console.WriteLine();
}
```

#### Jagged Arrays (Düzensiz Diziler)

Düzensiz diziler, "dizi dizileri" olarak düşünülebilir. Her bir alt dizi farklı uzunlukta olabilir.

```csharp
// Jagged dizi tanımlama
int[][] jaggedArray = new int[3][];

// Alt dizileri başlatma
jaggedArray[0] = new int[] { 1, 2, 3 };
jaggedArray[1] = new int[] { 4, 5 };
jaggedArray[2] = new int[] { 6, 7, 8, 9 };

// Alternatif başlatma
int[][] jaggedArray2 = new int[][] {
    new int[] { 1, 2, 3 },
    new int[] { 4, 5 },
    new int[] { 6, 7, 8, 9 }
};

// Elemanlara erişim
Console.WriteLine(jaggedArray[2][1]); // 7 yazdırır

// Jagged dizi üzerinde döngü
for (int i = 0; i < jaggedArray.Length; i++)
{
    for (int j = 0; j < jaggedArray[i].Length; j++)
    {
        Console.Write($"{jaggedArray[i][j]} ");
    }
    Console.WriteLine();
}
```

### Array Methods ve Properties

`System.Array` sınıfı, dizilerle çalışmak için çeşitli metotlar ve özellikler sağlar:

```csharp
int[] numbers = { 5, 2, 8, 1, 9 };

// Dizi özellikleri
Console.WriteLine($"Uzunluk: {numbers.Length}"); // 5
Console.WriteLine($"Rank (Boyut sayısı): {numbers.Rank}"); // 1

// Dizi metotları
Array.Sort(numbers); // Diziyi sıralar: { 1, 2, 5, 8, 9 }
Array.Reverse(numbers); // Diziyi tersine çevirir: { 9, 8, 5, 2, 1 }

int index = Array.IndexOf(numbers, 5); // 5 değerinin indeksini bulur: 2
Array.Clear(numbers, 0, 2); // İlk 2 elemanı temizler: { 0, 0, 5, 2, 1 }

int[] copy = new int[5];
Array.Copy(numbers, copy, 5); // numbers dizisini copy dizisine kopyalar

bool exists = Array.Exists(numbers, x => x > 5); // 5'ten büyük eleman var mı: true
int find = Array.Find(numbers, x => x > 5); // 5'ten büyük ilk elemanı bulur: 8
int[] findAll = Array.FindAll(numbers, x => x > 5); // 5'ten büyük tüm elemanları bulur: { 8, 9 }
```

## 2. Collections Framework

.NET Collections Framework, farklı veri yapıları ve koleksiyon türleri sağlayan kapsamlı bir kütüphanedir. Bu koleksiyonlar, `System.Collections`, `System.Collections.Generic` ve `System.Collections.Concurrent` namespace'lerinde bulunur.

### Collection Interface Hierarchy

.NET Collections Framework, aşağıdaki temel arayüzlere dayanır:

1. **IEnumerable<T>**: Koleksiyonun üzerinde döngü yapılmasını sağlar.
2. **ICollection<T>**: Koleksiyona eleman ekleme, çıkarma ve sayma işlemlerini sağlar.
3. **IList<T>**: İndeks tabanlı erişim ve manipülasyon sağlar.
4. **IDictionary<TKey, TValue>**: Anahtar-değer çiftleri için erişim sağlar.
5. **ISet<T>**: Küme işlemleri için metotlar sağlar.

```csharp
// Interface hiyerarşisi örneği
IEnumerable<int> enumerable = new List<int> { 1, 2, 3 };
ICollection<int> collection = new List<int> { 1, 2, 3 };
IList<int> list = new List<int> { 1, 2, 3 };
IDictionary<string, int> dictionary = new Dictionary<string, int> { { "one", 1 }, { "two", 2 } };
ISet<int> set = new HashSet<int> { 1, 2, 3 };
```

### Yaygın Koleksiyon Türleri

#### List<T>

`List<T>`, dinamik boyutlu bir dizi gibi davranır ve en yaygın kullanılan koleksiyon türüdür.

```csharp
// List oluşturma
List<string> names = new List<string>();

// Eleman ekleme
names.Add("Ali");
names.Add("Ayşe");
names.Add("Mehmet");

// Belirli bir indekse eleman ekleme
names.Insert(1, "Zeynep"); // { "Ali", "Zeynep", "Ayşe", "Mehmet" }

// Eleman çıkarma
names.Remove("Ayşe"); // { "Ali", "Zeynep", "Mehmet" }
names.RemoveAt(0); // { "Zeynep", "Mehmet" }

// Eleman arama
bool contains = names.Contains("Zeynep"); // true
int index = names.IndexOf("Mehmet"); // 1

// Listeyi temizleme
names.Clear(); // { }

// Kapasiteyi ayarlama
names.Capacity = 100;

// Listeyi başlatma
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
```

#### Dictionary<TKey, TValue>

`Dictionary<TKey, TValue>`, anahtar-değer çiftlerini depolamak için kullanılır ve anahtarlar benzersiz olmalıdır.

```csharp
// Dictionary oluşturma
Dictionary<string, int> ages = new Dictionary<string, int>();

// Eleman ekleme
ages.Add("Ali", 30);
ages.Add("Ayşe", 25);
ages["Mehmet"] = 35; // Alternatif ekleme yöntemi

// Değer güncelleme
ages["Ali"] = 31;

// Eleman çıkarma
ages.Remove("Ayşe");

// Anahtar kontrolü
bool containsKey = ages.ContainsKey("Ali"); // true

// Değer kontrolü
bool containsValue = ages.ContainsValue(35); // true

// Anahtarları ve değerleri alma
ICollection<string> keys = ages.Keys;
ICollection<int> values = ages.Values;

// Dictionary başlatma
Dictionary<string, string> capitals = new Dictionary<string, string>
{
    { "Türkiye", "Ankara" },
    { "Fransa", "Paris" },
    { "İtalya", "Roma" }
};
```

#### HashSet<T>

`HashSet<T>`, benzersiz elemanları depolamak için kullanılır ve küme işlemlerini destekler.

```csharp
// HashSet oluşturma
HashSet<int> set1 = new HashSet<int>();

// Eleman ekleme
set1.Add(1);
set1.Add(2);
set1.Add(3);
set1.Add(1); // Tekrar eden eleman eklenmez

// Eleman çıkarma
set1.Remove(2);

// Eleman kontrolü
bool contains = set1.Contains(3); // true

// Küme işlemleri
HashSet<int> set2 = new HashSet<int> { 3, 4, 5 };

set1.UnionWith(set2); // Birleşim: { 1, 3, 4, 5 }
set1.IntersectWith(set2); // Kesişim: { 3 }
set1.ExceptWith(set2); // Fark: { 1 }
set1.SymmetricExceptWith(set2); // Simetrik fark: { 1, 4, 5 }

// HashSet başlatma
HashSet<string> fruits = new HashSet<string> { "Elma", "Armut", "Muz" };
```

#### Queue<T>

`Queue<T>`, FIFO (First-In-First-Out) mantığıyla çalışan bir kuyruk veri yapısıdır.

```csharp
// Queue oluşturma
Queue<string> queue = new Queue<string>();

// Eleman ekleme (enqueue)
queue.Enqueue("İlk");
queue.Enqueue("İkinci");
queue.Enqueue("Üçüncü");

// Eleman çıkarma (dequeue)
string first = queue.Dequeue(); // "İlk"

// Sıradaki elemanı görüntüleme (peek)
string next = queue.Peek(); // "İkinci"

// Queue başlatma
Queue<int> numbers = new Queue<int>(new[] { 1, 2, 3, 4, 5 });
```

#### Stack<T>

`Stack<T>`, LIFO (Last-In-First-Out) mantığıyla çalışan bir yığın veri yapısıdır.

```csharp
// Stack oluşturma
Stack<string> stack = new Stack<string>();

// Eleman ekleme (push)
stack.Push("İlk");
stack.Push("İkinci");
stack.Push("Üçüncü");

// Eleman çıkarma (pop)
string last = stack.Pop(); // "Üçüncü"

// Üstteki elemanı görüntüleme (peek)
string top = stack.Peek(); // "İkinci"

// Stack başlatma
Stack<int> numbers = new Stack<int>(new[] { 1, 2, 3, 4, 5 });
```

### Collection Initialization

C#, koleksiyonları başlatmak için çeşitli sözdizimi seçenekleri sunar:

```csharp
// Liste başlatma
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

// Dictionary başlatma
Dictionary<string, int> ages = new Dictionary<string, int>
{
    { "Ali", 30 },
    { "Ayşe", 25 },
    { "Mehmet", 35 }
};

// C# 6.0+ ile Dictionary başlatma
Dictionary<string, int> ages2 = new Dictionary<string, int>
{
    ["Ali"] = 30,
    ["Ayşe"] = 25,
    ["Mehmet"] = 35
};

// HashSet başlatma
HashSet<string> fruits = new HashSet<string> { "Elma", "Armut", "Muz" };

// C# 9.0+ ile yeni başlatma sözdizimi
List<int> numbers2 = new() { 1, 2, 3, 4, 5 };
Dictionary<string, int> ages3 = new() { ["Ali"] = 30, ["Ayşe"] = 25 };
```

## 3. Collection Performance Comparison

Farklı koleksiyon türleri, farklı operasyonlar için farklı performans özellikleri gösterir. Aşağıdaki tablo, yaygın koleksiyon türlerinin temel operasyonlar için zaman karmaşıklığını gösterir:

| Koleksiyon Türü | Erişim | Arama | Ekleme | Silme |
|-----------------|--------|-------|--------|-------|
| Array           | O(1)   | O(n)  | N/A    | N/A   |
| List<T>         | O(1)   | O(n)  | O(1)*  | O(n)  |
| Dictionary<K,V> | O(1)   | O(1)  | O(1)   | O(1)  |
| HashSet<T>      | N/A    | O(1)  | O(1)   | O(1)  |
| LinkedList<T>   | O(n)   | O(n)  | O(1)** | O(1)**|
| Queue<T>        | O(1)*** | O(n) | O(1)   | O(1)  |
| Stack<T>        | O(1)*** | O(n) | O(1)   | O(1)  |

\* Listenin sonuna ekleme O(1), ortasına ekleme O(n)
\** Referans varsa
\*** Sadece ilk/son eleman

```csharp
// Performans karşılaştırması örneği
const int itemCount = 1000000;

// List<T> ile arama
List<int> list = Enumerable.Range(0, itemCount).ToList();
var listStopwatch = Stopwatch.StartNew();
bool listContains = list.Contains(itemCount - 1);
listStopwatch.Stop();
Console.WriteLine($"List<T> arama süresi: {listStopwatch.ElapsedMilliseconds} ms");

// HashSet<T> ile arama
HashSet<int> hashSet = new HashSet<int>(Enumerable.Range(0, itemCount));
var hashSetStopwatch = Stopwatch.StartNew();
bool hashSetContains = hashSet.Contains(itemCount - 1);
hashSetStopwatch.Stop();
Console.WriteLine($"HashSet<T> arama süresi: {hashSetStopwatch.ElapsedMilliseconds} ms");

// Dictionary<K,V> ile arama
Dictionary<int, int> dictionary = Enumerable.Range(0, itemCount).ToDictionary(x => x, x => x);
var dictionaryStopwatch = Stopwatch.StartNew();
bool dictionaryContains = dictionary.ContainsKey(itemCount - 1);
dictionaryStopwatch.Stop();
Console.WriteLine($"Dictionary<K,V> arama süresi: {dictionaryStopwatch.ElapsedMilliseconds} ms");
```

## 4. En İyi Pratikler

1. **Doğru Koleksiyon Türünü Seçin**
   - Sık erişim ve arama gerektiren işlemler için `Dictionary<TKey, TValue>` veya `HashSet<T>` kullanın
   - Sıralı veriler için `List<T>` veya `SortedDictionary<TKey, TValue>` kullanın
   - FIFO işlemleri için `Queue<T>`, LIFO işlemleri için `Stack<T>` kullanın

2. **Koleksiyon Kapasitesini Önceden Belirleyin**
   - Eleman sayısı biliniyorsa, koleksiyonun kapasitesini başlangıçta ayarlayın
   ```csharp
   List<int> numbers = new List<int>(10000); // 10000 elemanlık kapasite
   ```

3. **Uygun Döngü Yöntemini Kullanın**
   - Sadece okuma için `foreach` kullanın
   - Değiştirme işlemleri için `for` veya `while` kullanın
   - LINQ sorgularını gereksiz yere tekrarlamaktan kaçının

4. **Koleksiyonları Değiştirirken Dikkatli Olun**
   - Bir koleksiyon üzerinde döngü yaparken, koleksiyonu değiştirmekten kaçının
   - Değiştirme gerekiyorsa, koleksiyonun bir kopyasını oluşturun veya tersten döngü kullanın

5. **Generic Koleksiyonları Tercih Edin**
   - Non-generic koleksiyonlar yerine generic koleksiyonları kullanın
   - Generic koleksiyonlar tip güvenliği sağlar ve boxing/unboxing işlemlerini önler

## 5. Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Yanlış Koleksiyon Türü Seçimi**
   ```csharp
   // Kötü - Sık arama gerektiren işlemler için List kullanımı
   List<Customer> customers = new List<Customer>();
   // Müşteri arama
   Customer customer = customers.FirstOrDefault(c => c.Id == customerId); // O(n)
   
   // İyi - Sık arama için Dictionary kullanımı
   Dictionary<int, Customer> customerDict = new Dictionary<int, Customer>();
   // Müşteri arama
   Customer customer = customerDict.TryGetValue(customerId, out var c) ? c : null; // O(1)
   ```

2. **Döngü Sırasında Koleksiyon Değiştirme**
   ```csharp
   // Kötü - Döngü sırasında koleksiyonu değiştirme
   foreach (var item in items)
   {
       if (item.ShouldRemove)
           items.Remove(item); // InvalidOperationException fırlatır
   }
   
   // İyi - Kaldırılacak öğeleri ayrı bir listede topla
   var itemsToRemove = items.Where(item => item.ShouldRemove).ToList();
   foreach (var item in itemsToRemove)
   {
       items.Remove(item);
   }
   
   // Veya daha iyi - RemoveAll kullan
   items.RemoveAll(item => item.ShouldRemove);
   ```

3. **Gereksiz Koleksiyon Kopyalama**
   ```csharp
   // Kötü - Gereksiz kopyalama
   List<int> numbers = GetNumbers();
   List<int> copy = new List<int>(numbers); // Gereksiz kopyalama
   foreach (var number in copy)
   {
       Console.WriteLine(number);
   }
   
   // İyi - Doğrudan orijinal koleksiyonu kullan
   List<int> numbers = GetNumbers();
   foreach (var number in numbers)
   {
       Console.WriteLine(number);
   }
   ```

4. **Koleksiyon Kapasitesini Göz Ardı Etme**
   ```csharp
   // Kötü - Kapasite belirtilmemiş
   List<int> numbers = new List<int>();
   for (int i = 0; i < 10000; i++)
   {
       numbers.Add(i); // Birçok kapasite artırımı gerektirir
   }
   
   // İyi - Kapasite önceden belirtilmiş
   List<int> numbers = new List<int>(10000);
   for (int i = 0; i < 10000; i++)
   {
       numbers.Add(i); // Kapasite artırımı gerekmez
   }
   ```

5. **Non-Generic Koleksiyonları Kullanma**
   ```csharp
   // Kötü - Non-generic koleksiyon
   ArrayList list = new ArrayList();
   list.Add(1);
   list.Add("string"); // Tip güvenliği yok
   int number = (int)list[0]; // Unboxing gerektirir
   
   // İyi - Generic koleksiyon
   List<int> list = new List<int>();
   list.Add(1);
   // list.Add("string"); // Derleme hatası - tip güvenliği
   int number = list[0]; // Unboxing gerekmez
   ```

Arrays ve Collections Framework, C# programlama dilinde veri yönetiminin temel yapı taşlarıdır. Doğru koleksiyon türünü seçmek ve uygun şekilde kullanmak, uygulamanızın performansını ve bakım kolaylığını önemli ölçüde etkileyebilir. 