# Memory Optimization (Bellek Optimizasyonu)

.NET uygulamalarında bellek kullanımını optimize etmek, performansı artırmak ve kaynak tüketimini azaltmak için kritik öneme sahiptir. Bu bölümde, bellek optimizasyonu için çeşitli teknikleri inceleyeceğiz.

## 1. Object Pooling (Nesne Havuzlama)

Nesne havuzlama, sık oluşturulan ve imha edilen nesnelerin yeniden kullanılmasını sağlayarak GC baskısını azaltan bir tekniktir.

```csharp
// Basit bir nesne havuzu
public class ObjectPool<T> where T : class, new()
{
    private readonly ConcurrentBag<T> _objects;
    private readonly Func<T> _objectGenerator;
    private readonly Action<T> _objectReset;
    
    public ObjectPool(Func<T> objectGenerator = null, Action<T> objectReset = null)
    {
        _objects = new ConcurrentBag<T>();
        _objectGenerator = objectGenerator ?? (() => new T());
        _objectReset = objectReset ?? (_ => { });
    }
    
    // Havuzdan nesne alma
    public T Get()
    {
        if (_objects.TryTake(out T item))
        {
            return item;
        }
        
        return _objectGenerator();
    }
    
    // Nesneyi havuza geri verme
    public void Return(T item)
    {
        _objectReset(item);
        _objects.Add(item);
    }
}

// Kullanım örneği
public class ExpensiveObject
{
    private byte[] _buffer;
    
    public ExpensiveObject()
    {
        _buffer = new byte[1024 * 1024]; // 1 MB
        Console.WriteLine("Pahalı nesne oluşturuldu");
    }
    
    public void Reset()
    {
        Array.Clear(_buffer, 0, _buffer.Length);
    }
    
    public void Process()
    {
        Console.WriteLine("Nesne işleniyor");
    }
}

public void ObjectPoolExample()
{
    // Nesne havuzu oluşturma
    var pool = new ObjectPool<ExpensiveObject>(
        objectGenerator: () => new ExpensiveObject(),
        objectReset: obj => obj.Reset()
    );
    
    // Havuzdan nesne alma
    var obj1 = pool.Get();
    obj1.Process();
    
    // Nesneyi havuza geri verme
    pool.Return(obj1);
    
    // Aynı nesneyi tekrar kullanma
    var obj2 = pool.Get(); // Muhtemelen aynı nesne
    obj2.Process();
    pool.Return(obj2);
    
    // Çoklu nesne kullanımı
    for (int i = 0; i < 10; i++)
    {
        var obj = pool.Get();
        obj.Process();
        pool.Return(obj);
    }
}
```

### Microsoft.Extensions.ObjectPool Kullanımı

.NET, nesne havuzlama için yerleşik bir kütüphane sunar.

```csharp
// Microsoft.Extensions.ObjectPool kullanımı
public void MicrosoftObjectPoolExample()
{
    // Havuz oluşturucu
    var provider = new DefaultObjectPoolProvider();
    
    // Havuz politikası
    var policy = new DefaultPooledObjectPolicy<ExpensiveObject>();
    
    // Nesne havuzu oluşturma
    var pool = provider.Create(policy);
    
    // Havuzdan nesne alma
    var obj = pool.Get();
    obj.Process();
    
    // Nesneyi havuza geri verme
    pool.Return(obj);
}
```

## 2. Array Pooling (Dizi Havuzlama)

Dizi havuzlama, özellikle geçici diziler için bellek tahsisini azaltır.

```csharp
// ArrayPool kullanımı
public void ArrayPoolExample()
{
    // Yerleşik ArrayPool'dan bir örnek alma
    ArrayPool<byte> pool = ArrayPool<byte>.Shared;
    
    // Havuzdan dizi kiralama
    byte[] buffer = pool.Rent(1024); // En az 1024 baytlık bir dizi
    
    try
    {
        // Diziyi kullanma
        for (int i = 0; i < buffer.Length; i++)
        {
            buffer[i] = (byte)(i % 256);
        }
        
        // Dizi ile işlem yapma
        ProcessBuffer(buffer);
    }
    finally
    {
        // Diziyi havuza geri verme
        pool.Return(buffer, clearArray: true);
    }
}

private void ProcessBuffer(byte[] buffer)
{
    // Dizi işleme simülasyonu
    Console.WriteLine($"Dizi işleniyor, boyut: {buffer.Length}");
}
```

