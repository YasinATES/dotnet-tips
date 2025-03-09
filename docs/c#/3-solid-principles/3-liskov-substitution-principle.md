# Liskov Substitution Principle (LSP)

Liskov Substitution Principle (Liskov Yerine Geçme Prensibi), SOLID prensiplerinin üçüncü harfi olan "L"yi temsil eder. Bu prensip, Barbara Liskov tarafından 1987'de tanımlanmıştır ve şöyle ifade edilir: "Bir programda, bir üst sınıfın nesneleri, programın doğruluğunu bozmadan alt sınıfların nesneleriyle değiştirilebilmelidir."

## 1. LSP'nin Temel Kavramı

LSP'nin özü, alt sınıfların üst sınıfların davranışlarını koruması gerektiğidir. Bir alt sınıf, üst sınıfın tüm özelliklerini desteklemeli ve üst sınıfın yerine kullanıldığında beklenmeyen davranışlar sergilememelidir.

### Contract ve Invariants Kavramları

LSP'yi anlamak için iki önemli kavramı bilmek gerekir:

1. **Contract (Sözleşme)**: Bir sınıfın metotlarının ne yapması gerektiğini tanımlayan kurallar bütünüdür. Bu, metotların giriş parametreleri (preconditions) ve çıkış değerleri (postconditions) için beklentileri içerir.

2. **Invariants (Değişmezler)**: Bir sınıfın her zaman doğru olması gereken koşullarıdır. Örneğin, bir yığın (stack) sınıfı için "son eklenen eleman ilk çıkarılır" bir invariant'tır.

LSP'ye göre, alt sınıflar:
- Üst sınıfın precondition'larını güçlendiremez (daha kısıtlayıcı yapamaz)
- Üst sınıfın postcondition'larını zayıflatamazlar (daha az garantili yapamaz)
- Üst sınıfın invariant'larını korumalıdır

### LSP İhlali Örneği: Klasik Dikdörtgen-Kare Problemi

```csharp
// LSP ihlali - Klasik dikdörtgen-kare problemi
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
    
    public int CalculateArea()
    {
        return Width * Height;
    }
}

public class Square : Rectangle
{
    private int _side;
    
    public override int Width
    {
        get { return _side; }
        set { _side = value; }
    }
    
    public override int Height
    {
        get { return _side; }
        set { _side = value; }
    }
}

// Kullanım
public void ResizeRectangle(Rectangle rectangle)
{
    rectangle.Width = 10;
    rectangle.Height = 5;
    
    // Dikdörtgen için beklenen alan: 10 * 5 = 50
    // Ancak kare için: 5 * 5 = 25 (çünkü Width'i değiştirdiğimizde Height da değişir)
    Console.WriteLine($"Beklenen alan: 50, Gerçek alan: {rectangle.CalculateArea()}");
}
```

Bu örnekte, `Square` sınıfı `Rectangle` sınıfından türetilmiştir, ancak bir karenin doğası gereği genişlik ve yükseklik eşit olmalıdır. Bu nedenle, `Square` sınıfı `Width` ve `Height` özelliklerini override ederek, birini değiştirdiğimizde diğerinin de değişmesini sağlar.

Ancak, `ResizeRectangle` metodu bir dikdörtgenin genişliğini ve yüksekliğini ayrı ayrı değiştirmeyi bekler. Eğer bu metoda bir `Square` nesnesi geçirirsek, beklenen davranışı elde edemeyiz. Bu, LSP'nin ihlalidir çünkü `Square` nesnesi, `Rectangle` nesnesinin yerine geçemez.

### LSP Uyumlu Tasarım

```csharp
// LSP uyumlu tasarım - Ortak arayüz kullanımı
public interface IShape
{
    int CalculateArea();
}

public class Rectangle : IShape
{
    public int Width { get; set; }
    public int Height { get; set; }
    
    public int CalculateArea()
    {
        return Width * Height;
    }
}

public class Square : IShape
{
    public int Side { get; set; }
    
    public int CalculateArea()
    {
        return Side * Side;
    }
}

// Kullanım
public void PrintArea(IShape shape)
{
    Console.WriteLine($"Alan: {shape.CalculateArea()}");
}
```

Bu tasarımda, `Rectangle` ve `Square` sınıfları ortak bir `IShape` arayüzünü uygular, ancak birbirlerinden türetilmezler. Her biri kendi doğasına uygun şekilde davranır ve LSP ihlali ortadan kalkar.

