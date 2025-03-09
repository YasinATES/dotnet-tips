# Operatörler ve İfadeler

C# dilinde operatörler, değişkenler ve sabitler üzerinde işlem yapmak için kullanılan sembollerdir. İfadeler ise operatörlerin, değişkenlerin ve sabitlerin birleşiminden oluşan kod parçalarıdır. Bu bölümde, C#'taki farklı operatör türlerini ve bunların kullanımlarını detaylı olarak inceleyeceğiz.

## 1. Aritmetik Operatörler

Aritmetik operatörler, matematiksel işlemler gerçekleştirmek için kullanılır.

| Operatör | Açıklama | Örnek |
|----------|----------|-------|
| + | Toplama | a + b |
| - | Çıkarma | a - b |
| * | Çarpma | a * b |
| / | Bölme | a / b |
| % | Mod (Kalan) | a % b |
| ++ | Artırma | a++ veya ++a |
| -- | Azaltma | a-- veya --a |

```csharp
int a = 10;
int b = 3;

int toplam = a + b;      // 13
int fark = a - b;        // 7
int carpim = a * b;      // 30
int bolum = a / b;       // 3 (int bölmesi, ondalık kısmı atar)
int kalan = a % b;       // 1

// Artırma ve azaltma operatörleri
int c = 5;
int d = c++;             // d = 5, c = 6 (önce atama, sonra artırma)
int e = ++c;             // c = 7, e = 7 (önce artırma, sonra atama)

int f = 5;
int g = f--;             // g = 5, f = 4 (önce atama, sonra azaltma)
int h = --f;             // f = 3, h = 3 (önce azaltma, sonra atama)
```

Dikkat edilmesi gereken noktalar:
- İnt bölmesi ondalık kısmı atar. Ondalıklı sonuç için en az bir operandın float, double veya decimal olması gerekir.
- Mod operatörü (%) bölme işleminden kalan değeri verir.
- Artırma (++) ve azaltma (--) operatörleri, değişkenin değerini 1 artırır veya azaltır. Operatörün konumu (önek veya sonek) işlem sırasını etkiler.

## 2. Karşılaştırma Operatörleri

Karşılaştırma operatörleri, iki değeri karşılaştırmak için kullanılır ve her zaman boolean (true/false) değer döndürür.

| Operatör | Açıklama | Örnek |
|----------|----------|-------|
| == | Eşittir | a == b |
| != | Eşit değildir | a != b |
| < | Küçüktür | a < b |
| > | Büyüktür | a > b |
| <= | Küçük veya eşittir | a <= b |
| >= | Büyük veya eşittir | a >= b |

```csharp
int a = 10;
int b = 20;

bool esitMi = (a == b);          // false
bool esitDegilMi = (a != b);     // true
bool kucukMu = (a < b);          // true
bool buyukMu = (a > b);          // false
bool kucukEsitMi = (a <= b);     // true
bool buyukEsitMi = (a >= b);     // false

// Referans tipleri için karşılaştırma
string str1 = "Hello";
string str2 = "Hello";
string str3 = new string(new char[] { 'H', 'e', 'l', 'l', 'o' });

bool stringsEqual = (str1 == str2);      // true - değer karşılaştırması
bool stringsEqual2 = (str1 == str3);     // true - string için == operatörü içeriği karşılaştırır
bool refsEqual = ReferenceEquals(str1, str3); // false - farklı referanslar
```

Dikkat edilmesi gereken noktalar:
- Referans tipleri için `==` operatörü, varsayılan olarak referansları karşılaştırır (aynı nesneyi işaret edip etmediklerini).
- String sınıfı için `==` operatörü özel olarak aşırı yüklenmiştir ve içerik karşılaştırması yapar.
- Özel sınıflar için `==` operatörünü aşırı yükleyebilirsiniz.

## 3. Mantıksal Operatörler

Mantıksal operatörler, boolean değerler üzerinde işlem yapmak için kullanılır.