### Özel Dizi Havuzu

Özel gereksinimler için kendi dizi havuzunuzu oluşturabilirsiniz.

```csharp
// Özel dizi havuzu
public class CustomArrayPool<T>
{
    private readonly ConcurrentDictionary<int, ConcurrentBag<T[]>> _buckets = new ConcurrentDictionary<int, ConcurrentBag<T[]>>();
    private readonly int[] _sizeBuckets;
    
    public CustomArrayPool(int[] sizeBuckets)
    {
        _sizeBuckets = sizeBuckets;
        
        foreach (var size in sizeBuckets)
        {
            _buckets[size] = new ConcurrentBag<T[]>();
        }
    }
    
    // En uygun boyut kategorisini bulma
    private int GetBestBucketSize(int minimumSize)
    {
        for (int i = 0; i < _sizeBuckets.Length; i++)
        {
            if (_sizeBuckets[i] >= minimumSize)
            {
                return _sizeBuckets[i];
            }
        }
        
        return minimumSize; // Uygun kategori yoksa, tam boyut
    }
    
    // Dizi kiralama
    public T[] Rent(int minimumSize)
    {
        int bucketSize = GetBestBucketSize(minimumSize);
        
        if (_buckets.TryGetValue(bucketSize, out var bucket) && bucket.TryTake(out var array))
        {
            return array;
        }
        
        return new T[bucketSize];
    }
    
    // Diziyi geri verme
    public void Return(T[] array, bool clearArray = false)
    {
        if (array == null)
        {
            throw new ArgumentNullException(nameof(array));
        }
        
        if (clearArray)
        {
            Array.Clear(array, 0, array.Length);
        }
        
        if (_buckets.TryGetValue(array.Length, out var bucket))
        {
            bucket.Add(array);
        }
    }
}

// Özel dizi havuzu kullanımı
public void CustomArrayPoolExample()
{
    // Farklı boyut kategorileri
    var pool = new CustomArrayPool<byte>(new[] { 128, 256, 512, 1024, 2048, 4096 });
    
    // 300 baytlık bir dizi gerekiyor
    byte[] buffer = pool.Rent(300); // 512 baytlık bir dizi alınır
    
    try
    {
        // Diziyi kullanma
        ProcessBuffer(buffer);
    }
    finally
    {
        // Diziyi havuza geri verme
        pool.Return(buffer, true);
    }
}
```

## 3. Buffer Management (Tampon Yönetimi)

Tampon yönetimi, özellikle I/O işlemleri için bellek kullanımını optimize eder.

```csharp
// Tampon yönetimi
public class BufferManager
{
    private readonly ArrayPool<byte> _arrayPool;
    private readonly int _bufferSize;
    
    public BufferManager(int bufferSize)
    {
        _arrayPool = ArrayPool<byte>.Shared;
        _bufferSize = bufferSize;
    }
    
    // Tampon kiralama
    public byte[] GetBuffer()
    {
        return _arrayPool.Rent(_bufferSize);
    }
    
    // Tamponu geri verme
    public void ReturnBuffer(byte[] buffer)
    {
        _arrayPool.Return(buffer);
    }
    
    // Tamponla işlem yapma
    public async Task ProcessDataAsync(Stream source, Stream destination)
    {
        byte[] buffer = GetBuffer();
        
        try
        {
            int bytesRead;
            while ((bytesRead = await source.ReadAsync(buffer, 0, buffer.Length)) > 0)
            {
                await destination.WriteAsync(buffer, 0, bytesRead);
            }
        }
        finally
        {
            ReturnBuffer(buffer);
        }
    }
}

// Tampon yöneticisi kullanımı
public async Task BufferManagerExample()
{
    var bufferManager = new BufferManager(4096);
    
    using (var sourceStream = new FileStream("source.dat", FileMode.Open))
    using (var destStream = new FileStream("destination.dat", FileMode.Create))
    {
        await bufferManager.ProcessDataAsync(sourceStream, destStream);
    }
}
```

