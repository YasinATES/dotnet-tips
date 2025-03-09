# Task ve Task\<T>

C#'ta asenkron programlamanın temelini `Task` ve `Task<T>` sınıfları oluşturur. Bu sınıflar, asenkron işlemleri temsil eder ve yönetir. Bu bölümde, `Task` ve `Task<T>` sınıflarının kullanımını ve özelliklerini inceleyeceğiz.

## 1. Task Creation (Task Oluşturma)

`Task` ve `Task<T>` nesneleri çeşitli yöntemlerle oluşturulabilir.

### Task.Run

`Task.Run` metodu, bir işlemi arka planda çalıştırmak için en yaygın kullanılan yöntemdir.

```csharp
// Task.Run ile basit bir görev oluşturma
public Task SimpleTaskExample()
{
    // Arka planda çalışacak bir görev oluşturma
    Task task = Task.Run(() =>
    {
        Console.WriteLine("Görev çalışıyor...");
        // Uzun süren bir işlem simülasyonu
        Thread.Sleep(1000);
        Console.WriteLine("Görev tamamlandı.");
    });
    
    return task;
}

// Task<T>.Run ile değer döndüren bir görev oluşturma
public Task<int> CalculationTaskExample()
{
    // Arka planda çalışacak ve değer döndürecek bir görev oluşturma
    Task<int> task = Task.Run(() =>
    {
        Console.WriteLine("Hesaplama yapılıyor...");
        // Uzun süren bir hesaplama simülasyonu
        Thread.Sleep(2000);
        return 42; // Hesaplama sonucu
    });
    
    return task;
}

// Kullanım örneği
public async Task RunTaskExamples()
{
    // Basit görev örneği
    Task simpleTask = SimpleTaskExample();
    
    // Değer döndüren görev örneği
    Task<int> calculationTask = CalculationTaskExample();
    
    // Görevlerin tamamlanmasını bekleme
    await simpleTask;
    
    // Değer döndüren görevin sonucunu alma
    int result = await calculationTask;
    Console.WriteLine($"Hesaplama sonucu: {result}");
}
```

### Task Constructor ve TaskCompletionSource

`Task` constructor'ı ve `TaskCompletionSource` sınıfı, daha fazla kontrol gerektiren durumlarda kullanılabilir.

```csharp
// TaskCompletionSource ile görev oluşturma
public Task<string> CreateManualTask()
{
    // TaskCompletionSource oluşturma
    var tcs = new TaskCompletionSource<string>();
    
    // Arka planda çalışacak bir iş parçacığı başlatma
    Thread thread = new Thread(() =>
    {
        try
        {
            // Uzun süren bir işlem simülasyonu
            Thread.Sleep(1500);
            
            // İşlem başarılı olduğunda sonucu ayarlama
            tcs.SetResult("İşlem başarıyla tamamlandı.");
        }
        catch (Exception ex)
        {
            // Hata durumunda istisnayı ayarlama
            tcs.SetException(ex);
        }
    });
    
    // İş parçacığını başlatma
    thread.Start();
    
    // TaskCompletionSource'un Task özelliğini döndürme
    return tcs.Task;
}

// Kullanım örneği
public async Task UseManualTaskExample()
{
    try
    {
        Task<string> manualTask = CreateManualTask();
        
        // Görevin tamamlanmasını bekleme ve sonucu alma
        string result = await manualTask;
        Console.WriteLine(result);
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Hata: {ex.Message}");
    }
}
```

### Task.FromResult ve Task.CompletedTask

Zaten tamamlanmış görevler oluşturmak için `Task.FromResult` ve `Task.CompletedTask` kullanılabilir.

