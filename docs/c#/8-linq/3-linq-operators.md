# LINQ Operators (LINQ Operatörleri)

LINQ, veri sorgulama ve dönüştürme işlemleri için çeşitli operatörler sunar. Bu bölümde, LINQ'nun en yaygın kullanılan operatörlerini kategorilere ayırarak inceleyeceğiz.

## 1. Filtering Operators (Filtreleme Operatörleri)

Filtreleme operatörleri, bir koleksiyondaki öğeleri belirli koşullara göre filtrelemenizi sağlar.

### Where

`Where` operatörü, bir koleksiyondaki öğeleri belirli bir koşula göre filtrelemenizi sağlar.

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
    public string RiskLevel { get; set; }
}

// Örnek veri
List<Customer> customers = new List<Customer>
{
    new Customer { Id = 1, Name = "Ali Yılmaz", Email = "ali.yilmaz@example.com", RegistrationDate = new DateTime(2020, 5, 12), TotalBalance = 15000, IsActive = true, CustomerType = "Individual", RiskLevel = "Low" },
    new Customer { Id = 2, Name = "Ayşe Demir", Email = "ayse.demir@example.com", RegistrationDate = new DateTime(2019, 3, 8), TotalBalance = 28500, IsActive = true, CustomerType = "Individual", RiskLevel = "Medium" },
    new Customer { Id = 3, Name = "Mehmet Kaya", Email = "mehmet.kaya@example.com", RegistrationDate = new DateTime(2021, 1, 15), TotalBalance = 7200, IsActive = false, CustomerType = "Individual", RiskLevel = "High" },
    new Customer { Id = 4, Name = "ABC Şirketi", Email = "info@abccompany.com", RegistrationDate = new DateTime(2018, 11, 20), TotalBalance = 142000, IsActive = true, CustomerType = "Corporate", RiskLevel = "Low" },
    new Customer { Id = 5, Name = "XYZ Ltd. Şti.", Email = "contact@xyzltd.com", RegistrationDate = new DateTime(2022, 2, 3), TotalBalance = 83500, IsActive = true, CustomerType = "Corporate", RiskLevel = "Medium" }
};

// Aktif müşterileri filtreleme
var activeCustomers = customers.Where(c => c.IsActive);

// Yüksek bakiyeli müşterileri filtreleme
var highBalanceCustomers = customers.Where(c => c.TotalBalance > 50000);

// Birden fazla koşul kullanma
var targetCustomers = customers.Where(c => c.IsActive && c.CustomerType == "Individual" && c.TotalBalance > 10000);

// Tarih bazlı filtreleme
var newCustomers = customers.Where(c => c.RegistrationDate >= new DateTime(2020, 1, 1));

// Sonuçları yazdırma
foreach (var customer in targetCustomers)
{
    Console.WriteLine($"{customer.Name} - {customer.Email} - {customer.TotalBalance:C}");
}
```

### OfType

`OfType` operatörü, bir koleksiyondaki belirli bir türdeki öğeleri filtrelemenizi sağlar.

```csharp
// Farklı hesap türleri
public abstract class Account
{
    public int Id { get; set; }
    public string AccountNumber { get; set; }
    public decimal Balance { get; set; }
    public DateTime OpenDate { get; set; }
    public bool IsActive { get; set; }
}

public class CheckingAccount : Account
{
    public decimal OverdraftLimit { get; set; }
}

public class SavingsAccount : Account
{
    public decimal InterestRate { get; set; }
}

public class LoanAccount : Account
{
    public decimal InterestRate { get; set; }
    public DateTime MaturityDate { get; set; }
}

