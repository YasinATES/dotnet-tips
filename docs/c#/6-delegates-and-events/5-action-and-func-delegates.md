# Action ve Func Delegates

Bu bölümde, C#'ın yerleşik delege türleri olan `Action<T>` ve `Func<T, TResult>` delegelerini inceleyeceğiz. Bu delegeler, .NET Framework'te yaygın olarak kullanılan ve kod yazarken delege tanımlamayı kolaylaştıran önceden tanımlanmış delege türleridir.

## 1. Action Delegate Types

`Action` delegeleri, değer döndürmeyen (void) metotları temsil eder. Farklı sayıda parametre alan çeşitli `Action` türleri vardır.

### Temel Action Kullanımı

```csharp
public class ActionExample
{
    public static void Main()
    {
        // Parametresiz Action
        Action sayHello = () => Console.WriteLine("Merhaba Dünya!");
        
        // Tek parametreli Action<T>
        Action<string> greet = name => Console.WriteLine($"Merhaba, {name}!");
        
        // İki parametreli Action<T1, T2>
        Action<string, int> repeatGreeting = (name, count) =>
        {
            for (int i = 0; i < count; i++)
            {
                Console.WriteLine($"Merhaba, {name}!");
            }
        };
        
        // Action delegelerini çağırma
        sayHello(); // Merhaba Dünya!
        greet("Ali"); // Merhaba, Ali!
        repeatGreeting("Ayşe", 3); // 3 kez Merhaba, Ayşe!
    }
}
```

### Action Delegelerini Metot Parametresi Olarak Kullanma

```csharp
public class ActionAsParameterExample
{
    // Action parametreli metot
    public static void ProcessItems<T>(IEnumerable<T> items, Action<T> processor)
    {
        foreach (T item in items)
        {
            processor(item);
        }
    }
    
    // Çoklu Action parametreli metot
    public static void ProcessWithCallbacks<T>(
        IEnumerable<T> items, 
        Action<T> processor, 
        Action onStart = null, 
        Action onComplete = null)
    {
        onStart?.Invoke();
        
        foreach (T item in items)
        {
            processor(item);
        }
        
        onComplete?.Invoke();
    }
    
    public static void Main()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
        
        // Basit işleme
        Console.WriteLine("Sayıları yazdırma:");
        ProcessItems(numbers, n => Console.WriteLine(n));
        
        // Geri çağrılarla işleme
        Console.WriteLine("\nGeri çağrılarla işleme:");
        ProcessWithCallbacks(
            numbers,
            n => Console.WriteLine($"İşleniyor: {n}"),
            () => Console.WriteLine("İşlem başladı"),
            () => Console.WriteLine("İşlem tamamlandı")
        );
    }
}
```

### Action Delegelerini Sınıf Üyesi Olarak Kullanma

```csharp
public class ActionAsFieldExample
{
    // Action delegelerini sınıf üyesi olarak tanımlama
    private Action _initializeAction;
    private Action<string> _logAction;
    private Action _cleanupAction;
    
    public ActionAsFieldExample()
    {
        // Delegeleri başlatma
        _initializeAction = () => Console.WriteLine("Sistem başlatılıyor...");
        _logAction = message => Console.WriteLine($"LOG: {message}");
        _cleanupAction = () => Console.WriteLine("Temizleme işlemi yapılıyor...");
    }
    
    public void Run()
    {
        _initializeAction();
        
        _logAction("İşlem 1 başladı");
        // İş mantığı...
        _logAction("İşlem 1 tamamlandı");
        
        _logAction("İşlem 2 başladı");
        // İş mantığı...
        _logAction("İşlem 2 tamamlandı");
        
        _cleanupAction();
    }
    
    public static void Main()
    {
        var example = new ActionAsFieldExample();
        example.Run();
    }
}
```

## 2. Func Delegate Types

`Func` delegeleri, değer döndüren metotları temsil eder. Son tip parametresi her zaman dönüş değerinin türünü belirtir.

### Temel Func Kullanımı

