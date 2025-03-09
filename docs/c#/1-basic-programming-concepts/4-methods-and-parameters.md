# Metotlar ve Parametreler

Metotlar, C# programlamanın temel yapı taşlarından biridir. Belirli bir işlevi yerine getiren kod bloklarını tanımlar ve kodun yeniden kullanılabilirliğini sağlar. Bu bölümde, C#'taki metot tanımlama, çağırma ve parametre kullanımı konularını detaylı olarak inceleyeceğiz.

## 1. Metot Tanımlama ve Çağırma

Metotlar, belirli bir işlevi yerine getiren ve yeniden kullanılabilen kod bloklarıdır.

### Temel Metot Tanımlama

```csharp
// Geri dönüş değeri olmayan metot (void)
void SayHello()
{
    Console.WriteLine("Merhaba!");
}

// Geri dönüş değeri olan metot
int Add(int a, int b)
{
    return a + b;
}

// Metotları çağırma
SayHello(); // Çıktı: Merhaba!
int sum = Add(5, 3); // sum = 8
```

### Access Modifiers (Erişim Belirleyiciler)

```csharp
// Public metot - her yerden erişilebilir
public void PublicMethod()
{
    Console.WriteLine("Bu bir public metottur.");
}

// Private metot - sadece tanımlandığı sınıf içinden erişilebilir
private void PrivateMethod()
{
    Console.WriteLine("Bu bir private metottur.");
}

// Protected metot - tanımlandığı sınıf ve alt sınıflardan erişilebilir
protected void ProtectedMethod()
{
    Console.WriteLine("Bu bir protected metottur.");
}

// Internal metot - aynı assembly içinden erişilebilir
internal void InternalMethod()
{
    Console.WriteLine("Bu bir internal metottur.");
}

// Protected internal - aynı assembly veya alt sınıflardan erişilebilir
protected internal void ProtectedInternalMethod()
{
    Console.WriteLine("Bu bir protected internal metottur.");
}

// Private protected (C# 7.2+) - aynı assembly içindeki alt sınıflardan erişilebilir
private protected void PrivateProtectedMethod()
{
    Console.WriteLine("Bu bir private protected metottur.");
}
```

### Static ve Instance Metotlar

```csharp
public class Calculator
{
    // Instance metot - nesne örneği gerektirir
    public int Add(int a, int b)
    {
        return a + b;
    }
    
    // Static metot - sınıf adı ile çağrılır
    public static int Multiply(int a, int b)
    {
        return a * b;
    }
}

// Kullanım
Calculator calc = new Calculator();
int sum = calc.Add(5, 3); // Instance metot çağrısı

int product = Calculator.Multiply(5, 3); // Static metot çağrısı
```

## 2. Parametre Türleri

C#'ta farklı parametre türleri bulunur ve her biri farklı senaryolarda kullanışlıdır.

### Value Parameters (Değer Parametreleri)

```csharp
// Değer parametresi - parametrenin bir kopyası metoda geçirilir
void IncrementValue(int number)
{
    number++; // Sadece yerel kopya değişir
    Console.WriteLine($"Metot içinde: {number}");
}

int x = 5;
IncrementValue(x); // Çıktı: Metot içinde: 6
Console.WriteLine($"Metot sonrası: {x}"); // Çıktı: Metot sonrası: 5 (değişmedi)
```

### ref Parameters (Referans Parametreleri)

```csharp
// ref parametresi - değişkenin referansı metoda geçirilir
void IncrementRef(ref int number)
{
    number++; // Orijinal değişken değişir
    Console.WriteLine($"Metot içinde: {number}");
}

int x = 5;
IncrementRef(ref x); // Çıktı: Metot içinde: 6
Console.WriteLine($"Metot sonrası: {x}"); // Çıktı: Metot sonrası: 6 (değişti)

// ref parametresi kullanırken, değişkenin metot çağrılmadan önce başlatılması gerekir
int y; // Başlatılmamış
// IncrementRef(ref y); // Derleme hatası: Başlatılmamış değişken
```

