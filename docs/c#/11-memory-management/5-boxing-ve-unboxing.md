# Boxing ve Unboxing

Boxing ve unboxing, .NET'te değer türleri (value types) ile referans türleri (reference types) arasındaki dönüşüm mekanizmalarıdır. Bu işlemler, bellek kullanımı ve performans açısından önemli etkilere sahiptir.

## 1. Value Type Boxing (Değer Türü Kutulama)

Boxing, bir değer türünün (struct, enum, primitive types) object türüne (referans türü) dönüştürülmesi işlemidir.

```csharp
// Boxing örneği
public void BoxingExample()
{
    // Değer türü
    int number = 42;
    
    // Boxing - değer türü object'e dönüştürülür
    object boxedNumber = number;
    
    Console.WriteLine($"Orijinal değer: {number}");
    Console.WriteLine($"Kutulanmış değer: {boxedNumber}");
    
    // Değer türü değiştirildiğinde kutulanmış nesne etkilenmez
    number = 100;
    
    Console.WriteLine($"Değiştirilen orijinal değer: {number}");
    Console.WriteLine($"Kutulanmış değer (değişmedi): {boxedNumber}");
}
```

### Boxing İşleminin Adımları

Boxing işlemi sırasında gerçekleşen adımlar:

```csharp
// Boxing işleminin adımları
public void BoxingSteps()
{
    // 1. Değer türü stack'te oluşturulur
    int value = 42;
    
    // 2. Boxing işlemi
    object boxed = value; // Şunlar gerçekleşir:
    // - Heap'te yeni bir object örneği ayrılır
    // - Değer türünün değeri heap'teki nesneye kopyalanır
    // - Object referansı stack'te saklanır
    
    // Bellek durumu gösterimi
    Console.WriteLine("Stack'te: value = 42");
    Console.WriteLine("Heap'te: boxed -> [42]");
    
    // 3. Değer türü değiştirildiğinde
    value = 100;
    
    // Bellek durumu gösterimi
    Console.WriteLine("Stack'te: value = 100");
    Console.WriteLine("Heap'te: boxed -> [42] (değişmedi)");
}
```

### Farklı Değer Türlerinin Kutulanması

Tüm değer türleri kutulanabilir.

```csharp
// Farklı değer türlerinin kutulanması
public void BoxingDifferentValueTypes()
{
    // Primitive değer türleri
    int intValue = 42;
    object boxedInt = intValue;
    
    double doubleValue = 3.14;
    object boxedDouble = doubleValue;
    
    bool boolValue = true;
    object boxedBool = boolValue;
    
    char charValue = 'A';
    object boxedChar = charValue;
    
    // Enum değer türü
    DayOfWeek day = DayOfWeek.Monday;
    object boxedEnum = day;
    
    // Struct değer türü
    DateTime dateValue = DateTime.Now;
    object boxedStruct = dateValue;
    
    Point point = new Point(10, 20);
    object boxedPoint = point;
    
    // Nullable değer türü
    int? nullableInt = 42;
    object boxedNullable = nullableInt;
    
    // Tüm kutulanmış değerleri yazdırma
    Console.WriteLine($"Kutulanmış int: {boxedInt}");
    Console.WriteLine($"Kutulanmış double: {boxedDouble}");
    Console.WriteLine($"Kutulanmış bool: {boxedBool}");
    Console.WriteLine($"Kutulanmış char: {boxedChar}");
    Console.WriteLine($"Kutulanmış enum: {boxedEnum}");
    Console.WriteLine($"Kutulanmış struct (DateTime): {boxedStruct}");
    Console.WriteLine($"Kutulanmış struct (Point): {boxedPoint}");
    Console.WriteLine($"Kutulanmış nullable: {boxedNullable}");
}

// Örnek struct
public struct Point
{
    public int X { get; }
    public int Y { get; }
    
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    public override string ToString() => $"({X}, {Y})";
}
```

## 2. Unboxing (Kutu Açma)

Unboxing, kutulanmış bir nesnenin tekrar değer türüne dönüştürülmesi işlemidir.

