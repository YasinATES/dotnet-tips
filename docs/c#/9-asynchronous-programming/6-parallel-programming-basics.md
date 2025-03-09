# Parallel Programming Basics (Paralel Programlama Temelleri)

Modern bilgisayarlar çoklu çekirdek işlemcilere sahiptir ve bu işlemcilerin gücünden tam olarak yararlanmak için paralel programlama teknikleri kullanılır. Bu bölümde, C#'ta paralel programlama temellerini inceleyeceğiz.

## 1. Parallel.For/ForEach

`Parallel` sınıfı, döngüleri paralel olarak çalıştırmak için kullanılır.

```csharp
// Parallel.For kullanımı
public void ParallelForExample()
{
    Console.WriteLine("Parallel.For başlatılıyor...");
    
    // 1'den 10'a kadar olan sayıları paralel olarak işleme
    Parallel.For(1, 11, i =>
    {
        Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: {i} işleniyor");
        
        // Simüle edilmiş iş yükü
        Thread.Sleep(100);
        
        Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: {i} tamamlandı");
    });
    
    Console.WriteLine("Parallel.For tamamlandı.");
}

// Parallel.ForEach kullanımı
public void ParallelForEachExample()
{
    // İşlenecek veri kümesi
    var items = Enumerable.Range(1, 10).Select(i => $"Öğe-{i}").ToList();
    
    Console.WriteLine("Parallel.ForEach başlatılıyor...");
    
    // Öğeleri paralel olarak işleme
    Parallel.ForEach(items, item =>
    {
        Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: {item} işleniyor");
        
        // Simüle edilmiş iş yükü
        Thread.Sleep(100);
        
        Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: {item} tamamlandı");
    });
    
    Console.WriteLine("Parallel.ForEach tamamlandı.");
}
```

### ParallelOptions Kullanımı

```csharp
// ParallelOptions kullanımı
public void ParallelOptionsExample()
{
    // Paralel işlem seçenekleri
    var options = new ParallelOptions
    {
        MaxDegreeOfParallelism = Environment.ProcessorCount / 2, // Çekirdek sayısının yarısı kadar paralel işlem
        CancellationToken = CancellationToken.None
    };
    
    Console.WriteLine($"Maksimum paralellik derecesi: {options.MaxDegreeOfParallelism}");
    
    // Sınırlı paralellik ile işlem
    Parallel.For(1, 20, options, i =>
    {
        Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: {i} işleniyor");
        Thread.Sleep(100);
    });
}

// İptal edilebilir paralel işlem
public void CancellableParallelExample()
{
    using var cts = new CancellationTokenSource();
    
    // 2 saniye sonra iptal etme
    cts.CancelAfter(TimeSpan.FromSeconds(2));
    
    var options = new ParallelOptions
    {
        CancellationToken = cts.Token
    };
    
    try
    {
        // Çok sayıda öğeyi paralel olarak işleme (iptal edilebilir)
        Parallel.For(1, 1000, options, i =>
        {
            Console.WriteLine($"Öğe {i} işleniyor...");
            Thread.Sleep(50); // Her öğe için 50ms
            
            // İptal kontrolü (isteğe bağlı, Parallel.For zaten kontrol eder)
            options.CancellationToken.ThrowIfCancellationRequested();
        });
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("Paralel işlem iptal edildi.");
    }
}
```

## 2. PLINQ (Parallel LINQ)

PLINQ, LINQ sorgularını paralel olarak çalıştırmak için kullanılır.

