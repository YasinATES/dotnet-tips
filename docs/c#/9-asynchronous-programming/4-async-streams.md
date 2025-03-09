# Async Streams (Asenkron Akışlar)

C# 8.0 ile birlikte gelen Async Streams (Asenkron Akışlar), asenkron olarak üretilen veri dizilerini tüketmek için kullanılan güçlü bir özelliktir. Bu bölümde, asenkron akışların nasıl kullanılacağını ve gerçek dünya uygulamalarını inceleyeceğiz.

## 1. IAsyncEnumerable\<T>

`IAsyncEnumerable<T>`, asenkron olarak üretilen veri dizilerini temsil eden bir arayüzdür.

```csharp
// IAsyncEnumerable<T> arayüzü
public interface IAsyncEnumerable<out T>
{
    IAsyncEnumerator<T> GetAsyncEnumerator(CancellationToken cancellationToken = default);
}
```

### Temel Kullanım

```csharp
// Asenkron akış üreten bir metot
public async IAsyncEnumerable<Transaction> GetTransactionsAsync(
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    // Veritabanı bağlantısı oluşturma
    using var connection = new SqlConnection(_connectionString);
    await connection.OpenAsync(cancellationToken);
    
    // Sorgu oluşturma
    using var command = new SqlCommand(
        "SELECT Id, Amount, Date, Description FROM Transactions ORDER BY Date DESC",
        connection);
    
    // Sorguyu asenkron olarak çalıştırma
    using var reader = await command.ExecuteReaderAsync(cancellationToken);
    
    // Sonuçları asenkron olarak okuma ve yield etme
    while (await reader.ReadAsync(cancellationToken))
    {
        // Her bir satırı işleme ve yield etme
        yield return new Transaction
        {
            Id = reader.GetInt32(0),
            Amount = reader.GetDecimal(1),
            Date = reader.GetDateTime(2),
            Description = reader.GetString(3)
        };
        
        // Simüle edilmiş gecikme (gerçek uygulamada olmayabilir)
        await Task.Delay(10, cancellationToken);
    }
}

// İşlem sınıfı
public class Transaction
{
    public int Id { get; set; }
    public decimal Amount { get; set; }
    public DateTime Date { get; set; }
    public string Description { get; set; }
}
```

## 2. IAsyncEnumerator\<T>

`IAsyncEnumerator<T>`, asenkron akışın elemanlarını tek tek almak için kullanılan bir arayüzdür.

```csharp
// IAsyncEnumerator<T> arayüzü
public interface IAsyncEnumerator<out T> : IAsyncDisposable
{
    ValueTask<bool> MoveNextAsync();
    T Current { get; }
}
```

### Manuel Kullanım

```csharp
// IAsyncEnumerator<T>'nin manuel kullanımı
public async Task ProcessTransactionsManuallyAsync()
{
    // Asenkron akış alma
    var transactionsStream = GetTransactionsAsync();
    
    // Asenkron numaralandırıcı alma
    var enumerator = transactionsStream.GetAsyncEnumerator();
    
    try
    {
        // Akışı manuel olarak işleme
        decimal total = 0;
        
        // Bir sonraki elemana geçme
        while (await enumerator.MoveNextAsync())
        {
            // Mevcut elemanı alma
            var transaction = enumerator.Current;
            
            // İşlem yapma
            total += transaction.Amount;
            Console.WriteLine($"İşlem: {transaction.Description}, Tutar: {transaction.Amount:C}");
        }
        
        Console.WriteLine($"Toplam: {total:C}");
    }
    finally
    {
        // Numaralandırıcıyı temizleme
        await enumerator.DisposeAsync();
    }
}
```

## 3. await foreach

`await foreach` deyimi, asenkron akışları daha kolay tüketmek için kullanılır.

```csharp
// await foreach kullanımı
public async Task ProcessTransactionsAsync()
{
    decimal total = 0;
    
    // await foreach ile asenkron akışı işleme
    await foreach (var transaction in GetTransactionsAsync())
    {
        // Her bir işlemi işleme
        total += transaction.Amount;
        Console.WriteLine($"İşlem: {transaction.Description}, Tutar: {transaction.Amount:C}");
        
        // Gerekirse ek asenkron işlemler yapma
        await UpdateDashboardAsync(transaction);
    }
    
    Console.WriteLine($"Toplam: {total:C}");
}

// Dashboard güncelleme
private async Task UpdateDashboardAsync(Transaction transaction)
{
    // Simüle edilmiş dashboard güncelleme
    await Task.Delay(5);
    Console.WriteLine($"Dashboard güncellendi: {transaction.Id}");
}
```