```csharp
// Unboxing örneği
public void UnboxingExample()
{
    // Değer türü
    int number = 42;
    
    // Boxing
    object boxedNumber = number;
    
    // Unboxing - object'ten değer türüne dönüştürme
    int unboxedNumber = (int)boxedNumber;
    
    Console.WriteLine($"Kutulanmış değer: {boxedNumber}");
    Console.WriteLine($"Kutusu açılmış değer: {unboxedNumber}");
    
    // Yanlış tür ile unboxing - hata oluşur
    try
    {
        double invalidUnboxing = (double)boxedNumber; // InvalidCastException
    }
    catch (InvalidCastException ex)
    {
        Console.WriteLine($"Hata: {ex.Message}");
    }
}
```

### Unboxing İşleminin Adımları

Unboxing işlemi sırasında gerçekleşen adımlar:

```csharp
// Unboxing işleminin adımları
public void UnboxingSteps()
{
    // 1. Değer türü ve boxing
    int value = 42;
    object boxed = value;
    
    // 2. Unboxing işlemi
    int unboxed = (int)boxed; // Şunlar gerçekleşir:
    // - Boxed nesnenin doğru türde olup olmadığı kontrol edilir
    // - Heap'teki değer stack'teki değişkene kopyalanır
    
    // Bellek durumu gösterimi
    Console.WriteLine("Stack'te: value = 42");
    Console.WriteLine("Heap'te: boxed -> [42]");
    Console.WriteLine("Stack'te: unboxed = 42");
    
    // 3. Unboxing sonrası değişiklikler
    unboxed = 100;
    
    // Bellek durumu gösterimi
    Console.WriteLine("Stack'te: value = 42");
    Console.WriteLine("Heap'te: boxed -> [42] (değişmedi)");
    Console.WriteLine("Stack'te: unboxed = 100");
}
```

### Güvenli Unboxing

Unboxing işleminin güvenli bir şekilde yapılması:

```csharp
// Güvenli unboxing
public void SafeUnboxing()
{
    // Çeşitli değer türleri
    object[] mixedObjects = new object[]
    {
        42,                // int
        3.14,              // double
        "Not a number",    // string (değer türü değil)
        true,              // bool
        DateTime.Now       // DateTime struct
    };
    
    foreach (object obj in mixedObjects)
    {
        // is operatörü ile tür kontrolü
        if (obj is int)
        {
            int number = (int)obj;
            Console.WriteLine($"Int değeri: {number}");
        }
        else if (obj is double)
        {
            double number = (double)obj;
            Console.WriteLine($"Double değeri: {number}");
        }
        else if (obj is bool)
        {
            bool flag = (bool)obj;
            Console.WriteLine($"Bool değeri: {flag}");
        }
        else if (obj is DateTime)
        {
            DateTime date = (DateTime)obj;
            Console.WriteLine($"DateTime değeri: {date}");
        }
        else
        {
            Console.WriteLine($"Değer türü değil: {obj}");
        }
        
        // C# 7.0+ pattern matching ile daha temiz yaklaşım
        switch (obj)
        {
            case int i:
                Console.WriteLine($"Int değeri (pattern): {i}");
                break;
            case double d:
                Console.WriteLine($"Double değeri (pattern): {d}");
                break;
            case bool b:
                Console.WriteLine($"Bool değeri (pattern): {b}");
                break;
            case DateTime dt:
                Console.WriteLine($"DateTime değeri (pattern): {dt}");
                break;
            default:
                Console.WriteLine($"Değer türü değil (pattern): {obj}");
                break;
        }
    }
}
```

## 3. Performance Impact (Performans Etkisi)

Boxing ve unboxing işlemleri performans açısından maliyetlidir.