```csharp
// Temel PLINQ kullanımı
public void BasicPlinqExample()
{
    // Büyük bir veri kümesi oluşturma
    var numbers = Enumerable.Range(1, 10000000).ToList();
    
    Console.WriteLine("PLINQ sorgusu başlatılıyor...");
    
    // Paralel LINQ sorgusu
    var evenSquares = numbers
        .AsParallel() // Paralel işleme geçiş
        .Where(n => n % 2 == 0) // Çift sayıları filtreleme
        .Select(n => n * n) // Karesini alma
        .Take(10) // İlk 10 sonucu alma
        .ToList(); // Sonuçları bir listeye dönüştürme
    
    // Sonuçları gösterme
    foreach (var square in evenSquares)
    {
        Console.WriteLine(square);
    }
}

// PLINQ sorgu seçenekleri
public void PlinqOptionsExample()
{
    var numbers = Enumerable.Range(1, 1000000);
    
    // PLINQ sorgu seçenekleri
    var result = numbers
        .AsParallel()
        .WithDegreeOfParallelism(Environment.ProcessorCount) // Paralellik derecesi
        .WithExecutionMode(ParallelExecutionMode.ForceParallelism) // Paralelliği zorlama
        .WithMergeOptions(ParallelMergeOptions.FullyBuffered) // Sonuçları tamamen arabellekleme
        .Where(n => n % 2 == 0)
        .Select(n => n * n)
        .ToList();
    
    Console.WriteLine($"Toplam {result.Count} sonuç bulundu.");
}

// PLINQ istisna işleme
public void PlinqExceptionHandlingExample()
{
    var numbers = Enumerable.Range(1, 100);
    
    try
    {
        // Hata fırlatan bir PLINQ sorgusu
        var result = numbers
            .AsParallel()
            .Select(n => 
            {
                if (n == 50) throw new InvalidOperationException($"Hata: {n}");
                return n * 2;
            })
            .ToList(); // Bu noktada istisna fırlatılacak
    }
    catch (AggregateException ex)
    {
        Console.WriteLine("PLINQ sorgusu sırasında hatalar oluştu:");
        
        foreach (var innerEx in ex.InnerExceptions)
        {
            Console.WriteLine($"- {innerEx.Message}");
        }
    }
}
```

## 3. Task Parallelism (Görev Paralelliği)

Görev paralelliği, birden fazla görevi paralel olarak çalıştırmak için kullanılır.

```csharp
// Task.Run ile paralel görevler
public async Task TaskParallelismExample()
{
    Console.WriteLine("Paralel görevler başlatılıyor...");
    
    // Paralel görevler oluşturma
    Task task1 = Task.Run(() => ProcessData("Veri 1", 2));
    Task task2 = Task.Run(() => ProcessData("Veri 2", 3));
    Task task3 = Task.Run(() => ProcessData("Veri 3", 1));
    
    // Tüm görevlerin tamamlanmasını bekleme
    await Task.WhenAll(task1, task2, task3);
    
    Console.WriteLine("Tüm görevler tamamlandı.");
}

// Veri işleme simülasyonu
private void ProcessData(string data, int seconds)
{
    Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: {data} işleniyor...");
    Thread.Sleep(seconds * 1000); // Simüle edilmiş işlem süresi
    Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: {data} tamamlandı.");
}

// Task.Factory.StartNew ile paralel görevler
public async Task TaskFactoryParallelismExample()
{
    // Görev fabrikası seçenekleri
    var taskOptions = new TaskCreationOptions
    {
        PreferFairness = true,
        LongRunning = false
    };
    
    // Paralel görevler oluşturma
    var tasks = new List<Task>();
    
    for (int i = 1; i <= 5; i++)
    {
        int taskId = i;
        tasks.Add(Task.Factory.StartNew(() =>
        {
            Console.WriteLine($"Görev {taskId} başladı.");
            Thread.Sleep(taskId * 200);
            Console.WriteLine($"Görev {taskId} tamamlandı.");
        }, CancellationToken.None, taskOptions, TaskScheduler.Default));
    }
    
    // Tüm görevlerin tamamlanmasını bekleme
    await Task.WhenAll(tasks);
}
```

## 4. Data Parallelism (Veri Paralelliği)

Veri paralelliği, büyük veri kümelerini paralel olarak işlemek için kullanılır.

```csharp
// Veri paralelliği örneği
public void DataParallelismExample()
{
    // Büyük bir veri kümesi oluşturma
    var data = new double[10000000];
    
    // Veriyi başlatma
    for (int i = 0; i < data.Length; i++)
    {
        data[i] = i;
    }
    
    Console.WriteLine("Seri işlem başlatılıyor...");
    var watch = Stopwatch.StartNew();
    
    // Seri işlem
    for (int i = 0; i < data.Length; i++)
    {
        data[i] = Math.Sqrt(data[i]);
    }
    
    watch.Stop();
    Console.WriteLine($"Seri işlem tamamlandı: {watch.ElapsedMilliseconds}ms");
    
    // Veriyi sıfırlama
    for (int i = 0; i < data.Length; i++)
    {
        data[i] = i;
    }
    
    Console.WriteLine("Paralel işlem başlatılıyor...");
    watch.Restart();
    
    // Paralel işlem
    Parallel.For(0, data.Length, i =>
    {
        data[i] = Math.Sqrt(data[i]);
    });
    
    watch.Stop();
    Console.WriteLine($"Paralel işlem tamamlandı: {watch.ElapsedMilliseconds}ms");
}

// Partition-based parallelism (Bölüm tabanlı paralellik)
public void PartitionBasedParallelismExample()
{
    var items = Enumerable.Range(1, 1000000).ToArray();
    
    Console.WriteLine("Bölüm tabanlı paralellik başlatılıyor...");
    
    // Veriyi bölümlere ayırarak paralel işleme
    Parallel.ForEach(
        Partitioner.Create(0, items.Length),
        range =>
        {
            // Her iş parçacığı bir veri bölümünü işler
            Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: {range.Item1}-{range.Item2} aralığını işliyor");
            
            // Bölüm içindeki tüm öğeleri işleme
            for (int i = range.Item1; i < range.Item2; i++)
            {
                // Öğeyi işleme (burada karesini alıyoruz)
                items[i] = items[i] * items[i];
            }
        });
    
    Console.WriteLine("Bölüm tabanlı paralellik tamamlandı.");
}
```

