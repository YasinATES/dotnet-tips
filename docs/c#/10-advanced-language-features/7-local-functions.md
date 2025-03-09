# Local Functions (Yerel Fonksiyonlar)

C# 7.0 ile tanıtılan Local Functions (Yerel Fonksiyonlar), bir metot içinde tanımlanan ve yalnızca o metot içinde kullanılabilen fonksiyonlardır. Bu özellik, kodunuzu daha modüler ve okunabilir hale getirmenize yardımcı olur.

## 1. Declaration Syntax (Tanımlama Sözdizimi)

Yerel fonksiyonlar, bir metot içinde tanımlanır ve yalnızca o metot içinde kullanılabilir.

```csharp
// Temel yerel fonksiyon tanımı
public void ProcessData(int[] data)
{
    // Ana metot kodu
    Console.WriteLine($"Veri işleniyor: {data.Length} öğe");
    
    // Yerel fonksiyon tanımı
    int Sum(int[] values)
    {
        int result = 0;
        foreach (int value in values)
        {
            result += value;
        }
        return result;
    }
    
    // Yerel fonksiyonu çağırma
    int total = Sum(data);
    Console.WriteLine($"Toplam: {total}");
}
```

### Dönüş Türleri ve Parametreler

Yerel fonksiyonlar, normal metotlar gibi farklı dönüş türleri ve parametreler alabilir.

```csharp
// Farklı dönüş türleri ve parametreler
public void CalculateStatistics(double[] values)
{
    // Ortalama hesaplayan yerel fonksiyon
    double CalculateAverage(double[] numbers)
    {
        return numbers.Sum() / numbers.Length;
    }
    
    // Standart sapma hesaplayan yerel fonksiyon
    double CalculateStandardDeviation(double[] numbers, double average)
    {
        double sumOfSquares = 0;
        foreach (double number in numbers)
        {
            sumOfSquares += Math.Pow(number - average, 2);
        }
        return Math.Sqrt(sumOfSquares / numbers.Length);
    }
    
    // Yerel fonksiyonları kullanma
    double average = CalculateAverage(values);
    double stdDev = CalculateStandardDeviation(values, average);
    
    Console.WriteLine($"Ortalama: {average:F2}");
    Console.WriteLine($"Standart Sapma: {stdDev:F2}");
}
```

### Ref ve Out Parametreler

Yerel fonksiyonlar, ref ve out parametreleri destekler.

```csharp
// Ref ve out parametreler
public void ProcessValues(int a, int b)
{
    // Ref parametreli yerel fonksiyon
    void Swap(ref int x, ref int y)
    {
        int temp = x;
        x = y;
        y = temp;
    }
    
    // Out parametreli yerel fonksiyon
    void Divide(int dividend, int divisor, out int quotient, out int remainder)
    {
        quotient = dividend / divisor;
        remainder = dividend % divisor;
    }
    
    Console.WriteLine($"Önce: a = {a}, b = {b}");
    
    // Ref parametreli fonksiyonu çağırma
    Swap(ref a, ref b);
    Console.WriteLine($"Swap sonrası: a = {a}, b = {b}");
    
    // Out parametreli fonksiyonu çağırma
    Divide(a, b, out int result, out int remainder);
    Console.WriteLine($"Bölme sonucu: {result}, Kalan: {remainder}");
}
```

## 2. Capture Semantics (Yakalama Semantiği)

Yerel fonksiyonlar, dış kapsamdaki değişkenleri yakalayabilir ve kullanabilir.

```csharp
// Değişken yakalama
public int FindIndexOfValue(int[] array, int value)
{
    // Dış kapsamdaki değişkenleri yakalama
    int targetValue = value;
    int foundIndex = -1;
    
    // Yerel fonksiyon
    void Search(int[] data)
    {
        for (int i = 0; i < data.Length; i++)
        {
            if (data[i] == targetValue)
            {
                foundIndex = i;
                break;
            }
        }
    }
    
    // Yerel fonksiyonu çağırma
    Search(array);
    
    return foundIndex;
}
```

### Değişken Yakalama Mekanizması

Yerel fonksiyonlar, dış kapsamdaki değişkenleri referans yoluyla yakalar.