## 2. Gerçek Hayat Örneği: Banka Hesapları

Banka hesapları, LSP'yi açıklamak için iyi bir örnektir. Farklı hesap türleri farklı davranışlara sahip olabilir, ancak temel hesap işlemleri tüm hesap türleri için tutarlı olmalıdır.

### LSP İhlali

```csharp
// LSP ihlali - Para çekme işlemi kısıtlaması
public class BankAccount
{
    protected decimal _balance;
    
    public BankAccount(decimal initialBalance)
    {
        _balance = initialBalance;
    }
    
    public virtual void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Yatırılan miktar pozitif olmalıdır.");
            
        _balance += amount;
        Console.WriteLine($"{amount:C} yatırıldı. Yeni bakiye: {_balance:C}");
    }
    
    public virtual void Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Çekilen miktar pozitif olmalıdır.");
            
        if (amount > _balance)
            throw new InvalidOperationException("Yetersiz bakiye.");
            
        _balance -= amount;
        Console.WriteLine($"{amount:C} çekildi. Yeni bakiye: {_balance:C}");
    }
    
    public decimal GetBalance()
    {
        return _balance;
    }
}

public class SavingsAccount : BankAccount
{
    private int _withdrawalsThisMonth;
    private const int MaxWithdrawalsPerMonth = 3;
    
    public SavingsAccount(decimal initialBalance) : base(initialBalance)
    {
        _withdrawalsThisMonth = 0;
    }
    
    public override void Withdraw(decimal amount)
    {
        if (_withdrawalsThisMonth >= MaxWithdrawalsPerMonth)
            throw new InvalidOperationException("Bu ay maksimum para çekme limitine ulaştınız.");
            
        base.Withdraw(amount);
        _withdrawalsThisMonth++;
    }
}

// Kullanım
public void ProcessBankOperations(BankAccount account)
{
    account.Deposit(1000);
    
    for (int i = 0; i < 4; i++)
    {
        try
        {
            account.Withdraw(100);
            Console.WriteLine("Para çekme işlemi başarılı.");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Hata: {ex.Message}");
        }
    }
}
```

Bu örnekte, `SavingsAccount` sınıfı `BankAccount` sınıfından türetilmiştir, ancak para çekme işlemini aylık maksimum 3 işlemle sınırlar. Eğer `ProcessBankOperations` metoduna bir `SavingsAccount` nesnesi geçirirsek, dördüncü para çekme işlemi başarısız olur. Bu, LSP'nin ihlalidir çünkü `SavingsAccount` nesnesi, `BankAccount` nesnesinin yerine tam olarak geçemez.

### LSP Uyumlu Tasarım

```csharp
// LSP uyumlu tasarım - Davranış garantileri
public interface IAccount
{
    void Deposit(decimal amount);
    bool TryWithdraw(decimal amount, out string errorMessage);
    decimal GetBalance();
}

public class StandardAccount : IAccount
{
    protected decimal _balance;
    
    public StandardAccount(decimal initialBalance)
    {
        _balance = initialBalance;
    }
    
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Yatırılan miktar pozitif olmalıdır.");
            
        _balance += amount;
        Console.WriteLine($"{amount:C} yatırıldı. Yeni bakiye: {_balance:C}");
    }
    
    public bool TryWithdraw(decimal amount, out string errorMessage)
    {
        errorMessage = string.Empty;
        
        if (amount <= 0)
        {
            errorMessage = "Çekilen miktar pozitif olmalıdır.";
            return false;
        }
        
        if (amount > _balance)
        {
            errorMessage = "Yetersiz bakiye.";
            return false;
        }
        
        _balance -= amount;
        Console.WriteLine($"{amount:C} çekildi. Yeni bakiye: {_balance:C}");
        return true;
    }
    
    public decimal GetBalance()
    {
        return _balance;
    }
}

public class SavingsAccount : IAccount
{
    private decimal _balance;
    private int _withdrawalsThisMonth;
    private const int MaxWithdrawalsPerMonth = 3;
    
    public SavingsAccount(decimal initialBalance)
    {
        _balance = initialBalance;
        _withdrawalsThisMonth = 0;
    }
    
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Yatırılan miktar pozitif olmalıdır.");
            
        _balance += amount;
        Console.WriteLine($"{amount:C} yatırıldı. Yeni bakiye: {_balance:C}");
    }
    
    public bool TryWithdraw(decimal amount, out string errorMessage)
    {
        errorMessage = string.Empty;
        
        if (amount <= 0)
        {
            errorMessage = "Çekilen miktar pozitif olmalıdır.";
            return false;
        }
        
        if (amount > _balance)
        {
            errorMessage = "Yetersiz bakiye.";
            return false;
        }
        
        if (_withdrawalsThisMonth >= MaxWithdrawalsPerMonth)
        {
            errorMessage = "Bu ay maksimum para çekme limitine ulaştınız.";
            return false;
        }
        
        _balance -= amount;
        _withdrawalsThisMonth++;
        Console.WriteLine($"{amount:C} çekildi. Yeni bakiye: {_balance:C}");
        return true;
    }
    
    public decimal GetBalance()
    {
        return _balance;
    }
}

// Kullanım
public void ProcessBankOperations(IAccount account)
{
    account.Deposit(1000);
    
    for (int i = 0; i < 4; i++)
    {
        if (account.TryWithdraw(100, out string errorMessage))
        {
            Console.WriteLine("Para çekme işlemi başarılı.");
        }
        else
        {
            Console.WriteLine($"Hata: {errorMessage}");
        }
    }
}
```