| Operatör | Açıklama | Örnek |
|----------|----------|-------|
| && | Mantıksal VE | a && b |
| \|\| | Mantıksal VEYA | a \|\| b |
| ! | Mantıksal DEĞİL | !a |

```csharp
bool a = true;
bool b = false;

bool vesonucu = a && b;      // false (her iki operand da true olmalı)
bool veyasonucu = a || b;    // true (en az bir operand true olmalı)
bool degilsonucu = !a;       // false (a'nın değilini alır)

// Kısa devre değerlendirmesi
bool result = (GetValue() > 0) && ProcessValue(); // ProcessValue() sadece GetValue() > 0 ise çağrılır

// Bitwise operatörler ile karşılaştırma
bool bitwiseAnd = a & b;     // false (kısa devre değerlendirmesi yapmaz)
bool bitwiseOr = a | b;      // true (kısa devre değerlendirmesi yapmaz)
```

Dikkat edilmesi gereken noktalar:
- `&&` ve `||` operatörleri kısa devre değerlendirmesi yapar. Yani, sonuç ilk operanddan belirlenebiliyorsa, ikinci operand değerlendirilmez.
- `&` ve `|` operatörleri boolean değerler üzerinde kullanıldığında, her iki operandı da değerlendirir (kısa devre değerlendirmesi yapmaz).

## 4. Bitwise Operatörler

Bitwise operatörler, bit düzeyinde işlemler gerçekleştirmek için kullanılır.

| Operatör | Açıklama | Örnek |
|----------|----------|-------|
| & | Bitwise AND | a & b |
| \| | Bitwise OR | a \| b |
| ^ | Bitwise XOR | a ^ b |
| ~ | Bitwise NOT | ~a |
| << | Sola kaydırma | a << n |
| >> | Sağa kaydırma | a >> n |

```csharp
int a = 5;     // 0101 (ikili)
int b = 3;     // 0011 (ikili)

int bitwiseAnd = a & b;      // 0001 = 1 (her iki bit de 1 ise 1)
int bitwiseOr = a | b;       // 0111 = 7 (en az bir bit 1 ise 1)
int bitwiseXor = a ^ b;      // 0110 = 6 (bitler farklı ise 1)
int bitwiseNot = ~a;         // 1010 = -6 (tüm bitleri tersine çevirir)

int leftShift = a << 1;      // 1010 = 10 (bitleri 1 pozisyon sola kaydırır)
int rightShift = a >> 1;     // 0010 = 2 (bitleri 1 pozisyon sağa kaydırır)

// Bitwise operatörlerin kullanım örnekleri
// Bayrak (flag) değerlerini işlemek
[Flags]
enum Permissions
{
    None = 0,        // 0000
    Read = 1,        // 0001
    Write = 2,       // 0010
    Execute = 4,     // 0100
    All = Read | Write | Execute // 0111
}

Permissions userPermissions = Permissions.Read | Permissions.Write; // 0011
bool canRead = (userPermissions & Permissions.Read) == Permissions.Read; // true
bool canExecute = (userPermissions & Permissions.Execute) == Permissions.Execute; // false
```

Dikkat edilmesi gereken noktalar:
- Bitwise operatörler genellikle bayrak (flag) değerlerini işlemek için kullanılır.
- Kaydırma operatörleri (`<<` ve `>>`) bitleri belirtilen sayıda pozisyon kaydırır. Sola kaydırma, değeri 2^n ile çarpar; sağa kaydırma, değeri 2^n'e böler.
- Bitwise NOT (`~`) operatörü, tüm bitleri tersine çevirir (0'ları 1, 1'leri 0 yapar).

## 5. Ternary Operatör (?:)

Ternary operatör, bir koşula bağlı olarak iki değerden birini seçmek için kullanılan kısa bir if-else ifadesidir.

