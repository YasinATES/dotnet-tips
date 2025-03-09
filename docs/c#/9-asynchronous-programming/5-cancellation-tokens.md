# Cancellation Tokens (İptal Belirteçleri)

Modern C# uygulamalarında, uzun süren işlemleri iptal edebilmek önemli bir özelliktir. `CancellationToken` ve ilgili sınıflar, asenkron işlemlerin düzgün bir şekilde iptal edilmesini sağlar. Bu bölümde, iptal belirteçlerinin kullanımını ve en iyi uygulamaları inceleyeceğiz.

## 1. CancellationTokenSource

`CancellationTokenSource`, iptal belirteçleri oluşturmak ve iptal isteklerini tetiklemek için kullanılan bir sınıftır.

```csharp
// Temel CancellationTokenSource kullanımı
public async Task BasicCancellationExampleAsync()
{
    // CancellationTokenSource oluşturma
    using var cts = new CancellationTokenSource();
    
    // İptal belirtecini alma
    CancellationToken token = cts.Token;
    
    // Asenkron işlemi başlatma
    Task processTask = ProcessDataAsync(token);
    
    try
    {
        // Kullanıcı iptal isteği simülasyonu (gerçek uygulamada bir düğme tıklaması olabilir)
        await Task.Delay(2000); // 2 saniye bekle
        
        // İptal isteği gönderme
        cts.Cancel();
        
        // İşlemin tamamlanmasını bekleme
        await processTask;
        
        Console.WriteLine("İşlem başarıyla tamamlandı.");
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("İşlem iptal edildi.");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"İşlem hatası: {ex.Message}");
    }
}

// İptal edilebilir asenkron işlem
private async Task ProcessDataAsync(CancellationToken cancellationToken)
{
    Console.WriteLine("Veri işleme başladı...");
    
    // Uzun süren bir işlem simülasyonu
    for (int i = 0; i < 10; i++)
    {
        // İptal isteği kontrol edildi mi diye kontrol etme
        cancellationToken.ThrowIfCancellationRequested();
        
        // İşlem adımı
        Console.WriteLine($"İşlem adımı {i + 1}/10 tamamlandı.");
        
        // Simüle edilmiş işlem süresi
        await Task.Delay(500, cancellationToken);
    }
    
    Console.WriteLine("Veri işleme tamamlandı.");
}
```

### Zaman Aşımı ile İptal

```csharp
// Zaman aşımı ile iptal
public async Task CancellationWithTimeoutAsync()
{
    // 5 saniye zaman aşımı ile CancellationTokenSource oluşturma
    using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
    
    try
    {
        // İptal belirteci ile uzun süren işlemi çalıştırma
        await LongRunningOperationAsync(cts.Token);
        
        Console.WriteLine("İşlem başarıyla tamamlandı.");
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("İşlem zaman aşımına uğradı ve iptal edildi.");
    }
}

// Uzun süren işlem
private async Task LongRunningOperationAsync(CancellationToken cancellationToken)
{
    Console.WriteLine("Uzun süren işlem başladı...");
    
    // 10 saniye süren bir işlem simülasyonu (zaman aşımından daha uzun)
    await Task.Delay(10000, cancellationToken);
    
    Console.WriteLine("Uzun süren işlem tamamlandı."); // Bu satır çalışmayacak
}
```

## 2. Token Propagation (Belirteç Yayılımı)

İptal belirteçleri, çağrı zinciri boyunca yayılmalıdır.

