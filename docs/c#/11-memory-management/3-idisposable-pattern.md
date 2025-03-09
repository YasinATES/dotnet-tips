# IDisposable Pattern (IDisposable Deseni)

IDisposable deseni, .NET'te yönetilmeyen kaynakların (dosya tanıtıcıları, ağ bağlantıları, veritabanı bağlantıları vb.) güvenli ve zamanında serbest bırakılmasını sağlayan bir mekanizmadır. Bu desen, bellek sızıntılarını önler ve sistem kaynaklarının verimli kullanılmasını sağlar.

## 1. Dispose Pattern Implementation (Dispose Deseni Uygulaması)

IDisposable arabirimini uygulayan bir sınıf, Dispose() metodunu sağlamalıdır.

```csharp
// Temel IDisposable uygulaması
public class SimpleDisposable : IDisposable
{
    private bool _disposed = false;
    
    // Yönetilmeyen kaynak simülasyonu
    private IntPtr _unmanagedResource;
    
    public SimpleDisposable()
    {
        // Yönetilmeyen kaynak tahsisi
        _unmanagedResource = Marshal.AllocHGlobal(100);
        Console.WriteLine("Yönetilmeyen kaynak tahsis edildi");
    }
    
    public void Dispose()
    {
        // Kaynakları temizle
        if (!_disposed)
        {
            // Yönetilmeyen kaynakları serbest bırak
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
            throw new ObjectDisposedException(nameof(SimpleDisposable));
        }
        
        Console.WriteLine("Kaynak kullanılıyor");
    }
}

// Kullanım örneği
public void SimpleDisposableExample()
{
    var resource = new SimpleDisposable();
    
    try
    {
        resource.UseResource();
    }
    finally
    {
        resource.Dispose();
    }
}
```

### Tam Dispose Deseni

Tam Dispose deseni, hem yönetilen hem de yönetilmeyen kaynakları temizlemek için kullanılır.

```csharp
// Tam Dispose deseni
public class CompleteDisposable : IDisposable
{
    private bool _disposed = false;
    
    // Yönetilmeyen kaynak
    private IntPtr _unmanagedResource;
    
    // Yönetilen kaynak
    private FileStream _managedResource;
    
    public CompleteDisposable(string filePath)
    {
        // Yönetilmeyen kaynak tahsisi
        _unmanagedResource = Marshal.AllocHGlobal(100);
        
        // Yönetilen kaynak tahsisi
        _managedResource = new FileStream(filePath, FileMode.Create);
        
        Console.WriteLine("Kaynaklar tahsis edildi");
    }
    
    // IDisposable uygulaması
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    // Korumalı Dispose metodu
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Yönetilen kaynakları temizle
                if (_managedResource != null)
                {
                    _managedResource.Dispose();
                    _managedResource = null;
                    Console.WriteLine("Yönetilen kaynak temizlendi");
                }
            }
            
            // Yönetilmeyen kaynakları temizle
            if (_unmanagedResource != IntPtr.Zero)
            {
                Marshal.FreeHGlobal(_unmanagedResource);
                _unmanagedResource = IntPtr.Zero;
                Console.WriteLine("Yönetilmeyen kaynak temizlendi");
            }
            
            _disposed = true;
        }
    }
    
    // Finalizer
    ~CompleteDisposable()
    {
        Dispose(false);
    }
    
    public void UseResource()
    {
        if (_disposed)
        {
            throw new ObjectDisposedException(nameof(CompleteDisposable));
        }
        
        Console.WriteLine("Kaynaklar kullanılıyor");
    }
}

// Kullanım örneği
public void CompleteDisposableExample()
{
    var resource = new CompleteDisposable("test.txt");
    
    try
    {
        resource.UseResource();
    }
    finally
    {
        resource.Dispose();
    }
}
```

## 2. Using Statement (Using İfadesi)

