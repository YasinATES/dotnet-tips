# Kontrol Yapıları

Kontrol yapıları, programın akışını yönlendiren ve belirli koşullara göre farklı kod bloklarının çalıştırılmasını sağlayan yapılardır. C#'ta kontrol yapıları, kodun daha esnek, verimli ve okunabilir olmasını sağlar. Bu bölümde, C#'taki temel kontrol yapılarını detaylı olarak inceleyeceğiz.

## 1. if-else İfadeleri

if-else ifadeleri, belirli bir koşula bağlı olarak farklı kod bloklarının çalıştırılmasını sağlar.

### Temel if İfadesi

```csharp
int age = 18;

if (age >= 18)
{
    Console.WriteLine("Reşitsiniz.");
}
```

### if-else İfadesi

```csharp
int age = 16;

if (age >= 18)
{
    Console.WriteLine("Reşitsiniz.");
}
else
{
    Console.WriteLine("Reşit değilsiniz.");
}
```

### if-else if-else İfadesi

```csharp
int score = 85;

if (score >= 90)
{
    Console.WriteLine("A");
}
else if (score >= 80)
{
    Console.WriteLine("B");
}
else if (score >= 70)
{
    Console.WriteLine("C");
}
else
{
    Console.WriteLine("F");
}
```

### Tek Satırlık if İfadesi

```csharp
int age = 20;

if (age >= 18) Console.WriteLine("Reşitsiniz.");
else Console.WriteLine("Reşit değilsiniz.");
```

### Mantıksal Operatörlerle Birlikte Kullanım

```csharp
int age = 25;
bool hasLicense = true;

if (age >= 18 && hasLicense)
{
    Console.WriteLine("Araba kullanabilirsiniz.");
}
else if (age >= 18 && !hasLicense)
{
    Console.WriteLine("Ehliyet almanız gerekiyor.");
}
else
{
    Console.WriteLine("Araba kullanamazsınız.");
}
```

### Null Kontrolü

```csharp
string name = GetName(); // null olabilir

if (name != null && name.Length > 0)
{
    Console.WriteLine($"Merhaba, {name}!");
}
else
{
    Console.WriteLine("İsim bulunamadı.");
}

// C# 6.0+ ile string.IsNullOrEmpty kullanımı
if (!string.IsNullOrEmpty(name))
{
    Console.WriteLine($"Merhaba, {name}!");
}
```

### Ternary Operatör ile if-else Alternatifi

```csharp
int age = 20;

// Geleneksel if-else
string message;
if (age >= 18)
{
    message = "Reşitsiniz.";
}
else
{
    message = "Reşit değilsiniz.";
}

// Ternary operatör ile aynı işlem
string message = age >= 18 ? "Reşitsiniz." : "Reşit değilsiniz.";

// Boolean değer atama
bool isAdult = age >= 18;

// Gereksiz kullanım (kaçınılması gereken)
bool isAdult = age >= 18 ? true : false; // Gereksiz, direkt 'age >= 18' kullanın
```

Ternary operatör, basit if-else ifadelerini daha kısa ve okunabilir hale getirebilir. Ancak karmaşık koşullar için geleneksel if-else yapısı daha uygundur.

## 2. switch-case Yapısı ve Pattern Matching

switch-case yapısı, bir değişkenin değerine göre farklı kod bloklarının çalıştırılmasını sağlar.

### Temel switch-case Yapısı

```csharp
int day = 3;
string dayName;

switch (day)
{
    case 1:
        dayName = "Pazartesi";
        break;
    case 2:
        dayName = "Salı";
        break;
    case 3:
        dayName = "Çarşamba";
        break;
    case 4:
        dayName = "Perşembe";
        break;
    case 5:
        dayName = "Cuma";
        break;
    case 6:
        dayName = "Cumartesi";
        break;
    case 7:
        dayName = "Pazar";
        break;
    default:
        dayName = "Geçersiz gün";
        break;
}

Console.WriteLine(dayName); // Çarşamba
```