```csharp
// Belirteç yayılımı örneği
public async Task TokenPropagationExampleAsync()
{
    using var cts = new CancellationTokenSource();
    
    try
    {
        // Ana işlemi başlatma
        await ProcessOrderAsync(123, cts.Token);
        
        Console.WriteLine("Sipariş işlemi tamamlandı.");
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("Sipariş işlemi iptal edildi.");
    }
    
    // 2 saniye sonra iptal etme
    await Task.Delay(2000);
    cts.Cancel();
}

// Ana işlem
private async Task ProcessOrderAsync(int orderId, CancellationToken cancellationToken)
{
    Console.WriteLine($"Sipariş #{orderId} işleniyor...");
    
    // Alt işlemlere belirteci yayma
    await ValidateOrderAsync(orderId, cancellationToken);
    await ProcessPaymentAsync(orderId, cancellationToken);
    await UpdateInventoryAsync(orderId, cancellationToken);
    await SendConfirmationEmailAsync(orderId, cancellationToken);
    
    Console.WriteLine($"Sipariş #{orderId} tamamlandı.");
}

// Sipariş doğrulama
private async Task ValidateOrderAsync(int orderId, CancellationToken cancellationToken)
{
    Console.WriteLine($"Sipariş #{orderId} doğrulanıyor...");
    await Task.Delay(500, cancellationToken);
}

// Ödeme işleme
private async Task ProcessPaymentAsync(int orderId, CancellationToken cancellationToken)
{
    Console.WriteLine($"Sipariş #{orderId} için ödeme işleniyor...");
    await Task.Delay(1000, cancellationToken);
}

// Envanter güncelleme
private async Task UpdateInventoryAsync(int orderId, CancellationToken cancellationToken)
{
    Console.WriteLine($"Sipariş #{orderId} için envanter güncelleniyor...");
    await Task.Delay(700, cancellationToken);
}

// Onay e-postası gönderme
private async Task SendConfirmationEmailAsync(int orderId, CancellationToken cancellationToken)
{
    Console.WriteLine($"Sipariş #{orderId} için onay e-postası gönderiliyor...");
    await Task.Delay(300, cancellationToken);
}
```

## 3. Cooperative Cancellation (İşbirlikçi İptal)

İşbirlikçi iptal, iptal isteğinin işlem tarafından düzgün bir şekilde işlenmesini sağlar.

```csharp
// İşbirlikçi iptal örneği
public async Task CooperativeCancellationExampleAsync()
{
    using var cts = new CancellationTokenSource();
    
    // Kullanıcı arayüzü iptal düğmesi simülasyonu
    var cancellationTask = Task.Run(async () =>
    {
        Console.WriteLine("İptal etmek için herhangi bir tuşa basın...");
        Console.ReadKey();
        
        Console.WriteLine("İptal isteği gönderiliyor...");
        cts.Cancel();
    });
    
    // Dosya işleme görevi
    var processingTask = ProcessLargeFileAsync("data.csv", cts.Token);
    
    // Her iki görevin de tamamlanmasını bekleme
    await Task.WhenAny(processingTask, cancellationTask);
    
    if (processingTask.IsCompleted && !processingTask.IsCanceled)
    {
        Console.WriteLine("Dosya işleme tamamlandı.");
    }
    else
    {
        Console.WriteLine("Dosya işleme iptal edildi veya hata oluştu.");
    }
}

// Büyük dosya işleme
private async Task ProcessLargeFileAsync(string filePath, CancellationToken cancellationToken)
{
    Console.WriteLine($"{filePath} dosyası işleniyor...");
    
    // Dosya işleme simülasyonu
    int totalLines = 1000;
    int processedLines = 0;
    
    try
    {
        for (int i = 0; i < totalLines; i++)
        {
            // İptal isteği kontrol etme
            if (cancellationToken.IsCancellationRequested)
            {
                // Temiz bir şekilde iptal etme
                Console.WriteLine("İptal isteği alındı, kaynaklar temizleniyor...");
                
                // Kaynakları temizleme (açık dosyaları kapatma, vb.)
                await CleanupResourcesAsync();
                
                // İptal istisnası fırlatma
                cancellationToken.ThrowIfCancellationRequested();
            }
            
            // Satır işleme simülasyonu
            await Task.Delay(10);
            processedLines++;
            
            // Her 100 satırda bir ilerleme raporu
            if (i % 100 == 0)
            {
                Console.WriteLine($"İlerleme: {processedLines}/{totalLines} satır işlendi.");
            }
        }
        
        Console.WriteLine($"Tüm dosya işlendi: {processedLines}/{totalLines} satır.");
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine($"Dosya işleme iptal edildi. {processedLines}/{totalLines} satır işlendi.");
        throw; // İstisnayı yeniden fırlat
    }
}

// Kaynakları temizleme
private async Task CleanupResourcesAsync()
{
    // Simüle edilmiş temizlik işlemi
    await Task.Delay(100);
    Console.WriteLine("Kaynaklar temizlendi.");
}
```

