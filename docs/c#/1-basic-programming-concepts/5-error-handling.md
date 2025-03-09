# Hata Yönetimi

Hata yönetimi, güvenilir ve sağlam yazılımlar geliştirmenin temel bir parçasıdır. C#, kapsamlı bir exception handling mekanizması sunar ve bu mekanizma, programın çalışması sırasında oluşabilecek hataları ele almanıza olanak tanır. Bu bölümde, C#'taki hata yönetimi tekniklerini detaylı olarak inceleyeceğiz.

## 1. try-catch-finally Blokları

try-catch-finally blokları, C#'ta hata yönetiminin temel yapı taşlarıdır.

### Temel try-catch Yapısı

```csharp
try
{
    // Hata oluşabilecek kod
    int result = 10 / 0; // Bu satır bir DivideByZeroException fırlatacak
}
catch (Exception ex)
{
    // Hata yakalandığında çalışacak kod
    Console.WriteLine($"Bir hata oluştu: {ex.Message}");
}
```

### Çoklu catch Blokları

```csharp
try
{
    string input = Console.ReadLine();
    int number = int.Parse(input);
    int result = 10 / number;
    Console.WriteLine($"Sonuç: {result}");
}
catch (FormatException ex)
{
    // Sayısal olmayan bir giriş yapıldığında
    Console.WriteLine($"Geçersiz format: {ex.Message}");
}
catch (DivideByZeroException ex)
{
    // Sıfıra bölme hatası oluştuğunda
    Console.WriteLine($"Sıfıra bölme hatası: {ex.Message}");
}
catch (Exception ex)
{
    // Diğer tüm hatalar için
    Console.WriteLine($"Beklenmeyen hata: {ex.Message}");
}
```

### finally Bloğu

```csharp
FileStream file = null;
try
{
    file = File.Open("data.txt", FileMode.Open);
    // Dosya işlemleri...
}
catch (FileNotFoundException ex)
{
    Console.WriteLine($"Dosya bulunamadı: {ex.Message}");
}
finally
{
    // finally bloğu her durumda çalışır (hata olsun veya olmasın)
    file?.Dispose(); // Dosyayı kapat
    Console.WriteLine("Dosya kapatıldı.");
}
```

### Nested try-catch Blokları

```csharp
try
{
    try
    {
        // İç try bloğu
        int result = 10 / 0;
    }
    catch (DivideByZeroException ex)
    {
        Console.WriteLine($"İç catch: {ex.Message}");
        throw; // Hatayı dış bloğa yeniden fırlat
    }
}
catch (Exception ex)
{
    Console.WriteLine($"Dış catch: {ex.Message}");
}
```

## 2. Exception Tipleri ve Hiyerarşisi

C#'ta tüm exception sınıfları `System.Exception` sınıfından türetilir.

### Temel Exception Hiyerarşisi

```
System.Exception
├── System.SystemException
│   ├── System.ArithmeticException
│   │   └── System.DivideByZeroException
│   ├── System.NullReferenceException
│   ├── System.IndexOutOfRangeException
│   ├── System.InvalidCastException
│   ├── System.IO.IOException
│   │   ├── System.IO.FileNotFoundException
│   │   └── System.IO.DirectoryNotFoundException
│   └── System.OutOfMemoryException
└── System.ApplicationException
    └── [Custom Exceptions]
```

### Yaygın Exception Tipleri

```csharp
// ArithmeticException
try
{
    int result = 10 / 0;
}
catch (ArithmeticException ex)
{
    Console.WriteLine($"Aritmetik hata: {ex.Message}");
}

// NullReferenceException
try
{
    string name = null;
    int length = name.Length; // NullReferenceException fırlatır
}
catch (NullReferenceException ex)
{
    Console.WriteLine($"Null referans hatası: {ex.Message}");
}

// IndexOutOfRangeException
try
{
    int[] numbers = new int[3];
    int value = numbers[5]; // IndexOutOfRangeException fırlatır
}
catch (IndexOutOfRangeException ex)
{
    Console.WriteLine($"Dizi indeks hatası: {ex.Message}");
}

// FileNotFoundException
try
{
    string content = File.ReadAllText("nonexistent.txt");
}
catch (FileNotFoundException ex)
{
    Console.WriteLine($"Dosya bulunamadı: {ex.Message}");
}

// ArgumentException ve türevleri
try
{
    string name = "";
    if (string.IsNullOrEmpty(name))
    {
        throw new ArgumentException("İsim boş olamaz.", nameof(name));
    }
    
    int age = -5;
    if (age < 0)
    {
        throw new ArgumentOutOfRangeException(nameof(age), "Yaş negatif olamaz.");
    }
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Argüman hatası: {ex.Message}");
}
```

### Exception Özellikleri

