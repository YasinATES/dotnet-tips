# Lambda Expressions

Bu bölümde, C#'ın lambda ifadelerini (lambda expressions) inceleyeceğiz. Lambda ifadeleri, anonim metotların daha kısa ve daha okunabilir bir şekilde yazılmasını sağlayan bir özelliktir. Lambda ifadeleri, delegeler, LINQ sorguları ve fonksiyonel programlama senaryolarında yaygın olarak kullanılır.

## 1. Expression Lambda Syntax

Expression lambda'lar, tek bir ifade içeren ve değer döndüren lambda ifadeleridir. Sözdizimi `(parametreler) => ifade` şeklindedir.

### Temel Expression Lambda Sözdizimi

```csharp
public class ExpressionLambdaExample
{
    public static void Main()
    {
        // Parametre almayan lambda
        Func<int> getRandomNumber = () => new Random().Next(1, 100);
        Console.WriteLine($"Rastgele sayı: {getRandomNumber()}");
        
        // Tek parametreli lambda
        Func<int, int> square = x => x * x;
        Console.WriteLine($"5'in karesi: {square(5)}");
        
        // Çok parametreli lambda
        Func<int, int, int> add = (x, y) => x + y;
        Console.WriteLine($"3 + 7 = {add(3, 7)}");
        
        // Tip belirtilen parametreler
        Func<double, double, double> divide = (double x, double y) => x / y;
        Console.WriteLine($"10 / 3 = {divide(10, 3)}");
        
        // Parantezler gereklidir (birden fazla parametre veya tip belirtildiğinde)
        Func<int, int, bool> isGreater = (x, y) => x > y;
        Console.WriteLine($"5 > 3: {isGreater(5, 3)}");
    }
}
```

### Expression Lambda'ları Delege Olarak Kullanma

```csharp
public class DelegateLambdaExample
{
    // Özel delege türü
    public delegate bool FilterDelegate<T>(T item);
    
    public static List<T> Filter<T>(List<T> items, FilterDelegate<T> predicate)
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
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        
        // Lambda ifadesi ile filtreleme
        List<int> evenNumbers = Filter(numbers, n => n % 2 == 0);
        Console.WriteLine($"Çift sayılar: {string.Join(", ", evenNumbers)}");
        
        // Başka bir lambda ifadesi
        List<int> greaterThanFive = Filter(numbers, n => n > 5);
        Console.WriteLine($"5'ten büyük sayılar: {string.Join(", ", greaterThanFive)}");
        
        // Func<T, bool> ile aynı işlev
        Func<int, bool> isEven = n => n % 2 == 0;
        List<int> evenNumbersWithFunc = numbers.Where(isEven).ToList();
        Console.WriteLine($"Func ile çift sayılar: {string.Join(", ", evenNumbersWithFunc)}");
    }
}
```

## 2. Statement Lambda Syntax

Statement lambda'lar, birden fazla ifade içeren ve bir kod bloğu olarak yazılan lambda ifadeleridir. Sözdizimi `(parametreler) => { ifadeler }` şeklindedir.

### Temel Statement Lambda Sözdizimi

```csharp
public class StatementLambdaExample
{
    public static void Main()
    {
        // Tek ifadeli statement lambda
        Action<string> greet = (name) => { Console.WriteLine($"Merhaba, {name}!"); };
        greet("Ahmet");
        
        // Çok ifadeli statement lambda
        Func<int, int, int> calculateSum = (x, y) =>
        {
            int sum = x + y;
            Console.WriteLine($"{x} + {y} = {sum}");
            return sum;
        };
        
        int result = calculateSum(5, 7);
        
        // Koşullu ifadeler içeren lambda
        Func<int, string> evaluateNumber = (number) =>
        {
            if (number < 0)
                return "Negatif";
            else if (number == 0)
                return "Sıfır";
            else
                return "Pozitif";
        };
        
        Console.WriteLine($"-5: {evaluateNumber(-5)}");
        Console.WriteLine($"0: {evaluateNumber(0)}");
        Console.WriteLine($"10: {evaluateNumber(10)}");
    }
}
```

### Statement Lambda'ları Olay İşleyicisi Olarak Kullanma