## 5. Synchronization Primitives (Senkronizasyon İlkelleri)

Paralel programlamada, paylaşılan verilere erişimi senkronize etmek için çeşitli mekanizmalar kullanılır.

```csharp
// Lock kullanımı
public void LockExample()
{
    int counter = 0;
    object lockObj = new object();
    
    // Paralel olarak sayacı artırma
    Parallel.For(0, 1000, _ =>
    {
        // Kritik bölgeyi kilitleme
        lock (lockObj)
        {
            counter++;
        }
    });
    
    Console.WriteLine($"Sayaç değeri: {counter}");
}

// Monitor kullanımı
public void MonitorExample()
{
    int counter = 0;
    object lockObj = new object();
    
    // Paralel olarak sayacı artırma
    Parallel.For(0, 1000, _ =>
    {
        bool lockTaken = false;
        
        try
        {
            // Kilidi alma
            Monitor.Enter(lockObj, ref lockTaken);
            
            // Kritik bölge
            counter++;
        }
        finally
        {
            // Kilidi serbest bırakma
            if (lockTaken)
            {
                Monitor.Exit(lockObj);
            }
        }
    });
    
    Console.WriteLine($"Sayaç değeri: {counter}");
}

// Interlocked kullanımı
public void InterlockedExample()
{
    int counter = 0;
    
    // Paralel olarak sayacı artırma
    Parallel.For(0, 1000, _ =>
    {
        // Atomik artırma
        Interlocked.Increment(ref counter);
    });
    
    Console.WriteLine($"Sayaç değeri: {counter}");
}

// ReaderWriterLockSlim kullanımı
public void ReaderWriterLockExample()
{
    var cache = new Dictionary<int, string>();
    var cacheLock = new ReaderWriterLockSlim();
    
    // Paralel okuma ve yazma işlemleri
    Parallel.For(0, 100, i =>
    {
        if (i % 10 == 0)
        {
            // Yazma işlemi (daha az sıklıkta)
            cacheLock.EnterWriteLock();
            try
            {
                cache[i] = $"Veri-{i}";
                Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: Veri-{i} yazıldı.");
                Thread.Sleep(10);
            }
            finally
            {
                cacheLock.ExitWriteLock();
            }
        }
        else
        {
            // Okuma işlemi (daha sık)
            cacheLock.EnterReadLock();
            try
            {
                if (cache.TryGetValue(i - 1, out string value))
                {
                    Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: {value} okundu.");
                }
                Thread.Sleep(5);
            }
            finally
            {
                cacheLock.ExitReadLock();
            }
        }
    });
}
```

## 6. Thread Pool Management (İş Parçacığı Havuzu Yönetimi)

.NET, iş parçacığı havuzunu kullanarak iş parçacığı oluşturma ve yönetme maliyetlerini azaltır.