```csharp
try
{
    throw new Exception("Bir hata oluştu.");
}
catch (Exception ex)
{
    // Temel özellikler
    Console.WriteLine($"Message: {ex.Message}"); // Hata mesajı
    Console.WriteLine($"Source: {ex.Source}"); // Hatanın kaynağı
    Console.WriteLine($"StackTrace: {ex.StackTrace}"); // Çağrı yığını
    Console.WriteLine($"TargetSite: {ex.TargetSite}"); // Hatanın oluştuğu metot
    
    // İç exception
    if (ex.InnerException != null)
    {
        Console.WriteLine($"Inner Exception: {ex.InnerException.Message}");
    }
    
    // Ek veri
    foreach (DictionaryEntry entry in ex.Data)
    {
        Console.WriteLine($"{entry.Key}: {entry.Value}");
    }
}
```

## 3. Custom Exception Oluşturma

Özel ihtiyaçlar için kendi exception sınıflarınızı oluşturabilirsiniz.

### Temel Custom Exception

```csharp
// Özel exception sınıfı
public class UserNotFoundException : Exception
{
    public string Username { get; }
    
    // Temel constructor
    public UserNotFoundException(string username)
        : base($"Kullanıcı bulunamadı: {username}")
    {
        Username = username;
    }
    
    // InnerException için constructor
    public UserNotFoundException(string username, Exception innerException)
        : base($"Kullanıcı bulunamadı: {username}", innerException)
    {
        Username = username;
    }
    
    // Serialization için constructor (opsiyonel)
    protected UserNotFoundException(SerializationInfo info, StreamingContext context)
        : base(info, context)
    {
        Username = info.GetString("Username");
    }
    
    // Serialization için GetObjectData override (opsiyonel)
    public override void GetObjectData(SerializationInfo info, StreamingContext context)
    {
        base.GetObjectData(info, context);
        info.AddValue("Username", Username);
    }
}

// Kullanım
void FindUser(string username)
{
    if (username == "admin")
    {
        return; // Kullanıcı bulundu
    }
    
    throw new UserNotFoundException(username);
}

try
{
    FindUser("guest");
}
catch (UserNotFoundException ex)
{
    Console.WriteLine(ex.Message);
    Console.WriteLine($"Aranan kullanıcı: {ex.Username}");
}
```

### Exception Sınıfı Seçimi

```csharp
// ApplicationException'dan türetme (eski yaklaşım)
public class OldStyleException : ApplicationException
{
    public OldStyleException(string message) : base(message) { }
}

// Doğrudan Exception'dan türetme (önerilen yaklaşım)
public class ModernException : Exception
{
    public ModernException(string message) : base(message) { }
}

// Özel bir exception tipinden türetme
public class DatabaseConnectionException : IOException
{
    public string ConnectionString { get; }
    
    public DatabaseConnectionException(string connectionString, string message)
        : base(message)
    {
        ConnectionString = connectionString;
    }
}
```

## 4. throw vs throw ex

Exception yeniden fırlatma yöntemleri arasında önemli farklar vardır.

### throw Kullanımı

```csharp
try
{
    // Hata oluşabilecek kod
    ProcessData();
}
catch (Exception ex)
{
    // Hatayı logla
    Logger.Log(ex);
    
    // Orijinal stack trace'i koruyarak hatayı yeniden fırlat
    throw;
}
```

### throw ex Kullanımı (Kaçınılması Gereken)

```csharp
try
{
    // Hata oluşabilecek kod
    ProcessData();
}
catch (Exception ex)
{
    // Hatayı logla
    Logger.Log(ex);
    
    // YANLIŞ KULLANIM: Orijinal stack trace kaybolur
    throw ex;
}
```

### Farklar ve Etkileri

```csharp
void Method1()
{
    try
    {
        Method2();
    }
    catch (Exception ex)
    {
        // throw kullanımı - orijinal stack trace korunur
        Console.WriteLine("Hata yakalandı, yeniden fırlatılıyor...");
        throw;
        
        // throw ex kullanımı - stack trace Method1'den başlar, Method2 bilgisi kaybolur
        // throw ex;
    }
}

void Method2()
{
    throw new Exception("Method2'de bir hata oluştu.");
}

try
{
    Method1();
}
catch (Exception ex)
{
    Console.WriteLine($"Hata: {ex.Message}");
    Console.WriteLine($"Stack Trace: {ex.StackTrace}");
}
```

### Exception Sarmalama (Wrapping)