```csharp
public class FuncExample
{
    public static void Main()
    {
        // Parametresiz, int döndüren Func<TResult>
        Func<int> getRandomNumber = () => new Random().Next(1, 100);
        
        // Bir parametre alan, bool döndüren Func<T, TResult>
        Func<int, bool> isEven = number => number % 2 == 0;
        
        // İki parametre alan, int döndüren Func<T1, T2, TResult>
        Func<int, int, int> add = (x, y) => x + y;
        
        // Func delegelerini çağırma
        int randomNum = getRandomNumber();
        Console.WriteLine($"Rastgele sayı: {randomNum}");
        
        bool isRandomEven = isEven(randomNum);
        Console.WriteLine($"{randomNum} çift mi? {isRandomEven}");
        
        int sum = add(3, 5);
        Console.WriteLine($"3 + 5 = {sum}");
    }
}
```

### Func Delegelerini Metot Parametresi Olarak Kullanma

```csharp
public class FuncAsParameterExample
{
    // Func parametreli metot - dönüşüm yapma
    public static List<TResult> TransformItems<T, TResult>(
        IEnumerable<T> items, Func<T, TResult> transformer)
    {
        List<TResult> results = new List<TResult>();
        
        foreach (T item in items)
        {
            results.Add(transformer(item));
        }
        
        return results;
    }
    
    // Func parametreli metot - filtreleme
    public static List<T> FilterItems<T>(
        IEnumerable<T> items, Func<T, bool> predicate)
    {
        List<T> results = new List<T>();
        
        foreach (T item in items)
        {
            if (predicate(item))
            {
                results.Add(item);
            }
        }
        
        return results;
    }
    
    public static void Main()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
        
        // Dönüştürme
        List<string> numberTexts = TransformItems(numbers, n => $"Sayı: {n}");
        Console.WriteLine("Dönüştürülmüş sayılar:");
        foreach (string text in numberTexts)
        {
            Console.WriteLine(text);
        }
        
        // Filtreleme
        List<int> evenNumbers = FilterItems(numbers, n => n % 2 == 0);
        Console.WriteLine("\nÇift sayılar:");
        foreach (int number in evenNumbers)
        {
            Console.WriteLine(number);
        }
    }
}
```

### Func Delegelerini Sınıf Üyesi Olarak Kullanma

```csharp
public class FuncAsFieldExample
{
    // Func delegelerini sınıf üyesi olarak tanımlama
    private Func<int> _getNextId;
    private Func<string, string> _formatMessage;
    private Func<int, int, double> _calculateAverage;
    
    private int _currentId = 0;
    
    public FuncAsFieldExample()
    {
        // Delegeleri başlatma
        _getNextId = () => ++_currentId;
        _formatMessage = message => $"[{DateTime.Now:HH:mm:ss}] {message}";
        _calculateAverage = (x, y) => (x + y) / 2.0;
    }
    
    public void Run()
    {
        Console.WriteLine(_formatMessage($"Yeni ID: {_getNextId()}"));
        Console.WriteLine(_formatMessage($"Yeni ID: {_getNextId()}"));
        
        double avg = _calculateAverage(10, 20);
        Console.WriteLine(_formatMessage($"Ortalama: {avg}"));
    }
    
    public static void Main()
    {
        var example = new FuncAsFieldExample();
        example.Run();
    }
}
```

## 3. Predicate<T> Usage

`Predicate<T>` delegesi, bir koşulu değerlendiren ve `bool` döndüren özel bir delege türüdür. Aslında `Func<T, bool>` ile eşdeğerdir, ancak amacı daha spesifiktir.

### Temel Predicate Kullanımı