### RecyclableMemoryStream Kullanımı

Microsoft'un RecyclableMemoryStream kütüphanesi, bellek akışlarını verimli bir şekilde yönetir.

```csharp
// RecyclableMemoryStream kullanımı
public void RecyclableMemoryStreamExample()
{
    // Yönetici oluşturma
    var manager = new RecyclableMemoryStreamManager();
    
    // Akış oluşturma
    using (var stream = manager.GetStream("ExampleTag"))
    {
        // Veri yazma
        byte[] data = Encoding.UTF8.GetBytes("Örnek veri");
        stream.Write(data, 0, data.Length);
        
        // Akışı başa sarma
        stream.Position = 0;
        
        // Veri okuma
        var reader = new StreamReader(stream);
        string content = reader.ReadToEnd();
        
        Console.WriteLine($"Okunan içerik: {content}");
    }
    
    // Akış otomatik olarak havuza geri döner
}
```

## 4. Memory Cache (Bellek Önbelleği)

Bellek önbelleği, sık erişilen verileri RAM'de saklayarak performansı artırır.

```csharp
// Bellek önbelleği
public class MemoryCacheExample
{
    private readonly IMemoryCache _cache;
    
    public MemoryCacheExample()
    {
        // Önbellek oluşturma
        _cache = new MemoryCache(new MemoryCacheOptions
        {
            SizeLimit = 1024, // 1024 birim
            ExpirationScanFrequency = TimeSpan.FromMinutes(1)
        });
    }
    
    // Veri alma veya ekleme
    public string GetOrCreateData(string key, Func<string> dataFactory)
    {
        return _cache.GetOrCreate(key, entry =>
        {
            // Önbellek giriş ayarları
            entry.SetSize(1); // 1 birim boyut
            entry.SetAbsoluteExpiration(TimeSpan.FromHours(1)); // 1 saat sonra sona erer
            entry.SetSlidingExpiration(TimeSpan.FromMinutes(10)); // 10 dakika kullanılmazsa sona erer
            
            // Veriyi oluştur
            string data = dataFactory();
            Console.WriteLine($"Veri oluşturuldu: {key}");
            return data;
        });
    }
    
    // Önbellekten veri kaldırma
    public void RemoveFromCache(string key)
    {
        _cache.Remove(key);
    }
    
    // Önbellek kullanım örneği
    public void CacheUsageExample()
    {
        // İlk çağrı - veri oluşturulur
        string data1 = GetOrCreateData("key1", () => "Değer 1");
        Console.WriteLine($"Alınan veri: {data1}");
        
        // İkinci çağrı - önbellekten alınır
        string data2 = GetOrCreateData("key1", () => "Bu değer kullanılmaz");
        Console.WriteLine($"Alınan veri: {data2}");
        
        // Veriyi önbellekten kaldırma
        RemoveFromCache("key1");
        
        // Üçüncü çağrı - veri tekrar oluşturulur
        string data3 = GetOrCreateData("key1", () => "Yeni değer");
        Console.WriteLine($"Alınan veri: {data3}");
    }
}
```

### Önbellek Eviction Politikaları

Bellek baskısı durumunda önbellekten öğelerin çıkarılma stratejileri.