```csharp
try
{
    // Veritabanı işlemi
    ExecuteDatabaseQuery();
}
catch (SqlException ex)
{
    // Düşük seviyeli hatayı daha anlamlı bir hata ile sarmala
    throw new DatabaseOperationException("Veritabanı sorgusu başarısız oldu.", ex);
}

// Kullanım
try
{
    PerformOperation();
}
catch (DatabaseOperationException ex)
{
    Console.WriteLine(ex.Message);
    
    // İç exception'a erişim
    if (ex.InnerException != null)
    {
        Console.WriteLine($"Orijinal hata: {ex.InnerException.Message}");
    }
}
```

## 5. Exception Filters (C# 6.0+)

Exception filters, belirli koşullara göre exception'ları filtrelemenizi sağlar.

### Temel Exception Filter

```csharp
try
{
    ProcessFile("data.txt");
}
catch (IOException ex) when (ex.HResult == -2147024816) // Dosya bulunamadı hatası
{
    Console.WriteLine("Belirtilen dosya bulunamadı.");
}
catch (IOException ex) when (ex.HResult == -2147024864) // Erişim engellendi hatası
{
    Console.WriteLine("Dosyaya erişim engellendi.");
}
catch (IOException) // Diğer tüm IO hataları
{
    Console.WriteLine("Genel bir IO hatası oluştu.");
}
```

### Özel Koşullarla Filtreleme

```csharp
try
{
    ProcessOrder(order);
}
catch (OrderException ex) when (ex.OrderId > 1000)
{
    Console.WriteLine("Yüksek öncelikli sipariş hatası.");
}
catch (OrderException ex) when (ex.OrderAmount > 10000)
{
    Console.WriteLine("Yüksek tutarlı sipariş hatası.");
}
catch (OrderException ex)
{
    Console.WriteLine("Standart sipariş hatası.");
}
```

### Loglama için Exception Filter

```csharp
try
{
    RiskyOperation();
}
catch (Exception ex) when (LogException(ex))
{
    // Bu blok asla çalışmaz çünkü LogException her zaman false döner
}
catch (Exception ex)
{
    // Normal exception handling
    Console.WriteLine($"Hata ele alındı: {ex.Message}");
}

// Loglama metodu
bool LogException(Exception ex)
{
    Logger.Log(ex);
    return false; // false döndürerek catch bloğunun çalışmasını engeller
}
```

## 6. async/await ve Exception Handling

Asenkron programlamada exception handling, senkron koddan biraz farklıdır.

### Temel Asenkron Exception Handling

```csharp
async Task ProcessDataAsync()
{
    try
    {
        await FetchDataAsync();
        await SaveDataAsync();
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"Veri çekme hatası: {ex.Message}");
    }
    catch (IOException ex)
    {
        Console.WriteLine($"Veri kaydetme hatası: {ex.Message}");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Beklenmeyen hata: {ex.Message}");
    }
}
```

### Task'lerde Exception Handling

```csharp
async Task<string> FetchDataAsync(string url)
{
    try
    {
        using var client = new HttpClient();
        return await client.GetStringAsync(url);
    }
    catch (HttpRequestException ex)
    {
        Console.WriteLine($"HTTP hatası: {ex.Message}");
        return string.Empty;
    }
}

// Çoklu Task'leri beklerken
async Task ProcessMultipleRequestsAsync()
{
    try
    {
        Task<string> task1 = FetchDataAsync("https://api.example.com/data1");
        Task<string> task2 = FetchDataAsync("https://api.example.com/data2");
        
        // Tüm task'lerin tamamlanmasını bekle
        await Task.WhenAll(task1, task2);
        
        // Sonuçları işle
        string data1 = await task1;
        string data2 = await task2;
    }
    catch (Exception ex)
    {
        // WhenAll'dan fırlatılan ilk exception yakalanır
        Console.WriteLine($"İstek hatası: {ex.Message}");
    }
}
```

### AggregateException Handling

```csharp
async Task ProcessParallelTasksAsync()
{
    var tasks = new List<Task>();
    
    for (int i = 0; i < 5; i++)
    {
        int taskId = i;
        tasks.Add(Task.Run(() => {
            if (taskId == 3)
            {
                throw new Exception($"Task {taskId} failed");
            }
            return taskId;
        }));
    }
    
    try
    {
        // Task.WhenAll yerine Task.WaitAll kullanıldığında AggregateException fırlatılır
        Task.WaitAll(tasks.ToArray());
    }
    catch (AggregateException ex)
    {
        // Tüm iç exception'ları işle
        foreach (var innerEx in ex.InnerExceptions)
        {
            Console.WriteLine($"İç hata: {innerEx.Message}");
        }
    }
}
```

### ConfigureAwait ve Exception Context

