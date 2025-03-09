# LINQ (Language Integrated Query)

LINQ, C# programlama dilinin en güçlü özelliklerinden biridir ve veri sorgulama işlemlerini doğrudan dil içerisinde yapabilmenizi sağlar. Bu bölümde, LINQ'nun temel kavramlarını ve kullanım şekillerini inceleyeceğiz.

## 1. LINQ to Objects

LINQ to Objects, bellek içi koleksiyonlar üzerinde sorgu yapmanızı sağlar. Bu, diziler, listeler, sözlükler ve diğer koleksiyon tipleri üzerinde çalışabilir.

### Where, Select, SelectMany

#### Where

`Where` metodu, bir koleksiyondaki öğeleri belirli bir koşula göre filtrelemenizi sağlar.

```csharp
// Müşteri sınıfı
public class Customer
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public DateTime RegistrationDate { get; set; }
    public decimal TotalBalance { get; set; }
    public bool IsActive { get; set; }
}

// Müşteri listesi
List<Customer> customers = new List<Customer>
{
    new Customer { Id = 1, FirstName = "Ali", LastName = "Yılmaz", Email = "ali.yilmaz@example.com", RegistrationDate = new DateTime(2020, 5, 12), TotalBalance = 15000, IsActive = true },
    new Customer { Id = 2, FirstName = "Ayşe", LastName = "Demir", Email = "ayse.demir@example.com", RegistrationDate = new DateTime(2019, 3, 8), TotalBalance = 28500, IsActive = true },
    new Customer { Id = 3, FirstName = "Mehmet", LastName = "Kaya", Email = "mehmet.kaya@example.com", RegistrationDate = new DateTime(2021, 1, 15), TotalBalance = 7200, IsActive = false },
    new Customer { Id = 4, FirstName = "Zeynep", LastName = "Çelik", Email = "zeynep.celik@example.com", RegistrationDate = new DateTime(2018, 11, 20), TotalBalance = 42000, IsActive = true },
    new Customer { Id = 5, FirstName = "Mustafa", LastName = "Şahin", Email = "mustafa.sahin@example.com", RegistrationDate = new DateTime(2022, 2, 3), TotalBalance = 3500, IsActive = true }
};

// Aktif müşterileri filtreleme
var activeCustomers = customers.Where(c => c.IsActive);

// Bakiyesi 10000'den yüksek olan müşterileri filtreleme
var highBalanceCustomers = customers.Where(c => c.TotalBalance > 10000);

// 2020 yılından sonra kaydolan aktif müşterileri filtreleme
var newActiveCustomers = customers.Where(c => c.RegistrationDate >= new DateTime(2020, 1, 1) && c.IsActive);

// Sonuçları yazdırma
foreach (var customer in newActiveCustomers)
{
    Console.WriteLine($"{customer.FirstName} {customer.LastName} - {customer.RegistrationDate.ToShortDateString()} - {customer.TotalBalance:C}");
}
```

#### Select

`Select` metodu, bir koleksiyondaki her öğeyi başka bir forma dönüştürmenizi sağlar.

```csharp
// Müşteri adlarını seçme
var customerNames = customers.Select(c => $"{c.FirstName} {c.LastName}");

// Müşteri özet bilgilerini seçme
var customerSummaries = customers.Select(c => new
{
    FullName = $"{c.FirstName} {c.LastName}",
    Email = c.Email,
    Balance = c.TotalBalance
});

// Sonuçları yazdırma
foreach (var summary in customerSummaries)
{
    Console.WriteLine($"{summary.FullName} - {summary.Email} - {summary.Balance:C}");
}
```

#### SelectMany

`SelectMany` metodu, iç içe koleksiyonları düzleştirmenizi sağlar.

