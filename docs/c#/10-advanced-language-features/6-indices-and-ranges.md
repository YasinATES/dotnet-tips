# Indices ve Ranges (İndeksler ve Aralıklar)

C# 8.0 ile tanıtılan Indices ve Ranges özellikleri, dizilere ve koleksiyonlara erişimi daha kolay ve okunabilir hale getirir. Bu özellikler, özellikle dizi dilimlemesi (array slicing) gibi işlemleri daha temiz bir şekilde ifade etmenize olanak tanır.

## 1. Index Syntax (İndeks Sözdizimi)

`Index` yapısı, bir dizinin veya koleksiyonun belirli bir elemanına erişmek için kullanılır. Sondan indeksleme için `^` operatörü kullanılır.

```csharp
// Temel indeks kullanımı
public void BasicIndexUsage()
{
    string[] names = { "Ali", "Ayşe", "Mehmet", "Zeynep", "Can" };
    
    // Geleneksel indeksleme (0-tabanlı)
    string first = names[0]; // "Ali"
    string last = names[names.Length - 1]; // "Can"
    
    // Index yapısı ile indeksleme
    Index firstIndex = 0;
    string firstWithIndex = names[firstIndex]; // "Ali"
    
    // Sondan indeksleme (^1 son elemanı ifade eder)
    Index lastIndex = ^1;
    string lastWithIndex = names[lastIndex]; // "Can"
    
    // Sondan ikinci eleman
    string secondToLast = names[^2]; // "Zeynep"
    
    Console.WriteLine($"İlk: {first}, Son: {last}");
    Console.WriteLine($"Index ile ilk: {firstWithIndex}, Index ile son: {lastWithIndex}");
    Console.WriteLine($"Sondan ikinci: {secondToLast}");
}
```

### Index Yapısı

`Index` yapısı, bir indeksin baştan mı yoksa sondan mı olduğunu belirten bir değer içerir.

```csharp
// Index yapısının kullanımı
public void IndexStructureUsage()
{
    // Index oluşturma yöntemleri
    Index fromStart = new Index(2, fromEnd: false); // 3. eleman (0-tabanlı)
    Index fromEnd = new Index(2, fromEnd: true);    // Sondan 2. eleman
    
    // Kısa sözdizimi
    Index fromStartShort = 2;  // 3. eleman (0-tabanlı)
    Index fromEndShort = ^2;   // Sondan 2. eleman
    
    string[] names = { "Ali", "Ayşe", "Mehmet", "Zeynep", "Can" };
    
    Console.WriteLine($"Baştan 3. eleman: {names[fromStart]}"); // "Mehmet"
    Console.WriteLine($"Sondan 2. eleman: {names[fromEnd]}");   // "Zeynep"
    
    // Index'in değerini alma
    int value = fromStart.Value;       // 2
    bool isFromEnd = fromStart.IsFromEnd; // false
    
    Console.WriteLine($"Index değeri: {value}, Sondan mı: {isFromEnd}");
}
```

### Değişkenler ve Hesaplamalar ile İndeksleme

İndeksler, değişkenler ve hesaplamalar kullanılarak da oluşturulabilir.

```csharp
// Değişkenler ve hesaplamalar ile indeksleme
public void IndexingWithVariables()
{
    string[] names = { "Ali", "Ayşe", "Mehmet", "Zeynep", "Can" };
    
    // Değişken kullanarak indeksleme
    int position = 2;
    string thirdElement = names[position]; // "Mehmet"
    
    // Hesaplama ile indeksleme
    int offset = 1;
    string secondElement = names[position - offset]; // "Ayşe"
    
    // Sondan indeksleme için değişken kullanma
    int fromEnd = 2;
    string secondToLast = names[^fromEnd]; // "Zeynep"
    
    // Hesaplama ile sondan indeksleme
    string thirdToLast = names[^(fromEnd + offset)]; // "Mehmet"
    
    Console.WriteLine($"3. eleman: {thirdElement}");
    Console.WriteLine($"2. eleman: {secondElement}");
    Console.WriteLine($"Sondan 2. eleman: {secondToLast}");
    Console.WriteLine($"Sondan 3. eleman: {thirdToLast}");
}
```

