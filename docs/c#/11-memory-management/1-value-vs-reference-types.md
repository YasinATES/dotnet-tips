# Value vs Reference Types (Değer ve Referans Türleri)

C#'ta iki temel tür kategorisi vardır: değer türleri (value types) ve referans türleri (reference types). Bu iki kategori arasındaki farkları anlamak, bellek yönetimi, performans ve doğru kod yazımı açısından kritik öneme sahiptir.

## 1. Stack Allocation (Yığın Tahsisi)

Değer türleri genellikle stack (yığın) bellek bölgesinde saklanır.

```csharp
// Stack üzerinde değer türleri
public void StackAllocationExample()
{
    // Değer türleri stack'te saklanır
    int number = 42;
    bool flag = true;
    char character = 'A';
    double price = 29.99;
    
    // Struct'lar da değer türleridir
    DateTime today = DateTime.Now;
    Point2D point = new Point2D(10, 20);
    
    Console.WriteLine($"Sayı: {number}");
    Console.WriteLine($"Bayrak: {flag}");
    Console.WriteLine($"Karakter: {character}");
    Console.WriteLine($"Fiyat: {price}");
    Console.WriteLine($"Bugün: {today}");
    Console.WriteLine($"Nokta: ({point.X}, {point.Y})");
}

// Örnek bir struct
public struct Point2D
{
    public int X { get; }
    public int Y { get; }
    
    public Point2D(int x, int y)
    {
        X = x;
        Y = y;
    }
}
```

### Stack'in Özellikleri

Stack bellek bölgesi, LIFO (Last In, First Out - Son Giren, İlk Çıkar) prensibiyle çalışır ve genellikle daha hızlı erişim sağlar.

```csharp
// Stack'in özellikleri
public void StackProperties()
{
    // Stack bellek, metot çağrıları ve yerel değişkenler için kullanılır
    void NestedMethod()
    {
        int localVar = 100; // Stack'te saklanır
        Console.WriteLine($"Yerel değişken: {localVar}");
    }
    
    int mainVar = 42; // Stack'te saklanır
    Console.WriteLine($"Ana değişken: {mainVar}");
    
    NestedMethod(); // Yeni bir stack frame oluşturulur
    
    // NestedMethod tamamlandığında, onun stack frame'i temizlenir
    // localVar artık erişilemez
}
```

## 2. Heap Allocation (Öbek Tahsisi)

Referans türleri heap (öbek) bellek bölgesinde saklanır.

```csharp
// Heap üzerinde referans türleri
public void HeapAllocationExample()
{
    // Referans türleri heap'te saklanır
    string message = "Merhaba, Dünya!"; // String bir referans türüdür
    
    // Sınıf nesneleri heap'te saklanır
    Person person = new Person("Ahmet", 30);
    List<int> numbers = new List<int> { 1, 2, 3, 4, 5 };
    
    Console.WriteLine($"Mesaj: {message}");
    Console.WriteLine($"Kişi: {person.Name}, {person.Age} yaşında");
    Console.WriteLine($"Sayılar: {string.Join(", ", numbers)}");
}

// Örnek bir sınıf
public class Person
{
    public string Name { get; }
    public int Age { get; }
    
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
}
```

### Heap'in Özellikleri

Heap bellek bölgesi, dinamik bellek tahsisi için kullanılır ve Garbage Collector (Çöp Toplayıcı) tarafından yönetilir.

```csharp
// Heap'in özellikleri
public void HeapProperties()
{
    // Heap bellek, dinamik olarak oluşturulan nesneler için kullanılır
    var largeObject = new byte[1000000]; // Büyük dizi heap'te saklanır
    
    // Referans türleri heap'te saklanır, ancak referansın kendisi stack'tedir
    Person person = new Person("Mehmet", 25);
    
    // person değişkeni stack'te, Person nesnesi heap'tedir
    
    // Garbage Collector, erişilemeyen nesneleri temizler
    person = null; // Artık Person nesnesine referans yok
    
    // Bir sonraki GC çalıştığında, Person nesnesi temizlenebilir
}
```

## 3. Memory Layout (Bellek Düzeni)

Değer ve referans türlerinin bellek düzenleri farklıdır.

