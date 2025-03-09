# Veri Tipleri ve Değişkenler

C#'ta veri tipleri ve değişkenler, programlamanın temel yapı taşlarıdır. Bu bölümde, C#'taki farklı veri tiplerini ve değişken kavramlarını detaylı olarak inceleyeceğiz.

## 1. Değer Tipleri (Value Types)

Değer tipleri, verinin doğrudan bellekte saklandığı tiplerdir. Değer tipleri stack bellek bölgesinde tutulur ve her değişken kendi değerini saklar.

Değer tiplerinin önemli özellikleri:
- Stack bellek bölgesinde saklanırlar
- Bir değişkenden diğerine atama yapıldığında değerin bir kopyası oluşturulur
- Genellikle daha küçük boyutlu veriler için kullanılırlar
- Scope dışına çıktıklarında otomatik olarak bellekten temizlenirler

```csharp
// Tam sayı tipleri
byte b = 255;         // 0 ile 255 arası
short s = 32767;      // -32,768 ile 32,767 arası
int i = 2147483647;   // -2,147,483,648 ile 2,147,483,647 arası
long l = 9223372036854775807; // Çok büyük tam sayılar için

// Ondalıklı sayı tipleri
float f = 3.14f;      // ~7 basamak hassasiyet
double d = 3.14159;   // ~15-16 basamak hassasiyet
decimal m = 3.14159265359m; // ~28-29 basamak hassasiyet

// Diğer değer tipleri
bool isDone = true;   // true veya false
char c = 'A';         // Unicode karakterler
```

Değer tiplerinin kopyalanma davranışı:

```csharp
int a = 10;
int b = a; // a'nın değeri b'ye kopyalanır
b = 20;    // b'nin değeri değişir, a etkilenmez (hala 10)

Console.WriteLine($"a: {a}, b: {b}"); // Çıktı: a: 10, b: 20
```

## 2. Referans Tipleri (Reference Types)

Referans tipleri, heap bellek bölgesinde saklanan ve değişkenin bu veriye referans tuttuğu tiplerdir. Değişken, verinin kendisini değil, verinin bellekteki adresini (referansını) tutar.

Referans tiplerinin önemli özellikleri:
- Heap bellek bölgesinde saklanırlar
- Değişkenler, verilerin kendisini değil referanslarını tutar
- Bir değişkenden diğerine atama yapıldığında referans kopyalanır, veri kopyalanmaz
- Garbage Collector tarafından yönetilirler

```csharp
// String (immutable)
string name = "John Doe";

// Object (tüm tiplerin base sınıfı)
object obj = new object();

// Array (dizi)
int[] numbers = new int[5] { 1, 2, 3, 4, 5 };

// Class örneği
Person person = new Person();
```

Referans tiplerinin paylaşılan referans davranışı:

```csharp
List<int> list1 = new List<int> { 1, 2, 3 };
List<int> list2 = list1; // list1'in referansı list2'ye kopyalanır
list2.Add(4);            // list2 üzerinden yapılan değişiklik list1'i de etkiler

Console.WriteLine($"list1 eleman sayısı: {list1.Count}"); // Çıktı: list1 eleman sayısı: 4
Console.WriteLine($"list2 eleman sayısı: {list2.Count}"); // Çıktı: list2 eleman sayısı: 4
```

## 3. Nullable Tipler ve ? Operatörü

Değer tiplerinin null değer alabilmesini sağlar. Normal şartlarda değer tipleri null olamaz, ancak `?` operatörü ile nullable hale getirilebilirler.

Nullable tipler, iki parçadan oluşur:
1. Değerin kendisi
2. Değerin null olup olmadığını belirten bir flag

Bu yapı, veritabanı işlemlerinde veya opsiyonel değerlerin temsil edilmesinde çok kullanışlıdır.

```csharp
// Nullable tipler
int? nullableInt = null;
bool? nullableBool = null;

// Null kontrolü
if (nullableInt.HasValue)
{
    Console.WriteLine($"Değer: {nullableInt.Value}");
}
else
{
    Console.WriteLine("Değer null");
}

// Null coalescing operator
int result = nullableInt ?? 0; // nullableInt null ise 0 kullanılır
Console.WriteLine($"Sonuç: {result}"); // Çıktı: Sonuç: 0
```

## 4. var Keyword'ü ve Tip Çıkarımı

Derleyici tarafından tipin otomatik belirlenmesini sağlar. `var` anahtar kelimesi, değişkenin tipini ilk atanan değere göre belirler.

`var` kullanımının avantajları:
- Uzun tip isimlerini yazmaktan kurtarır
- Generic tiplerde okunabilirliği artırır
- LINQ sorgularında anonim tipleri kullanmayı kolaylaştırır

Önemli: `var` ile tanımlanan değişkenler hala statik olarak tiplendirilir, dinamik değildir.

```csharp
// Tip çıkarımı
var number = 42;                // int olarak çıkarım yapar
var text = "Hello";             // string olarak çıkarım yapar
var items = new List<int>();    // List<int> olarak çıkarım yapar
var dict = new Dictionary<string, List<Customer>>(); // Karmaşık tipler için idealdir

// YANLIŞ Kullanım
// var x;                       // Hata: İlk değer ataması yapılmalı
// var y = null;                // Hata: Tip belirlenemiyor
```