```csharp
// Hesap sınıfı
public class Account
{
    public int Id { get; set; }
    public string AccountNumber { get; set; }
    public decimal Balance { get; set; }
    public string AccountType { get; set; }
    public DateTime OpenDate { get; set; }
}

// Müşteri sınıfını genişletme
public class CustomerWithAccounts : Customer
{
    public List<Account> Accounts { get; set; } = new List<Account>();
}

// Hesapları olan müşteri listesi
List<CustomerWithAccounts> customersWithAccounts = new List<CustomerWithAccounts>
{
    new CustomerWithAccounts
    {
        Id = 1,
        FirstName = "Ali",
        LastName = "Yılmaz",
        Email = "ali.yilmaz@example.com",
        IsActive = true,
        Accounts = new List<Account>
        {
            new Account { Id = 101, AccountNumber = "1001234", Balance = 8500, AccountType = "Vadesiz", OpenDate = new DateTime(2020, 6, 15) },
            new Account { Id = 102, AccountNumber = "1001235", Balance = 6500, AccountType = "Vadeli", OpenDate = new DateTime(2021, 2, 10) }
        }
    },
    new CustomerWithAccounts
    {
        Id = 2,
        FirstName = "Ayşe",
        LastName = "Demir",
        Email = "ayse.demir@example.com",
        IsActive = true,
        Accounts = new List<Account>
        {
            new Account { Id = 201, AccountNumber = "2001234", Balance = 12500, AccountType = "Vadesiz", OpenDate = new DateTime(2019, 4, 20) },
            new Account { Id = 202, AccountNumber = "2001235", Balance = 16000, AccountType = "Vadeli", OpenDate = new DateTime(2020, 8, 5) }
        }
    }
};

// Tüm hesapları düzleştirme
var allAccounts = customersWithAccounts.SelectMany(c => c.Accounts);

// Müşteri bilgisiyle birlikte tüm hesapları düzleştirme
var customerAccounts = customersWithAccounts.SelectMany(c => c.Accounts, (customer, account) => new
{
    CustomerName = $"{customer.FirstName} {customer.LastName}",
    AccountNumber = account.AccountNumber,
    Balance = account.Balance,
    AccountType = account.AccountType
});

// Sonuçları yazdırma
foreach (var account in customerAccounts)
{
    Console.WriteLine($"{account.CustomerName} - {account.AccountNumber} - {account.AccountType} - {account.Balance:C}");
}
```

### OrderBy, ThenBy

`OrderBy` ve `ThenBy` metotları, koleksiyonları sıralamak için kullanılır.

```csharp
// İşlem sınıfı
public class Transaction
{
    public int Id { get; set; }
    public int AccountId { get; set; }
    public DateTime Date { get; set; }
    public decimal Amount { get; set; }
    public string Description { get; set; }
    public string TransactionType { get; set; }
}

// İşlem listesi
List<Transaction> transactions = new List<Transaction>
{
    new Transaction { Id = 1, AccountId = 101, Date = new DateTime(2023, 1, 15), Amount = 1500, Description = "Maaş", TransactionType = "Yatırma" },
    new Transaction { Id = 2, AccountId = 101, Date = new DateTime(2023, 1, 20), Amount = -200, Description = "Market alışverişi", TransactionType = "Çekme" },
    new Transaction { Id = 3, AccountId = 102, Date = new DateTime(2023, 1, 18), Amount = 3000, Description = "Havale", TransactionType = "Yatırma" },
    new Transaction { Id = 4, AccountId = 201, Date = new DateTime(2023, 1, 10), Amount = -150, Description = "Fatura ödemesi", TransactionType = "Çekme" },
    new Transaction { Id = 5, AccountId = 201, Date = new DateTime(2023, 1, 25), Amount = -500, Description = "Kira ödemesi", TransactionType = "Çekme" },
    new Transaction { Id = 6, AccountId = 102, Date = new DateTime(2023, 1, 22), Amount = -350, Description = "Alışveriş", TransactionType = "Çekme" }
};

// İşlemleri tarihe göre sıralama
var transactionsByDate = transactions.OrderBy(t => t.Date);

// İşlemleri hesap ID'sine göre sıralama, ardından tarihe göre sıralama
var transactionsByAccountAndDate = transactions.OrderBy(t => t.AccountId).ThenBy(t => t.Date);

// İşlemleri hesap ID'sine göre sıralama, ardından tutara göre azalan sıralama
var transactionsByAccountAndAmount = transactions.OrderBy(t => t.AccountId).ThenByDescending(t => t.Amount);

// Sonuçları yazdırma
Console.WriteLine("Hesap ve Tutara Göre Sıralı İşlemler:");
foreach (var transaction in transactionsByAccountAndAmount)
{
    Console.WriteLine($"Hesap: {transaction.AccountId}, Tarih: {transaction.Date.ToShortDateString()}, Tutar: {transaction.Amount:C}, Açıklama: {transaction.Description}");
}
```

