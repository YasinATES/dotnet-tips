# Deferred Execution (Ertelenmiş Yürütme)

LINQ'nun en önemli özelliklerinden biri, sorguların tanımlandıkları anda değil, sonuçların kullanıldığı anda yürütülmesidir. Bu özelliğe "Ertelenmiş Yürütme" (Deferred Execution) denir. Bu bölümde, ertelenmiş yürütmenin nasıl çalıştığını ve bunun performans üzerindeki etkilerini inceleyeceğiz.

## 1. IEnumerable vs IQueryable

LINQ sorgularının iki temel dönüş tipi vardır: `IEnumerable<T>` ve `IQueryable<T>`. Bu iki arayüz arasındaki farkları anlamak, LINQ'yu etkili bir şekilde kullanmak için önemlidir.

### IEnumerable

`IEnumerable<T>`, bellek içi koleksiyonlar için kullanılır ve LINQ to Objects'in temelini oluşturur.

```csharp
// Veri modelleri
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public DateTime RegistrationDate { get; set; }
    public decimal TotalBalance { get; set; }
    public bool IsActive { get; set; }
    public string CustomerType { get; set; }
}

// Örnek veri
List<Customer> customers = new List<Customer>
{
    new Customer { Id = 1, Name = "Ali Yılmaz", Email = "ali.yilmaz@example.com", RegistrationDate = new DateTime(2020, 5, 12), TotalBalance = 15000, IsActive = true, CustomerType = "Individual" },
    new Customer { Id = 2, Name = "Ayşe Demir", Email = "ayse.demir@example.com", RegistrationDate = new DateTime(2019, 3, 8), TotalBalance = 28500, IsActive = true, CustomerType = "Individual" },
    new Customer { Id = 3, Name = "Mehmet Kaya", Email = "mehmet.kaya@example.com", RegistrationDate = new DateTime(2021, 1, 15), TotalBalance = 7200, IsActive = false, CustomerType = "Individual" },
    new Customer { Id = 4, Name = "ABC Şirketi", Email = "info@abccompany.com", RegistrationDate = new DateTime(2018, 11, 20), TotalBalance = 142000, IsActive = true, CustomerType = "Corporate" },
    new Customer { Id = 5, Name = "XYZ Ltd. Şti.", Email = "contact@xyzltd.com", RegistrationDate = new DateTime(2022, 2, 3), TotalBalance = 83500, IsActive = true, CustomerType = "Corporate" }
};

// IEnumerable<T> sorgusu
IEnumerable<Customer> activeCustomers = customers.Where(c => c.IsActive);

// Bu noktada sorgu henüz yürütülmedi

// Sorgu sonuçlarını kullanma - bu noktada sorgu yürütülür
foreach (var customer in activeCustomers)
{
    Console.WriteLine($"{customer.Name} - {customer.Email}");
}
```

### IQueryable

`IQueryable<T>`, uzak veri kaynakları (örneğin, veritabanları) için kullanılır ve LINQ to SQL veya Entity Framework gibi ORM'lerin temelini oluşturur.

```csharp
// Entity Framework kullanarak veritabanı sorgusu
using (var context = new BankingContext())
{
    // IQueryable<T> sorgusu
    IQueryable<Customer> activeCustomers = context.Customers.Where(c => c.IsActive);
    
    // Bu noktada sorgu henüz veritabanına gönderilmedi
    
    // Sorgu sonuçlarını kullanma - bu noktada sorgu veritabanına gönderilir
    foreach (var customer in activeCustomers)
    {
        Console.WriteLine($"{customer.Name} - {customer.Email}");
    }
}
```

### IEnumerable vs IQueryable Karşılaştırması

```csharp
// Entity Framework örneği
using (var context = new BankingContext())
{
    // IQueryable<T> sorgusu - filtreleme veritabanında yapılır
    var highBalanceCustomers = context.Customers
        .Where(c => c.TotalBalance > 10000)
        .ToList();
    
    // IEnumerable<T> sorgusu - filtreleme bellekte yapılır
    var highBalanceCustomers2 = context.Customers
        .ToList() // Tüm müşteriler veritabanından çekilir
        .Where(c => c.TotalBalance > 10000); // Filtreleme bellekte yapılır
    
    // IQueryable<T> zincirleme sorgu - tüm filtreleme veritabanında yapılır
    var filteredCustomers = context.Customers
        .Where(c => c.IsActive)
        .Where(c => c.TotalBalance > 10000)
        .OrderBy(c => c.Name)
        .Take(10)
        .ToList();
    
    // SQL sorgusu şuna benzer:
    // SELECT TOP 10 * FROM Customers 
    // WHERE IsActive = 1 AND TotalBalance > 10000 
    // ORDER BY Name
}
```