```csharp
// Bellek düzeni
public void MemoryLayoutExample()
{
    // Değer türü - verinin kendisi stack'te saklanır
    int value = 42;
    
    // Referans türü - referans stack'te, nesne heap'te saklanır
    string text = "Örnek metin";
    
    // Değer türü içinde değer türü - tümü stack'te
    Point2D point = new Point2D(10, 20);
    
    // Referans türü içinde değer türü - değer türü, referans türünün içinde heap'te saklanır
    Person person = new Person("Ali", 35); // Age (int) değer türüdür, ancak Person nesnesi içinde heap'tedir
    
    // Değer türü içinde referans türü - referans stack'te, referans edilen nesne heap'te
    ValueWithReference container = new ValueWithReference();
    container.Text = "Değer türü içindeki referans";
}

// Referans türü içeren değer türü
public struct ValueWithReference
{
    public string Text; // String bir referans türüdür
}
```

### İç İçe Türler

İç içe türlerin bellek düzeni, içerdikleri türlere bağlıdır.

```csharp
// İç içe türler
public void NestedTypesExample()
{
    // Değer türü içinde değer türü dizisi
    StructWithArray structWithArray = new StructWithArray();
    structWithArray.Numbers = new int[3] { 1, 2, 3 }; // Dizi heap'te, referans struct içinde
    
    // Referans türü içinde değer türü dizisi
    ClassWithValueTypes classWithValues = new ClassWithValueTypes();
    classWithValues.Points = new Point2D[2]
    {
        new Point2D(1, 2),
        new Point2D(3, 4)
    };
    
    Console.WriteLine($"Struct içindeki dizi: {string.Join(", ", structWithArray.Numbers)}");
    Console.WriteLine($"Class içindeki noktalar: ({classWithValues.Points[0].X}, {classWithValues.Points[0].Y}), ({classWithValues.Points[1].X}, {classWithValues.Points[1].Y})");
}

// Dizi içeren struct
public struct StructWithArray
{
    public int[] Numbers; // Dizi bir referans türüdür
}

// Değer türleri içeren sınıf
public class ClassWithValueTypes
{
    public Point2D[] Points; // Point2D bir değer türüdür, ancak dizi bir referans türüdür
}
```

## 4. Performance Implications (Performans Etkileri)

Değer ve referans türleri arasındaki seçim, performansı etkileyebilir.

```csharp
// Performans etkileri
public void PerformanceImplicationsExample()
{
    const int iterations = 10000000;
    
    // Değer türü performans testi
    var stopwatch1 = Stopwatch.StartNew();
    
    for (int i = 0; i < iterations; i++)
    {
        Point2D point = new Point2D(i, i);
        int sum = point.X + point.Y;
    }
    
    stopwatch1.Stop();
    
    // Referans türü performans testi
    var stopwatch2 = Stopwatch.StartNew();
    
    for (int i = 0; i < iterations; i++)
    {
        PointClass point = new PointClass(i, i);
        int sum = point.X + point.Y;
    }
    
    stopwatch2.Stop();
    
    Console.WriteLine($"Değer türü süresi: {stopwatch1.ElapsedMilliseconds} ms");
    Console.WriteLine($"Referans türü süresi: {stopwatch2.ElapsedMilliseconds} ms");
}

// Karşılaştırma için referans türü
public class PointClass
{
    public int X { get; }
    public int Y { get; }
    
    public PointClass(int x, int y)
    {
        X = x;
        Y = y;
    }
}
```

### Bellek Kullanımı

Değer ve referans türleri, bellek kullanımı açısından da farklılık gösterir.

```csharp
// Bellek kullanımı
public void MemoryUsageExample()
{
    const int count = 1000000;
    
    // GC'yi zorla ve başlangıç bellek kullanımını ölç
    GC.Collect();
    long memoryBefore = GC.GetTotalMemory(true);
    
    // Değer türü dizisi
    Point2D[] valueArray = new Point2D[count];
    for (int i = 0; i < count; i++)
    {
        valueArray[i] = new Point2D(i, i);
    }
    
    // Bellek kullanımını ölç
    long memoryAfterValue = GC.GetTotalMemory(false);
    
    // Referans türü dizisi
    PointClass[] referenceArray = new PointClass[count];
    for (int i = 0; i < count; i++)
    {
        referenceArray[i] = new PointClass(i, i);
    }
    
    // Bellek kullanımını ölç
    long memoryAfterReference = GC.GetTotalMemory(false);
    
    long valueTypeMemory = memoryAfterValue - memoryBefore;
    long referenceTypeMemory = memoryAfterReference - memoryAfterValue;
    
    Console.WriteLine($"Değer türü bellek kullanımı: {valueTypeMemory / 1024 / 1024} MB");
    Console.WriteLine($"Referans türü bellek kullanımı: {referenceTypeMemory / 1024 / 1024} MB");
}
```

