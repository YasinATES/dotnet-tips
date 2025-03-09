# Async/Await Pattern (Asenkron/Bekle Deseni)

Modern C# uygulamalarında, asenkron programlama vazgeçilmez bir özelliktir. Asenkron programlama, uygulamanızın daha duyarlı ve verimli olmasını sağlar. Bu bölümde, C#'ın `async` ve `await` anahtar kelimelerini kullanarak asenkron programlama yapmanın temellerini inceleyeceğiz.

## 1. Async Method Structure (Asenkron Metot Yapısı)

Asenkron bir metot, `async` anahtar kelimesi ile işaretlenir ve genellikle en az bir `await` ifadesi içerir.

```csharp
// Temel asenkron metot yapısı
public async Task<string> GetDataAsync(string url)
{
    // HttpClient oluşturma
    using (var client = new HttpClient())
    {
        // Asenkron olarak veri alma
        string result = await client.GetStringAsync(url);
        
        // Veriyi işleme
        return result.ToUpper();
    }
}

// Asenkron metodu çağırma
public async Task ProcessDataAsync()
{
    Console.WriteLine("Veri alınıyor...");
    
    // Asenkron metodu çağırma ve sonucu bekleme
    string data = await GetDataAsync("https://api.example.com/data");
    
    Console.WriteLine($"Alınan veri: {data}");
}
```

Asenkron bir metot, çağrıldığında hemen bir `Task` veya `Task<T>` nesnesi döndürür. Bu görev, asenkron işlem tamamlandığında sonucu içerir. `await` anahtar kelimesi, görevin tamamlanmasını bekler ve sonucu çıkarır.

## 2. Return Types (Dönüş Tipleri)

Asenkron metotlar, üç farklı dönüş tipi kullanabilir: `Task`, `Task<T>` ve `ValueTask<T>`.

### Task

`Task` dönüş tipi, asenkron bir işlemin tamamlandığını temsil eder, ancak bir değer döndürmez.

```csharp
// Task döndüren asenkron metot
public async Task SaveDataAsync(string data)
{
    // Dosya işlemleri için asenkron metotlar
    using (StreamWriter writer = new StreamWriter("data.txt"))
    {
        // Asenkron olarak dosyaya yazma
        await writer.WriteAsync(data);
    }
    
    // Metot tamamlandığında Task dönecektir
}

// Kullanım örneği
public async Task SaveExampleAsync()
{
    try
    {
        Console.WriteLine("Veri kaydediliyor...");
        
        // Asenkron metodu çağırma ve tamamlanmasını bekleme
        await SaveDataAsync("Örnek veri");
        
        Console.WriteLine("Veri başarıyla kaydedildi.");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Hata: {ex.Message}");
    }
}
```

### Task\<T>

`Task<T>` dönüş tipi, asenkron bir işlemin tamamlandığını ve bir `T` tipi değer döndürdüğünü temsil eder.

```csharp
// Task<T> döndüren asenkron metot
public async Task<List<Customer>> GetCustomersAsync()
{
    // Veritabanı bağlantısı
    using (var connection = new SqlConnection(connectionString))
    {
        // Asenkron olarak bağlantıyı açma
        await connection.OpenAsync();
        
        // Sorgu oluşturma
        using (var command = new SqlCommand("SELECT * FROM Customers", connection))
        {
            // Asenkron olarak sorguyu çalıştırma
            using (var reader = await command.ExecuteReaderAsync())
            {
                var customers = new List<Customer>();
                
                // Sonuçları okuma
                while (await reader.ReadAsync())
                {
                    customers.Add(new Customer
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        Email = reader.GetString(2)
                    });
                }
                
                return customers;
            }
        }
    }
}

// Kullanım örneği
public async Task DisplayCustomersAsync()
{
    try
    {
        Console.WriteLine("Müşteriler alınıyor...");
        
        // Asenkron metodu çağırma ve sonucu bekleme
        List<Customer> customers = await GetCustomersAsync();
        
        Console.WriteLine($"{customers.Count} müşteri bulundu:");
        
        foreach (var customer in customers)
        {
            Console.WriteLine($"{customer.Id}: {customer.Name} ({customer.Email})");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Hata: {ex.Message}");
    }
}

// Müşteri sınıfı
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}
```

