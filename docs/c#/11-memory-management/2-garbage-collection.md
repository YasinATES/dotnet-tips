# Garbage Collection (Çöp Toplama)

.NET'in otomatik bellek yönetim sistemi olan Garbage Collection (GC), kullanılmayan nesneleri tespit ederek belleği otomatik olarak temizler. Bu mekanizma, bellek sızıntılarını önler ve geliştiricilerin manuel bellek yönetimi ile uğraşmasına gerek kalmaz.

## 1. GC Generations (GC Nesilleri)

.NET Garbage Collector, nesneleri üç farklı nesil (generation) içinde yönetir.

```csharp
// GC nesilleri
public void GCGenerationsExample()
{
    // Nesil 0 (Generation 0) - Yeni oluşturulan nesneler
    object gen0Object = new object();
    
    Console.WriteLine($"Nesil 0 nesnesi oluşturuldu");
    Console.WriteLine($"Nesil 0'daki nesne sayısı: {GC.CollectionCount(0)}");
    
    // GC çalıştırma
    GC.Collect(0);
    
    Console.WriteLine($"Nesil 0 GC sonrası nesne sayısı: {GC.CollectionCount(0)}");
    
    // Nesil 1 (Generation 1) - Bir GC'den hayatta kalan Nesil 0 nesneleri
    List<byte[]> gen1Objects = new List<byte[]>();
    for (int i = 0; i < 10; i++)
    {
        gen1Objects.Add(new byte[1000]);
    }
    
    GC.Collect(0);
    
    Console.WriteLine($"Nesil 1'e yükseltilen nesneler");
    Console.WriteLine($"Nesil 1'deki nesne sayısı: {GC.CollectionCount(1)}");
    
    // Nesil 2 (Generation 2) - Bir GC'den hayatta kalan Nesil 1 nesneleri
    GC.Collect(1);
    
    Console.WriteLine($"Nesil 2'ye yükseltilen nesneler");
    Console.WriteLine($"Nesil 2'deki nesne sayısı: {GC.CollectionCount(2)}");
}
```

### Nesil Temelli Toplama Stratejisi

Garbage Collector, farklı nesilleri farklı sıklıklarla toplar.

```csharp
// Nesil temelli toplama stratejisi
public void GenerationalCollectionStrategyExample()
{
    // Nesil 0 daha sık toplanır
    Console.WriteLine("Nesil 0 toplama başlatılıyor...");
    GC.Collect(0);
    
    // Nesil 1 daha az sıklıkla toplanır
    Console.WriteLine("Nesil 1 toplama başlatılıyor...");
    GC.Collect(1);
    
    // Nesil 2 en az sıklıkla toplanır
    Console.WriteLine("Nesil 2 toplama başlatılıyor...");
    GC.Collect(2);
    
    // Tüm nesilleri toplama
    Console.WriteLine("Tam toplama başlatılıyor...");
    GC.Collect();
    
    // Nesil istatistikleri
    Console.WriteLine($"Nesil 0 toplama sayısı: {GC.CollectionCount(0)}");
    Console.WriteLine($"Nesil 1 toplama sayısı: {GC.CollectionCount(1)}");
    Console.WriteLine($"Nesil 2 toplama sayısı: {GC.CollectionCount(2)}");
}
```

## 2. GC Modes (GC Modları)

.NET, farklı senaryolar için farklı GC modları sunar.

```csharp
// GC modları
public void GCModesExample()
{
    // Workstation GC - İstemci uygulamaları için varsayılan mod
    // Server GC - Sunucu uygulamaları için optimize edilmiş mod
    
    // GC modu bilgisi
    bool isServerGC = GCSettings.IsServerGC;
    Console.WriteLine($"Server GC modu aktif mi: {isServerGC}");
    
    // Eşzamanlı (Concurrent) GC
    GCSettings.LatencyMode = GCLatencyMode.Batch;
    Console.WriteLine($"Mevcut GC gecikme modu: {GCSettings.LatencyMode}");
    
    // Farklı gecikme modları:
    // - Batch: Yüksek verimlilik, yüksek gecikme
    // - Interactive: Orta verimlilik, orta gecikme
    // - LowLatency: Düşük verimlilik, düşük gecikme
    // - SustainedLowLatency: En düşük verimlilik, en düşük gecikme
}
```

