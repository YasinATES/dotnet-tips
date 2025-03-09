# Anonymous Methods

Bu bölümde, C#'ın anonim metotlarını (anonymous methods) inceleyeceğiz. Anonim metotlar, adı olmayan ve bir delege örneğine atanabilen metotlardır. C# 2.0 ile tanıtılan bu özellik, lambda ifadelerinin öncüsü olarak kabul edilir ve hala bazı senaryolarda kullanılmaktadır.

## 1. Anonymous Method Syntax

Anonim metotlar, `delegate` anahtar kelimesi kullanılarak tanımlanır. Sözdizimi `delegate(parametreler) { ifadeler }` şeklindedir.

### Temel Anonim Metot Sözdizimi

```csharp
public class AnonymousMethodExample
{
    public static void Main()
    {
        // Parametresiz anonim metot
        Action sayHello = delegate()
        {
            Console.WriteLine("Merhaba Dünya!");
        };
        
        // Parantezler isteğe bağlıdır
        Action sayGoodbye = delegate
        {
            Console.WriteLine("Güle güle!");
        };
        
        // Parametreli anonim metot
        Func<int, int, int> add = delegate(int x, int y)
        {
            return x + y;
        };
        
        // Metotları çağırma
        sayHello(); // Merhaba Dünya!
        sayGoodbye(); // Güle güle!
        Console.WriteLine($"3 + 5 = {add(3, 5)}"); // 3 + 5 = 8
    }
}
```

### Anonim Metotları Delege Olarak Kullanma

```csharp
public class DelegateAnonymousMethodExample
{
    // Özel delege türü
    public delegate bool Predicate<T>(T item);
    
    public static List<T> Filter<T>(List<T> items, Predicate<T> match)
    {
        List<T> result = new List<T>();
        
        foreach (T item in items)
        {
            if (match(item))
            {
                result.Add(item);
            }
        }
        
        return result;
    }
    
    public static void Main()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        
        // Anonim metot ile filtreleme
        List<int> evenNumbers = Filter(numbers, delegate(int n)
        {
            return n % 2 == 0;
        });
        
        Console.WriteLine($"Çift sayılar: {string.Join(", ", evenNumbers)}");
        
        // Daha karmaşık bir anonim metot
        List<int> specialNumbers = Filter(numbers, delegate(int n)
        {
            if (n < 5)
                return false;
                
            if (n % 3 == 0)
                return true;
                
            return n % 2 == 0;
        });
        
        Console.WriteLine($"Özel sayılar: {string.Join(", ", specialNumbers)}");
    }
}
```

### Olay İşleyicisi Olarak Anonim Metotlar

```csharp
public class Button
{
    public event EventHandler Click;
    
    public void PerformClick()
    {
        Console.WriteLine("Düğmeye tıklandı");
        Click?.Invoke(this, EventArgs.Empty);
    }
}

public class EventAnonymousMethodExample
{
    public static void Main()
    {
        Button button = new Button();
        
        // Anonim metot ile olay işleyicisi
        button.Click += delegate(object sender, EventArgs e)
        {
            Console.WriteLine("Düğme tıklama olayı işlendi");
            Console.WriteLine($"Gönderen: {sender.GetType().Name}");
        };
        
        // Olayı tetikleme
        button.PerformClick();
    }
}
```

## 2. Delegate vs Anonymous Methods

Anonim metotlar, isimli metotlara göre bazı avantajlar ve dezavantajlar sunar. Bu bölümde, anonim metotlar ile isimli metotlar arasındaki farkları inceleyeceğiz.

### İsimli Metot vs Anonim Metot

```csharp
public class NamedVsAnonymousExample
{
    // İsimli metot
    private static bool IsEven(int number)
    {
        return number % 2 == 0;
    }
    
    public static void Main()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        
        // İsimli metot ile filtreleme
        List<int> evenNumbers1 = numbers.FindAll(IsEven);
        
        // Anonim metot ile filtreleme
        List<int> evenNumbers2 = numbers.FindAll(delegate(int number)
        {
            return number % 2 == 0;
        });
        
        // Lambda ifadesi ile filtreleme (karşılaştırma için)
        List<int> evenNumbers3 = numbers.FindAll(number => number % 2 == 0);
        
        Console.WriteLine($"İsimli metot: {string.Join(", ", evenNumbers1)}");
        Console.WriteLine($"Anonim metot: {string.Join(", ", evenNumbers2)}");
        Console.WriteLine($"Lambda ifadesi: {string.Join(", ", evenNumbers3)}");
    }
}
```

### Avantajlar ve Dezavantajlar