## 4. Timeout Implementation (Zaman Aşımı Uygulaması)

Zaman aşımı, iptal belirteçleri kullanılarak uygulanabilir.

```csharp
// Zaman aşımı uygulaması
public async Task<T> WithTimeoutAsync<T>(Func<CancellationToken, Task<T>> taskFactory, TimeSpan timeout)
{
    using var timeoutCts = new CancellationTokenSource(timeout);
    
    try
    {
        // Belirtilen zaman aşımı ile görevi çalıştırma
        return await taskFactory(timeoutCts.Token);
    }
    catch (OperationCanceledException) when (timeoutCts.IsCancellationRequested)
    {
        // Zaman aşımı durumunda özel bir istisna fırlatma
        throw new TimeoutException($"İşlem {timeout.TotalSeconds} saniye içinde tamamlanamadı.");
    }
}

// Kullanım örneği
public async Task TimeoutExampleAsync()
{
    try
    {
        // 2 saniye zaman aşımı ile API çağrısı yapma
        var result = await WithTimeoutAsync(
            token => CallExternalApiAsync("https://api.example.com/data", token),
            TimeSpan.FromSeconds(2));
        
        Console.WriteLine($"API yanıtı: {result}");
    }
    catch (TimeoutException ex)
    {
        Console.WriteLine($"Zaman aşımı: {ex.Message}");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Hata: {ex.Message}");
    }
}

// Simüle edilmiş API çağrısı
private async Task<string> CallExternalApiAsync(string url, CancellationToken cancellationToken)
{
    Console.WriteLine($"{url} adresine API çağrısı yapılıyor...");
    
    // Uzun süren bir API çağrısı simülasyonu (3 saniye)
    await Task.Delay(3000, cancellationToken);
    
    return "API yanıtı verisi";
}
```

### HttpClient ile Zaman Aşımı

```csharp
// HttpClient ile zaman aşımı
public async Task HttpClientTimeoutExampleAsync()
{
    // 5 saniye zaman aşımı ile CancellationTokenSource oluşturma
    using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
    
    try
    {
        using var httpClient = new HttpClient();
        
        // API çağrısı yapma
        var response = await httpClient.GetAsync("https://api.example.com/data", cts.Token);
        
        // Yanıtı işleme
        if (response.IsSuccessStatusCode)
        {
            string content = await response.Content.ReadAsStringAsync();
            Console.WriteLine($"API yanıtı: {content.Substring(0, Math.Min(100, content.Length))}...");
        }
        else
        {
            Console.WriteLine($"API hatası: {response.StatusCode}");
        }
    }
    catch (TaskCanceledException)
    {
        Console.WriteLine("API çağrısı zaman aşımına uğradı.");
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"HTTP isteği hatası: {ex.Message}");
    }
}
```

## 5. Linked Cancellation (Bağlantılı İptal)

Birden fazla iptal kaynağını birleştirmek için bağlantılı iptal kullanılabilir.