## 2. Lazy Evaluation (Tembel Değerlendirme)

Lazy Evaluation (Tembel Değerlendirme), bir sorgunun sonuçlarının gerçekten ihtiyaç duyulana kadar hesaplanmaması anlamına gelir. Bu, LINQ'nun ertelenmiş yürütme özelliğinin temelidir.

```csharp
// İşlem sınıfı
public class Transaction
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public DateTime Date { get; set; }
    public decimal Amount { get; set; }
    public string Description { get; set; }
    public string TransactionType { get; set; }
}

// Örnek işlem verileri
List<Transaction> transactions = new List<Transaction>
{
    new Transaction { Id = 1, CustomerId = 1, Date = new DateTime(2023, 1, 15), Amount = 1500, Description = "Maaş", TransactionType = "Deposit" },
    new Transaction { Id = 2, CustomerId = 1, Date = new DateTime(2023, 1, 20), Amount = -200, Description = "Market alışverişi", TransactionType = "Withdrawal" },
    new Transaction { Id = 3, CustomerId = 2, Date = new DateTime(2023, 1, 18), Amount = 3000, Description = "Havale", TransactionType = "Deposit" },
    new Transaction { Id = 4, CustomerId = 3, Date = new DateTime(2023, 1, 10), Amount = -150, Description = "Fatura ödemesi", TransactionType = "Withdrawal" },
    new Transaction { Id = 5, CustomerId = 4, Date = new DateTime(2023, 1, 25), Amount = -5000, Description = "Kira ödemesi", TransactionType = "Withdrawal" },
    new Transaction { Id = 6, CustomerId = 5, Date = new DateTime(2023, 1, 22), Amount = 12000, Description = "Satış geliri", TransactionType = "Deposit" }
};

// Tembel değerlendirme örneği
Console.WriteLine("Sorgu tanımlanıyor...");
var largeDeposits = transactions
    .Where(t => {
        Console.WriteLine($"Where filtresi çalıştırılıyor: {t.Id}");
        return t.Amount > 1000 && t.TransactionType == "Deposit";
    })
    .OrderByDescending(t => {
        Console.WriteLine($"OrderBy sıralaması çalıştırılıyor: {t.Id}");
        return t.Amount;
    })
    .Select(t => {
        Console.WriteLine($"Select dönüşümü çalıştırılıyor: {t.Id}");
        return new { t.CustomerId, t.Date, t.Amount, t.Description };
    });

Console.WriteLine("Sorgu tanımlandı, ancak henüz yürütülmedi.");

// İlk öğeyi alma - bu noktada sorgu yürütülür
Console.WriteLine("İlk öğe alınıyor...");
var firstLargeDeposit = largeDeposits.FirstOrDefault();
Console.WriteLine($"İlk büyük yatırma: {firstLargeDeposit?.Amount ?? 0:C} - {firstLargeDeposit?.Description ?? "Yok"}");

// Tüm sonuçları alma - sorgu tekrar yürütülür
Console.WriteLine("\nTüm sonuçlar alınıyor...");
foreach (var deposit in largeDeposits)
{
    Console.WriteLine($"{deposit.Date.ToShortDateString()} - {deposit.Amount:C} - {deposit.Description}");
}
```

## 3. Multiple Enumeration (Çoklu Numaralandırma)

Ertelenmiş yürütmenin bir sonucu olarak, aynı sorgu birden fazla kez numaralandırıldığında, sorgu her seferinde yeniden yürütülür. Bu, beklenmeyen sonuçlara veya performans sorunlarına yol açabilir.

```csharp
// Çoklu numaralandırma örneği
var activeCustomers = customers.Where(c => {
    Console.WriteLine($"Müşteri filtreleniyor: {c.Id}");
    return c.IsActive;
});

// İlk numaralandırma
Console.WriteLine("İlk numaralandırma:");
int count = activeCustomers.Count();
Console.WriteLine($"Aktif müşteri sayısı: {count}");

// İkinci numaralandırma - sorgu tekrar yürütülür
Console.WriteLine("\nİkinci numaralandırma:");
foreach (var customer in activeCustomers)
{
    Console.WriteLine($"{customer.Name} - {customer.Email}");
}

// Üçüncü numaralandırma - sorgu tekrar yürütülür
Console.WriteLine("\nÜçüncü numaralandırma:");
var highBalanceActiveCustomers = activeCustomers.Where(c => c.TotalBalance > 20000);
foreach (var customer in highBalanceActiveCustomers)
{
    Console.WriteLine($"{customer.Name} - {customer.TotalBalance:C}");
}
```

