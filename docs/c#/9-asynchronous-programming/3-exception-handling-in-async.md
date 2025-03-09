# Exception Handling in Async (Asenkron Programlamada Hata İşleme)

Asenkron programlamada hata işleme, senkron programlamaya göre bazı farklılıklar gösterir. Bu bölümde, asenkron metotlarda hataların nasıl yakalanacağını, işleneceğini ve yayılacağını inceleyeceğiz.

## 1. Try-Catch-Finally

Asenkron metotlarda, senkron metotlardakine benzer şekilde `try-catch-finally` blokları kullanılabilir.

```csharp
// Temel try-catch-finally kullanımı
public async Task<string> FetchDataAsync(string url)
{
    try
    {
        // Potansiyel olarak hata fırlatabilecek asenkron işlem
        using (HttpClient client = new HttpClient())
        {
            string result = await client.GetStringAsync(url);
            return result;
        }
    }
    catch (HttpRequestException ex)
    {
        // Ağ hatası
        Console.WriteLine($"Ağ hatası: {ex.Message}");
        throw; // Hatayı yeniden fırlat
    }
    catch (Exception ex)
    {
        // Diğer hatalar
        Console.WriteLine($"Beklenmeyen hata: {ex.Message}");
        throw new ApplicationException("Veri alınamadı.", ex);
    }
    finally
    {
        // Temizlik işlemleri
        Console.WriteLine("İşlem tamamlandı (başarılı veya başarısız).");
    }
}

// Kullanım örneği
public async Task TryFetchDataExample()
{
    try
    {
        string data = await FetchDataAsync("https://api.example.com/data");
        Console.WriteLine($"Alınan veri: {data.Substring(0, 50)}...");
    }
    catch (ApplicationException ex)
    {
        Console.WriteLine($"Uygulama hatası: {ex.Message}");
        if (ex.InnerException != null)
        {
            Console.WriteLine($"İç hata: {ex.InnerException.Message}");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Beklenmeyen hata: {ex.Message}");
    }
}
```

### await Operatörü ve Hata Yayılımı

`await` operatörü, bir görevin tamamlanmasını beklerken, görevde oluşan hataları otomatik olarak çağıran metoda yayar.

```csharp
// await operatörü ile hata yayılımı
public async Task ProcessDataAsync()
{
    try
    {
        // İç içe asenkron çağrılar
        string data = await FetchDataAsync("https://api.example.com/data");
        var processedData = await TransformDataAsync(data);
        await SaveDataAsync(processedData);
        
        Console.WriteLine("Veri işleme tamamlandı.");
    }
    catch (HttpRequestException ex)
    {
        // FetchDataAsync'den yayılan ağ hatası
        Console.WriteLine($"Veri alınamadı: {ex.Message}");
    }
    catch (DataTransformException ex)
    {
        // TransformDataAsync'den yayılan dönüşüm hatası
        Console.WriteLine($"Veri dönüştürülemedi: {ex.Message}");
    }
    catch (DataStorageException ex)
    {
        // SaveDataAsync'den yayılan depolama hatası
        Console.WriteLine($"Veri kaydedilemedi: {ex.Message}");
    }
    catch (Exception ex)
    {
        // Diğer tüm hatalar
        Console.WriteLine($"İşlem hatası: {ex.Message}");
    }
}

// Veri dönüştürme
private async Task<byte[]> TransformDataAsync(string data)
{
    await Task.Delay(100); // Simüle edilmiş işlem
    
    if (string.IsNullOrEmpty(data))
        throw new DataTransformException("Dönüştürülecek veri boş.");
    
    return System.Text.Encoding.UTF8.GetBytes(data);
}

// Veri kaydetme
private async Task SaveDataAsync(byte[] data)
{
    await Task.Delay(100); // Simüle edilmiş işlem
    
    if (data == null || data.Length == 0)
        throw new DataStorageException("Kaydedilecek veri boş.");
    
    // Dosyaya kaydetme simülasyonu
    await File.WriteAllBytesAsync("data.bin", data);
}

// Özel istisna sınıfları
public class DataTransformException : Exception
{
    public DataTransformException(string message) : base(message) { }
    public DataTransformException(string message, Exception inner) : base(message, inner) { }
}

public class DataStorageException : Exception
{
    public DataStorageException(string message) : base(message) { }
    public DataStorageException(string message, Exception inner) : base(message, inner) { }
}
```

