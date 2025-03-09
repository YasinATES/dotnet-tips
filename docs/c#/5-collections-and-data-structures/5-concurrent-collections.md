# Concurrent Collections

Bu bölümde, çok iş parçacıklı (multi-threaded) uygulamalarda güvenli bir şekilde kullanılabilen eşzamanlı koleksiyonları (concurrent collections) inceleyeceğiz. Bu koleksiyonlar, `System.Collections.Concurrent` namespace'inde bulunur ve thread-safe operasyonlar sağlar.

## 1. ConcurrentDictionary<TKey, TValue>

`ConcurrentDictionary<TKey, TValue>`, çoklu iş parçacıklarının aynı anda okuma ve yazma işlemleri yapabildiği thread-safe bir sözlük uygulamasıdır. Standart `Dictionary<TKey, TValue>`'den farklı olarak, eşzamanlı erişim için optimize edilmiştir.

### Temel Operasyonlar

```csharp
// ConcurrentDictionary oluşturma
ConcurrentDictionary<string, int> concurrentDict = new ConcurrentDictionary<string, int>();

// Eleman ekleme
bool added = concurrentDict.TryAdd("one", 1);
concurrentDict["two"] = 2; // İndeksleyici kullanımı (yoksa ekler, varsa günceller)

// GetOrAdd - Anahtar varsa değerini döndürür, yoksa ekler ve döndürür
int three = concurrentDict.GetOrAdd("three", 3);
int four = concurrentDict.GetOrAdd("four", key => key.Length); // Değer üretmek için fonksiyon

// AddOrUpdate - Anahtar varsa günceller, yoksa ekler
int five = concurrentDict.AddOrUpdate("five", 5, (key, oldValue) => oldValue + 1);
int two2 = concurrentDict.AddOrUpdate("two", 20, (key, oldValue) => oldValue * 10); // "two" 20 olur

// Güvenli değer alma
if (concurrentDict.TryGetValue("one", out int oneValue))
{
    Console.WriteLine($"one: {oneValue}");
}

// Güvenli silme
bool removed = concurrentDict.TryRemove("two", out int removedValue);

// Güvenli güncelleme
bool updated = concurrentDict.TryUpdate("three", 33, 3); // Mevcut değer 3 ise 33 olarak güncelle
```

### Atomik Operasyonlar

`ConcurrentDictionary<TKey, TValue>`, atomik (bölünemez) operasyonlar sağlar. Bu, bir operasyonun başka bir iş parçacığı tarafından kesintiye uğratılamayacağı anlamına gelir.

```csharp
ConcurrentDictionary<string, int> counter = new ConcurrentDictionary<string, int>();

// Paralel olarak sayaçları artırma
Parallel.For(0, 1000, i =>
{
    string key = (i % 10).ToString();
    
    // Atomik artırma
    counter.AddOrUpdate(key, 1, (k, v) => v + 1);
    
    // Alternatif atomik artırma
    counter[key] = counter.GetOrAdd(key, 0) + 1; // Bu atomik DEĞİL!
});

// Doğru atomik artırma
Parallel.For(0, 1000, i =>
{
    string key = (i % 10).ToString();
    counter.AddOrUpdate(key, 1, (k, v) => v + 1);
});

// Daha kısa atomik artırma (.NET 6+)
Parallel.For(0, 1000, i =>
{
    string key = (i % 10).ToString();
    counter.AddOrUpdate(key, 1, (k, v) => v + 1);
});
```

### Performans Özellikleri

`ConcurrentDictionary<TKey, TValue>`, dahili olarak fine-grained locking mekanizması kullanır. Bu, tüm sözlüğü kilitlemek yerine, sadece etkilenen bölümleri (buckets) kilitler.

```csharp
// Performans karşılaştırması
const int itemCount = 1000000;
const int threadCount = 8;

// ConcurrentDictionary ile çoklu iş parçacığı testi
ConcurrentDictionary<int, int> concurrentDict = new ConcurrentDictionary<int, int>();
Stopwatch sw = Stopwatch.StartNew();

Parallel.For(0, itemCount, new ParallelOptions { MaxDegreeOfParallelism = threadCount }, i =>
{
    concurrentDict[i] = i;
});

sw.Stop();
Console.WriteLine($"ConcurrentDictionary ekleme: {sw.ElapsedMilliseconds} ms");

// Dictionary ile tek iş parçacığı testi
Dictionary<int, int> dict = new Dictionary<int, int>();
sw.Restart();

for (int i = 0; i < itemCount; i++)
{
    dict[i] = i;
}

sw.Stop();
Console.WriteLine($"Dictionary ekleme: {sw.ElapsedMilliseconds} ms");

// Dictionary ile çoklu iş parçacığı testi (lock kullanarak)
Dictionary<int, int> lockDict = new Dictionary<int, int>();
object lockObj = new object();
sw.Restart();

Parallel.For(0, itemCount, new ParallelOptions { MaxDegreeOfParallelism = threadCount }, i =>
{
    lock (lockObj)
    {
        lockDict[i] = i;
    }
});

sw.Stop();
Console.WriteLine($"Dictionary (lock ile) ekleme: {sw.ElapsedMilliseconds} ms");
```