## 5. Parameter Passing (Parametre Geçişi)

Değer ve referans türleri, metotlara parametre olarak geçirilirken farklı davranır.

```csharp
// Parametre geçişi
public void ParameterPassingExample()
{
    // Değer türü
    int number = 10;
    Console.WriteLine($"Değiştirmeden önce: {number}");
    
    ModifyValue(number);
    Console.WriteLine($"Değiştirdikten sonra: {number}"); // 10 (değişmedi)
    
    // Referans türü
    Person person = new Person("Ayşe", 28);
    Console.WriteLine($"Değiştirmeden önce: {person.Name}, {person.Age}");
    
    ModifyReference(person);
    Console.WriteLine($"Değiştirdikten sonra: {person.Name}, {person.Age}"); // Değişebilir
}

// Değer türü parametresi - kopya geçirilir
private void ModifyValue(int x)
{
    x = 20; // Orijinal değeri etkilemez
}

// Referans türü parametresi - referans geçirilir
private void ModifyReference(Person p)
{
    // Orijinal nesneyi değiştiremez (Person sınıfı immutable olduğu için)
    // Ancak mutable bir sınıf olsaydı, değişiklikler orijinal nesneyi etkilerdi
    
    // Örnek olarak, mutable bir sınıf:
    // p.Name = "Yeni İsim"; // Orijinal nesneyi etkilerdi
}
```

### Ref ve Out Parametreler

Ref ve out anahtar kelimeleri, değer türlerinin referans olarak geçirilmesini sağlar.

```csharp
// Ref ve out parametreler
public void RefAndOutExample()
{
    // Ref parametresi - mevcut bir değişken geçirilmelidir
    int x = 10;
    Console.WriteLine($"Ref öncesi: {x}");
    
    ModifyWithRef(ref x);
    Console.WriteLine($"Ref sonrası: {x}"); // 20
    
    // Out parametresi - başlangıç değeri gerekli değildir
    int y;
    ModifyWithOut(out y);
    Console.WriteLine($"Out sonrası: {y}"); // 30
    
    // Ref ve out ile struct
    Point2D point = new Point2D(5, 10);
    Console.WriteLine($"Ref öncesi: ({point.X}, {point.Y})");
    
    // Point2D değiştirilemez olduğu için, yeni bir değer atanır
    ModifyPointWithRef(ref point);
    Console.WriteLine($"Ref sonrası: ({point.X}, {point.Y})"); // (15, 25)
}

private void ModifyWithRef(ref int value)
{
    value = 20; // Orijinal değeri değiştirir
}

private void ModifyWithOut(out int value)
{
    // Out parametresi, metot içinde değer atanmalıdır
    value = 30;
}

private void ModifyPointWithRef(ref Point2D point)
{
    // Yeni bir Point2D nesnesi oluşturulur ve referansa atanır
    point = new Point2D(15, 25);
}
```

## 6. Assignment Semantics (Atama Semantiği)

Değer ve referans türleri, atama işlemlerinde farklı davranır.

