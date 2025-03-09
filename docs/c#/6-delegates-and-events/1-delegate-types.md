# Delegate Types

Bu bölümde, C#'ın güçlü özelliklerinden biri olan delegeleri (delegates) inceleyeceğiz. Delegeler, metot referanslarını tutan ve bu metotları çağırmak için kullanılan türlerdir. Delegeler, olay tabanlı programlama, geri çağırma (callback) fonksiyonları ve fonksiyonel programlama gibi senaryolarda yaygın olarak kullanılır.

## 1. Single Cast Delegates

Single cast delegeler, tek bir metot referansını tutan delegelerdir. Bir delege tanımlandığında, belirli bir imzaya (parametre türleri ve dönüş değeri) sahip metotları işaret edebilir.

### Delege Tanımlama ve Kullanma

```csharp
// Delege tanımlama
public delegate int MathOperation(int x, int y);

// Delege kullanımı
public class Calculator
{
    // Delege imzasına uygun metotlar
    public static int Add(int x, int y) => x + y;
    public static int Subtract(int x, int y) => x - y;
    public static int Multiply(int x, int y) => x * y;
    public static int Divide(int x, int y) => y != 0 ? x / y : 0;
    
    public static void Main()
    {
        // Delege örneği oluşturma
        MathOperation operation = Add;
        
        // Delege üzerinden metot çağırma
        int result = operation(10, 5); // 15
        Console.WriteLine($"Sonuç: {result}");
        
        // Delege referansını değiştirme
        operation = Multiply;
        result = operation(10, 5); // 50
        Console.WriteLine($"Sonuç: {result}");
        
        // Metot grubu dönüşümü (method group conversion)
        MathOperation divideOperation = Divide;
        result = divideOperation(10, 5); // 2
        Console.WriteLine($"Sonuç: {result}");
        
        // Lambda ifadesi ile delege oluşturma
        MathOperation powerOperation = (x, y) => (int)Math.Pow(x, y);
        result = powerOperation(2, 3); // 8
        Console.WriteLine($"Sonuç: {result}");
    }
}
```

### Delege Parametreleri

Delegeler, metot parametresi olarak kullanılabilir, bu da stratejileri veya davranışları dinamik olarak değiştirmeyi mümkün kılar:

```csharp
public class SortingExample
{
    public delegate int CompareDelegate(int x, int y);
    
    public static void Sort(int[] array, CompareDelegate compareMethod)
    {
        for (int i = 0; i < array.Length - 1; i++)
        {
            for (int j = i + 1; j < array.Length; j++)
            {
                if (compareMethod(array[i], array[j]) > 0)
                {
                    // Swap
                    int temp = array[i];
                    array[i] = array[j];
                    array[j] = temp;
                }
            }
        }
    }
    
    public static int Ascending(int x, int y) => x.CompareTo(y);
    public static int Descending(int x, int y) => y.CompareTo(x);
    
    public static void Main()
    {
        int[] numbers = { 5, 2, 8, 1, 9 };
        
        // Artan sıralama
        Sort(numbers, Ascending);
        Console.WriteLine(string.Join(", ", numbers)); // 1, 2, 5, 8, 9
        
        // Azalan sıralama
        Sort(numbers, Descending);
        Console.WriteLine(string.Join(", ", numbers)); // 9, 8, 5, 2, 1
        
        // Lambda ile özel sıralama
        Sort(numbers, (x, y) => (x % 2).CompareTo(y % 2)); // Önce çift sayılar
        Console.WriteLine(string.Join(", ", numbers));
    }
}
```

## 2. Multicast Delegates

Multicast delegeler, birden fazla metot referansını tutan ve bu metotları sırayla çağıran delegelerdir. C#'ta tüm delegeler multicast olabilir, çünkü `System.MulticastDelegate` sınıfından türetilirler.

### Multicast Delege Oluşturma ve Kullanma