## 2. Range Syntax (Aralık Sözdizimi)

`Range` yapısı, bir dizinin veya koleksiyonun belirli bir aralığını seçmek için kullanılır. Aralık belirtmek için `..` operatörü kullanılır.

```csharp
// Temel aralık kullanımı
public void BasicRangeUsage()
{
    string[] names = { "Ali", "Ayşe", "Mehmet", "Zeynep", "Can" };
    
    // Aralık oluşturma (başlangıç dahil, bitiş hariç)
    Range firstThree = 0..3;
    string[] subset = names[firstThree]; // { "Ali", "Ayşe", "Mehmet" }
    
    // Doğrudan aralık sözdizimi
    string[] middle = names[1..4]; // { "Ayşe", "Mehmet", "Zeynep" }
    
    // Başlangıç veya bitiş belirtmeden aralık
    string[] fromStart = names[..2];  // { "Ali", "Ayşe" }
    string[] toEnd = names[2..];      // { "Mehmet", "Zeynep", "Can" }
    
    // Tüm dizi
    string[] all = names[..]; // { "Ali", "Ayşe", "Mehmet", "Zeynep", "Can" }
    
    Console.WriteLine($"İlk üç: {string.Join(", ", subset)}");
    Console.WriteLine($"Orta: {string.Join(", ", middle)}");
    Console.WriteLine($"Baştan: {string.Join(", ", fromStart)}");
    Console.WriteLine($"Sona kadar: {string.Join(", ", toEnd)}");
    Console.WriteLine($"Tümü: {string.Join(", ", all)}");
}
```

### Sondan İndeksleme ile Aralıklar

Aralıklar, sondan indeksleme ile de kullanılabilir.

```csharp
// Sondan indeksleme ile aralıklar
public void RangesWithEndIndexing()
{
    string[] names = { "Ali", "Ayşe", "Mehmet", "Zeynep", "Can" };
    
    // Sondan indeksleme ile aralık
    string[] lastTwo = names[^2..]; // { "Zeynep", "Can" }
    
    // Başlangıç ve bitiş için sondan indeksleme
    string[] middleFromEnd = names[^4..^1]; // { "Ayşe", "Mehmet", "Zeynep" }
    
    // Karışık indeksleme
    string[] mixed = names[1..^1]; // { "Ayşe", "Mehmet", "Zeynep" }
    
    Console.WriteLine($"Son iki: {string.Join(", ", lastTwo)}");
    Console.WriteLine($"Sondan orta: {string.Join(", ", middleFromEnd)}");
    Console.WriteLine($"Karışık: {string.Join(", ", mixed)}");
}
```

### Range Yapısı

`Range` yapısı, bir aralığın başlangıç ve bitiş indekslerini içerir.

```csharp
// Range yapısının kullanımı
public void RangeStructureUsage()
{
    // Range oluşturma yöntemleri
    Range firstThree = new Range(new Index(0), new Index(3)); // 0..3
    Range lastTwo = new Range(new Index(2, fromEnd: true), new Index(0, fromEnd: true)); // ^2..^0
    
    // Kısa sözdizimi
    Range firstThreeShort = 0..3;
    Range lastTwoShort = ^2..;
    
    string[] names = { "Ali", "Ayşe", "Mehmet", "Zeynep", "Can" };
    
    Console.WriteLine($"İlk üç: {string.Join(", ", names[firstThree])}");
    Console.WriteLine($"Son iki: {string.Join(", ", names[lastTwo])}");
    
    // Range'in başlangıç ve bitiş değerlerini alma
    Index start = firstThree.Start;
    Index end = firstThree.End;
    
    Console.WriteLine($"Başlangıç: {start.Value} (Sondan: {start.IsFromEnd})");
    Console.WriteLine($"Bitiş: {end.Value} (Sondan: {end.IsFromEnd})");
}
```

