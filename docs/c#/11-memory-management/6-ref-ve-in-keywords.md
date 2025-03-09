# ref ve in Keywords (ref ve in Anahtar Kelimeleri)

C# dilinde `ref` ve `in` anahtar kelimeleri, değer türlerinin referans semantiği ile kullanılmasını sağlayan güçlü özelliklerdir. Bu özellikler, performans optimizasyonu ve bellek yönetimi açısından önemli avantajlar sunar.

## 1. ref Returns (ref Dönüşler)

`ref` dönüşler, bir metodun değer türünün referansını döndürmesine olanak tanır.

```csharp
// ref dönüş örneği
public ref int FindLargest(int[] numbers)
{
    if (numbers == null || numbers.Length == 0)
    {
        throw new ArgumentException("Dizi boş olamaz", nameof(numbers));
    }
    
    int largestIndex = 0;
    
    for (int i = 1; i < numbers.Length; i++)
    {
        if (numbers[i] > numbers[largestIndex])
        {
            largestIndex = i;
        }
    }
    
    // Dizideki en büyük elemanın referansını döndür
    return ref numbers[largestIndex];
}

// Kullanım örneği
public void RefReturnExample()
{
    int[] numbers = { 1, 5, 3, 9, 7 };
    
    // En büyük elemanın referansını al
    ref int largest = ref FindLargest(numbers);
    
    Console.WriteLine($"En büyük değer: {largest}"); // 9
    
    // Referans üzerinden değeri değiştir
    largest = 100;
    
    Console.WriteLine($"Değiştirilen değer: {largest}"); // 100
    Console.WriteLine($"Dizideki değer: {numbers[3]}"); // 100 - orijinal dizi değişti
    
    // Diziyi yazdır
    Console.WriteLine($"Dizi: [{string.Join(", ", numbers)}]"); // [1, 5, 3, 100, 7]
}
```

### ref Dönüşlerin Avantajları

`ref` dönüşlerin sağladığı avantajlar:

```csharp
// ref dönüşlerin avantajları
public void RefReturnAdvantages()
{
    // 1. Performans: Büyük struct'lar için kopyalama maliyetini önler
    LargeStruct[] largeStructs = new LargeStruct[100];
    for (int i = 0; i < largeStructs.Length; i++)
    {
        largeStructs[i] = new LargeStruct { Id = i };
    }
    
    // Referans olarak al ve değiştir
    ref LargeStruct largeStruct = ref GetLargeStruct(largeStructs, 50);
    largeStruct.Id = 999;
    
    // 2. Doğrudan erişim: Koleksiyonlardaki elemanlara doğrudan erişim
    int[,] matrix = new int[10, 10];
    ref int cell = ref GetMatrixElement(matrix, 5, 5);
    cell = 42;
    
    // 3. Zincirleme: Referans zinciri oluşturma
    int[] data = { 1, 2, 3 };
    ref int first = ref data[0];
    ref int anotherRef = ref first; // Referans zinciri
    anotherRef = 100;
    
    Console.WriteLine($"data[0]: {data[0]}"); // 100
}

// Büyük struct örneği
public struct LargeStruct
{
    public int Id;
    public byte[] Data; // Büyük veri
    
    public LargeStruct(int id)
    {
        Id = id;
        Data = new byte[1024]; // 1 KB
    }
}

// Büyük struct referansı döndüren metot
public ref LargeStruct GetLargeStruct(LargeStruct[] array, int index)
{
    return ref array[index];
}

// Matris elemanı referansı döndüren metot
public ref int GetMatrixElement(int[,] matrix, int row, int col)
{
    return ref matrix[row, col];
}
```

## 2. ref Locals (ref Yerel Değişkenler)

`ref` yerel değişkenler, bir değer türünün referansını tutan yerel değişkenlerdir.

```csharp
// ref yerel değişken örneği
public void RefLocalExample()
{
    int value = 10;
    
    // value değişkeninin referansını tutan ref yerel değişken
    ref int refValue = ref value;
    
    // refValue üzerinden değeri değiştir
    refValue = 20;
    
    Console.WriteLine($"value: {value}"); // 20
    Console.WriteLine($"refValue: {refValue}"); // 20
    
    // value değişkenini doğrudan değiştir
    value = 30;
    
    Console.WriteLine($"value: {value}"); // 30
    Console.WriteLine($"refValue: {refValue}"); // 30 - aynı bellek konumunu gösterir
}
```

### ref Yerel Değişkenlerin Kullanım Alanları