### Eşzamanlı ve Arka Plan GC

Eşzamanlı (Concurrent) ve Arka Plan (Background) GC, uygulama performansını artırır.

```csharp
// Eşzamanlı ve arka plan GC
public void ConcurrentAndBackgroundGCExample()
{
    // Arka plan GC'yi etkinleştirme/devre dışı bırakma
    bool isBackgroundGCEnabled = true;
    GCSettings.LatencyMode = isBackgroundGCEnabled ? 
        GCLatencyMode.SustainedLowLatency : 
        GCLatencyMode.Batch;
    
    Console.WriteLine($"Arka plan GC etkin mi: {GCSettings.LatencyMode == GCLatencyMode.SustainedLowLatency}");
    
    // Eşzamanlı GC, uygulama çalışırken arka planda çalışır
    // ve uygulama duraklamalarını azaltır
    
    // Arka plan GC, Nesil 2 toplamalarını arka planda yapar
    // ve uygulama duraklamalarını daha da azaltır
}
```

## 3. Large Object Heap (Büyük Nesne Öbeği)

85 KB'den büyük nesneler, Large Object Heap (LOH) adı verilen özel bir bellek bölgesinde saklanır.

```csharp
// Büyük nesne öbeği
public void LargeObjectHeapExample()
{
    // Küçük nesne (Small Object Heap'te saklanır)
    byte[] smallArray = new byte[80 * 1024]; // 80 KB
    
    // Büyük nesne (Large Object Heap'te saklanır)
    byte[] largeArray = new byte[86 * 1024]; // 86 KB
    
    Console.WriteLine($"Küçük dizi boyutu: {smallArray.Length / 1024} KB");
    Console.WriteLine($"Büyük dizi boyutu: {largeArray.Length / 1024} KB");
    
    // LOH'taki nesneler genellikle Nesil 2'de toplanır
    // ve daha az sıklıkla toplanır
    
    // LOH parçalanması
    List<byte[]> largeArrays = new List<byte[]>();
    for (int i = 0; i < 10; i++)
    {
        largeArrays.Add(new byte[86 * 1024]);
        largeArrays.RemoveAt(0);
    }
    
    // LOH parçalanmasını azaltmak için
    GCSettings.LargeObjectHeapCompactionMode = GCLargeObjectHeapCompactionMode.CompactOnce;
    GC.Collect();
}
```

### LOH Optimizasyonu

LOH'taki nesnelerin yönetimi, performans açısından önemlidir.

```csharp
// LOH optimizasyonu
public void LOHOptimizationExample()
{
    // LOH parçalanmasını önlemek için nesne havuzu kullanma
    ObjectPool<byte[]> largeArrayPool = new ObjectPool<byte[]>(() => new byte[86 * 1024], 10);
    
    // Havuzdan nesne alma
    byte[] array1 = largeArrayPool.Get();
    
    // Nesneyi kullanma
    Array.Fill(array1, (byte)1);
    
    // Nesneyi havuza geri verme
    largeArrayPool.Return(array1);
    
    // Nesne havuzu, LOH parçalanmasını azaltır ve GC baskısını düşürür
}

// Basit bir nesne havuzu uygulaması
public class ObjectPool<T> where T : class
{
    private readonly Func<T> _objectGenerator;
    private readonly Stack<T> _pool;
    
    public ObjectPool(Func<T> objectGenerator, int initialCapacity)
    {
        _objectGenerator = objectGenerator;
        _pool = new Stack<T>(initialCapacity);
        
        for (int i = 0; i < initialCapacity; i++)
        {
            _pool.Push(objectGenerator());
        }
    }
    
    public T Get()
    {
        return _pool.Count > 0 ? _pool.Pop() : _objectGenerator();
    }
    
    public void Return(T item)
    {
        _pool.Push(item);
    }
}
```

