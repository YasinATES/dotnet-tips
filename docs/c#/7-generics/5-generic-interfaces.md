# Generic Interfaces (Generic Arayüzler)

C#'ta generic arayüzler, farklı veri tipleriyle çalışabilen, tip güvenliği sağlayan ve yeniden kullanılabilir kod yazmanıza olanak tanıyan güçlü yapılardır. Bu bölümde, C#'taki temel generic arayüzleri ve bunların banka uygulamalarındaki kullanımlarını inceleyeceğiz.

## 1. IEnumerable\<T\>

`IEnumerable<T>`, bir koleksiyonun elemanları üzerinde yineleme yapabilmeyi sağlayan en temel generic arayüzdür. LINQ sorgularının çalışabilmesi için bir koleksiyonun en azından `IEnumerable<T>` arayüzünü uygulaması gerekir.

### Özellikleri ve Kullanımı

```csharp
// IEnumerable<T> arayüzünün temel yapısı
public interface IEnumerable<T> : IEnumerable
{
    IEnumerator<T> GetEnumerator();
}
```

###  IEnumerable\<T\> Kullanımı

```csharp
// Banka işlemleri için basit bir sınıf
public class Transaction
{
    public string TransactionId { get; set; }
    public decimal Amount { get; set; }
    public DateTime Date { get; set; }
    public TransactionType Type { get; set; }
    public string AccountNumber { get; set; }
}

public enum TransactionType
{
    Deposit,
    Withdrawal,
    Transfer,
    Payment
}

// IEnumerable<T> döndüren metotlar
public class TransactionService
{
    private List<Transaction> _transactions = new List<Transaction>();
    
    // Tüm işlemleri döndüren metot
    public IEnumerable<Transaction> GetAllTransactions()
    {
        return _transactions;
    }
    
    // Belirli bir hesaba ait işlemleri döndüren metot
    public IEnumerable<Transaction> GetTransactionsByAccount(string accountNumber)
    {
        return _transactions.Where(t => t.AccountNumber == accountNumber);
    }
    
    // Belirli bir tarih aralığındaki işlemleri döndüren metot
    public IEnumerable<Transaction> GetTransactionsByDateRange(DateTime startDate, DateTime endDate)
    {
        return _transactions.Where(t => t.Date >= startDate && t.Date <= endDate);
    }
    
    // LINQ kullanarak işlemleri filtreleme ve dönüştürme
    public IEnumerable<TransactionSummary> GetTransactionSummaries()
    {
        return _transactions
            .GroupBy(t => t.AccountNumber)
            .Select(g => new TransactionSummary
            {
                AccountNumber = g.Key,
                TotalDeposits = g.Where(t => t.Type == TransactionType.Deposit).Sum(t => t.Amount),
                TotalWithdrawals = g.Where(t => t.Type == TransactionType.Withdrawal).Sum(t => t.Amount),
                TransactionCount = g.Count()
            });
    }
}

public class TransactionSummary
{
    public string AccountNumber { get; set; }
    public decimal TotalDeposits { get; set; }
    public decimal TotalWithdrawals { get; set; }
    public int TransactionCount { get; set; }
}

// IEnumerable<T> kullanımı
public class TransactionReportGenerator
{
    private readonly TransactionService _transactionService;
    
    public TransactionReportGenerator(TransactionService transactionService)
    {
        _transactionService = transactionService;
    }
    
    public void GenerateMonthlyReport(string accountNumber, DateTime month)
    {
        // IEnumerable<T> üzerinde LINQ sorguları
        var transactions = _transactionService.GetTransactionsByAccount(accountNumber)
            .Where(t => t.Date.Month == month.Month && t.Date.Year == month.Year)
            .OrderBy(t => t.Date);
        
        decimal balance = 0;
        
        Console.WriteLine($"İşlem Raporu - Hesap: {accountNumber}, Tarih: {month:MMMM yyyy}");
        Console.WriteLine("------------------------------------------------------------");
        
        // IEnumerable<T> üzerinde foreach döngüsü
        foreach (var transaction in transactions)
        {
            if (transaction.Type == TransactionType.Deposit || transaction.Type == TransactionType.Transfer)
                balance += transaction.Amount;
            else
                balance -= transaction.Amount;
                
            Console.WriteLine($"{transaction.Date:dd/MM/yyyy} | {transaction.Type} | {transaction.Amount:C2} | Bakiye: {balance:C2}");
        }
    }
}
```