`ref` yerel değişkenlerin kullanım alanları:

```csharp
// ref yerel değişkenlerin kullanım alanları
public void RefLocalUseCases()
{
    // 1. Dizi elemanlarına referans
    int[] numbers = { 1, 2, 3, 4, 5 };
    ref int third = ref numbers[2];
    third *= 10;
    
    Console.WriteLine($"numbers[2]: {numbers[2]}"); // 30
    
    // 2. Struct alanlarına referans
    Point point = new Point(10, 20);
    ref int xCoord = ref point.X;
    xCoord = 100;
    
    Console.WriteLine($"point: {point}"); // (100, 20)
    
    // 3. Referans değiştirme
    int a = 1, b = 2;
    ref int refValue = ref a;
    Console.WriteLine($"refValue: {refValue}"); // 1
    
    refValue = ref b; // Referansı değiştir
    Console.WriteLine($"refValue: {refValue}"); // 2
    
    // 4. Metot dönüşüyle birlikte kullanım
    int[] data = { 5, 10, 15 };
    ref int max = ref FindLargest(data);
    max = 100;
    
    Console.WriteLine($"data: [{string.Join(", ", data)}]"); // [5, 10, 100]
}

// Değiştirilebilir X ve Y koordinatları olan Point struct'ı
public struct Point
{
    public int X;
    public int Y;
    
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    public override string ToString() => $"({X}, {Y})";
}
```

## 3. in Parameters (in Parametreleri)

`in` parametreleri, değer türlerinin referans olarak ancak salt okunur şekilde geçirilmesini sağlar.

```csharp
// in parametre örneği
public double CalculateDistance(in Point p1, in Point p2)
{
    // p1 ve p2 salt okunur referanslardır
    // p1.X = 100; // Hata: salt okunur referans değiştirilemez
    
    int deltaX = p2.X - p1.X;
    int deltaY = p2.Y - p1.Y;
    
    return Math.Sqrt(deltaX * deltaX + deltaY * deltaY);
}

// Kullanım örneği
public void InParameterExample()
{
    Point point1 = new Point(0, 0);
    Point point2 = new Point(3, 4);
    
    double distance = CalculateDistance(in point1, in point2);
    
    Console.WriteLine($"İki nokta arasındaki mesafe: {distance}"); // 5
    
    // in anahtar kelimesi isteğe bağlıdır
    distance = CalculateDistance(point1, point2);
    Console.WriteLine($"İki nokta arasındaki mesafe: {distance}"); // 5
}
```

### in Parametrelerinin Avantajları

`in` parametrelerinin sağladığı avantajlar:

```csharp
// in parametrelerinin avantajları
public void InParameterAdvantages()
{
    // 1. Büyük struct'lar için performans avantajı
    LargeStruct large = new LargeStruct(42);
    ProcessLargeStruct(in large); // Kopyalama olmadan geçirilir
    
    // 2. Değişmezlik garantisi
    Point point = new Point(10, 20);
    PrintPoint(in point); // point değiştirilemez
    
    // 3. Nullable değer türleri için
    int? nullableValue = 42;
    ProcessNullable(in nullableValue);
    
    // 4. Readonly struct'lar ile kullanım
    ReadOnlyPoint readOnlyPoint = new ReadOnlyPoint(5, 10);
    ProcessReadOnlyPoint(in readOnlyPoint);
}

// Büyük struct'ı işleyen metot
public void ProcessLargeStruct(in LargeStruct large)
{
    Console.WriteLine($"LargeStruct ID: {large.Id}");
    // large.Id = 100; // Hata: salt okunur referans değiştirilemez
}

// Noktayı yazdıran metot
public void PrintPoint(in Point point)
{
    Console.WriteLine($"Nokta: {point}");
    // point.X = 100; // Hata: salt okunur referans değiştirilemez
}

// Nullable değer türünü işleyen metot
public void ProcessNullable(in int? value)
{
    if (value.HasValue)
    {
        Console.WriteLine($"Değer: {value.Value}");
    }
    else
    {
        Console.WriteLine("Değer null");
    }
}

// Salt okunur struct
public readonly struct ReadOnlyPoint
{
    public readonly int X { get; }
    public readonly int Y { get; }
    
    public ReadOnlyPoint(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    public override string ToString() => $"({X}, {Y})";
}

// Salt okunur struct'ı işleyen metot
public void ProcessReadOnlyPoint(in ReadOnlyPoint point)
{
    Console.WriteLine($"ReadOnlyPoint: {point}");
}
```

## 4. ref readonly (ref Salt Okunur)