### out Parameters (Çıkış Parametreleri)

```csharp
// out parametresi - metot içinde değer atanması zorunludur
bool TryParse(string input, out int result)
{
    if (int.TryParse(input, out result))
    {
        return true;
    }
    
    result = 0; // out parametresine değer atanmalıdır
    return false;
}

// Kullanım
string input = "123";
if (TryParse(input, out int number))
{
    Console.WriteLine($"Başarılı: {number}"); // Çıktı: Başarılı: 123
}

// C# 7.0+ ile inline out değişken tanımlama
if (int.TryParse("456", out int anotherNumber))
{
    Console.WriteLine(anotherNumber); // Çıktı: 456
}

// Çoklu out parametresi
void GetMinMax(int[] numbers, out int min, out int max)
{
    min = numbers.Min();
    max = numbers.Max();
}

int[] values = { 5, 3, 9, 1, 7 };
GetMinMax(values, out int minimum, out int maximum);
Console.WriteLine($"Min: {minimum}, Max: {maximum}"); // Çıktı: Min: 1, Max: 9
```

### in Parameters (Giriş Parametreleri - C# 7.2+)

```csharp
// in parametresi - salt okunur referans parametresi
void PrintDistance(in Point point)
{
    // point.X = 10; // Derleme hatası: in parametresi değiştirilemez
    double distance = Math.Sqrt(point.X * point.X + point.Y * point.Y);
    Console.WriteLine($"Uzaklık: {distance}");
}

var point = new Point(3, 4);
PrintDistance(in point); // Çıktı: Uzaklık: 5

// in parametresi, özellikle büyük struct'lar için performans avantajı sağlar
void ProcessLargeStruct(in LargeStruct data)
{
    // data değiştirilemez, ancak kopyalanmaz (performans avantajı)
}
```

### Params Array (Değişken Sayıda Parametre)

```csharp
// params anahtar kelimesi - değişken sayıda parametre
int Sum(params int[] numbers)
{
    return numbers.Sum();
}

// Kullanım
int total1 = Sum(1, 2, 3); // 6
int total2 = Sum(10, 20, 30, 40); // 100
int total3 = Sum(); // 0

// Diğer parametrelerle birlikte kullanım (params son parametre olmalıdır)
string Format(string format, params object[] args)
{
    return string.Format(format, args);
}

string message = Format("Merhaba, {0}! Bugün {1}.", "Ahmet", "Pazartesi");
```

## 3. Method Overloading ve Resolution

Method overloading, aynı isimde ancak farklı parametre listesine sahip birden fazla metot tanımlamayı sağlar.

### Temel Method Overloading

```csharp
// Farklı parametre sayısı ile overloading
void Display(int number)
{
    Console.WriteLine($"Sayı: {number}");
}

void Display(int number1, int number2)
{
    Console.WriteLine($"Sayılar: {number1}, {number2}");
}

// Farklı parametre tipleri ile overloading
void Display(string text)
{
    Console.WriteLine($"Metin: {text}");
}

void Display(double number)
{
    Console.WriteLine($"Ondalıklı: {number}");
}

// Kullanım
Display(5); // Çıktı: Sayı: 5
Display(5, 10); // Çıktı: Sayılar: 5, 10
Display("Merhaba"); // Çıktı: Metin: Merhaba
Display(3.14); // Çıktı: Ondalıklı: 3.14
```

### Method Resolution (Metot Çözümleme)

```csharp
// Metot çözümleme örneği
void Process(int value) { Console.WriteLine("int"); }
void Process(long value) { Console.WriteLine("long"); }
void Process(object value) { Console.WriteLine("object"); }

// Çağrılar
Process(5); // Çıktı: int (tam eşleşme)
Process(5L); // Çıktı: long (tam eşleşme)
Process("text"); // Çıktı: object (string -> object dönüşümü)

// Belirsiz çağrı örneği
// void Ambiguous(int value, double option = 0) { }
// void Ambiguous(double value, int option = 0) { }
// Ambiguous(5); // Derleme hatası: Çağrı belirsiz
```