```csharp
// Değişken yakalama mekanizması
public void DemonstrateCapture()
{
    int counter = 0;
    
    // Yerel fonksiyon
    void Increment()
    {
        counter++; // Dış kapsamdaki counter değişkenini değiştirir
    }
    
    Console.WriteLine($"Başlangıç değeri: {counter}");
    
    // Yerel fonksiyonu birkaç kez çağırma
    Increment();
    Increment();
    Increment();
    
    Console.WriteLine($"Son değer: {counter}"); // 3
}
```

### Yakalama ve Yaşam Süresi

Yakalanan değişkenlerin yaşam süresi, yerel fonksiyonun kullanımı boyunca uzatılır.

```csharp
// Yakalama ve yaşam süresi
public Func<int> CreateCounter()
{
    int count = 0;
    
    // Yerel fonksiyon
    int IncrementAndGet()
    {
        count++;
        return count;
    }
    
    // Yerel fonksiyonu döndürme (closure oluşturma)
    return IncrementAndGet;
}

// Kullanım örneği
public void UseCounter()
{
    var counter = CreateCounter();
    
    Console.WriteLine(counter()); // 1
    Console.WriteLine(counter()); // 2
    Console.WriteLine(counter()); // 3
}
```

## 3. Static Local Functions (Statik Yerel Fonksiyonlar)

C# 8.0 ile tanıtılan statik yerel fonksiyonlar, dış kapsamdaki değişkenleri yakalayamaz.

```csharp
// Statik yerel fonksiyonlar
public void ProcessWithStaticLocal(int[] data)
{
    int threshold = 10;
    
    // Statik yerel fonksiyon - dış kapsamdaki değişkenleri yakalayamaz
    static bool IsAboveThreshold(int value, int limit)
    {
        // threshold değişkenine erişemez, parametre olarak alması gerekir
        return value > limit;
    }
    
    // Statik olmayan yerel fonksiyon - dış kapsamdaki değişkenleri yakalayabilir
    bool IsAboveThresholdNonStatic(int value)
    {
        // threshold değişkenine doğrudan erişebilir
        return value > threshold;
    }
    
    int countStatic = 0;
    int countNonStatic = 0;
    
    foreach (int value in data)
    {
        if (IsAboveThreshold(value, threshold))
        {
            countStatic++;
        }
        
        if (IsAboveThresholdNonStatic(value))
        {
            countNonStatic++;
        }
    }
    
    Console.WriteLine($"Eşik üstündeki değerler (statik): {countStatic}");
    Console.WriteLine($"Eşik üstündeki değerler (statik olmayan): {countNonStatic}");
}
```

### Statik Yerel Fonksiyonların Avantajları

Statik yerel fonksiyonlar, performans ve güvenlik avantajları sağlar.

```csharp
// Statik yerel fonksiyonların avantajları
public int SumEvenNumbers(int[] numbers)
{
    // Statik yerel fonksiyon - daha verimli
    static bool IsEven(int n)
    {
        return n % 2 == 0;
    }
    
    int sum = 0;
    foreach (int number in numbers)
    {
        if (IsEven(number))
        {
            sum += number;
        }
    }
    
    return sum;
}
```

## 4. Iterator Local Functions (Yineleyici Yerel Fonksiyonlar)

Yerel fonksiyonlar, yineleyici (iterator) olarak tanımlanabilir.

```csharp
// Yineleyici yerel fonksiyonlar
public void ProcessSequence()
{
    // Yineleyici yerel fonksiyon
    IEnumerable<int> GenerateSequence(int start, int count)
    {
        for (int i = 0; i < count; i++)
        {
            yield return start + i;
        }
    }
    
    // Yineleyici yerel fonksiyonu kullanma
    foreach (int number in GenerateSequence(10, 5))
    {
        Console.WriteLine(number); // 10, 11, 12, 13, 14
    }
}
```

### Ertelenmiş Yürütme

Yineleyici yerel fonksiyonlar, ertelenmiş yürütme (lazy evaluation) sağlar.

