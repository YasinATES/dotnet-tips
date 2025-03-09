# Custom LINQ Operators (Özel LINQ Operatörleri)

LINQ, zengin bir operatör seti sunmasına rağmen, bazen özel ihtiyaçlarınız için kendi LINQ operatörlerinizi oluşturmanız gerekebilir. Bu bölümde, özel LINQ operatörleri oluşturma tekniklerini ve bunların gerçek dünya uygulamalarını inceleyeceğiz.

## 1. Extension Method Pattern (Uzantı Metodu Deseni)

LINQ operatörleri, uzantı metotları olarak uygulanır. Kendi LINQ operatörlerinizi oluşturmak için, aynı deseni takip etmelisiniz.

```csharp
// Temel veri modelleri
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
    public decimal TotalBalance { get; set; }
    public bool IsActive { get; set; }
    public DateTime RegistrationDate { get; set; }
    public string CustomerType { get; set; }
    public List<Account> Accounts { get; set; } = new List<Account>();
}

public class Account
{
    public int Id { get; set; }
    public string AccountNumber { get; set; }
    public decimal Balance { get; set; }
    public string AccountType { get; set; }
    public DateTime OpenDate { get; set; }
    public bool IsActive { get; set; }
    public int CustomerId { get; set; }
}

public class Transaction
{
    public int Id { get; set; }
    public int AccountId { get; set; }
    public DateTime Date { get; set; }
    public decimal Amount { get; set; }
    public string Description { get; set; }
    public string TransactionType { get; set; }
}

// Basit uzantı metodu örneği
public static class CustomerExtensions
{
    // VIP müşterileri filtreleme
    public static IEnumerable<Customer> WhereVip(this IEnumerable<Customer> source, decimal threshold = 25000)
    {
        // Null kontrolü
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        // Filtreleme işlemi
        foreach (var customer in source)
        {
            if (customer.TotalBalance > threshold && customer.IsActive)
            {
                yield return customer;
            }
        }
    }
    
    // Müşterileri toplam bakiyelerine göre sıralama
    public static IOrderedEnumerable<Customer> OrderByTotalBalance(this IEnumerable<Customer> source)
    {
        // Null kontrolü
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        // Sıralama işlemi
        return source.OrderByDescending(c => c.TotalBalance);
    }
}

// Kullanım örneği
List<Customer> customers = GetCustomers(); // Müşteri listesini alma
var vipCustomers = customers.WhereVip(30000);
var sortedVipCustomers = vipCustomers.OrderByTotalBalance();

foreach (var customer in sortedVipCustomers)
{
    Console.WriteLine($"{customer.Name} - {customer.TotalBalance:C}");
}
```

## 2. Yield Return Usage (Yield Return Kullanımı)

`yield return` ifadesi, LINQ operatörlerinin tembel değerlendirme (lazy evaluation) özelliğini sağlamak için kullanılır. Bu, büyük veri kümeleriyle çalışırken bellek kullanımını optimize etmenize yardımcı olur.