```csharp
// Boxing ve unboxing performans etkisi
public void PerformanceImpact()
{
    const int iterations = 10_000_000;
    
    // Değer türü kullanımı
    var sw1 = Stopwatch.StartNew();
    
    int sum1 = 0;
    for (int i = 0; i < iterations; i++)
    {
        int value = i;
        sum1 += value;
    }
    
    sw1.Stop();
    Console.WriteLine($"Değer türü: {sw1.ElapsedMilliseconds} ms");
    
    // Boxing/unboxing kullanımı
    var sw2 = Stopwatch.StartNew();
    
    int sum2 = 0;
    for (int i = 0; i < iterations; i++)
    {
        object boxed = i; // Boxing
        int value = (int)boxed; // Unboxing
        sum2 += value;
    }
    
    sw2.Stop();
    Console.WriteLine($"Boxing/Unboxing: {sw2.ElapsedMilliseconds} ms");
    
    // Sonuçların doğruluğunu kontrol etme
    Console.WriteLine($"Sonuçlar eşit mi: {sum1 == sum2}");
    Console.WriteLine($"Performans farkı: {sw2.ElapsedMilliseconds / (double)sw1.ElapsedMilliseconds}x");
}
```

### Bellek Kullanımı Etkisi

Boxing işlemi ek bellek kullanımına neden olur.

```csharp
// Boxing'in bellek kullanımına etkisi
public void MemoryImpact()
{
    // Başlangıç bellek kullanımı
    long memoryBefore = GC.GetTotalMemory(true);
    
    // Değer türleri için dizi
    int[] valueArray = new int[1_000_000];
    for (int i = 0; i < valueArray.Length; i++)
    {
        valueArray[i] = i;
    }
    
    // Değer türü dizi sonrası bellek
    long memoryAfterValueArray = GC.GetTotalMemory(true);
    
    // Kutulanmış nesneler için dizi
    object[] boxedArray = new object[1_000_000];
    for (int i = 0; i < boxedArray.Length; i++)
    {
        boxedArray[i] = i; // Boxing
    }
    
    // Kutulanmış dizi sonrası bellek
    long memoryAfterBoxedArray = GC.GetTotalMemory(true);
    
    // Bellek kullanımı karşılaştırması
    Console.WriteLine($"Değer türü dizi bellek kullanımı: {(memoryAfterValueArray - memoryBefore) / 1024} KB");
    Console.WriteLine($"Kutulanmış dizi bellek kullanımı: {(memoryAfterBoxedArray - memoryAfterValueArray) / 1024} KB");
    Console.WriteLine($"Fark: {(memoryAfterBoxedArray - memoryAfterValueArray) / (double)(memoryAfterValueArray - memoryBefore)}x");
}
```

## 4. Boxing Scenarios (Kutulama Senaryoları)

Boxing işlemi çeşitli senaryolarda otomatik olarak gerçekleşir.

```csharp
// Yaygın boxing senaryoları
public void CommonBoxingScenarios()
{
    // 1. Object parametreli metotlara değer türü geçme
    int number = 42;
    PrintObject(number); // Boxing
    
    // 2. Interface uygulaması üzerinden değer türüne erişme
    int comparableValue = 100;
    IComparable comparable = comparableValue; // Boxing
    int comparisonResult = comparable.CompareTo(50);
    
    // 3. Object koleksiyonlarına değer türü ekleme
    ArrayList list = new ArrayList();
    list.Add(42); // Boxing
    list.Add(3.14); // Boxing
    list.Add(true); // Boxing
    
    // 4. String interpolation/concatenation
    int x = 10;
    int y = 20;
    string coordinates = $"Koordinatlar: {x}, {y}"; // Boxing
    
    // 5. Params object[] parametreli metotlar
    PrintValues(1, 2.0, "üç", true); // Boxing
    
    // 6. Nullable<T> kullanımı
    int? nullableInt = 42;
    object boxedNullable = nullableInt; // Boxing
}

// Object parametreli metot
private void PrintObject(object obj)
{
    Console.WriteLine($"Nesne: {obj}");
}

// Params object[] parametreli metot
private void PrintValues(params object[] values)
{
    foreach (var value in values)
    {
        Console.WriteLine($"Değer: {value}");
    }
}
```

### Gizli Boxing Senaryoları

Bazı durumlarda boxing işlemi fark edilmeden gerçekleşebilir.