## 4. Optional ve Named Parametreler

Optional parametreler, metot çağrısında belirtilmesi zorunlu olmayan parametrelerdir. Named parametreler, parametreleri isimleriyle belirtmeyi sağlar.

### Optional Parametreler

```csharp
// Optional parametreler - varsayılan değerler atanır
void ShowInfo(string name, int age = 30, string city = "İstanbul")
{
    Console.WriteLine($"İsim: {name}, Yaş: {age}, Şehir: {city}");
}

// Kullanım
ShowInfo("Ali"); // Çıktı: İsim: Ali, Yaş: 30, Şehir: İstanbul
ShowInfo("Ayşe", 25); // Çıktı: İsim: Ayşe, Yaş: 25, Şehir: İstanbul
ShowInfo("Mehmet", 40, "Ankara"); // Çıktı: İsim: Mehmet, Yaş: 40, Şehir: Ankara

// Optional parametreler sonda olmalıdır
// void Invalid(int age = 30, string name) { } // Derleme hatası
```

### Named Parametreler

```csharp
// Named parametreler - parametreleri isimleriyle belirtme
void Configure(string server, string database, bool useSSL = true, int timeout = 30)
{
    Console.WriteLine($"Server: {server}, DB: {database}, SSL: {useSSL}, Timeout: {timeout}");
}

// Kullanım
Configure("localhost", "mydb"); // Pozisyonel parametreler
Configure(server: "localhost", database: "mydb"); // Named parametreler
Configure(database: "mydb", server: "localhost"); // Sıra önemli değil

// Named ve pozisyonel parametreleri karıştırma
Configure("localhost", database: "mydb");

// Optional parametreleri atlama
Configure("localhost", "mydb", timeout: 60); // useSSL varsayılan değeri kullanılır
```

## 5. Extension Metotlar

Extension metotlar, mevcut tiplere yeni metotlar eklemeyi sağlar. Bu metotlar, static bir sınıf içinde tanımlanır ve ilk parametresi `this` anahtar kelimesi ile işaretlenir.

### Temel Extension Metot Tanımlama

```csharp
// Extension metotlar static bir sınıf içinde tanımlanmalıdır
public static class StringExtensions
{
    // String sınıfına extension metot ekleme
    public static bool IsNullOrWhiteSpace(this string value)
    {
        return string.IsNullOrWhiteSpace(value);
    }
    
    // Parametreli extension metot
    public static string Truncate(this string value, int maxLength)
    {
        if (string.IsNullOrEmpty(value)) return value;
        return value.Length <= maxLength ? value : value.Substring(0, maxLength) + "...";
    }
}

// Kullanım
string text = "Merhaba Dünya";
bool isEmpty = text.IsNullOrWhiteSpace(); // false
string truncated = text.Truncate(7); // "Merhaba..."

// Null değer ile kullanım
string nullText = null;
bool isNullEmpty = nullText.IsNullOrWhiteSpace(); // true
```

### LINQ Extension Metotları

```csharp
// LINQ extension metotları
List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };

// Where extension metodu
var evenNumbers = numbers.Where(n => n % 2 == 0); // 2, 4

// Select extension metodu
var squaredNumbers = numbers.Select(n => n * n); // 1, 4, 9, 16, 25

// Zincirleme extension metot çağrıları
var result = numbers
    .Where(n => n > 2)
    .Select(n => n * 2)
    .OrderByDescending(n => n);
// 10, 8, 6
```

## 6. Local Functions (C# 7.0+)

Local functions, bir metot içinde tanımlanan ve sadece o metot içinde kullanılabilen fonksiyonlardır.

### Temel Local Function Tanımlama

```csharp
void ProcessData(int[] data)
{
    // Local function tanımlama
    int Calculate(int value)
    {
        return value * value;
    }
    
    // Local function kullanımı
    foreach (var item in data)
    {
        int result = Calculate(item);
        Console.WriteLine(result);
    }
}

// Dışarıdan erişilemez
// Calculate(5); // Derleme hatası
```

### Closure ile Local Functions