Using ifadesi, IDisposable nesnelerinin otomatik olarak temizlenmesini sağlar.

```csharp
// Using ifadesi
public void UsingStatementExample()
{
    // Using bloğu, blok sonunda otomatik olarak Dispose() çağırır
    using (var resource = new CompleteDisposable("test.txt"))
    {
        resource.UseResource();
    } // Burada resource.Dispose() otomatik olarak çağrılır
    
    // Birden fazla kaynak için using
    using (var resource1 = new SimpleDisposable())
    using (var resource2 = new CompleteDisposable("test2.txt"))
    {
        resource1.UseResource();
        resource2.UseResource();
    } // Her iki kaynak da burada temizlenir
    
    // Try-finally bloğuna eşdeğerdir
    var resource3 = new SimpleDisposable();
    try
    {
        resource3.UseResource();
    }
    finally
    {
        if (resource3 != null)
        {
            resource3.Dispose();
        }
    }
}
```

### Using İfadesi ile Hata İşleme

Using ifadesi, istisna durumlarında bile kaynakların temizlenmesini sağlar.

```csharp
// Using ifadesi ile hata işleme
public void UsingWithExceptionHandling()
{
    try
    {
        using (var resource = new CompleteDisposable("test.txt"))
        {
            resource.UseResource();
            
            // İstisna fırlatma
            throw new Exception("Test istisnası");
            
            // Bu noktaya asla ulaşılmaz, ancak resource hala temizlenir
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"İstisna yakalandı: {ex.Message}");
        // Resource zaten temizlendi
    }
}
```

## 3. Using Declaration (Using Bildirimi)

C# 8.0 ile tanıtılan using bildirimi, daha temiz bir sözdizimi sağlar.

```csharp
// Using bildirimi (C# 8.0+)
public void UsingDeclarationExample()
{
    // Using bildirimi - kapsam sonunda Dispose() çağrılır
    using var resource = new CompleteDisposable("test.txt");
    resource.UseResource();
    
    // Diğer işlemler
    Console.WriteLine("Diğer işlemler yapılıyor");
    
    // Metot sonunda resource.Dispose() otomatik olarak çağrılır
}

// İç içe kapsamlarda using bildirimi
public void NestedScopeUsingDeclaration()
{
    using var outerResource = new SimpleDisposable();
    outerResource.UseResource();
    
    {
        using var innerResource = new CompleteDisposable("inner.txt");
        innerResource.UseResource();
        
        // İç kapsam sonu - innerResource.Dispose() çağrılır
    }
    
    // Dış işlemler devam eder
    outerResource.UseResource();
    
    // Metot sonu - outerResource.Dispose() çağrılır
}
```

### Using Bildirimi vs Using İfadesi

Using bildirimi ve using ifadesi arasındaki farklar.

```csharp
// Using bildirimi vs using ifadesi
public void UsingComparisonExample()
{
    // Using ifadesi - kapsam sınırlı
    using (var resource1 = new SimpleDisposable())
    {
        resource1.UseResource();
    } // resource1.Dispose() burada çağrılır
    
    // resource1 artık erişilemez
    
    // Using bildirimi - metot kapsamı
    using var resource2 = new SimpleDisposable();
    resource2.UseResource();
    
    // resource2 hala erişilebilir
    resource2.UseResource();
    
    // Metot sonunda resource2.Dispose() çağrılır
}
```

## 4. Finalizer Implementation (Sonlandırıcı Uygulaması)

Finalizer (yıkıcı), yönetilmeyen kaynakları temizlemek için bir güvenlik ağı sağlar.

