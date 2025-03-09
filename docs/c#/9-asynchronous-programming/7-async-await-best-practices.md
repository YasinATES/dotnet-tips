# Async/Await Best Practices (Async/Await En İyi Uygulamalar)

Asenkron programlama, modern C# uygulamalarının önemli bir parçasıdır. Ancak, yanlış kullanıldığında performans sorunlarına, bellek sızıntılarına ve hatalara neden olabilir. Bu bölümde, async/await kullanırken dikkat edilmesi gereken en iyi uygulamaları ve yaygın hataları inceleyeceğiz.

## 1. Async All the Way (Tamamen Asenkron)

Asenkron kodun en önemli kurallarından biri, "async all the way" (tamamen asenkron) prensibidir.

```csharp
// YANLIŞ: Asenkron zinciri kırma
public string GetDataWrong()
{
    // Asenkron metodu senkron olarak bekleme - deadlock riski!
    return GetDataAsync().Result; // veya .Wait() kullanımı
}

// DOĞRU: Asenkron zinciri koruma
public async Task<string> GetDataRight()
{
    // Asenkron metodu asenkron olarak bekleme
    return await GetDataAsync();
}

// Örnek asenkron veri alma metodu
private async Task<string> GetDataAsync()
{
    using var client = new HttpClient();
    return await client.GetStringAsync("https://api.example.com/data");
}
```

### Deadlock Tehlikesi

UI veya ASP.NET gibi SynchronizationContext kullanan ortamlarda `.Result` veya `.Wait()` kullanmak deadlock'a neden olabilir.

```csharp
// Deadlock örneği (UI uygulamasında)
private void Button_Click(object sender, EventArgs e)
{
    // YANLIŞ: UI thread'inde deadlock'a neden olur
    var result = GetDataAsync().Result;
    
    // DOĞRU: Asenkron metodu çağırma
    GetDataAndUpdateUIAsync();
}

private async Task GetDataAndUpdateUIAsync()
{
    var result = await GetDataAsync();
    UpdateUI(result);
}
```

## 2. ConfigureAwait Kullanımı

UI veya ASP.NET uygulamalarında, `ConfigureAwait(false)` kullanarak gereksiz bağlam değişikliklerinden kaçınabilirsiniz.

```csharp
// Kütüphane kodu için ConfigureAwait(false) kullanımı
public async Task<string> LibraryMethodAsync()
{
    // Orijinal bağlama dönmeye gerek olmayan kütüphane kodu
    using var client = new HttpClient();
    
    // ConfigureAwait(false) kullanarak bağlam yakalamayı önleme
    var response = await client.GetAsync("https://api.example.com/data")
        .ConfigureAwait(false);
    
    var content = await response.Content.ReadAsStringAsync()
        .ConfigureAwait(false);
    
    return content;
}

// UI uygulaması için ConfigureAwait(true) (varsayılan) kullanımı
public async Task UpdateUIAsync()
{
    // UI bağlamına dönmesi gereken kod
    var data = await GetDataAsync(); // ConfigureAwait(false) KULLANMAYIN
    
    // Bu kod UI thread'inde çalışmalı
    textBox.Text = data;
}
```

### ConfigureAwait Kullanım Kuralları

- **Kütüphane kodunda**: Her zaman `ConfigureAwait(false)` kullanın
- **Uygulama kodunda**: UI veya ASP.NET Controller'larında `ConfigureAwait(false)` kullanmaktan kaçının
- **Performans kritik kodda**: Bağlam değişikliği maliyetini azaltmak için `ConfigureAwait(false)` kullanın

## 3. Async Void Kullanımından Kaçınma

`async void` metotları, istisnaları yakalamak zor olduğu için tehlikelidir.