### Çoklu case Değerleri

```csharp
int day = 6;
bool isWeekend;

switch (day)
{
    case 1:
    case 2:
    case 3:
    case 4:
    case 5:
        isWeekend = false;
        break;
    case 6:
    case 7:
        isWeekend = true;
        break;
    default:
        throw new ArgumentException("Geçersiz gün", nameof(day));
}

Console.WriteLine(isWeekend ? "Hafta sonu" : "Hafta içi"); // Hafta sonu
```

### switch Expression (C# 8.0+)

```csharp
int day = 3;
string dayName = day switch
{
    1 => "Pazartesi",
    2 => "Salı",
    3 => "Çarşamba",
    4 => "Perşembe",
    5 => "Cuma",
    6 => "Cumartesi",
    7 => "Pazar",
    _ => "Geçersiz gün"
};

Console.WriteLine(dayName); // Çarşamba
```

### Pattern Matching (C# 7.0+)

```csharp
object obj = "Hello";

switch (obj)
{
    case int i:
        Console.WriteLine($"Tamsayı: {i}");
        break;
    case string s:
        Console.WriteLine($"String: {s}");
        break;
    case bool b when b == true:
        Console.WriteLine("True");
        break;
    case null:
        Console.WriteLine("Null değer");
        break;
    default:
        Console.WriteLine("Bilinmeyen tip");
        break;
}
```

### Property Pattern (C# 8.0+)

```csharp
var person = new { Name = "John", Age = 25 };

switch (person)
{
    case { Name: "John", Age: 25 }:
        Console.WriteLine("John, 25 yaşında");
        break;
    case { Name: "John" }:
        Console.WriteLine("Başka bir John");
        break;
    case { Age: > 18 }:
        Console.WriteLine("Reşit bir kişi");
        break;
    default:
        Console.WriteLine("Tanınmayan kişi");
        break;
}
```

### Tuple Pattern (C# 8.0+)

```csharp
var point = (X: 0, Y: 0);

string quadrant = point switch
{
    (0, 0) => "Orijin",
    (> 0, > 0) => "1. Bölge",
    (< 0, > 0) => "2. Bölge",
    (< 0, < 0) => "3. Bölge",
    (> 0, < 0) => "4. Bölge",
    (0, _) => "Y ekseni",
    (_, 0) => "X ekseni",
    _ => "Bilinmeyen"
};

Console.WriteLine(quadrant); // Orijin
```

## 3. for, while, do-while Döngüleri

Döngüler, belirli bir kod bloğunu tekrar tekrar çalıştırmak için kullanılır.

### for Döngüsü

```csharp
// Temel for döngüsü
for (int i = 0; i < 5; i++)
{
    Console.WriteLine(i); // 0, 1, 2, 3, 4
}

// Çoklu değişken kullanımı
for (int i = 0, j = 10; i < j; i++, j--)
{
    Console.WriteLine($"i = {i}, j = {j}");
}

// Sonsuz döngü
// for (;;)
// {
//     // Sonsuz döngü
// }

// Dizileri işleme
int[] numbers = { 1, 2, 3, 4, 5 };
for (int i = 0; i < numbers.Length; i++)
{
    Console.WriteLine(numbers[i]);
}
```

### while Döngüsü

```csharp
// Temel while döngüsü
int count = 0;
while (count < 5)
{
    Console.WriteLine(count); // 0, 1, 2, 3, 4
    count++;
}

// Kullanıcı girişi kontrolü
string input;
while ((input = Console.ReadLine()) != "exit")
{
    Console.WriteLine($"Girilen: {input}");
}

// Sonsuz döngü
// while (true)
// {
//     // Sonsuz döngü
// }
```

### do-while Döngüsü