### ConfigureAwait Kullanımı

```csharp
// ConfigureAwait ile await foreach kullanımı
public async Task ProcessTransactionsWithConfigureAwaitAsync()
{
    // ConfigureAwait(false) ile asenkron akışı işleme
    await foreach (var transaction in GetTransactionsAsync().ConfigureAwait(false))
    {
        // Bu kod orijinal bağlamda çalışmayabilir
        Console.WriteLine($"İşlem: {transaction.Description}, Tutar: {transaction.Amount:C}");
    }
}
```

## 4. Cancellation Support (İptal Desteği)

Asenkron akışlar, iptal belirteçleri (cancellation tokens) ile iptal edilebilir.

```csharp
// İptal desteği ile asenkron akış kullanımı
public async Task ProcessTransactionsWithCancellationAsync()
{
    // İptal kaynağı oluşturma
    using var cts = new CancellationTokenSource();
    
    // 5 saniye sonra iptal etme
    cts.CancelAfter(TimeSpan.FromSeconds(5));
    
    try
    {
        // İptal belirteci ile asenkron akışı işleme
        await foreach (var transaction in GetTransactionsAsync(cts.Token))
        {
            Console.WriteLine($"İşlem: {transaction.Description}, Tutar: {transaction.Amount:C}");
            
            // Simüle edilmiş işlem
            await Task.Delay(100, cts.Token);
        }
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("İşlem iptal edildi.");
    }
}
```

### WithCancellation Uzantı Metodu

```csharp
// WithCancellation uzantı metodu kullanımı
public async Task ProcessTransactionsWithCancellationExtensionAsync()
{
    // İptal kaynağı oluşturma
    using var cts = new CancellationTokenSource();
    
    // 5 saniye sonra iptal etme
    cts.CancelAfter(TimeSpan.FromSeconds(5));
    
    try
    {
        // WithCancellation uzantı metodu ile iptal belirteci ekleme
        await foreach (var transaction in GetTransactionsAsync().WithCancellation(cts.Token))
        {
            Console.WriteLine($"İşlem: {transaction.Description}, Tutar: {transaction.Amount:C}");
            await Task.Delay(100, cts.Token);
        }
    }
    catch (OperationCanceledException)
    {
        Console.WriteLine("İşlem iptal edildi.");
    }
}
```

## 5. BackPressure Handling (Geri Basınç İşleme)

Geri basınç, tüketicinin üreticiden daha yavaş olduğu durumlarda veri akışını kontrol etme mekanizmasıdır.

```csharp
// Geri basınç işleme örneği
public async Task ProcessTransactionsWithBackPressureAsync()
{
    // Semaphore ile eşzamanlı işlem sayısını sınırlama
    using var semaphore = new SemaphoreSlim(5); // En fazla 5 eşzamanlı işlem
    
    // İşlem görevlerini tutacak liste
    var processingTasks = new List<Task>();
    
    // Asenkron akışı işleme
    await foreach (var transaction in GetTransactionsAsync())
    {
        // Semaphore'u bekleme (geri basınç uygulama)
        await semaphore.WaitAsync();
        
        // İşlemi asenkron olarak başlatma
        var processingTask = Task.Run(async () =>
        {
            try
            {
                // İşlemi işleme (simüle edilmiş)
                await ProcessTransactionAsync(transaction);
            }
            finally
            {
                // İşlem tamamlandığında semaphore'u serbest bırakma
                semaphore.Release();
            }
        });
        
        // Görevi listeye ekleme
        processingTasks.Add(processingTask);
    }
    
    // Tüm işlemlerin tamamlanmasını bekleme
    await Task.WhenAll(processingTasks);
    
    Console.WriteLine("Tüm işlemler tamamlandı.");
}

// İşlem işleme (simüle edilmiş)
private async Task ProcessTransactionAsync(Transaction transaction)
{
    // Simüle edilmiş işlem süresi (işlem tutarına bağlı)
    int processingTime = (int)(Math.Abs(transaction.Amount) * 10);
    await Task.Delay(processingTime);
    
    Console.WriteLine($"İşlem tamamlandı: {transaction.Id}, Süre: {processingTime}ms");
}
```

