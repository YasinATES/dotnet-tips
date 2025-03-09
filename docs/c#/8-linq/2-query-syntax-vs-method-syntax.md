# Query Syntax vs Method Syntax (Sorgu Sözdizimi ve Metot Sözdizimi)

LINQ, C# programlama dilinde veri sorgulama işlemleri için iki farklı sözdizimi sunar: Sorgu Sözdizimi (Query Syntax) ve Metot Sözdizimi (Method Syntax). Bu bölümde, bu iki sözdiziminin özelliklerini, farklarını ve kullanım senaryolarını inceleyeceğiz.

## 1. Query Expression Syntax (Sorgu İfadesi Sözdizimi)

Sorgu sözdizimi, SQL sorgularına benzer bir yapıya sahiptir ve genellikle daha okunabilir kabul edilir. Bu sözdizimi, `from`, `where`, `select`, `orderby`, `group by` gibi anahtar kelimeler kullanır.

```csharp
// Veri modelleri
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public string PhoneNumber { get; set; }
    public DateTime RegistrationDate { get; set; }
    public decimal TotalBalance { get; set; }
    public bool IsActive { get; set; }
    public string CustomerType { get; set; }
}

// Örnek veri
List<Customer> customers = new List<Customer>
{
    new Customer { Id = 1, Name = "Ali Yılmaz", Email = "ali.yilmaz@example.com", PhoneNumber = "5551234567", RegistrationDate = new DateTime(2020, 5, 12), TotalBalance = 15000, IsActive = true, CustomerType = "Individual" },
    new Customer { Id = 2, Name = "Ayşe Demir", Email = "ayse.demir@example.com", PhoneNumber = "5559876543", RegistrationDate = new DateTime(2019, 3, 8), TotalBalance = 28500, IsActive = true, CustomerType = "Individual" },
    new Customer { Id = 3, Name = "Mehmet Kaya", Email = "mehmet.kaya@example.com", PhoneNumber = "5554567890", RegistrationDate = new DateTime(2021, 1, 15), TotalBalance = 7200, IsActive = false, CustomerType = "Individual" },
    new Customer { Id = 4, Name = "ABC Şirketi", Email = "info@abccompany.com", PhoneNumber = "5551112233", RegistrationDate = new DateTime(2018, 11, 20), TotalBalance = 142000, IsActive = true, CustomerType = "Corporate" },
    new Customer { Id = 5, Name = "XYZ Ltd. Şti.", Email = "contact@xyzltd.com", PhoneNumber = "5554445566", RegistrationDate = new DateTime(2022, 2, 3), TotalBalance = 83500, IsActive = true, CustomerType = "Corporate" }
};

// Sorgu sözdizimi ile aktif müşterileri filtreleme
var activeCustomers = from c in customers
                     where c.IsActive
                     select c;

// Sorgu sözdizimi ile müşterileri bakiyelerine göre sıralama
var sortedCustomers = from c in customers
                     orderby c.TotalBalance descending
                     select c;

// Sorgu sözdizimi ile müşteri tipine göre gruplandırma
var customersByType = from c in customers
                     group c by c.CustomerType into g
                     select new
                     {
                         CustomerType = g.Key,
                         Count = g.Count(),
                         TotalBalance = g.Sum(c => c.TotalBalance)
                     };

// Sorgu sözdizimi ile birden fazla koşul kullanma
var filteredCustomers = from c in customers
                       where c.IsActive && c.TotalBalance > 10000
                       orderby c.Name
                       select new
                       {
                           CustomerName = c.Name,
                           Email = c.Email,
                           Balance = c.TotalBalance
                       };

// Sonuçları yazdırma
foreach (var customer in filteredCustomers)
{
    Console.WriteLine($"{customer.CustomerName} - {customer.Email} - {customer.Balance:C}");
}
```

## 2. Method Chain Syntax (Metot Zinciri Sözdizimi)

Metot sözdizimi, uzantı metotlarını kullanarak zincirleme çağrılar yapar. Bu sözdizimi, daha esnek ve güçlüdür, çünkü LINQ'nun tüm özelliklerine erişim sağlar.

