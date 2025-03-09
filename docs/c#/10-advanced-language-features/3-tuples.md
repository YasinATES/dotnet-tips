# Tuples (Demetler)

C# 7.0 ile tanıtılan Tuple türü, farklı türdeki birden fazla değeri tek bir veri yapısında gruplandırmanın hafif ve kullanışlı bir yolunu sunar. Bu bölümde, C#'taki tuple'ların kullanımını ve özelliklerini inceleyeceğiz.

## 1. Tuple Creation (Demet Oluşturma)

C#'ta tuple'lar birkaç farklı şekilde oluşturulabilir.

```csharp
// Temel tuple oluşturma
public void BasicTupleCreation()
{
    // Açık tip belirtme ile tuple oluşturma
    Tuple<int, string> legacyTuple = new Tuple<int, string>(1, "Bir");
    
    // C# 7.0 değer tuple'ları
    (int, string) valueTuple = (1, "Bir");
    
    // var ile tuple oluşturma
    var person = (Id: 1, Name: "Ahmet", Age: 30);
    
    // Mevcut değişkenlerden tuple oluşturma
    int id = 2;
    string name = "Mehmet";
    var employee = (id, name);
    
    Console.WriteLine($"Legacy Tuple: {legacyTuple.Item1}, {legacyTuple.Item2}");
    Console.WriteLine($"Value Tuple: {valueTuple.Item1}, {valueTuple.Item2}");
    Console.WriteLine($"Person: {person.Id}, {person.Name}, {person.Age}");
    Console.WriteLine($"Employee: {employee.id}, {employee.name}");
}
```

### Eski ve Yeni Tuple'lar Arasındaki Farklar

C# 7.0 öncesinde `System.Tuple` sınıfı kullanılıyordu, ancak C# 7.0 ile birlikte `ValueTuple` yapısı tanıtıldı.

```csharp
// Eski ve yeni tuple'lar arasındaki farklar
public void TupleDifferences()
{
    // Eski tuple (referans türü)
    Tuple<int, string, bool> oldTuple = Tuple.Create(1, "Test", true);
    
    // Yeni tuple (değer türü)
    (int, string, bool) newTuple = (1, "Test", true);
    
    // Eski tuple'lar değiştirilemez (immutable)
    // oldTuple.Item1 = 2; // Derleme hatası
    
    // Yeni tuple'lar değiştirilebilir (mutable)
    newTuple.Item1 = 2;
    
    Console.WriteLine($"Eski Tuple: {oldTuple.Item1}, {oldTuple.Item2}, {oldTuple.Item3}");
    Console.WriteLine($"Yeni Tuple: {newTuple.Item1}, {newTuple.Item2}, {newTuple.Item3}");
}
```

## 2. Named Members (İsimlendirilmiş Üyeler)

Tuple'ların öğelerine anlamlı isimler verebilirsiniz, bu da kodunuzu daha okunabilir hale getirir.

```csharp
// İsimlendirilmiş tuple üyeleri
public void NamedTupleMembers()
{
    // Tanımlama sırasında isimlendirme
    (int UserId, string Username, bool IsActive) user = (1, "ahmet", true);
    
    // Atama sırasında isimlendirme
    var product = (ProductId: 101, Name: "Laptop", Price: 5000m);
    
    // İsimleri kullanarak erişim
    Console.WriteLine($"Kullanıcı: {user.UserId}, {user.Username}, {user.IsActive}");
    Console.WriteLine($"Ürün: {product.ProductId}, {product.Name}, {product.Price:C}");
    
    // Item# notasyonu hala çalışır
    Console.WriteLine($"Alternatif erişim: {user.Item1}, {product.Item2}");
}
```

### İsim Projeksiyonu

Değişken isimlerini tuple üye isimleri olarak kullanabilirsiniz.