```csharp
void ProcessWithState()
{
    int factor = 10;
    
    // Local function, dış değişkenlere erişebilir (closure)
    int Multiply(int value)
    {
        return value * factor;
    }
    
    Console.WriteLine(Multiply(5)); // 50
    
    factor = 20;
    Console.WriteLine(Multiply(5)); // 100 (güncel factor değeri kullanılır)
}
```

### Recursive Local Functions

```csharp
int CalculateFactorial(int n)
{
    // Recursive local function
    int Factorial(int number)
    {
        if (number <= 1) return 1;
        return number * Factorial(number - 1);
    }
    
    if (n < 0) throw new ArgumentException("Negatif sayılar için faktöriyel hesaplanamaz.");
    return Factorial(n);
}

Console.WriteLine(CalculateFactorial(5)); // 120
```

## 7. Expression-Bodied Members

Expression-bodied members, metot ve property tanımlarını daha kısa ve öz bir şekilde yazmayı sağlar.

### Expression-Bodied Methods

```csharp
// Geleneksel metot tanımı
public int Add(int a, int b)
{
    return a + b;
}

// Expression-bodied metot (C# 6.0+)
public int Add(int a, int b) => a + b;

// Void metotlar için (C# 7.0+)
public void PrintSum(int a, int b) => Console.WriteLine(a + b);
```

### Expression-Bodied Properties ve Indexers

```csharp
// Expression-bodied property (C# 6.0+)
public string FullName => $"{FirstName} {LastName}";

// Expression-bodied property setter (C# 7.0+)
private string _name;
public string Name
{
    get => _name;
    set => _name = value ?? throw new ArgumentNullException(nameof(value));
}

// Expression-bodied indexer
public int this[int index] => _data[index];
```

### Expression-Bodied Constructors ve Finalizers (C# 7.0+)

```csharp
public class Person
{
    private readonly string _name;
    private readonly int _age;
    
    // Expression-bodied constructor
    public Person(string name, int age) => (_name, _age) = (name, age);
    
    // Expression-bodied finalizer
    ~Person() => Console.WriteLine($"{_name} is being finalized");
}
```

## 8. Tuples ve Deconstruction

Tuple, birden fazla değeri tek bir veri yapısında gruplamayı sağlar. Deconstruction, tuple veya özel nesnelerin bileşenlerine ayrılmasını sağlar.

### Tuple Literals (C# 7.0+)

```csharp
// Tuple literal oluşturma
var person = ("John", 30);
Console.WriteLine($"Name: {person.Item1}, Age: {person.Item2}");

// Named tuple elements
var employee = (Name: "Alice", Age: 25, Department: "IT");
Console.WriteLine($"Name: {employee.Name}, Dept: {employee.Department}");

// Tuple dönüş değeri
(string, int) GetPersonInfo()
{
    return ("Bob", 35);
}

var info = GetPersonInfo();
Console.WriteLine($"Name: {info.Item1}, Age: {info.Item2}");

// Named tuple dönüş değeri
(string Name, int Age) GetEmployeeInfo()
{
    return (Name: "Charlie", Age: 40);
}

var empInfo = GetEmployeeInfo();
Console.WriteLine($"Name: {empInfo.Name}, Age: {empInfo.Age}");
```

### Tuple Deconstruction

```csharp
// Tuple deconstruction
var person = ("John", 30);
var (name, age) = person;
Console.WriteLine($"Name: {name}, Age: {age}");

// Metot dönüş değerini doğrudan deconstruct etme
(string, int) GetPersonInfo() => ("Bob", 35);
var (empName, empAge) = GetPersonInfo();

// Kısmi deconstruction
var (firstName, _) = ("Alice", 25); // Sadece isim alınır, yaş yok sayılır

// Mevcut değişkenlere deconstruct etme
string customerName;
int customerAge;
(customerName, customerAge) = ("David", 45);
```

### Custom Type Deconstruction