```csharp
public class ButtonWithEvent
{
    public event EventHandler<int> ButtonClicked;
    
    public void Click(int clickCount)
    {
        Console.WriteLine($"Düğmeye {clickCount} kez tıklandı");
        ButtonClicked?.Invoke(this, clickCount);
    }
}

public class StatementLambdaEventExample
{
    public static void Main()
    {
        ButtonWithEvent button = new ButtonWithEvent();
        
        // Statement lambda ile olay işleyicisi
        button.ButtonClicked += (sender, clickCount) =>
        {
            if (clickCount == 1)
            {
                Console.WriteLine("Tek tıklama algılandı");
            }
            else if (clickCount == 2)
            {
                Console.WriteLine("Çift tıklama algılandı");
            }
            else
            {
                Console.WriteLine($"{clickCount} kez tıklama algılandı");
                Console.WriteLine("Bu bir çoklu tıklama!");
            }
        };
        
        // Farklı tıklama senaryoları
        button.Click(1);
        button.Click(2);
        button.Click(3);
    }
}
```

## 3. Closure ve Captured Variables

Closure (kapanış), lambda ifadelerinin tanımlandıkları kapsamdaki değişkenlere erişebilme özelliğidir. Bu değişkenlere "captured variables" (yakalanan değişkenler) denir.

### Temel Closure Örneği

```csharp
public class ClosureExample
{
    public static void Main()
    {
        // Dış değişken
        int factor = 5;
        
        // Lambda içinde dış değişkeni kullanma (closure)
        Func<int, int> multiplier = x => x * factor;
        
        Console.WriteLine($"3 * {factor} = {multiplier(3)}"); // 15
        
        // Dış değişkeni değiştirme
        factor = 10;
        
        // Lambda, güncellenmiş değeri kullanır
        Console.WriteLine($"3 * {factor} = {multiplier(3)}"); // 30
    }
}
```

### Closure ile Fonksiyon Fabrikası

```csharp
public class FunctionFactoryExample
{
    // Belirli bir faktörle çarpan fonksiyon üreten fabrika metodu
    public static Func<int, int> CreateMultiplier(int factor)
    {
        return x => x * factor;
    }
    
    public static void Main()
    {
        // Farklı çarpanlar için fonksiyonlar oluşturma
        Func<int, int> double_ = CreateMultiplier(2);
        Func<int, int> triple = CreateMultiplier(3);
        Func<int, int> quadruple = CreateMultiplier(4);
        
        Console.WriteLine($"5'in 2 katı: {double_(5)}"); // 10
        Console.WriteLine($"5'in 3 katı: {triple(5)}"); // 15
        Console.WriteLine($"5'in 4 katı: {quadruple(5)}"); // 20
    }
}
```

### Closure ile Dikkat Edilmesi Gerekenler

```csharp
public class ClosurePitfallsExample
{
    public static void Main()
    {
        // Döngü değişkeni yakalama sorunu
        List<Action> actions = new List<Action>();
        
        // Sorunlu kod - tüm eylemler son i değerini kullanır
        for (int i = 0; i < 5; i++)
        {
            actions.Add(() => Console.WriteLine($"Sorunlu: {i}"));
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
            actions.Add(() => Console.WriteLine($"Düzeltilmiş: {capturedI}"));
        }
        
        // Her eylem kendi capturedI değerini yazdırır
        foreach (Action action in actions)
        {
            action(); // "Düzeltilmiş: 0" ... "Düzeltilmiş: 4"
        }
    }
}
```

## 4. Lambda Expression Trees

Lambda ifade ağaçları (expression trees), lambda ifadelerinin çalıştırılabilir kod yerine veri yapısı olarak temsil edilmesini sağlar. Bu, dinamik sorgu oluşturma ve LINQ to SQL gibi senaryolarda kullanılır.

### Expression Tree Oluşturma

```csharp
public class ExpressionTreeExample
{
    public static void Main()
    {
        // Lambda ifadesi
        Func<int, int, int> add = (x, y) => x + y;
        
        // Aynı lambda ifadesinin expression tree olarak temsili
        Expression<Func<int, int, int>> addExpression = (x, y) => x + y;
        
        // Expression tree'yi inceleme
        BinaryExpression body = (BinaryExpression)addExpression.Body;
        Console.WriteLine($"İşlem: {body.NodeType}");
        Console.WriteLine($"Sol operand: {body.Left}");
        Console.WriteLine($"Sağ operand: {body.Right}");
        
        // Expression tree'yi derleyip çalıştırma
        Func<int, int, int> compiledAdd = addExpression.Compile();
        Console.WriteLine($"3 + 5 = {compiledAdd(3, 5)}");
    }
}
```