```csharp
// Temel do-while döngüsü
int count = 0;
do
{
    Console.WriteLine(count); // 0, 1, 2, 3, 4
    count++;
} while (count < 5);

// En az bir kez çalışma garantisi
int x = 10;
do
{
    Console.WriteLine("Bu blok en az bir kez çalışır.");
} while (x < 5);

// Kullanıcı girişi kontrolü
string input;
do
{
    Console.Write("Komut girin (çıkmak için 'exit'): ");
    input = Console.ReadLine();
    Console.WriteLine($"Girilen: {input}");
} while (input != "exit");
```

## 4. foreach Döngüsü ve IEnumerable

foreach döngüsü, koleksiyonlar üzerinde dolaşmak için kullanılır ve IEnumerable arayüzünü uygulayan herhangi bir koleksiyon ile çalışır.

### Temel foreach Kullanımı

```csharp
// Dizi üzerinde foreach
int[] numbers = { 1, 2, 3, 4, 5 };
foreach (int number in numbers)
{
    Console.WriteLine(number);
}

// List üzerinde foreach
List<string> names = new List<string> { "Alice", "Bob", "Charlie" };
foreach (string name in names)
{
    Console.WriteLine(name);
}

// Dictionary üzerinde foreach
Dictionary<string, int> ages = new Dictionary<string, int>
{
    { "Alice", 25 },
    { "Bob", 30 },
    { "Charlie", 35 }
};

foreach (KeyValuePair<string, int> pair in ages)
{
    Console.WriteLine($"{pair.Key}: {pair.Value}");
}

// C# 7.0+ ile deconstruction
foreach (var (name, age) in ages)
{
    Console.WriteLine($"{name}: {age}");
}
```

### IEnumerable ile Çalışma

```csharp
// IEnumerable döndüren metot
IEnumerable<int> GetNumbers()
{
    yield return 1;
    yield return 2;
    yield return 3;
}

// IEnumerable üzerinde foreach
foreach (int number in GetNumbers())
{
    Console.WriteLine(number);
}

// LINQ ile IEnumerable
IEnumerable<int> evenNumbers = numbers.Where(n => n % 2 == 0);
foreach (int number in evenNumbers)
{
    Console.WriteLine(number);
}
```

### foreach vs for

```csharp
int[] numbers = { 1, 2, 3, 4, 5 };

// foreach - daha okunabilir, ama daha az esnek
foreach (int number in numbers)
{
    Console.WriteLine(number);
}

// for - daha esnek, indekse erişim sağlar
for (int i = 0; i < numbers.Length; i++)
{
    Console.WriteLine($"Index {i}: {numbers[i]}");
}
```

## 5. break, continue, return, goto İfadeleri

Bu ifadeler, döngülerin ve metodların akışını kontrol etmek için kullanılır.

### break İfadesi

```csharp
// Döngüden çıkma
for (int i = 0; i < 10; i++)
{
    if (i == 5)
    {
        break; // i 5 olduğunda döngüden çık
    }
    Console.WriteLine(i); // 0, 1, 2, 3, 4
}

// switch-case'den çıkma
int day = 3;
switch (day)
{
    case 1:
        Console.WriteLine("Pazartesi");
        break;
    case 2:
        Console.WriteLine("Salı");
        break;
    case 3:
        Console.WriteLine("Çarşamba");
        break;
    default:
        Console.WriteLine("Diğer gün");
        break;
}
```

### continue İfadesi

```csharp
// Döngünün geri kalanını atlama
for (int i = 0; i < 10; i++)
{
    if (i % 2 == 0)
    {
        continue; // Çift sayıları atla
    }
    Console.WriteLine(i); // 1, 3, 5, 7, 9
}

// İç içe döngülerde continue
for (int i = 0; i < 3; i++)
{
    for (int j = 0; j < 3; j++)
    {
        if (i == j)
        {
            continue; // Sadece iç döngüyü atlar
        }
        Console.WriteLine($"i = {i}, j = {j}");
    }
}
```