```csharp
public class PredicateExample
{
    public static void Main()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        
        // Predicate<T> kullanımı
        Predicate<int> isEven = n => n % 2 == 0;
        Predicate<int> isGreaterThanFive = n => n > 5;
        
        // Predicate ile filtreleme
        List<int> evenNumbers = numbers.FindAll(isEven);
        Console.WriteLine($"Çift sayılar: {string.Join(", ", evenNumbers)}");
        
        List<int> largeNumbers = numbers.FindAll(isGreaterThanFive);
        Console.WriteLine($"5'ten büyük sayılar: {string.Join(", ", largeNumbers)}");
        
        // Predicate'leri birleştirme
        Predicate<int> isEvenAndLarge = n => isEven(n) && isGreaterThanFive(n);
        List<int> evenAndLargeNumbers = numbers.FindAll(isEvenAndLarge);
        Console.WriteLine($"Çift ve 5'ten büyük sayılar: {string.Join(", ", evenAndLargeNumbers)}");
    }
}
```

### Predicate ile Özel Filtreleme

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string City { get; set; }
    
    public override string ToString() => $"{Name} ({Age}, {City})";
}

public class PredicateFilterExample
{
    public static List<T> Filter<T>(List<T> items, Predicate<T> predicate)
    {
        return items.FindAll(predicate);
    }
    
    public static void Main()
    {
        List<Person> people = new List<Person>
        {
            new Person { Name = "Ali", Age = 25, City = "İstanbul" },
            new Person { Name = "Ayşe", Age = 30, City = "Ankara" },
            new Person { Name = "Mehmet", Age = 20, City = "İzmir" },
            new Person { Name = "Zeynep", Age = 35, City = "İstanbul" },
            new Person { Name = "Fatma", Age = 28, City = "Ankara" }
        };
        
        // Yaşa göre filtreleme
        Predicate<Person> isYoung = p => p.Age < 30;
        List<Person> youngPeople = Filter(people, isYoung);
        
        Console.WriteLine("30 yaş altı kişiler:");
        foreach (Person person in youngPeople)
        {
            Console.WriteLine(person);
        }
        
        // Şehre göre filtreleme
        Predicate<Person> livesInIstanbul = p => p.City == "İstanbul";
        List<Person> istanbulPeople = Filter(people, livesInIstanbul);
        
        Console.WriteLine("\nİstanbul'da yaşayan kişiler:");
        foreach (Person person in istanbulPeople)
        {
            Console.WriteLine(person);
        }
        
        // Birden fazla koşul
        Predicate<Person> isYoungAndFromIstanbul = p => isYoung(p) && livesInIstanbul(p);
        List<Person> youngIstanbulPeople = Filter(people, isYoungAndFromIstanbul);
        
        Console.WriteLine("\n30 yaş altı ve İstanbul'da yaşayan kişiler:");
        foreach (Person person in youngIstanbulPeople)
        {
            Console.WriteLine(person);
        }
    }
}
```

## 4. Custom Delegate Types

Yerleşik delege türleri çoğu durumda yeterli olsa da, bazen özel delege türleri tanımlamak gerekebilir.

### Özel Delege Türü Tanımlama

```csharp
// Özel delege türleri
public delegate bool Validator<T>(T item, string errorMessage, out string error);
public delegate TResult Transformer<T, TResult>(T input, int factor);

public class CustomDelegateExample
{
    public static void Main()
    {
        // Validator kullanımı
        Validator<string> stringValidator = (value, message, out string error) =>
        {
            if (string.IsNullOrEmpty(value))
            {
                error = message;
                return false;
            }
            
            error = null;
            return true;
        };
        
        string input = "";
        string errorMessage;
        bool isValid = stringValidator(input, "Değer boş olamaz", out errorMessage);
        
        Console.WriteLine($"Geçerli mi: {isValid}");
        if (!isValid)
        {
            Console.WriteLine($"Hata: {errorMessage}");
        }
        
        // Transformer kullanımı
        Transformer<int, string> numberTransformer = (number, factor) =>
        {
            int result = number * factor;
            return $"{number} x {factor} = {result}";
        };
        
        string transformResult = numberTransformer(5, 3);
        Console.WriteLine(transformResult);
    }
}
```

### Özel Delege vs Yerleşik Delege

```csharp
public class CustomVsBuiltInExample
{
    // Özel delege
    public delegate TResult Calculator<T1, T2, TResult>(T1 a, T2 b);
    