## 2. ICollection\<T\>

`ICollection<T>`, `IEnumerable<T>` arayüzünü genişleterek koleksiyona eleman ekleme, çıkarma ve koleksiyon hakkında bilgi alma gibi işlemleri sağlar.

### Özellikleri ve Kullanımı

```csharp
// ICollection<T> arayüzünün temel yapısı
public interface ICollection<T> : IEnumerable<T>
{
    int Count { get; }
    bool IsReadOnly { get; }
    void Add(T item);
    void Clear();
    bool Contains(T item);
    void CopyTo(T[] array, int arrayIndex);
    bool Remove(T item);
}
```

###  ICollection\<T\> Kullanımı

```csharp
// Banka hesabı sınıfı
public class BankAccount
{
    public string AccountNumber { get; set; }
    public string CustomerName { get; set; }
    public decimal Balance { get; set; }
    public AccountType Type { get; set; }
    
    // Hesaba bağlı kartlar koleksiyonu
    private ICollection<BankCard> _cards = new List<BankCard>();
    
    // Sadece okuma amaçlı erişim sağlayan özellik
    public IEnumerable<BankCard> Cards => _cards;
    
    // Kart ekleme metodu
    public void AddCard(BankCard card)
    {
        // Kart numarası kontrolü
        if (_cards.Any(c => c.CardNumber == card.CardNumber))
            throw new InvalidOperationException("Bu kart numarası zaten kullanılıyor.");
            
        _cards.Add(card);
    }
    
    // Kart silme metodu
    public bool RemoveCard(string cardNumber)
    {
        var card = _cards.FirstOrDefault(c => c.CardNumber == cardNumber);
        if (card != null)
            return _cards.Remove(card);
            
        return false;
    }
    
    // Kart sayısını alma
    public int GetCardCount()
    {
        return _cards.Count;
    }
    
    // Belirli bir kart tipine sahip kartları filtreleme
    public IEnumerable<BankCard> GetCardsByType(CardType type)
    {
        return _cards.Where(c => c.Type == type);
    }
}

public enum AccountType
{
    Checking,
    Savings,
    Investment,
    Loan
}

// Banka kartı sınıfı
public class BankCard
{
    public string CardNumber { get; set; }
    public string CardHolderName { get; set; }
    public DateTime ExpiryDate { get; set; }
    public CardType Type { get; set; }
    public bool IsActive { get; set; }
}

public enum CardType
{
    DebitCard,
    CreditCard,
    PrepaidCard
}

// ICollection<T> kullanımı
public class CardManagementService
{
    private readonly ICollection<BankCard> _allCards = new List<BankCard>();
    
    // Yeni kart oluşturma ve koleksiyona ekleme
    public BankCard CreateCard(string customerName, CardType type)
    {
        var card = new BankCard
        {
            CardNumber = GenerateCardNumber(),
            CardHolderName = customerName,
            ExpiryDate = DateTime.Now.AddYears(5),
            Type = type,
            IsActive = true
        };
        
        _allCards.Add(card);
        return card;
    }
    
    // Kartı devre dışı bırakma
    public bool DeactivateCard(string cardNumber)
    {
        var card = _allCards.FirstOrDefault(c => c.CardNumber == cardNumber);
        if (card != null)
        {
            card.IsActive = false;
            return true;
        }
        return false;
    }
    
    // Aktif kart sayısını alma
    public int GetActiveCardCount()
    {
        return _allCards.Count(c => c.IsActive);
    }
    
    // Kart numarası üretme (örnek amaçlı basitleştirilmiş)
    private string GenerateCardNumber()
    {
        return Guid.NewGuid().ToString("N").Substring(0, 16);
    }
}
```