```csharp
// Yield return kullanımı
public static class TransactionExtensions
{
    // Belirli bir tutarın üzerindeki işlemleri filtreleme
    public static IEnumerable<Transaction> WhereAmountGreaterThan(
        this IEnumerable<Transaction> source, 
        decimal minAmount)
    {
        // Null kontrolü
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        // Tembel değerlendirme ile filtreleme
        foreach (var transaction in source)
        {
            // Bu kod, her öğe istendiğinde çalıştırılır
            if (transaction.Amount > minAmount)
            {
                // Sonraki öğeyi döndür ve durumu koru
                yield return transaction;
            }
        }
    }
    
    // Belirli bir tarih aralığındaki işlemleri filtreleme
    public static IEnumerable<Transaction> WhereDateBetween(
        this IEnumerable<Transaction> source,
        DateTime startDate,
        DateTime endDate)
    {
        // Null kontrolü
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        // Tarih kontrolü
        if (startDate > endDate)
            throw new ArgumentException("Başlangıç tarihi, bitiş tarihinden sonra olamaz.");
            
        // Tembel değerlendirme ile filtreleme
        foreach (var transaction in source)
        {
            if (transaction.Date >= startDate && transaction.Date <= endDate)
            {
                yield return transaction;
            }
        }
    }
}

// Kullanım örneği
List<Transaction> transactions = GetTransactions(); // İşlem listesini alma

// Tembel değerlendirme - bu noktada hiçbir işlem yapılmaz
var largeTransactions = transactions.WhereAmountGreaterThan(1000);
var recentLargeTransactions = largeTransactions.WhereDateBetween(
    DateTime.Now.AddMonths(-1),
    DateTime.Now
);

// Sonuçları kullanma - bu noktada filtreleme yapılır
foreach (var transaction in recentLargeTransactions)
{
    Console.WriteLine($"{transaction.Date.ToShortDateString()} - {transaction.Amount:C} - {transaction.Description}");
}
```

### Yield Return vs Normal Return

```csharp
// Yield return ile tembel değerlendirme
public static IEnumerable<Transaction> GetLargeTransactionsLazy(
    IEnumerable<Transaction> source, 
    decimal minAmount)
{
    Console.WriteLine("Tembel değerlendirme başladı");
    
    foreach (var transaction in source)
    {
        Console.WriteLine($"İşlem değerlendiriliyor: {transaction.Id}");
        
        if (transaction.Amount > minAmount)
        {
            Console.WriteLine($"İşlem döndürülüyor: {transaction.Id}");
            yield return transaction;
        }
    }
    
    Console.WriteLine("Tembel değerlendirme tamamlandı");
}

// Normal return ile anında değerlendirme
public static List<Transaction> GetLargeTransactionsEager(
    IEnumerable<Transaction> source, 
    decimal minAmount)
{
    Console.WriteLine("Anında değerlendirme başladı");
    
    var result = new List<Transaction>();
    
    foreach (var transaction in source)
    {
        Console.WriteLine($"İşlem değerlendiriliyor: {transaction.Id}");
        
        if (transaction.Amount > minAmount)
        {
            Console.WriteLine($"İşlem listeye ekleniyor: {transaction.Id}");
            result.Add(transaction);
        }
    }
    
    Console.WriteLine("Anında değerlendirme tamamlandı");
    return result;
}

// Kullanım karşılaştırması
var transactions = GetTransactions();

// Tembel değerlendirme
Console.WriteLine("--- Tembel Değerlendirme ---");
var lazyResult = GetLargeTransactionsLazy(transactions, 1000);
Console.WriteLine("Sorgu tanımlandı, ancak henüz yürütülmedi");

// İlk öğeyi alma
Console.WriteLine("\nİlk öğeyi alma:");
var firstLarge = lazyResult.FirstOrDefault();
Console.WriteLine($"İlk büyük işlem: {firstLarge?.Id}");

// Tüm sonuçları alma
Console.WriteLine("\nTüm sonuçları alma:");
foreach (var transaction in lazyResult)
{
    Console.WriteLine($"İşlem: {transaction.Id}");
}

// Anında değerlendirme
Console.WriteLine("\n--- Anında Değerlendirme ---");
var eagerResult = GetLargeTransactionsEager(transactions, 1000);
Console.WriteLine("Sorgu yürütüldü ve sonuçlar döndürüldü");

// Sonuçları kullanma
Console.WriteLine("\nSonuçları kullanma:");
foreach (var transaction in eagerResult)
{
    Console.WriteLine($"İşlem: {transaction.Id}");
}
```

## 3. Query Operator Design (Sorgu Operatörü Tasarımı)

Etkili LINQ operatörleri tasarlamak için, belirli desenleri ve en iyi uygulamaları takip etmelisiniz.

### Zincirlenebilir Operatörler