## 3. Array Slicing (Dizi Dilimleme)

Aralıklar, dizileri dilimlemek için kullanılabilir, yani bir dizinin belirli bir bölümünü seçebilirsiniz.

```csharp
// Dizi dilimleme
public void ArraySlicing()
{
    int[] numbers = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
    
    // Dizi dilimleme örnekleri
    int[] firstHalf = numbers[..5];     // { 0, 1, 2, 3, 4 }
    int[] secondHalf = numbers[5..];    // { 5, 6, 7, 8, 9 }
    int[] middle = numbers[3..7];       // { 3, 4, 5, 6 }
    int[] lastThree = numbers[^3..];    // { 7, 8, 9 }
    int[] exceptEnds = numbers[1..^1];  // { 1, 2, 3, 4, 5, 6, 7, 8 }
    
    Console.WriteLine($"İlk yarı: {string.Join(", ", firstHalf)}");
    Console.WriteLine($"İkinci yarı: {string.Join(", ", secondHalf)}");
    Console.WriteLine($"Orta: {string.Join(", ", middle)}");
    Console.WriteLine($"Son üç: {string.Join(", ", lastThree)}");
    Console.WriteLine($"Uçlar hariç: {string.Join(", ", exceptEnds)}");
}
```

### Çok Boyutlu Dizilerde Dilimleme

C# 8.0'daki aralıklar, şu anda doğrudan çok boyutlu dizileri dilimlemek için kullanılamaz, ancak yardımcı metotlar yazabilirsiniz.

```csharp
// Çok boyutlu dizilerde dilimleme
public void MultidimensionalArraySlicing()
{
    int[,] matrix = {
        { 1, 2, 3 },
        { 4, 5, 6 },
        { 7, 8, 9 }
    };
    
    // Tek bir satırı dilimlemek
    int[] row = GetRow(matrix, 1); // { 4, 5, 6 }
    
    // Tek bir sütunu dilimlemek
    int[] column = GetColumn(matrix, 2); // { 3, 6, 9 }
    
    Console.WriteLine($"2. satır: {string.Join(", ", row)}");
    Console.WriteLine($"3. sütun: {string.Join(", ", column)}");
}

// Matrisin belirli bir satırını almak için yardımcı metot
private int[] GetRow(int[,] matrix, int rowIndex)
{
    int columns = matrix.GetLength(1);
    int[] row = new int[columns];
    
    for (int i = 0; i < columns; i++)
    {
        row[i] = matrix[rowIndex, i];
    }
    
    return row;
}

// Matrisin belirli bir sütununu almak için yardımcı metot
private int[] GetColumn(int[,] matrix, int columnIndex)
{
    int rows = matrix.GetLength(0);
    int[] column = new int[rows];
    
    for (int i = 0; i < rows; i++)
    {
        column[i] = matrix[i, columnIndex];
    }
    
    return column;
}
```

## 4. Custom Index/Range Support (Özel İndeks/Aralık Desteği)

Kendi sınıflarınıza ve koleksiyonlarınıza indeks ve aralık desteği ekleyebilirsiniz.

```csharp
// Özel indeks ve aralık desteği
public class CustomCollection
{
    private readonly string[] _items;
    
    public CustomCollection(string[] items)
    {
        _items = items;
    }
    
    // İndeks operatörü desteği
    public string this[Index index]
    {
        get
        {
            // Index'i dizi indeksine dönüştürme
            int actualIndex = index.IsFromEnd ? _items.Length - index.Value : index.Value;
            return _items[actualIndex];
        }
    }
    
    // Aralık operatörü desteği
    public string[] this[Range range]
    {
        get
        {
            // Range'i dizi aralığına dönüştürme
            (int offset, int length) = range.GetOffsetAndLength(_items.Length);
            string[] result = new string[length];
            Array.Copy(_items, offset, result, 0, length);
            return result;
        }
    }
}

// Kullanım örneği
public void CustomIndexRangeSupport()
{
    string[] names = { "Ali", "Ayşe", "Mehmet", "Zeynep", "Can" };
    var collection = new CustomCollection(names);
    
    // Özel indeks kullanımı
    string first = collection[0];     // "Ali"
    string last = collection[^1];     // "Can"
    
    // Özel aralık kullanımı
    string[] middle = collection[1..4]; // { "Ayşe", "Mehmet", "Zeynep" }
    
    Console.WriteLine($"İlk: {first}, Son: {last}");
    Console.WriteLine($"Orta: {string.Join(", ", middle)}");
}
```