```csharp
// Zaten tamamlanmış görevler oluşturma
public Task<int> GetCachedValueAsync(string key)
{
    // Önbellekte değer var mı kontrol et
    if (_cache.TryGetValue(key, out int cachedValue))
    {
        // Değer önbellekte varsa, hemen tamamlanan bir görev döndür
        return Task.FromResult(cachedValue);
    }
    
    // Değer önbellekte yoksa, asenkron olarak al
    return FetchValueAsync(key);
}

// Değer döndürmeyen tamamlanmış görev
public Task LogMessageAsync(string message)
{
    // Mesajı hemen günlüğe kaydet
    Console.WriteLine($"Günlük: {message}");
    
    // Tamamlanmış bir görev döndür
    return Task.CompletedTask;
}

// Asenkron olarak değer alma
private async Task<int> FetchValueAsync(string key)
{
    // Uzun süren bir işlem simülasyonu
    await Task.Delay(1000);
    
    // Değeri hesapla
    int value = key.GetHashCode();
    
    // Değeri önbelleğe ekle
    _cache[key] = value;
    
    return value;
}

// Önbellek
private Dictionary<string, int> _cache = new Dictionary<string, int>();
```

## 2. Task Continuation (Task Devamı)

Bir görevin tamamlanmasından sonra başka bir görev çalıştırmak için continuation (devam) görevleri kullanılabilir.

### ContinueWith

`ContinueWith` metodu, bir görevin tamamlanmasından sonra başka bir görev çalıştırmak için kullanılır.

```csharp
// ContinueWith kullanımı
public Task ProcessDataWithContinuationExample()
{
    return Task.Run(() =>
    {
        // İlk görev: Veri alma
        Console.WriteLine("Veri alınıyor...");
        Thread.Sleep(1000);
        return "Örnek veri";
    })
    .ContinueWith(previousTask =>
    {
        // İkinci görev: Veriyi işleme
        string data = previousTask.Result;
        Console.WriteLine($"Veri işleniyor: {data}");
        Thread.Sleep(500);
        return data.ToUpper();
    })
    .ContinueWith(previousTask =>
    {
        // Üçüncü görev: Sonucu kaydetme
        string processedData = previousTask.Result;
        Console.WriteLine($"Sonuç kaydediliyor: {processedData}");
    });
}

// Hata işleme ile ContinueWith kullanımı
public Task ProcessDataWithErrorHandlingExample()
{
    return Task.Run<string>(() =>
    {
        // İlk görev: Veri alma (hata fırlatabilir)
        Console.WriteLine("Veri alınıyor...");
        Thread.Sleep(1000);
        
        // Simüle edilmiş hata
        throw new Exception("Veri alınamadı!");
        
        // Bu satır çalışmayacak
        return "Örnek veri";
    })
    .ContinueWith(previousTask =>
    {
        if (previousTask.IsFaulted)
        {
            // Hata durumunda
            Exception ex = previousTask.Exception.InnerException;
            Console.WriteLine($"Hata oluştu: {ex.Message}");
            return "Hata nedeniyle işlenemedi";
        }
        else
        {
            // Başarı durumunda
            string data = previousTask.Result;
            Console.WriteLine($"Veri işleniyor: {data}");
            return data.ToUpper();
        }
    });
}
```

### await Kullanımı

Modern C# kodunda, `ContinueWith` yerine genellikle `await` kullanılır, çünkü daha okunabilir ve hata işleme daha kolaydır.

```csharp
// await kullanımı
public async Task ProcessDataWithAwaitExample()
{
    try
    {
        // İlk görev: Veri alma
        Console.WriteLine("Veri alınıyor...");
        string data = await Task.Run(() =>
        {
            Thread.Sleep(1000);
            return "Örnek veri";
        });
        
        // İkinci görev: Veriyi işleme
        Console.WriteLine($"Veri işleniyor: {data}");
        string processedData = await Task.Run(() =>
        {
            Thread.Sleep(500);
            return data.ToUpper();
        });
        
        // Üçüncü görev: Sonucu kaydetme
        Console.WriteLine($"Sonuç kaydediliyor: {processedData}");
    }
    catch (Exception ex)
    {
        // Herhangi bir aşamada oluşan hatayı yakalama
        Console.WriteLine($"Hata oluştu: {ex.Message}");
    }
}
```