```csharp
// Gizli boxing senaryoları
public void HiddenBoxingScenarios()
{
    // 1. Enum.ToString() çağrısı
    DayOfWeek day = DayOfWeek.Monday;
    string dayName = day.ToString(); // Boxing
    
    // 2. Value type üzerinde virtual metot çağrısı
    int number = 42;
    string numberString = number.ToString(); // Boxing
    
    // 3. Value type üzerinde GetType() çağrısı
    Type numberType = number.GetType(); // Boxing
    
    // 4. Value type üzerinde extension metot çağrısı (bazı durumlarda)
    bool isEven = number.IsEven(); // Extension metoda bağlı
    
    // 5. LINQ sorguları
    int[] numbers = { 1, 2, 3, 4, 5 };
    var evenNumbers = numbers.Where(n => n % 2 == 0); // Implementasyona bağlı
    
    // 6. Dynamic kullanımı
    dynamic dynamicNumber = 42;
    dynamicNumber++; // Boxing/unboxing olabilir
}

// Extension metot örneği
public static class Extensions
{
    public static bool IsEven(this int number)
    {
        return number % 2 == 0;
    }
}
```

## 5. Avoiding Boxing (Kutulama İşleminden Kaçınma)

Boxing işleminden kaçınmak için çeşitli stratejiler:

```csharp
// Boxing işleminden kaçınma
public void AvoidingBoxing()
{
    // 1. Generic koleksiyonlar kullanma
    // Kötü: ArrayList (boxing gerektirir)
    ArrayList arrayList = new ArrayList();
    arrayList.Add(42); // Boxing
    int arrayListValue = (int)arrayList[0]; // Unboxing
    
    // İyi: Generic List<T> (boxing gerektirmez)
    List<int> genericList = new List<int>();
    genericList.Add(42); // Boxing yok
    int genericListValue = genericList[0]; // Unboxing yok
    
    // 2. Generic metotlar kullanma
    // Kötü: Object parametreli metot
    PrintObject(42); // Boxing
    
    // İyi: Generic metot
    PrintGeneric(42); // Boxing yok
    
    // 3. Uygun arayüzler kullanma
    // Kötü: IComparable (boxing gerektirir)
    IComparable comparable = 42; // Boxing
    
    // İyi: IComparable<T> (boxing gerektirmez)
    IComparable<int> genericComparable = 42; // Boxing yok
    
    // 4. StringBuilder kullanma (string concatenation yerine)
    // Kötü: String interpolation/concatenation
    int x = 10, y = 20;
    string badCoordinates = $"Koordinatlar: {x}, {y}"; // Boxing
    
    // İyi: StringBuilder
    var sb = new StringBuilder();
    sb.Append("Koordinatlar: ");
    sb.Append(x);
    sb.Append(", ");
    sb.Append(y);
    string goodCoordinates = sb.ToString(); // Boxing yok
    
    // 5. ToString() yerine alternatifler kullanma
    // Kötü: ToString()
    int number = 42;
    string numberString1 = number.ToString(); // Boxing
    
    // İyi: String.Format veya alternatif metotlar
    string numberString2 = number.ToString(CultureInfo.InvariantCulture); // Daha az boxing
    string numberString3 = string.Format("{0}", number); // Daha kontrollü boxing
}

// Generic metot örneği
private void PrintGeneric<T>(T value)
{
    Console.WriteLine($"Değer: {value}");
}
```

## 6. Generic Alternatives (Generic Alternatifler)

Generic'ler, boxing işleminden kaçınmak için en etkili yöntemlerden biridir.