    // Özel delege ile metot
    public static void ProcessWithCustomDelegate<T1, T2, TResult>(
        T1 a, T2 b, Calculator<T1, T2, TResult> calculator)
    {
        TResult result = calculator(a, b);
        Console.WriteLine($"Özel delege sonucu: {result}");
    }
    
    // Yerleşik delege ile metot
    public static void ProcessWithBuiltInDelegate<T1, T2, TResult>(
        T1 a, T2 b, Func<T1, T2, TResult> calculator)
    {
        TResult result = calculator(a, b);
        Console.WriteLine($"Yerleşik delege sonucu: {result}");
    }
    
    public static void Main()
    {
        // Özel delege kullanımı
        Calculator<int, int, int> addCustom = (a, b) => a + b;
        ProcessWithCustomDelegate(5, 3, addCustom);
        
        // Yerleşik delege kullanımı
        Func<int, int, int> addBuiltIn = (a, b) => a + b;
        ProcessWithBuiltInDelegate(5, 3, addBuiltIn);
        
        // İşlevsel olarak aynıdır, ancak yerleşik delegeler daha yaygın kullanılır
    }
}
```

## 5. Generic Type Constraints

Generic delege türlerinde, tip parametrelerine kısıtlamalar (constraints) ekleyebilirsiniz. Bu, delegenin belirli türlerle çalışmasını sağlar.

### Generic Kısıtlamalar ile Delegeler

```csharp
// Generic kısıtlamalı delegeler
public delegate T Factory<T>() where T : new();
public delegate bool Comparer<T>(T x, T y) where T : IComparable<T>;
public delegate void Logger<T>(T item) where T : class;

public class GenericConstraintsExample
{
    public static void Main()
    {
        // Factory kullanımı - sadece parametre almayan constructor'a sahip türler
        Factory<List<int>> listFactory = () => new List<int>();
        List<int> newList = listFactory();
        newList.Add(1);
        newList.Add(2);
        Console.WriteLine($"Liste eleman sayısı: {newList.Count}");
        
        // Comparer kullanımı - sadece IComparable<T> uygulayan türler
        Comparer<string> stringComparer = (x, y) => x.CompareTo(y) > 0;
        bool isGreater = stringComparer("b", "a");
        Console.WriteLine($"'b' > 'a': {isGreater}");
        
        // Logger kullanımı - sadece referans türleri
        Logger<string> stringLogger = item => Console.WriteLine($"LOG: {item}");
        stringLogger("Test mesajı");
    }
}
```

### Kısıtlamalı Delege Metotları

```csharp
public class ConstrainedDelegateMethodsExample
{
    // T, IComparable<T> uygulamalı
    public static void SortItems<T>(List<T> items, Func<T, T, bool> comparer) where T : IComparable<T>
    {
        for (int i = 0; i < items.Count - 1; i++)
        {
            for (int j = i + 1; j < items.Count; j++)
            {
                if (comparer(items[i], items[j]))
                {
                    T temp = items[i];
                    items[i] = items[j];
                    items[j] = temp;
                }
            }
        }
    }
    
    // T, new() ile oluşturulabilmeli
    public static List<T> CreateItems<T>(int count, Func<int, T> factory) where T : new()
    {
        List<T> items = new List<T>(count);
        
        for (int i = 0; i < count; i++)
        {
            items.Add(factory(i));
        }
        
        return items;
    }
    
    public static void Main()
    {
        // Sıralama örneği
        List<string> names = new List<string> { "Zeynep", "Ali", "Mehmet", "Ayşe" };
        SortItems(names, (x, y) => string.Compare(x, y) > 0);
        Console.WriteLine($"Sıralanmış isimler: {string.Join(", ", names)}");
        
        // Oluşturma örneği
        List<Person> people = CreateItems<Person>(3, i => new Person { Name = $"Person {i}", Age = 20 + i });
        Console.WriteLine("Oluşturulan kişiler:");
        foreach (Person person in people)
        {
            Console.WriteLine(person);
        }
    }
}
```

## 6. Delegate Performance

Delegelerin kullanımı, performans açısından bazı etkilere sahip olabilir. Bu bölümde, delegelerin performans etkilerini ve optimizasyon tekniklerini inceleyeceğiz.

### Delege Çağrı Performansı

```csharp
public class DelegatePerformanceExample
{
    // Doğrudan metot çağrısı
    public static int Add(int x, int y)
    {
        return x + y;
    }
    