### Channel Kullanımı

```csharp
// Channel ile geri basınç işleme
public async Task ProcessTransactionsWithChannelAsync()
{
    // Sınırlı kapasiteli kanal oluşturma
    var channel = Channel.CreateBounded<Transaction>(new BoundedChannelOptions(100)
    {
        FullMode = BoundedChannelFullMode.Wait // Kanal doluysa bekle
    });
    
    // Üretici görevi
    var producerTask = ProduceTransactionsAsync(channel.Writer);
    
    // Tüketici görevi
    var consumerTask = ConsumeTransactionsAsync(channel.Reader);
    
    // Her iki görevin de tamamlanmasını bekleme
    await Task.WhenAll(producerTask, consumerTask);
}

// Üretici: İşlemleri kanala yazma
private async Task ProduceTransactionsAsync(ChannelWriter<Transaction> writer)
{
    try
    {
        // Asenkron akıştan işlemleri okuma ve kanala yazma
        await foreach (var transaction in GetTransactionsAsync())
        {
            // Kanala yazma (kanal doluysa bekler - geri basınç)
            await writer.WriteAsync(transaction);
            Console.WriteLine($"Kanala yazıldı: {transaction.Id}");
        }
    }
    finally
    {
        // Tüm işlemler yazıldığında kanalı kapatma
        writer.Complete();
    }
}

// Tüketici: Kanaldan işlemleri okuma ve işleme
private async Task ConsumeTransactionsAsync(ChannelReader<Transaction> reader)
{
    // Kanaldan işlemleri okuma
    await foreach (var transaction in reader.ReadAllAsync())
    {
        // İşlemi işleme (simüle edilmiş)
        await ProcessTransactionAsync(transaction);
    }
    
    Console.WriteLine("Tüm işlemler tüketildi.");
}
```

## 6. Stream Processing Patterns (Akış İşleme Desenleri)

Asenkron akışlar için çeşitli işleme desenleri mevcuttur.

### Filtreleme

```csharp
// Asenkron akış filtreleme
public async IAsyncEnumerable<Transaction> FilterTransactionsAsync(
    decimal minAmount,
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    await foreach (var transaction in GetTransactionsAsync(cancellationToken))
    {
        // Sadece belirli bir tutarın üzerindeki işlemleri yield etme
        if (Math.Abs(transaction.Amount) >= minAmount)
        {
            yield return transaction;
        }
    }
}

// Kullanım örneği
public async Task ProcessLargeTransactionsAsync()
{
    // 1000 birimden büyük işlemleri filtreleme
    await foreach (var transaction in FilterTransactionsAsync(1000))
    {
        Console.WriteLine($"Büyük işlem: {transaction.Description}, Tutar: {transaction.Amount:C}");
    }
}
```

### Dönüştürme (Mapping)

```csharp
// Asenkron akış dönüştürme
public async IAsyncEnumerable<TransactionSummary> MapTransactionsAsync(
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    await foreach (var transaction in GetTransactionsAsync(cancellationToken))
    {
        // İşlemi özete dönüştürme
        yield return new TransactionSummary
        {
            Id = transaction.Id,
            FormattedAmount = $"{transaction.Amount:C}",
            FormattedDate = transaction.Date.ToString("yyyy-MM-dd"),
            ShortDescription = transaction.Description.Length > 30
                ? transaction.Description.Substring(0, 27) + "..."
                : transaction.Description
        };
    }
}

// İşlem özeti sınıfı
public class TransactionSummary
{
    public int Id { get; set; }
    public string FormattedAmount { get; set; }
    public string FormattedDate { get; set; }
    public string ShortDescription { get; set; }
}

// Kullanım örneği
public async Task DisplayTransactionSummariesAsync()
{
    await foreach (var summary in MapTransactionsAsync())
    {
        Console.WriteLine($"{summary.FormattedDate}: {summary.ShortDescription} ({summary.FormattedAmount})");
    }
}
```

### Gruplama