### Çoklu Numaralandırma Sorunlarını Çözme

```csharp
// Sorguyu hemen yürütmek için ToList() kullanma
var activeCustomersList = customers.Where(c => {
    Console.WriteLine($"Müşteri filtreleniyor: {c.Id}");
    return c.IsActive;
}).ToList();

// İlk numaralandırma - sorgu zaten yürütüldü
Console.WriteLine("İlk numaralandırma (liste):");
int countList = activeCustomersList.Count();
Console.WriteLine($"Aktif müşteri sayısı: {countList}");

// İkinci numaralandırma - sorgu tekrar yürütülmez
Console.WriteLine("\nİkinci numaralandırma (liste):");
foreach (var customer in activeCustomersList)
{
    Console.WriteLine($"{customer.Name} - {customer.Email}");
}

// Üçüncü numaralandırma - yeni sorgu oluşturulur, ancak önceki sonuçlar üzerinde çalışır
Console.WriteLine("\nÜçüncü numaralandırma (liste):");
var highBalanceActiveCustomersList = activeCustomersList.Where(c => c.TotalBalance > 20000);
foreach (var customer in highBalanceActiveCustomersList)
{
    Console.WriteLine($"{customer.Name} - {customer.TotalBalance:C}");
}
```

## 4. ToList, ToArray, ToDictionary

Bu metotlar, ertelenmiş sorguları hemen yürütür ve sonuçları belirli bir koleksiyon tipine dönüştürür.

### ToList

`ToList()` metodu, sorgu sonuçlarını bir `List<T>` koleksiyonuna dönüştürür.

```csharp
// ToList() örneği
var activeCustomersList = customers
    .Where(c => c.IsActive)
    .ToList();

// Listeye yeni bir müşteri ekleme
customers.Add(new Customer
{
    Id = 6,
    Name = "Fatma Öztürk",
    Email = "fatma.ozturk@example.com",
    RegistrationDate = DateTime.Now,
    TotalBalance = 9500,
    IsActive = true,
    CustomerType = "Individual"
});

// activeCustomersList, yeni eklenen müşteriyi içermez
Console.WriteLine($"Aktif müşteri sayısı (liste): {activeCustomersList.Count}");

// Sorguyu tekrar çalıştırma
var updatedActiveCustomers = customers
    .Where(c => c.IsActive)
    .ToList();

Console.WriteLine($"Güncellenmiş aktif müşteri sayısı: {updatedActiveCustomers.Count}");
```

### ToArray

`ToArray()` metodu, sorgu sonuçlarını bir dizi (array) olarak döndürür.

```csharp
// ToArray() örneği
var transactionsArray = transactions
    .Where(t => t.Amount > 0)
    .OrderBy(t => t.Date)
    .ToArray();

// Dizi üzerinde işlemler yapma
Console.WriteLine($"Pozitif işlem sayısı: {transactionsArray.Length}");
Console.WriteLine($"İlk işlem: {transactionsArray[0].Description} - {transactionsArray[0].Amount:C}");
```

### ToDictionary

`ToDictionary()` metodu, sorgu sonuçlarını bir `Dictionary<TKey, TValue>` koleksiyonuna dönüştürür.

```csharp
// ToDictionary() örneği
var customerDictionary = customers
    .ToDictionary(c => c.Id, c => c);

// Sözlük üzerinde işlemler yapma
if (customerDictionary.TryGetValue(3, out var customer3))
{
    Console.WriteLine($"ID 3 olan müşteri: {customer3.Name} - {customer3.Email}");
}

// Hesap numarasına göre işlem sözlüğü oluşturma
var transactionsByAccount = transactions
    .GroupBy(t => t.CustomerId)
    .ToDictionary(
        g => g.Key,
        g => g.ToList()
    );

// Belirli bir müşterinin işlemlerini alma
if (transactionsByAccount.TryGetValue(1, out var customerTransactions))
{
    Console.WriteLine($"Müşteri 1'in işlemleri:");
    foreach (var transaction in customerTransactions)
    {
        Console.WriteLine($"{transaction.Date.ToShortDateString()} - {transaction.Amount:C} - {transaction.Description}");
    }
}
```