## 3. IList\<T\>

`IList<T>`, `ICollection<T>` arayüzünü genişleterek indeks tabanlı erişim ve eleman ekleme/çıkarma işlemlerini sağlar.

### Özellikleri ve Kullanımı

```csharp
// IList<T> arayüzünün temel yapısı
public interface IList<T> : ICollection<T>
{
    T this[int index] { get; set; }
    int IndexOf(T item);
    void Insert(int index, T item);
    void RemoveAt(int index);
}
```

###  IList\<T\> Kullanımı

```csharp
// Müşteri portföyü sınıfı
public class CustomerPortfolio
{
    public string CustomerId { get; set; }
    public string CustomerName { get; set; }
    
    // Müşterinin yatırım araçları listesi
    private IList<Investment> _investments = new List<Investment>();
    
    // Yatırım ekleme
    public void AddInvestment(Investment investment)
    {
        _investments.Add(investment);
    }
    
    // Belirli bir indeksteki yatırımı alma
    public Investment GetInvestmentAt(int index)
    {
        if (index < 0 || index >= _investments.Count)
            throw new IndexOutOfRangeException("Geçersiz yatırım indeksi.");
            
        return _investments[index];
    }
    
    // Belirli bir indekse yatırım ekleme
    public void InsertInvestmentAt(int index, Investment investment)
    {
        _investments.Insert(index, investment);
    }
    
    // Belirli bir indeksteki yatırımı güncelleme
    public void UpdateInvestmentAt(int index, Investment investment)
    {
        if (index < 0 || index >= _investments.Count)
            throw new IndexOutOfRangeException("Geçersiz yatırım indeksi.");
            
        _investments[index] = investment;
    }
    
    // Belirli bir indeksteki yatırımı silme
    public void RemoveInvestmentAt(int index)
    {
        _investments.RemoveAt(index);
    }
    
    // Yatırımları öncelik sırasına göre yeniden düzenleme
    public void ReorderInvestmentsByPriority()
    {
        // Geçici bir liste oluşturup sıralama
        var sortedList = _investments.OrderByDescending(i => i.Priority).ToList();
        
        // Mevcut listeyi temizleme
        _investments.Clear();
        
        // Sıralanmış elemanları ekleme
        foreach (var investment in sortedList)
        {
            _investments.Add(investment);
        }
    }
    
    // Toplam portföy değerini hesaplama
    public decimal CalculateTotalValue()
    {
        return _investments.Sum(i => i.CurrentValue);
    }
}

// Yatırım sınıfı
public class Investment
{
    public string InvestmentId { get; set; }
    public string Name { get; set; }
    public InvestmentType Type { get; set; }
    public decimal InitialValue { get; set; }
    public decimal CurrentValue { get; set; }
    public DateTime PurchaseDate { get; set; }
    public int Priority { get; set; } // Yatırım önceliği
}

public enum InvestmentType
{
    Stock,
    Bond,
    MutualFund,
    FixedDeposit,
    RealEstate
}

// IList<T> kullanımı
public class PortfolioManager
{
    private readonly IList<CustomerPortfolio> _portfolios = new List<CustomerPortfolio>();
    
    // Yeni portföy oluşturma
    public CustomerPortfolio CreatePortfolio(string customerId, string customerName)
    {
        var portfolio = new CustomerPortfolio
        {
            CustomerId = customerId,
            CustomerName = customerName
        };
        
        _portfolios.Add(portfolio);
        return portfolio;
    }
    
    // Portföyleri değere göre sıralama ve ilk N tanesini alma
    public IList<CustomerPortfolio> GetTopPortfolios(int count)
    {
        return _portfolios
            .OrderByDescending(p => p.CalculateTotalValue())
            .Take(count)
            .ToList();
    }
    
    // Belirli bir müşteri ID'sine göre portföy bulma
    public CustomerPortfolio FindPortfolioByCustomerId(string customerId)
    {
        for (int i = 0; i < _portfolios.Count; i++)
        {
            if (_portfolios[i].CustomerId == customerId)
                return _portfolios[i];
        }
        
        return null;
    }
}
```