## 2. AggregateException

`Task.Wait()` veya `Task.Result` kullanıldığında, asenkron görevlerde oluşan hatalar bir `AggregateException` içinde sarmalanır.

```csharp
// AggregateException örneği
public void HandleAggregateExceptionExample()
{
    try
    {
        // Asenkron görevi senkron olarak bekleme (bu yaklaşım önerilmez)
        Task<string> task = FetchDataAsync("https://api.example.com/data");
        string result = task.Result; // AggregateException fırlatabilir
        
        Console.WriteLine($"Sonuç: {result}");
    }
    catch (AggregateException ex)
    {
        // AggregateException içindeki iç istisnaları işleme
        foreach (var innerEx in ex.InnerExceptions)
        {
            if (innerEx is HttpRequestException)
            {
                Console.WriteLine($"Ağ hatası: {innerEx.Message}");
            }
            else
            {
                Console.WriteLine($"Diğer hata: {innerEx.Message}");
            }
        }
    }
}

// Birden fazla görevde hata işleme
public async Task HandleMultipleTasksExample()
{
    var urls = new[]
    {
        "https://api.example.com/data1",
        "https://api.example.com/data2",
        "https://api.example.com/data3"
    };
    
    var tasks = urls.Select(url => FetchDataAsync(url)).ToArray();
    
    try
    {
        // Tüm görevleri paralel olarak çalıştırma
        string[] results = await Task.WhenAll(tasks);
        
        // Tüm sonuçları işleme
        for (int i = 0; i < results.Length; i++)
        {
            Console.WriteLine($"URL {i+1} sonucu: {results[i].Substring(0, 20)}...");
        }
    }
    catch (Exception ex)
    {
        // Task.WhenAll, ilk karşılaşılan hatayı fırlatır
        Console.WriteLine($"Hata: {ex.Message}");
        
        // Tüm görevleri kontrol etme
        for (int i = 0; i < tasks.Length; i++)
        {
            if (tasks[i].IsFaulted)
            {
                // Her görevin hatasını ayrı ayrı işleme
                Exception taskEx = tasks[i].Exception?.InnerException;
                Console.WriteLine($"URL {i+1} hatası: {taskEx?.Message}");
            }
            else if (tasks[i].IsCompleted)
            {
                Console.WriteLine($"URL {i+1} başarıyla tamamlandı.");
            }
        }
    }
}
```

### AggregateException.Flatten

`AggregateException.Flatten()` metodu, iç içe geçmiş `AggregateException` nesnelerini düzleştirerek tüm iç istisnaları tek bir seviyede toplar.

```csharp
// AggregateException.Flatten kullanımı
public void FlattenAggregateExceptionExample()
{
    var task1 = Task.Run(() => { throw new InvalidOperationException("Hata 1"); });
    var task2 = Task.Run(() => { throw new ArgumentException("Hata 2"); });
    var task3 = Task.Run(() => { throw new NullReferenceException("Hata 3"); });
    
    try
    {
        // Tüm görevleri bekleme (senkron)
        Task.WaitAll(task1, task2, task3);
    }
    catch (AggregateException ex)
    {
        // AggregateException'ı düzleştirme
        AggregateException flattened = ex.Flatten();
        
        Console.WriteLine($"Toplam {flattened.InnerExceptions.Count} hata oluştu:");
        
        foreach (var innerEx in flattened.InnerExceptions)
        {
            Console.WriteLine($"- {innerEx.GetType().Name}: {innerEx.Message}");
        }
    }
}
```

## 3. Task Exception Propagation (Task İstisna Yayılımı)

Asenkron görevlerde oluşan istisnalar, görevin durumunu `Faulted` olarak işaretler ve `Task.Exception` özelliğinde saklanır.