```csharp
// Metot sözdizimi ile aktif müşterileri filtreleme
var activeCustomers = customers.Where(c => c.IsActive);

// Metot sözdizimi ile müşterileri bakiyelerine göre sıralama
var sortedCustomers = customers.OrderByDescending(c => c.TotalBalance);

// Metot sözdizimi ile müşteri tipine göre gruplandırma
var customersByType = customers
    .GroupBy(c => c.CustomerType)
    .Select(g => new
    {
        CustomerType = g.Key,
        Count = g.Count(),
        TotalBalance = g.Sum(c => c.TotalBalance)
    });

// Metot sözdizimi ile birden fazla koşul kullanma
var filteredCustomers = customers
    .Where(c => c.IsActive && c.TotalBalance > 10000)
    .OrderBy(c => c.Name)
    .Select(c => new
    {
        CustomerName = c.Name,
        Email = c.Email,
        Balance = c.TotalBalance
    });

// Sonuçları yazdırma
foreach (var customer in filteredCustomers)
{
    Console.WriteLine($"{customer.CustomerName} - {customer.Email} - {customer.Balance:C}");
}
```

## 3. Mixed Syntax Usage (Karışık Sözdizimi Kullanımı)

Bazı durumlarda, sorgu sözdizimi ve metot sözdizimini birlikte kullanmak daha uygun olabilir. Özellikle, sorgu sözdiziminin desteklemediği bazı operatörler için metot sözdizimini kullanabilirsiniz.

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

// Karışık sözdizimi kullanımı - sorgu sözdizimi ile başlayıp metot sözdizimi ile devam etme
var customerTransactions = from c in customers
                          where c.IsActive
                          select new
                          {
                              Customer = c,
                              Transactions = transactions.Where(t => t.CustomerId == c.Id).ToList()
                          };

// Karışık sözdizimi kullanımı - metot sözdizimi içinde sorgu sözdizimi kullanma
var transactionSummary = customers
    .Where(c => c.IsActive)
    .Select(c => new
    {
        CustomerName = c.Name,
        TransactionCount = transactions.Count(t => t.CustomerId == c.Id),
        Transactions = from t in transactions
                      where t.CustomerId == c.Id
                      orderby t.Date descending
                      select t
    });

// Sonuçları yazdırma
foreach (var item in transactionSummary)
{
    Console.WriteLine($"Müşteri: {item.CustomerName}, İşlem Sayısı: {item.TransactionCount}");
    
    foreach (var transaction in item.Transactions)
    {
        Console.WriteLine($"  - Tarih: {transaction.Date.ToShortDateString()}, Tutar: {transaction.Amount:C}, Açıklama: {transaction.Description}");
    }
    
    Console.WriteLine();
}
```

## 4. Query Expression Translation (Sorgu İfadesi Çevirisi)

Sorgu sözdizimi, derleme zamanında metot sözdizimi çağrılarına dönüştürülür. Bu, her iki sözdiziminin de aynı performansa sahip olduğu anlamına gelir.

```csharp
// Sorgu sözdizimi
var query1 = from c in customers
            where c.TotalBalance > 10000
            orderby c.Name
            select c;

// Yukarıdaki sorgu, derleme zamanında aşağıdaki metot sözdizimi çağrılarına dönüştürülür
var query2 = customers
    .Where(c => c.TotalBalance > 10000)
    .OrderBy(c => c.Name);

// Her iki sorgu da aynı sonucu üretir
Console.WriteLine("Sorgu Sözdizimi Sonuçları:");
foreach (var customer in query1)
{
    Console.WriteLine($"{customer.Name} - {customer.TotalBalance:C}");
}

Console.WriteLine("\nMetot Sözdizimi Sonuçları:");
foreach (var customer in query2)
{
    Console.WriteLine($"{customer.Name} - {customer.TotalBalance:C}");
}
```

## 5. Performance Implications (Performans Etkileri)

Sorgu sözdizimi ve metot sözdizimi arasında performans açısından önemli bir fark yoktur, çünkü sorgu sözdizimi derleme zamanında metot sözdizimi çağrılarına dönüştürülür. Ancak, bazı durumlarda metot sözdizimi daha verimli kod yazmanıza olanak tanır.

```csharp
// Hesap sınıfı
public class Account
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public string AccountNumber { get; set; }
    public decimal Balance { get; set; }
    public string AccountType { get; set; }
    public DateTime OpenDate { get; set; }
    public bool IsActive { get; set; }
}