```csharp
// Bağlantılı iptal örneği
public async Task LinkedCancellationExampleAsync()
{
    // Kullanıcı iptal kaynağı
    using var userCts = new CancellationTokenSource();
    
    // Zaman aşımı iptal kaynağı (10 saniye)
    using var timeoutCts = new CancellationTokenSource(TimeSpan.FromSeconds(10));
    
    // İki kaynağı birleştirme
    using var linkedCts = CancellationTokenSource.CreateLinkedTokenSource(
        userCts.Token, timeoutCts.Token);
    
    try
    {
        // Bağlantılı belirteç ile işlemi başlatma
        Task processTask = ProcessBatchJobAsync(linkedCts.Token);
        
        // Kullanıcı iptal simülasyonu
        await Task.Delay(3000); // 3 saniye bekle
        Console.WriteLine("Kullanıcı işlemi iptal ediyor...");
        userCts.Cancel();
        
        // İşlemin tamamlanmasını bekleme
        await processTask;
    }
    catch (OperationCanceledException)
    {
        if (timeoutCts.IsCancellationRequested)
        {
            Console.WriteLine("İşlem zaman aşımına uğradı.");
        }
        else if (userCts.IsCancellationRequested)
        {
            Console.WriteLine("İşlem kullanıcı tarafından iptal edildi.");
        }
        else
        {
            Console.WriteLine("İşlem iptal edildi.");
        }
    }
}

// Toplu iş işleme
private async Task ProcessBatchJobAsync(CancellationToken cancellationToken)
{
    Console.WriteLine("Toplu iş işlemi başladı...");
    
    // Uzun süren bir işlem simülasyonu
    for (int i = 0; i < 20; i++)
    {
        cancellationToken.ThrowIfCancellationRequested();
        
        Console.WriteLine($"Batch item {i + 1}/20 işleniyor...");
        await Task.Delay(500, cancellationToken);
    }
    
    Console.WriteLine("Toplu iş işlemi tamamlandı.");
}
```

### Çoklu Koşullu İptal

```csharp
// Çoklu koşullu iptal örneği
public async Task MultiConditionCancellationExampleAsync()
{
    // Kullanıcı iptal kaynağı
    using var userCts = new CancellationTokenSource();
    
    // Zaman aşımı iptal kaynağı (30 saniye)
    using var timeoutCts = new CancellationTokenSource(TimeSpan.FromSeconds(30));
    
    // Sistem yükü iptal kaynağı
    using var systemLoadCts = new CancellationTokenSource();
    
    // Tüm kaynakları birleştirme
    using var linkedCts = CancellationTokenSource.CreateLinkedTokenSource(
        userCts.Token, timeoutCts.Token, systemLoadCts.Token);
    
    // Sistem yükü izleme görevi
    _ = Task.Run(async () =>
    {
        while (!linkedCts.Token.IsCancellationRequested)
        {
            // Sistem yükünü kontrol etme (simüle edilmiş)
            double cpuLoad = GetCurrentCpuLoad();
            
            if (cpuLoad > 90)
            {
                Console.WriteLine($"Yüksek CPU yükü tespit edildi: {cpuLoad}%. İşlem iptal ediliyor...");
                systemLoadCts.Cancel();
                break;
            }
            
            await Task.Delay(1000, linkedCts.Token);
        }
    });
    
    try
    {
        // Bağlantılı belirteç ile veri işleme
        await ProcessLargeDatasetAsync(linkedCts.Token);
    }
    catch (OperationCanceledException)
    {
        if (timeoutCts.IsCancellationRequested)
        {
            Console.WriteLine("Veri işleme zaman aşımına uğradı.");
        }
        else if (userCts.IsCancellationRequested)
        {
            Console.WriteLine("Veri işleme kullanıcı tarafından iptal edildi.");
        }
        else if (systemLoadCts.IsCancellationRequested)
        {
            Console.WriteLine("Veri işleme yüksek sistem yükü nedeniyle iptal edildi.");
        }
        else
        {
            Console.WriteLine("Veri işleme iptal edildi.");
        }
    }
}

// Simüle edilmiş CPU yükü
private double GetCurrentCpuLoad()
{
    // Gerçek uygulamada, gerçek CPU yükünü ölçebilirsiniz
    Random random = new Random();
    return random.NextDouble() * 100;
}

// Büyük veri kümesi işleme
private async Task ProcessLargeDatasetAsync(CancellationToken cancellationToken)
{
    Console.WriteLine("Büyük veri kümesi işleniyor...");
    
    // Uzun süren bir işlem simülasyonu
    for (int i = 0; i < 100; i++)
    {
        cancellationToken.ThrowIfCancellationRequested();
        
        Console.WriteLine($"Veri bloğu {i + 1}/100 işleniyor...");
        await Task.Delay(300, cancellationToken);
    }
    
    Console.WriteLine("Büyük veri kümesi işleme tamamlandı.");
}
```