```csharp
// Önbellek eviction politikaları
public class CacheEvictionExample
{
    private readonly IMemoryCache _cache;
    
    public CacheEvictionExample()
    {
        _cache = new MemoryCache(new MemoryCacheOptions());
    }
    
    // Önbellek eviction callback'i ile veri ekleme
    public void AddWithEvictionCallback(string key, object value)
    {
        var cacheEntryOptions = new MemoryCacheEntryOptions()
            .SetPriority(CacheItemPriority.Normal) // Öncelik
            .RegisterPostEvictionCallback(EvictionCallback); // Eviction callback
        
        _cache.Set(key, value, cacheEntryOptions);
    }
    
    // Eviction callback metodu
    private void EvictionCallback(object key, object value, EvictionReason reason, object state)
    {
        Console.WriteLine($"Önbellekten çıkarıldı: {key}, Neden: {reason}");
        
        // Eviction nedenine göre işlem yapma
        switch (reason)
        {
            case EvictionReason.Capacity:
                // Kapasite aşıldığında
                Console.WriteLine("Önbellek kapasitesi aşıldı");
                break;
                
            case EvictionReason.Expired:
                // Süre dolduğunda
                Console.WriteLine("Önbellek öğesinin süresi doldu");
                break;
                
            case EvictionReason.Removed:
                // Manuel kaldırıldığında
                Console.WriteLine("Önbellek öğesi manuel olarak kaldırıldı");
                break;
        }
    }
    
    // Kullanım örneği
    public void EvictionExample()
    {
        // Veri ekleme
        AddWithEvictionCallback("key1", "Değer 1");
        
        // Manuel kaldırma
        _cache.Remove("key1");
        
        // Süreli veri ekleme
        _cache.Set("key2", "Değer 2", TimeSpan.FromSeconds(1));
        
        // Sürenin dolmasını bekleme
        Thread.Sleep(1500);
        
        // key2 artık önbellekte yok
        var value = _cache.Get("key2");
        Console.WriteLine($"key2 değeri: {value ?? "null"}");
    }
}
```

## 5. String Interning (String İçselleştirme)

String interning, aynı içeriğe sahip string nesnelerinin paylaşılmasını sağlar.

```csharp
// String interning
public void StringInterningExample()
{
    // Otomatik interning
    string s1 = "Hello"; // Literal string, otomatik olarak intern edilir
    string s2 = "Hello"; // Aynı string havuzundan alınır
    
    // Referans karşılaştırma
    bool areSameReference = ReferenceEquals(s1, s2);
    Console.WriteLine($"s1 ve s2 aynı referans mı: {areSameReference}"); // True
    
    // Manuel interning
    string s3 = new string(new char[] { 'H', 'e', 'l', 'l', 'o' }); // Yeni bir string nesnesi
    string s4 = string.Intern(s3); // s3'ü intern et
    
    // s4 artık s1 ve s2 ile aynı referansa sahip
    bool s4IsSameAsS1 = ReferenceEquals(s1, s4);
    Console.WriteLine($"s1 ve s4 aynı referans mı: {s4IsSameAsS1}"); // True
    
    // IsInterned ile kontrol
    string s5 = string.IsInterned(s3);
    Console.WriteLine($"s3 intern edilmiş mi: {s5 != null}"); // True
}
```

### String İçselleştirme Performans Analizi

String interning'in performans etkisi.

```csharp
// String içselleştirme performans analizi
public void StringInterningPerformanceAnalysis()
{
    const int iterations = 1000000;
    
    // Test verileri
    var strings = new List<string>();
    for (int i = 0; i < 1000; i++)
    {
        strings.Add($"String_{i % 10}"); // Sadece 10 farklı string
    }
    
    // İçselleştirme olmadan
    var sw1 = Stopwatch.StartNew();
    int equalCount1 = 0;
    
    for (int i = 0; i < iterations; i++)
    {
        string s1 = strings[i % strings.Count];
        string s2 = strings[(i + 1) % strings.Count];
        
        if (s1.Equals(s2)) // İçerik karşılaştırma
        {
            equalCount1++;
        }
    }
    
    sw1.Stop();
    Console.WriteLine($"İçselleştirme olmadan: {sw1.ElapsedMilliseconds} ms, Eşit sayısı: {equalCount1}");
    
    // İçselleştirme ile
    var internedStrings = strings.Select(s => string.Intern(s)).ToList();
    
    var sw2 = Stopwatch.StartNew();
    int equalCount2 = 0;
    
    for (int i = 0; i < iterations; i++)
    {
        string s1 = internedStrings[i % internedStrings.Count];
        string s2 = internedStrings[(i + 1) % internedStrings.Count];
        
        if (ReferenceEquals(s1, s2)) // Referans karşılaştırma (daha hızlı)
        {
            equalCount2++;
        }
    }
    
    sw2.Stop();
    Console.WriteLine($"İçselleştirme ile: {sw2.ElapsedMilliseconds} ms, Eşit sayısı: {equalCount2}");
}
```