## 4. IDictionary\<TKey, TValue\>

`IDictionary<TKey, TValue>`, anahtar-değer çiftlerini depolamak için kullanılan bir generic arayüzdür. Anahtarlar benzersiz olmalıdır.

### Özellikleri ve Kullanımı

```csharp
// IDictionary<TKey, TValue> arayüzünün temel yapısı
public interface IDictionary<TKey, TValue> : ICollection<KeyValuePair<TKey, TValue>>
{
    TValue this[TKey key] { get; set; }
    ICollection<TKey> Keys { get; }
    ICollection<TValue> Values { get; }
    bool ContainsKey(TKey key);
    bool TryGetValue(TKey key, out TValue value);
    void Add(TKey key, TValue value);
    bool Remove(TKey key);
}
```

###  IDictionary\<TKey, TValue\> Kullanımı

```csharp
// Müşteri veri deposu
public class CustomerRepository
{
    // Müşteri ID'sine göre müşteri bilgilerini saklayan sözlük
    private readonly IDictionary<string, Customer> _customers = new Dictionary<string, Customer>();
    
    // Müşteri ekleme
    public void AddCustomer(Customer customer)
    {
        if (_customers.ContainsKey(customer.Id))
            throw new InvalidOperationException($"Bu ID'ye sahip müşteri zaten var: {customer.Id}");
            
        _customers.Add(customer.Id, customer);
    }
    
    // Müşteri güncelleme
    public void UpdateCustomer(Customer customer)
    {
        if (!_customers.ContainsKey(customer.Id))
            throw new KeyNotFoundException($"Bu ID'ye sahip müşteri bulunamadı: {customer.Id}");
            
        _customers[customer.Id] = customer;
    }
    
    // Müşteri silme
    public bool RemoveCustomer(string customerId)
    {
        return _customers.Remove(customerId);
    }
    
    // Müşteri bulma
    public Customer GetCustomer(string customerId)
    {
        if (_customers.TryGetValue(customerId, out Customer customer))
            return customer;
            
        return null;
    }
    
    // Tüm müşteri ID'lerini alma
    public IEnumerable<string> GetAllCustomerIds()
    {
        return _customers.Keys;
    }
    
    // Tüm müşterileri alma
    public IEnumerable<Customer> GetAllCustomers()
    {
        return _customers.Values;
    }
    
    // Müşteri sayısını alma
    public int GetCustomerCount()
    {
        return _customers.Count;
    }
}

// Müşteri sınıfı
public class Customer
{
    public string Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
    public string Phone { get; set; }
    public DateTime RegistrationDate { get; set; }
    public CustomerStatus Status { get; set; }
}

public enum CustomerStatus
{
    Active,
    Inactive,
    Blocked,
    Closed
}

// Hesap yönetim servisi
public class AccountManagementService
{
    // Hesap numarasına göre hesap bilgilerini saklayan sözlük
    private readonly IDictionary<string, BankAccount> _accounts = new Dictionary<string, BankAccount>();
    
    // Müşteri ID'sine göre müşterinin hesaplarını saklayan sözlük
    private readonly IDictionary<string, List<string>> _customerAccounts = new Dictionary<string, List<string>>();
    
    // Yeni hesap oluşturma
    public BankAccount CreateAccount(string customerId, AccountType type)
    {
        var account = new BankAccount
        {
            AccountNumber = GenerateAccountNumber(),
            Type = type,
            Balance = 0,
            CustomerName = GetCustomerName(customerId)
        };
        
        // Hesabı hesaplar sözlüğüne ekleme
        _accounts.Add(account.AccountNumber, account);
        
        // Müşteri-hesap ilişkisini güncelleme
        if (!_customerAccounts.ContainsKey(customerId))
            _customerAccounts[customerId] = new List<string>();
            
        _customerAccounts[customerId].Add(account.AccountNumber);
        
        return account;
    }
    
    // Müşterinin hesaplarını getirme
    public IEnumerable<BankAccount> GetCustomerAccounts(string customerId)
    {
        if (_customerAccounts.TryGetValue(customerId, out List<string> accountNumbers))
        {
            foreach (var accountNumber in accountNumbers)
            {
                if (_accounts.TryGetValue(accountNumber, out BankAccount account))
                    yield return account;
            }
        }
    }
    
    // Hesap numarası üretme (örnek amaçlı basitleştirilmiş)
    private string GenerateAccountNumber()
    {
        return Guid.NewGuid().ToString("N").Substring(0, 10);
    }
    
    // Müşteri adını getirme (örnek amaçlı basitleştirilmiş)
    private string GetCustomerName(string customerId)
    {
        return $"Customer {customerId}";
    }
}
```