### ValueTask\<T>

`ValueTask<T>`, `Task<T>`'ye benzer, ancak belirli durumlarda daha iyi performans sağlayabilir. Özellikle, asenkron işlemin senkron olarak tamamlanabileceği durumlarda kullanışlıdır.

```csharp
// ValueTask<T> döndüren asenkron metot
public async ValueTask<string> GetCachedDataAsync(string key)
{
    // Önbellekte veri var mı kontrol et
    if (_cache.TryGetValue(key, out string cachedData))
    {
        // Veri önbellekte varsa, hemen döndür (asenkron işlem yok)
        return cachedData;
    }
    
    // Veri önbellekte yoksa, asenkron olarak al
    string data = await FetchDataFromDatabaseAsync(key);
    
    // Veriyi önbelleğe ekle
    _cache.Add(key, data);
    
    return data;
}

// Veritabanından veri alma
private async Task<string> FetchDataFromDatabaseAsync(string key)
{
    // Veritabanı bağlantısı ve sorgu işlemleri
    await Task.Delay(100); // Veritabanı gecikmesini simüle etme
    return $"Data for {key}";
}

// Önbellek
private Dictionary<string, string> _cache = new Dictionary<string, string>();

// Kullanım örneği
public async Task UseValueTaskExampleAsync()
{
    // İlk çağrı - veritabanından alınacak
    string data1 = await GetCachedDataAsync("key1");
    Console.WriteLine($"İlk çağrı: {data1}");
    
    // İkinci çağrı - önbellekten alınacak (senkron)
    string data2 = await GetCachedDataAsync("key1");
    Console.WriteLine($"İkinci çağrı: {data2}");
}
```

## 3. Exception Handling (Hata İşleme)

Asenkron metotlarda hata işleme, senkron metotlardakine benzer şekilde `try-catch-finally` blokları kullanılarak yapılır.

```csharp
// Asenkron metotlarda hata işleme
public async Task<string> FetchDataWithErrorHandlingAsync(string url)
{
    try
    {
        // HttpClient oluşturma
        using (var client = new HttpClient())
        {
            // Zaman aşımı ayarlama
            client.Timeout = TimeSpan.FromSeconds(5);
            
            // Asenkron olarak veri alma
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
    catch (TaskCanceledException ex)
    {
        // Zaman aşımı
        Console.WriteLine($"İstek zaman aşımına uğradı: {ex.Message}");
        throw new TimeoutException("Veri alma işlemi zaman aşımına uğradı.", ex);
    }
    catch (Exception ex)
    {
        // Diğer hatalar
        Console.WriteLine($"Beklenmeyen hata: {ex.Message}");
        throw;
    }
}

// Kullanım örneği
public async Task TryFetchDataAsync()
{
    try
    {
        string data = await FetchDataWithErrorHandlingAsync("https://api.example.com/data");
        Console.WriteLine($"Alınan veri: {data}");
    }
    catch (TimeoutException ex)
    {
        Console.WriteLine($"Zaman aşımı hatası: {ex.Message}");
        // Kullanıcıya bildir
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Hata: {ex.Message}");
        // Hatayı günlüğe kaydet
    }
}
```

### AggregateException

Birden fazla asenkron işlemi `Task.WhenAll` ile beklerken, herhangi bir görev hata verirse, sonuç bir `AggregateException` olacaktır.