// Karışık hesap listesi
List<Account> accounts = new List<Account>
{
    new CheckingAccount { Id = 1, AccountNumber = "C-1001", Balance = 5000, OpenDate = new DateTime(2020, 5, 15), IsActive = true, OverdraftLimit = 1000 },
    new SavingsAccount { Id = 2, AccountNumber = "S-2001", Balance = 15000, OpenDate = new DateTime(2019, 8, 10), IsActive = true, InterestRate = 0.03m },
    new CheckingAccount { Id = 3, AccountNumber = "C-1002", Balance = 2500, OpenDate = new DateTime(2021, 2, 20), IsActive = true, OverdraftLimit = 500 },
    new LoanAccount { Id = 4, AccountNumber = "L-3001", Balance = -50000, OpenDate = new DateTime(2022, 1, 5), IsActive = true, InterestRate = 0.08m, MaturityDate = new DateTime(2027, 1, 5) },
    new SavingsAccount { Id = 5, AccountNumber = "S-2002", Balance = 30000, OpenDate = new DateTime(2018, 11, 12), IsActive = false, InterestRate = 0.025m }
};

// Vadesiz hesapları filtreleme
var checkingAccounts = accounts.OfType<CheckingAccount>();

// Tasarruf hesaplarını filtreleme
var savingsAccounts = accounts.OfType<SavingsAccount>().Where(a => a.IsActive);

// Sonuçları yazdırma
Console.WriteLine("Vadesiz Hesaplar:");
foreach (var account in checkingAccounts)
{
    Console.WriteLine($"{account.AccountNumber} - {account.Balance:C} - Limit: {account.OverdraftLimit:C}");
}

Console.WriteLine("\nAktif Tasarruf Hesapları:");
foreach (var account in savingsAccounts)
{
    Console.WriteLine($"{account.AccountNumber} - {account.Balance:C} - Faiz Oranı: {account.InterestRate:P}");
}
```

## 2. Projection Operators (Projeksiyon Operatörleri)

Projeksiyon operatörleri, bir koleksiyondaki öğeleri başka bir forma dönüştürmenizi sağlar.

### Select

`Select` operatörü, bir koleksiyondaki her öğeyi başka bir forma dönüştürmenizi sağlar.

```csharp
// Müşteri bilgilerini dönüştürme
var customerInfos = customers.Select(c => new
{
    FullName = c.Name,
    Email = c.Email,
    Balance = c.TotalBalance,
    CustomerSince = c.RegistrationDate.Year
});

// Hesap özetleri oluşturma
var accountSummaries = accounts.Select(a => new
{
    AccountNo = a.AccountNumber,
    Type = a.GetType().Name.Replace("Account", ""),
    Balance = a.Balance,
    Status = a.IsActive ? "Aktif" : "Pasif"
});

// İndeks kullanarak dönüştürme
var numberedCustomers = customers.Select((c, index) => new
{
    No = index + 1,
    Name = c.Name,
    Balance = c.TotalBalance
});

// Sonuçları yazdırma
Console.WriteLine("Müşteri Bilgileri:");
foreach (var info in customerInfos)
{
    Console.WriteLine($"{info.FullName} - {info.Email} - {info.Balance:C} - Müşteri Yılı: {info.CustomerSince}");
}
```

### SelectMany

`SelectMany` operatörü, iç içe koleksiyonları düzleştirmenizi sağlar.

```csharp
// Müşteri ve hesapları
public class CustomerWithAccounts
{
    public int Id { get; set; }
    public string Name { get; set; }
    public List<Account> Accounts { get; set; } = new List<Account>();
}

// Örnek veri
List<CustomerWithAccounts> customersWithAccounts = new List<CustomerWithAccounts>
{
    new CustomerWithAccounts
    {
        Id = 1,
        Name = "Ali Yılmaz",
        Accounts = new List<Account>
        {
            new CheckingAccount { Id = 101, AccountNumber = "C-1001", Balance = 5000, IsActive = true },
            new SavingsAccount { Id = 102, AccountNumber = "S-2001", Balance = 15000, IsActive = true }
        }
    },
    new CustomerWithAccounts
    {
        Id = 2,
        Name = "Ayşe Demir",
        Accounts = new List<Account>
        {
            new CheckingAccount { Id = 201, AccountNumber = "C-1002", Balance = 3500, IsActive = true },
            new SavingsAccount { Id = 202, AccountNumber = "S-2002", Balance = 25000, IsActive = true },
            new LoanAccount { Id = 203, AccountNumber = "L-3001", Balance = -75000, IsActive = true }
        }
    }
};