`ref readonly` dönüşler, bir değer türünün salt okunur referansını döndürür.

```csharp
// ref readonly dönüş örneği
public ref readonly Point GetOrigin()
{
    // Statik bir Point nesnesi
    static Point origin = new Point(0, 0);
    
    // Salt okunur referans olarak döndür
    return ref origin;
}

// Kullanım örneği
public void RefReadonlyExample()
{
    // Salt okunur referans al
    ref readonly Point origin = ref GetOrigin();
    
    Console.WriteLine($"Orijin: {origin}"); // (0, 0)
    
    // origin.X = 100; // Hata: salt okunur referans değiştirilemez
    
    // Normal değişkene kopyalama
    Point copyOfOrigin = origin;
    copyOfOrigin.X = 100; // Kopya değiştirilebilir
    
    Console.WriteLine($"Kopya: {copyOfOrigin}"); // (100, 0)
    Console.WriteLine($"Orijin: {origin}"); // (0, 0) - değişmedi
}
```

### ref readonly vs in

`ref readonly` ve `in` arasındaki farklar:

```csharp
// ref readonly vs in
public void RefReadonlyVsIn()
{
    // 1. ref readonly - dönüş değeri için
    ref readonly int refReadonlyValue = ref GetConstantValue();
    // refReadonlyValue = 100; // Hata: salt okunur
    
    // 2. in - parametre için
    int value = 42;
    ProcessValue(in value);
    
    // 3. Performans karakteristikleri benzer
    LargeStruct large = new LargeStruct(42);
    
    // Her ikisi de kopyalama yapmaz
    ref readonly LargeStruct refReadonlyLarge = ref large;
    ProcessLargeStruct(in large);
    
    // 4. Kullanım amacı farklı
    // - ref readonly: Değiştirilemez referans döndürmek için
    // - in: Değiştirilemez referans parametresi için
}

// Sabit değer döndüren metot
public ref readonly int GetConstantValue()
{
    static int constant = 42;
    return ref constant;
}

// in parametreli metot
public void ProcessValue(in int value)
{
    Console.WriteLine($"Değer: {value}");
    // value = 100; // Hata: salt okunur
}
```

## 5. Performance Benefits (Performans Faydaları)

`ref`, `in` ve `ref readonly` kullanımının performans faydaları:

```csharp
// Performans faydaları
public void PerformanceBenefits()
{
    const int iterations = 10_000_000;
    
    // Büyük struct örneği
    var largeStruct = new LargeStruct(1);
    
    // 1. Normal parametre geçişi (kopyalama)
    var sw1 = Stopwatch.StartNew();
    
    for (int i = 0; i < iterations; i++)
    {
        ProcessByValue(largeStruct);
    }
    
    sw1.Stop();
    Console.WriteLine($"Değer ile geçiş: {sw1.ElapsedMilliseconds} ms");
    
    // 2. in parametre geçişi (referans)
    var sw2 = Stopwatch.StartNew();
    
    for (int i = 0; i < iterations; i++)
    {
        ProcessByIn(in largeStruct);
    }
    
    sw2.Stop();
    Console.WriteLine($"in ile geçiş: {sw2.ElapsedMilliseconds} ms");
    
    // 3. ref parametre geçişi (referans)
    var sw3 = Stopwatch.StartNew();
    
    for (int i = 0; i < iterations; i++)
    {
        ProcessByRef(ref largeStruct);
    }
    
    sw3.Stop();
    Console.WriteLine($"ref ile geçiş: {sw3.ElapsedMilliseconds} ms");
    
    // Performans farkı
    Console.WriteLine($"Değer vs in farkı: {sw1.ElapsedMilliseconds / (double)sw2.ElapsedMilliseconds}x");
    Console.WriteLine($"Değer vs ref farkı: {sw1.ElapsedMilliseconds / (double)sw3.ElapsedMilliseconds}x");
}

// Değer ile parametre alan metot
public void ProcessByValue(LargeStruct large)
{
    // Struct kopyalanır
    int id = large.Id;
}

// in ile parametre alan metot
public void ProcessByIn(in LargeStruct large)
{
    // Struct referans olarak geçirilir (salt okunur)
    int id = large.Id;
}

// ref ile parametre alan metot
public void ProcessByRef(ref LargeStruct large)
{
    // Struct referans olarak geçirilir (değiştirilebilir)
    int id = large.Id;
}
```

### Bellek Kullanımı Optimizasyonu

`ref` ve `in` kullanımının bellek optimizasyonu:

```csharp
// Bellek kullanımı optimizasyonu
public void MemoryOptimization()
{
    // 1. Dizi işlemleri için ref kullanımı
    int[] array = new int[1000];
    
    // Geleneksel yaklaşım
    for (int i = 0; i < array.Length; i++)
    {
        array[i] = i * 2;
    }
    
    // ref kullanarak optimizasyon
    for (int i = 0; i < array.Length; i++)
    {
        ref int item = ref array[i];
        item = i * 2;
    }
    
    // 2. Büyük struct'lar için in kullanımı
    var points = new Point[1000];
    for (int i = 0; i < points.Length; i++)
    {
        points[i] = new Point(i, i);
    }
    
    double totalDistance = 0;
    
    // Geleneksel yaklaşım - her seferinde kopyalama
    for (int i = 1; i < points.Length; i++)
    {
        totalDistance += CalculateDistance(points[i - 1], points[i]);
    }
    
    // in kullanarak optimizasyon - kopyalama yok
    totalDistance = 0;
    for (int i = 1; i < points.Length; i++)
    {
        totalDistance += CalculateDistance(in points[i - 1], in points[i]);
    }
}
```

## 6. Safety Considerations (Güvenlik Değerlendirmeleri)

`ref` ve `in` kullanırken dikkat edilmesi gereken güvenlik hususları:

```csharp
// Güvenlik değerlendirmeleri
public void SafetyConsiderations()
{
    // 1. Geçici değerlere ref verilmemesi
    int value = 42;
    ref int refValue = ref value; // Güvenli
    
    // ref int badRef = ref GetTemporaryValue(); // Hata: geçici değere ref verilemez
    
    // 2. Null referans kontrolü
    int[] array = null;
    
    try
    {
        ref int item = ref GetArrayItem(array, 0);
        item = 42;
    }
    catch (ArgumentNullException ex)
    {
        Console.WriteLine($"Hata: {ex.Message}");
    }
    
    // 3. Sınırlar dışına çıkma kontrolü
    int[] validArray = { 1, 2, 3 };
    
    try
    {
        ref int outOfBounds = ref GetArrayItem(validArray, 10);
        outOfBounds = 42;
    }
    catch (IndexOutOfRangeException ex)
    {
        Console.WriteLine($"Hata: {ex.Message}");
    }
    
    // 4. ref readonly ile değiştirme girişimi
    ref readonly int readonlyRef = ref GetConstantValue();
    // readonlyRef = 100; // Hata: salt okunur referans değiştirilemez
    
    // 5. Yerel değişken kapsamı
    {
        int localValue = 42;
        // ref int escapingRef = ref localValue; // Tehlikeli - kapsam dışına çıkabilir
    }
    // escapingRef artık geçersiz bir bellek konumunu gösterir
}

// Geçici değer döndüren metot
public int GetTemporaryValue()
{
    return 42;
}

// Dizi elemanı referansı döndüren güvenli metot
public ref int GetArrayItem(int[] array, int index)
{
    if (array == null)
    {
        throw new ArgumentNullException(nameof(array));
    }
    
    if (index < 0 || index >= array.Length)
    {
        throw new IndexOutOfRangeException($"Geçersiz indeks: {index}");
    }
    
    return ref array[index];
}
```

## 7. Best Practices (En İyi Uygulamalar)

`ref` ve `in` kullanımı için en iyi uygulamalar:

```csharp
// En iyi uygulamalar
public void BestPractices()
{
    // 1. Büyük struct'lar için in kullanımı
    var largePoint = new LargePoint(10, 20);
    ProcessLargePoint(in largePoint); // Performans için in kullan
    
    // 2. Küçük struct'lar için normal parametre kullanımı
    var smallPoint = new Point(10, 20);
    ProcessSmallPoint(smallPoint); // Küçük struct'lar için normal parametre yeterli
    
    // 3. ref readonly dönüş değerleri için readonly struct kullanımı
    ref readonly ReadOnlyPoint origin = ref GetReadOnlyOrigin();
    Console.WriteLine($"Origin: {origin}");
    
    // 4. Değiştirilebilir referanslar için ref kullanımı
    int[] numbers = { 1, 2, 3 };
    ref int firstNumber = ref numbers[0];
    firstNumber = 100; // Değiştirilebilir
    
    // 5. Performans kritik kod için ref locals kullanımı
    ProcessArrayFast(numbers);
    
    // 6. API tasarımında tutarlılık
    // - Değiştirilebilir referanslar için ref
    // - Salt okunur referanslar için in
    // - Salt okunur dönüşler için ref readonly
}

// Büyük struct örneği
public struct LargePoint
{
    public int X;
    public int Y;
    public byte[] Data; // Büyük veri
    
    public LargePoint(int x, int y)
    {
        X = x;
        Y = y;
        Data = new byte[1024]; // 1 KB
    }
}

// Büyük struct için in parametre
public void ProcessLargePoint(in LargePoint point)
{
    Console.WriteLine($"LargePoint: ({point.X}, {point.Y})");
}

// Küçük struct için normal parametre
public void ProcessSmallPoint(Point point)
{
    Console.WriteLine($"SmallPoint: {point}");
}

// readonly struct döndüren metot
public ref readonly ReadOnlyPoint GetReadOnlyOrigin()
{
    static ReadOnlyPoint origin = new ReadOnlyPoint(0, 0);
    return ref origin;
}

// ref locals ile hızlı dizi işleme
public void ProcessArrayFast(int[] array)
{
    for (int i = 0; i < array.Length; i++)
    {
        ref int item = ref array[i];
        item *= 2; // Doğrudan referans üzerinden değiştirme
    }
}
```

### Performans İpuçları

`ref` ve `in` kullanımı için performans ipuçları:

```csharp
// Performans ipuçları
public void PerformanceTips()
{
    // 1. 16 bayttan büyük struct'lar için in kullanımı
    var vector = new Vector3D(1.0, 2.0, 3.0);
    double length = CalculateVectorLength(in vector);
    
    // 2. Yüksek çağrı frekansı olan metotlar için ref kullanımı
    int[] data = new int[1000];
    FillArray(data);
    
    // 3. Zincirleme metot çağrıları için ref dönüşler
    ref int value = ref FindAndModify(data, 42);
    value = 100;
    
    // 4. Struct'lar için readonly metotlar
    var point = new PointWithMethods(10, 20);
    double distanceFromOrigin = point.DistanceFromOrigin();
    
    // 5. Kritik performans gerektiren döngüler için ref locals
    SumArrayFast(data);
}

// 3D vektör struct'ı (24 bayt)
public struct Vector3D
{
    public double X;
    public double Y;
    public double Z;
    
    public Vector3D(double x, double y, double z)
    {
        X = x;
        Y = y;
        Z = z;
    }
}

// in parametre ile vektör uzunluğu hesaplama
public double CalculateVectorLength(in Vector3D vector)
{
    return Math.Sqrt(vector.X * vector.X + vector.Y * vector.Y + vector.Z * vector.Z);
}

// ref parametre ile dizi doldurma
public void FillArray(int[] array)
{
    for (int i = 0; i < array.Length; i++)
    {
        ref int item = ref array[i];
        item = i * 2;
    }
}

// Değer bulup referansını döndürme
public ref int FindAndModify(int[] array, int valueToFind)
{
    for (int i = 0; i < array.Length; i++)
    {
        if (array[i] == valueToFind)
        {
            return ref array[i];
        }
    }
    
    // Bulunamazsa ilk elemanı döndür
    return ref array[0];
}

// readonly metotlu struct
public struct PointWithMethods
{
    public int X;
    public int Y;
    
    public PointWithMethods(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    // readonly metot - struct'ı değiştirmez
    public readonly double DistanceFromOrigin()
    {
        return Math.Sqrt(X * X + Y * Y);
    }
}

// ref locals ile hızlı toplama
public int SumArrayFast(int[] array)
{
    int sum = 0;
    for (int i = 0; i < array.Length; i++)
    {
        ref int item = ref array[i];
        sum += item;
    }
    return sum;
}
```

## Özet

Bu bölümde, C#'taki `ref` ve `in` anahtar kelimelerini ve bunların kullanım alanlarını inceledik:

1. **ref Returns**: Değer türlerinin referansını döndürme.

2. **ref Locals**: Değer türlerinin referansını tutan yerel değişkenler.

3. **in Parameters**: Değer türlerini salt okunur referans olarak geçirme.

4. **ref readonly**: Salt okunur referans döndürme.

5. **Performance Benefits**: `ref` ve `in` kullanımının performans avantajları.

6. **Safety Considerations**: Güvenli kullanım için dikkat edilmesi gereken noktalar.

7. **Best Practices**: `ref` ve `in` kullanımı için en iyi uygulamalar.

Bu anahtar kelimeler, özellikle performans kritik uygulamalarda ve büyük değer türleriyle çalışırken bellek kullanımını optimize etmek için güçlü araçlardır. 