```csharp
// Birden fazla asenkron işlemi bekleme ve hata işleme
public async Task FetchMultipleDataAsync()
{
    var urls = new[]
    {
        "https://api.example.com/data1",
        "https://api.example.com/data2",
        "https://api.example.com/data3"
    };
    
    var tasks = urls.Select(url => FetchDataWithErrorHandlingAsync(url)).ToArray();
    
    try
    {
        // Tüm görevleri paralel olarak çalıştırma ve tamamlanmasını bekleme
        string[] results = await Task.WhenAll(tasks);
        
        // Tüm sonuçları işleme
        for (int i = 0; i < results.Length; i++)
        {
            Console.WriteLine($"URL {i+1} sonucu: {results[i].Substring(0, 50)}...");
        }
    }
    catch (Exception ex)
    {
        // Hata veren görevleri kontrol etme
        for (int i = 0; i < tasks.Length; i++)
        {
            if (tasks[i].IsFaulted)
            {
                Console.WriteLine($"URL {i+1} hatası: {tasks[i].Exception?.InnerException?.Message}");
            }
        }
        
        // Ana hatayı işleme
        Console.WriteLine($"Ana hata: {ex.Message}");
    }
}
```

## 4. Context Synchronization (Bağlam Senkronizasyonu)

Asenkron metotlar, çağrıldıkları bağlamı (UI thread, ASP.NET request context vb.) yakalayabilir ve `await` ifadesinden sonra bu bağlama geri dönebilir.

### ConfigureAwait

`ConfigureAwait(false)` metodu, asenkron işlemin tamamlanmasından sonra orijinal bağlama dönme gereksinimini kaldırır. Bu, özellikle kitaplık kodunda performansı artırabilir.

```csharp
// ConfigureAwait kullanımı
public async Task<string> FetchDataWithConfigureAwaitAsync(string url)
{
    using (var client = new HttpClient())
    {
        // ConfigureAwait(false) ile bağlam yakalamayı devre dışı bırakma
        string result = await client.GetStringAsync(url).ConfigureAwait(false);
        
        // Bu noktada, orijinal bağlamda olmayabiliriz
        return result.ToUpper();
    }
}

// UI uygulaması örneği
public async void OnButtonClick(object sender, EventArgs e)
{
    try
    {
        // UI thread'inde çalışıyor
        Button button = (Button)sender;
        button.Enabled = false;
        button.Text = "Yükleniyor...";
        
        // Asenkron işlemi başlatma
        string data = await FetchDataWithConfigureAwaitAsync("https://api.example.com/data");
        
        // ConfigureAwait(false) kullanılmadığı için, bu kod UI thread'inde çalışacak
        button.Text = "Tamamlandı";
        DisplayData(data);
    }
    catch (Exception ex)
    {
        MessageBox.Show($"Hata: {ex.Message}");
    }
    finally
    {
        ((Button)sender).Enabled = true;
    }
}

// ASP.NET Core örneği
[HttpGet]
public async Task<IActionResult> GetDataAsync()
{
    try
    {
        // HTTP request bağlamında çalışıyor
        string data = await FetchDataWithConfigureAwaitAsync("https://api.example.com/data");
        
        // ConfigureAwait(false) kullanıldığı için, bu kod farklı bir thread'de çalışabilir
        // Ancak ASP.NET Core'da bağlam yakalama maliyeti düşüktür
        return Ok(data);
    }
    catch (Exception ex)
    {
        return StatusCode(500, ex.Message);
    }
}
```

## 5. Async Void

Asenkron metotlar genellikle `Task` veya `Task<T>` döndürmelidir. `async void` metotları, yalnızca event handler'lar için kullanılmalıdır.

```csharp
// Kötü uygulama - async void metot
public async void SaveDataFireAndForget(string data)
{
    try
    {
        // Bu metottaki hatalar yakalanmaz ve uygulamanın çökmesine neden olabilir
        await Task.Delay(100); // Simüle edilmiş işlem
        throw new Exception("Beklenmeyen hata!");
    }
    catch (Exception ex)
    {
        // Bu catch bloğu çalışır, ancak hata uygulamanın geri kalanına yayılmaz
        Console.WriteLine($"Hata: {ex.Message}");
    }
}

// İyi uygulama - Task döndüren metot
public async Task SaveDataAsync(string data)
{
    await Task.Delay(100); // Simüle edilmiş işlem
    // Hata fırlatılırsa, çağıran tarafından yakalanabilir
}

// Event handler örneği - async void kullanımının uygun olduğu tek durum
public async void OnSaveButtonClick(object sender, EventArgs e)
{
    try
    {
        Button button = (Button)sender;
        button.Enabled = false;
        
        // Task döndüren asenkron metodu çağırma
        await SaveDataAsync("Örnek veri");
        
        MessageBox.Show("Veri başarıyla kaydedildi.");
    }
    catch (Exception ex)
    {
        MessageBox.Show($"Hata: {ex.Message}");
    }
    finally
    {
        ((Button)sender).Enabled = true;
    }
}
```