## 5. const vs readonly

Sabit değerleri tanımlamak için iki farklı yaklaşım sunar, ancak önemli farkları vardır.

**const:**
- Derleme zamanında değeri belirlenir
- Sadece primitive tipler için kullanılabilir (int, string, vb.)
- Değer, IL koduna gömülür
- Static olarak davranır

**readonly:**
- Çalışma zamanında değeri belirlenebilir
- Herhangi bir tip için kullanılabilir
- Constructor'da değer atanabilir
- Instance veya static olabilir

```csharp
public class Constants
{
    // Compile-time sabiti
    public const double PI = 3.14159;
    public const string APP_NAME = "MyApp";
    
    // Runtime sabiti
    public readonly string ConnectionString;
    public readonly DateTime ApplicationStartTime;
    
    public Constants()
    {
        // readonly değişkenler constructor'da değer alabilir
        ConnectionString = "Server=localhost;Database=test;";
        ApplicationStartTime = DateTime.Now;
    }
}

// Kullanım
Console.WriteLine(Constants.PI); // Doğrudan sınıf üzerinden erişilebilir (static gibi)
var constants = new Constants();
Console.WriteLine(constants.ConnectionString); // Instance üzerinden erişilir
```

## 6. Değişken Kapsamı ve Yaşam Süresi

Değişkenlerin erişilebilir olduğu kod bölgesi ve bellekte ne kadar süre kaldığını belirler.

Değişken kapsamı türleri:
- **Sınıf kapsamı:** Sınıf içinde tanımlanan değişkenler (fields)
- **Metot kapsamı:** Metot içinde tanımlanan değişkenler (local variables)
- **Blok kapsamı:** Bir kod bloğu içinde tanımlanan değişkenler (if, for, vb. içinde)

Değişken yaşam süresi:
- Değer tipleri: Kapsamları sona erdiğinde bellekten silinirler
- Referans tipleri: Referans kalmadığında Garbage Collector tarafından temizlenirler

```csharp
public class ScopeExample
{
    private int classLevel = 1; // Sınıf seviyesi değişken - tüm sınıf içinde erişilebilir
    
    public void Method()
    {
        int methodLevel = 2; // Metot seviyesi değişken - sadece bu metot içinde erişilebilir
        Console.WriteLine(classLevel); // Erişilebilir
        
        {
            int blockLevel = 3; // Blok seviyesi değişken - sadece bu blok içinde erişilebilir
            Console.WriteLine(methodLevel); // Erişilebilir
            Console.WriteLine(blockLevel); // Erişilebilir
        }
        
        // Console.WriteLine(blockLevel); // Hata: blockLevel bu noktada yaşamı sona ermiş
    }
    
    public void AnotherMethod()
    {
        Console.WriteLine(classLevel); // Erişilebilir
        // Console.WriteLine(methodLevel); // Hata: methodLevel bu metotta tanımlı değil
    }
}
```

## 7. Boxing ve Unboxing

Değer tiplerinin referans tiplerine (ve tersi) dönüştürülmesi işlemidir. Bu işlemler performans maliyeti getirir.

**Boxing:** Değer tipinin object tipine (referans tipi) dönüştürülmesi
**Unboxing:** Object tipinin tekrar değer tipine dönüştürülmesi

Bu işlemler sırasında:
1. Boxing: Stack'teki değer, heap'e kopyalanır ve referansı döndürülür
2. Unboxing: Heap'teki değer, stack'e geri kopyalanır

```csharp
// Boxing
int number = 123;
object boxed = number; // Boxing - int değeri heap'e kopyalanır

// Unboxing
int unboxed = (int)boxed; // Unboxing - heap'teki değer stack'e kopyalanır

// Dikkat edilmesi gereken durum
object boxed2 = 123;
// long unboxed2 = (long)boxed2; // InvalidCastException - int, long'a dönüştürülemez

// Boxing/Unboxing performans etkisi
Stopwatch sw = new Stopwatch();
sw.Start();

for (int i = 0; i < 1000000; i++)
{
    object o = i;     // Boxing
    int j = (int)o;   // Unboxing
}

sw.Stop();
Console.WriteLine($"Boxing/Unboxing süresi: {sw.ElapsedMilliseconds}ms");
```

## 8. Struct vs Class

Struct ve Class, veri yapılarını tanımlamak için kullanılan iki farklı tiptir. Struct değer tipi, Class referans tipidir.