```csharp
// Generic alternatifler
public void GenericAlternatives()
{
    // 1. Generic koleksiyonlar
    List<int> intList = new List<int>();
    Dictionary<string, int> dictionary = new Dictionary<string, int>();
    HashSet<double> doubleSet = new HashSet<double>();
    
    // 2. Generic metotlar
    int max = FindMax(1, 2, 3, 4, 5);
    double average = CalculateAverage(1.0, 2.0, 3.0);
    
    // 3. Generic sınıflar
    var intProcessor = new ValueProcessor<int>();
    intProcessor.Process(42);
    
    var dateProcessor = new ValueProcessor<DateTime>();
    dateProcessor.Process(DateTime.Now);
    
    // 4. Generic kısıtlamalar
    var calculator = new Calculator<int>();
    int result = calculator.Add(5, 10);
    
    // 5. Generic delegeler
    Func<int, int, int> addFunc = (a, b) => a + b;
    int sum = addFunc(5, 10);
    
    // 6. Generic events
    var eventSource = new EventSource<int>();
    eventSource.ValueChanged += (sender, value) => Console.WriteLine($"Değer değişti: {value}");
    eventSource.RaiseValueChanged(42);
}

// Generic metot örneği
private T FindMax<T>(params T[] values) where T : IComparable<T>
{
    if (values == null || values.Length == 0)
    {
        throw new ArgumentException("En az bir değer gerekli", nameof(values));
    }
    
    T max = values[0];
    for (int i = 1; i < values.Length; i++)
    {
        if (values[i].CompareTo(max) > 0)
        {
            max = values[i];
        }
    }
    
    return max;
}

// Generic metot örneği
private double CalculateAverage<T>(params T[] values) where T : struct, IConvertible
{
    if (values == null || values.Length == 0)
    {
        throw new ArgumentException("En az bir değer gerekli", nameof(values));
    }
    
    double sum = 0;
    foreach (var value in values)
    {
        sum += Convert.ToDouble(value);
    }
    
    return sum / values.Length;
}

// Generic sınıf örneği
public class ValueProcessor<T> where T : struct
{
    public void Process(T value)
    {
        Console.WriteLine($"İşleniyor: {value}");
    }
}

// Generic kısıtlama örneği
public class Calculator<T> where T : struct, IConvertible
{
    public T Add(T a, T b)
    {
        double result = Convert.ToDouble(a) + Convert.ToDouble(b);
        return (T)Convert.ChangeType(result, typeof(T));
    }
}

// Generic event örneği
public class EventSource<T>
{
    public event EventHandler<T> ValueChanged;
    
    public void RaiseValueChanged(T value)
    {
        ValueChanged?.Invoke(this, value);
    }
}
```

## 7. Collection Considerations (Koleksiyon Değerlendirmeleri)

Koleksiyonlarda boxing/unboxing işlemlerinin etkisi:

```csharp
// Koleksiyon değerlendirmeleri
public void CollectionConsiderations()
{
    const int itemCount = 1_000_000;
    
    // 1. ArrayList vs List<T> performans karşılaştırması
    var sw1 = Stopwatch.StartNew();
    
    ArrayList arrayList = new ArrayList();
    for (int i = 0; i < itemCount; i++)
    {
        arrayList.Add(i); // Boxing
    }
    
    int arrayListSum = 0;
    foreach (object item in arrayList)
    {
        arrayListSum += (int)item; // Unboxing
    }
    
    sw1.Stop();
    Console.WriteLine($"ArrayList: {sw1.ElapsedMilliseconds} ms");
    
    var sw2 = Stopwatch.StartNew();
    
    List<int> genericList = new List<int>();
    for (int i = 0; i < itemCount; i++)
    {
        genericList.Add(i); // Boxing yok
    }
    
    int genericListSum = 0;
    foreach (int item in genericList)
    {
        genericListSum += item; // Unboxing yok
    }
    
    sw2.Stop();
    Console.WriteLine($"List<int>: {sw2.ElapsedMilliseconds} ms");
    Console.WriteLine($"Performans farkı: {sw1.ElapsedMilliseconds / (double)sw2.ElapsedMilliseconds}x");
    
    // 2. Dictionary<TKey, TValue> vs Hashtable
    var sw3 = Stopwatch.StartNew();
    
    Hashtable hashtable = new Hashtable();
    for (int i = 0; i < itemCount; i++)
    {
        hashtable[i] = i * 2; // Boxing
    }
    
    int hashtableSum = 0;
    foreach (DictionaryEntry entry in hashtable)
    {
        hashtableSum += (int)entry.Value; // Unboxing
    }
    
    sw3.Stop();
    Console.WriteLine($"Hashtable: {sw3.ElapsedMilliseconds} ms");
    
    var sw4 = Stopwatch.StartNew();
    
    Dictionary<int, int> genericDict = new Dictionary<int, int>();
    for (int i = 0; i < itemCount; i++)
    {
        genericDict[i] = i * 2; // Boxing yok
    }
    
    int genericDictSum = 0;
    foreach (var entry in genericDict)
    {
        genericDictSum += entry.Value; // Unboxing yok
    }
    
    sw4.Stop();
    Console.WriteLine($"Dictionary<int, int>: {sw4.ElapsedMilliseconds} ms");
    Console.WriteLine($"Performans farkı: {sw3.ElapsedMilliseconds / (double)sw4.ElapsedMilliseconds}x");
}
```