```csharp
public class AdvantagesDisadvantagesExample
{
    private static int _counter = 0;
    
    // Avantaj: Yerel değişkenlere erişim
    public static void LocalVariableAccess()
    {
        int localCounter = 0;
        
        // Anonim metot yerel değişkene erişebilir
        Action incrementCounter = delegate
        {
            localCounter++; // Yerel değişkene erişim
            Console.WriteLine($"Yerel sayaç: {localCounter}");
        };
        
        incrementCounter(); // Yerel sayaç: 1
        incrementCounter(); // Yerel sayaç: 2
    }
    
    // Dezavantaj: Okunabilirlik ve tekrar kullanılabilirlik
    public static void ReadabilityExample()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
        
        // Karmaşık anonim metot - okunması zor
        var result = numbers.FindAll(delegate(int n)
        {
            if (n < 3)
                return n % 2 == 0;
            else if (n < 5)
                return n % 3 == 0;
            else
                return n % 2 != 0;
        });
        
        Console.WriteLine($"Sonuç: {string.Join(", ", result)}");
        
        // Aynı mantık isimli metotla daha okunabilir olabilir
        var result2 = numbers.FindAll(ComplexFilter);
        Console.WriteLine($"İsimli metot sonucu: {string.Join(", ", result2)}");
    }
    
    private static bool ComplexFilter(int n)
    {
        if (n < 3)
            return n % 2 == 0;
        else if (n < 5)
            return n % 3 == 0;
        else
            return n % 2 != 0;
    }
    
    public static void Main()
    {
        LocalVariableAccess();
        ReadabilityExample();
    }
}
```

## 3. Variable Capture

Anonim metotlar, tanımlandıkları kapsamdaki değişkenlere erişebilir ve bu değişkenleri "yakalayabilir" (capture). Bu özellik, closure (kapanış) olarak bilinir.

### Değişken Yakalama Örneği

```csharp
public class VariableCaptureExample
{
    public static void Main()
    {
        // Yerel değişken
        int counter = 0;
        
        // Değişkeni yakalayan anonim metot
        Action increment = delegate
        {
            counter++; // Dış değişkeni yakalama
            Console.WriteLine($"Sayaç: {counter}");
        };
        
        increment(); // Sayaç: 1
        increment(); // Sayaç: 2
        
        Console.WriteLine($"Son sayaç değeri: {counter}"); // Son sayaç değeri: 2
        
        // Yakalanan değişkeni değiştirme
        counter = 10;
        increment(); // Sayaç: 11
    }
}
```

### Değişken Yakalama ile Fonksiyon Fabrikası

```csharp
public class FunctionFactoryExample
{
    // Belirli bir faktörle çarpan fonksiyon üreten fabrika metodu
    public static Func<int, int> CreateMultiplier(int factor)
    {
        return delegate(int x)
        {
            return x * factor; // factor değişkenini yakalama
        };
    }
    
    public static void Main()
    {
        // Farklı çarpanlar için fonksiyonlar oluşturma
        Func<int, int> double_ = CreateMultiplier(2);
        Func<int, int> triple = CreateMultiplier(3);
        
        Console.WriteLine($"5'in 2 katı: {double_(5)}"); // 10
        Console.WriteLine($"5'in 3 katı: {triple(5)}"); // 15
    }
}
```

### Değişken Yakalama Tuzakları

```csharp
public class VariableCapturePitfallsExample
{
    public static void Main()
    {
        // Döngü değişkeni yakalama sorunu
        List<Action> actions = new List<Action>();
        
        // Sorunlu kod - tüm eylemler son i değerini kullanır
        for (int i = 0; i < 5; i++)
        {
            actions.Add(delegate
            {
                Console.WriteLine($"Sorunlu: {i}");
            });
        }
        
        // Tüm eylemler i=5 değerini yazdırır (veya hata verir)
        foreach (Action action in actions)
        {
            action(); // Tümü "Sorunlu: 5" yazdırır (veya hata verir)
        }
        
        Console.WriteLine("-------------------");
        
        // Çözüm - her iterasyonda yerel değişken kullanma
        actions.Clear();
        for (int i = 0; i < 5; i++)
        {
            int capturedI = i; // Yerel değişken
            actions.Add(delegate
            {
                Console.WriteLine($"Düzeltilmiş: {capturedI}");
            });
        }
        
        // Her eylem kendi capturedI değerini yazdırır
        foreach (Action action in actions)
        {
            action(); // "Düzeltilmiş: 0" ... "Düzeltilmiş: 4"
        }
    }
}
```

## 4. Method Group Conversion