**Struct:**
- Değer tipidir (stack'te saklanır)
- Kopyalandığında değerin kendisi kopyalanır
- Genellikle küçük, basit veri yapıları için kullanılır
- Mutable olabilir ama immutable olarak tasarlanması önerilir
- Parametre olarak geçirildiğinde kopyalanır

**Class:**
- Referans tipidir (heap'te saklanır)
- Kopyalandığında referans kopyalanır
- Karmaşık veri yapıları ve davranışlar için kullanılır
- Mutable veya immutable olabilir
- Parametre olarak geçirildiğinde referans geçirilir

```csharp
// Struct örneği (değer tipi)
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
    
    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }
    
    public double Distance => Math.Sqrt(X * X + Y * Y);
}

// Class örneği (referans tipi)
public class Rectangle
{
    public int Width { get; set; }
    public int Height { get; set; }
    
    public Rectangle(int width, int height)
    {
        Width = width;
        Height = height;
    }
    
    public int Area => Width * Height;
}

// Kullanım farklılıkları
Point p1 = new Point(1, 1);
Point p2 = p1; // Değer kopyalanır
p2.X = 2;      // p1.X hala 1'dir

Console.WriteLine($"p1: ({p1.X}, {p1.Y})"); // Çıktı: p1: (1, 1)
Console.WriteLine($"p2: ({p2.X}, {p2.Y})"); // Çıktı: p2: (2, 1)

Rectangle r1 = new Rectangle(1, 1);
Rectangle r2 = r1; // Referans kopyalanır
r2.Width = 2;      // r1.Width de 2 olur

Console.WriteLine($"r1: {r1.Width}x{r1.Height}"); // Çıktı: r1: 2x1
Console.WriteLine($"r2: {r2.Width}x{r2.Height}"); // Çıktı: r2: 2x1
```

## En İyi Pratikler

1. **Değişken isimlendirmelerinde anlamlı ve açıklayıcı isimler kullanın.**
   - Kötü: `var x = 5;`
   - İyi: `var userAge = 5;`

2. **Mümkün olduğunca var yerine açık tip belirtmeyi tercih edin.**
   - Özellikle metodun dönüş tipi açık değilse: `User user = GetUser();`
   - LINQ sorgularında ve anonim tiplerde `var` kullanımı uygundur

3. **Nullable tipleri sadece gerektiğinde kullanın.**
   - Veritabanı işlemlerinde null olabilecek alanlar için
   - Opsiyonel parametreler için
   - Her zaman null kontrolü yapmayı unutmayın

4. **Büyük değer tipleri için struct yerine class kullanmayı tercih edin.**
   - 16 byte'tan büyük yapılar için class daha verimli olabilir
   - Çok sık kopyalanan tipler için struct performans sorunlarına yol açabilir

5. **const ve readonly arasında seçim yaparken derleme zamanı vs çalışma zamanı gereksinimlerini göz önünde bulundurun.**
   - Derleme zamanında bilinen sabitler için `const`
   - Çalışma zamanında belirlenen veya karmaşık tipler için `readonly`

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Boxing/Unboxing işlemlerinin performans etkisini göz ardı etmek**
   ```csharp
   // Kötü - çok sayıda boxing/unboxing
   ArrayList list = new ArrayList();
   for (int i = 0; i < 1000000; i++)
       list.Add(i); // Boxing
   
   // İyi - generic koleksiyon kullanımı
   List<int> list = new List<int>();
   for (int i = 0; i < 1000000; i++)
       list.Add(i); // Boxing yok
   ```

2. **Null kontrolü yapmadan nullable tipleri kullanmak**
   ```csharp
   // Kötü - NullReferenceException riski
   int? nullableInt = GetNullableValue();
   int value = nullableInt.Value;
   
   // İyi - null kontrolü
   int? nullableInt = GetNullableValue();
   int value = nullableInt ?? 0;
   // veya
   if (nullableInt.HasValue)
       value = nullableInt.Value;
   ```

3. **Gereksiz yere geniş veri tipleri kullanmak (int yerine long gibi)**
   ```csharp
   // Kötü - gereksiz bellek kullanımı
   long smallNumber = 100; // 8 byte
   
   // İyi - uygun tip seçimi
   int smallNumber = 100;  // 4 byte
   ```

4. **readonly yerine public field kullanmak**
   ```csharp
   // Kötü - encapsulation ihlali
   public class Config
   {
       public string ConnectionString = "...";
   }
   
   // İyi - readonly ile koruma
   public class Config
   {
       public readonly string ConnectionString = "...";
   }
   
   // Daha iyi - property ile encapsulation
   public class Config
   {
       public string ConnectionString { get; }
       
       public Config(string connectionString)
       {
           ConnectionString = connectionString;
       }
   }
   ```

5. **Değişken kapsamını gereğinden geniş tutmak**
   ```csharp
   // Kötü - gereğinden geniş kapsam
   public void ProcessData()
   {
       int result = 0;
       // ... 100 satır kod ...
       result = Calculate();
       // ... 100 satır kod ...
       Console.WriteLine(result);
   }
   
   // İyi - dar kapsam
   public void ProcessData()
   {
       // ... 100 satır kod ...
       int result = Calculate();
       Console.WriteLine(result);
       // ... 100 satır kod ...
   }
   ```

Bu temel kavramları iyi anlamak, C# ile sağlam uygulamalar geliştirmenin ilk adımıdır. Veri tipleri ve değişkenler, tüm programlama mantığının temelini oluşturur ve doğru kullanımları performans ve kod kalitesi açısından kritik öneme sahiptir.