## 6. Struct Layout (Struct Düzeni)

Struct düzeni, bellek kullanımını ve erişim hızını optimize eder.

```csharp
// Struct düzeni
[StructLayout(LayoutKind.Sequential)]
public struct SequentialStruct
{
    public int Id;
    public double Value;
    public bool Flag;
}

[StructLayout(LayoutKind.Explicit)]
public struct ExplicitStruct
{
    [FieldOffset(0)]
    public int Id;
    
    [FieldOffset(4)]
    public double Value;
    
    [FieldOffset(12)]
    public bool Flag;
}

[StructLayout(LayoutKind.Auto)]
public struct AutoStruct
{
    public int Id;
    public double Value;
    public bool Flag;
}

// Struct düzeni kullanımı
public void StructLayoutExample()
{
    // Sequential layout - alanlar tanımlandığı sırada yerleştirilir
    SequentialStruct seq = new SequentialStruct { Id = 1, Value = 3.14, Flag = true };
    
    // Explicit layout - alanlar belirtilen offset'lerde yerleştirilir
    ExplicitStruct exp = new ExplicitStruct { Id = 2, Value = 2.71, Flag = false };
    
    // Auto layout - derleyici en verimli düzeni seçer
    AutoStruct auto = new AutoStruct { Id = 3, Value = 1.41, Flag = true };
    
    // Struct boyutları
    Console.WriteLine($"SequentialStruct boyutu: {Marshal.SizeOf<SequentialStruct>()} bayt");
    Console.WriteLine($"ExplicitStruct boyutu: {Marshal.SizeOf<ExplicitStruct>()} bayt");
    Console.WriteLine($"AutoStruct boyutu: {Marshal.SizeOf<AutoStruct>()} bayt");
}
```

### Struct Packing ve Alignment

Struct packing ve alignment, bellek kullanımını optimize eder.

```csharp
// Struct packing ve alignment
[StructLayout(LayoutKind.Sequential, Pack = 1)]
public struct PackedStruct
{
    public byte A;
    public int B;
    public short C;
}

[StructLayout(LayoutKind.Sequential, Pack = 4)]
public struct AlignedStruct
{
    public byte A;
    public int B;
    public short C;
}

// Packing ve alignment kullanımı
public void PackingAndAlignmentExample()
{
    // Pack = 1: Sıkıştırılmış, hizalama yok
    PackedStruct packed = new PackedStruct { A = 1, B = 100, C = 10 };
    
    // Pack = 4: 4 bayt hizalama
    AlignedStruct aligned = new AlignedStruct { A = 1, B = 100, C = 10 };
    
    // Boyut karşılaştırma
    Console.WriteLine($"PackedStruct boyutu: {Marshal.SizeOf<PackedStruct>()} bayt");
    Console.WriteLine($"AlignedStruct boyutu: {Marshal.SizeOf<AlignedStruct>()} bayt");
}
```

## 7. Memory-Mapped Files (Bellek Eşlemeli Dosyalar)

Bellek eşlemeli dosyalar, büyük dosyaları verimli bir şekilde işlemenizi sağlar.