### return İfadesi

```csharp
// Metottan çıkma
int Add(int a, int b)
{
    return a + b; // Metodu sonlandırır ve değer döndürür
}

// Koşullu return
bool IsEven(int number)
{
    if (number % 2 == 0)
    {
        return true;
    }
    return false;
}

// Void metotlarda return
void ProcessData(int[] data)
{
    if (data == null || data.Length == 0)
    {
        return; // Metodu sonlandırır, değer döndürmez
    }
    
    // İşlemler...
}
```

### goto İfadesi

```csharp
// Etiketli goto
int i = 0;
start:
if (i < 5)
{
    Console.WriteLine(i);
    i++;
    goto start;
}

// switch-case içinde goto
int option = 2;
switch (option)
{
    case 1:
        Console.WriteLine("Seçenek 1");
        break;
    case 2:
        Console.WriteLine("Seçenek 2");
        goto case 3; // case 3'e atla
    case 3:
        Console.WriteLine("Seçenek 3");
        break;
    default:
        Console.WriteLine("Diğer seçenek");
        break;
}
```

> **Not:** `goto` ifadesi genellikle kötü bir programlama pratiği olarak kabul edilir ve kullanımından kaçınılmalıdır. Kodun okunabilirliğini azaltır ve "spagetti kod" oluşumuna neden olabilir.

## 6. try-catch-finally ve Exception Handling

try-catch-finally blokları, hataları yakalamak ve yönetmek için kullanılır.

### Temel try-catch Kullanımı

```csharp
try
{
    int a = 10;
    int b = 0;
    int result = a / b; // Bu satır bir DivideByZeroException fırlatacak
    Console.WriteLine(result); // Bu satır çalışmayacak
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Hata: Sıfıra bölme hatası. {ex.Message}");
}
```

### Çoklu catch Blokları

```csharp
try
{
    string input = Console.ReadLine();
    int number = int.Parse(input);
    int result = 10 / number;
    Console.WriteLine(result);
}
catch (FormatException ex)
{
    Console.WriteLine($"Hata: Geçersiz format. {ex.Message}");
}
catch (DivideByZeroException ex)
{
    Console.WriteLine($"Hata: Sıfıra bölme hatası. {ex.Message}");
}
catch (Exception ex) // Genel exception handler - en sonda olmalı
{
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
    Console.WriteLine($"Hata: Dosya bulunamadı. {ex.Message}");
}
catch (IOException ex)
{
    Console.WriteLine($"Hata: Dosya işlemi başarısız. {ex.Message}");
}
finally
{
    // finally bloğu her durumda çalışır (hata olsun veya olmasın)
    file?.Dispose(); // Dosyayı kapat
    Console.WriteLine("Dosya kapatıldı.");
}
```

### Exception Filtreleme (C# 6.0+)

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
catch (IOException ex)
{
    Console.WriteLine($"Genel IO hatası: {ex.Message}");
}
```

### throw İfadesi

```csharp
// Yeni bir exception fırlatma
void ValidateAge(int age)
{
    if (age < 0)
    {
        throw new ArgumentException("Yaş negatif olamaz.", nameof(age));
    }
    
    if (age > 120)
    {
        throw new ArgumentOutOfRangeException(nameof(age), "Yaş çok büyük.");
    }
}

// Exception'ı yeniden fırlatma
try
{
    ValidateAge(-5);
}
catch (ArgumentException ex)
{
    Console.WriteLine($"Doğrulama hatası: {ex.Message}");
    throw; // Orijinal exception'ı stack trace ile birlikte yeniden fırlat
}
```

### Custom Exception Sınıfları

```csharp
// Özel exception sınıfı
public class UserNotFoundException : Exception
{
    public string Username { get; }
    
    public UserNotFoundException(string username)
        : base($"Kullanıcı bulunamadı: {username}")
    {
        Username = username;
    }
    