// Tüm hesapları düzleştirme
var allAccounts = customersWithAccounts.SelectMany(c => c.Accounts);

// Müşteri bilgisiyle birlikte tüm hesapları düzleştirme
var customerAccounts = customersWithAccounts.SelectMany(c => c.Accounts, (customer, account) => new
{
    CustomerName = customer.Name,
    AccountNumber = account.AccountNumber,
    Balance = account.Balance,
    AccountType = account.GetType().Name
});

// Sonuçları yazdırma
Console.WriteLine("Müşteri Hesapları:");
foreach (var account in customerAccounts)
{
    Console.WriteLine($"{account.CustomerName} - {account.AccountNumber} - {account.AccountType} - {account.Balance:C}");
}
```

## 3. Ordering Operators (Sıralama Operatörleri)

Sıralama operatörleri, bir koleksiyondaki öğeleri belirli kriterlere göre sıralamanızı sağlar.

### OrderBy, OrderByDescending

`OrderBy` ve `OrderByDescending` operatörleri, bir koleksiyonu belirli bir özelliğe göre artan veya azalan sırada sıralamanızı sağlar.

```csharp
// İşlem sınıfı
public class Transaction
{
    public int Id { get; set; }
    public string AccountNumber { get; set; }
    public DateTime Date { get; set; }
    public decimal Amount { get; set; }
    public string Description { get; set; }
    public string TransactionType { get; set; }
}

// Örnek işlem verileri
List<Transaction> transactions = new List<Transaction>
{
    new Transaction { Id = 1, AccountNumber = "C-1001", Date = new DateTime(2023, 1, 15), Amount = 1500, Description = "Maaş", TransactionType = "Deposit" },
    new Transaction { Id = 2, AccountNumber = "C-1001", Date = new DateTime(2023, 1, 20), Amount = -200, Description = "Market alışverişi", TransactionType = "Withdrawal" },
    new Transaction { Id = 3, AccountNumber = "S-2001", Date = new DateTime(2023, 1, 18), Amount = 3000, Description = "Havale", TransactionType = "Deposit" },
    new Transaction { Id = 4, AccountNumber = "C-1002", Date = new DateTime(2023, 1, 10), Amount = -150, Description = "Fatura ödemesi", TransactionType = "Withdrawal" },
    new Transaction { Id = 5, AccountNumber = "S-2002", Date = new DateTime(2023, 1, 25), Amount = -500, Description = "Kira ödemesi", TransactionType = "Withdrawal" },
    new Transaction { Id = 6, AccountNumber = "L-3001", Date = new DateTime(2023, 1, 22), Amount = -1200, Description = "Kredi ödemesi", TransactionType = "Payment" }
};

// İşlemleri tarihe göre sıralama (artan)
var transactionsByDate = transactions.OrderBy(t => t.Date);

// İşlemleri tutara göre sıralama (azalan)
var transactionsByAmount = transactions.OrderByDescending(t => Math.Abs(t.Amount));

// Sonuçları yazdırma
Console.WriteLine("İşlemler (Tarihe Göre):");
foreach (var transaction in transactionsByDate)
{
    Console.WriteLine($"{transaction.Date.ToShortDateString()} - {transaction.AccountNumber} - {transaction.Amount:C} - {transaction.Description}");
}

Console.WriteLine("\nİşlemler (Tutara Göre):");
foreach (var transaction in transactionsByAmount)
{
    Console.WriteLine($"{transaction.Date.ToShortDateString()} - {transaction.AccountNumber} - {transaction.Amount:C} - {transaction.Description}");
}
```

### ThenBy, ThenByDescending

`ThenBy` ve `ThenByDescending` operatörleri, ilk sıralama kriterinden sonra ikincil sıralama kriterleri eklemenizi sağlar.

```csharp
// İşlemleri hesap numarasına göre sıralama, ardından tarihe göre sıralama
var transactionsByAccountAndDate = transactions
    .OrderBy(t => t.AccountNumber)
    .ThenBy(t => t.Date);