## 4. Memory Pressure (Bellek Baskısı)

Bellek baskısı, uygulamanın bellek kullanımının artması ve GC'nin daha sık çalışmasına neden olmasıdır.

```csharp
// Bellek baskısı
public void MemoryPressureExample()
{
    // Bellek baskısı oluşturma
    List<byte[]> memoryHog = new List<byte[]>();
    
    // Bellek kullanımını izleme
    long memoryBefore = GC.GetTotalMemory(false);
    
    // Bellek tüketimi
    for (int i = 0; i < 1000; i++)
    {
        memoryHog.Add(new byte[1000]);
        
        if (i % 100 == 0)
        {
            long currentMemory = GC.GetTotalMemory(false);
            Console.WriteLine($"Bellek kullanımı: {(currentMemory - memoryBefore) / 1024 / 1024} MB");
        }
    }
    
    // Bellek baskısını azaltma
    memoryHog.Clear();
    memoryHog = null;
    
    // Manuel GC tetikleme (genellikle önerilmez)
    GC.Collect();
    
    long memoryAfter = GC.GetTotalMemory(true);
    Console.WriteLine($"Temizleme sonrası bellek kullanımı: {memoryAfter / 1024 / 1024} MB");
}
```

### Bellek Baskısını Azaltma Teknikleri

Bellek baskısını azaltmak için çeşitli teknikler kullanılabilir.

```csharp
// Bellek baskısını azaltma teknikleri
public void ReduceMemoryPressureExample()
{
    // 1. Nesne havuzları kullanma
    var bufferPool = new ObjectPool<byte[]>(() => new byte[4096], 20);
    
    // 2. Büyük nesneleri yeniden kullanma
    byte[] buffer = new byte[4096];
    for (int i = 0; i < 10; i++)
    {
        ProcessData(buffer); // Aynı buffer'ı yeniden kullanma
    }
    
    // 3. Gereksiz referansları temizleme
    List<string> data = LoadLargeData();
    ProcessLargeData(data);
    data.Clear(); // İşlem bittikten sonra temizleme
    data = null; // Referansı kaldırma
    
    // 4. Bellek kullanımını izleme
    long memory = GC.GetTotalMemory(false);
    Console.WriteLine($"Mevcut bellek kullanımı: {memory / 1024 / 1024} MB");
}

private void ProcessData(byte[] buffer)
{
    // Veri işleme simülasyonu
    Array.Fill(buffer, (byte)0);
}

private List<string> LoadLargeData()
{
    // Büyük veri yükleme simülasyonu
    return new List<string>(1000);
}

private void ProcessLargeData(List<string> data)
{
    // Veri işleme simülasyonu
}
```

## 5. Finalization (Sonlandırma)

Finalization, yönetilmeyen kaynakları temizlemek için kullanılır.