```csharp
// Void dönüş değerine sahip delege
public delegate void NotificationDelegate(string message);

public class NotificationService
{
    public static void SendEmail(string message)
    {
        Console.WriteLine($"E-posta gönderiliyor: {message}");
    }
    
    public static void SendSMS(string message)
    {
        Console.WriteLine($"SMS gönderiliyor: {message}");
    }
    
    public static void LogToConsole(string message)
    {
        Console.WriteLine($"Konsola yazılıyor: {message}");
    }
    
    public static void Main()
    {
        // Multicast delege oluşturma
        NotificationDelegate notifier = SendEmail;
        
        // Delege zincirine metot ekleme
        notifier += SendSMS;
        notifier += LogToConsole;
        
        // Tüm metotları çağırma
        notifier("Önemli bildirim!"); 
        // Çıktı:
        // E-posta gönderiliyor: Önemli bildirim!
        // SMS gönderiliyor: Önemli bildirim!
        // Konsola yazılıyor: Önemli bildirim!
        
        Console.WriteLine("-------------------");
        
        // Zincirden metot çıkarma
        notifier -= SendSMS;
        
        // Güncellenmiş zinciri çağırma
        notifier("Başka bir bildirim");
        // Çıktı:
        // E-posta gönderiliyor: Başka bir bildirim
        // Konsola yazılıyor: Başka bir bildirim
    }
}
```

### Multicast Delege Operasyonları

```csharp
public delegate void ProcessDelegate();

public class MulticastExample
{
    public static void Process1() => Console.WriteLine("Process1 çalıştı");
    public static void Process2() => Console.WriteLine("Process2 çalıştı");
    public static void Process3() => Console.WriteLine("Process3 çalıştı");
    
    public static void Main()
    {
        // Delegeleri oluşturma
        ProcessDelegate d1 = Process1;
        ProcessDelegate d2 = Process2;
        ProcessDelegate d3 = Process3;
        
        // Delegeleri birleştirme
        ProcessDelegate allProcesses = d1 + d2 + d3;
        
        // Tüm metotları çağırma
        allProcesses();
        
        Console.WriteLine("-------------------");
        
        // Delege zincirini ayırma
        ProcessDelegate firstTwoProcesses = allProcesses - d3;
        firstTwoProcesses();
        
        Console.WriteLine("-------------------");
        
        // Delege listesini temizleme
        allProcesses = null;
        
        // Null kontrolü
        if (allProcesses != null)
        {
            allProcesses();
        }
        
        // Daha güvenli çağrı (C# 6.0+)
        allProcesses?.Invoke();
    }
}
```

## 3. Generic Delegates

Generic delegeler, farklı veri türleriyle çalışabilen esnek delege tanımları oluşturmanıza olanak tanır. C#, yaygın kullanım senaryoları için önceden tanımlanmış generic delegeler sağlar: `Action<T>`, `Func<T, TResult>` ve `Predicate<T>`.

### Action<T> Delegesi

`Action<T>` delegesi, bir veya daha fazla parametre alan ve değer döndürmeyen (void) metotları temsil eder:

```csharp
public class ActionExample
{
    public static void ProcessString(string input)
    {
        Console.WriteLine($"String işleniyor: {input}");
    }
    
    public static void ProcessInt(int number)
    {
        Console.WriteLine($"Sayı işleniyor: {number}");
    }
    
    public static void ProcessData<T>(T data, Action<T> processor)
    {
        Console.WriteLine($"Veri tipi: {typeof(T).Name}");
        processor(data);
    }
    
    public static void Main()
    {
        // Action<string> kullanımı
        Action<string> stringProcessor = ProcessString;
        stringProcessor("Merhaba Dünya");
        
        // Action<int> kullanımı
        Action<int> intProcessor = ProcessInt;
        intProcessor(42);
        
        // Generic metot ile kullanım
        ProcessData("Test", ProcessString);
        ProcessData(100, ProcessInt);
        
        // Lambda ile kullanım
        Action<double> doubleProcessor = (d) => Console.WriteLine($"Double: {d}");
        doubleProcessor(3.14);
        
        // Çoklu parametre
        Action<string, int> printRepeated = (s, count) =>
        {
            for (int i = 0; i < count; i++)
            {
                Console.WriteLine(s);
            }
        };
        
        printRepeated("Tekrar", 3);
    }
}
```

### Func<T, TResult> Delegesi

`Func<T, TResult>` delegesi, bir veya daha fazla parametre alan ve değer döndüren metotları temsil eder:

```csharp
public class FuncExample
{
    public static int Square(int x) => x * x;
    
    public static string FormatName(string firstName, string lastName)
    {
        return $"{lastName}, {firstName}";
    }
    
    public static T Transform<T, U>(U input, Func<U, T> transformer)
    {
        return transformer(input);
    }
    
    public static void Main()
    {
        // Func<int, int> kullanımı
        Func<int, int> squareFunc = Square;
        int result = squareFunc(5); // 25
        Console.WriteLine($"Kare: {result}");
        
        // Func<string, string, string> kullanımı
        Func<string, string, string> nameFormatter = FormatName;
        string formattedName = nameFormatter("Ali", "Yılmaz"); // "Yılmaz, Ali"
        Console.WriteLine($"Formatlanmış isim: {formattedName}");
        
        // Generic metot ile kullanım
        int transformedInt = Transform(5, Square);
        Console.WriteLine($"Dönüştürülmüş: {transformedInt}");
        
        // Lambda ile kullanım
        Func<double, double> circleArea = r => Math.PI * r * r;
        double area = circleArea(2.5);
        Console.WriteLine($"Daire alanı: {area}");
        
        // Çoklu parametre ve farklı dönüş tipi
        Func<int, int, bool> isGreater = (a, b) => a > b;
        bool comparison = isGreater(10, 5); // true
        Console.WriteLine($"10 > 5: {comparison}");
    }
}
```

### Predicate<T> Delegesi

`Predicate<T>` delegesi, bir parametre alan ve boolean değer döndüren metotları temsil eder. Genellikle filtreleme işlemleri için kullanılır:

```csharp
public class PredicateExample
{
    public static bool IsEven(int number) => number % 2 == 0;
    
    public static bool IsUpperCase(string text) => text.Equals(text.ToUpper());
    
    public static List<T> Filter<T>(List<T> items, Predicate<T> predicate)
    {
        List<T> result = new List<T>();
        foreach (T item in items)
        {
            if (predicate(item))
            {
                result.Add(item);
            }
        }
        return result;
    }
    
    public static void Main()
    {
        // Predicate<int> kullanımı
        Predicate<int> isEvenPredicate = IsEven;
        bool isEvenResult = isEvenPredicate(4); // true
        Console.WriteLine($"4 çift mi: {isEvenResult}");
        
        // Predicate<string> kullanımı
        Predicate<string> isUpperCasePredicate = IsUpperCase;
        bool isUpperResult = isUpperCasePredicate("HELLO"); // true
        Console.WriteLine($"'HELLO' büyük harfli mi: {isUpperResult}");
        
        // Özel filtreleme metodu ile kullanım
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        List<int> evenNumbers = Filter(numbers, IsEven);
        Console.WriteLine($"Çift sayılar: {string.Join(", ", evenNumbers)}");
        
        // Lambda ile kullanım
        List<string> words = new List<string> { "apple", "BANANA", "Cherry", "DATE", "elderberry" };
        List<string> upperCaseWords = Filter(words, s => s.Equals(s.ToUpper()));
        Console.WriteLine($"Büyük harfli kelimeler: {string.Join(", ", upperCaseWords)}");
        
        // .NET'in yerleşik metotları ile kullanım
        List<int> greaterThanFive = numbers.FindAll(n => n > 5);
        Console.WriteLine($"5'ten büyük sayılar: {string.Join(", ", greaterThanFive)}");
    }
}
```

## 4. Delegate Covariance/Contravariance

Delege kovaryansı (covariance) ve kontravaryansı (contravariance), delegelerin daha esnek bir şekilde kullanılmasını sağlayan özelliklerdir. Kovaryans, dönüş değeri için daha türetilmiş bir tür kullanabilmeyi; kontravaryans ise parametre için daha temel bir tür kullanabilmeyi ifade eder.

### Kovaryans (Covariance)