// İşlemleri işlem tipine göre sıralama, ardından tutara göre azalan sıralama
var transactionsByTypeAndAmount = transactions
    .OrderBy(t => t.TransactionType)
    .ThenByDescending(t => Math.Abs(t.Amount));

// Sonuçları yazdırma
Console.WriteLine("İşlemler (Hesap ve Tarihe Göre):");
foreach (var transaction in transactionsByAccountAndDate)
{
    Console.WriteLine($"{transaction.AccountNumber} - {transaction.Date.ToShortDateString()} - {transaction.Amount:C} - {transaction.Description}");
}
```

## 4. Grouping Operators (Gruplama Operatörleri)

Gruplama operatörleri, bir koleksiyondaki öğeleri belirli özelliklere göre gruplandırmanızı sağlar.

### GroupBy

`GroupBy` operatörü, bir koleksiyondaki öğeleri belirli bir özelliğe göre gruplandırmanızı sağlar.

```csharp
// İşlemleri işlem tipine göre gruplandırma
var transactionsByType = transactions.GroupBy(t => t.TransactionType);

// İşlemleri hesap numarasına göre gruplandırma ve özet bilgileri hesaplama
var transactionSummaryByAccount = transactions
    .GroupBy(t => t.AccountNumber)
    .Select(g => new
    {
        AccountNumber = g.Key,
        TransactionCount = g.Count(),
        TotalDeposit = g.Where(t => t.Amount > 0).Sum(t => t.Amount),
        TotalWithdrawal = g.Where(t => t.Amount < 0).Sum(t => t.Amount),
        NetAmount = g.Sum(t => t.Amount)
    });

// Sonuçları yazdırma
Console.WriteLine("İşlem Tipleri:");
foreach (var group in transactionsByType)
{
    Console.WriteLine($"Tip: {group.Key}");
    Console.WriteLine($"İşlem Sayısı: {group.Count()}");
    Console.WriteLine($"Toplam Tutar: {group.Sum(t => t.Amount):C}");
    
    foreach (var transaction in group)
    {
        Console.WriteLine($"  - {transaction.Date.ToShortDateString()} - {transaction.Amount:C} - {transaction.Description}");
    }
    
    Console.WriteLine();
}

Console.WriteLine("Hesap Bazlı İşlem Özeti:");
foreach (var summary in transactionSummaryByAccount)
{
    Console.WriteLine($"Hesap: {summary.AccountNumber}");
    Console.WriteLine($"İşlem Sayısı: {summary.TransactionCount}");
    Console.WriteLine($"Toplam Yatırma: {summary.TotalDeposit:C}");
    Console.WriteLine($"Toplam Çekme: {summary.TotalWithdrawal:C}");
    Console.WriteLine($"Net Tutar: {summary.NetAmount:C}");
    Console.WriteLine();
}
```

## 5. Set Operators (Küme Operatörleri)

Küme operatörleri, koleksiyonlar arasında küme işlemleri yapmanızı sağlar.

### Distinct

`Distinct` operatörü, bir koleksiyondaki tekrarlayan öğeleri kaldırmanızı sağlar.

```csharp
// İşlem yapılan hesap numaralarını alma
var accountNumbers = transactions.Select(t => t.AccountNumber);

// Tekrarlayan hesap numaralarını kaldırma
var distinctAccountNumbers = accountNumbers.Distinct();

// Sonuçları yazdırma
Console.WriteLine("İşlem Yapılan Hesaplar:");
foreach (var accountNumber in distinctAccountNumbers)
{
    Console.WriteLine(accountNumber);
}
```

### Union, Intersect, Except

Bu operatörler, iki koleksiyon arasında birleşim, kesişim ve fark işlemleri yapmanızı sağlar.

```csharp
// İki farklı müşteri listesi
List<Customer> premiumCustomers = customers.Where(c => c.TotalBalance > 50000).ToList();
List<Customer> longTermCustomers = customers.Where(c => c.RegistrationDate < new DateTime(2020, 1, 1)).ToList();