    public UserNotFoundException(string username, Exception innerException)
        : base($"Kullanıcı bulunamadı: {username}", innerException)
    {
        Username = username;
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

### Exception Yeniden Fırlatma (Re-throwing)

```csharp
// YANLIŞ - Exception stack trace'i kaybeder
try
{
    SomeMethod();
}
catch (Exception ex)
{
    Logger.Log(ex);
    throw ex; // YANLIŞ: Orijinal stack trace kaybedilir
}

// DOĞRU - Orijinal exception'ı korur
try
{
    SomeMethod();
}
catch (Exception ex)
{
    Logger.Log(ex);
    throw; // DOĞRU: Orijinal exception ve stack trace korunur
}

// Yeni exception ile sarmalama
try
{
    SomeMethod();
}
catch (SqlException ex)
{
    throw new ApplicationException("Veritabanı hatası oluştu", ex); // İç exception olarak orijinal hatayı saklar
}
```

Exception yeniden fırlatırken `throw ex` yerine sadece `throw` kullanmak önemlidir. `throw ex` kullanıldığında, orijinal exception'ın stack trace'i kaybedilir ve hata noktasını bulmak zorlaşır. Sadece `throw` kullanıldığında, orijinal exception ve stack trace korunur.

## 7. using Statement ve IDisposable Pattern

using statement, IDisposable arayüzünü uygulayan nesnelerin otomatik olarak dispose edilmesini sağlar.

### Temel using Statement

```csharp
// Geleneksel using bloğu
using (StreamReader reader = new StreamReader("file.txt"))
{
    string content = reader.ReadToEnd();
    Console.WriteLine(content);
} // reader.Dispose() otomatik olarak çağrılır

// Çoklu using
using (FileStream fileStream = File.OpenRead("file.txt"))
using (StreamReader reader = new StreamReader(fileStream))
{
    string content = reader.ReadToEnd();
    Console.WriteLine(content);
}
```

### using Declaration (C# 8.0+)

```csharp
// using declaration - blok sonunda dispose edilir
void ProcessFile(string path)
{
    using FileStream fileStream = File.OpenRead(path);
    using StreamReader reader = new StreamReader(fileStream);
    
    string content = reader.ReadToEnd();
    Console.WriteLine(content);
} // fileStream ve reader burada dispose edilir
```

### IDisposable Pattern Uygulama

```csharp
public class ResourceManager : IDisposable
{
    private bool _disposed = false;
    private FileStream _fileStream;
    
    public ResourceManager(string path)
    {
        _fileStream = File.OpenRead(path);
    }
    
    // Public Dispose metodu
    public void Dispose()
    {
        Dispose(true);
        GC.SuppressFinalize(this);
    }
    
    // Protected virtual Dispose metodu
    protected virtual void Dispose(bool disposing)
    {
        if (!_disposed)
        {
            if (disposing)
            {
                // Managed kaynakları temizle
                _fileStream?.Dispose();
            }
            
            // Unmanaged kaynakları temizle
            
            _disposed = true;
        }
    }
    
    // Finalizer
    ~ResourceManager()
    {
        Dispose(false);
    }
    
    // Diğer metotlar
    public string ReadContent()
    {
        if (_disposed)
        {
            throw new ObjectDisposedException(nameof(ResourceManager));
        }
        
        using var reader = new StreamReader(_fileStream);
        return reader.ReadToEnd();
    }
}

// Kullanım
using (var manager = new ResourceManager("file.txt"))
{
    string content = manager.ReadContent();
    Console.WriteLine(content);
}
```

### using vs try-finally

```csharp
// using statement
using (var resource = new DisposableResource())
{
    resource.DoWork();
}

// Eşdeğer try-finally bloğu
DisposableResource resource = null;
try
{
    resource = new DisposableResource();
    resource.DoWork();
}
finally
{
    resource?.Dispose();
}
```

## En İyi Pratikler

1. **if-else İfadeleri**
   - Karmaşık koşulları parçalara ayırın ve anlamlı değişken isimleri kullanın.
   - Çok fazla iç içe if-else kullanmaktan kaçının, kodun okunabilirliğini azaltır.
   - Null kontrolü için null-conditional operatörü (`?.`) ve null-coalescing operatörü (`??`) kullanmayı düşünün.

2. **switch-case Yapısı**
   - Çok sayıda if-else ifadesi yerine switch-case kullanmayı düşünün.
   - C# 8.0+ ile switch expression kullanarak daha kısa ve okunabilir kod yazın.
   - Pattern matching özelliklerini kullanarak daha güçlü ve esnek switch ifadeleri oluşturun.

3. **Döngüler**
   - Doğru döngü tipini seçin: Belirli sayıda tekrar için `for`, koşula bağlı tekrar için `while`, en az bir kez çalışması gereken döngüler için `do-while`.
   - Koleksiyonlar üzerinde dolaşmak için `foreach` kullanın.
   - Sonsuz döngülerde mutlaka bir çıkış koşulu olduğundan emin olun.

4. **Exception Handling**
   - Sadece gerçekten ele alabileceğiniz hataları yakalayın.
   - Çok genel exception yakalamaktan kaçının (`catch (Exception ex)`).
   - Kaynakları temizlemek için `finally` bloğu veya `using` statement kullanın.
   - Custom exception sınıfları oluştururken, anlamlı hata mesajları ve ek bilgiler sağlayın.

5. **IDisposable Pattern**
   - Unmanaged kaynakları kullanan sınıflar için IDisposable pattern'i uygulayın.
   - IDisposable nesneleri her zaman `using` statement ile kullanın.
   - Dispose edilmiş nesnelere erişimi kontrol edin ve ObjectDisposedException fırlatın.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Boş catch Blokları**
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

2. **goto Kullanımı**
   ```csharp
   // Kötü - goto kullanımı
   int i = 0;
   start:
   Console.WriteLine(i);
   i++;
   if (i < 5) goto start;
   
   // İyi - döngü kullanımı
   for (int i = 0; i < 5; i++)
   {
       Console.WriteLine(i);
   }
   ```

3. **Gereksiz Nested if-else**
   ```csharp
   // Kötü - çok fazla iç içe if-else
   if (condition1)
   {
       if (condition2)
       {
           if (condition3)
           {
               // İşlem
           }
       }
   }
   
   // İyi - düzleştirilmiş koşullar
   if (!condition1 || !condition2 || !condition3)
   {
       return; // Erken çıkış
   }
   
   // İşlem
   ```

4. **Kaynakları Dispose Etmemek**
   ```csharp
   // Kötü - dispose edilmeyen kaynaklar
   FileStream file = File.Open("file.txt", FileMode.Open);
   // İşlemler...
   
   // İyi - using statement
   using (FileStream file = File.Open("file.txt", FileMode.Open))
   {
       // İşlemler...
   }
   ```

5. **Yanlış Exception Handling**
   ```csharp
   // Kötü - exception'ı yutmak ve devam etmek
   try
   {
       int result = 10 / 0;
   }
   catch
   {
       // Hata yutuldu, program devam ediyor
   }
   
   // İyi - anlamlı hata mesajı ve uygun işlem
   try
   {
       int result = 10 / 0;
   }
   catch (DivideByZeroException ex)
   {
       Console.WriteLine("Sıfıra bölme hatası oluştu.");
       // Alternatif işlem veya kullanıcıya bildirim
   }
   ```

Kontrol yapıları, C# programlamanın temel yapı taşlarıdır. Bu yapıları doğru ve etkili bir şekilde kullanmak, daha güvenilir, okunabilir ve bakımı kolay kod yazmanıza yardımcı olacaktır. 