```csharp
// Task istisna yayılımı
public async Task ExceptionPropagationExample()
{
    // Hata fırlatan bir görev oluşturma
    Task faultedTask = Task.Run(() =>
    {
        throw new InvalidOperationException("Görev hatası!");
    });
    
    try
    {
        // await kullanarak hatayı yakalama
        await faultedTask;
    }
    catch (InvalidOperationException ex)
    {
        // await, AggregateException'ı açar ve iç istisnayı fırlatır
        Console.WriteLine($"Yakalanan hata: {ex.Message}");
    }
    
    // Görevin durumunu kontrol etme
    if (faultedTask.IsFaulted)
    {
        Console.WriteLine("Görev hatalı durumda.");
        
        // Task.Exception bir AggregateException döndürür
        AggregateException aggEx = faultedTask.Exception;
        
        if (aggEx != null)
        {
            foreach (var innerEx in aggEx.InnerExceptions)
            {
                Console.WriteLine($"İç hata: {innerEx.Message}");
            }
        }
    }
}

// ContinueWith ile hata işleme
public Task ContinueWithExceptionHandlingExample()
{
    return Task.Run(() =>
    {
        // Potansiyel olarak hata fırlatan bir işlem
        throw new InvalidOperationException("İşlem hatası!");
    })
    .ContinueWith(task =>
    {
        if (task.IsFaulted)
        {
            // Görev hata verdiğinde çalışır
            Exception ex = task.Exception?.InnerException;
            Console.WriteLine($"Hata işlendi: {ex?.Message}");
            return "Hata durumu";
        }
        else
        {
            // Görev başarıyla tamamlandığında çalışır
            return "Başarı durumu";
        }
    });
}
```

## 4. UnobservedTaskException

Bir görevde oluşan istisna yakalanmazsa ve görev çöp toplayıcı tarafından temizlenirse, `TaskScheduler.UnobservedTaskException` olayı tetiklenir.

```csharp
// UnobservedTaskException örneği
public void UnobservedTaskExceptionExample()
{
    // Gözlemlenmeyen görev istisnalarını yakalama
    TaskScheduler.UnobservedTaskException += (sender, e) =>
    {
        Console.WriteLine($"Gözlemlenmeyen görev istisnası: {e.Exception.Message}");
        
        // İstisnayı işlenmiş olarak işaretleme
        e.SetObserved();
    };
    
    // Hata fırlatan bir görev oluşturma ve referansını kaybetme
    CreateFaultedTaskAndForget();
    
    // Çöp toplayıcıyı zorla çalıştırma (gerçek uygulamalarda yapılmamalıdır)
    GC.Collect();
    GC.WaitForPendingFinalizers();
}

private void CreateFaultedTaskAndForget()
{
    // Hata fırlatan bir görev oluşturma
    Task.Run(() =>
    {
        throw new InvalidOperationException("Gözlemlenmeyen hata!");
    });
    
    // Görev referansı burada kayboluyor ve hata yakalanmıyor
}
```

### ConfigureAwait ve İstisna Bağlamı

`ConfigureAwait(false)` kullanıldığında, istisnalar orijinal bağlam olmadan yayılır.

```csharp
// ConfigureAwait ve istisna bağlamı
public async Task ConfigureAwaitExceptionExample()
{
    try
    {
        // ConfigureAwait(false) ile bağlam yakalamayı devre dışı bırakma
        await Task.Run(() =>
        {
            throw new InvalidOperationException("Bağlamsız hata!");
        }).ConfigureAwait(false);
    }
    catch (InvalidOperationException ex)
    {
        // Bu catch bloğu çalışır, ancak orijinal bağlamda olmayabilir
        Console.WriteLine($"Yakalanan hata: {ex.Message}");
        
        // UI uygulamasında, bu noktada UI thread'inde olmayabiliriz
        // Bu nedenle, UI güncellemeleri için Dispatcher veya SynchronizationContext kullanılmalıdır
    }
}
```

## 5. Exception Filters (İstisna Filtreleri)

C# 6.0 ve sonrasında, `when` anahtar kelimesi ile istisna filtreleri kullanılabilir.