```csharp
// Asenkron akış gruplama
public async Task GroupTransactionsByDateAsync()
{
    // Tarih bazlı gruplamak için sözlük
    var transactionsByDate = new Dictionary<DateTime, List<Transaction>>();
    
    // İşlemleri tarihlerine göre gruplama
    await foreach (var transaction in GetTransactionsAsync())
    {
        // Sadece tarih kısmını alma (saat olmadan)
        var dateOnly = transaction.Date.Date;
        
        // Sözlükte tarih yoksa yeni liste oluşturma
        if (!transactionsByDate.ContainsKey(dateOnly))
        {
            transactionsByDate[dateOnly] = new List<Transaction>();
        }
        
        // İşlemi ilgili tarih listesine ekleme
        transactionsByDate[dateOnly].Add(transaction);
    }
    
    // Grupları işleme
    foreach (var group in transactionsByDate.OrderByDescending(g => g.Key))
    {
        Console.WriteLine($"Tarih: {group.Key:yyyy-MM-dd}");
        Console.WriteLine($"İşlem sayısı: {group.Value.Count}");
        Console.WriteLine($"Toplam tutar: {group.Value.Sum(t => t.Amount):C}");
        Console.WriteLine();
    }
}
```

### Birleştirme (Merging)

```csharp
// İki asenkron akışı birleştirme
public async IAsyncEnumerable<Transaction> MergeTransactionsAsync(
    IAsyncEnumerable<Transaction> source1,
    IAsyncEnumerable<Transaction> source2,
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    // Her iki kaynaktan da işlemleri okuma ve birleştirme
    var transactions = new List<Transaction>();
    
    // İlk kaynaktan işlemleri okuma
    await foreach (var transaction in source1.WithCancellation(cancellationToken))
    {
        transactions.Add(transaction);
    }
    
    // İkinci kaynaktan işlemleri okuma
    await foreach (var transaction in source2.WithCancellation(cancellationToken))
    {
        transactions.Add(transaction);
    }
    
    // Birleştirilmiş ve sıralanmış işlemleri yield etme
    foreach (var transaction in transactions.OrderByDescending(t => t.Date))
    {
        yield return transaction;
    }
}

// Kullanım örneği
public async Task ProcessMergedTransactionsAsync()
{
    // İki farklı kaynak oluşturma
    var bankTransactions = GetBankTransactionsAsync();
    var creditCardTransactions = GetCreditCardTransactionsAsync();
    
    // Kaynakları birleştirme
    await foreach (var transaction in MergeTransactionsAsync(bankTransactions, creditCardTransactions))
    {
        Console.WriteLine($"İşlem: {transaction.Description}, Tarih: {transaction.Date:yyyy-MM-dd}, Tutar: {transaction.Amount:C}");
    }
}

// Banka işlemleri (simüle edilmiş)
private async IAsyncEnumerable<Transaction> GetBankTransactionsAsync(
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    // Simüle edilmiş banka işlemleri
    var transactions = new[]
    {
        new Transaction { Id = 1, Amount = 1000, Date = DateTime.Now.AddDays(-1), Description = "Maaş" },
        new Transaction { Id = 2, Amount = -200, Date = DateTime.Now.AddDays(-2), Description = "Market Alışverişi" }
    };
    
    foreach (var transaction in transactions)
    {
        await Task.Delay(100, cancellationToken);
        yield return transaction;
    }
}

// Kredi kartı işlemleri (simüle edilmiş)
private async IAsyncEnumerable<Transaction> GetCreditCardTransactionsAsync(
    [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    // Simüle edilmiş kredi kartı işlemleri
    var transactions = new[]
    {
        new Transaction { Id = 101, Amount = -150, Date = DateTime.Now.AddDays(-1), Description = "Restoran" },
        new Transaction { Id = 102, Amount = -300, Date = DateTime.Now.AddDays(-3), Description = "Giyim Alışverişi" }
    };
    
    foreach (var transaction in transactions)
    {
        await Task.Delay(150, cancellationToken);
        yield return transaction;
    }
}
```

### Özel Uzantı Metotları