### Span<T> ile Özel Aralık Desteği

`Span<T>` ve `Memory<T>` türleri, indeks ve aralık operatörlerini destekler ve bellek kopyalamadan dilimleme yapmanıza olanak tanır.

```csharp
// Span<T> ile özel aralık desteği
public void SpanRangeSupport()
{
    int[] numbers = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
    
    // Span oluşturma
    Span<int> span = numbers;
    
    // Span dilimleme
    Span<int> firstHalf = span[..5];     // { 0, 1, 2, 3, 4 }
    Span<int> secondHalf = span[5..];    // { 5, 6, 7, 8, 9 }
    Span<int> middle = span[3..7];       // { 3, 4, 5, 6 }
    
    // Orijinal diziyi değiştirme
    middle[0] = 30; // numbers[3] = 30
    middle[1] = 40; // numbers[4] = 40
    
    Console.WriteLine($"Değiştirilmiş dizi: {string.Join(", ", numbers)}");
    // Çıktı: "Değiştirilmiş dizi: 0, 1, 2, 30, 40, 5, 6, 7, 8, 9"
}
```

## 5. Collection Integration (Koleksiyon Entegrasyonu)

Standart koleksiyonlar, indeks ve aralık operatörlerini farklı şekillerde destekler.

```csharp
// Koleksiyon entegrasyonu
public void CollectionIntegration()
{
    // List<T> ile indeks ve aralık
    List<string> nameList = new List<string> { "Ali", "Ayşe", "Mehmet", "Zeynep", "Can" };
    
    // List<T> indeks desteği
    string firstInList = nameList[0];     // "Ali"
    string lastInList = nameList[^1];     // "Can"
    
    // List<T> doğrudan aralık desteği yoktur, ancak GetRange kullanılabilir
    List<string> middleNames = nameList.GetRange(1, 3); // { "Ayşe", "Mehmet", "Zeynep" }
    
    // String ile indeks ve aralık
    string text = "Merhaba Dünya";
    
    // String indeks desteği
    char firstChar = text[0];     // 'M'
    char lastChar = text[^1];     // 'a'
    
    // String aralık desteği
    string substring = text[8..]; // "Dünya"
    
    Console.WriteLine($"List - İlk: {firstInList}, Son: {lastInList}");
    Console.WriteLine($"List - Orta: {string.Join(", ", middleNames)}");
    Console.WriteLine($"String - İlk: {firstChar}, Son: {lastChar}");
    Console.WriteLine($"String - Alt dize: {substring}");
}
```

### LINQ ile Entegrasyon

LINQ metotları, indeks ve aralık operatörleriyle birlikte kullanılabilir.

```csharp
// LINQ ile entegrasyon
public void LinqIntegration()
{
    int[] numbers = { 0, 1, 2, 3, 4, 5, 6, 7, 8, 9 };
    
    // Aralık ile dilimleme, sonra LINQ
    var evenInFirstHalf = numbers[..5].Where(n => n % 2 == 0);
    
    // LINQ sonuçlarını diziye dönüştürüp dilimleme
    var topThreeSquares = numbers.Select(n => n * n).OrderByDescending(n => n).ToArray()[..3];
    
    Console.WriteLine($"İlk yarıdaki çift sayılar: {string.Join(", ", evenInFirstHalf)}");
    Console.WriteLine($"En büyük üç kare: {string.Join(", ", topThreeSquares)}");
}
```