```csharp
// İsim projeksiyonu
public void NameProjection()
{
    int id = 1;
    string name = "Ahmet";
    DateTime birthDate = new DateTime(1990, 5, 15);
    
    // Değişken isimleri tuple üye isimleri olarak kullanılır
    var person = (id, name, birthDate);
    
    Console.WriteLine($"Kişi: {person.id}, {person.name}, {person.birthDate:d}");
}
```

## 3. Deconstruction (Parçalara Ayırma)

Tuple'ları bileşenlerine ayırabilirsiniz, bu da değerleri ayrı değişkenlere atamayı kolaylaştırır.

```csharp
// Tuple deconstruction
public void TupleDeconstruction()
{
    // Tuple oluşturma
    var person = (Id: 1, Name: "Ahmet", Age: 30);
    
    // Deconstruction ile ayrı değişkenlere atama
    (int id, string name, int age) = person;
    
    // var ile deconstruction
    var (id2, name2, age2) = person;
    
    // Bazı değerleri yok sayma
    (int id3, _, int age3) = person;
    
    Console.WriteLine($"Deconstruction: {id}, {name}, {age}");
    Console.WriteLine($"Var ile: {id2}, {name2}, {age2}");
    Console.WriteLine($"Kısmi: {id3}, _, {age3}");
}
```

### Özel Türler için Deconstruction

Kendi sınıflarınız ve yapılarınız için deconstruction desteği ekleyebilirsiniz.

```csharp
// Deconstruct metodu ile özel tür
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    
    // Deconstruct metodu
    public void Deconstruct(out int id, out string name, out string email)
    {
        id = Id;
        name = Name;
        email = Email;
    }
}

// Kullanım örneği
public void CustomTypeDeconstruction()
{
    var customer = new Customer { Id = 1, Name = "Ahmet", Email = "ahmet@example.com" };
    
    // Customer nesnesini deconstruct etme
    (int id, string name, string email) = customer;
    
    Console.WriteLine($"Müşteri: {id}, {name}, {email}");
}
```

## 4. Return Values (Dönüş Değerleri)

Tuple'lar, bir metottan birden fazla değer döndürmek için idealdir.

```csharp
// Tuple dönüş değeri
public (bool Success, string Message, int ResultCode) ProcessRequest(string request)
{
    if (string.IsNullOrEmpty(request))
    {
        return (false, "Boş istek", 400);
    }
    
    // İşlem başarılı
    return (true, "İstek başarıyla işlendi", 200);
}

// Kullanım örneği
public void ReturnValueExample()
{
    // Tuple döndüren metodu çağırma
    var result = ProcessRequest("Örnek istek");
    
    // Sonucu kontrol etme
    if (result.Success)
    {
        Console.WriteLine($"Başarılı: {result.Message}, Kod: {result.ResultCode}");
    }
    else
    {
        Console.WriteLine($"Hata: {result.Message}, Kod: {result.ResultCode}");
    }
    
    // Deconstruction ile kullanma
    var (success, message, code) = ProcessRequest("");
    Console.WriteLine($"Deconstruction: {success}, {message}, {code}");
}
```

### Tuple'ları Metot İmzalarında Kullanma

Tuple'ları parametre ve dönüş değeri olarak kullanabilirsiniz.

```csharp
// Tuple'ları metot imzalarında kullanma
public (int Min, int Max, double Average) CalculateStatistics(IEnumerable<int> numbers)
{
    if (!numbers.Any())
    {
        return (0, 0, 0);
    }
    
    return (numbers.Min(), numbers.Max(), numbers.Average());
}

// Tuple parametresi
public string FormatCoordinates((double Latitude, double Longitude) coordinates)
{
    return $"({coordinates.Latitude:F2}, {coordinates.Longitude:F2})";
}

// Kullanım örneği
public void MethodSignatureExample()
{
    var numbers = new[] { 1, 5, 3, 9, 7 };
    var stats = CalculateStatistics(numbers);
    
    Console.WriteLine($"İstatistikler - Min: {stats.Min}, Max: {stats.Max}, Ortalama: {stats.Average:F2}");
    
    var location = (Latitude: 41.0082, Longitude: 28.9784);
    string formatted = FormatCoordinates(location);
    
    Console.WriteLine($"Konum: {formatted}");
}
```