Bu tasarımda, hem `StandardAccount` hem de `SavingsAccount` sınıfları ortak bir `IAccount` arayüzünü uygular. Para çekme işlemi, başarı/başarısızlık durumunu döndüren ve hata mesajını `out` parametresi olarak veren bir `TryWithdraw` metodu ile gerçekleştirilir. Bu şekilde, her hesap türü kendi kısıtlamalarını uygulayabilir ve istemci kod bu kısıtlamaları uygun şekilde ele alabilir.

## 3. LSP İhlallerini Tespit Etme ve Düzeltme

LSP ihlallerini tespit etmek için bazı belirtilere dikkat edebilirsiniz:

1. **Tip Kontrolü**: Eğer kodunuzda sık sık `is` veya `as` operatörleri kullanıyorsanız, bu LSP ihlalinin bir belirtisi olabilir.

```csharp
// LSP ihlali belirtisi - Tip kontrolü
public void ProcessPayment(Payment payment)
{
    if (payment is CreditCardPayment)
    {
        // Kredi kartı ödemesi için özel işlem
    }
    else if (payment is PayPalPayment)
    {
        // PayPal ödemesi için özel işlem
    }
}
```

2. **Metot Reddi**: Alt sınıflar, üst sınıfın metotlarını uygulamayı reddediyorsa veya `NotImplementedException` fırlatıyorsa, bu LSP ihlalidir.

```csharp
// LSP ihlali - Metot reddi
public class Bird
{
    public virtual void Fly()
    {
        Console.WriteLine("Kuş uçuyor...");
    }
}

public class Penguin : Bird
{
    public override void Fly()
    {
        throw new NotImplementedException("Penguenler uçamaz!");
    }
}
```

3. **Değişen Davranış**: Alt sınıflar, üst sınıfın davranışını beklenmedik şekilde değiştiriyorsa, bu LSP ihlalidir.

```csharp
// LSP ihlali - Değişen davranış
public class Collection
{
    protected List<object> _items = new List<object>();
    
    public virtual void Add(object item)
    {
        _items.Add(item);
    }
    
    public virtual int Count => _items.Count;
}

public class UniqueCollection : Collection
{
    public override void Add(object item)
    {
        if (!_items.Contains(item))
        {
            _items.Add(item);
        }
    }
}
```

### LSP İhlallerini Düzeltme Stratejileri

1. **Arayüz Kullanımı**: Sınıflar arasında is-a ilişkisi yerine, ortak bir arayüz kullanın.

2. **Kompozisyon**: Kalıtım yerine kompozisyon kullanarak, davranışları daha esnek bir şekilde birleştirin.

3. **Davranış Garantileri**: Metotların davranışlarını açıkça belirtin ve alt sınıfların bu garantileri korumasını sağlayın.

4. **Tasarım Desenlerini Kullanma**: Strateji, Dekoratör veya Adaptör gibi tasarım desenleri, LSP ihlallerini önlemeye yardımcı olabilir.

## En İyi Pratikler

1. **Davranış Sözleşmelerini Açıkça Belirtin**
   - Metotların precondition ve postcondition'larını dokümante edin.
   - Sınıfların invariant'larını açıkça belirtin.
   - Alt sınıfların bu sözleşmelere uymasını sağlayın.