## 6. Best Practices (En İyi Uygulamalar)

Asenkron programlamada en iyi uygulamaları takip etmek, kodunuzun daha güvenilir ve bakımı daha kolay olmasını sağlar.

### Asenkron Tüm Yol (Async All the Way)

Bir metot zincirinde asenkron bir metot çağrılıyorsa, zincirdeki tüm metotlar asenkron olmalıdır.

```csharp
// Kötü uygulama - senkron ve asenkron karışımı
public string GetData(string url)
{
    // Asenkron metodu senkron olarak çağırma - deadlock riski!
    return GetDataAsync(url).Result;
}

// İyi uygulama - asenkron tüm yol
public async Task<string> GetDataAsync(string url)
{
    using (var client = new HttpClient())
    {
        return await client.GetStringAsync(url);
    }
}

public async Task<string> ProcessDataAsync(string url)
{
    string data = await GetDataAsync(url);
    return data.ToUpper();
}

public async Task DisplayDataAsync(string url)
{
    string processedData = await ProcessDataAsync(url);
    Console.WriteLine(processedData);
}
```

### Task.Run Kullanımı

CPU-bound işlemleri asenkron hale getirmek için `Task.Run` kullanılabilir, ancak bu I/O-bound işlemler için uygun değildir.

```csharp
// CPU-bound işlem için Task.Run kullanımı
public async Task<int> CalculateComplexValueAsync(int input)
{
    // CPU-bound işlemi arka planda çalıştırma
    return await Task.Run(() =>
    {
        // Yoğun CPU işlemi
        int result = 0;
        for (int i = 0; i < input * 1000000; i++)
        {
            result += i % input;
        }
        return result;
    });
}

// I/O-bound işlem için doğrudan asenkron API kullanımı
public async Task<string> ReadFileAsync(string filePath)
{
    // Task.Run KULLANMAYIN - doğrudan asenkron API kullanın
    return await File.ReadAllTextAsync(filePath);
}
```

### Asenkron Metot İsimlendirme

Asenkron metotlar, adlarının sonunda "Async" son ekini içermelidir.

```csharp
// Doğru isimlendirme
public async Task<Customer> GetCustomerAsync(int id)
{
    // Asenkron işlemler
    return await _repository.GetCustomerByIdAsync(id);
}

// Yanlış isimlendirme
public async Task<Customer> GetCustomer(int id)
{
    // Asenkron işlemler
    return await _repository.GetCustomerByIdAsync(id);
}
```

### Paralel İşlemler

Birden fazla asenkron işlemi paralel olarak çalıştırmak için `Task.WhenAll` veya `Task.WhenAny` kullanılabilir.