```csharp
// YANLIŞ: async void kullanımı
public async void ProcessDataVoid()
{
    try
    {
        await ProcessDataAsync();
    }
    catch (Exception ex)
    {
        // Bu istisna yakalanmayabilir ve uygulamanın çökmesine neden olabilir
        Console.WriteLine($"Hata: {ex.Message}");
    }
}

// DOĞRU: async Task kullanımı
public async Task ProcessDataTask()
{
    await ProcessDataAsync();
}

// Olay işleyicileri için async void kullanımı (tek istisna)
private async void Button_Click(object sender, EventArgs e)
{
    try
    {
        await ProcessDataAsync();
        UpdateUI("İşlem tamamlandı");
    }
    catch (Exception ex)
    {
        UpdateUI($"Hata: {ex.Message}");
    }
}
```

## 4. Asenkron Metot İsimlendirme

Asenkron metotlar için tutarlı isimlendirme kuralları kullanın.

```csharp
// DOĞRU: Asenkron metot isimlendirme
public async Task<string> GetDataAsync()
{
    // Asenkron işlem
    return await FetchDataFromApiAsync();
}

// YANLIŞ: Async son eki olmayan asenkron metot
public async Task<string> GetData() // Async son eki eksik
{
    return await FetchDataFromApiAsync();
}

// YANLIŞ: Async son eki olan senkron metot
public string GetDataAsync() // Async son eki var ama metot asenkron değil
{
    return "Veri";
}
```

## 5. Task.Run Kullanımı

UI uygulamalarında uzun süren işlemleri arka plana taşımak için `Task.Run` kullanabilirsiniz, ancak ASP.NET uygulamalarında dikkatli olun.

```csharp
// UI uygulamasında Task.Run kullanımı
private async void Button_Click(object sender, EventArgs e)
{
    // UI thread'ini bloke etmeden uzun süren işlemi arka plana taşıma
    var result = await Task.Run(() => 
    {
        // Uzun süren CPU-bound işlem
        return ProcessLargeData();
    });
    
    // UI güncellemesi
    UpdateUI(result);
}

// YANLIŞ: ASP.NET'te gereksiz Task.Run kullanımı
[HttpGet]
public async Task<IActionResult> GetDataWrong()
{
    // ASP.NET zaten bir thread havuzu kullanır, gereksiz Task.Run
    var result = await Task.Run(() => ProcessData());
    return Ok(result);
}

// DOĞRU: ASP.NET'te I/O-bound işlemler için doğrudan await kullanımı
[HttpGet]
public async Task<IActionResult> GetDataRight()
{
    var result = await GetDataFromDatabaseAsync();
    return Ok(result);
}
```

## 6. Asenkron Lambda İfadeleri

Lambda ifadelerinde asenkron kod kullanırken dikkatli olun.

```csharp
// YANLIŞ: Asenkron lambda ifadesi ile ForEach kullanımı
public async Task ProcessItemsWrong(List<string> items)
{
    // Bu kod beklediğiniz gibi çalışmaz - tüm görevler hemen başlatılır ve beklenmez
    items.ForEach(async item =>
    {
        await ProcessItemAsync(item);
    });
    
    // Bu noktada, görevler hala çalışıyor olabilir!
}

// DOĞRU: Asenkron işlemler için Task.WhenAll kullanımı
public async Task ProcessItemsRight(List<string> items)
{
    var tasks = items.Select(item => ProcessItemAsync(item));
    await Task.WhenAll(tasks);
    
    // Bu noktada, tüm görevler tamamlanmıştır
}

// Alternatif: Sıralı işleme
public async Task ProcessItemsSequentially(List<string> items)
{
    foreach (var item in items)
    {
        await ProcessItemAsync(item);
    }
}
```

## 7. İptal Belirteçleri Kullanma

Uzun süren asenkron işlemleri iptal edebilmek için her zaman iptal belirteçleri kullanın.