## 5. Equality Comparison (Eşitlik Karşılaştırması)

ValueTuple'lar, yapısal eşitlik karşılaştırması destekler.

```csharp
// Tuple eşitlik karşılaştırması
public void EqualityComparison()
{
    var tuple1 = (Id: 1, Name: "Ahmet");
    var tuple2 = (Id: 1, Name: "Ahmet");
    var tuple3 = (Id: 2, Name: "Ahmet");
    
    // Yapısal eşitlik karşılaştırması
    bool areEqual1 = tuple1 == tuple2; // True
    bool areEqual2 = tuple1 == tuple3; // False
    
    Console.WriteLine($"tuple1 == tuple2: {areEqual1}");
    Console.WriteLine($"tuple1 == tuple3: {areEqual2}");
    
    // Equals metodu
    bool areEqual3 = tuple1.Equals(tuple2); // True
    
    Console.WriteLine($"tuple1.Equals(tuple2): {areEqual3}");
    
    // Üye isimleri eşitlik karşılaştırmasında dikkate alınmaz
    var tuple4 = (X: 1, Y: "Ahmet");
    bool areEqual4 = tuple1 == tuple4; // True
    
    Console.WriteLine($"tuple1 == tuple4 (farklı üye isimleri): {areEqual4}");
}
```

## 6. Pattern Matching (Desen Eşleştirme)

Tuple'lar, desen eşleştirme ile kullanılabilir.

```csharp
// Tuple desen eşleştirme
public string EvaluateCoordinates((double X, double Y) point)
{
    return point switch
    {
        (0, 0) => "Orijin",
        (0, _) => "Y ekseni üzerinde",
        (_, 0) => "X ekseni üzerinde",
        (var x, var y) when x == y => "X = Y çizgisi üzerinde",
        (var x, var y) when x > 0 && y > 0 => "1. bölgede",
        (var x, var y) when x < 0 && y > 0 => "2. bölgede",
        (var x, var y) when x < 0 && y < 0 => "3. bölgede",
        (var x, var y) when x > 0 && y < 0 => "4. bölgede",
        _ => "Belirsiz konum"
    };
}

// Kullanım örneği
public void PatternMatchingExample()
{
    var points = new[]
    {
        (X: 0.0, Y: 0.0),
        (X: 0.0, Y: 5.0),
        (X: 5.0, Y: 0.0),
        (X: 3.0, Y: 3.0),
        (X: 2.0, Y: 4.0),
        (X: -2.0, Y: 4.0),
        (X: -3.0, Y: -3.0),
        (X: 4.0, Y: -2.0)
    };
    
    foreach (var point in points)
    {
        string evaluation = EvaluateCoordinates(point);
        Console.WriteLine($"Nokta ({point.X}, {point.Y}): {evaluation}");
    }
}
```

## 7. Performance Considerations (Performans Değerlendirmeleri)

ValueTuple'lar, performans açısından avantajlar sunar.

```csharp
// Performans değerlendirmeleri
public void PerformanceConsiderations()
{
    // ValueTuple değer türüdür (stack'te depolanır)
    (int, string) valueTuple = (1, "Test");
    
    // Tuple referans türüdür (heap'te depolanır)
    Tuple<int, string> referenceTuple = new Tuple<int, string>(1, "Test");
    
    // ValueTuple'lar büyük veri kümeleri için daha verimli olabilir
    const int iterations = 1000000;
    
    // ValueTuple performans testi
    var stopwatch1 = Stopwatch.StartNew();
    for (int i = 0; i < iterations; i++)
    {
        var vt = (Id: i, Name: "Test");
        int id = vt.Id;
    }
    stopwatch1.Stop();
    
    // Tuple performans testi
    var stopwatch2 = Stopwatch.StartNew();
    for (int i = 0; i < iterations; i++)
    {
        var rt = Tuple.Create(i, "Test");
        int id = rt.Item1;
    }
    stopwatch2.Stop();
    
    Console.WriteLine($"ValueTuple süresi: {stopwatch1.ElapsedMilliseconds} ms");
    Console.WriteLine($"Tuple süresi: {stopwatch2.ElapsedMilliseconds} ms");
}
```