## 2. ConcurrentQueue<T>

`ConcurrentQueue<T>`, çoklu iş parçacıkları tarafından güvenli bir şekilde kullanılabilen FIFO (First-In-First-Out) veri yapısıdır.

### Temel Operasyonlar

```csharp
// ConcurrentQueue oluşturma
ConcurrentQueue<string> messageQueue = new ConcurrentQueue<string>();

// Kuyruğa eleman ekleme
messageQueue.Enqueue("Mesaj 1");
messageQueue.Enqueue("Mesaj 2");
messageQueue.Enqueue("Mesaj 3");

// Kuyruktan eleman çıkarma (güvenli)
if (messageQueue.TryDequeue(out string message))
{
    Console.WriteLine($"İşlenen mesaj: {message}"); // "Mesaj 1"
}

// Kuyruğun başındaki elemanı görüntüleme (güvenli)
if (messageQueue.TryPeek(out string nextMessage))
{
    Console.WriteLine($"Sıradaki mesaj: {nextMessage}"); // "Mesaj 2"
}

// Kuyruk boş mu kontrolü
bool isEmpty = messageQueue.IsEmpty;

// Kuyruk boyutu
int count = messageQueue.Count;

// Kuyruğu diziye dönüştürme
string[] messages = messageQueue.ToArray(); // Anlık bir görüntü (snapshot) alır

// Kuyruğu temizleme
messageQueue.Clear();
```

### Üretici-Tüketici Senaryosu

```csharp
ConcurrentQueue<int> taskQueue = new ConcurrentQueue<int>();
CancellationTokenSource cts = new CancellationTokenSource();
CancellationToken token = cts.Token;

// Üretici görevi
Task producer = Task.Run(() =>
{
    for (int i = 0; i < 100; i++)
    {
        taskQueue.Enqueue(i);
        Console.WriteLine($"Üretildi: {i}");
        Thread.Sleep(10); // Üretim hızını simüle et
        
        if (token.IsCancellationRequested)
            break;
    }
});

// Tüketici görevleri
Task[] consumers = new Task[3];
for (int i = 0; i < consumers.Length; i++)
{
    int consumerId = i;
    consumers[i] = Task.Run(() =>
    {
        while (!token.IsCancellationRequested || !taskQueue.IsEmpty)
        {
            if (taskQueue.TryDequeue(out int item))
            {
                Console.WriteLine($"Tüketici {consumerId}: {item}");
                Thread.Sleep(30); // İşlem süresini simüle et
            }
            else
            {
                Thread.Sleep(1); // Kuyruk boşsa kısa bir süre bekle
            }
        }
    });
}

// Ana iş parçacığında bekle
Thread.Sleep(2000);
cts.Cancel(); // İşlemi sonlandır
Task.WaitAll(producer);
Task.WaitAll(consumers);
```

## 3. ConcurrentStack<T>

`ConcurrentStack<T>`, çoklu iş parçacıkları tarafından güvenli bir şekilde kullanılabilen LIFO (Last-In-First-Out) veri yapısıdır.

### Temel Operasyonlar

```csharp
// ConcurrentStack oluşturma
ConcurrentStack<int> stack = new ConcurrentStack<int>();

// Yığına eleman ekleme
stack.Push(10);
stack.Push(20);
stack.Push(30);

// Çoklu eleman ekleme
stack.PushRange(new[] { 40, 50, 60 });

// Yığından eleman çıkarma (güvenli)
if (stack.TryPop(out int topItem))
{
    Console.WriteLine($"Çıkarılan eleman: {topItem}"); // 60
}

// Yığının üstündeki elemanı görüntüleme (güvenli)
if (stack.TryPeek(out int peekItem))
{
    Console.WriteLine($"Üstteki eleman: {peekItem}"); // 50
}

// Çoklu eleman çıkarma
int[] items = new int[3];
int popped = stack.TryPopRange(items); // Kaç eleman çıkarıldığını döndürür
Console.WriteLine($"Çıkarılan eleman sayısı: {popped}");

// Yığın boş mu kontrolü
bool isEmpty = stack.IsEmpty;

// Yığın boyutu
int count = stack.Count;

// Yığını diziye dönüştürme
int[] allItems = stack.ToArray(); // Anlık bir görüntü (snapshot) alır
```

### Paralel İşlem Takibi