// Birleşim: Premium veya uzun süreli müşteriler
var unionCustomers = premiumCustomers.Union(longTermCustomers, new CustomerComparer());

// Kesişim: Hem premium hem de uzun süreli müşteriler
var intersectCustomers = premiumCustomers.Intersect(longTermCustomers, new CustomerComparer());

// Fark: Premium ama uzun süreli olmayan müşteriler
var exceptCustomers = premiumCustomers.Except(longTermCustomers, new CustomerComparer());

// Müşteri karşılaştırıcı
public class CustomerComparer : IEqualityComparer<Customer>
{
    public bool Equals(Customer x, Customer y)
    {
        return x.Id == y.Id;
    }

    public int GetHashCode(Customer obj)
    {
        return obj.Id.GetHashCode();
    }
}

// Sonuçları yazdırma
Console.WriteLine("Premium veya Uzun Süreli Müşteriler:");
foreach (var customer in unionCustomers)
{
    Console.WriteLine($"{customer.Name} - {customer.TotalBalance:C} - Kayıt: {customer.RegistrationDate.ToShortDateString()}");
}
```

## 6. Quantifier Operators (Niceleyici Operatörler)

Niceleyici operatörler, bir koleksiyondaki öğelerin belirli koşulları sağlayıp sağlamadığını kontrol etmenizi sağlar.

### Any, All, Contains

Bu operatörler, koleksiyondaki öğelerin belirli koşulları sağlayıp sağlamadığını kontrol etmenizi sağlar.

```csharp
// Aktif olmayan müşteri var mı?
bool hasInactiveCustomers = customers.Any(c => !c.IsActive);

// Tüm müşterilerin bakiyesi 1000'den büyük mü?
bool allCustomersHaveHighBalance = customers.All(c => c.TotalBalance > 1000);

// Yüksek riskli müşteri var mı?
bool hasHighRiskCustomers = customers.Any(c => c.RiskLevel == "High");

// Belirli bir hesap numarası var mı?
bool containsAccount = transactions.Any(t => t.AccountNumber == "S-2001");

// Sonuçları yazdırma
Console.WriteLine($"Aktif Olmayan Müşteri Var Mı: {hasInactiveCustomers}");
Console.WriteLine($"Tüm Müşterilerin Bakiyesi 1000'den Büyük Mü: {allCustomersHaveHighBalance}");
Console.WriteLine($"Yüksek Riskli Müşteri Var Mı: {hasHighRiskCustomers}");
Console.WriteLine($"S-2001 Hesabı İşlem Yapmış Mı: {containsAccount}");
```

## 7. Partitioning Operators (Bölümleme Operatörleri)

Bölümleme operatörleri, bir koleksiyonda belirli kriterlere göre bölümlere ayırmanızı sağlar.

### Take, Skip, TakeWhile, SkipWhile

Bu operatörler, bir koleksiyondan belirli sayıda öğe almanızı veya atlamanızı sağlar.

```csharp
// En yüksek bakiyeli 3 müşteri
var top3Customers = customers
    .OrderByDescending(c => c.TotalBalance)
    .Take(3);

// İlk 2 müşteriyi atlayıp sonraki 3 müşteriyi alma
var page2Customers = customers
    .OrderBy(c => c.Name)
    .Skip(2)
    .Take(3);

// Bakiyesi 10000'den büyük olan müşterileri alma (koşul sağlanana kadar)
var highBalanceCustomers = customers
    .OrderByDescending(c => c.TotalBalance)
    .TakeWhile(c => c.TotalBalance > 10000);

// Bakiyesi 10000'den küçük olan müşterileri atlama (koşul sağlanana kadar)
var remainingCustomers = customers
    .OrderBy(c => c.TotalBalance)
    .SkipWhile(c => c.TotalBalance < 10000);