```csharp
// Sonlandırma
public class ResourceHolder : IDisposable
{
    private bool _disposed = false;
    private IntPtr _unmanagedResource;
    
    public ResourceHolder()
    {
        // Yönetilmeyen kaynak tahsisi simülasyonu
        _unmanagedResource = Marshal.AllocHGlobal(1000);
        Console.WriteLine("Yönetilmeyen kaynak tahsis edildi");
    }
    
    // Finalizer (Yıkıcı)
    ~ResourceHolder()
    {
        Dispose(false);
    }
    
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Yönetilen kaynakları temizle
                Console.WriteLine("Yönetilen kaynaklar temizlendi");
            }
            
            // Yönetilmeyen kaynakları temizle
            if (_unmanagedResource != IntPtr.Zero)
            {
                Marshal.FreeHGlobal(_unmanagedResource);
                _unmanagedResource = IntPtr.Zero;
                Console.WriteLine("Yönetilmeyen kaynak serbest bırakıldı");
            }
            
            _disposed = true;
        }
    }
    
    public void UseResource()
    {
        if (_disposed)
        {
            throw new ObjectDisposedException(nameof(ResourceHolder));
        }
        
        Console.WriteLine("Kaynak kullanılıyor");
    }
}

// Sonlandırma örneği
public void FinalizationExample()
{
    // using bloğu ile otomatik Dispose
    using (ResourceHolder resource1 = new ResourceHolder())
    {
        resource1.UseResource();
    } // resource1.Dispose() otomatik olarak çağrılır
    
    // Manuel Dispose
    ResourceHolder resource2 = new ResourceHolder();
    resource2.UseResource();
    resource2.Dispose();
    
    // Dispose edilmeyen nesne
    {
        ResourceHolder resource3 = new ResourceHolder();
        resource3.UseResource();
        // resource3.Dispose() çağrılmadı, finalizer tarafından temizlenecek
    }
    
    // Finalizer'ın çalışmasını zorlamak için GC'yi tetikleme
    GC.Collect();
    GC.WaitForPendingFinalizers();
}
```

### Finalizer Kuyruk ve Sonlandırıcı Thread

Finalizer'lar, özel bir thread tarafından işlenir.

```csharp
// Finalizer kuyruk ve sonlandırıcı thread
public void FinalizerQueueExample()
{
    // Finalizer'ı olan 1000 nesne oluşturma
    for (int i = 0; i < 1000; i++)
    {
        new ResourceWithFinalizer();
    }
    
    // GC'yi tetikleme
    GC.Collect();
    
    // Finalizer thread'inin çalışmasını bekleme
    GC.WaitForPendingFinalizers();
    
    Console.WriteLine("Tüm finalizer'lar tamamlandı");
}

// Finalizer'ı olan basit bir sınıf
public class ResourceWithFinalizer
{
    private static int _instanceCount = 0;
    private int _id;
    
    public ResourceWithFinalizer()
    {
        _id = Interlocked.Increment(ref _instanceCount);
    }
    
    ~ResourceWithFinalizer()
    {
        // Finalizer işlemi simülasyonu
        Thread.Sleep(1); // 1 ms gecikme
        
        if (_id % 100 == 0)
        {
            Console.WriteLine($"Nesne {_id} sonlandırıldı");
        }
    }
}
```

## 6. Weak References (Zayıf Referanslar)

Zayıf referanslar, GC'nin nesneleri toplamadan önce referansları izlemesine olanak tanır.

```csharp
// Zayıf referanslar
public void WeakReferenceExample()
{
    // Normal (güçlü) referans
    LargeObject strongRef = new LargeObject();
    
    // Zayıf referans
    WeakReference<LargeObject> weakRef = new WeakReference<LargeObject>(strongRef);
    
    // Zayıf referansı kontrol etme
    if (weakRef.TryGetTarget(out LargeObject target1))
    {
        Console.WriteLine("Zayıf referans hala geçerli, nesne kullanılabilir");
        target1.DoWork();
    }
    
    // Güçlü referansı kaldırma
    strongRef = null;
    
    // GC'yi tetikleme
    GC.Collect();
    
    // Zayıf referansı tekrar kontrol etme
    if (weakRef.TryGetTarget(out LargeObject target2))
    {
        Console.WriteLine("Zayıf referans hala geçerli (beklenmedik)");
        target2.DoWork();
    }
    else
    {
        Console.WriteLine("Zayıf referans artık geçerli değil, nesne GC tarafından toplandı");
    }
}

// Büyük nesne sınıfı
public class LargeObject
{
    private byte[] _data;
    
    public LargeObject()
    {
        // Büyük bellek tahsisi
        _data = new byte[10 * 1024 * 1024]; // 10 MB
        Console.WriteLine("Büyük nesne oluşturuldu");
    }
    
    public void DoWork()
    {
        Console.WriteLine("Büyük nesne üzerinde çalışılıyor");
    }
}
```