```csharp
// Asenkron akışlar için uzantı metotları
public static class AsyncEnumerableExtensions
{
    // Filtreleme uzantı metodu
    public static async IAsyncEnumerable<T> WhereAsync<T>(
        this IAsyncEnumerable<T> source,
        Func<T, bool> predicate,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        await foreach (var item in source.WithCancellation(cancellationToken))
        {
            if (predicate(item))
            {
                yield return item;
            }
        }
    }
    
    // Dönüştürme uzantı metodu
    public static async IAsyncEnumerable<TResult> SelectAsync<TSource, TResult>(
        this IAsyncEnumerable<TSource> source,
        Func<TSource, TResult> selector,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        await foreach (var item in source.WithCancellation(cancellationToken))
        {
            yield return selector(item);
        }
    }
    
    // Asenkron dönüştürme uzantı metodu
    public static async IAsyncEnumerable<TResult> SelectAwaitAsync<TSource, TResult>(
        this IAsyncEnumerable<TSource> source,
        Func<TSource, Task<TResult>> selector,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        await foreach (var item in source.WithCancellation(cancellationToken))
        {
            yield return await selector(item);
        }
    }
    
    // İlk n elemanı alma uzantı metodu
    public static async IAsyncEnumerable<T> TakeAsync<T>(
        this IAsyncEnumerable<T> source,
        int count,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        if (count <= 0)
            yield break;
            
        int taken = 0;
        await foreach (var item in source.WithCancellation(cancellationToken))
        {
            yield return item;
            taken++;
            
            if (taken >= count)
                yield break;
        }
    }
}

// Uzantı metotlarının kullanımı
public async Task UseExtensionMethodsAsync()
{
    // Filtreleme ve dönüştürme
    var query = GetTransactionsAsync()
        .WhereAsync(t => t.Amount > 0)
        .SelectAsync(t => new { t.Id, t.Description, t.Amount });
    
    // Sonuçları işleme
    await foreach (var item in query)
    {
        Console.WriteLine($"Pozitif işlem: {item.Description}, Tutar: {item.Amount:C}");
    }
    
    // Asenkron dönüştürme
    var enrichedQuery = GetTransactionsAsync()
        .SelectAwaitAsync(async t => 
        {
            // Simüle edilmiş asenkron zenginleştirme
            var category = await GetTransactionCategoryAsync(t);
            return new { t.Id, t.Description, t.Amount, Category = category };
        })
        .TakeAsync(10); // Sadece ilk 10 işlem
    
    // Zenginleştirilmiş sonuçları işleme
    await foreach (var item in enrichedQuery)
    {
        Console.WriteLine($"İşlem: {item.Description}, Kategori: {item.Category}, Tutar: {item.Amount:C}");
    }
}

// İşlem kategorisi alma (simüle edilmiş)
private async Task<string> GetTransactionCategoryAsync(Transaction transaction)
{
    // Simüle edilmiş gecikme
    await Task.Delay(50);
    
    // Basit kategori belirleme mantığı
    if (transaction.Description.Contains("Maaş"))
        return "Gelir";
    else if (transaction.Description.Contains("Market"))
        return "Gıda";
    else if (transaction.Description.Contains("Restoran"))
        return "Yemek";
    else if (transaction.Description.Contains("Giyim"))
        return "Alışveriş";
    else
        return "Diğer";
}
```

## Özet

Bu bölümde, C# 8.0 ile gelen Async Streams (Asenkron Akışlar) özelliğini inceledik:

1. **IAsyncEnumerable\<T>**: Asenkron olarak üretilen veri dizilerini temsil eden arayüz.

2. **IAsyncEnumerator\<T>**: Asenkron akışın elemanlarını tek tek almak için kullanılan arayüz.

3. **await foreach**: Asenkron akışları daha kolay tüketmek için kullanılan deyim.

4. **Cancellation Support**: Asenkron akışların iptal belirteçleri ile nasıl iptal edilebileceği.

5. **BackPressure Handling**: Tüketicinin üreticiden daha yavaş olduğu durumlarda veri akışını kontrol etme mekanizmaları.

6. **Stream Processing Patterns**: Asenkron akışlar için filtreleme, dönüştürme, gruplama ve birleştirme gibi işleme desenleri.

Asenkron akışlar, özellikle büyük veri kümeleri üzerinde çalışırken veya ağ, dosya sistemi veya veritabanı gibi I/O sınırlı işlemlerde verimli ve duyarlı uygulamalar geliştirmek için güçlü bir araçtır. 