```csharp
// Syntax: koşul ? doğruysa_değer : yanlışsa_değer

int a = 10;
int b = 20;

int buyukOlan = (a > b) ? a : b;  // b (20) döner çünkü a > b yanlış

string durum = (a > b) ? "a büyüktür" : "b büyüktür veya eşittir";  // "b büyüktür veya eşittir"

// İç içe ternary operatörler (okunabilirlik için dikkatli kullanın)
int c = 30;
int enBuyuk = (a > b) ? ((a > c) ? a : c) : ((b > c) ? b : c);  // c (30)
```

Dikkat edilmesi gereken noktalar:
- Ternary operatör, basit if-else ifadelerini daha kısa yazmak için kullanışlıdır.
- İç içe ternary operatörlerden kaçının, kodun okunabilirliğini azaltır.
- Her iki sonuç ifadesi de aynı tipe sahip olmalı veya otomatik tür dönüşümü yapılabilmelidir.

## 6. Null-Coalescing Operatör (??)

Null-coalescing operatör, bir değer null ise alternatif bir değer döndürmek için kullanılır.

```csharp
// Syntax: değer ?? alternatif_değer

string name = null;
string displayName = name ?? "Misafir";  // "Misafir" döner çünkü name null

// Zincirleme kullanım
string firstName = null;
string middleName = null;
string lastName = "Doe";
string fullName = firstName ?? middleName ?? lastName ?? "Anonim";  // "Doe" döner

// Null-coalescing atama operatörü (C# 8.0+)
string userName = null;
userName ??= "DefaultUser";  // userName null ise "DefaultUser" atar
```

Dikkat edilmesi gereken noktalar:
- Null-coalescing operatör sadece sol operand null ise sağ operandı değerlendirir.
- Zincirleme kullanılabilir ve ilk null olmayan değeri döndürür.
- C# 8.0'dan itibaren null-coalescing atama operatörü (`??=`) de kullanılabilir.

## 7. Null-Conditional Operatör (?.)

Null-conditional operatör, bir nesne null ise null döndürmek, değilse belirtilen üyeye erişmek için kullanılır.

```csharp
// Syntax: nesne?.üye

Person person = null;
string name = person?.Name;  // null döner çünkü person null

// Zincirleme kullanım
string city = person?.Address?.City;  // null döner

// Diziler ve indeksleyiciler ile kullanım
int? firstItem = myArray?[0];  // myArray null ise null döner

// Null-conditional ve null-coalescing operatörlerin birlikte kullanımı
string displayName = person?.Name ?? "Anonim";  // "Anonim" döner
```

Dikkat edilmesi gereken noktalar:
- Null-conditional operatör, null referans hatalarını önlemek için çok kullanışlıdır.
- Zincirleme kullanılabilir ve herhangi bir nesne null ise, tüm ifade null döner.
- Değer tipleri için kullanıldığında, sonuç otomatik olarak nullable tipe dönüştürülür.

## 8. is ve as Operatörleri

`is` ve `as` operatörleri, tür kontrolü ve dönüşümü için kullanılır.

### is Operatörü

`is` operatörü, bir nesnenin belirli bir türe ait olup olmadığını kontrol eder.

```csharp
object obj = "Hello";

bool isString = obj is string;  // true
bool isInt = obj is int;        // false

// C# 7.0+ pattern matching
if (obj is string message)
{
    // message değişkeni string tipinde ve obj'nin değerine sahip
    Console.WriteLine(message.Length);  // 5
}

// C# 9.0+ pattern matching geliştirmeleri
if (obj is string { Length: > 3 } longString)
{
    Console.WriteLine($"Uzun string: {longString}");
}
```

### as Operatörü

`as` operatörü, bir nesneyi belirli bir türe dönüştürmeye çalışır. Dönüşüm başarısız olursa null döndürür.