### GroupBy, Join, GroupJoin

#### GroupBy

`GroupBy` metodu, koleksiyondaki öğeleri belirli bir özelliğe göre gruplandırmanızı sağlar.

```csharp
// İşlemleri işlem tipine göre gruplandırma
var transactionsByType = transactions.GroupBy(t => t.TransactionType);

// Sonuçları yazdırma
foreach (var group in transactionsByType)
{
    Console.WriteLine($"İşlem Tipi: {group.Key}");
    Console.WriteLine($"İşlem Sayısı: {group.Count()}");
    Console.WriteLine($"Toplam Tutar: {group.Sum(t => t.Amount):C}");
    Console.WriteLine();
}

// İşlemleri hesap ID'sine göre gruplandırma ve özet bilgileri hesaplama
var transactionSummaryByAccount = transactions
    .GroupBy(t => t.AccountId)
    .Select(g => new
    {
        AccountId = g.Key,
        TransactionCount = g.Count(),
        TotalDeposit = g.Where(t => t.Amount > 0).Sum(t => t.Amount),
        TotalWithdrawal = g.Where(t => t.Amount < 0).Sum(t => t.Amount),
        NetAmount = g.Sum(t => t.Amount)
    });

// Sonuçları yazdırma
Console.WriteLine("Hesap Bazlı İşlem Özeti:");
foreach (var summary in transactionSummaryByAccount)
{
    Console.WriteLine($"Hesap: {summary.AccountId}");
    Console.WriteLine($"İşlem Sayısı: {summary.TransactionCount}");
    Console.WriteLine($"Toplam Yatırma: {summary.TotalDeposit:C}");
    Console.WriteLine($"Toplam Çekme: {summary.TotalWithdrawal:C}");
    Console.WriteLine($"Net Tutar: {summary.NetAmount:C}");
    Console.WriteLine();
}
```

#### Join

`Join` metodu, iki koleksiyonu ortak bir anahtar üzerinden birleştirmenizi sağlar.

```csharp
// Hesap ve işlem listelerini birleştirme
var accountTransactions = accounts.Join(
    transactions,
    account => account.Id,
    transaction => transaction.AccountId,
    (account, transaction) => new
    {
        AccountNumber = account.AccountNumber,
        AccountType = account.AccountType,
        TransactionDate = transaction.Date,
        Amount = transaction.Amount,
        Description = transaction.Description
    });

// Sonuçları yazdırma
Console.WriteLine("Hesap ve İşlem Detayları:");
foreach (var item in accountTransactions)
{
    Console.WriteLine($"Hesap No: {item.AccountNumber}, Tip: {item.AccountType}, Tarih: {item.TransactionDate.ToShortDateString()}, Tutar: {item.Amount:C}, Açıklama: {item.Description}");
}
```

#### GroupJoin

`GroupJoin` metodu, iki koleksiyonu birleştirirken, ikinci koleksiyondaki eşleşen tüm öğeleri bir grup olarak almanızı sağlar.

```csharp
// Hesapları ve ilgili işlemleri birleştirme
var accountsWithTransactions = accounts.GroupJoin(
    transactions,
    account => account.Id,
    transaction => transaction.AccountId,
    (account, accountTransactions) => new
    {
        AccountNumber = account.AccountNumber,
        AccountType = account.AccountType,
        Balance = account.Balance,
        Transactions = accountTransactions.ToList()
    });

// Sonuçları yazdırma
Console.WriteLine("Hesaplar ve İlgili İşlemler:");
foreach (var account in accountsWithTransactions)
{
    Console.WriteLine($"Hesap No: {account.AccountNumber}, Tip: {account.AccountType}, Bakiye: {account.Balance:C}");
    Console.WriteLine($"İşlem Sayısı: {account.Transactions.Count}");
    
    foreach (var transaction in account.Transactions)
    {
        Console.WriteLine($"  - Tarih: {transaction.Date.ToShortDateString()}, Tutar: {transaction.Amount:C}, Açıklama: {transaction.Description}");
    }
    
    Console.WriteLine();
}
```