    public static void Main()
    {
        const int iterations = 10000000;
        
        // Doğrudan metot çağrısı
        Stopwatch sw1 = Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            int result = Add(5, 3);
        }
        sw1.Stop();
        
        // Func delege çağrısı
        Func<int, int, int> addFunc = Add;
        Stopwatch sw2 = Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            int result = addFunc(5, 3);
        }
        sw2.Stop();
        
        // Lambda ifadesi
        Func<int, int, int> addLambda = (x, y) => x + y;
        Stopwatch sw3 = Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            int result = addLambda(5, 3);
        }
        sw3.Stop();
        
        Console.WriteLine($"Doğrudan metot çağrısı: {sw1.ElapsedMilliseconds} ms");
        Console.WriteLine($"Func delege çağrısı: {sw2.ElapsedMilliseconds} ms");
        Console.WriteLine($"Lambda ifadesi çağrısı: {sw3.ElapsedMilliseconds} ms");
    }
}
```

### Delege Oluşturma Performansı

```csharp
public class DelegateCreationPerformanceExample
{
    public static void Main()
    {
        const int iterations = 1000000;
        
        // Tek bir delege oluşturma ve tekrar kullanma
        Stopwatch sw1 = Stopwatch.StartNew();
        Func<int, int, int> singleDelegate = (x, y) => x + y;
        
        for (int i = 0; i < iterations; i++)
        {
            int result = singleDelegate(i, i + 1);
        }
        sw1.Stop();
        
        // Her iterasyonda yeni delege oluşturma
        Stopwatch sw2 = Stopwatch.StartNew();
        for (int i = 0; i < iterations; i++)
        {
            Func<int, int, int> newDelegate = (x, y) => x + y;
            int result = newDelegate(i, i + 1);
        }
        sw2.Stop();
        
        Console.WriteLine($"Tek delege: {sw1.ElapsedMilliseconds} ms");
        Console.WriteLine($"Her iterasyonda yeni delege: {sw2.ElapsedMilliseconds} ms");
    }
}
```

### Performans İyileştirme Teknikleri

```csharp
public class DelegateOptimizationExample
{
    // Büyük veri işleme örneği
    public static void ProcessLargeData(int[] data, Action<int> processor)
    {
        foreach (int item in data)
        {
            processor(item);
        }
    }
    
    // Paralel işleme ile optimizasyon
    public static void ProcessLargeDataParallel(int[] data, Action<int> processor)
    {
        Parallel.ForEach(data, item =>
        {
            processor(item);
        });
    }
    
    public static void Main()
    {
        const int dataSize = 10000000;
        int[] data = new int[dataSize];
        for (int i = 0; i < dataSize; i++)
        {
            data[i] = i;
        }
        
        // Basit işleme
        Action<int> simpleProcessor = x => { int result = x * x; };
        
        // Seri işleme
        Stopwatch sw1 = Stopwatch.StartNew();
        ProcessLargeData(data, simpleProcessor);
        sw1.Stop();
        
        // Paralel işleme
        Stopwatch sw2 = Stopwatch.StartNew();
        ProcessLargeDataParallel(data, simpleProcessor);
        sw2.Stop();
        
        Console.WriteLine($"Seri işleme: {sw1.ElapsedMilliseconds} ms");
        Console.WriteLine($"Paralel işleme: {sw2.ElapsedMilliseconds} ms");
    }
}
```

Action ve Func delegeleri, C#'ta yaygın olarak kullanılan ve kod yazarken delege tanımlamayı kolaylaştıran önceden tanımlanmış delege türleridir. Bu delegeler, kodunuzu daha okunabilir ve bakımı daha kolay hale getirebilir. Doğru delege türünü seçmek ve performans etkilerini göz önünde bulundurmak, etkili C# kodu yazmanın önemli bir parçasıdır. 