## 5. Immediate Execution Methods (Anında Yürütme Metotları)

Bazı LINQ metotları, çağrıldıkları anda sorguyu yürütür. Bu metotlar, ertelenmiş yürütme yerine anında yürütme sağlar.

### Aggregate, Average, Count, First, Last, Max, Min, Sum

Bu metotlar, bir koleksiyon üzerinde tek bir değer döndüren işlemler yapar ve çağrıldıkları anda sorguyu yürütür.

```csharp
// Anında yürütme metotları örnekleri
Console.WriteLine("Anında yürütme metotları:");

// Count
int activeCount = customers.Count(c => {
    Console.WriteLine($"Count filtresi çalıştırılıyor: {c.Id}");
    return c.IsActive;
});
Console.WriteLine($"Aktif müşteri sayısı: {activeCount}");

// Sum
decimal totalBalance = customers.Sum(c => {
    Console.WriteLine($"Sum hesaplaması çalıştırılıyor: {c.Id}");
    return c.TotalBalance;
});
Console.WriteLine($"Toplam bakiye: {totalBalance:C}");

// Average
decimal averageBalance = customers.Average(c => c.TotalBalance);
Console.WriteLine($"Ortalama bakiye: {averageBalance:C}");

// First
var firstCorporate = customers.First(c => c.CustomerType == "Corporate");
Console.WriteLine($"İlk kurumsal müşteri: {firstCorporate.Name}");

// FirstOrDefault
var firstInactive = customers.FirstOrDefault(c => !c.IsActive);
Console.WriteLine($"İlk aktif olmayan müşteri: {firstInactive?.Name ?? "Yok"}");

// Last
var lastCustomer = customers.OrderBy(c => c.RegistrationDate).Last();
Console.WriteLine($"En son kaydolan müşteri: {lastCustomer.Name} - {lastCustomer.RegistrationDate.ToShortDateString()}");

// Max
var maxBalance = customers.Max(c => c.TotalBalance);
Console.WriteLine($"En yüksek bakiye: {maxBalance:C}");

// Min
var minBalance = customers.Min(c => c.TotalBalance);
Console.WriteLine($"En düşük bakiye: {minBalance:C}");
```

### Any, All, Contains

Bu metotlar, bir koleksiyondaki öğelerin belirli koşulları sağlayıp sağlamadığını kontrol eder ve çağrıldıkları anda sorguyu yürütür.

```csharp
// Any, All, Contains örnekleri
Console.WriteLine("\nAny, All, Contains örnekleri:");

// Any
bool hasHighBalance = customers.Any(c => {
    Console.WriteLine($"Any filtresi çalıştırılıyor: {c.Id}");
    return c.TotalBalance > 100000;
});
Console.WriteLine($"Yüksek bakiyeli müşteri var mı: {hasHighBalance}");

// All
bool allActive = customers.All(c => {
    Console.WriteLine($"All filtresi çalıştırılıyor: {c.Id}");
    return c.IsActive;
});
Console.WriteLine($"Tüm müşteriler aktif mi: {allActive}");

// Contains
var customerToFind = customers.First();
bool containsCustomer = customers.Contains(customerToFind);
Console.WriteLine($"Müşteri listede var mı: {containsCustomer}");
```

## 6. Performance Considerations (Performans Değerlendirmeleri)

Ertelenmiş yürütme ve anında yürütme arasındaki farkları anlamak, LINQ sorgularının performansını optimize etmek için önemlidir.

### Veritabanı Sorguları için Performans İpuçları

```csharp
// Entity Framework örneği
using (var context = new BankingContext())
{
    // Kötü performans - tüm müşteriler veritabanından çekilir
    var badQuery = context.Customers
        .ToList() // Tüm müşteriler veritabanından çekilir
        .Where(c => c.IsActive && c.TotalBalance > 10000) // Filtreleme bellekte yapılır
        .OrderBy(c => c.Name)
        .Take(10);
    
    // İyi performans - filtreleme veritabanında yapılır
    var goodQuery = context.Customers
        .Where(c => c.IsActive && c.TotalBalance > 10000) // Filtreleme veritabanında yapılır
        .OrderBy(c => c.Name)
        .Take(10)
        .ToList(); // Sadece 10 müşteri veritabanından çekilir
    
    // İlişkili verileri yükleme - kötü performans (N+1 sorunu)
    var customersWithTransactionsBad = context.Customers
        .Where(c => c.IsActive)
        .ToList(); // Müşteriler yüklenir
    
    foreach (var customer in customersWithTransactionsBad)
    {
        // Her müşteri için ayrı bir sorgu çalıştırılır
        var transactions = context.Transactions
            .Where(t => t.CustomerId == customer.Id)
            .ToList();
        
        Console.WriteLine($"{customer.Name} - İşlem sayısı: {transactions.Count}");
    }
    
    // İlişkili verileri yükleme - iyi performans (Include kullanımı)
    var customersWithTransactionsGood = context.Customers
        .Where(c => c.IsActive)
        .Include(c => c.Transactions) // İlişkili veriler tek sorguda yüklenir
        .ToList();
    
    foreach (var customer in customersWithTransactionsGood)
    {
        Console.WriteLine($"{customer.Name} - İşlem sayısı: {customer.Transactions.Count}");
    }
}
```