### First, Single, Last

Bu metotlar, koleksiyondan belirli koşullara uyan öğeleri almanızı sağlar.

```csharp
// Müşteri listesinden ilk öğeyi alma
var firstCustomer = customers.First();

// Belirli bir koşula uyan ilk müşteri
var firstHighBalanceCustomer = customers.First(c => c.TotalBalance > 20000);

// Belirli bir koşula uyan ilk müşteri (yoksa null döner)
var firstInactiveCustomer = customers.FirstOrDefault(c => !c.IsActive);

// Belirli bir koşula uyan tek müşteri
try
{
    var singleCustomer = customers.Single(c => c.Id == 3);
    Console.WriteLine($"Bulunan müşteri: {singleCustomer.FirstName} {singleCustomer.LastName}");
}
catch (InvalidOperationException)
{
    Console.WriteLine("Koşula uyan birden fazla müşteri var veya hiç müşteri yok.");
}

// Belirli bir koşula uyan tek müşteri (yoksa null döner)
var singleOrDefaultCustomer = customers.SingleOrDefault(c => c.Email == "zeynep.celik@example.com");

// Koleksiyondaki son öğe
var lastCustomer = customers.Last();

// Belirli bir koşula uyan son müşteri
var lastActiveCustomer = customers.Last(c => c.IsActive);
```

### Any, All, Contains

Bu metotlar, koleksiyondaki öğelerin belirli koşulları sağlayıp sağlamadığını kontrol etmenizi sağlar.

```csharp
// Aktif olmayan müşteri var mı?
bool hasInactiveCustomers = customers.Any(c => !c.IsActive);

// Tüm müşterilerin bakiyesi 1000'den büyük mü?
bool allCustomersHaveHighBalance = customers.All(c => c.TotalBalance > 1000);

// Belirli bir e-posta adresine sahip müşteri var mı?
bool containsEmail = customers.Any(c => c.Email == "mehmet.kaya@example.com");

// Belirli bir müşteri listede var mı?
Customer customerToFind = new Customer { Id = 2, FirstName = "Ayşe", LastName = "Demir" };
bool containsCustomer = customers.Contains(customerToFind, new CustomerComparer());

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
```

### Count, Sum, Average, Min, Max

Bu metotlar, koleksiyon üzerinde sayısal işlemler yapmanızı sağlar.

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

// En yüksek bakiyeye sahip müşteri
var customerWithMaxBalance = customers.OrderByDescending(c => c.TotalBalance).First();

// İstatistikleri yazdırma
Console.WriteLine("Müşteri İstatistikleri:");
Console.WriteLine($"Toplam Müşteri: {totalCustomers}");
Console.WriteLine($"Aktif Müşteri: {activeCustomers}");
Console.WriteLine($"Toplam Bakiye: {totalBalance:C}");
Console.WriteLine($"Ortalama Bakiye: {averageBalance:C}");
Console.WriteLine($"En Düşük Bakiye: {minBalance:C}");
Console.WriteLine($"En Yüksek Bakiye: {maxBalance:C}");
Console.WriteLine($"En Yüksek Bakiyeli Müşteri: {customerWithMaxBalance.FirstName} {customerWithMaxBalance.LastName} - {customerWithMaxBalance.TotalBalance:C}");
```

## Sorgu Sözdizimi

LINQ, iki farklı sözdizimi sunar: metot sözdizimi ve sorgu sözdizimi. Yukarıdaki örneklerde metot sözdizimini kullandık. Aşağıda, aynı işlemleri sorgu sözdizimi ile nasıl yapabileceğinizi görebilirsiniz.

```csharp
// Metot sözdizimi
var highBalanceCustomers = customers.Where(c => c.TotalBalance > 10000);

// Sorgu sözdizimi
var highBalanceCustomersQuery = from c in customers
                               where c.TotalBalance > 10000
                               select c;