Metot grubu dönüşümü (method group conversion), bir metot adının doğrudan bir delege örneğine dönüştürülmesidir. Bu, anonim metotlara bir alternatiftir.

### Metot Grubu Dönüşümü Örneği

```csharp
public class MethodGroupConversionExample
{
    // İsimli metotlar
    private static bool IsEven(int number)
    {
        return number % 2 == 0;
    }
    
    private static bool IsPositive(int number)
    {
        return number > 0;
    }
    
    private static bool IsMultipleOfThree(int number)
    {
        return number % 3 == 0;
    }
    
    public static void Main()
    {
        List<int> numbers = new List<int> { -3, -2, -1, 0, 1, 2, 3, 4, 5, 6 };
        
        // Metot grubu dönüşümü
        Predicate<int> evenPredicate = IsEven; // IsEven metodu bir Predicate<int> delegesine dönüştürülür
        Predicate<int> positivePredicate = IsPositive;
        Predicate<int> multipleOfThreePredicate = IsMultipleOfThree;
        
        // Delegeleri kullanma
        List<int> evenNumbers = numbers.FindAll(evenPredicate);
        List<int> positiveNumbers = numbers.FindAll(positivePredicate);
        List<int> multiplesOfThree = numbers.FindAll(multipleOfThreePredicate);
        
        Console.WriteLine($"Çift sayılar: {string.Join(", ", evenNumbers)}");
        Console.WriteLine($"Pozitif sayılar: {string.Join(", ", positiveNumbers)}");
        Console.WriteLine($"3'ün katları: {string.Join(", ", multiplesOfThree)}");
        
        // Doğrudan metot adı kullanma (daha kısa)
        List<int> evenNumbers2 = numbers.FindAll(IsEven);
        Console.WriteLine($"Çift sayılar (doğrudan): {string.Join(", ", evenNumbers2)}");
    }
}
```

### Metot Grubu Dönüşümü vs Anonim Metot

```csharp
public class MethodGroupVsAnonymousExample
{
    // Olay işleyicisi metodu
    private static void Button_Click(object sender, EventArgs e)
    {
        Console.WriteLine("Düğme tıklandı (isimli metot)");
    }
    
    public static void Main()
    {
        Button button1 = new Button();
        Button button2 = new Button();
        Button button3 = new Button();
        
        // 1. Metot grubu dönüşümü
        button1.Click += Button_Click;
        
        // 2. Anonim metot
        button2.Click += delegate(object sender, EventArgs e)
        {
            Console.WriteLine("Düğme tıklandı (anonim metot)");
        };
        
        // 3. Lambda ifadesi (karşılaştırma için)
        button3.Click += (sender, e) => Console.WriteLine("Düğme tıklandı (lambda)");
        
        // Düğmeleri tıklama
        button1.PerformClick();
        button2.PerformClick();
        button3.PerformClick();
    }
}
```

## 5. Async Anonymous Methods

C# 5.0 ve sonrasında, anonim metotlar da asenkron olabilir. Asenkron anonim metotlar, `async` ve `await` anahtar kelimeleri kullanılarak tanımlanır.

### Asenkron Anonim Metot Örneği

```csharp
public class AsyncAnonymousMethodExample
{
    public static void Main()
    {
        Console.WriteLine("Asenkron işlem başlıyor...");
        
        // Asenkron anonim metot
        Func<Task> asyncOperation = async delegate
        {
            Console.WriteLine("Asenkron işlem başladı");
            
            // Asenkron bekleme
            await Task.Delay(2000);
            
            Console.WriteLine("Asenkron işlem tamamlandı");
        };
        
        // Asenkron metodu çalıştırma
        Task task = asyncOperation();
        
        Console.WriteLine("Ana iş parçacığı devam ediyor");
        
        // İşlemin tamamlanmasını bekleme
        task.Wait();
        
        Console.WriteLine("Program tamamlandı");
    }
}
```

### Asenkron Olay İşleyicisi

```csharp
public class AsyncEventHandlerExample
{
    public static async Task Main()
    {
        Button button = new Button();
        
        // Asenkron anonim metot ile olay işleyicisi
        button.Click += async delegate(object sender, EventArgs e)
        {
            Console.WriteLine("Asenkron işlem başlıyor...");
            
            // Uzun süren bir işlemi simüle etme
            await Task.Delay(2000);
            
            Console.WriteLine("Asenkron işlem tamamlandı");
        };
        
        // Olayı tetikleme
        button.PerformClick();
        
        // Ana iş parçacığının devam etmesini sağlama
        Console.WriteLine("Ana iş parçacığı devam ediyor");
        
        // Tüm asenkron işlemlerin tamamlanmasını bekleme
        await Task.Delay(3000);
    }
}
```