```csharp
// Zincirlenebilir operatörler
public static class AccountExtensions
{
    // Aktif hesapları filtreleme
    public static IEnumerable<Account> WhereActive(this IEnumerable<Account> source)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        return source.Where(a => a.IsActive);
    }
    
    // Belirli bir bakiyenin üzerindeki hesapları filtreleme
    public static IEnumerable<Account> WhereBalanceGreaterThan(
        this IEnumerable<Account> source, 
        decimal minBalance)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        return source.Where(a => a.Balance > minBalance);
    }
    
    // Belirli bir hesap türüne sahip hesapları filtreleme
    public static IEnumerable<Account> WhereAccountType(
        this IEnumerable<Account> source, 
        string accountType)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        if (string.IsNullOrEmpty(accountType))
            return source;
            
        return source.Where(a => a.AccountType == accountType);
    }
    
    // Hesap özeti oluşturma
    public static IEnumerable<AccountSummary> ToAccountSummary(this IEnumerable<Account> source)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        return source.Select(a => new AccountSummary
        {
            AccountNumber = a.AccountNumber,
            AccountType = a.AccountType,
            Balance = a.Balance,
            IsActive = a.IsActive
        });
    }
}

// Hesap özeti sınıfı
public class AccountSummary
{
    public string AccountNumber { get; set; }
    public string AccountType { get; set; }
    public decimal Balance { get; set; }
    public bool IsActive { get; set; }
}

// Kullanım örneği - operatörleri zincirleme
List<Account> accounts = GetAccounts(); // Hesap listesini alma

var activeSavingsAccounts = accounts
    .WhereActive()
    .WhereAccountType("Savings")
    .WhereBalanceGreaterThan(5000)
    .ToAccountSummary();

foreach (var account in activeSavingsAccounts)
{
    Console.WriteLine($"{account.AccountNumber} - {account.AccountType} - {account.Balance:C}");
}
```

### Parametre Doğrulama

```csharp
// Parametre doğrulama
public static class ValidationExtensions
{
    // Null olmayan değerleri filtreleme
    public static IEnumerable<T> WhereNotNull<T>(this IEnumerable<T> source)
        where T : class
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        foreach (var item in source)
        {
            if (item != null)
            {
                yield return item;
            }
        }
    }
    
    // Geçerli e-posta adreslerini filtreleme
    public static IEnumerable<Customer> WhereValidEmail(this IEnumerable<Customer> source)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        foreach (var customer in source)
        {
            if (customer != null && IsValidEmail(customer.Email))
            {
                yield return customer;
            }
        }
    }
    
    // E-posta doğrulama yardımcı metodu
    private static bool IsValidEmail(string email)
    {
        if (string.IsNullOrEmpty(email))
            return false;
            
        // Basit e-posta doğrulama
        return email.Contains("@") && email.Contains(".");
    }
}

// Kullanım örneği
List<Customer> customers = GetCustomers(); // Müşteri listesini alma

var validCustomers = customers
    .WhereNotNull()
    .WhereValidEmail();

foreach (var customer in validCustomers)
{
    Console.WriteLine($"{customer.Name} - {customer.Email}");
}
```

## 4. Performance Optimization (Performans Optimizasyonu)

Özel LINQ operatörleri oluştururken, performansı göz önünde bulundurmanız önemlidir. İşte bazı performans optimizasyon teknikleri:

### Erken Filtreleme