### Bellek İçi Koleksiyonlar için Performans İpuçları

```csharp
// Büyük veri kümesi simülasyonu
var largeCustomerList = Enumerable.Range(1, 1000000)
    .Select(i => new Customer
    {
        Id = i,
        Name = $"Customer {i}",
        TotalBalance = i * 100,
        IsActive = i % 10 != 0
    })
    .ToList();

// Performans ölçümü
var stopwatch = new System.Diagnostics.Stopwatch();

// Kötü performans - çoklu numaralandırma
stopwatch.Start();
var badPerformanceQuery = largeCustomerList.Where(c => c.IsActive);
var count1 = badPerformanceQuery.Count();
var sum1 = badPerformanceQuery.Sum(c => c.TotalBalance);
var avg1 = badPerformanceQuery.Average(c => c.TotalBalance);
stopwatch.Stop();
Console.WriteLine($"Kötü performans süresi: {stopwatch.ElapsedMilliseconds} ms");

// İyi performans - tek numaralandırma
stopwatch.Restart();
var goodPerformanceQuery = largeCustomerList.Where(c => c.IsActive).ToList();
var count2 = goodPerformanceQuery.Count;
var sum2 = goodPerformanceQuery.Sum(c => c.TotalBalance);
var avg2 = goodPerformanceQuery.Average(c => c.TotalBalance);
stopwatch.Stop();
Console.WriteLine($"İyi performans süresi: {stopwatch.ElapsedMilliseconds} ms");

// Daha iyi performans - LINQ metotlarını zincirleme
stopwatch.Restart();
var betterPerformanceQuery = largeCustomerList
    .Where(c => c.IsActive)
    .Aggregate(
        new { Count = 0, Sum = 0m },
        (acc, c) => new { Count = acc.Count + 1, Sum = acc.Sum + c.TotalBalance },
        acc => new { Count = acc.Count, Sum = acc.Sum, Average = acc.Sum / acc.Count }
    );
stopwatch.Stop();
Console.WriteLine($"Daha iyi performans süresi: {stopwatch.ElapsedMilliseconds} ms");
Console.WriteLine($"Sonuçlar: Sayı={betterPerformanceQuery.Count}, Toplam={betterPerformanceQuery.Sum:C}, Ortalama={betterPerformanceQuery.Average:C}");
```

## Özet

Bu bölümde, LINQ'nun ertelenmiş yürütme özelliğini ve bunun performans üzerindeki etkilerini inceledik:

1. **IEnumerable vs IQueryable**: `IEnumerable<T>` bellek içi koleksiyonlar için kullanılırken, `IQueryable<T>` uzak veri kaynakları için kullanılır.

2. **Lazy Evaluation**: LINQ sorguları, sonuçlar gerçekten ihtiyaç duyulana kadar hesaplanmaz.

3. **Multiple Enumeration**: Aynı sorgu birden fazla kez numaralandırıldığında, sorgu her seferinde yeniden yürütülür.

4. **ToList, ToArray, ToDictionary**: Bu metotlar, ertelenmiş sorguları hemen yürütür ve sonuçları belirli bir koleksiyon tipine dönüştürür.

5. **Immediate Execution Methods**: Bazı LINQ metotları, çağrıldıkları anda sorguyu yürütür.

6. **Performance Considerations**: Ertelenmiş yürütme ve anında yürütme arasındaki farkları anlamak, LINQ sorgularının performansını optimize etmek için önemlidir.

LINQ'nun ertelenmiş yürütme özelliğini anlamak ve doğru şekilde kullanmak, daha verimli ve performanslı uygulamalar geliştirmenize yardımcı olacaktır. 