```csharp
// Finalizer uygulaması
public class ResourceWithFinalizer : IDisposable
{
    private bool _disposed = false;
    private IntPtr _handle;
    
    public ResourceWithFinalizer()
    {
        _handle = Marshal.AllocHGlobal(100);
        Console.WriteLine($"Kaynak oluşturuldu: {_handle}");
    }
    
    // Finalizer
    ~ResourceWithFinalizer()
    {
        Console.WriteLine($"Finalizer çağrıldı: {_handle}");
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
                Console.WriteLine($"Yönetilen kaynaklar temizlendi: {_handle}");
            }
            
            // Yönetilmeyen kaynakları temizle
            if (_handle != IntPtr.Zero)
            {
                Marshal.FreeHGlobal(_handle);
                _handle = IntPtr.Zero;
                Console.WriteLine($"Yönetilmeyen kaynaklar temizlendi");
            }
            
            _disposed = true;
        }
    }
}

// Finalizer kullanım örneği
public void FinalizerExample()
{
    CreateAndForgetResource();
    
    // GC'yi zorla ve finalizer'ın çalışmasını bekle
    GC.Collect();
    GC.WaitForPendingFinalizers();
    
    Console.WriteLine("Finalizer tamamlandı");
}

private void CreateAndForgetResource()
{
    // Dispose edilmeyen kaynak
    var resource = new ResourceWithFinalizer();
    
    // Metot sonu - resource'a referans kaybolur, ancak Dispose çağrılmaz
    // Finalizer, GC tarafından çağrılacak
}
```

### Finalizer ve Dispose İlişkisi

Finalizer ve Dispose metodu arasındaki ilişki.

```csharp
// Finalizer ve Dispose ilişkisi
public class ResourceWithDisposeFinalizer : IDisposable
{
    private bool _disposed = false;
    private IntPtr _handle;
    
    public ResourceWithDisposeFinalizer()
    {
        _handle = Marshal.AllocHGlobal(100);
    }
    
    // Dispose metodu
    public void Dispose()
    {
        // Dispose(true) çağrılır - yönetilen ve yönetilmeyen kaynaklar temizlenir
        Dispose(true);
        
        // Finalizer'ı kuyruğundan çıkar - artık gerekli değil
        GC.SuppressFinalize(this);
    }
    
    // Korumalı Dispose metodu
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Yönetilen kaynakları temizle (Dispose(true) çağrıldığında)
                Console.WriteLine("Yönetilen kaynaklar temizlendi");
            }
            
            // Yönetilmeyen kaynakları temizle (hem Dispose(true) hem de finalizer için)
            if (_handle != IntPtr.Zero)
            {
                Marshal.FreeHGlobal(_handle);
                _handle = IntPtr.Zero;
                Console.WriteLine("Yönetilmeyen kaynaklar temizlendi");
            }
            
            _disposed = true;
        }
    }
    
    // Finalizer
    ~ResourceWithDisposeFinalizer()
    {
        // Dispose(false) çağrılır - sadece yönetilmeyen kaynaklar temizlenir
        Dispose(false);
    }
}

// Kullanım örneği
public void DisposeFinalizerExample()
{
    // Düzgün temizlenen kaynak
    using (var resource1 = new ResourceWithDisposeFinalizer())
    {
        // Kaynak kullanımı
    } // Dispose çağrılır, finalizer çalışmaz
    
    // Unutulan kaynak
    var resource2 = new ResourceWithDisposeFinalizer();
    // Dispose çağrılmaz, finalizer GC tarafından çağrılır
}
```

## 5. SafeHandle Classes (SafeHandle Sınıfları)

SafeHandle sınıfları, yönetilmeyen kaynakların güvenli bir şekilde yönetilmesini sağlar.