## 6. Best Practices

Anonim metotları kullanırken dikkat edilmesi gereken bazı en iyi uygulamalar vardır.

### Ne Zaman Anonim Metot Kullanmalı

```csharp
public class BestPracticesExample
{
    public static void Main()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
        
        // İyi kullanım: Basit, tek kullanımlık işlevler
        numbers.ForEach(delegate(int n)
        {
            Console.WriteLine($"Sayı: {n}");
        });
        
        // Daha iyi: Lambda ifadesi (daha kısa ve okunabilir)
        numbers.ForEach(n => Console.WriteLine($"Sayı: {n}"));
        
        // Kötü kullanım: Karmaşık, tekrar kullanılabilir mantık
        var filteredNumbers = numbers.FindAll(delegate(int n)
        {
            // Karmaşık filtreleme mantığı
            if (n < 3)
                return n % 2 == 0;
            else
                return n % 2 != 0 && IsPrime(n);
        });
        
        // Daha iyi: İsimli metot kullanma
        var betterFilteredNumbers = numbers.FindAll(ComplexFilter);
    }
    
    // Karmaşık mantık için isimli metot
    private static bool ComplexFilter(int n)
    {
        if (n < 3)
            return n % 2 == 0;
        else
            return n % 2 != 0 && IsPrime(n);
    }
    
    private static bool IsPrime(int number)
    {
        if (number <= 1) return false;
        if (number <= 3) return true;
        
        if (number % 2 == 0 || number % 3 == 0) return false;
        
        for (int i = 5; i * i <= number; i += 6)
        {
            if (number % i == 0 || number % (i + 2) == 0)
                return false;
        }
        
        return true;
    }
}
```

### Anonim Metot vs Lambda İfadesi

```csharp
public class AnonymousVsLambdaExample
{
    public static void Main()
    {
        // 1. Anonim metot
        Func<int, int, int> add1 = delegate(int x, int y)
        {
            return x + y;
        };
        
        // 2. Lambda ifadesi (expression lambda)
        Func<int, int, int> add2 = (x, y) => x + y;
        
        // 3. Lambda ifadesi (statement lambda)
        Func<int, int, int> add3 = (x, y) =>
        {
            return x + y;
        };
        
        Console.WriteLine($"Anonim metot: 3 + 5 = {add1(3, 5)}");
        Console.WriteLine($"Expression lambda: 3 + 5 = {add2(3, 5)}");
        Console.WriteLine($"Statement lambda: 3 + 5 = {add3(3, 5)}");
        
        // Modern C# kodunda lambda ifadeleri tercih edilir
        // Anonim metotlar genellikle eski kodlarda görülür
    }
}
```

### Performans Hususları

```csharp
public class PerformanceConsiderationsExample
{
    private static bool IsEven(int n) => n % 2 == 0;
    
    public static void Main()
    {
        List<int> numbers = Enumerable.Range(1, 1000000).ToList();
        
        // 1. İsimli metot (metot grubu dönüşümü)
        Stopwatch sw1 = Stopwatch.StartNew();
        var result1 = numbers.Where(IsEven).ToList();
        sw1.Stop();
        
        // 2. Anonim metot
        Stopwatch sw2 = Stopwatch.StartNew();
        var result2 = numbers.Where(delegate(int n) { return n % 2 == 0; }).ToList();
        sw2.Stop();
        
        // 3. Lambda ifadesi
        Stopwatch sw3 = Stopwatch.StartNew();
        var result3 = numbers.Where(n => n % 2 == 0).ToList();
        sw3.Stop();
        
        Console.WriteLine($"İsimli metot: {sw1.ElapsedMilliseconds} ms");
        Console.WriteLine($"Anonim metot: {sw2.ElapsedMilliseconds} ms");
        Console.WriteLine($"Lambda ifadesi: {sw3.ElapsedMilliseconds} ms");
        
        // Not: Modern C# derleyicileri, bu üç yaklaşım arasındaki
        // performans farklarını büyük ölçüde ortadan kaldırmıştır.
    }
}
```

Anonim metotlar, C#'ta delegeleri daha esnek bir şekilde kullanmanın bir yoludur. Lambda ifadelerinin tanıtılmasıyla birlikte, anonim metotların kullanımı azalmıştır, ancak hala bazı senaryolarda ve eski kodlarda karşılaşabilirsiniz. Modern C# kodunda, genellikle daha kısa ve okunabilir olan lambda ifadeleri tercih edilir. 