### Expression Tree ile Dinamik Sorgu Oluşturma

```csharp
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
}

public class DynamicQueryExample
{
    public static Expression<Func<T, bool>> BuildPredicate<T, TValue>(
        Expression<Func<T, TValue>> selector, TValue value)
    {
        // Parametre ifadesi (x => ...)
        ParameterExpression parameter = selector.Parameters[0];
        
        // Karşılaştırma ifadesi (x.Property == value)
        BinaryExpression comparison = Expression.Equal(
            selector.Body,
            Expression.Constant(value, typeof(TValue))
        );
        
        // Lambda ifadesi (x => x.Property == value)
        return Expression.Lambda<Func<T, bool>>(comparison, parameter);
    }
    
    public static void Main()
    {
        List<Product> products = new List<Product>
        {
            new Product { Id = 1, Name = "Laptop", Price = 5000, Category = "Elektronik" },
            new Product { Id = 2, Name = "Telefon", Price = 3000, Category = "Elektronik" },
            new Product { Id = 3, Name = "Masa", Price = 1000, Category = "Mobilya" },
            new Product { Id = 4, Name = "Sandalye", Price = 500, Category = "Mobilya" }
        };
        
        // Dinamik sorgu oluşturma
        Expression<Func<Product, string>> categorySelector = p => p.Category;
        Expression<Func<Product, bool>> categoryFilter = BuildPredicate(categorySelector, "Elektronik");
        
        // Sorguyu çalıştırma
        List<Product> filteredProducts = products.AsQueryable().Where(categoryFilter).ToList();
        
        foreach (Product product in filteredProducts)
        {
            Console.WriteLine($"{product.Name} - {product.Category} - {product.Price:C}");
        }
    }
}
```

## 5. Func vs Action Usage

`Func<T, TResult>` ve `Action<T>` delegeleri, lambda ifadeleriyle sıkça kullanılan önceden tanımlanmış delege türleridir.

### Func ve Action Karşılaştırması

```csharp
public class FuncVsActionExample
{
    public static void Main()
    {
        // Action - değer döndürmeyen lambda
        Action sayHello = () => Console.WriteLine("Merhaba!");
        Action<string> greet = name => Console.WriteLine($"Merhaba, {name}!");
        Action<string, int> repeatGreeting = (name, count) =>
        {
            for (int i = 0; i < count; i++)
            {
                Console.WriteLine($"Merhaba, {name}!");
            }
        };
        
        sayHello(); // Merhaba!
        greet("Ali"); // Merhaba, Ali!
        repeatGreeting("Ayşe", 3); // 3 kez Merhaba, Ayşe!
        
        // Func - değer döndüren lambda
        Func<int> getRandomNumber = () => new Random().Next(1, 100);
        Func<int, bool> isEven = number => number % 2 == 0;
        Func<int, int, int> add = (x, y) => x + y;
        
        int randomNum = getRandomNumber();
        Console.WriteLine($"Rastgele sayı: {randomNum}");
        Console.WriteLine($"{randomNum} çift mi? {isEven(randomNum)}");
        Console.WriteLine($"3 + 5 = {add(3, 5)}");
    }
}
```

### Func ve Action Kullanım Senaryoları

```csharp
public class FuncActionUsageExample
{
    // Action parametreli metot - işlem yapma
    public static void ProcessItems<T>(IEnumerable<T> items, Action<T> processor)
    {
        foreach (T item in items)
        {
            processor(item);
        }
    }
    
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
        
        // Action ile işleme
        Console.WriteLine("Sayıları yazdırma:");
        ProcessItems(numbers, n => Console.WriteLine(n));
        
        // Func ile dönüştürme
        List<string> numberTexts = TransformItems(numbers, n => $"Sayı: {n}");
        Console.WriteLine("\nDönüştürülmüş sayılar:");
        foreach (string text in numberTexts)
        {
            Console.WriteLine(text);
        }
        
        // Func ile filtreleme
        List<int> evenNumbers = FilterItems(numbers, n => n % 2 == 0);
        Console.WriteLine("\nÇift sayılar:");
        foreach (int number in evenNumbers)
        {
            Console.WriteLine(number);
        }
    }
}
```

## 6. LINQ with Lambda