```csharp
// Atama semantiği
public void AssignmentSemanticsExample()
{
    // Değer türü atama - değerin kopyası oluşturulur
    int a = 10;
    int b = a; // b, a'nın bir kopyasıdır
    
    b = 20; // a etkilenmez
    
    Console.WriteLine($"a: {a}, b: {b}"); // a: 10, b: 20
    
    // Referans türü atama - referans kopyalanır
    Person person1 = new Person("Zeynep", 32);
    Person person2 = person1; // person2, person1 ile aynı nesneyi referans eder
    
    // person2 = new Person("Yeni Kişi", 40); // Bu, person1'i etkilemez
    
    Console.WriteLine($"person1: {person1.Name}, {person1.Age}");
    Console.WriteLine($"person2: {person2.Name}, {person2.Age}");
    
    // Struct atama - değerin kopyası oluşturulur
    Point2D point1 = new Point2D(10, 20);
    Point2D point2 = point1; // point2, point1'in bir kopyasıdır
    
    // point2 = new Point2D(30, 40); // point1 etkilenmez
    
    Console.WriteLine($"point1: ({point1.X}, {point1.Y})");
    Console.WriteLine($"point2: ({point2.X}, {point2.Y})");
}
```

### Derin ve Sığ Kopyalama

Referans türleri için derin ve sığ kopyalama kavramları önemlidir.

```csharp
// Derin ve sığ kopyalama
public void DeepAndShallowCopyExample()
{
    // Sığ kopyalama - iç referanslar paylaşılır
    PersonWithFriends original = new PersonWithFriends("Ali", 30);
    original.Friends.Add(new Person("Veli", 28));
    original.Friends.Add(new Person("Ayşe", 32));
    
    // Sığ kopya
    PersonWithFriends shallowCopy = original.ShallowCopy();
    
    // Derin kopya
    PersonWithFriends deepCopy = original.DeepCopy();
    
    // Orijinal nesneyi değiştirme
    original.Friends.Add(new Person("Fatma", 35));
    
    // Sığ kopya etkilenir, derin kopya etkilenmez
    Console.WriteLine($"Orijinal arkadaş sayısı: {original.Friends.Count}"); // 3
    Console.WriteLine($"Sığ kopya arkadaş sayısı: {shallowCopy.Friends.Count}"); // 3
    Console.WriteLine($"Derin kopya arkadaş sayısı: {deepCopy.Friends.Count}"); // 2
}

// Arkadaşları olan kişi sınıfı
public class PersonWithFriends
{
    public string Name { get; }
    public int Age { get; }
    public List<Person> Friends { get; }
    
    public PersonWithFriends(string name, int age)
    {
        Name = name;
        Age = age;
        Friends = new List<Person>();
    }
    
    // Sığ kopya
    public PersonWithFriends ShallowCopy()
    {
        PersonWithFriends copy = new PersonWithFriends(Name, Age);
        copy.Friends.AddRange(Friends); // Aynı Person nesnelerini referans eder
        return copy;
    }
    
    // Derin kopya
    public PersonWithFriends DeepCopy()
    {
        PersonWithFriends copy = new PersonWithFriends(Name, Age);
        
        // Her arkadaşın yeni bir kopyasını oluştur
        foreach (Person friend in Friends)
        {
            copy.Friends.Add(new Person(friend.Name, friend.Age));
        }
        
        return copy;
    }
}
```

## Gerçek Dünya Uygulamaları

Değer ve referans türlerinin gerçek dünya uygulamalarında nasıl kullanıldığına dair örnekler.

```csharp
// Finansal hesaplama örneği
public void FinancialCalculationExample()
{
    // Değer türleri, finansal hesaplamalar için idealdir
    decimal accountBalance = 1000.00m;
    decimal interestRate = 0.05m;
    
    // Faiz hesaplama
    decimal CalculateInterest(decimal balance, decimal rate)
    {
        return balance * rate;
    }
    
    // Yıllık faiz hesaplama
    decimal annualInterest = CalculateInterest(accountBalance, interestRate);
    
    // Hesap güncellemesi
    accountBalance += annualInterest;
    
    Console.WriteLine($"Yeni bakiye: {accountBalance:C}");
    
    // Değer türleri, parasal değerler için kesinlik sağlar
    // Referans türleri, karmaşık hesap bilgilerini modellemek için kullanılabilir
    BankAccount account = new BankAccount("123456789", accountBalance);
    account.AddTransaction(new Transaction("Faiz", annualInterest, DateTime.Now));
    
    Console.WriteLine($"Hesap: {account.AccountNumber}, Bakiye: {account.Balance:C}");
    Console.WriteLine($"İşlem sayısı: {account.Transactions.Count}");
}

// Banka hesabı sınıfı
public class BankAccount
{
    public string AccountNumber { get; }
    public decimal Balance { get; private set; }
    public List<Transaction> Transactions { get; }
    
    public BankAccount(string accountNumber, decimal initialBalance)
    {
        AccountNumber = accountNumber;
        Balance = initialBalance;
        Transactions = new List<Transaction>();
    }
    
    public void AddTransaction(Transaction transaction)
    {
        Transactions.Add(transaction);
        Balance += transaction.Amount;
    }
}

// İşlem sınıfı
public class Transaction
{
    public string Description { get; }
    public decimal Amount { get; }
    public DateTime Date { get; }
    
    public Transaction(string description, decimal amount, DateTime date)
    {
        Description = description;
        Amount = amount;
        Date = date;
    }
}
```