```csharp
// İptal belirteçleri ile asenkron işlem
public async Task<string> GetDataWithCancellationAsync(CancellationToken cancellationToken)
{
    using var client = new HttpClient();
    
    // İptal belirtecini HttpClient'a iletme
    var response = await client.GetAsync("https://api.example.com/data", cancellationToken);
    
    // İptal belirtecini manuel kontrol etme
    cancellationToken.ThrowIfCancellationRequested();
    
    return await response.Content.ReadAsStringAsync();
}

// Kullanım örneği
public async Task CallWithTimeoutAsync()
{
    // 5 saniye zaman aşımı ile iptal kaynağı oluşturma
    using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
    
    try
    {
        var result = await GetDataWithCancellationAsync(cts.Token);
        Console.WriteLine(result);
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("İşlem zaman aşımına uğradı veya iptal edildi.");
    }
}
```

## 8. ValueTask Kullanımı

Bazı durumlarda, `Task` yerine `ValueTask` kullanarak performansı artırabilirsiniz.

```csharp
// Önbellek kullanarak ValueTask döndürme
private Dictionary<string, string> _cache = new Dictionary<string, string>();

public ValueTask<string> GetCachedDataAsync(string key)
{
    // Önbellekte varsa, hemen değer döndür (Task oluşturmadan)
    if (_cache.TryGetValue(key, out string cachedValue))
    {
        return new ValueTask<string>(cachedValue);
    }
    
    // Önbellekte yoksa, asenkron olarak al
    return new ValueTask<string>(FetchAndCacheDataAsync(key));
}

private async Task<string> FetchAndCacheDataAsync(string key)
{
    // Veriyi asenkron olarak alma
    string data = await FetchDataFromSourceAsync(key);
    
    // Önbelleğe ekleme
    _cache[key] = data;
    
    return data;
}
```

### ValueTask Kullanım Kuralları

- ValueTask'ı birden fazla kez await etmeyin
- ValueTask'ı değişkenlerde saklamaktan kaçının
- Önbellek kullanımı veya sık sık senkron tamamlanan işlemler için kullanın

## 9. Asenkron Akışlar (IAsyncEnumerable)

C# 8.0 ve sonrasında, büyük veri kümeleri üzerinde çalışırken `IAsyncEnumerable<T>` kullanabilirsiniz.

```csharp
// IAsyncEnumerable kullanımı
public async IAsyncEnumerable<string> GetDataStreamAsync([EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    using var connection = new SqlConnection(_connectionString);
    await connection.OpenAsync(cancellationToken);
    
    using var command = new SqlCommand("SELECT Data FROM LargeTable", connection);
    using var reader = await command.ExecuteReaderAsync(cancellationToken);
    
    while (await reader.ReadAsync(cancellationToken))
    {
        // Her öğeyi okudukça yield etme
        yield return reader.GetString(0);
        
        // İşlem arası (isteğe bağlı)
        await Task.Delay(10, cancellationToken);
    }
}

// Kullanım örneği
public async Task ProcessDataStreamAsync()
{
    await foreach (var item in GetDataStreamAsync())
    {
        await ProcessItemAsync(item);
    }
}
```

## 10. Yaygın Hatalar ve Çözümleri

### 10.1. Asenkron Boilerplate Kodu

Tekrarlayan try-catch blokları için yardımcı metotlar kullanın.

```csharp
// Asenkron işlemleri sarmalayan yardımcı metot
public static async Task<Result<T>> TryAsync<T>(Func<Task<T>> asyncFunc)
{
    try
    {
        T result = await asyncFunc();
        return Result<T>.Success(result);
    }
    catch (Exception ex)
    {
        return Result<T>.Failure(ex);
    }
}

// Kullanım örneği
public async Task<IActionResult> GetUserData(int userId)
{
    var result = await TryAsync(() => _userService.GetUserDataAsync(userId));
    
    if (result.IsSuccess)
    {
        return Ok(result.Value);
    }
    
    return StatusCode(500, result.Error.Message);
}

// Sonuç sınıfı
public class Result<T>
{
    public bool IsSuccess { get; }
    public T Value { get; }
    public Exception Error { get; }
    
    private Result(bool isSuccess, T value, Exception error)
    {
        IsSuccess = isSuccess;
        Value = value;
        Error = error;
    }
    
    public static Result<T> Success(T value) => new Result<T>(true, value, null);
    public static Result<T> Failure(Exception error) => new Result<T>(false, default, error);
}
```