```csharp
// İstisna filtreleri
public async Task ExceptionFiltersExample()
{
    try
    {
        await ProcessTransactionAsync(100);
    }
    catch (TransactionException ex) when (ex.ErrorCode == "INSUFFICIENT_FUNDS")
    {
        Console.WriteLine("Yetersiz bakiye hatası.");
    }
    catch (TransactionException ex) when (ex.ErrorCode == "ACCOUNT_LOCKED")
    {
        Console.WriteLine("Hesap kilitli hatası.");
    }
    catch (TransactionException ex)
    {
        Console.WriteLine($"Diğer işlem hatası: {ex.ErrorCode}");
    }
    catch (Exception ex) when (IsRecoverable(ex))
    {
        Console.WriteLine("Kurtarılabilir hata: Yeniden deneniyor...");
        await RetryOperationAsync();
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Kurtarılamaz hata: {ex.Message}");
    }
}

private async Task ProcessTransactionAsync(decimal amount)
{
    await Task.Delay(100); // Simüle edilmiş işlem
    
    if (amount > 1000)
        throw new TransactionException("İşlem limiti aşıldı.", "LIMIT_EXCEEDED");
    else if (amount < 0)
        throw new TransactionException("Geçersiz tutar.", "INVALID_AMOUNT");
    else if (amount > 500)
        throw new TransactionException("Yetersiz bakiye.", "INSUFFICIENT_FUNDS");
    
    // İşlem başarılı
    Console.WriteLine($"{amount:C} tutarında işlem tamamlandı.");
}

private bool IsRecoverable(Exception ex)
{
    // Kurtarılabilir hataları belirleme mantığı
    return ex is TimeoutException || 
           ex is HttpRequestException || 
           (ex.Message?.Contains("geçici") ?? false);
}

private async Task RetryOperationAsync()
{
    // Yeniden deneme mantığı
    await Task.Delay(1000);
    Console.WriteLine("İşlem yeniden deneniyor...");
}

// Özel işlem istisnası
public class TransactionException : Exception
{
    public string ErrorCode { get; }
    
    public TransactionException(string message, string errorCode) 
        : base(message)
    {
        ErrorCode = errorCode;
    }
}
```

### İstisna Filtreleri ile Günlükleme

İstisna filtreleri, istisnaları yakalamadan günlükleme yapmak için de kullanılabilir.

```csharp
// İstisna filtreleri ile günlükleme
public async Task LoggingWithExceptionFiltersExample()
{
    try
    {
        await FetchDataAsync("https://api.example.com/data");
    }
    catch (Exception ex) when (LogException(ex))
    {
        // Bu catch bloğu asla çalışmaz, çünkü LogException her zaman false döner
        // Ancak, istisna günlüklenir
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"Ağ hatası: {ex.Message}");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Diğer hata: {ex.Message}");
    }
}

private bool LogException(Exception ex)
{
    // İstisnayı günlükleme
    Console.WriteLine($"[LOG] {DateTime.Now}: {ex.GetType().Name} - {ex.Message}");
    
    // false döndürerek istisnanın yakalanmasını engelleme
    return false;
}
```

## 6. Global Exception Handling (Global İstisna İşleme)

Uygulamanın farklı bölümlerinde, global istisna işleme mekanizmaları kullanılabilir.

### Console Uygulamaları

```csharp
// Console uygulamaları için global istisna işleme
public static void SetupGlobalExceptionHandlingForConsole()
{
    // İşlenmemiş istisnaları yakalama
    AppDomain.CurrentDomain.UnhandledException += (sender, e) =>
    {
        Exception ex = e.ExceptionObject as Exception;
        Console.WriteLine($"İşlenmemiş istisna: {ex?.Message}");
        
        // Günlükleme
        LogUnhandledException(ex);
        
        // Uygulama çıkışı
        Environment.Exit(1);
    };
    
    // Görev istisnalarını yakalama
    TaskScheduler.UnobservedTaskException += (sender, e) =>
    {
        Console.WriteLine($"Gözlemlenmeyen görev istisnası: {e.Exception.Message}");
        
        // Günlükleme
        LogUnhandledException(e.Exception);
        
        // İstisnayı işlenmiş olarak işaretleme
        e.SetObserved();
    };
}

private static void LogUnhandledException(Exception ex)
{
    // Gerçek uygulamada, dosyaya veya bir günlükleme servisine yazılabilir
    string logMessage = $"[{DateTime.Now}] HATA: {ex?.GetType().Name}: {ex?.Message}\n{ex?.StackTrace}";
    File.AppendAllText("error.log", logMessage);
}
```

### WPF Uygulamaları