## 5. IComparable\<T\>

`IComparable<T>`, nesnelerin sıralanabilmesini sağlayan bir generic arayüzdür. Bu arayüzü uygulayan sınıflar, `CompareTo` metodunu tanımlayarak nesnelerin nasıl karşılaştırılacağını belirtir.

### Özellikleri ve Kullanımı

```csharp
// IComparable<T> arayüzünün temel yapısı
public interface IComparable<T>
{
    int CompareTo(T other);
}
```

###  IComparable\<T\> Kullanımı

```csharp
// Kredi başvurusu sınıfı
public class LoanApplication : IComparable<LoanApplication>
{
    public string ApplicationId { get; set; }
    public string CustomerId { get; set; }
    public decimal RequestedAmount { get; set; }
    public int CreditScore { get; set; }
    public DateTime ApplicationDate { get; set; }
    public LoanType Type { get; set; }
    public LoanStatus Status { get; set; }
    
    // Kredi başvurularını kredi skoruna göre karşılaştırma
    public int CompareTo(LoanApplication other)
    {
        if (other == null)
            return 1;
            
        // Önce kredi skoruna göre azalan sıralama
        int creditScoreComparison = other.CreditScore.CompareTo(this.CreditScore);
        if (creditScoreComparison != 0)
            return creditScoreComparison;
            
        // Kredi skorları eşitse, başvuru tarihine göre artan sıralama
        return this.ApplicationDate.CompareTo(other.ApplicationDate);
    }
}

public enum LoanType
{
    Personal,
    Mortgage,
    Vehicle,
    Education,
    Business
}

public enum LoanStatus
{
    Pending,
    UnderReview,
    Approved,
    Rejected,
    Disbursed,
    Closed
}

// Kredi değerlendirme servisi
public class LoanEvaluationService
{
    private List<LoanApplication> _applications = new List<LoanApplication>();
    
    // Yeni kredi başvurusu ekleme
    public void AddApplication(LoanApplication application)
    {
        _applications.Add(application);
    }
    
    // Kredi başvurularını değerlendirme sırasına göre sıralama
    public IEnumerable<LoanApplication> GetPrioritizedApplications()
    {
        // IComparable<T> arayüzü sayesinde sıralama yapılabilir
        _applications.Sort();
        return _applications;
    }
    
    // Belirli bir kredi skorunun üzerindeki başvuruları otomatik onaylama
    public void AutoApproveHighScoreApplications(int minimumCreditScore)
    {
        foreach (var application in _applications)
        {
            if (application.Status == LoanStatus.Pending && application.CreditScore >= minimumCreditScore)
            {
                application.Status = LoanStatus.Approved;
                Console.WriteLine($"Başvuru otomatik onaylandı: {application.ApplicationId}");
            }
        }
    }
}
```

## 6. IEquatable\<T\>

`IEquatable<T>`, nesnelerin eşitlik karşılaştırması yapabilmesini sağlayan bir generic arayüzdür. Bu arayüzü uygulayan sınıflar, `Equals` metodunu tanımlayarak nesnelerin nasıl karşılaştırılacağını belirtir.