```csharp
// SafeHandle kullanımı
public class FileSafeHandle : IDisposable
{
    private SafeFileHandle _handle;
    
    public FileSafeHandle(string fileName, FileAccess access)
    {
        // SafeFileHandle, yönetilmeyen dosya tanıtıcısını güvenli bir şekilde sarar
        _handle = File.OpenHandle(fileName, access);
        Console.WriteLine("Dosya tanıtıcısı açıldı");
    }
    
    public void Dispose()
    {
        if (_handle != null && !_handle.IsInvalid)
        {
            _handle.Dispose();
            _handle = null;
            Console.WriteLine("Dosya tanıtıcısı kapatıldı");
        }
    }
    
    public void ReadData(byte[] buffer, long position)
    {
        if (_handle == null || _handle.IsInvalid)
        {
            throw new ObjectDisposedException(nameof(FileSafeHandle));
        }
        
        RandomAccess.Read(_handle, buffer, position);
    }
}

// SafeHandle kullanım örneği
public void SafeHandleExample()
{
    string fileName = "test.dat";
    
    // Dosya oluştur
    File.WriteAllText(fileName, "Test verisi");
    
    using (var fileHandle = new FileSafeHandle(fileName, FileAccess.Read))
    {
        byte[] buffer = new byte[10];
        fileHandle.ReadData(buffer, 0);
        
        string data = Encoding.UTF8.GetString(buffer).TrimEnd('\0');
        Console.WriteLine($"Okunan veri: {data}");
    }
}
```

### Özel SafeHandle Sınıfları

Özel SafeHandle sınıfları oluşturarak yönetilmeyen kaynakları güvenli bir şekilde yönetebilirsiniz.

```csharp
// Özel SafeHandle sınıfı
public class CustomSafeHandle : SafeHandle
{
    // SafeHandle sınıfından türetilen sınıflar, geçersiz tanıtıcı değerini belirtmelidir
    public CustomSafeHandle() : base(IntPtr.Zero, true)
    {
    }
    
    // Tanıtıcının geçersiz olup olmadığını belirten özellik
    public override bool IsInvalid => handle == IntPtr.Zero;
    
    // Yönetilmeyen kaynağı serbest bırakma
    protected override bool ReleaseHandle()
    {
        if (!IsInvalid)
        {
            // Yönetilmeyen API çağrısı ile kaynağı serbest bırak
            // Örnek: NativeMethods.CloseHandle(handle);
            Console.WriteLine("Yönetilmeyen kaynak serbest bırakıldı");
            
            // Tanıtıcıyı sıfırla
            handle = IntPtr.Zero;
            return true;
        }
        
        return false;
    }
    
    // Kaynak tahsisi
    public static CustomSafeHandle Allocate(int size)
    {
        CustomSafeHandle safeHandle = new CustomSafeHandle();
        
        // Yönetilmeyen bellek tahsisi
        safeHandle.handle = Marshal.AllocHGlobal(size);
        Console.WriteLine($"Yönetilmeyen bellek tahsis edildi: {safeHandle.handle}");
        
        return safeHandle;
    }
}

// Özel SafeHandle kullanımı
public void CustomSafeHandleExample()
{
    using (CustomSafeHandle handle = CustomSafeHandle.Allocate(100))
    {
        // Güvenli tanıtıcı kullanımı
        if (!handle.IsInvalid)
        {
            Console.WriteLine("Güvenli tanıtıcı kullanılıyor");
        }
    } // ReleaseHandle otomatik olarak çağrılır
}
```

## 6. Resource Management (Kaynak Yönetimi)

Kaynakları etkili bir şekilde yönetmek için stratejiler.

```csharp
// Kaynak yönetimi stratejileri
public class ResourceManager : IDisposable
{
    private readonly List<IDisposable> _resources = new List<IDisposable>();
    private bool _disposed = false;
    
    // Kaynak ekleme
    public void AddResource(IDisposable resource)
    {
        if (_disposed)
        {
            throw new ObjectDisposedException(nameof(ResourceManager));
        }
        
        _resources.Add(resource);
    }
    
    // Tüm kaynakları temizleme
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
                // Kaynakları ters sırada temizle (LIFO)
                for (int i = _resources.Count - 1; i >= 0; i--)
                {
                    _resources[i].Dispose();
                }
                
                _resources.Clear();
            }
            
            _disposed = true;
        }
    }
}

// Kaynak yöneticisi kullanımı
public void ResourceManagerExample()
{
    using (var manager = new ResourceManager())
    {
        // Kaynakları ekle
        manager.AddResource(new SimpleDisposable());
        manager.AddResource(new CompleteDisposable("managed.txt"));
        
        // Kaynakları kullan
        Console.WriteLine("Kaynaklar kullanılıyor");
        
        // Tüm kaynaklar otomatik olarak temizlenir
    }
}
```