// Sonuçları yazdırma
Console.WriteLine("En Yüksek Bakiyeli 3 Müşteri:");
foreach (var customer in top3Customers)
{
    Console.WriteLine($"{customer.Name} - {customer.TotalBalance:C}");
}
```

## 8. Aggregation Operators (Toplama Operatörleri)

Toplama operatörleri, bir koleksiyonda sayısal işlemler yapmanızı sağlar.

### Count, Sum, Average, Min, Max

Bu operatörler, koleksiyonda sayısal işlemler yapmanızı sağlar.

```csharp
// Toplam müşteri sayısı
int totalCustomers = customers.Count();

// Aktif müşteri sayısı
int activeCustomers = customers.Count(c => c.IsActive);

// Toplam bakiye
decimal totalBalance = customers.Sum(c => c.TotalBalance);

// Ortalama bakiye
decimal averageBalance = customers.Average(c => c.TotalBalance);

// En düşük bakiye
decimal minBalance = customers.Min(c => c.TotalBalance);

// En yüksek bakiye
decimal maxBalance = customers.Max(c => c.TotalBalance);

// Kurumsal müşterilerin toplam bakiyesi
decimal corporateTotalBalance = customers
    .Where(c => c.CustomerType == "Corporate")
    .Sum(c => c.TotalBalance);

// İstatistikleri yazdırma
Console.WriteLine("Müşteri İstatistikleri:");
Console.WriteLine($"Toplam Müşteri: {totalCustomers}");
Console.WriteLine($"Aktif Müşteri: {activeCustomers}");
Console.WriteLine($"Toplam Bakiye: {totalBalance:C}");
Console.WriteLine($"Ortalama Bakiye: {averageBalance:C}");
Console.WriteLine($"En Düşük Bakiye: {minBalance:C}");
Console.WriteLine($"En Yüksek Bakiye: {maxBalance:C}");
Console.WriteLine($"Kurumsal Müşteri Toplam Bakiye: {corporateTotalBalance:C}");
```

### Aggregate

`Aggregate` operatörü, bir koleksiyonda özel toplama işlemleri yapmanızı sağlar.

```csharp
// Tüm müşteri isimlerini birleştirme
string allCustomerNames = customers.Aggregate("", (result, c) => result + (result.Length > 0 ? ", " : "") + c.Name);

// Toplam pozitif ve negatif işlem tutarlarını hesaplama
var transactionStats = transactions.Aggregate(
    new { Positive = 0m, Negative = 0m },
    (result, t) => new
    {
        Positive = result.Positive + (t.Amount > 0 ? t.Amount : 0),
        Negative = result.Negative + (t.Amount < 0 ? t.Amount : 0)
    });

// Sonuçları yazdırma
Console.WriteLine($"Tüm Müşteriler: {allCustomerNames}");
Console.WriteLine($"Toplam Pozitif İşlem: {transactionStats.Positive:C}");
Console.WriteLine($"Toplam Negatif İşlem: {transactionStats.Negative:C}");
```

## Özet

Bu bölümde, LINQ'nun en yaygın kullanılan operatörlerini kategorilere ayırarak inceledik:

1. **Filtreleme Operatörleri**: Where, OfType
2. **Projeksiyon Operatörleri**: Select, SelectMany
3. **Sıralama Operatörleri**: OrderBy, OrderByDescending, ThenBy, ThenByDescending
4. **Gruplama Operatörleri**: GroupBy
5. **Küme Operatörleri**: Distinct, Union, Intersect, Except
6. **Niceleyici Operatörleri**: Any, All, Contains
7. **Bölümleme Operatörleri**: Take, Skip, TakeWhile, SkipWhile
8. **Toplama Operatörleri**: Count, Sum, Average, Min, Max, Aggregate

LINQ operatörleri, veri işleme görevlerini daha temiz, daha okunabilir ve daha bakımı kolay hale getirmenize yardımcı olur. Bu operatörleri etkin bir şekilde kullanarak, karmaşık veri işleme görevlerini daha az kod yazarak gerçekleştirebilirsiniz. 