```csharp
// Ertelenmiş yürütme
public void DemonstrateLazyEvaluation()
{
    Console.WriteLine("Ana metot başladı");
    
    // Yineleyici yerel fonksiyon
    IEnumerable<int> GetNumbers()
    {
        Console.WriteLine("GetNumbers başladı");
        
        for (int i = 0; i < 3; i++)
        {
            Console.WriteLine($"Üretiliyor: {i}");
            yield return i;
        }
        
        Console.WriteLine("GetNumbers bitti");
    }
    
    // Yineleyiciyi oluşturma (henüz yürütülmez)
    var numbers = GetNumbers();
    Console.WriteLine("Yineleyici oluşturuldu");
    
    // Yineleyiciyi tüketme (şimdi yürütülür)
    foreach (int number in numbers)
    {
        Console.WriteLine($"Tüketiliyor: {number}");
    }
    
    Console.WriteLine("Ana metot bitti");
}
```

## 5. Async Local Functions (Asenkron Yerel Fonksiyonlar)

Yerel fonksiyonlar, asenkron olarak tanımlanabilir.

```csharp
// Asenkron yerel fonksiyonlar
public async Task ProcessFilesAsync(string[] filePaths)
{
    // Asenkron yerel fonksiyon
    async Task<string> ReadFileAsync(string path)
    {
        Console.WriteLine($"Dosya okunuyor: {path}");
        
        using (StreamReader reader = new StreamReader(path))
        {
            return await reader.ReadToEndAsync();
        }
    }
    
    // Asenkron yerel fonksiyonu kullanma
    List<Task<string>> tasks = new List<Task<string>>();
    
    foreach (string path in filePaths)
    {
        tasks.Add(ReadFileAsync(path));
    }
    
    string[] contents = await Task.WhenAll(tasks);
    
    for (int i = 0; i < filePaths.Length; i++)
    {
        Console.WriteLine($"Dosya: {filePaths[i]}, Boyut: {contents[i].Length} karakter");
    }
}
```

### Asenkron Yerel Fonksiyonların Avantajları

Asenkron yerel fonksiyonlar, asenkron işlemleri daha modüler hale getirir.

```csharp
// Asenkron yerel fonksiyonların avantajları
public async Task<UserData> GetUserDataAsync(int userId)
{
    // Asenkron yerel fonksiyonlar
    async Task<UserProfile> GetProfileAsync()
    {
        // Profil verilerini getirme simülasyonu
        await Task.Delay(100); // API çağrısı simülasyonu
        return new UserProfile { Id = userId, Name = "Kullanıcı " + userId };
    }
    
    async Task<UserSettings> GetSettingsAsync()
    {
        // Ayarları getirme simülasyonu
        await Task.Delay(150); // API çağrısı simülasyonu
        return new UserSettings { UserId = userId, Theme = "Dark" };
    }
    
    async Task<List<UserActivity>> GetActivitiesAsync()
    {
        // Aktiviteleri getirme simülasyonu
        await Task.Delay(200); // API çağrısı simülasyonu
        return new List<UserActivity>
        {
            new UserActivity { UserId = userId, Action = "Login", Timestamp = DateTime.Now.AddHours(-1) },
            new UserActivity { UserId = userId, Action = "UpdateProfile", Timestamp = DateTime.Now.AddMinutes(-30) }
        };
    }
    
    // Tüm verileri paralel olarak getirme
    var profileTask = GetProfileAsync();
    var settingsTask = GetSettingsAsync();
    var activitiesTask = GetActivitiesAsync();
    
    await Task.WhenAll(profileTask, settingsTask, activitiesTask);
    
    // Sonuçları birleştirme
    return new UserData
    {
        Profile = await profileTask,
        Settings = await settingsTask,
        Activities = await activitiesTask
    };
}

// Yardımcı sınıflar
public class UserProfile { public int Id { get; set; } public string Name { get; set; } }
public class UserSettings { public int UserId { get; set; } public string Theme { get; set; } }
public class UserActivity { public int UserId { get; set; } public string Action { get; set; } public DateTime Timestamp { get; set; } }
public class UserData { public UserProfile Profile { get; set; } public UserSettings Settings { get; set; } public List<UserActivity> Activities { get; set; } }
```

## 6. Best Practice Scenarios (En İyi Uygulama Senaryoları)