```csharp
// WPF uygulamaları için global istisna işleme
public partial class App : Application
{
    protected override void OnStartup(StartupEventArgs e)
    {
        base.OnStartup(e);
        
        // İşlenmemiş istisnaları yakalama
        this.DispatcherUnhandledException += App_DispatcherUnhandledException;
        AppDomain.CurrentDomain.UnhandledException += CurrentDomain_UnhandledException;
        TaskScheduler.UnobservedTaskException += TaskScheduler_UnobservedTaskException;
    }
    
    private void App_DispatcherUnhandledException(object sender, System.Windows.Threading.DispatcherUnhandledExceptionEventArgs e)
    {
        // UI thread'inde oluşan istisnaları işleme
        MessageBox.Show($"Uygulama hatası: {e.Exception.Message}", "Hata", MessageBoxButton.OK, MessageBoxImage.Error);
        
        // Günlükleme
        LogUnhandledException(e.Exception);
        
        // İstisnayı işlenmiş olarak işaretleme (uygulamanın çökmesini engeller)
        e.Handled = true;
    }
    
    private void CurrentDomain_UnhandledException(object sender, UnhandledExceptionEventArgs e)
    {
        // Diğer thread'lerde oluşan istisnaları işleme
        Exception ex = e.ExceptionObject as Exception;
        MessageBox.Show($"Kritik hata: {ex?.Message}", "Kritik Hata", MessageBoxButton.OK, MessageBoxImage.Error);
        
        // Günlükleme
        LogUnhandledException(ex);
    }
    
    private void TaskScheduler_UnobservedTaskException(object sender, UnobservedTaskExceptionEventArgs e)
    {
        // Görev istisnalarını işleme
        MessageBox.Show($"Görev hatası: {e.Exception.Message}", "Görev Hatası", MessageBoxButton.OK, MessageBoxImage.Error);
        
        // Günlükleme
        LogUnhandledException(e.Exception);
        
        // İstisnayı işlenmiş olarak işaretleme
        e.SetObserved();
    }
}
```

### ASP.NET Core Uygulamaları

```csharp
// ASP.NET Core uygulamaları için global istisna işleme
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        // Diğer servis yapılandırmaları...
    }
    
    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            // Geliştirme ortamında hata sayfası
            app.UseDeveloperExceptionPage();
        }
        else
        {
            // Üretim ortamında özel hata işleme
            app.UseExceptionHandler("/Home/Error");
            app.UseStatusCodePagesWithReExecute("/Home/Error/{0}");
        }
        
        // Diğer middleware yapılandırmaları...
        
        app.UseRouting();
        app.UseEndpoints(endpoints =>
        {
            endpoints.MapControllerRoute(
                name: "default",
                pattern: "{controller=Home}/{action=Index}/{id?}");
        });
    }
}

// Özel istisna işleme middleware'i
public class CustomExceptionHandlerMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<CustomExceptionHandlerMiddleware> _logger;
    
    public CustomExceptionHandlerMiddleware(RequestDelegate next, ILogger<CustomExceptionHandlerMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }
    
    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            await _next(context);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "İstek işlenirken hata oluştu.");
            
            // İstemciye hata yanıtı gönderme
            context.Response.StatusCode = StatusCodes.Status500InternalServerError;
            context.Response.ContentType = "application/json";
            
            var response = new
            {
                error = "Bir hata oluştu.",
                details = env.IsDevelopment() ? ex.Message : null
            };
            
            await context.Response.WriteAsJsonAsync(response);
        }
    }
}

// Middleware uzantısı
public static class CustomExceptionHandlerMiddlewareExtensions
{
    public static IApplicationBuilder UseCustomExceptionHandler(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<CustomExceptionHandlerMiddleware>();
    }
}
```

## Özet

Bu bölümde, asenkron programlamada hata işleme tekniklerini inceledik:

1. **Try-Catch-Finally**: Asenkron metotlarda, senkron metotlardakine benzer şekilde `try-catch-finally` blokları kullanılabilir.

2. **AggregateException**: `Task.Wait()` veya `Task.Result` kullanıldığında, asenkron görevlerde oluşan hatalar bir `AggregateException` içinde sarmalanır.

3. **Task Exception Propagation**: Asenkron görevlerde oluşan istisnalar, görevin durumunu `Faulted` olarak işaretler ve `Task.Exception` özelliğinde saklanır.

4. **UnobservedTaskException**: Bir görevde oluşan istisna yakalanmazsa ve görev çöp toplayıcı tarafından temizlenirse, `TaskScheduler.UnobservedTaskException` olayı tetiklenir.

5. **Exception Filters**: C# 6.0 ve sonrasında, `when` anahtar kelimesi ile istisna filtreleri kullanılabilir.

6. **Global Exception Handling**: Uygulamanın farklı bölümlerinde, global istisna işleme mekanizmaları kullanılabilir.

Asenkron programlamada hata işleme, uygulamanızın güvenilirliğini ve bakımını kolaylaştırmak için önemlidir. Doğru hata işleme stratejileri, kullanıcı deneyimini iyileştirir ve hata ayıklamayı kolaylaştırır. 