### Önbellek Uygulaması

Zayıf referanslar, bellek dostu önbellekler oluşturmak için kullanılabilir.

```csharp
// Zayıf referans tabanlı önbellek
public class WeakCache<TKey, TValue> where TValue : class
{
    private readonly Dictionary<TKey, WeakReference<TValue>> _cache = new Dictionary<TKey, WeakReference<TValue>>();
    private readonly Func<TKey, TValue> _valueFactory;
    
    public WeakCache(Func<TKey, TValue> valueFactory)
    {
        _valueFactory = valueFactory;
    }
    
    public TValue GetOrCreate(TKey key)
    {
        // Önbellekte zaten varsa ve hala geçerliyse, onu döndür
        if (_cache.TryGetValue(key, out WeakReference<TValue> weakRef) && 
            weakRef.TryGetTarget(out TValue cachedValue))
        {
            return cachedValue;
        }
        
        // Yoksa veya GC tarafından toplanmışsa, yeni değer oluştur
        TValue newValue = _valueFactory(key);
        
        // Yeni değeri önbelleğe ekle veya güncelle
        _cache[key] = new WeakReference<TValue>(newValue);
        
        return newValue;
    }
    
    public void CleanupCache()
    {
        // Geçersiz referansları temizle
        List<TKey> keysToRemove = new List<TKey>();
        
        foreach (var entry in _cache)
        {
            if (!entry.Value.TryGetTarget(out _))
            {
                keysToRemove.Add(entry.Key);
            }
        }
        
        foreach (TKey key in keysToRemove)
        {
            _cache.Remove(key);
        }
    }
}

// Zayıf önbellek kullanımı
public void WeakCacheExample()
{
    // Zayıf önbellek oluşturma
    WeakCache<string, LargeObject> cache = new WeakCache<string, LargeObject>(
        key => new LargeObject());
    
    // Önbellekten değer alma
    LargeObject obj1 = cache.GetOrCreate("key1");
    obj1.DoWork();
    
    // GC'yi tetikleme
    GC.Collect();
    
    // Önbellekten tekrar değer alma (yeni nesne oluşturulabilir)
    LargeObject obj2 = cache.GetOrCreate("key1");
    obj2.DoWork();
    
    // Önbelleği temizleme
    cache.CleanupCache();
}
```

## 7. GC Notifications (GC Bildirimleri)

GC bildirimleri, GC olaylarını izlemenize ve uygulamanızı buna göre ayarlamanıza olanak tanır.

```csharp
// GC bildirimleri
public void GCNotificationsExample()
{
    // GC bildirimleri için kayıt olma
    GCNotificationStatus status = GC.RegisterForFullGCNotification(10, 10);
    
    if (status == GCNotificationStatus.Succeeded)
    {
        Console.WriteLine("GC bildirimleri için başarıyla kayıt olundu");
        
        // GC bildirimlerini dinleyen bir thread başlatma
        Thread notificationThread = new Thread(MonitorGC)
        {
            IsBackground = true
        };
        
        notificationThread.Start();
        
        // Bellek baskısı oluşturma
        List<byte[]> memoryPressure = new List<byte[]>();
        for (int i = 0; i < 100; i++)
        {
            memoryPressure.Add(new byte[1024 * 1024]); // 1 MB
            Thread.Sleep(10);
        }
        
        // Bildirimleri durdurma
        // notificationThread.Join();
    }
    else
    {
        Console.WriteLine("GC bildirimleri için kayıt başarısız oldu");
    }
}

// GC bildirimlerini izleyen metot
private void MonitorGC()
{
    while (true)
    {
        // Yaklaşan tam GC için bekleme
        GCNotificationStatus status = GC.WaitForFullGCApproach();
        
        if (status == GCNotificationStatus.Succeeded)
        {
            Console.WriteLine("Tam GC yaklaşıyor - Kritik işlemleri duraklat");
            
            // Burada kritik işlemleri duraklatabilirsiniz
        }
        
        // Tam GC'nin tamamlanmasını bekleme
        status = GC.WaitForFullGCComplete();
        
        if (status == GCNotificationStatus.Succeeded)
        {
            Console.WriteLine("Tam GC tamamlandı - İşlemleri devam ettir");
            
            // Burada işlemleri devam ettirebilirsiniz
        }
    }
}
```