Yerel fonksiyonların en etkili kullanım senaryoları.

### Yardımcı Fonksiyonlar

Yerel fonksiyonlar, bir metot içinde yalnızca bir kez kullanılan yardımcı fonksiyonlar için idealdir.

```csharp
// Yardımcı fonksiyonlar
public List<int> FilterAndTransform(List<int> input)
{
    List<int> result = new List<int>();
    
    // Yardımcı yerel fonksiyonlar
    bool IsValid(int n)
    {
        return n > 0 && n % 2 == 0;
    }
    
    int Transform(int n)
    {
        return n * n;
    }
    
    // Ana işlem
    foreach (int item in input)
    {
        if (IsValid(item))
        {
            result.Add(Transform(item));
        }
    }
    
    return result;
}
```

### Algoritma Parçalama

Karmaşık algoritmaları daha küçük, yönetilebilir parçalara bölmek için yerel fonksiyonlar kullanılabilir.

```csharp
// Algoritma parçalama
public List<int> MergeSort(List<int> list)
{
    // Ana metot sadece ilk çağrıyı yapar
    return Sort(list);
    
    // Yerel fonksiyonlar algoritmanın detaylarını gizler
    List<int> Sort(List<int> input)
    {
        if (input.Count <= 1)
        {
            return input;
        }
        
        int middle = input.Count / 2;
        List<int> left = input.GetRange(0, middle);
        List<int> right = input.GetRange(middle, input.Count - middle);
        
        left = Sort(left);
        right = Sort(right);
        
        return Merge(left, right);
    }
    
    List<int> Merge(List<int> left, List<int> right)
    {
        List<int> result = new List<int>();
        int leftIndex = 0, rightIndex = 0;
        
        while (leftIndex < left.Count && rightIndex < right.Count)
        {
            if (left[leftIndex] <= right[rightIndex])
            {
                result.Add(left[leftIndex]);
                leftIndex++;
            }
            else
            {
                result.Add(right[rightIndex]);
                rightIndex++;
            }
        }
        
        while (leftIndex < left.Count)
        {
            result.Add(left[leftIndex]);
            leftIndex++;
        }
        
        while (rightIndex < right.Count)
        {
            result.Add(right[rightIndex]);
            rightIndex++;
        }
        
        return result;
    }
}
```

### Doğrulama Mantığı

Giriş parametrelerini doğrulamak için yerel fonksiyonlar kullanılabilir.

```csharp
// Doğrulama mantığı
public double CalculateAverage(int[] grades)
{
    // Doğrulama yerel fonksiyonu
    void ValidateGrades()
    {
        if (grades == null)
        {
            throw new ArgumentNullException(nameof(grades));
        }
        
        if (grades.Length == 0)
        {
            throw new ArgumentException("Notlar dizisi boş olamaz", nameof(grades));
        }
        
        foreach (int grade in grades)
        {
            if (grade < 0 || grade > 100)
            {
                throw new ArgumentOutOfRangeException(nameof(grades), "Tüm notlar 0-100 aralığında olmalıdır");
            }
        }
    }
    
    // Önce doğrulama yap
    ValidateGrades();
    
    // Sonra hesaplama yap
    int sum = 0;
    foreach (int grade in grades)
    {
        sum += grade;
    }
    
    return (double)sum / grades.Length;
}
```

### Özyinelemeli Algoritmalar

Özyinelemeli (recursive) algoritmalar için yerel fonksiyonlar idealdir.

```csharp
// Özyinelemeli algoritmalar
public int Factorial(int n)
{
    // Parametreyi doğrula
    if (n < 0)
    {
        throw new ArgumentOutOfRangeException(nameof(n), "Faktöriyel negatif sayılar için tanımlı değildir");
    }
    
    // Özyinelemeli yerel fonksiyon
    int CalculateFactorial(int number)
    {
        if (number <= 1)
        {
            return 1;
        }
        
        return number * CalculateFactorial(number - 1);
    }
    
    // Yerel fonksiyonu çağır
    return CalculateFactorial(n);
}
```

### Temizleme Kodu

Kaynakları serbest bırakmak için yerel fonksiyonlar kullanılabilir.