## 3. Task Composition (Task Kompozisyonu)

Birden fazla görevi birleştirmek için çeşitli yöntemler kullanılabilir.

### Task.WhenAll

`Task.WhenAll` metodu, birden fazla görevin tamamlanmasını beklemek için kullanılır.

```csharp
// Task.WhenAll kullanımı
public async Task FetchMultipleDataExample()
{
    // Farklı veri kaynaklarından veri alma görevleri
    Task<string> task1 = FetchDataFromSource("API");
    Task<string> task2 = FetchDataFromSource("Database");
    Task<string> task3 = FetchDataFromSource("Cache");
    
    // Tüm görevlerin tamamlanmasını bekleme
    string[] results = await Task.WhenAll(task1, task2, task3);
    
    // Sonuçları işleme
    for (int i = 0; i < results.Length; i++)
    {
        Console.WriteLine($"Kaynak {i+1} sonucu: {results[i]}");
    }
}

// Simüle edilmiş veri alma işlemi
private async Task<string> FetchDataFromSource(string source)
{
    // Farklı kaynaklar için farklı gecikmeler
    int delay = source switch
    {
        "API" => 1500,
        "Database" => 1000,
        "Cache" => 500,
        _ => 1000
    };
    
    await Task.Delay(delay);
    return $"{source} verileri";
}
```

### Task.WhenAny

`Task.WhenAny` metodu, birden fazla görevden herhangi birinin tamamlanmasını beklemek için kullanılır.

```csharp
// Task.WhenAny kullanımı
public async Task GetFastestResponseExample()
{
    // Farklı sunuculardan veri alma görevleri
    Task<string> server1Task = FetchDataFromServer("Server1", 1500);
    Task<string> server2Task = FetchDataFromServer("Server2", 1000);
    Task<string> server3Task = FetchDataFromServer("Server3", 2000);
    
    // Tüm görevleri bir diziye ekle
    var tasks = new[] { server1Task, server2Task, server3Task };
    
    // İlk tamamlanan görevi bekleme
    Task<string> fastestTask = await Task.WhenAny(tasks);
    
    // İlk tamamlanan görevin sonucunu alma
    string fastestResult = await fastestTask;
    Console.WriteLine($"En hızlı yanıt: {fastestResult}");
    
    // Diğer görevlerin tamamlanmasını bekleme (isteğe bağlı)
    await Task.WhenAll(tasks);
    Console.WriteLine("Tüm görevler tamamlandı.");
}

// Simüle edilmiş sunucu isteği
private async Task<string> FetchDataFromServer(string serverName, int latency)
{
    await Task.Delay(latency);
    return $"{serverName} yanıtı (Gecikme: {latency}ms)";
}
```

## 4. Task Factory (Task Fabrikası)

`TaskFactory` sınıfı, görev oluşturma ve yapılandırma için daha fazla seçenek sunar.

```csharp
// TaskFactory kullanımı
public Task CreateTaskWithFactoryExample()
{
    // Özel bir TaskFactory oluşturma
    var factory = new TaskFactory(
        TaskCreationOptions.LongRunning,
        TaskContinuationOptions.None);
    
    // Factory ile görev oluşturma
    return factory.StartNew(() =>
    {
        Console.WriteLine("Uzun süren görev başladı...");
        // Uzun süren bir işlem simülasyonu
        Thread.Sleep(3000);
        Console.WriteLine("Uzun süren görev tamamlandı.");
    });
}

// Task.Factory kullanımı
public Task CreateChildTasksExample()
{
    return Task.Factory.StartNew(() =>
    {
        Console.WriteLine("Ana görev başladı.");
        
        // Alt görevler oluşturma
        var childTasks = new List<Task>();
        
        for (int i = 0; i < 3; i++)
        {
            int taskId = i;
            
            // AttachedToParent seçeneği ile alt görev oluşturma
            var childTask = Task.Factory.StartNew(() =>
            {
                Console.WriteLine($"Alt görev {taskId} başladı.");
                Thread.Sleep(1000);
                Console.WriteLine($"Alt görev {taskId} tamamlandı.");
            }, TaskCreationOptions.AttachedToParent);
            
            childTasks.Add(childTask);
        }
        
        Console.WriteLine("Ana görev, alt görevlerin tamamlanmasını bekliyor...");
        // Ana görev, tüm alt görevler tamamlandığında tamamlanacak
    });
}
```