### Oyun Geliştirme Örneği

Oyun geliştirmede, değer ve referans türleri farklı amaçlar için kullanılır.

```csharp
// Oyun geliştirme örneği
public void GameDevelopmentExample()
{
    // Değer türleri, oyun nesnelerinin pozisyonları için idealdir
    Vector2D playerPosition = new Vector2D(100, 200);
    
    // Oyuncu hareketi
    void MovePlayer(Vector2D currentPosition, Vector2D movement)
    {
        return new Vector2D(
            currentPosition.X + movement.X,
            currentPosition.Y + movement.Y
        );
    }
    
    // Oyuncuyu hareket ettirme
    Vector2D movement = new Vector2D(5, 10);
    playerPosition = MovePlayer(playerPosition, movement);
    
    Console.WriteLine($"Oyuncu pozisyonu: ({playerPosition.X}, {playerPosition.Y})");
    
    // Referans türleri, oyun nesnelerini modellemek için kullanılır
    GameObject player = new GameObject("Player", playerPosition);
    GameObject enemy = new GameObject("Enemy", new Vector2D(150, 250));
    
    // Çarpışma kontrolü
    bool isColliding = CheckCollision(player, enemy, 50);
    
    Console.WriteLine($"Çarpışma durumu: {isColliding}");
}

// 2D vektör struct'ı
public struct Vector2D
{
    public float X { get; }
    public float Y { get; }
    
    public Vector2D(float x, float y)
    {
        X = x;
        Y = y;
    }
    
    // İki vektör arasındaki mesafeyi hesaplama
    public static float Distance(Vector2D a, Vector2D b)
    {
        float dx = a.X - b.X;
        float dy = a.Y - b.Y;
        return (float)Math.Sqrt(dx * dx + dy * dy);
    }
}

// Oyun nesnesi sınıfı
public class GameObject
{
    public string Name { get; }
    public Vector2D Position { get; set; }
    
    public GameObject(string name, Vector2D position)
    {
        Name = name;
        Position = position;
    }
}

// Çarpışma kontrolü
private bool CheckCollision(GameObject a, GameObject b, float threshold)
{
    float distance = Vector2D.Distance(a.Position, b.Position);
    return distance < threshold;
}
```

## Özet

Bu bölümde, C#'taki değer ve referans türleri arasındaki temel farkları inceledik:

1. **Stack Allocation**: Değer türleri genellikle stack bellek bölgesinde saklanır ve daha hızlı erişim sağlar.

2. **Heap Allocation**: Referans türleri heap bellek bölgesinde saklanır ve Garbage Collector tarafından yönetilir.

3. **Memory Layout**: Değer ve referans türlerinin bellek düzenleri farklıdır ve iç içe türler karmaşık bellek yapıları oluşturabilir.

4. **Performance Implications**: Tür seçimi, uygulamanızın performansını ve bellek kullanımını etkileyebilir.

5. **Parameter Passing**: Değer türleri varsayılan olarak kopyalanarak, referans türleri ise referans olarak geçirilir.

6. **Assignment Semantics**: Atama işlemleri, değer ve referans türleri için farklı davranır ve derin/sığ kopyalama kavramları önemlidir.

Doğru tür seçimi, uygulamanızın gereksinimlerine, performans hedeflerine ve bellek kullanımı kısıtlamalarına bağlıdır. Her iki tür kategorisinin de avantajları ve dezavantajları vardır, bu nedenle duruma göre en uygun seçimi yapmak önemlidir. 