```csharp
// Temizleme kodu
public async Task<string> ReadFileWithRetryAsync(string filePath, int maxRetries)
{
    int retryCount = 0;
    
    // Temizleme yerel fonksiyonu
    void CleanupResources(StreamReader reader)
    {
        if (reader != null)
        {
            reader.Dispose();
            Console.WriteLine("Kaynaklar temizlendi");
        }
    }
    
    // Asenkron okuma yerel fonksiyonu
    async Task<string> TryReadAsync()
    {
        StreamReader reader = null;
        
        try
        {
            reader = new StreamReader(filePath);
            string content = await reader.ReadToEndAsync();
            return content;
        }
        catch (IOException ex) when (retryCount < maxRetries)
        {
            Console.WriteLine($"Hata oluştu: {ex.Message}. Yeniden deneniyor...");
            CleanupResources(reader);
            retryCount++;
            await Task.Delay(1000 * retryCount); // Her denemede daha uzun bekle
            return await TryReadAsync(); // Yeniden dene
        }
        finally
        {
            if (reader != null)
            {
                CleanupResources(reader);
            }
        }
    }
    
    // Okuma işlemini başlat
    return await TryReadAsync();
}
```

## Gerçek Dünya Uygulamaları

Yerel fonksiyonların gerçek dünya uygulamalarında nasıl kullanılabileceğine dair örnekler.

```csharp
// Web API Controller örneği
public class ProductController
{
    private readonly IProductRepository _repository;
    
    public ProductController(IProductRepository repository)
    {
        _repository = repository;
    }
    
    public async Task<IActionResult> GetProductsAsync(string category, decimal? minPrice, decimal? maxPrice)
    {
        // Filtreleme mantığını yerel fonksiyonda kapsülleme
        async Task<List<Product>> FilterProductsAsync()
        {
            var products = await _repository.GetAllProductsAsync();
            
            // Kategori filtresi
            if (!string.IsNullOrEmpty(category))
            {
                products = products.Where(p => p.Category == category).ToList();
            }
            
            // Fiyat filtreleri
            if (minPrice.HasValue)
            {
                products = products.Where(p => p.Price >= minPrice.Value).ToList();
            }
            
            if (maxPrice.HasValue)
            {
                products = products.Where(p => p.Price <= maxPrice.Value).ToList();
            }
            
            return products;
        }
        
        try
        {
            var filteredProducts = await FilterProductsAsync();
            return new OkObjectResult(filteredProducts);
        }
        catch (Exception ex)
        {
            return new StatusCodeResult(500);
        }
    }
}

// Yardımcı sınıflar
public interface IProductRepository
{
    Task<List<Product>> GetAllProductsAsync();
}

public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Category { get; set; }
    public decimal Price { get; set; }
}

public class OkObjectResult
{
    public OkObjectResult(object value) { }
}

public class StatusCodeResult
{
    public StatusCodeResult(int statusCode) { }
}

public interface IActionResult { }
```

## Özet

Bu bölümde, C# 7.0 ile tanıtılan Local Functions (Yerel Fonksiyonlar) özelliğini inceledik:

1. **Declaration Syntax**: Yerel fonksiyonların nasıl tanımlanacağı ve kullanılacağı.

2. **Capture Semantics**: Yerel fonksiyonların dış kapsamdaki değişkenleri nasıl yakaladığı.

3. **Static Local Functions**: C# 8.0 ile tanıtılan statik yerel fonksiyonlar ve avantajları.

4. **Iterator Local Functions**: Yineleyici olarak tanımlanan yerel fonksiyonlar ve ertelenmiş yürütme.

5. **Async Local Functions**: Asenkron olarak tanımlanan yerel fonksiyonlar ve kullanım senaryoları.

6. **Best Practice Scenarios**: Yerel fonksiyonların en etkili kullanım senaryoları.

Yerel fonksiyonlar, kodunuzu daha modüler, okunabilir ve bakımı kolay hale getirmenize yardımcı olur. Özellikle karmaşık algoritmaları parçalamak, yardımcı fonksiyonları kapsüllemek ve asenkron işlemleri düzenlemek için idealdir. 