### Bellek Kullanımı

ValueTuple'lar, Tuple'lara göre daha az bellek kullanır.

```csharp
// Bellek kullanımı
public void MemoryUsage()
{
    // ValueTuple'lar, Tuple'lara göre daha az bellek kullanır
    Console.WriteLine($"ValueTuple<int, string> boyutu: {Marshal.SizeOf<ValueTuple<int, string>>()} bayt");
    
    // Büyük koleksiyonlarda fark daha belirgin olur
    const int count = 1000000;
    
    var valueTuples = new List<(int, string)>(count);
    var referenceTuples = new List<Tuple<int, string>>(count);
    
    // Bellek kullanımını ölçmek için GC'yi zorla
    GC.Collect();
    long memoryBefore = GC.GetTotalMemory(true);
    
    // ValueTuple'ları doldur
    for (int i = 0; i < count; i++)
    {
        valueTuples.Add((i, "Test"));
    }
    
    GC.Collect();
    long memoryAfterValueTuples = GC.GetTotalMemory(true);
    
    // Referans Tuple'ları doldur
    for (int i = 0; i < count; i++)
    {
        referenceTuples.Add(Tuple.Create(i, "Test"));
    }
    
    GC.Collect();
    long memoryAfterReferenceTuples = GC.GetTotalMemory(true);
    
    Console.WriteLine($"ValueTuple koleksiyonu bellek kullanımı: {(memoryAfterValueTuples - memoryBefore) / 1024 / 1024} MB");
    Console.WriteLine($"Tuple koleksiyonu bellek kullanımı: {(memoryAfterReferenceTuples - memoryAfterValueTuples) / 1024 / 1024} MB");
}
```

## Gerçek Dünya Uygulamaları

### Veri Erişim Katmanı

Tuple'lar, veritabanı işlemlerinin sonuçlarını döndürmek için kullanılabilir.

```csharp
// Veri erişim katmanı örneği
public class UserRepository
{
    // Kullanıcı kaydı ve sonuç bilgisi döndürme
    public (bool Success, int UserId, string ErrorMessage) CreateUser(string username, string email)
    {
        try
        {
            // Veritabanı işlemleri simülasyonu
            if (string.IsNullOrEmpty(username))
            {
                return (false, 0, "Kullanıcı adı boş olamaz");
            }
            
            if (string.IsNullOrEmpty(email) || !email.Contains("@"))
            {
                return (false, 0, "Geçersiz e-posta adresi");
            }
            
            // Başarılı kayıt simülasyonu
            int newUserId = new Random().Next(1000, 9999);
            return (true, newUserId, null);
        }
        catch (Exception ex)
        {
            return (false, 0, $"Hata: {ex.Message}");
        }
    }
    
    // Kullanıcı bilgilerini ve istatistiklerini getirme
    public (UserInfo User, UserStats Stats) GetUserDetails(int userId)
    {
        // Gerçek uygulamada, bu veriler veritabanından gelir
        var user = new UserInfo { Id = userId, Username = "ahmet", Email = "ahmet@example.com" };
        var stats = new UserStats { LoginCount = 42, LastLoginDate = DateTime.Now.AddDays(-2) };
        
        return (user, stats);
    }
}

public class UserInfo
{
    public int Id { get; set; }
    public string Username { get; set; }
    public string Email { get; set; }
}

public class UserStats
{
    public int LoginCount { get; set; }
    public DateTime LastLoginDate { get; set; }
}

// Kullanım örneği
public void RepositoryExample()
{
    var repository = new UserRepository();
    
    // Kullanıcı oluşturma
    var createResult = repository.CreateUser("ahmet", "ahmet@example.com");
    
    if (createResult.Success)
    {
        Console.WriteLine($"Kullanıcı oluşturuldu, ID: {createResult.UserId}");
        
        // Kullanıcı detaylarını getirme
        var (user, stats) = repository.GetUserDetails(createResult.UserId);
        
        Console.WriteLine($"Kullanıcı: {user.Username}, E-posta: {user.Email}");
        Console.WriteLine($"İstatistikler: {stats.LoginCount} giriş, Son giriş: {stats.LastLoginDate:g}");
    }
    else
    {
        Console.WriteLine($"Hata: {createResult.ErrorMessage}");
    }
}
```