```csharp
object obj = "Hello";

string str = obj as string;  // "Hello"
int? num = obj as int?;      // null (dönüşüm başarısız)

// Güvenli tür dönüşümü
if (obj is string)
{
    string message = obj as string;  // Güvenli, çünkü obj'nin string olduğunu biliyoruz
    Console.WriteLine(message.Length);
}

// as operatörü sadece referans tipleri ve nullable değer tipleri için kullanılabilir
// int value = obj as int;  // Derleme hatası
```

Dikkat edilmesi gereken noktalar:
- `is` operatörü her zaman boolean değer döndürür.
- `as` operatörü başarısız dönüşümlerde exception fırlatmaz, null döndürür.
- `as` operatörü sadece referans tipleri ve nullable değer tipleri için kullanılabilir.
- C# 7.0 ve sonrasında, `is` operatörü ile pattern matching kullanılabilir.

## 9. sizeof ve typeof Operatörleri

### sizeof Operatörü

`sizeof` operatörü, bir değer tipinin bellekte kapladığı byte sayısını döndürür.

```csharp
// sizeof operatörü genellikle unsafe kod bloklarında kullanılır
// veya derleme zamanında bilinen tipler için kullanılabilir

int size1 = sizeof(int);      // 4
int size2 = sizeof(double);   // 8
int size3 = sizeof(char);     // 2
int size4 = sizeof(bool);     // 1

// Özel struct için sizeof kullanımı
unsafe
{
    int size5 = sizeof(MyStruct);  // MyStruct'ın boyutu
}
```

### typeof Operatörü

`typeof` operatörü, bir tipin System.Type nesnesini döndürür.

```csharp
Type stringType = typeof(string);
Type intType = typeof(int);
Type listType = typeof(List<int>);

// Type bilgisini kullanma
Console.WriteLine(stringType.FullName);  // System.String
Console.WriteLine(intType.IsValueType);  // true
Console.WriteLine(listType.IsGenericType);  // true

// Reflection ile kullanım
MethodInfo[] methods = typeof(string).GetMethods();
foreach (var method in methods)
{
    Console.WriteLine(method.Name);
}
```

Dikkat edilmesi gereken noktalar:
- `sizeof` operatörü genellikle unsafe kod bloklarında veya derleme zamanında bilinen tipler için kullanılır.
- `typeof` operatörü, reflection işlemleri için kullanışlıdır ve çalışma zamanında tip bilgisine erişim sağlar.
- `typeof` operatörü, generic tip parametreleri ile de kullanılabilir.

## Operatör Önceliği

C#'ta operatörler belirli bir öncelik sırasına göre değerlendirilir. Aşağıdaki tablo, operatörlerin öncelik sırasını (en yüksekten en düşüğe) gösterir:

1. Birincil operatörler: `x.y`, `x?.y`, `x?[y]`, `f(x)`, `a[i]`, `x++`, `x--`, `new`, `typeof`, `checked`, `unchecked`, `default`, `nameof`, `sizeof`, `stackalloc`
2. Unary operatörler: `+x`, `-x`, `!x`, `~x`, `++x`, `--x`, `(T)x`
3. Çarpımsal operatörler: `x * y`, `x / y`, `x % y`
4. Toplamsal operatörler: `x + y`, `x - y`
5. Kaydırma operatörleri: `x << y`, `x >> y`
6. İlişkisel operatörler: `x < y`, `x > y`, `x <= y`, `x >= y`, `is`, `as`
7. Eşitlik operatörleri: `x == y`, `x != y`
8. Bitwise AND: `x & y`
9. Bitwise XOR: `x ^ y`
10. Bitwise OR: `x | y`
11. Mantıksal AND: `x && y`
12. Mantıksal OR: `x || y`
13. Null-coalescing operatör: `x ?? y`
14. Ternary operatör: `c ? t : f`
15. Atama operatörleri: `x = y`, `x += y`, `x -= y`, `x *= y`, `x /= y`, `x %= y`, `x &= y`, `x |= y`, `x ^= y`, `x <<= y`, `x >>= y`, `x ??= y`