### Kaynak Havuzları

Kaynak havuzları, sık kullanılan kaynakların yeniden kullanılmasını sağlar.

```csharp
// Kaynak havuzu
public class ResourcePool<T> : IDisposable where T : IDisposable, new()
{
    private readonly Queue<T> _pool = new Queue<T>();
    private readonly int _maxSize;
    private bool _disposed = false;
    
    public ResourcePool(int initialSize, int maxSize)
    {
        _maxSize = maxSize;
        
        // Havuzu başlangıç boyutuyla doldur
        for (int i = 0; i < initialSize; i++)
        {
            _pool.Enqueue(new T());
        }
    }
    
    // Havuzdan kaynak alma
    public T Get()
    {
        if (_disposed)
        {
            throw new ObjectDisposedException(nameof(ResourcePool<T>));
        }
        
        lock (_pool)
        {
            if (_pool.Count > 0)
            {
                return _pool.Dequeue();
            }
        }
        
        // Havuz boşsa yeni kaynak oluştur
        return new T();
    }
    
    // Kaynağı havuza geri verme
    public void Return(T resource)
    {
        if (_disposed)
        {
            // Havuz temizlendiyse, kaynağı da temizle
            resource.Dispose();
            return;
        }
        
        lock (_pool)
        {
            // Havuz maksimum boyuta ulaşmadıysa, kaynağı havuza ekle
            if (_pool.Count < _maxSize)
            {
                _pool.Enqueue(resource);
            }
            else
            {
                // Havuz doluysa, kaynağı temizle
                resource.Dispose();
            }
        }
    }
    
    // Havuzu temizleme
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
                lock (_pool)
                {
                    // Havuzdaki tüm kaynakları temizle
                    while (_pool.Count > 0)
                    {
                        T resource = _pool.Dequeue();
                        resource.Dispose();
                    }
                }
            }
            
            _disposed = true;
        }
    }
}

// Kaynak havuzu kullanımı
public void ResourcePoolExample()
{
    // SimpleDisposable nesneleri için havuz oluştur
    using (var pool = new ResourcePool<SimpleDisposable>(5, 10))
    {
        // Havuzdan kaynak al
        var resource1 = pool.Get();
        var resource2 = pool.Get();
        
        // Kaynakları kullan
        resource1.UseResource();
        resource2.UseResource();
        
        // Kaynakları havuza geri ver
        pool.Return(resource1);
        pool.Return(resource2);
        
        // Havuz temizlendiğinde tüm kaynaklar da temizlenir
    }
}
```

## 7. Cleanup Patterns (Temizleme Desenleri)

Farklı senaryolar için temizleme desenleri.