```csharp
// ThreadPool kullanımı
public void ThreadPoolExample()
{
    Console.WriteLine("İş parçacığı havuzu bilgileri:");
    ThreadPool.GetAvailableThreads(out int workerThreads, out int completionPortThreads);
    Console.WriteLine($"Kullanılabilir iş parçacıkları: {workerThreads}");
    Console.WriteLine($"Kullanılabilir I/O tamamlama bağlantı noktası iş parçacıkları: {completionPortThreads}");
    
    // İş parçacığı havuzu yapılandırması
    ThreadPool.GetMinThreads(out int minWorkerThreads, out int minCompletionPortThreads);
    Console.WriteLine($"Minimum iş parçacıkları: {minWorkerThreads}");
    Console.WriteLine($"Minimum I/O tamamlama bağlantı noktası iş parçacıkları: {minCompletionPortThreads}");
    
    // İş parçacığı havuzunu yapılandırma
    ThreadPool.SetMinThreads(Environment.ProcessorCount * 2, minCompletionPortThreads);
    
    // İş parçacığı havuzuna iş gönderme
    for (int i = 0; i < 10; i++)
    {
        int taskId = i;
        ThreadPool.QueueUserWorkItem(_ =>
        {
            Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: Görev {taskId} başladı.");
            Thread.Sleep(100);
            Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: Görev {taskId} tamamlandı.");
        });
    }
    
    // İşlerin tamamlanması için bekleme
    Thread.Sleep(2000);
}

// Task Scheduler kullanımı
public async Task TaskSchedulerExample()
{
    // Varsayılan görev zamanlayıcısı
    TaskScheduler defaultScheduler = TaskScheduler.Default;
    Console.WriteLine($"Varsayılan görev zamanlayıcısı: {defaultScheduler}");
    
    // Mevcut görev zamanlayıcısı
    TaskScheduler currentScheduler = TaskScheduler.Current;
    Console.WriteLine($"Mevcut görev zamanlayıcısı: {currentScheduler}");
    
    // Özel görev zamanlayıcısı ile görev çalıştırma
    var options = new TaskCreationOptions { PreferFairness = true };
    
    var task = Task.Factory.StartNew(() =>
    {
        Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId}: Görev çalışıyor.");
        Thread.Sleep(100);
        return "Görev tamamlandı.";
    }, CancellationToken.None, options, defaultScheduler);
    
    string result = await task;
    Console.WriteLine(result);
}
```

## Paralel Programlama En İyi Uygulamaları

```csharp
// Paralel programlama en iyi uygulamaları
public void ParallelBestPracticesExample()
{
    // 1. İş parçacığı güvenli veri yapıları kullanma
    var concurrentDict = new ConcurrentDictionary<int, string>();
    var concurrentQueue = new ConcurrentQueue<int>();
    var concurrentBag = new ConcurrentBag<string>();
    
    // Paralel olarak veri ekleme
    Parallel.For(0, 100, i =>
    {
        // ConcurrentDictionary kullanımı
        concurrentDict.TryAdd(i, $"Değer-{i}");
        
        // ConcurrentQueue kullanımı
        concurrentQueue.Enqueue(i);
        
        // ConcurrentBag kullanımı
        concurrentBag.Add($"Öğe-{i}");
    });
    
    Console.WriteLine($"ConcurrentDictionary öğe sayısı: {concurrentDict.Count}");
    Console.WriteLine($"ConcurrentQueue öğe sayısı: {concurrentQueue.Count}");
    Console.WriteLine($"ConcurrentBag öğe sayısı: {concurrentBag.Count}");
    
    // 2. Uygun paralellik derecesi seçme
    int processorCount = Environment.ProcessorCount;
    Console.WriteLine($"İşlemci çekirdek sayısı: {processorCount}");
    
    var options = new ParallelOptions
    {
        MaxDegreeOfParallelism = processorCount
    };
    
    // 3. İş parçacığı yerel değişkenler kullanma
    Parallel.For(0, 1000, options, () => 0, // İş parçacığı yerel başlangıç değeri
        (i, loop, localSum) =>
        {
            // İş parçacığı yerel değişkeni güncelleme
            localSum += i;
            return localSum;
        },
        localSum => // Son işlem
        {
            Console.WriteLine($"İş parçacığı {Thread.CurrentThread.ManagedThreadId} toplamı: {localSum}");
        });
}
```

## Özet

Bu bölümde, C#'ta paralel programlama temellerini inceledik:

1. **Parallel.For/ForEach**: Döngüleri paralel olarak çalıştırmak için kullanılır.

2. **PLINQ**: LINQ sorgularını paralel olarak çalıştırmak için kullanılır.

3. **Task Parallelism**: Birden fazla görevi paralel olarak çalıştırmak için kullanılır.

4. **Data Parallelism**: Büyük veri kümelerini paralel olarak işlemek için kullanılır.

5. **Synchronization Primitives**: Paylaşılan verilere erişimi senkronize etmek için kullanılır.

6. **Thread Pool Management**: İş parçacığı havuzunu yönetmek için kullanılır.

Paralel programlama, çok çekirdekli işlemcilerin gücünden yararlanmak için önemli bir araçtır. Ancak, doğru kullanılmadığında performans sorunlarına ve hatalara neden olabilir. Bu nedenle, paralel programlama yaparken dikkatli olunmalı ve uygun senkronizasyon mekanizmaları kullanılmalıdır. 