### 10.2. Asenkron Metotlarda Dispose Kullanımı

Asenkron metotlarda `IAsyncDisposable` ve `await using` kullanın.

```csharp
// DOĞRU: IAsyncDisposable kullanımı
public async Task ProcessDataAsync()
{
    await using var resource = new AsyncResource();
    await resource.ProcessAsync();
}

// IAsyncDisposable implementasyonu
public class AsyncResource : IAsyncDisposable
{
    public async Task ProcessAsync()
    {
        // Asenkron işlem
        await Task.Delay(100);
    }
    
    public async ValueTask DisposeAsync()
    {
        // Asenkron temizlik
        await CleanupAsync();
    }
    
    private async Task CleanupAsync()
    {
        // Kaynakları temizleme
        await Task.Delay(50);
    }
}
```

### 10.3. Asenkron Metotlarda Yerel Değişkenler

Asenkron metotlarda yerel değişkenlerin kapsamına dikkat edin.

```csharp
// YANLIŞ: Asenkron metotta yerel değişken kapsamı sorunu
public async Task ProcessWithScopeIssueAsync()
{
    var resource = new DisposableResource();
    
    try
    {
        await Task.Delay(100);
        // resource kullanımı
    }
    finally
    {
        // Bu noktada resource hala erişilebilir
        resource.Dispose();
    }
    
    // Bu noktada resource kullanılmamalı
}

// DOĞRU: using bloğu ile kapsam kontrolü
public async Task ProcessWithProperScopeAsync()
{
    using (var resource = new DisposableResource())
    {
        await Task.Delay(100);
        // resource kullanımı
    } // resource burada otomatik olarak dispose edilir
    
    // Bu noktada resource kullanılamaz
}
```

### 10.4. Asenkron Metotlarda Paralel İşlemler

Asenkron metotlarda paralel işlemleri doğru şekilde yönetin.

```csharp
// YANLIŞ: Sınırsız paralel işlem
public async Task ProcessItemsUnboundedAsync(List<string> items)
{
    // Çok fazla paralel görev oluşturabilir ve sistem kaynaklarını tüketebilir
    var tasks = items.Select(item => ProcessItemAsync(item));
    await Task.WhenAll(tasks);
}

// DOĞRU: Sınırlı paralel işlem
public async Task ProcessItemsBoundedAsync(List<string> items)
{
    // Paralel işlem sayısını sınırlama
    int maxParallelism = Environment.ProcessorCount;
    
    // SemaphoreSlim ile paralel işlem sayısını sınırlama
    using var semaphore = new SemaphoreSlim(maxParallelism);
    var tasks = new List<Task>();
    
    foreach (var item in items)
    {
        await semaphore.WaitAsync();
        
        tasks.Add(Task.Run(async () =>
        {
            try
            {
                await ProcessItemAsync(item);
            }
            finally
            {
                semaphore.Release();
            }
        }));
    }
    
    await Task.WhenAll(tasks);
}
```

## 11. Performans İpuçları

### 11.1. Gereksiz Async/Await Kullanımından Kaçınma

Bazı durumlarda, async/await kullanmak gereksiz olabilir.

```csharp
// YANLIŞ: Gereksiz async/await
public async Task<int> GetValueUnnecessaryAsync()
{
    return await Task.FromResult(42);
}

// DOĞRU: Doğrudan Task döndürme
public Task<int> GetValueBetterAsync()
{
    return Task.FromResult(42);
}

// YANLIŞ: Zaten tamamlanmış Task için await kullanma
public async Task<string> GetCachedDataUnnecessaryAsync(string key)
{
    if (_cache.TryGetValue(key, out string value))
    {
        // Gereksiz await - zaten senkron olarak tamamlanmış
        return await Task.FromResult(value);
    }
    
    return await FetchDataAsync(key);
}

// DOĞRU: Zaten tamamlanmış değer için doğrudan dönüş
public Task<string> GetCachedDataBetterAsync(string key)
{
    if (_cache.TryGetValue(key, out string value))
    {
        return Task.FromResult(value);
    }
    
    return FetchDataAsync(key);
}
```