## 5. Task Scheduling (Task Zamanlama)

Görevlerin ne zaman ve nerede çalıştırılacağını kontrol etmek için çeşitli seçenekler mevcuttur.

### TaskScheduler

`TaskScheduler` sınıfı, görevlerin hangi iş parçacığında çalıştırılacağını kontrol eder.

```csharp
// UI uygulaması için TaskScheduler kullanımı
public async Task UpdateUIExample(Button button)
{
    // UI thread'inin TaskScheduler'ını alma
    TaskScheduler uiScheduler = TaskScheduler.FromCurrentSynchronizationContext();
    
    // Arka planda uzun süren bir işlem yapma
    await Task.Run(() =>
    {
        Console.WriteLine("Arka plan işlemi başladı...");
        Thread.Sleep(2000);
        Console.WriteLine("Arka plan işlemi tamamlandı.");
    });
    
    // UI güncellemelerini UI thread'inde yapma
    await Task.Factory.StartNew(() =>
    {
        // Bu kod UI thread'inde çalışacak
        button.Text = "İşlem Tamamlandı";
        button.Enabled = true;
    }, CancellationToken.None, TaskCreationOptions.None, uiScheduler);
}
```

### TaskCreationOptions

`TaskCreationOptions` enum'ı, görevlerin nasıl oluşturulacağını ve çalıştırılacağını yapılandırmak için kullanılır.

```csharp
// TaskCreationOptions kullanımı
public Task CreateConfiguredTasksExample()
{
    // LongRunning seçeneği ile görev oluşturma
    Task longRunningTask = Task.Factory.StartNew(() =>
    {
        Console.WriteLine("Uzun süren görev başladı...");
        Thread.Sleep(5000);
        Console.WriteLine("Uzun süren görev tamamlandı.");
    }, TaskCreationOptions.LongRunning);
    
    // PreferFairness seçeneği ile görev oluşturma
    Task fairTask = Task.Factory.StartNew(() =>
    {
        Console.WriteLine("Adil görev başladı...");
        Thread.Sleep(1000);
        Console.WriteLine("Adil görev tamamlandı.");
    }, TaskCreationOptions.PreferFairness);
    
    return Task.WhenAll(longRunningTask, fairTask);
}
```

## 6. Task Exception Handling (Task İstisna İşleme)

Görevlerde oluşan istisnaları işlemek için çeşitli yöntemler mevcuttur.

### Try-Catch-Finally

Asenkron metotlarda, normal metotlardaki gibi `try-catch-finally` blokları kullanılabilir.

```csharp
// Try-catch-finally ile hata işleme
public async Task TryCatchFinallyExample()
{
    try
    {
        // Potansiyel olarak hata fırlatan bir görev
        await Task.Run(() =>
        {
            Console.WriteLine("Görev çalışıyor...");
            Thread.Sleep(1000);
            
            // Simüle edilmiş hata
            throw new InvalidOperationException("Bir hata oluştu!");
        });
    }
    catch (InvalidOperationException ex)
    {
        // Belirli bir hata türünü yakalama
        Console.WriteLine($"İşlem hatası: {ex.Message}");
    }
    catch (Exception ex)
    {
        // Diğer tüm hataları yakalama
        Console.WriteLine($"Beklenmeyen hata: {ex.Message}");
    }
    finally
    {
        // Her durumda çalışacak kod
        Console.WriteLine("Temizlik işlemleri yapılıyor...");
    }
}
```