### Web API Yanıtları

Tuple'lar, API yanıtlarını modellemek için kullanılabilir.

```csharp
// Web API yanıtları
public class ApiController
{
    // API yanıtı döndürme
    public (int StatusCode, object Data, string ErrorMessage) GetResource(int id)
    {
        try
        {
            if (id <= 0)
            {
                return (400, null, "Geçersiz ID");
            }
            
            // Kayıt bulunamadı
            if (id > 1000)
            {
                return (404, null, "Kaynak bulunamadı");
            }
            
            // Başarılı yanıt
            var data = new { Id = id, Name = "Örnek Kaynak", CreatedDate = DateTime.Now };
            return (200, data, null);
        }
        catch (Exception ex)
        {
            return (500, null, $"Sunucu hatası: {ex.Message}");
        }
    }
}

// Kullanım örneği
public void ApiExample()
{
    var controller = new ApiController();
    
    // Farklı senaryolar için API çağrıları
    var validResponse = controller.GetResource(42);
    var invalidResponse = controller.GetResource(-1);
    var notFoundResponse = controller.GetResource(1001);
    
    // Yanıtları işleme
    ProcessApiResponse(validResponse);
    ProcessApiResponse(invalidResponse);
    ProcessApiResponse(notFoundResponse);
}

private void ProcessApiResponse((int StatusCode, object Data, string ErrorMessage) response)
{
    switch (response.StatusCode)
    {
        case 200:
            Console.WriteLine($"Başarılı: {response.Data}");
            break;
        case 400:
            Console.WriteLine($"Geçersiz istek: {response.ErrorMessage}");
            break;
        case 404:
            Console.WriteLine($"Bulunamadı: {response.ErrorMessage}");
            break;
        case 500:
            Console.WriteLine($"Sunucu hatası: {response.ErrorMessage}");
            break;
        default:
            Console.WriteLine($"Beklenmeyen durum kodu: {response.StatusCode}");
            break;
    }
}
```

## Özet

Bu bölümde, C# 7.0 ile tanıtılan Tuple türlerini inceledik:

1. **Tuple Creation**: Tuple'lar, birden fazla değeri tek bir veri yapısında gruplandırmak için kullanılır.

2. **Named Members**: Tuple üyelerine anlamlı isimler vererek kodunuzu daha okunabilir hale getirebilirsiniz.

3. **Deconstruction**: Tuple'ları bileşenlerine ayırarak değerleri ayrı değişkenlere atayabilirsiniz.

4. **Return Values**: Tuple'lar, bir metottan birden fazla değer döndürmek için idealdir.

5. **Equality Comparison**: ValueTuple'lar, yapısal eşitlik karşılaştırması destekler.

6. **Pattern Matching**: Tuple'lar, desen eşleştirme ile kullanılabilir.

7. **Performance Considerations**: ValueTuple'lar, performans ve bellek kullanımı açısından avantajlar sunar.

Tuple'lar, özellikle birden fazla değer döndürmeniz veya geçici olarak birden fazla değeri gruplandırmanız gereken durumlarda kodunuzu daha temiz ve daha verimli hale getirebilir. 