```csharp
public class Point
{
    public int X { get; }
    public int Y { get; }
    
    public Point(int x, int y) => (X, Y) = (x, y);
    
    // Deconstruct metodu tanımlama
    public void Deconstruct(out int x, out int y) => (x, y) = (X, Y);
}

// Kullanım
var point = new Point(10, 20);
var (x, y) = point; // Deconstruct metodu çağrılır
Console.WriteLine($"X: {x}, Y: {y}");

// Extension metot olarak deconstruct tanımlama
public static class PersonExtensions
{
    public static void Deconstruct(this Person person, out string name, out int age)
    {
        name = person.Name;
        age = person.Age;
    }
}
```

## En İyi Pratikler

1. **Metot İsimlendirme**
   - Metot isimleri, yaptıkları işi açıkça belirtmeli (örn. `CalculateTotalPrice`, `GetUserById`).
   - Fiil veya fiil-isim kombinasyonu kullanın.
   - Pascal case kullanın (her kelimenin ilk harfi büyük).

2. **Parametre Sayısı**
   - Metotlar genellikle 3-4'ten fazla parametre almamalıdır.
   - Çok sayıda parametre gerekiyorsa, bunları bir sınıf veya struct içinde gruplamayı düşünün.

3. **Metot Uzunluğu**
   - Metotlar tek bir işlevi yerine getirmeli ve genellikle 20-30 satırdan uzun olmamalıdır.
   - Uzun metotları daha küçük, özel amaçlı metotlara bölün.

4. **Extension Metotlar**
   - Extension metotları, mevcut tiplerin doğal davranışlarını genişletmek için kullanın.
   - Framework sınıflarını taklit eden extension metotlar yazmaktan kaçının.

5. **ref, out ve in Parametreleri**
   - `ref` ve `out` parametrelerini sadece gerektiğinde kullanın, kodun okunabilirliğini azaltabilirler.
   - Büyük struct'lar için performans optimizasyonu amacıyla `in` parametrelerini kullanın.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Çok Fazla Parametre**
   ```csharp
   // Kötü - çok fazla parametre
   void ConfigureConnection(string server, string database, string username, string password, 
                           bool useSSL, int timeout, bool pooling, int maxPoolSize)
   {
       // ...
   }
   
   // İyi - parametreleri bir sınıfta gruplandırma
   void ConfigureConnection(ConnectionOptions options)
   {
       // ...
   }
   ```

2. **Side Effects (Yan Etkiler)**
   ```csharp
   // Kötü - beklenmeyen yan etkiler
   int Calculate(int value)
   {
       _globalState = value; // Yan etki: global durumu değiştiriyor
       return value * 2;
   }
   
   // İyi - yan etki yok, sadece girdi parametrelerine bağlı
   int Calculate(int value)
   {
       return value * 2;
   }
   ```

3. **Gereksiz out Parametreleri**
   ```csharp
   // Kötü - gereksiz out parametresi
   bool TryGetUserInfo(int userId, out string name, out int age, out string email)
   {
       // ...
   }
   
   // İyi - tuple dönüş değeri
   (bool Success, string Name, int Age, string Email) GetUserInfo(int userId)
   {
       // ...
   }
   ```

4. **Extension Metotların Yanlış Kullanımı**
   ```csharp
   // Kötü - uygunsuz extension metot
   public static void SaveToDatabase(this string text)
   {
       // String sınıfı için uygun olmayan bir işlem
   }
   
   // İyi - uygun extension metot
   public static string Truncate(this string text, int maxLength)
   {
       // String işlemi için uygun
   }
   ```

5. **Metot Aşırı Yükleme Karmaşası**
   ```csharp
   // Kötü - kafa karıştırıcı overload'lar
   void Process(int value);
   void Process(int value, bool flag);
   void Process(int value, string option);
   void Process(int value, string option, bool flag);
   
   // İyi - optional parametreler kullanma
   void Process(int value, string option = null, bool flag = false);
   ```

Metotlar ve parametreler, C# programlamanın temel yapı taşlarıdır. Bu kavramları doğru ve etkili bir şekilde kullanmak, daha modüler, bakımı kolay ve okunabilir kod yazmanıza yardımcı olacaktır. 