### Task.Exception ve AggregateException

`Task.Exception` özelliği, bir görevde oluşan istisnaları içeren bir `AggregateException` döndürür.

```csharp
// Task.Exception kullanımı
public async Task HandleTaskExceptionExample()
{
    // Hata fırlatan bir görev oluşturma
    Task faultedTask = Task.Run(() =>
    {
        throw new InvalidOperationException("Görev hatası!");
    });
    
    try
    {
        // Görevin tamamlanmasını bekleme
        await faultedTask;
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Yakalanan hata: {ex.Message}");
    }
    
    // Alternatif olarak, görevin durumunu kontrol etme
    if (faultedTask.IsFaulted)
    {
        // Task.Exception bir AggregateException döndürür
        AggregateException aggEx = faultedTask.Exception;
        
        // İç istisnaları işleme
        foreach (var innerEx in aggEx.InnerExceptions)
        {
            Console.WriteLine($"İç hata: {innerEx.Message}");
        }
    }
}
```

### ContinueWith ve TaskContinuationOptions

`ContinueWith` metodu, görevin durumuna bağlı olarak farklı devam görevleri oluşturmak için kullanılabilir.

```csharp
// ContinueWith ile hata işleme
public Task ContinueWithErrorHandlingExample()
{
    return Task.Run(() =>
    {
        // Potansiyel olarak hata fırlatan bir işlem
        Console.WriteLine("Görev çalışıyor...");
        Thread.Sleep(1000);
        
        // Rastgele hata fırlatma
        if (DateTime.Now.Millisecond % 2 == 0)
        {
            throw new InvalidOperationException("Rastgele hata!");
        }
        
        return "Başarılı sonuç";
    })
    .ContinueWith(task =>
    {
        // Görev başarıyla tamamlandığında çalışır
        Console.WriteLine($"Başarı: {task.Result}");
    }, TaskContinuationOptions.OnlyOnRanToCompletion)
    .ContinueWith(task =>
    {
        // Görev hata verdiğinde çalışır
        Console.WriteLine($"Hata: {task.Exception.InnerException.Message}");
    }, TaskContinuationOptions.OnlyOnFaulted)
    .ContinueWith(task =>
    {
        // Görev iptal edildiğinde çalışır
        Console.WriteLine("Görev iptal edildi.");
    }, TaskContinuationOptions.OnlyOnCanceled)
    .ContinueWith(task =>
    {
        // Her durumda çalışır
        Console.WriteLine("Görev tamamlandı (başarılı, hatalı veya iptal edilmiş).");
    });
}
```

## Özet

Bu bölümde, C#'ta asenkron programlamanın temelini oluşturan `Task` ve `Task<T>` sınıflarını inceledik:

1. **Task Creation**: Görevleri `Task.Run`, constructor, `TaskCompletionSource`, `Task.FromResult` ve `Task.CompletedTask` ile oluşturma.

2. **Task Continuation**: Görevlerin tamamlanmasından sonra başka görevleri `ContinueWith` veya `await` ile çalıştırma.

3. **Task Composition**: Birden fazla görevi `Task.WhenAll` ve `Task.WhenAny` ile birleştirme.

4. **Task Factory**: Görev oluşturma ve yapılandırma için `TaskFactory` kullanımı.

5. **Task Scheduling**: Görevlerin ne zaman ve nerede çalıştırılacağını `TaskScheduler` ve `TaskCreationOptions` ile kontrol etme.

6. **Task Exception Handling**: Görevlerde oluşan istisnaları `try-catch-finally`, `Task.Exception` ve `ContinueWith` ile işleme.

`Task` ve `Task<T>` sınıfları, modern C# uygulamalarında asenkron programlama için temel yapı taşlarıdır ve doğru kullanıldığında, uygulamanızın daha duyarlı ve verimli olmasını sağlar. 