```csharp
// Operatör önceliği örneği
int result = 2 + 3 * 4;  // 14 (çarpma işlemi önce yapılır)
int result2 = (2 + 3) * 4;  // 20 (parantez içi önce değerlendirilir)

// Karmaşık bir örnek
bool complex = 5 > 3 && 10 < 20 || 7 == 8;  // true (5 > 3 && 10 < 20 önce değerlendirilir)
bool complex2 = 5 > 3 && (10 < 20 || 7 == 8);  // true (parantez içi önce değerlendirilir)
```

## En İyi Pratikler

1. **Karmaşık ifadelerde parantez kullanın**
   - Operatör önceliğine güvenmek yerine, parantezler ile ifadenin nasıl değerlendirileceğini açıkça belirtin.
   - Bu, kodun okunabilirliğini artırır ve hataları önler.

2. **Kısa devre değerlendirmesini akıllıca kullanın**
   - `&&` ve `||` operatörlerinin kısa devre davranışını, yan etkileri olan ifadeleri kontrol etmek için kullanın.
   - Örneğin: `if (obj != null && obj.Property > 0)` ifadesinde, obj null değilse Property'ye erişilir.

3. **Ternary operatörü basit durumlar için kullanın**
   - Karmaşık koşullar için if-else ifadelerini tercih edin.
   - İç içe ternary operatörlerden kaçının, kodun okunabilirliğini azaltır.

4. **Null-conditional ve null-coalescing operatörleri birlikte kullanın**
   - `person?.Name ?? "Anonim"` gibi ifadeler, null kontrolü için çok etkilidir.

5. **Bitwise operatörleri bayrak (flag) değerleri için kullanın**
   - Enum flag değerlerini işlemek için bitwise operatörler idealdir.
   - [Flags] özniteliğini kullanmayı unutmayın.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Integer bölme tuzağı**
   ```csharp
   // Kötü - int bölmesi ondalık kısmı atar
   double ratio = 5 / 2;  // 2.0
   
   // İyi - en az bir operandın double olduğundan emin olun
   double ratio = 5.0 / 2;  // 2.5
   // veya
   double ratio = 5 / 2.0;  // 2.5
   // veya
   double ratio = (double)5 / 2;  // 2.5
   ```

2. **Null referans hataları**
   ```csharp
   // Kötü - NullReferenceException riski
   string name = person.Name;
   
   // İyi - null-conditional operatör kullanımı
   string name = person?.Name;
   
   // Daha iyi - varsayılan değer ile
   string name = person?.Name ?? "Anonim";
   ```

3. **Bitwise ve mantıksal operatörleri karıştırmak**
   ```csharp
   // Kötü - & operatörü her iki tarafı da değerlendirir
   if (obj != null & obj.IsValid())  // obj null ise NullReferenceException
   
   // İyi - && operatörü kısa devre değerlendirmesi yapar
   if (obj != null && obj.IsValid())  // obj null ise IsValid() çağrılmaz
   ```

4. **Operatör önceliğini yanlış anlamak**
   ```csharp
   // Kötü - operatör önceliği yanlış anlaşılabilir
   if (a > b || c > d && e > f)  // && operatörü || operatöründen önce değerlendirilir
   
   // İyi - parantezler ile açıkça belirtin
   if ((a > b) || ((c > d) && (e > f)))
   ```

5. **as operatörünü değer tipleri için kullanmaya çalışmak**
   ```csharp
   // Kötü - derleme hatası
   int number = obj as int;
   
   // İyi - nullable değer tipi kullanın
   int? number = obj as int?;
   
   // veya cast kullanın (exception riski var)
   int number = (int)obj;
   ```

Operatörleri ve ifadeleri doğru şekilde kullanmak, C# programlamanın temel becerilerinden biridir. Bu kavramları iyi anlamak, daha verimli, okunabilir ve hatasız kod yazmanıza yardımcı olacaktır. 