```csharp
// Bellek eşlemeli dosyalar
public void MemoryMappedFileExample()
{
    string filePath = "large_data.bin";
    long fileSize = 100 * 1024 * 1024; // 100 MB
    
    // Büyük dosya oluşturma
    using (var file = new FileStream(filePath, FileMode.Create, FileAccess.ReadWrite))
    {
        file.SetLength(fileSize);
    }
    
    // Bellek eşlemeli dosya oluşturma
    using (var mmf = MemoryMappedFile.CreateFromFile(filePath, FileMode.Open))
    {
        // Tüm dosyayı kapsayan bir görünüm oluşturma
        using (var accessor = mmf.CreateViewAccessor(0, fileSize))
        {
            // Veri yazma
            for (long i = 0; i < fileSize; i += 8)
            {
                accessor.Write(i, i);
            }
            
            // Veri okuma
            for (long i = 0; i < 10; i++)
            {
                long value = accessor.ReadInt64(i * 8);
                Console.WriteLine($"Offset {i * 8}: {value}");
            }
        }
        
        // Belirli bir bölüm için görünüm oluşturma
        using (var stream = mmf.CreateViewStream(1024, 4096))
        {
            // Stream olarak kullanma
            var buffer = new byte[100];
            stream.Read(buffer, 0, buffer.Length);
            
            Console.WriteLine($"Okunan bayt sayısı: {buffer.Length}");
        }
    }
}
```

### Bellek Eşlemeli Dosyalarla Süreçler Arası İletişim

Bellek eşlemeli dosyalar, süreçler arası iletişim için kullanılabilir.

```csharp
// Süreçler arası iletişim için bellek eşlemeli dosyalar
public void InterprocessCommunicationExample()
{
    string mapName = "SharedMemoryRegion";
    int bufferSize = 4096;
    
    // Bellek eşlemeli dosya oluşturma (sunucu)
    using (var mmf = MemoryMappedFile.CreateNew(mapName, bufferSize))
    {
        Console.WriteLine("Bellek eşlemeli dosya oluşturuldu");
        
        // Veri yazma
        using (var accessor = mmf.CreateViewAccessor())
        {
            // Mesaj yazma
            string message = "Merhaba, bu paylaşılan bellek üzerinden bir mesajdır!";
            byte[] messageBytes = Encoding.UTF8.GetBytes(message);
            
            // Mesaj boyutunu yaz
            accessor.Write(0, messageBytes.Length);
            
            // Mesajı yaz
            accessor.WriteArray(4, messageBytes, 0, messageBytes.Length);
            
            Console.WriteLine("Veri yazıldı");
        }
        
        // İstemci işleminin okuması için bekle
        Console.WriteLine("İstemci işleminin okuması için bekleyin...");
        Thread.Sleep(10000);
    }
}

// İstemci tarafı (ayrı bir işlemde çalışır)
public void ClientSideExample()
{
    string mapName = "SharedMemoryRegion";
    
    try
    {
        // Var olan bellek eşlemeli dosyayı aç
        using (var mmf = MemoryMappedFile.OpenExisting(mapName))
        {
            // Veri okuma
            using (var accessor = mmf.CreateViewAccessor())
            {
                // Mesaj boyutunu oku
                int messageLength = accessor.ReadInt32(0);
                
                // Mesajı oku
                byte[] messageBytes = new byte[messageLength];
                accessor.ReadArray(4, messageBytes, 0, messageLength);
                
                string message = Encoding.UTF8.GetString(messageBytes);
                Console.WriteLine($"Okunan mesaj: {message}");
            }
        }
    }
    catch (FileNotFoundException)
    {
        Console.WriteLine("Bellek eşlemeli dosya bulunamadı");
    }
}
```

## Özet

Bu bölümde, .NET uygulamalarında bellek optimizasyonu için çeşitli teknikleri inceledik:

1. **Object Pooling**: Sık oluşturulan ve imha edilen nesnelerin yeniden kullanılması.

2. **Array Pooling**: Geçici diziler için bellek tahsisini azaltma.

3. **Buffer Management**: I/O işlemleri için bellek kullanımını optimize etme.

4. **Memory Cache**: Sık erişilen verileri RAM'de saklama.

5. **String Interning**: Aynı içeriğe sahip string nesnelerinin paylaşılması.

6. **Struct Layout**: Struct düzenini optimize ederek bellek kullanımını ve erişim hızını artırma.

7. **Memory-Mapped Files**: Büyük dosyaları verimli bir şekilde işleme ve süreçler arası iletişim sağlama.

Bu teknikleri uygulamak, uygulamanızın bellek kullanımını optimize ederek performansını artırabilir ve GC baskısını azaltabilir. 