```csharp
// Temizleme desenleri
public class CleanupPatterns
{
    // 1. Erken çıkış temizleme
    public void EarlyExitCleanup(string filePath)
    {
        FileStream file = null;
        
        try
        {
            file = new FileStream(filePath, FileMode.Open);
            
            // Erken çıkış koşulu
            if (file.Length == 0)
            {
                // Erken çıkış öncesi temizleme
                file.Dispose();
                file = null;
                return;
            }
            
            // Normal işlem
            byte[] buffer = new byte[file.Length];
            file.Read(buffer, 0, buffer.Length);
        }
        finally
        {
            // Temizleme
            file?.Dispose();
        }
    }
    
    // 2. Koşullu temizleme
    public void ConditionalCleanup(IDisposable resource, bool shouldCleanup)
    {
        try
        {
            // Kaynak kullanımı
        }
        finally
        {
            if (shouldCleanup)
            {
                resource?.Dispose();
            }
        }
    }
    
    // 3. Çoklu kaynak temizleme
    public void MultipleResourceCleanup(string filePath1, string filePath2)
    {
        FileStream file1 = null;
        FileStream file2 = null;
        
        try
        {
            file1 = new FileStream(filePath1, FileMode.Open);
            file2 = new FileStream(filePath2, FileMode.Create);
            
            // Dosyalar arası kopyalama
            file1.CopyTo(file2);
        }
        finally
        {
            // Her iki kaynağı da temizle
            file1?.Dispose();
            file2?.Dispose();
        }
    }
    
    // 4. Hiyerarşik temizleme
    public void HierarchicalCleanup()
    {
        // Dış kaynak
        using (var outerResource = new SimpleDisposable())
        {
            try
            {
                // İç kaynak
                using (var innerResource = new CompleteDisposable("inner.txt"))
                {
                    // Kaynakları kullan
                }
                // İç kaynak burada temizlenir
            }
            catch (Exception)
            {
                // Hata işleme
                throw;
            }
            // Dış kaynak burada temizlenir
        }
    }
}
```

### Asenkron Temizleme

Asenkron işlemlerde kaynak temizleme.

```csharp
// Asenkron temizleme
public class AsyncCleanup
{
    // Asenkron Dispose deseni
    public class AsyncDisposable : IDisposable, IAsyncDisposable
    {
        private bool _disposed = false;
        private FileStream _fileStream;
        
        public AsyncDisposable(string filePath)
        {
            _fileStream = new FileStream(filePath, FileMode.Create);
        }
        
        // Senkron Dispose
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
                    _fileStream?.Dispose();
                }
                
                _disposed = true;
            }
        }
        
        // Asenkron Dispose
        public async ValueTask DisposeAsync()
        {
            if (!_disposed)
            {
                if (_fileStream != null)
                {
                    await _fileStream.DisposeAsync();
                }
                
                _disposed = true;
            }
            
            GC.SuppressFinalize(this);
        }
    }
    
    // Asenkron using kullanımı
    public async Task AsyncUsingExample()
    {
        // await using ifadesi
        await using (var resource = new AsyncDisposable("async.txt"))
        {
            // Asenkron işlemler
            await Task.Delay(100);
        } // DisposeAsync otomatik olarak çağrılır
        
        // await using bildirimi
        await using var resource2 = new AsyncDisposable("async2.txt");
        
        // Asenkron işlemler
        await Task.Delay(100);
        
        // Metot sonunda resource2.DisposeAsync() çağrılır
    }
}
```

## Özet

Bu bölümde, .NET'teki IDisposable deseni ve ilgili konuları inceledik:

1. **Dispose Pattern Implementation**: IDisposable arabirimini uygulayarak kaynakları temizleme.

2. **Using Statement**: IDisposable nesnelerinin otomatik olarak temizlenmesini sağlayan using ifadesi.

3. **Using Declaration**: C# 8.0 ile gelen daha temiz bir using sözdizimi.

4. **Finalizer Implementation**: Yönetilmeyen kaynakları temizlemek için güvenlik ağı sağlayan finalizer.

5. **SafeHandle Classes**: Yönetilmeyen kaynakların güvenli bir şekilde yönetilmesini sağlayan sınıflar.

6. **Resource Management**: Kaynakları etkili bir şekilde yönetmek için stratejiler.

7. **Cleanup Patterns**: Farklı senaryolar için temizleme desenleri.

IDisposable deseni, .NET uygulamalarında kaynakların doğru şekilde yönetilmesi için kritik öneme sahiptir. Bu deseni doğru bir şekilde uygulamak, bellek sızıntılarını önler ve uygulamanızın daha güvenilir olmasını sağlar. 