```csharp
public class Animal { }
public class Dog : Animal { }
public class Cat : Animal { }

public class CovarianceExample
{
    // Delege tanımları
    public delegate Animal AnimalCreator();
    public delegate Dog DogCreator();
    
    // Metotlar
    public static Dog CreateDog() => new Dog();
    public static Cat CreateCat() => new Cat();
    
    public static void Main()
    {
        // Kovaryans örneği - DogCreator, AnimalCreator'a atanabilir
        DogCreator dogCreator = CreateDog;
        AnimalCreator animalCreator = dogCreator; // Kovaryans
        
        Animal animal = animalCreator(); // Aslında bir Dog döndürür
        Console.WriteLine($"Oluşturulan hayvan: {animal.GetType().Name}");
        
        // Func ile kovaryans
        Func<Dog> createDogFunc = () => new Dog();
        Func<Animal> createAnimalFunc = createDogFunc; // Kovaryans
        
        Animal animal2 = createAnimalFunc();
        Console.WriteLine($"Func ile oluşturulan hayvan: {animal2.GetType().Name}");
    }
}
```

### Kontravaryans (Contravariance)

```csharp
public class ContravarianceExample
{
    // Delege tanımları
    public delegate void AnimalProcessor(Animal animal);
    public delegate void DogProcessor(Dog dog);
    
    // Metotlar
    public static void ProcessAnimal(Animal animal)
    {
        Console.WriteLine($"Hayvan işleniyor: {animal.GetType().Name}");
    }
    
    public static void ProcessDog(Dog dog)
    {
        Console.WriteLine($"Köpek işleniyor: {dog.GetType().Name}");
    }
    
    public static void Main()
    {
        // Kontravaryans örneği - AnimalProcessor, DogProcessor'a atanabilir
        AnimalProcessor animalProcessor = ProcessAnimal;
        DogProcessor dogProcessor = animalProcessor; // Kontravaryans
        
        Dog dog = new Dog();
        dogProcessor(dog); // Aslında ProcessAnimal çağrılır
        
        // Action ile kontravaryans
        Action<Animal> processAnimalAction = a => Console.WriteLine($"Action: {a.GetType().Name}");
        Action<Dog> processDogAction = processAnimalAction; // Kontravaryans
        
        processDogAction(new Dog());
    }
}
```

### Kovaryans ve Kontravaryans Birlikte Kullanımı

```csharp
public class Converter<TInput, TOutput>
{
    private Func<TInput, TOutput> _converterFunc;
    
    public Converter(Func<TInput, TOutput> converterFunc)
    {
        _converterFunc = converterFunc;
    }
    
    public TOutput Convert(TInput input)
    {
        return _converterFunc(input);
    }
}

public class VarianceExample
{
    public static Animal ConvertToAnimal(object obj)
    {
        Console.WriteLine("Object -> Animal dönüşümü");
        return new Animal();
    }
    
    public static Dog ConvertToDog(string text)
    {
        Console.WriteLine($"String -> Dog dönüşümü: {text}");
        return new Dog();
    }
    
    public static void Main()
    {
        // Kontravaryans (parametre) ve Kovaryans (dönüş değeri)
        Func<object, Animal> animalConverter = ConvertToAnimal;
        Func<string, Dog> dogConverter = ConvertToDog;
        
        // Kontravaryans: string daha spesifik (object'ten türetilmiş)
        // Kovaryans: Dog daha spesifik (Animal'dan türetilmiş)
        Func<string, Animal> converter = dogConverter; // Her iki varyans da çalışıyor
        
        Animal result = converter("Köpek adı");
        Console.WriteLine($"Sonuç: {result.GetType().Name}");
        
        // Converter sınıfı ile kullanım
        Converter<object, Animal> animalConverterObj = new Converter<object, Animal>(ConvertToAnimal);
        Converter<string, Dog> dogConverterObj = new Converter<string, Dog>(ConvertToDog);
        
        // Varyans ile dönüşüm
        Animal animal1 = animalConverterObj.Convert(new object());
        Dog dog1 = dogConverterObj.Convert("Rex");
    }
}
```

## 5. Delegate Chaining

Delege zincirleme (chaining), birden fazla metodu bir delege içinde birleştirme ve sırayla çağırma işlemidir. Bu, özellikle olay işleme (event handling) senaryolarında yaygın olarak kullanılır.

### Temel Delege Zincirleme