2. **"Tell, Don't Ask" Prensibini Uygulayın**
   - Nesnelerin tipini kontrol etmek yerine, polimorfizmi kullanın.
   - Nesnelere ne yapmaları gerektiğini söyleyin, ne olduklarını sormayın.

   ```csharp
   // Kötü - tip kontrolü
   public void ProcessShape(Shape shape)
   {
       if (shape is Circle)
       {
           // Daire işleme kodu...
       }
       else if (shape is Rectangle)
       {
           // Dikdörtgen işleme kodu...
       }
   }
   
   // İyi - polimorfizm
   public void ProcessShape(Shape shape)
   {
       shape.Process(); // Her şekil kendi işleme kodunu uygular
   }
   ```

3. **Kalıtımı Dikkatli Kullanın**
   - Kalıtımı yalnızca gerçek "is-a" ilişkileri için kullanın.
   - Davranış değişikliği gerektiren durumlarda, arayüzler ve kompozisyon kullanmayı düşünün.
   - "Composition over inheritance" (Kalıtım yerine kompozisyon) prensibini hatırlayın.

4. **Arayüzleri Küçük ve Odaklanmış Tutun**
   - Büyük arayüzler yerine, küçük ve odaklanmış arayüzler tasarlayın.
   - Bu, sınıfların yalnızca ihtiyaç duydukları davranışları uygulamasını sağlar.
   - Interface Segregation Principle (ISP) ile yakından ilişkilidir.

5. **Metot Dönüş Değerlerini Tutarlı Tutun**
   - Alt sınıfların metotları, üst sınıfın metotlarıyla aynı tip veya alt tip değerler döndürmelidir.
   - Dönüş değeri null olabilen metotlar için, tüm alt sınıflar da null dönebilmelidir.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Metot Reddi**
   ```csharp
   // Kötü - metot reddi
   public class Bird
   {
       public virtual void Fly()
       {
           Console.WriteLine("Kuş uçuyor...");
       }
   }
   
   public class Penguin : Bird
   {
       public override void Fly()
       {
           throw new NotImplementedException("Penguenler uçamaz!");
       }
   }
   
   // İyi - arayüz kullanımı
   public interface IBird
   {
       void Move();
   }
   
   public interface IFlyingBird : IBird
   {
       void Fly();
   }
   
   public class Sparrow : IFlyingBird
   {
       public void Move()
       {
           Console.WriteLine("Serçe hareket ediyor...");
       }
       
       public void Fly()
       {
           Console.WriteLine("Serçe uçuyor...");
       }
   }
   
   public class Penguin : IBird
   {
       public void Move()
       {
           Console.WriteLine("Penguen yürüyor ve yüzüyor...");
       }
   }
   ```

2. **Precondition Güçlendirme**
   ```csharp
   // Kötü - precondition güçlendirme
   public class FileProcessor
   {
       public virtual void ProcessFile(string filePath)
       {
           if (string.IsNullOrEmpty(filePath))
               throw new ArgumentException("Dosya yolu boş olamaz.");
               
           // Dosya işleme kodu...
       }
   }
   
   public class ImageFileProcessor : FileProcessor
   {
       public override void ProcessFile(string filePath)
       {
           if (string.IsNullOrEmpty(filePath))
               throw new ArgumentException("Dosya yolu boş olamaz.");
               
           if (!filePath.EndsWith(".jpg") && !filePath.EndsWith(".png"))
               throw new ArgumentException("Sadece JPG ve PNG dosyaları desteklenir.");
               
           // Resim dosyası işleme kodu...
       }
   }
   
   // İyi - strateji deseni kullanımı
   public interface IFileProcessor
   {
       bool CanProcess(string filePath);
       void ProcessFile(string filePath);
   }
   
   public class TextFileProcessor : IFileProcessor
   {
       public bool CanProcess(string filePath)
       {
           return !string.IsNullOrEmpty(filePath) && filePath.EndsWith(".txt");
       }
       
       public void ProcessFile(string filePath)
       {
           // Metin dosyası işleme kodu...
       }
   }
   
   public class ImageFileProcessor : IFileProcessor
   {
       public bool CanProcess(string filePath)
       {
           return !string.IsNullOrEmpty(filePath) && 
                  (filePath.EndsWith(".jpg") || filePath.EndsWith(".png"));
       }
       
       public void ProcessFile(string filePath)
       {
           // Resim dosyası işleme kodu...
       }
   }
   
   public class FileProcessingService
   {
       private readonly List<IFileProcessor> _processors;
       
       public FileProcessingService(IEnumerable<IFileProcessor> processors)
       {
           _processors = new List<IFileProcessor>(processors);
       }
       
       public void ProcessFile(string filePath)
       {
           var processor = _processors.FirstOrDefault(p => p.CanProcess(filePath));
           
           if (processor == null)
               throw new ArgumentException($"Desteklenmeyen dosya türü: {filePath}");
               
           processor.ProcessFile(filePath);
       }
   }
   ```