### Özel Koleksiyon Stratejileri

Değer türleri için özel koleksiyon stratejileri:

```csharp
// Özel koleksiyon stratejileri
public void CustomCollectionStrategies()
{
    // 1. Struct koleksiyonu için özel sınıf
    var points = new PointCollection();
    points.Add(new Point(1, 2));
    points.Add(new Point(3, 4));
    
    foreach (var point in points)
    {
        Console.WriteLine(point);
    }
    
    // 2. Span<T> ve Memory<T> kullanımı
    int[] array = { 1, 2, 3, 4, 5 };
    Span<int> span = array;
    
    for (int i = 0; i < span.Length; i++)
    {
        span[i] *= 2;
    }
    
    // 3. ArrayPool<T> kullanımı
    ArrayPool<int> pool = ArrayPool<int>.Shared;
    int[] rentedArray = pool.Rent(100);
    
    try
    {
        // Diziyi kullan
        for (int i = 0; i < 100; i++)
        {
            rentedArray[i] = i;
        }
    }
    finally
    {
        pool.Return(rentedArray);
    }
    
    // 4. Değer türü koleksiyonları için struct enumerator
    var valueCollection = new ValueCollection<int>();
    valueCollection.Add(1);
    valueCollection.Add(2);
    valueCollection.Add(3);
    
    foreach (int value in valueCollection)
    {
        Console.WriteLine(value);
    }
}

// Özel Point koleksiyonu
public class PointCollection : IEnumerable<Point>
{
    private readonly List<Point> _points = new List<Point>();
    
    public void Add(Point point)
    {
        _points.Add(point);
    }
    
    public IEnumerator<Point> GetEnumerator()
    {
        return _points.GetEnumerator();
    }
    
    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}

// Struct enumerator örneği
public class ValueCollection<T> : IEnumerable<T> where T : struct
{
    private readonly List<T> _values = new List<T>();
    
    public void Add(T value)
    {
        _values.Add(value);
    }
    
    public Enumerator GetEnumerator()
    {
        return new Enumerator(this);
    }
    
    IEnumerator<T> IEnumerable<T>.GetEnumerator()
    {
        return GetEnumerator();
    }
    
    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
    
    // Struct enumerator - boxing önler
    public struct Enumerator : IEnumerator<T>
    {
        private readonly ValueCollection<T> _collection;
        private int _index;
        private T _current;
        
        public Enumerator(ValueCollection<T> collection)
        {
            _collection = collection;
            _index = -1;
            _current = default;
        }
        
        public T Current => _current;
        
        object IEnumerator.Current => Current;
        
        public bool MoveNext()
        {
            if (++_index < _collection._values.Count)
            {
                _current = _collection._values[_index];
                return true;
            }
            
            _current = default;
            return false;
        }
        
        public void Reset()
        {
            _index = -1;
            _current = default;
        }
        
        public void Dispose()
        {
        }
    }
}
```

## Özet

Bu bölümde, .NET'teki boxing ve unboxing işlemlerini ve bunların performans etkilerini inceledik:

1. **Value Type Boxing**: Değer türlerinin object türüne dönüştürülmesi.

2. **Unboxing**: Kutulanmış nesnelerin tekrar değer türüne dönüştürülmesi.

3. **Performance Impact**: Boxing ve unboxing işlemlerinin performans ve bellek kullanımına etkileri.

4. **Boxing Scenarios**: Yaygın ve gizli boxing senaryoları.

5. **Avoiding Boxing**: Boxing işleminden kaçınma stratejileri.

6. **Generic Alternatives**: Boxing işlemini önlemek için generic kullanımı.

7. **Collection Considerations**: Koleksiyonlarda boxing/unboxing etkilerini azaltma.

Boxing ve unboxing işlemlerinden mümkün olduğunca kaçınmak, .NET uygulamalarının performansını artırmak için önemli bir adımdır. Generic türler ve modern .NET API'leri, bu işlemleri minimize etmek için birçok olanak sunar. 