## 6. Resource Cleanup (Kaynak Temizleme)

İptal durumunda kaynakların düzgün bir şekilde temizlenmesi önemlidir.

```csharp
// Kaynak temizleme örneği
public async Task ResourceCleanupExampleAsync()
{
    using var cts = new CancellationTokenSource();
    
    // Dosya işleme görevi
    var processingTask = Task.Run(async () =>
    {
        // Kaynakları açma
        await using var resource = new DisposableResource();
        
        try
        {
            // Kaynağı kullanma
            await resource.ProcessAsync(cts.Token);
        }
        catch (OperationCanceledException)
        {
            Console.WriteLine("İşlem iptal edildi, kaynaklar temizleniyor...");
            throw; // İstisnayı yeniden fırlat
        }
    });
    
    // 2 saniye sonra iptal etme
    await Task.Delay(2000);
    cts.Cancel();
    
    try
    {
        await processingTask;
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("İşlem iptal edildi ve kaynaklar temizlendi.");
    }
}

// Temizlenebilir kaynak
public class DisposableResource : IAsyncDisposable
{
    private bool _isDisposed = false;
    
    public DisposableResource()
    {
        Console.WriteLine("Kaynak oluşturuldu.");
    }
    
    public async Task ProcessAsync(CancellationToken cancellationToken)
    {
        Console.WriteLine("Kaynak işleme başladı...");
        
        for (int i = 0; i < 10; i++)
        {
            cancellationToken.ThrowIfCancellationRequested();
            
            Console.WriteLine($"Kaynak işleme adımı {i + 1}/10");
            await Task.Delay(500, cancellationToken);
        }
        
        Console.WriteLine("Kaynak işleme tamamlandı.");
    }
    
    public async ValueTask DisposeAsync()
    {
        if (!_isDisposed)
        {
            // Simüle edilmiş asenkron temizlik
            await Task.Delay(100);
            
            Console.WriteLine("Kaynak temizlendi.");
            _isDisposed = true;
        }
    }
}
```

### CancellationToken ile using Bloğu