```csharp
// Birden fazla asenkron işlemi paralel olarak çalıştırma
public async Task<IEnumerable<Customer>> GetTopCustomersAsync()
{
    // Farklı veri kaynaklarından müşteri verilerini alma
    Task<List<Customer>> dbTask = _dbRepository.GetCustomersAsync();
    Task<List<Customer>> cacheTask = _cacheRepository.GetCustomersAsync();
    Task<List<Customer>> apiTask = _apiService.GetCustomersAsync();
    
    // Tüm görevlerin tamamlanmasını bekleme
    await Task.WhenAll(dbTask, cacheTask, apiTask);
    
    // Sonuçları birleştirme
    var allCustomers = new List<Customer>();
    allCustomers.AddRange(dbTask.Result);
    allCustomers.AddRange(cacheTask.Result);
    allCustomers.AddRange(apiTask.Result);
    
    // Tekrarlanan müşterileri kaldırma ve en yüksek bakiyeye sahip olanları seçme
    return allCustomers
        .GroupBy(c => c.Id)
        .Select(g => g.First())
        .OrderByDescending(c => c.TotalBalance)
        .Take(10);
}

// İlk tamamlanan görevi kullanma
public async Task<string> GetFastestDataSourceAsync()
{
    // Farklı veri kaynaklarından veri alma
    Task<string> source1Task = GetDataFromSource1Async();
    Task<string> source2Task = GetDataFromSource2Async();
    Task<string> source3Task = GetDataFromSource3Async();
    
    // İlk tamamlanan görevi bekleme
    Task<string> completedTask = await Task.WhenAny(source1Task, source2Task, source3Task);
    
    // Tamamlanan görevin sonucunu alma
    return await completedTask;
}
```

### İptal İşlemi (Cancellation)

Uzun süren asenkron işlemleri iptal etmek için `CancellationToken` kullanılabilir.

```csharp
// İptal edilebilir asenkron işlem
public async Task<string> DownloadDataAsync(string url, CancellationToken cancellationToken = default)
{
    using (var client = new HttpClient())
    {
        // İptal belirtecini HttpClient isteğine geçirme
        return await client.GetStringAsync(url, cancellationToken);
    }
}

// Kullanım örneği
public async Task DownloadWithTimeoutAsync()
{
    // 5 saniyelik bir iptal belirteci oluşturma
    using (var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5)))
    {
        try
        {
            Console.WriteLine("İndirme başlıyor...");
            
            // İptal belirtecini asenkron metoda geçirme
            string data = await DownloadDataAsync("https://api.example.com/largedata", cts.Token);
            
            Console.WriteLine($"İndirme tamamlandı. Veri boyutu: {data.Length} karakter");
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("İndirme zaman aşımına uğradı veya iptal edildi.");
        }
    }
}

// Manuel iptal örneği
public async Task DownloadWithManualCancellationAsync()
{
    using (var cts = new CancellationTokenSource())
    {
        // İptal işlemini 10 saniye sonra tetikleyecek bir görev
        Task cancelTask = Task.Delay(10000).ContinueWith(_ => cts.Cancel());
        
        try
        {
            Console.WriteLine("İndirme başlıyor...");
            
            // İptal belirtecini asenkron metoda geçirme
            string data = await DownloadDataAsync("https://api.example.com/largedata", cts.Token);
            
            Console.WriteLine($"İndirme tamamlandı. Veri boyutu: {data.Length} karakter");
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("İndirme iptal edildi.");
        }
    }
}
```

## Özet

Bu bölümde, C#'ın `async` ve `await` anahtar kelimelerini kullanarak asenkron programlama yapmanın temellerini inceledik:

1. **Async Method Structure**: Asenkron metotlar, `async` anahtar kelimesi ile işaretlenir ve genellikle en az bir `await` ifadesi içerir.

2. **Return Types**: Asenkron metotlar, `Task`, `Task<T>` veya `ValueTask<T>` döndürebilir.

3. **Exception Handling**: Asenkron metotlarda hata işleme, `try-catch-finally` blokları kullanılarak yapılır.

4. **Context Synchronization**: `ConfigureAwait(false)` metodu, asenkron işlemin tamamlanmasından sonra orijinal bağlama dönme gereksinimini kaldırır.

5. **Async Void**: Asenkron metotlar genellikle `Task` veya `Task<T>` döndürmelidir. `async void` metotları, yalnızca event handler'lar için kullanılmalıdır.

6. **Best Practices**: Asenkron programlamada en iyi uygulamaları takip etmek, kodunuzun daha güvenilir ve bakımı daha kolay olmasını sağlar.

Asenkron programlama, modern C# uygulamalarının önemli bir parçasıdır ve doğru kullanıldığında, uygulamanızın daha duyarlı ve verimli olmasını sağlar. 