```csharp
// Erken filtreleme
public static class PerformanceExtensions
{
    // Kötü performans - tüm öğeler dönüştürülür, sonra filtrelenir
    public static IEnumerable<AccountSummary> ToSummaryThenFilter(
        this IEnumerable<Account> source,
        decimal minBalance)
    {
        // Önce dönüştürme
        var summaries = source.Select(a => new AccountSummary
        {
            AccountNumber = a.AccountNumber,
            AccountType = a.AccountType,
            Balance = a.Balance,
            IsActive = a.IsActive
        });
        
        // Sonra filtreleme
        return summaries.Where(s => s.Balance > minBalance);
    }
    
    // İyi performans - önce filtreleme, sonra dönüştürme
    public static IEnumerable<AccountSummary> FilterThenToSummary(
        this IEnumerable<Account> source,
        decimal minBalance)
    {
        // Önce filtreleme
        var filtered = source.Where(a => a.Balance > minBalance);
        
        // Sonra dönüştürme
        return filtered.Select(a => new AccountSummary
        {
            AccountNumber = a.AccountNumber,
            AccountType = a.AccountType,
            Balance = a.Balance,
            IsActive = a.IsActive
        });
    }
}

// Performans karşılaştırması
List<Account> accounts = GetLargeAccountList(); // Büyük hesap listesi alma

var stopwatch1 = System.Diagnostics.Stopwatch.StartNew();
var result1 = accounts.ToSummaryThenFilter(10000).ToList();
stopwatch1.Stop();

var stopwatch2 = System.Diagnostics.Stopwatch.StartNew();
var result2 = accounts.FilterThenToSummary(10000).ToList();
stopwatch2.Stop();

Console.WriteLine($"Kötü performans: {stopwatch1.ElapsedMilliseconds} ms");
Console.WriteLine($"İyi performans: {stopwatch2.ElapsedMilliseconds} ms");
```

### Önbelleğe Alma

```csharp
// Önbelleğe alma
public static class CachingExtensions
{
    // Sonuçları önbelleğe alma
    public static IEnumerable<T> Cache<T>(this IEnumerable<T> source)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        // Sonuçları bir listeye dönüştürerek önbelleğe alma
        return source.ToList();
    }
    
    // Koşullu önbelleğe alma
    public static IEnumerable<T> CacheIf<T>(
        this IEnumerable<T> source,
        bool condition)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        return condition ? source.ToList() : source;
    }
}

// Kullanım örneği
List<Transaction> transactions = GetTransactions(); // İşlem listesini alma

// Çoklu numaralandırma için önbelleğe alma
var recentTransactions = transactions
    .Where(t => t.Date >= DateTime.Now.AddDays(-30))
    .Cache(); // Sonuçları önbelleğe alma

// İlk kullanım - sorgu yürütülür
var count = recentTransactions.Count();
Console.WriteLine($"Son 30 günde yapılan işlem sayısı: {count}");

// İkinci kullanım - önbelleğe alınmış sonuçlar kullanılır
var total = recentTransactions.Sum(t => t.Amount);
Console.WriteLine($"Son 30 günde yapılan işlemlerin toplam tutarı: {total:C}");
```

## 5. Error Handling (Hata İşleme)

Özel LINQ operatörleri oluştururken, hataları düzgün bir şekilde işlemek önemlidir.