```csharp
// CancellationToken ile using bloğu
public async Task UsingWithCancellationExampleAsync()
{
    using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
    
    try
    {
        // Veritabanı bağlantısı açma
        await using var connection = await OpenDatabaseConnectionAsync(cts.Token);
        
        // Veritabanı işlemleri
        await using var transaction = await connection.BeginTransactionAsync(cts.Token);
        
        try
        {
            // Veritabanı işlemleri yapma
            await ExecuteDatabaseOperationsAsync(connection, transaction, cts.Token);
            
            // İşlem onaylama
            await transaction.CommitAsync(cts.Token);
        }
        catch
        {
            // Hata durumunda işlemi geri alma
            await transaction.RollbackAsync();
            throw;
        }
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("Veritabanı işlemi iptal edildi.");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Veritabanı hatası: {ex.Message}");
    }
}

// Simüle edilmiş veritabanı bağlantısı
private async Task<DbConnection> OpenDatabaseConnectionAsync(CancellationToken cancellationToken)
{
    Console.WriteLine("Veritabanı bağlantısı açılıyor...");
    
    // Bağlantı açma simülasyonu
    await Task.Delay(1000, cancellationToken);
    
    Console.WriteLine("Veritabanı bağlantısı açıldı.");
    
    // Simüle edilmiş bağlantı nesnesi
    return new MockDbConnection();
}

// Simüle edilmiş veritabanı işlemleri
private async Task ExecuteDatabaseOperationsAsync(
    DbConnection connection, 
    DbTransaction transaction, 
    CancellationToken cancellationToken)
{
    Console.WriteLine("Veritabanı işlemleri yapılıyor...");
    
    // Veritabanı işlemleri simülasyonu
    for (int i = 0; i < 3; i++)
    {
        cancellationToken.ThrowIfCancellationRequested();
        
        Console.WriteLine($"Veritabanı işlemi {i + 1}/3 yapılıyor...");
        await Task.Delay(1000, cancellationToken);
    }
    
    Console.WriteLine("Veritabanı işlemleri tamamlandı.");
}

// Simüle edilmiş veritabanı sınıfları
public class MockDbConnection : DbConnection
{
    public override string ConnectionString { get; set; }
    public override string Database => "MockDb";
    public override string DataSource => "MockServer";
    public override string ServerVersion => "1.0";
    public override ConnectionState State => ConnectionState.Open;
    
    public override void Close() { }
    
    public override void Open() { }
    
    public async Task<MockDbTransaction> BeginTransactionAsync(CancellationToken cancellationToken)
    {
        await Task.Delay(100, cancellationToken);
        return new MockDbTransaction();
    }
    
    protected override DbTransaction BeginDbTransaction(IsolationLevel isolationLevel)
    {
        return new MockDbTransaction();
    }
    
    protected override DbCommand CreateDbCommand()
    {
        return new MockDbCommand();
    }
    
    public override async ValueTask DisposeAsync()
    {
        Console.WriteLine("Veritabanı bağlantısı kapatıldı.");
        await Task.CompletedTask;
    }
}

public class MockDbTransaction : DbTransaction
{
    public override IsolationLevel IsolationLevel => IsolationLevel.ReadCommitted;
    
    protected override DbConnection DbConnection => new MockDbConnection();
    
    public async Task CommitAsync(CancellationToken cancellationToken)
    {
        await Task.Delay(200, cancellationToken);
        Console.WriteLine("İşlem onaylandı.");
    }
    
    public async Task RollbackAsync()
    {
        await Task.Delay(200);
        Console.WriteLine("İşlem geri alındı.");
    }
    
    public override void Commit()
    {
        Console.WriteLine("İşlem onaylandı.");
    }
    
    public override void Rollback()
    {
        Console.WriteLine("İşlem geri alındı.");
    }
    
    public override async ValueTask DisposeAsync()
    {
        Console.WriteLine("İşlem kapatıldı.");
        await Task.CompletedTask;
    }
}

public class MockDbCommand : DbCommand
{
    public override string CommandText { get; set; }
    public override int CommandTimeout { get; set; }
    public override CommandType CommandType { get; set; }
    public override bool DesignTimeVisible { get; set; }
    public override UpdateRowSource UpdatedRowSource { get; set; }
    protected override DbConnection DbConnection { get; set; }
    protected override DbParameterCollection DbParameterCollection => null;
    protected override DbTransaction DbTransaction { get; set; }
    
    public override void Cancel() { }
    
    public override int ExecuteNonQuery()
    {
        return 0;
    }
    
    public override object ExecuteScalar()
    {
        return null;
    }
    
    public override void Prepare() { }
    
    protected override DbParameter CreateDbParameter()
    {
        return null;
    }
    
    protected override DbDataReader ExecuteDbDataReader(CommandBehavior behavior)
    {
        return null;
    }
}
```

## Özet

Bu bölümde, C#'ta iptal belirteçlerinin kullanımını inceledik:

1. **CancellationTokenSource**: İptal belirteçleri oluşturmak ve iptal isteklerini tetiklemek için kullanılan sınıf.

2. **Token Propagation**: İptal belirteçlerinin çağrı zinciri boyunca nasıl yayılacağı.

3. **Cooperative Cancellation**: İptal isteğinin işlem tarafından düzgün bir şekilde işlenmesi.

4. **Timeout Implementation**: Zaman aşımı uygulaması için iptal belirteçlerinin kullanımı.

5. **Linked Cancellation**: Birden fazla iptal kaynağını birleştirme.

6. **Resource Cleanup**: İptal durumunda kaynakların düzgün bir şekilde temizlenmesi.

İptal belirteçleri, modern C# uygulamalarında uzun süren işlemleri kontrol etmek ve kullanıcı deneyimini iyileştirmek için önemli bir araçtır. Doğru kullanıldığında, uygulamanızın daha duyarlı ve güvenilir olmasını sağlar. 