### Bellek Kullanımını İzleme

GC bildirimleri, bellek kullanımını izlemek için kullanılabilir.

```csharp
// Bellek kullanımını izleme
public class MemoryMonitor
{
    private readonly Timer _monitorTimer;
    private long _lastMemoryUsage;
    
    public MemoryMonitor(int monitorIntervalMs)
    {
        _lastMemoryUsage = GC.GetTotalMemory(false);
        _monitorTimer = new Timer(MonitorMemory, null, monitorIntervalMs, monitorIntervalMs);
    }
    
    private void MonitorMemory(object state)
    {
        long currentMemory = GC.GetTotalMemory(false);
        long difference = currentMemory - _lastMemoryUsage;
        
        Console.WriteLine($"Bellek kullanımı: {currentMemory / 1024 / 1024} MB " +
                         $"(Değişim: {difference / 1024 / 1024} MB)");
        
        // Bellek kullanımı belirli bir eşiği aşarsa uyarı
        if (currentMemory > 500 * 1024 * 1024) // 500 MB
        {
            Console.WriteLine("UYARI: Yüksek bellek kullanımı!");
            
            // Burada bellek kullanımını azaltmak için önlemler alabilirsiniz
            // Örneğin, önbellekleri temizleme, kullanılmayan kaynakları serbest bırakma
        }
        
        _lastMemoryUsage = currentMemory;
    }
    
    public void Stop()
    {
        _monitorTimer.Dispose();
    }
}

// Bellek izleme kullanımı
public void MemoryMonitoringExample()
{
    // Bellek izleyici başlatma
    MemoryMonitor monitor = new MemoryMonitor(1000); // 1 saniye aralıklarla
    
    // Bellek tüketimi simülasyonu
    List<byte[]> memoryConsumer = new List<byte[]>();
    
    for (int i = 0; i < 10; i++)
    {
        // Her döngüde 50 MB bellek tüketme
        memoryConsumer.Add(new byte[50 * 1024 * 1024]);
        Thread.Sleep(2000);
    }
    
    // İzleyiciyi durdurma
    monitor.Stop();
}
```

## Özet

Bu bölümde, .NET'in Garbage Collection mekanizmasını ve ilgili konuları inceledik:

1. **GC Generations**: Nesnelerin yaşam döngüsünü yöneten üç nesil (0, 1 ve 2).

2. **GC Modes**: Workstation, Server, Concurrent ve Background GC modları.

3. **Large Object Heap**: Büyük nesnelerin yönetimi ve optimizasyonu.

4. **Memory Pressure**: Bellek baskısının nedenleri ve azaltma teknikleri.

5. **Finalization**: Yönetilmeyen kaynakların temizlenmesi için kullanılan mekanizma.

6. **Weak References**: GC'nin nesneleri toplamadan önce referansları izlemesine olanak tanıyan yapı.

7. **GC Notifications**: GC olaylarını izleme ve uygulamayı buna göre ayarlama.

Etkili bellek yönetimi, yüksek performanslı ve güvenilir .NET uygulamaları geliştirmek için kritik öneme sahiptir. Garbage Collection mekanizmasını anlamak, bellek kullanımını optimize etmenize ve bellek ile ilgili sorunları çözmenize yardımcı olur. 