## 6. Performance Implications (Performans Etkileri)

İndeks ve aralık operatörleri, performans açısından bazı etkilere sahiptir.

```csharp
// Performans etkileri
public void PerformanceImplications()
{
    int[] largeArray = Enumerable.Range(0, 1000000).ToArray();
    
    // Dizi dilimleme yeni bir dizi oluşturur (bellek kopyalama)
    int[] slice = largeArray[1000..2000];
    
    // Span dilimleme bellek kopyalamaz
    Span<int> spanSlice = largeArray.AsSpan()[1000..2000];
    
    // Performans karşılaştırması
    var stopwatch = Stopwatch.StartNew();
    
    // 1000 dizi dilimi oluşturma
    for (int i = 0; i < 1000; i++)
    {
        int[] arraySlice = largeArray[i..(i + 1000)];
        int sum = 0;
        foreach (int n in arraySlice)
        {
            sum += n;
        }
    }
    
    long arrayTime = stopwatch.ElapsedMilliseconds;
    stopwatch.Restart();
    
    // 1000 span dilimi oluşturma
    for (int i = 0; i < 1000; i++)
    {
        Span<int> span = largeArray.AsSpan()[i..(i + 1000)];
        int sum = 0;
        foreach (int n in span)
        {
            sum += n;
        }
    }
    
    long spanTime = stopwatch.ElapsedMilliseconds;
    
    Console.WriteLine($"Dizi dilimleme süresi: {arrayTime} ms");
    Console.WriteLine($"Span dilimleme süresi: {spanTime} ms");
    Console.WriteLine($"Fark: {arrayTime - spanTime} ms");
}
```

### Bellek Kullanımı

Dizi dilimleme, yeni bir dizi oluşturur ve bellek kopyalama gerektirir, bu da bellek kullanımını artırabilir.

```csharp
// Bellek kullanımı
public void MemoryUsage()
{
    int[] largeArray = Enumerable.Range(0, 10000000).ToArray();
    
    // GC'yi zorla ve başlangıç bellek kullanımını ölç
    GC.Collect();
    long memoryBefore = GC.GetTotalMemory(true);
    
    // 100 dizi dilimi oluştur
    int[][] slices = new int[100][];
    for (int i = 0; i < 100; i++)
    {
        slices[i] = largeArray[(i * 100000)..((i + 1) * 100000)];
    }
    
    // Bellek kullanımını ölç
    long memoryAfter = GC.GetTotalMemory(false);
    long memoryUsed = memoryAfter - memoryBefore;
    
    Console.WriteLine($"Dizi dilimleme için kullanılan bellek: {memoryUsed / 1024 / 1024} MB");
    
    // Span kullanımı için bellek ölçümü
    GC.Collect();
    memoryBefore = GC.GetTotalMemory(true);
    
    // 100 span dilimi oluştur (referans olarak)
    Span<int>[] spanSlices = new Span<int>[100];
    for (int i = 0; i < 100; i++)
    {
        spanSlices[i] = largeArray.AsSpan()[(i * 100000)..((i + 1) * 100000)];
    }
    
    memoryAfter = GC.GetTotalMemory(false);
    long spanMemoryUsed = memoryAfter - memoryBefore;
    
    Console.WriteLine($"Span dilimleme için kullanılan bellek: {spanMemoryUsed / 1024 / 1024} MB");
}
```

## Gerçek Dünya Uygulamaları

İndeks ve aralık operatörleri, çeşitli gerçek dünya senaryolarında kullanılabilir.