```csharp
public delegate void ProcessHandler(string data);

public class ChainExample
{
    public static void LogToConsole(string data)
    {
        Console.WriteLine($"Konsol: {data}");
    }
    
    public static void LogToFile(string data)
    {
        Console.WriteLine($"Dosya: {data} kaydedildi");
    }
    
    public static void SendNotification(string data)
    {
        Console.WriteLine($"Bildirim: {data} gönderildi");
    }
    
    public static void Main()
    {
        // Delege zinciri oluşturma
        ProcessHandler chain = LogToConsole;
        chain += LogToFile;
        chain += SendNotification;
        
        // Zinciri çağırma
        chain("Önemli veri");
        
        Console.WriteLine("-------------------");
        
        // Zincirden metot çıkarma
        chain -= LogToFile;
        chain("Güncellenmiş veri");
        
        Console.WriteLine("-------------------");
        
        // Zinciri temizleme ve yeniden oluşturma
        chain = null;
        chain += LogToConsole;
        chain += SendNotification;
        
        chain?.Invoke("Yeni zincir");
    }
}
```

### Delege Zincirini Yönetme

```csharp
public class DelegateManager
{
    private ProcessHandler _handlers;
    
    public void AddHandler(ProcessHandler handler)
    {
        _handlers += handler;
    }
    
    public void RemoveHandler(ProcessHandler handler)
    {
        _handlers -= handler;
    }
    
    public void ClearHandlers()
    {
        _handlers = null;
    }
    
    public void Process(string data)
    {
        _handlers?.Invoke(data);
    }
    
    // Delege listesini alma
    public Delegate[] GetInvocationList()
    {
        return _handlers?.GetInvocationList() ?? new Delegate[0];
    }
}

public class ChainManagementExample
{
    public static void Main()
    {
        DelegateManager manager = new DelegateManager();
        
        // İşleyicileri ekleme
        manager.AddHandler(ChainExample.LogToConsole);
        manager.AddHandler(ChainExample.LogToFile);
        manager.AddHandler(ChainExample.SendNotification);
        
        // İşleme
        manager.Process("Test verisi");
        
        Console.WriteLine("-------------------");
        
        // İşleyici listesini görüntüleme
        Delegate[] handlers = manager.GetInvocationList();
        Console.WriteLine($"İşleyici sayısı: {handlers.Length}");
        
        foreach (Delegate handler in handlers)
        {
            Console.WriteLine($"İşleyici: {handler.Method.Name}");
        }
        
        Console.WriteLine("-------------------");
        
        // İşleyici çıkarma
        manager.RemoveHandler(ChainExample.LogToFile);
        manager.Process("Güncellenmiş veri");
        
        Console.WriteLine("-------------------");
        
        // Tüm işleyicileri temizleme
        manager.ClearHandlers();
        manager.Process("Bu çağrı hiçbir şey yapmayacak");
    }
}
```

## 6. Return Value Handling

Multicast delegelerde, birden fazla metot çağrıldığında, sadece son çağrılan metodun dönüş değeri alınır. Bu durumu yönetmek için çeşitli stratejiler kullanılabilir.

### Multicast Delege Dönüş Değeri Sorunu

```csharp
public delegate int NumberOperation(int number);

public class ReturnValueExample
{
    public static int Double(int number)
    {
        int result = number * 2;
        Console.WriteLine($"Double: {number} -> {result}");
        return result;
    }
    
    public static int Square(int number)
    {
        int result = number * number;
        Console.WriteLine($"Square: {number} -> {result}");
        return result;
    }
    
    public static int AddFive(int number)
    {
        int result = number + 5;
        Console.WriteLine($"AddFive: {number} -> {result}");
        return result;
    }
    
    public static void Main()
    {
        // Multicast delege oluşturma
        NumberOperation operations = Double;
        operations += Square;
        operations += AddFive;
        
        // Çağrı - sadece son metodun dönüş değeri alınır
        int result = operations(5);
        Console.WriteLine($"Sonuç: {result}"); // Sadece AddFive'ın sonucu: 10
        
        Console.WriteLine("-------------------");
        
        // Tüm dönüş değerlerini almak için
        Delegate[] delegates = operations.GetInvocationList();
        List<int> results = new List<int>();
        
        foreach (NumberOperation operation in delegates)
        {
            results.Add(operation(5));
        }
        
        Console.WriteLine($"Tüm sonuçlar: {string.Join(", ", results)}");
    }
}
```