```csharp
// Hata işleme
public static class ErrorHandlingExtensions
{
    // Güvenli dönüşüm - hataları yakalar ve raporlar
    public static IEnumerable<TResult> SafeSelect<TSource, TResult>(
        this IEnumerable<TSource> source,
        Func<TSource, TResult> selector,
        Action<TSource, Exception> errorHandler = null)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        if (selector == null)
            throw new ArgumentNullException(nameof(selector));
            
        foreach (var item in source)
        {
            TResult result;
            
            try
            {
                result = selector(item);
            }
            catch (Exception ex)
            {
                errorHandler?.Invoke(item, ex);
                continue;
            }
            
            yield return result;
        }
    }
    
    // Güvenli filtreleme - hataları yakalar ve raporlar
    public static IEnumerable<T> SafeWhere<T>(
        this IEnumerable<T> source,
        Func<T, bool> predicate,
        Action<T, Exception> errorHandler = null)
    {
        if (source == null)
            throw new ArgumentNullException(nameof(source));
            
        if (predicate == null)
            throw new ArgumentNullException(nameof(predicate));
            
        foreach (var item in source)
        {
            bool shouldInclude;
            
            try
            {
                shouldInclude = predicate(item);
            }
            catch (Exception ex)
            {
                errorHandler?.Invoke(item, ex);
                continue;
            }
            
            if (shouldInclude)
            {
                yield return item;
            }
        }
    }
}

// Kullanım örneği
List<Transaction> transactions = GetTransactions(); // İşlem listesini alma

// Güvenli dönüşüm
var descriptions = transactions.SafeSelect(
    t => GetDetailedDescription(t), // Potansiyel olarak hata fırlatabilir
    (t, ex) => Console.WriteLine($"İşlem {t.Id} için açıklama alınamadı: {ex.Message}")
);

// Güvenli filtreleme
var largeTransactions = transactions.SafeWhere(
    t => IsLargeTransaction(t), // Potansiyel olarak hata fırlatabilir
    (t, ex) => Console.WriteLine($"İşlem {t.Id} değerlendirilemedi: {ex.Message}")
);

// Yardımcı metotlar
string GetDetailedDescription(Transaction transaction)
{
    // Gerçek uygulamada, bu bir veritabanı çağrısı veya karmaşık bir işlem olabilir
    if (transaction == null)
        throw new ArgumentNullException(nameof(transaction));
        
    return $"{transaction.Date.ToShortDateString()} - {transaction.Amount:C} - {transaction.Description}";
}

bool IsLargeTransaction(Transaction transaction)
{
    // Gerçek uygulamada, bu bir veritabanı çağrısı veya karmaşık bir işlem olabilir
    if (transaction == null)
        throw new ArgumentNullException(nameof(transaction));
        
    return transaction.Amount > 10000;
}
```

## 6. Operator Composition (Operatör Kompozisyonu)

Karmaşık işlevselliği elde etmek için basit operatörleri birleştirerek daha karmaşık operatörler oluşturabilirsiniz.

```csharp
// Operatör kompozisyonu
public static class CompositionExtensions
{
    // Temel operatörler
    public static IEnumerable<Transaction> WhereDeposit(this IEnumerable<Transaction> source)
    {
        return source.Where(t => t.Amount > 0);
    }
    
    public static IEnumerable<Transaction> WhereWithdrawal(this IEnumerable<Transaction> source)
    {
        return source.Where(t => t.Amount < 0);
    }
    
    public static IEnumerable<Transaction> WhereRecent(
        this IEnumerable<Transaction> source,
        int days)
    {
        var cutoffDate = DateTime.Now.AddDays(-days);
        return source.Where(t => t.Date >= cutoffDate);
    }
    
    // Kompozit operatörler
    public static IEnumerable<Transaction> WhereRecentDeposit(
        this IEnumerable<Transaction> source,
        int days)
    {
        // Temel operatörleri birleştirme
        return source
            .WhereDeposit()
            .WhereRecent(days);
    }
    
    public static IEnumerable<Transaction> WhereRecentWithdrawal(
        this IEnumerable<Transaction> source,
        int days)
    {
        // Temel operatörleri birleştirme
        return source
            .WhereWithdrawal()
            .WhereRecent(days);
    }
    
    // Daha karmaşık kompozit operatör
    public static IEnumerable<TransactionSummary> GetRecentTransactionSummary(
        this IEnumerable<Transaction> source,
        int days)
    {
        var cutoffDate = DateTime.Now.AddDays(-days);
        
        return source
            .Where(t => t.Date >= cutoffDate)
            .GroupBy(t => t.TransactionType)
            .Select(g => new TransactionSummary
            {
                TransactionType = g.Key,
                Count = g.Count(),
                TotalAmount = g.Sum(t => t.Amount),
                AverageAmount = g.Average(t => t.Amount)
            });
    }
}

// İşlem özeti sınıfı
public class TransactionSummary
{
    public string TransactionType { get; set; }
    public int Count { get; set; }
    public decimal TotalAmount { get; set; }
    public decimal AverageAmount { get; set; }
}

// Kullanım örneği
List<Transaction> transactions = GetTransactions(); // İşlem listesini alma

// Temel operatörler
var deposits = transactions.WhereDeposit();
var withdrawals = transactions.WhereWithdrawal();
var recentTransactions = transactions.WhereRecent(30);

// Kompozit operatörler
var recentDeposits = transactions.WhereRecentDeposit(30);
var recentWithdrawals = transactions.WhereRecentWithdrawal(30);

// Karmaşık kompozit operatör
var transactionSummary = transactions.GetRecentTransactionSummary(30);

foreach (var summary in transactionSummary)
{
    Console.WriteLine($"İşlem Tipi: {summary.TransactionType}");
    Console.WriteLine($"İşlem Sayısı: {summary.Count}");
    Console.WriteLine($"Toplam Tutar: {summary.TotalAmount:C}");
    Console.WriteLine($"Ortalama Tutar: {summary.AverageAmount:C}");
    Console.WriteLine();
}
```