```csharp
// Metin işleme
public void TextProcessing()
{
    string text = "C# 8.0 introduces indices and ranges for better array slicing.";
    
    // İlk 10 karakter
    string start = text[..10]; // "C# 8.0 int"
    
    // Son 10 karakter
    string end = text[^10..]; // "slicing."
    
    // Belirli bir kelimeyi çıkarma
    int indexOfIndices = text.IndexOf("indices");
    int indexOfAnd = text.IndexOf("and", indexOfIndices);
    string extracted = text[indexOfIndices..indexOfAnd].Trim(); // "indices"
    
    Console.WriteLine($"Başlangıç: \"{start}\"");
    Console.WriteLine($"Son: \"{end}\"");
    Console.WriteLine($"Çıkarılan: \"{extracted}\"");
}

// Veri analizi
public void DataAnalysis()
{
    double[] temperatures = { 22.5, 23.1, 24.0, 25.5, 26.2, 27.0, 26.8, 26.0, 25.5, 24.8, 23.5, 22.0 };
    
    // İlk çeyrek
    double[] firstQuarter = temperatures[..(temperatures.Length / 4)];
    
    // Son çeyrek
    double[] lastQuarter = temperatures[^(temperatures.Length / 4)..];
    
    // Orta yarı
    int quarter = temperatures.Length / 4;
    double[] middleHalf = temperatures[quarter..^quarter];
    
    double firstQuarterAvg = firstQuarter.Average();
    double lastQuarterAvg = lastQuarter.Average();
    double middleHalfAvg = middleHalf.Average();
    
    Console.WriteLine($"İlk çeyrek ortalama: {firstQuarterAvg:F1}°C");
    Console.WriteLine($"Son çeyrek ortalama: {lastQuarterAvg:F1}°C");
    Console.WriteLine($"Orta yarı ortalama: {middleHalfAvg:F1}°C");
}

// Görüntü işleme
public void ImageProcessing()
{
    // 10x10 piksellik bir görüntüyü temsil eden 2D dizi
    int[,] image = new int[10, 10];
    
    // Görüntüyü rastgele değerlerle doldur
    Random random = new Random();
    for (int i = 0; i < 10; i++)
    {
        for (int j = 0; j < 10; j++)
        {
            image[i, j] = random.Next(256); // 0-255 arası gri tonları
        }
    }
    
    // Görüntünün merkezini kırp (4x4)
    int[] centerPixels = GetCenterCrop(image, 4, 4);
    
    // Ortalama hesapla
    double average = centerPixels.Average();
    
    Console.WriteLine($"Merkez bölge ortalama piksel değeri: {average:F1}");
}

// Görüntünün merkez bölgesini kırpma
private int[] GetCenterCrop(int[,] image, int width, int height)
{
    int rows = image.GetLength(0);
    int cols = image.GetLength(1);
    
    int startRow = (rows - height) / 2;
    int startCol = (cols - width) / 2;
    
    int[] result = new int[width * height];
    int index = 0;
    
    for (int i = startRow; i < startRow + height; i++)
    {
        for (int j = startCol; j < startCol + width; j++)
        {
            result[index++] = image[i, j];
        }
    }
    
    return result;
}
```

## Özet

Bu bölümde, C# 8.0 ile tanıtılan Indices ve Ranges özelliklerini inceledik:

1. **Index Syntax**: Dizilerin ve koleksiyonların elemanlarına erişmek için kullanılan `^` operatörü ve `Index` yapısı.

2. **Range Syntax**: Dizilerin ve koleksiyonların belirli bir aralığını seçmek için kullanılan `..` operatörü ve `Range` yapısı.

3. **Array Slicing**: Dizileri dilimlemek ve belirli bir bölümünü seçmek için aralıkların kullanımı.

4. **Custom Index/Range Support**: Kendi sınıflarınıza ve koleksiyonlarınıza indeks ve aralık desteği ekleme.

5. **Collection Integration**: Standart koleksiyonların indeks ve aralık operatörleriyle entegrasyonu.

6. **Performance Implications**: İndeks ve aralık operatörlerinin performans etkileri ve bellek kullanımı.

Indices ve Ranges özellikleri, dizilerle ve koleksiyonlarla çalışmayı daha kolay ve okunabilir hale getirir. Bu özellikler, özellikle dizi dilimleme gibi işlemleri daha temiz bir şekilde ifade etmenize olanak tanır. 