### Özel Dönüş Değeri İşleme Stratejileri

```csharp
public class ReturnValueStrategies
{
    // Tüm dönüş değerlerini toplama
    public static int SumResults(NumberOperation operations, int input)
    {
        int sum = 0;
        foreach (NumberOperation operation in operations.GetInvocationList())
        {
            sum += operation(input);
        }
        return sum;
    }
    
    // Dönüş değerlerinin ortalamasını alma
    public static double AverageResults(NumberOperation operations, int input)
    {
        Delegate[] delegates = operations.GetInvocationList();
        int sum = 0;
        
        foreach (NumberOperation operation in delegates)
        {
            sum += operation(input);
        }
        
        return (double)sum / delegates.Length;
    }
    
    // Minimum dönüş değerini bulma
    public static int MinResult(NumberOperation operations, int input)
    {
        Delegate[] delegates = operations.GetInvocationList();
        if (delegates.Length == 0)
            throw new InvalidOperationException("Delege zinciri boş");
            
        int min = ((NumberOperation)delegates[0])(input);
        
        for (int i = 1; i < delegates.Length; i++)
        {
            int result = ((NumberOperation)delegates[i])(input);
            if (result < min)
                min = result;
        }
        
        return min;
    }
    
    // Maksimum dönüş değerini bulma
    public static int MaxResult(NumberOperation operations, int input)
    {
        Delegate[] delegates = operations.GetInvocationList();
        if (delegates.Length == 0)
            throw new InvalidOperationException("Delege zinciri boş");
            
        int max = ((NumberOperation)delegates[0])(input);
        
        for (int i = 1; i < delegates.Length; i++)
        {
            int result = ((NumberOperation)delegates[i])(input);
            if (result > max)
                max = result;
        }
        
        return max;
    }
    
    public static void Main()
    {
        NumberOperation operations = ReturnValueExample.Double;
        operations += ReturnValueExample.Square;
        operations += ReturnValueExample.AddFive;
        
        int input = 5;
        
        // Farklı stratejileri uygulama
        int sum = SumResults(operations, input);
        double average = AverageResults(operations, input);
        int min = MinResult(operations, input);
        int max = MaxResult(operations, input);
        
        Console.WriteLine($"Toplam: {sum}");
        Console.WriteLine($"Ortalama: {average}");
        Console.WriteLine($"Minimum: {min}");
        Console.WriteLine($"Maksimum: {max}");
    }
}
```

### Sonuç Toplayıcı Sınıf

```csharp
public class ResultCollector<T>
{
    private List<T> _results = new List<T>();
    
    public void AddResult(T result)
    {
        _results.Add(result);
    }
    
    public List<T> GetResults()
    {
        return new List<T>(_results);
    }
    
    public T GetLastResult()
    {
        if (_results.Count == 0)
            return default;
            
        return _results[_results.Count - 1];
    }
    
    public void Clear()
    {
        _results.Clear();
    }
}

public class CollectorExample
{
    public delegate int NumberFunc(int n);
    
    public static void ProcessWithCollector(NumberFunc func, int input, ResultCollector<int> collector)
    {
        foreach (NumberFunc operation in func.GetInvocationList())
        {
            int result = operation(input);
            collector.AddResult(result);
        }
    }
    
    public static void Main()
    {
        NumberFunc operations = ReturnValueExample.Double;
        operations += ReturnValueExample.Square;
        operations += ReturnValueExample.AddFive;
        
        ResultCollector<int> collector = new ResultCollector<int>();
        ProcessWithCollector(operations, 5, collector);
        
        List<int> results = collector.GetResults();
        Console.WriteLine($"Tüm sonuçlar: {string.Join(", ", results)}");
        Console.WriteLine($"Son sonuç: {collector.GetLastResult()}");
    }
}
```

Delegeler, C#'ta metot referanslarını temsil eden güçlü bir özelliktir. Single cast delegeler, multicast delegeler, generic delegeler ve varyans özellikleri ile birlikte, delegeler çeşitli programlama senaryolarında esneklik ve modülerlik sağlar. Delege zincirleme ve dönüş değeri işleme stratejileri, karmaşık işlevselliği yönetmek için etkili yöntemler sunar. 