// Örnek hesap verileri
List<Account> accounts = new List<Account>
{
    new Account { Id = 1, CustomerId = 1, AccountNumber = "1001234", Balance = 8500, AccountType = "Checking", OpenDate = new DateTime(2020, 6, 15), IsActive = true },
    new Account { Id = 2, CustomerId = 1, AccountNumber = "1001235", Balance = 6500, AccountType = "Savings", OpenDate = new DateTime(2021, 2, 10), IsActive = true },
    new Account { Id = 3, CustomerId = 2, AccountNumber = "2001234", Balance = 12500, AccountType = "Checking", OpenDate = new DateTime(2019, 4, 20), IsActive = true },
    new Account { Id = 4, CustomerId = 2, AccountNumber = "2001235", Balance = 16000, AccountType = "Savings", OpenDate = new DateTime(2020, 8, 5), IsActive = true },
    new Account { Id = 5, CustomerId = 3, AccountNumber = "3001234", Balance = 3200, AccountType = "Checking", OpenDate = new DateTime(2021, 3, 12), IsActive = false },
    new Account { Id = 6, CustomerId = 4, AccountNumber = "4001234", Balance = 142000, AccountType = "Business", OpenDate = new DateTime(2018, 12, 5), IsActive = true }
};

// Performans karşılaştırması için örnek
Console.WriteLine("Performans Karşılaştırması:");

// Senaryo 1: Büyük veri kümesinde filtreleme ve sıralama
var stopwatch1 = System.Diagnostics.Stopwatch.StartNew();

// Metot sözdizimi - erken filtreleme
var result1 = accounts
    .Where(a => a.IsActive) // Önce filtreleme
    .OrderBy(a => a.Balance) // Sonra sıralama
    .Select(a => new { a.AccountNumber, a.Balance });

stopwatch1.Stop();

var stopwatch2 = System.Diagnostics.Stopwatch.StartNew();

// Sorgu sözdizimi - aynı işlem
var result2 = from a in accounts
             where a.IsActive
             orderby a.Balance
             select new { a.AccountNumber, a.Balance };

stopwatch2.Stop();

Console.WriteLine($"Metot Sözdizimi: {stopwatch1.ElapsedTicks} ticks");
Console.WriteLine($"Sorgu Sözdizimi: {stopwatch2.ElapsedTicks} ticks");

// Senaryo 2: Zincirleme işlemler
var stopwatch3 = System.Diagnostics.Stopwatch.StartNew();

// Metot sözdizimi - zincirleme işlemler
var result3 = accounts
    .Where(a => a.IsActive)
    .GroupBy(a => a.AccountType)
    .Select(g => new
    {
        AccountType = g.Key,
        TotalBalance = g.Sum(a => a.Balance),
        AverageBalance = g.Average(a => a.Balance),
        AccountCount = g.Count()
    })
    .OrderByDescending(g => g.TotalBalance);

stopwatch3.Stop();

var stopwatch4 = System.Diagnostics.Stopwatch.StartNew();

// Sorgu sözdizimi - aynı işlem
var result4 = from a in accounts
             where a.IsActive
             group a by a.AccountType into g
             select new
             {
                 AccountType = g.Key,
                 TotalBalance = g.Sum(a => a.Balance),
                 AverageBalance = g.Average(a => a.Balance),
                 AccountCount = g.Count()
             } into result
             orderby result.TotalBalance descending
             select result;

stopwatch4.Stop();

Console.WriteLine($"Metot Sözdizimi (Zincirleme): {stopwatch3.ElapsedTicks} ticks");
Console.WriteLine($"Sorgu Sözdizimi (Zincirleme): {stopwatch4.ElapsedTicks} ticks");
```

## 6. Best Practice Scenarios (En İyi Uygulama Senaryoları)

Her iki sözdiziminin de avantajları ve dezavantajları vardır. Hangi sözdizimini kullanacağınız, genellikle kişisel tercih ve sorgunun karmaşıklığına bağlıdır.

### Sorgu Sözdiziminin Avantajlı Olduğu Durumlar

```csharp
// 1. Karmaşık sorgular
// Sorgu sözdizimi, karmaşık sorgularda daha okunabilir olabilir
var complexQuery = from c in customers
                  join a in accounts on c.Id equals a.CustomerId
                  where c.IsActive && a.IsActive
                  group new { Customer = c, Account = a } by c.CustomerType into g
                  select new
                  {
                      CustomerType = g.Key,
                      TotalCustomers = g.Select(x => x.Customer.Id).Distinct().Count(),
                      TotalAccounts = g.Count(),
                      TotalBalance = g.Sum(x => x.Account.Balance)
                  };

// 2. Çoklu join işlemleri
var multiJoinQuery = from c in customers
                    join a in accounts on c.Id equals a.CustomerId
                    join t in transactions on a.Id equals t.CustomerId
                    where c.IsActive && a.IsActive
                    select new
                    {
                        CustomerName = c.Name,
                        AccountNumber = a.AccountNumber,
                        TransactionDate = t.Date,
                        Amount = t.Amount,
                        Description = t.Description
                    };