LINQ (Language Integrated Query), veri sorgulama işlemlerini kolaylaştıran bir C# özelliğidir. Lambda ifadeleri, LINQ sorgularında yaygın olarak kullanılır.

### Temel LINQ Lambda Kullanımı

```csharp
public class LinqLambdaExample
{
    public static void Main()
    {
        List<int> numbers = new List<int> { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };
        
        // Where - filtreleme
        var evenNumbers = numbers.Where(n => n % 2 == 0);
        Console.WriteLine($"Çift sayılar: {string.Join(", ", evenNumbers)}");
        
        // Select - dönüştürme
        var squaredNumbers = numbers.Select(n => n * n);
        Console.WriteLine($"Kareleri: {string.Join(", ", squaredNumbers)}");
        
        // OrderBy - sıralama
        var descendingNumbers = numbers.OrderByDescending(n => n);
        Console.WriteLine($"Azalan sıralama: {string.Join(", ", descendingNumbers)}");
        
        // First, FirstOrDefault - ilk eleman
        int firstEven = numbers.First(n => n % 2 == 0); // 2
        int firstBigNumber = numbers.FirstOrDefault(n => n > 100); // 0 (default)
        
        Console.WriteLine($"İlk çift sayı: {firstEven}");
        Console.WriteLine($"100'den büyük ilk sayı: {firstBigNumber}");
        
        // Any, All - koşul kontrolü
        bool hasEvenNumbers = numbers.Any(n => n % 2 == 0); // true
        bool allPositive = numbers.All(n => n > 0); // true
        
        Console.WriteLine($"Çift sayı var mı? {hasEvenNumbers}");
        Console.WriteLine($"Tümü pozitif mi? {allPositive}");
    }
}
```

### Kompleks LINQ Sorguları

```csharp
public class Person
{
    public string Name { get; set; }
    public int Age { get; set; }
    public string City { get; set; }
}

public class ComplexLinqExample
{
    public static void Main()
    {
        List<Person> people = new List<Person>
        {
            new Person { Name = "Ali", Age = 25, City = "İstanbul" },
            new Person { Name = "Ayşe", Age = 30, City = "Ankara" },
            new Person { Name = "Mehmet", Age = 20, City = "İzmir" },
            new Person { Name = "Zeynep", Age = 35, City = "İstanbul" },
            new Person { Name = "Fatma", Age = 28, City = "Ankara" },
            new Person { Name = "Ahmet", Age = 22, City = "İzmir" }
        };
        
        // Filtreleme ve sıralama
        var youngPeopleFromIstanbul = people
            .Where(p => p.City == "İstanbul" && p.Age < 30)
            .OrderBy(p => p.Age);
            
        Console.WriteLine("İstanbul'da yaşayan 30 yaş altı kişiler (yaşa göre sıralı):");
        foreach (var person in youngPeopleFromIstanbul)
        {
            Console.WriteLine($"{person.Name}, {person.Age}");
        }
        
        // Gruplama
        var peopleByCity = people
            .GroupBy(p => p.City)
            .Select(g => new { City = g.Key, Count = g.Count(), AverageAge = g.Average(p => p.Age) });
            
        Console.WriteLine("\nŞehirlere göre kişi sayıları ve yaş ortalamaları:");
        foreach (var cityGroup in peopleByCity)
        {
            Console.WriteLine($"{cityGroup.City}: {cityGroup.Count} kişi, Ortalama yaş: {cityGroup.AverageAge:F1}");
        }
        
        // Dönüştürme ve filtreleme
        var namesByAgeGroup = people
            .GroupBy(p => p.Age / 10 * 10) // 20-29, 30-39 gibi yaş grupları
            .OrderBy(g => g.Key)
            .Select(g => new { 
                AgeGroup = $"{g.Key}-{g.Key + 9}", 
                Names = string.Join(", ", g.Select(p => p.Name))
            });
            
        Console.WriteLine("\nYaş gruplarına göre kişiler:");
        foreach (var group in namesByAgeGroup)
        {
            Console.WriteLine($"{group.AgeGroup}: {group.Names}");
        }
    }
}
```

Lambda ifadeleri, C#'ta kısa ve okunabilir kod yazmanın güçlü bir yoludur. Delegeler, LINQ sorguları ve fonksiyonel programlama senaryolarında lambda ifadelerinin kullanımı, kodunuzu daha özlü ve anlaşılır hale getirebilir. 