```csharp
ConcurrentStack<string> operationStack = new ConcurrentStack<string>();

// Paralel işlemleri izleme
Parallel.For(0, 100, i =>
{
    string operation = $"İşlem {i} başladı";
    operationStack.Push(operation);
    
    // İşlem simülasyonu
    Thread.Sleep(Random.Shared.Next(10, 50));
    
    operation = $"İşlem {i} tamamlandı";
    operationStack.Push(operation);
});

// Tüm işlem kayıtlarını görüntüleme (LIFO sırasında)
foreach (string log in operationStack)
{
    Console.WriteLine(log);
}
```

## 4. ConcurrentBag<T>

`ConcurrentBag<T>`, çoklu iş parçacıkları tarafından güvenli bir şekilde kullanılabilen, sırasız bir koleksiyondur. Özellikle aynı iş parçacığının eklediği öğeleri çıkarma konusunda optimize edilmiştir.

### Temel Operasyonlar

```csharp
// ConcurrentBag oluşturma
ConcurrentBag<string> bag = new ConcurrentBag<string>();

// Eleman ekleme
bag.Add("Eleman 1");
bag.Add("Eleman 2");
bag.Add("Eleman 3");

// Eleman çıkarma (güvenli)
if (bag.TryTake(out string item))
{
    Console.WriteLine($"Çıkarılan eleman: {item}"); // Sıra garantisi yok
}

// Eleman görüntüleme (güvenli)
if (bag.TryPeek(out string peekItem))
{
    Console.WriteLine($"Görüntülenen eleman: {peekItem}"); // Sıra garantisi yok
}

// Bag boş mu kontrolü
bool isEmpty = bag.IsEmpty;

// Bag boyutu
int count = bag.Count;

// Bag'i diziye dönüştürme
string[] allItems = bag.ToArray(); // Anlık bir görüntü (snapshot) alır
```

### İş Parçacığı Yerel Optimizasyon

`ConcurrentBag<T>`, her iş parçacığı için yerel bir depolama alanı kullanır. Bu, aynı iş parçacığının eklediği öğeleri çıkarırken performansı artırır.

```csharp
ConcurrentBag<int> workItems = new ConcurrentBag<int>();

// Paralel iş yükü
Parallel.For(0, 10, i =>
{
    // Her iş parçacığı kendi öğelerini ekler
    for (int j = 0; j < 1000; j++)
    {
        int item = i * 1000 + j;
        workItems.Add(item);
    }
    
    // Aynı iş parçacığı kendi eklediği öğeleri işler (optimize edilmiş)
    int processedCount = 0;
    while (processedCount < 1000 && workItems.TryTake(out int workItem))
    {
        // İş öğesini işle
        processedCount++;
    }
    
    Console.WriteLine($"İş parçacığı {i}: {processedCount} öğe işlendi");
});
```

## 5. Thread-Safe Operations

Eşzamanlı koleksiyonlar, çoklu iş parçacıklarının aynı koleksiyona güvenli bir şekilde erişmesini sağlar. Bu, veri bozulmasını ve race condition'ları önler.

### Atomik Operasyonlar

Atomik operasyonlar, bölünemez işlemlerdir ve diğer iş parçacıkları tarafından kesintiye uğratılamazlar.

```csharp
ConcurrentDictionary<string, long> counters = new ConcurrentDictionary<string, long>();

// Atomik artırma
long newValue = counters.AddOrUpdate("visits", 1, (key, oldValue) => oldValue + 1);

// Atomik değer güncelleme
counters.AddOrUpdate("lastVisit", DateTime.Now.Ticks, (key, oldValue) => DateTime.Now.Ticks);

// Atomik koşullu güncelleme
bool updated = counters.TryUpdate("status", 1, 0); // Mevcut değer 0 ise 1 olarak güncelle
```

### Koleksiyon Değiştirme Sırasında İterasyon

Eşzamanlı koleksiyonlar, iterasyon sırasında değiştirildiğinde `ConcurrentModificationException` fırlatmaz. Bunun yerine, iterasyon başladığında koleksiyonun bir anlık görüntüsünü (snapshot) alır veya değişiklikleri güvenli bir şekilde yönetir.

```csharp
ConcurrentBag<int> numbers = new ConcurrentBag<int>();
for (int i = 0; i < 10; i++)
{
    numbers.Add(i);
}

// Iterasyon sırasında ekleme
Task addTask = Task.Run(() =>
{
    for (int i = 10; i < 20; i++)
    {
        numbers.Add(i);
        Thread.Sleep(10);
    }
});

// Aynı anda iterasyon
foreach (int number in numbers)
{
    Console.WriteLine($"Okunan: {number}");
    Thread.Sleep(15);
}

addTask.Wait();
Console.WriteLine($"Son eleman sayısı: {numbers.Count}");
```