// Metot sözdizimi - sıralama ve dönüştürme
var customerSummaries = customers
    .Where(c => c.IsActive)
    .OrderBy(c => c.LastName)
    .Select(c => new
    {
        FullName = $"{c.FirstName} {c.LastName}",
        Email = c.Email,
        Balance = c.TotalBalance
    });

// Sorgu sözdizimi - sıralama ve dönüştürme
var customerSummariesQuery = from c in customers
                            where c.IsActive
                            orderby c.LastName
                            select new
                            {
                                FullName = $"{c.FirstName} {c.LastName}",
                                Email = c.Email,
                                Balance = c.TotalBalance
                            };

// Metot sözdizimi - gruplama
var transactionsByType = transactions
    .GroupBy(t => t.TransactionType)
    .Select(g => new
    {
        Type = g.Key,
        Count = g.Count(),
        Total = g.Sum(t => t.Amount)
    });

// Sorgu sözdizimi - gruplama
var transactionsByTypeQuery = from t in transactions
                             group t by t.TransactionType into g
                             select new
                             {
                                 Type = g.Key,
                                 Count = g.Count(),
                                 Total = g.Sum(t => t.Amount)
                             };

// Metot sözdizimi - join
var accountTransactions = accounts.Join(
    transactions,
    account => account.Id,
    transaction => transaction.AccountId,
    (account, transaction) => new
    {
        AccountNumber = account.AccountNumber,
        TransactionDate = transaction.Date,
        Amount = transaction.Amount
    });

// Sorgu sözdizimi - join
var accountTransactionsQuery = from account in accounts
                              join transaction in transactions
                              on account.Id equals transaction.AccountId
                              select new
                              {
                                  AccountNumber = account.AccountNumber,
                                  TransactionDate = transaction.Date,
                                  Amount = transaction.Amount
                              };
```

## Ertelenmiş Yürütme (Deferred Execution)

LINQ sorguları, ertelenmiş yürütme özelliğine sahiptir. Bu, sorgunun tanımlandığı anda değil, sonuçların kullanıldığı anda yürütüldüğü anlamına gelir.

```csharp
// Sorgu tanımı
var activeCustomersQuery = customers.Where(c => c.IsActive);

// Bu noktada sorgu henüz yürütülmedi

// Listeye yeni bir müşteri ekleme
customers.Add(new Customer
{
    Id = 6,
    FirstName = "Fatma",
    LastName = "Öztürk",
    Email = "fatma.ozturk@example.com",
    RegistrationDate = DateTime.Now,
    TotalBalance = 9500,
    IsActive = true
});

// Sorgu sonuçlarını kullanma - bu noktada sorgu yürütülür
// Yeni eklenen müşteri de sonuçlara dahil edilir
foreach (var customer in activeCustomersQuery)
{
    Console.WriteLine($"{customer.FirstName} {customer.LastName}");
}

// Sorguyu hemen yürütmek için ToList(), ToArray() veya benzeri metotlar kullanılabilir
var activeCustomersList = customers.Where(c => c.IsActive).ToList();

// Bu noktada sorgu yürütüldü ve sonuçlar bir listeye dönüştürüldü
// Artık orijinal koleksiyondaki değişiklikler bu listeyi etkilemez
```

## Özet

Bu bölümde, LINQ to Objects'in temel özelliklerini ve kullanım şekillerini inceledik. LINQ, C# dilinin en güçlü özelliklerinden biridir ve veri işleme görevlerini daha temiz, daha okunabilir ve daha bakımı kolay hale getirmenize yardımcı olur.

LINQ, aşağıdaki avantajları sağlar:

- Daha az kod yazarak daha fazla iş yapabilirsiniz.
- Kodunuz daha okunabilir ve anlaşılır olur.
- Tip güvenliği sağlar, derleme zamanında hataları yakalamanıza yardımcı olur.
- Farklı veri kaynakları için tutarlı bir programlama modeli sunar.
- Performans optimizasyonları içerir.

LINQ'yu etkin bir şekilde kullanarak, veri işleme görevlerinizi daha verimli ve etkili bir şekilde gerçekleştirebilirsiniz. 