### Fonksiyonel Kompozisyon

```csharp
// Fonksiyonel kompozisyon
public static class FunctionalExtensions
{
    // Fonksiyon kompozisyonu
    public static Func<T, TResult> Compose<T, TIntermediate, TResult>(
        this Func<T, TIntermediate> first,
        Func<TIntermediate, TResult> second)
    {
        return x => second(first(x));
    }
    
    // LINQ operatörü kompozisyonu
    public static Func<IEnumerable<T>, IEnumerable<TResult>> ComposeOperators<T, TIntermediate, TResult>(
        this Func<IEnumerable<T>, IEnumerable<TIntermediate>> first,
        Func<IEnumerable<TIntermediate>, IEnumerable<TResult>> second)
    {
        return source => second(first(source));
    }
}

// Kullanım örneği
// Fonksiyon kompozisyonu
Func<Transaction, decimal> getAmount = t => t.Amount;
Func<decimal, bool> isLarge = amount => amount > 1000;
Func<Transaction, bool> isLargeTransaction = getAmount.Compose(isLarge);

// LINQ operatörü kompozisyonu
Func<IEnumerable<Transaction>, IEnumerable<Transaction>> getDeposits = 
    transactions => transactions.Where(t => t.Amount > 0);
    
Func<IEnumerable<Transaction>, IEnumerable<Transaction>> getRecent = 
    transactions => transactions.Where(t => t.Date >= DateTime.Now.AddDays(-30));
    
Func<IEnumerable<Transaction>, IEnumerable<Transaction>> getRecentDeposits = 
    getDeposits.ComposeOperators(getRecent);

// Kullanım
List<Transaction> transactions = GetTransactions(); // İşlem listesini alma
var recentDeposits = getRecentDeposits(transactions);

foreach (var transaction in recentDeposits)
{
    Console.WriteLine($"{transaction.Date.ToShortDateString()} - {transaction.Amount:C} - {transaction.Description}");
}
```

## Özet

Bu bölümde, özel LINQ operatörleri oluşturma tekniklerini ve bunların gerçek dünya uygulamalarını inceledik:

1. **Extension Method Pattern**: LINQ operatörleri, uzantı metotları olarak uygulanır.

2. **Yield Return Usage**: `yield return` ifadesi, tembel değerlendirme özelliğini sağlamak için kullanılır.

3. **Query Operator Design**: Etkili LINQ operatörleri tasarlamak için belirli desenleri ve en iyi uygulamaları takip etmelisiniz.

4. **Performance Optimization**: Erken filtreleme ve önbelleğe alma gibi tekniklerle performansı optimize edebilirsiniz.

5. **Error Handling**: Özel LINQ operatörleri oluştururken, hataları düzgün bir şekilde işlemek önemlidir.

6. **Operator Composition**: Karmaşık işlevselliği elde etmek için basit operatörleri birleştirebilirsiniz.

Özel LINQ operatörleri oluşturarak, kodunuzu daha okunabilir, bakımı daha kolay ve daha verimli hale getirebilirsiniz. Bu teknikler, özellikle karmaşık veri işleme senaryolarında ve domain-specific dil (DSL) oluşturmada yararlıdır. 