### 11.2. Task Tamamlanma Kontrolü

Bir Task'ın zaten tamamlanıp tamamlanmadığını kontrol etmek için `IsCompleted` özelliğini kullanabilirsiniz.

```csharp
// Task tamamlanma kontrolü
public async Task<string> GetDataWithCompletionCheckAsync()
{
    Task<string> dataTask = FetchDataAsync();
    
    // Task zaten tamamlandıysa, await kullanmadan sonucu al
    if (dataTask.IsCompleted && !dataTask.IsFaulted && !dataTask.IsCanceled)
    {
        return dataTask.Result; // Bu noktada güvenli, çünkü task tamamlandı
    }
    
    // Tamamlanmadıysa, await kullanarak bekle
    return await dataTask;
}
```

### 11.3. ValueTask ve Task Havuzlama

Sık çağrılan asenkron metotlarda Task havuzlama veya ValueTask kullanarak performansı artırabilirsiniz.

```csharp
// ValueTask kullanımı
public ValueTask<int> GetValueAsync()
{
    if (_cachedValue.HasValue)
    {
        return new ValueTask<int>(_cachedValue.Value);
    }
    
    return new ValueTask<int>(FetchValueAsync());
}

// Task havuzlama
private static readonly Task<int> _zeroTask = Task.FromResult(0);
private static readonly Task<int> _oneTask = Task.FromResult(1);

public Task<int> GetCommonValueAsync(int value)
{
    // Sık kullanılan değerler için önceden oluşturulmuş Task'ları kullanma
    if (value == 0) return _zeroTask;
    if (value == 1) return _oneTask;
    
    // Diğer değerler için yeni Task oluşturma
    return Task.FromResult(value);
}
```

## Özet

Bu bölümde, async/await kullanırken dikkat edilmesi gereken en iyi uygulamaları ve yaygın hataları inceledik:

1. **Async All the Way**: Asenkron zinciri koruyun, senkron bloklamadan kaçının.
2. **ConfigureAwait Kullanımı**: Kütüphane kodunda `ConfigureAwait(false)` kullanın.
3. **Async Void Kullanımından Kaçınma**: Olay işleyicileri dışında async void kullanmayın.
4. **Asenkron Metot İsimlendirme**: Tutarlı isimlendirme kuralları kullanın.
5. **Task.Run Kullanımı**: UI uygulamalarında uzun süren işlemler için kullanın.
6. **Asenkron Lambda İfadeleri**: ForEach ile asenkron lambda kullanmaktan kaçının.
7. **İptal Belirteçleri Kullanma**: Uzun süren işlemleri iptal edebilmek için iptal belirteçleri kullanın.
8. **ValueTask Kullanımı**: Önbellek kullanımı veya sık sık senkron tamamlanan işlemler için ValueTask kullanın.
9. **Asenkron Akışlar**: Büyük veri kümeleri için IAsyncEnumerable kullanın.
10. **Yaygın Hatalar ve Çözümleri**: Asenkron boilerplate kodu, dispose kullanımı, yerel değişken kapsamı ve paralel işlemler.
11. **Performans İpuçları**: Gereksiz async/await kullanımından kaçınma, Task tamamlanma kontrolü ve Task havuzlama.

Asenkron programlama, doğru kullanıldığında uygulamanızın daha duyarlı ve verimli olmasını sağlar. Ancak, yanlış kullanıldığında performans sorunlarına ve hatalara neden olabilir. Bu en iyi uygulamaları takip ederek, asenkron kodunuzu daha güvenilir ve verimli hale getirebilirsiniz. 