// 3. let anahtar kelimesi kullanımı (ara değişkenler)
var letQuery = from c in customers
              let totalBalance = accounts.Where(a => a.CustomerId == c.Id).Sum(a => a.Balance)
              where totalBalance > 10000
              orderby totalBalance descending
              select new
              {
                  CustomerName = c.Name,
                  TotalBalance = totalBalance
              };
```

### Metot Sözdiziminin Avantajlı Olduğu Durumlar

```csharp
// 1. Sorgu sözdiziminin desteklemediği operatörler
// Örneğin: Skip, Take, SkipWhile, TakeWhile, Distinct, Concat, Zip, vb.
var methodOnlyQuery = customers
    .Where(c => c.IsActive)
    .OrderByDescending(c => c.TotalBalance)
    .Take(3) // Sorgu sözdizimi Take'i doğrudan desteklemez
    .Select(c => new { c.Name, c.TotalBalance });

// 2. Fonksiyonel programlama yaklaşımı
var functionalQuery = customers
    .Where(c => c.IsActive)
    .Select(c => new
    {
        Customer = c,
        Accounts = accounts.Where(a => a.CustomerId == c.Id).ToList()
    })
    .Where(x => x.Accounts.Any(a => a.Balance > 10000))
    .Select(x => new
    {
        CustomerName = x.Customer.Name,
        AccountCount = x.Accounts.Count,
        TotalBalance = x.Accounts.Sum(a => a.Balance)
    });

// 3. Dinamik sorgu oluşturma
Func<Customer, bool> customerFilter = null;

// Dinamik koşullar
bool filterByBalance = true;
bool filterByType = true;

if (filterByBalance)
{
    var balanceFilter = new Func<Customer, bool>(c => c.TotalBalance > 10000);
    customerFilter = customerFilter == null ? balanceFilter : c => customerFilter(c) && balanceFilter(c);
}

if (filterByType)
{
    var typeFilter = new Func<Customer, bool>(c => c.CustomerType == "Individual");
    customerFilter = customerFilter == null ? typeFilter : c => customerFilter(c) && typeFilter(c);
}

// Dinamik sorgu
var dynamicQuery = customers.Where(customerFilter ?? (c => true));
```

### Karışık Sözdizimi Kullanımı İçin En İyi Uygulamalar

```csharp
// 1. Sorgu sözdizimi ile başlayıp, metot sözdizimi ile devam etme
var mixedQuery1 = (from c in customers
                  where c.IsActive
                  select c)
                  .OrderByDescending(c => c.TotalBalance)
                  .Take(5);

// 2. Metot sözdizimi içinde sorgu sözdizimi kullanma
var mixedQuery2 = customers
    .Where(c => c.IsActive)
    .Select(c => new
    {
        Customer = c,
        ActiveAccounts = from a in accounts
                        where a.CustomerId == c.Id && a.IsActive
                        select a
    });

// 3. Karmaşık gruplandırma ve birleştirme işlemleri
var mixedQuery3 = (from c in customers
                  join a in accounts on c.Id equals a.CustomerId
                  where c.IsActive && a.IsActive
                  select new { Customer = c, Account = a })
                  .GroupBy(x => x.Customer.CustomerType)
                  .Select(g => new
                  {
                      CustomerType = g.Key,
                      TotalBalance = g.Sum(x => x.Account.Balance),
                      AccountCount = g.Count()
                  })
                  .OrderByDescending(x => x.TotalBalance);
```

## Özet

Bu bölümde, LINQ'nun iki farklı sözdizimini inceledik: Sorgu Sözdizimi (Query Syntax) ve Metot Sözdizimi (Method Syntax). Her iki sözdiziminin de avantajları ve dezavantajları vardır:

### Sorgu Sözdizimi (Query Syntax)
- SQL benzeri sözdizimi, SQL bilgisi olan geliştiriciler için daha tanıdık
- Karmaşık sorgularda daha okunabilir
- `let` anahtar kelimesi ile ara değişkenler tanımlama imkanı
- Çoklu join işlemlerinde daha temiz kod

### Metot Sözdizimi (Method Syntax)
- Daha esnek ve güçlü, LINQ'nun tüm operatörlerine erişim
- Fonksiyonel programlama yaklaşımına daha uygun
- Dinamik sorgu oluşturma için daha uygun
- IntelliSense desteği daha iyi

Hangi sözdizimini kullanacağınız, genellikle kişisel tercih ve sorgunun karmaşıklığına bağlıdır. Birçok durumda, her iki sözdizimini de karışık olarak kullanmak en iyi sonucu verebilir. Önemli olan, kodunuzun okunabilir, bakımı kolay ve verimli olmasıdır. 