### BlockingCollection<T> Kullanımı

`BlockingCollection<T>`, üretici-tüketici senaryoları için daha yüksek seviyeli bir API sağlar. Dahili olarak bir eşzamanlı koleksiyon kullanır ve bloke edici operasyonlar ekler.

```csharp
// Sınırlı kapasiteli BlockingCollection
using BlockingCollection<int> blockingQueue = new BlockingCollection<int>(boundedCapacity: 100);

// Üretici görevi
Task producer = Task.Run(() =>
{
    for (int i = 0; i < 200; i++)
    {
        blockingQueue.Add(i); // Kuyruk doluysa bloklar
        Console.WriteLine($"Üretildi: {i}");
        Thread.Sleep(10);
    }
    blockingQueue.CompleteAdding(); // Ekleme işleminin bittiğini bildir
});

// Tüketici görevleri
Task[] consumers = new Task[3];
for (int i = 0; i < consumers.Length; i++)
{
    int consumerId = i;
    consumers[i] = Task.Run(() =>
    {
        foreach (int item in blockingQueue.GetConsumingEnumerable())
        {
            Console.WriteLine($"Tüketici {consumerId}: {item}");
            Thread.Sleep(30);
        }
    });
}

// Tüm görevlerin tamamlanmasını bekle
Task.WaitAll(producer);
Task.WaitAll(consumers);
```

## 6. Lock-Free Algorithms

Eşzamanlı koleksiyonlar, geleneksel kilitleme mekanizmalarına göre daha verimli olan lock-free veya lock-minimal algoritmalar kullanır. Bu algoritmalar, Interlocked sınıfı ve Compare-and-Swap (CAS) operasyonları gibi düşük seviyeli atomik işlemlere dayanır.

### Interlocked Operasyonları

```csharp
// Atomik artırma/azaltma
long counter = 0;
Interlocked.Increment(ref counter); // counter += 1
Interlocked.Decrement(ref counter); // counter -= 1
Interlocked.Add(ref counter, 5); // counter += 5

// Atomik değer değiştirme
long oldValue = Interlocked.Exchange(ref counter, 10); // counter = 10, eski değeri döndürür

// Atomik koşullu değiştirme
long comparand = 10;
long newValue = 20;
long result = Interlocked.CompareExchange(ref counter, newValue, comparand);
// Eğer counter == comparand ise counter = newValue olur, eski değeri döndürür
```

### SpinWait Kullanımı

`SpinWait`, kısa süreli beklemelerde CPU'yu daha verimli kullanmak için kullanılır.

```csharp
ConcurrentQueue<int> queue = new ConcurrentQueue<int>();
SpinWait spinWait = new SpinWait();

// Üretici
Task.Run(() =>
{
    for (int i = 0; i < 100; i++)
    {
        queue.Enqueue(i);
        Thread.Sleep(10);
    }
});

// Tüketici (SpinWait kullanarak)
while (true)
{
    if (queue.TryDequeue(out int item))
    {
        Console.WriteLine($"İşlenen: {item}");
        spinWait.Reset();
    }
    else
    {
        spinWait.SpinOnce(); // Verimli bekleme
        
        if (spinWait.NextSpinWillYield)
        {
            // Çok uzun süre bekledik, çıkış yap veya başka bir şey yap
            break;
        }
    }
}
```

### Özel Lock-Free Veri Yapısı Örneği

```csharp
// Basit bir lock-free yığın implementasyonu
public class LockFreeStack<T>
{
    private class Node
    {
        public T Value;
        public Node Next;
        
        public Node(T value)
        {
            Value = value;
            Next = null;
        }
    }
    
    private Node _head;
    
    public void Push(T item)
    {
        Node newNode = new Node(item);
        Node oldHead;
        
        do
        {
            oldHead = _head;
            newNode.Next = oldHead;
        }
        while (Interlocked.CompareExchange(ref _head, newNode, oldHead) != oldHead);
    }
    
    public bool TryPop(out T result)
    {
        Node oldHead;
        Node newHead;
        
        do
        {
            oldHead = _head;
            
            if (oldHead == null)
            {
                result = default;
                return false;
            }
            
            newHead = oldHead.Next;
        }
        while (Interlocked.CompareExchange(ref _head, newHead, oldHead) != oldHead);
        
        result = oldHead.Value;
        return true;
    }
    
    public bool IsEmpty => _head == null;
}
```

Eşzamanlı koleksiyonlar, çoklu iş parçacıklı uygulamalarda veri paylaşımını güvenli ve verimli hale getirir. Doğru koleksiyonu seçmek ve uygun şekilde kullanmak, uygulamanızın performansını ve ölçeklenebilirliğini önemli ölçüde etkileyebilir. 