### Özellikleri ve Kullanımı

```csharp
// IEquatable<T> arayüzünün temel yapısı
public interface IEquatable<T>
{
    bool Equals(T other);
}
```

###  IEquatable\<T\> Kullanımı

```csharp
// Banka işlemi sınıfı
public class BankTransaction : IEquatable<BankTransaction>
{
    public string TransactionId { get; set; }
    public string SourceAccountNumber { get; set; }
    public string DestinationAccountNumber { get; set; }
    public decimal Amount { get; set; }
    public DateTime Timestamp { get; set; }
    public string ReferenceNumber { get; set; }
    
    // İşlemleri karşılaştırma (aynı referans numarasına sahip işlemler aynı kabul edilir)
    public bool Equals(BankTransaction other)
    {
        if (other == null)
            return false;
            
        return this.ReferenceNumber == other.ReferenceNumber;
    }
    
    // Object.Equals metodunu override etme
    public override bool Equals(object obj)
    {
        if (obj == null || GetType() != obj.GetType())
            return false;
            
        return Equals((BankTransaction)obj);
    }
    
    // GetHashCode metodunu override etme
    public override int GetHashCode()
    {
        return ReferenceNumber?.GetHashCode() ?? 0;
    }
}

// İşlem doğrulama servisi
public class TransactionVerificationService
{
    private readonly HashSet<BankTransaction> _processedTransactions = new HashSet<BankTransaction>();
    
    // İşlemi işleme ve mükerrer işlemleri kontrol etme
    public bool ProcessTransaction(BankTransaction transaction)
    {
        // IEquatable<T> arayüzü sayesinde mükerrer işlem kontrolü yapılabilir
        if (_processedTransactions.Contains(transaction))
        {
            Console.WriteLine($"Mükerrer işlem tespit edildi: {transaction.ReferenceNumber}");
            return false;
        }
        
        // İşlemi işleme ve kaydetme
        _processedTransactions.Add(transaction);
        Console.WriteLine($"İşlem başarıyla işlendi: {transaction.TransactionId}");
        return true;
    }
    
    // İşlenmiş işlem sayısını alma
    public int GetProcessedTransactionCount()
    {
        return _processedTransactions.Count;
    }
    
    // Belirli bir referans numarasına sahip işlemin daha önce işlenip işlenmediğini kontrol etme
    public bool IsTransactionProcessed(string referenceNumber)
    {
        var dummyTransaction = new BankTransaction { ReferenceNumber = referenceNumber };
        return _processedTransactions.Contains(dummyTransaction);
    }
}
```

## Özet

Bu bölümde, C#'taki temel generic arayüzleri ve bunların banka uygulamalarındaki kullanımlarını inceledik:

- **IEnumerable\<T\>**: Koleksiyonlar üzerinde yineleme yapmayı sağlar ve LINQ sorgularının temelini oluşturur.
- **ICollection\<T\>**: Koleksiyona eleman ekleme, çıkarma ve koleksiyon hakkında bilgi alma işlemlerini sağlar.
- **IList\<T\>**: İndeks tabanlı erişim ve eleman ekleme/çıkarma işlemlerini sağlar.
- **IDictionary\<TKey, TValue\>**: Anahtar-değer çiftlerini depolamak için kullanılır.
- **IComparable\<T\>**: Nesnelerin sıralanabilmesini sağlar.
- **IEquatable\<T\>**: Nesnelerin eşitlik karşılaştırması yapabilmesini sağlar.

Generic arayüzler, tip güvenliği sağlayarak, kod tekrarını azaltarak ve performansı artırarak daha güvenilir ve bakımı kolay uygulamalar geliştirmenize yardımcı olur. Özellikle banka gibi karmaşık iş mantığına sahip uygulamalarda, generic arayüzlerin doğru kullanımı, kodun kalitesini ve esnekliğini önemli ölçüde artırabilir. 