3. **Postcondition Zayıflatma**
   ```csharp
   // Kötü - postcondition zayıflatma
   public class UserRepository
   {
       public virtual User GetUserById(int id)
       {
           // Kullanıcıyı veritabanından getir
           // Bu metot her zaman geçerli bir User nesnesi döndürür
           return new User { Id = id, Name = "Test User" };
       }
   }
   
   public class CachedUserRepository : UserRepository
   {
       private Dictionary<int, User> _cache = new Dictionary<int, User>();
       
       public override User GetUserById(int id)
       {
           if (_cache.TryGetValue(id, out var user))
               return user;
               
           // Cache'te yoksa null dönebilir (postcondition zayıflatma)
           return null;
       }
   }
   
   // İyi - tutarlı postcondition
   public class UserRepository
   {
       public virtual User GetUserById(int id)
       {
           // Kullanıcıyı veritabanından getir
           // Bulunamazsa null döndür
           return new User { Id = id, Name = "Test User" };
       }
   }
   
   public class CachedUserRepository : UserRepository
   {
       private Dictionary<int, User> _cache = new Dictionary<int, User>();
       
       public override User GetUserById(int id)
       {
           if (_cache.TryGetValue(id, out var user))
               return user;
               
           // Üst sınıfın davranışını koru
           var result = base.GetUserById(id);
           
           if (result != null)
               _cache[id] = result;
               
           return result;
       }
   }
   ```

4. **Invariant İhlali**
   ```csharp
   // Kötü - invariant ihlali
   public class Stack<T>
   {
       protected List<T> _items = new List<T>();
       
       public virtual void Push(T item)
       {
           _items.Add(item);
       }
       
       public virtual T Pop()
       {
           if (_items.Count == 0)
               throw new InvalidOperationException("Yığın boş.");
               
           var item = _items[_items.Count - 1];
           _items.RemoveAt(_items.Count - 1);
           return item;
       }
       
       public virtual int Count => _items.Count;
   }
   
   public class QueueAdapter<T> : Stack<T>
   {
       public override T Pop()
       {
           if (_items.Count == 0)
               throw new InvalidOperationException("Kuyruk boş.");
               
           // FIFO davranışı (invariant ihlali)
           var item = _items[0];
           _items.RemoveAt(0);
           return item;
       }
   }
   
   // İyi - kompozisyon kullanımı
   public interface ICollection<T>
   {
       void Add(T item);
       T Remove();
       int Count { get; }
   }
   
   public class Stack<T> : ICollection<T>
   {
       private List<T> _items = new List<T>();
       
       public void Add(T item)
       {
           _items.Add(item);
       }
       
       public T Remove()
       {
           if (_items.Count == 0)
               throw new InvalidOperationException("Yığın boş.");
               
           var item = _items[_items.Count - 1];
           _items.RemoveAt(_items.Count - 1);
           return item;
       }
       
       public int Count => _items.Count;
   }
   
   public class Queue<T> : ICollection<T>
   {
       private List<T> _items = new List<T>();
       
       public void Add(T item)
       {
           _items.Add(item);
       }
       
       public T Remove()
       {
           if (_items.Count == 0)
               throw new InvalidOperationException("Kuyruk boş.");
               
           var item = _items[0];
           _items.RemoveAt(0);
           return item;
       }
       
       public int Count => _items.Count;
   }
   ```

Liskov Substitution Principle, nesne yönelimli programlamanın temel prensiplerinden biridir ve doğru uygulandığında, kodunuzun daha güvenilir, bakımı kolay ve genişletilebilir olmasını sağlar. Bu prensibi anlamak ve uygulamak, daha sağlam yazılım sistemleri tasarlamanıza yardımcı olacaktır. 