```csharp
async Task ProcessWithConfigureAwaitAsync()
{
    try
    {
        // ConfigureAwait(true) - orijinal senkronizasyon bağlamına dön (varsayılan)
        await Task.Delay(1000).ConfigureAwait(true);
        
        // ConfigureAwait(false) - herhangi bir thread'de devam et
        string data = await FetchDataAsync("https://api.example.com/data")
            .ConfigureAwait(false);
            
        // Bu noktada UI thread'inde olmayabiliriz
        ProcessData(data);
    }
    catch (Exception ex)
    {
        // ConfigureAwait(false) kullanıldığında, bu catch bloğu
        // orijinal thread'de çalışmayabilir
        Console.WriteLine($"Hata: {ex.Message}");
    }
}
```

## En İyi Pratikler

1. **Spesifik Exception Yakalama**
   - Genel `Exception` yerine, mümkün olduğunca spesifik exception tipleri yakalayın.
   - En spesifik exception'ları önce, daha genel olanları sonra yakalayın.

2. **Anlamlı Hata Mesajları**
   - Exception fırlatırken açıklayıcı ve anlamlı hata mesajları kullanın.
   - Parametre adlarını ve değerlerini hata mesajlarına dahil edin.

3. **Exception Sarmalama**
   - Düşük seviyeli exception'ları daha anlamlı, yüksek seviyeli exception'larla sarmalayın.
   - Orijinal exception'ı her zaman `innerException` olarak saklayın.

4. **finally Bloğu Kullanımı**
   - Kaynakları temizlemek için `finally` bloğu veya `using` statement kullanın.
   - `finally` bloğunda exception fırlatmaktan kaçının.

5. **Exception Filtreleme**
   - Belirli koşullara göre exception'ları filtrelemek için exception filters kullanın.
   - Loglama için yan etki içermeyen exception filters kullanın.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Boş Catch Blokları**
   ```csharp
   // Kötü - hataları sessizce yutmak
   try
   {
       riskyOperation();
   }
   catch (Exception)
   {
       // Boş catch bloğu
   }
   
   // İyi - en azından hatayı logla
   try
   {
       riskyOperation();
   }
   catch (Exception ex)
   {
       Logger.LogError(ex, "Riskli işlem başarısız oldu.");
   }
   ```

2. **throw ex Kullanımı**
   ```csharp
   // Kötü - stack trace kaybı
   try
   {
       ProcessData();
   }
   catch (Exception ex)
   {
       Logger.Log(ex);
       throw ex; // Orijinal stack trace kaybolur
   }
   
   // İyi - stack trace korunur
   try
   {
       ProcessData();
   }
   catch (Exception ex)
   {
       Logger.Log(ex);
       throw; // Orijinal stack trace korunur
   }
   ```

3. **Çok Fazla try-catch Kullanımı**
   ```csharp
   // Kötü - her satır için try-catch
   try
   {
       var data = FetchData();
   }
   catch (Exception ex)
   {
       Logger.Log(ex);
   }
   
   try
   {
       ProcessData(data);
   }
   catch (Exception ex)
   {
       Logger.Log(ex);
   }
   
   // İyi - mantıksal bir birim için tek try-catch
   try
   {
       var data = FetchData();
       ProcessData(data);
   }
   catch (DataFetchException ex)
   {
       Logger.Log(ex);
       // Veri çekme hatası için özel işlem
   }
   catch (DataProcessException ex)
   {
       Logger.Log(ex);
       // Veri işleme hatası için özel işlem
   }
   ```

4. **Gereksiz Exception Fırlatma**
   ```csharp
   // Kötü - gereksiz exception fırlatma
   public bool IsValidEmail(string email)
   {
       if (string.IsNullOrEmpty(email))
       {
           throw new ArgumentException("Email boş olamaz.");
       }
       
       return email.Contains("@");
   }
   
   // İyi - boolean dönüş değeri
   public bool IsValidEmail(string email)
   {
       if (string.IsNullOrEmpty(email))
       {
           return false;
       }
       
       return email.Contains("@");
   }
   ```

5. **Asenkron Metotlarda Exception Handling Unutmak**
   ```csharp
   // Kötü - asenkron metotta exception handling yok
   async Task ProcessDataAsync()
   {
       var data = await FetchDataAsync(); // Exception fırlatabilir
       await SaveDataAsync(data); // Exception fırlatabilir
   }
   
   // İyi - asenkron metotta exception handling
   async Task ProcessDataAsync()
   {
       try
       {
           var data = await FetchDataAsync();
           await SaveDataAsync(data);
       }
       catch (Exception ex)
       {
           Logger.LogError(ex, "Veri işleme hatası.");
           throw; // Gerekirse yeniden fırlat
       }
   }
   ```

Hata yönetimi, güvenilir ve sağlam yazılımlar geliştirmenin temel bir parçasıdır. C#'ın exception handling mekanizmasını doğru şekilde kullanarak, programınızın beklenmeyen durumlarla başa çıkabilmesini ve kullanıcılara anlamlı hata mesajları sunabilmesini sağlayabilirsiniz. 