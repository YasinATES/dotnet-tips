# Generic Collections (Generic Koleksiyonlar)

C#'ta generic koleksiyonlar, farklı veri tipleriyle çalışabilen, tip güvenliği sağlayan ve performans avantajları sunan veri yapılarıdır. Bu bölümde, generic koleksiyonların özelliklerini, avantajlarını ve gerçek dünya banka uygulamalarındaki kullanım örneklerini inceleyeceğiz.

## 1. Generic vs Non-Generic

C#'ta iki tür koleksiyon bulunur: Generic ve non-generic koleksiyonlar. Generic koleksiyonlar, belirli bir veri tipi için optimize edilmiş ve tip güvenliği sağlayan koleksiyonlardır. Non-generic koleksiyonlar ise `object` tipinde elemanlar içerir ve tip dönüşümü gerektirir.

### Non-Generic Koleksiyonlar (System.Collections)

```csharp
// Non-generic ArrayList kullanımı
ArrayList accounts = new ArrayList();
accounts.Add(new BankAccount { AccountNumber = "123456", Balance = 1000 });
accounts.Add(new BankAccount { AccountNumber = "789012", Balance = 2500 });
accounts.Add("Geçersiz veri"); // Tip güvenliği yok, herhangi bir tipte veri eklenebilir

// Elemanlara erişim için tip dönüşümü gerekir
BankAccount account = (BankAccount)accounts[0]; // Açık tip dönüşümü
```

### Generic Koleksiyonlar (System.Collections.Generic)

```csharp
// Generic List<T> kullanımı
List<BankAccount> accounts = new List<BankAccount>();
accounts.Add(new BankAccount { AccountNumber = "123456", Balance = 1000 });
accounts.Add(new BankAccount { AccountNumber = "789012", Balance = 2500 });
// accounts.Add("Geçersiz veri"); // Derleme hatası - tip güvenliği sağlanır

// Elemanlara erişim için tip dönüşümü gerekmez
BankAccount account = accounts[0]; // Doğrudan erişim
```

###  Karşılaştırma

```csharp
// Banka hesabı sınıfı
public class BankAccount
{
    public string AccountNumber { get; set; }
    public decimal Balance { get; set; }
    public string CustomerId { get; set; }
    public DateTime OpeningDate { get; set; }
    public AccountType Type { get; set; }
}

public enum AccountType
{
    Checking,
    Savings,
    Credit,
    Investment
}

// Non-generic koleksiyon kullanımı
public class OldBankAccountManager
{
    private ArrayList _accounts = new ArrayList();
    
    public void AddAccount(BankAccount account)
    {
        _accounts.Add(account);
    }
    
    public BankAccount FindAccount(string accountNumber)
    {
        foreach (object obj in _accounts)
        {
            BankAccount account = obj as BankAccount;
            if (account != null && account.AccountNumber == accountNumber)
                return account;
        }
        return null;
    }
    
    public decimal CalculateTotalBalance()
    {
        decimal total = 0;
        foreach (object obj in _accounts)
        {
            BankAccount account = obj as BankAccount;
            if (account != null)
                total += account.Balance;
        }
        return total;
    }
}

// Generic koleksiyon kullanımı
public class BankAccountManager
{
    private List<BankAccount> _accounts = new List<BankAccount>();
    
    public void AddAccount(BankAccount account)
    {
        _accounts.Add(account);
    }
    
    public BankAccount FindAccount(string accountNumber)
    {
        return _accounts.FirstOrDefault(a => a.AccountNumber == accountNumber);
    }
    
    public decimal CalculateTotalBalance()
    {
        return _accounts.Sum(a => a.Balance);
    }
    
    public List<BankAccount> FilterAccounts(AccountType type)
    {
        return _accounts.Where(a => a.Type == type).ToList();
    }
}
```

## 2. Collection Performance

Generic koleksiyonlar, non-generic koleksiyonlara göre daha iyi performans sunar. Bunun nedenleri:

1. **Boxing/Unboxing İşlemlerinin Olmaması**: Non-generic koleksiyonlar `object` tipinde veri sakladığı için değer tiplerini kullanırken boxing/unboxing işlemleri gerekir, bu da performansı düşürür.
2. **Tip Dönüşümlerinin Olmaması**: Generic koleksiyonlarda tip dönüşümü gerekmez, bu da performansı artırır.
3. **Daha İyi Bellek Kullanımı**: Generic koleksiyonlar, belirli bir tip için optimize edilmiştir ve daha verimli bellek kullanımı sağlar.

### Performans Karşılaştırması Örneği

```csharp
// Performans testi
public class CollectionPerformanceTest
{
    public void ComparePerformance(int elementCount)
    {
        // Non-generic ArrayList
        Stopwatch sw1 = Stopwatch.StartNew();
        ArrayList nonGenericList = new ArrayList();
        for (int i = 0; i < elementCount; i++)
        {
            nonGenericList.Add(i); // Boxing işlemi
        }
        int sum1 = 0;
        foreach (object obj in nonGenericList)
        {
            sum1 += (int)obj; // Unboxing işlemi
        }
        sw1.Stop();
        
        // Generic List<T>
        Stopwatch sw2 = Stopwatch.StartNew();
        List<int> genericList = new List<int>();
        for (int i = 0; i < elementCount; i++)
        {
            genericList.Add(i); // Boxing yok
        }
        int sum2 = 0;
        foreach (int num in genericList)
        {
            sum2 += num; // Unboxing yok
        }
        sw2.Stop();
        
        Console.WriteLine($"Non-generic ArrayList: {sw1.ElapsedMilliseconds} ms");
        Console.WriteLine($"Generic List<int>: {sw2.ElapsedMilliseconds} ms");
    }
}
```

###  Performans Optimizasyonu

```csharp
// Yüksek performanslı işlem kaydı tutan sınıf
public class TransactionRecordManager
{
    // Milyonlarca işlem kaydı için optimize edilmiş generic koleksiyon
    private List<TransactionRecord> _transactions = new List<TransactionRecord>(1000000); // Başlangıç kapasitesi belirtilerek performans artırılır
    
    public void AddTransaction(TransactionRecord transaction)
    {
        _transactions.Add(transaction);
    }
    
    public List<TransactionRecord> GetDailyTransactions(DateTime date)
    {
        return _transactions.Where(t => t.TransactionDate.Date == date.Date).ToList();
    }
    
    // Yüksek performanslı toplu işlem
    public void AddBulkTransactions(IEnumerable<TransactionRecord> transactions)
    {
        // AddRange metodu tek tek Add çağrılarından daha verimlidir
        _transactions.AddRange(transactions);
    }
}

public class TransactionRecord
{
    public string TransactionId { get; set; }
    public string AccountNumber { get; set; }
    public decimal Amount { get; set; }
    public DateTime TransactionDate { get; set; }
    public TransactionType Type { get; set; }
}

public enum TransactionType
{
    Deposit,
    Withdrawal,
    Transfer,
    EFT,
    AutomaticPayment
}
```

## 3. Custom Generic Collections

Özel ihtiyaçlar için kendi generic koleksiyonlarınızı oluşturabilirsiniz. Bu, belirli bir iş mantığına özel davranışlar eklemek veya performansı optimize etmek için kullanışlıdır.

### Banka Uygulaması için Özel Generic Koleksiyon

```csharp
// Özel generic koleksiyon
public class AccountCollection<T> : IEnumerable<T> where T : BankAccount
{
    private List<T> _accounts = new List<T>();
    private Dictionary<string, T> _accountNumberIndex = new Dictionary<string, T>();
    private Dictionary<string, List<T>> _customerIdIndex = new Dictionary<string, List<T>>();
    
    // Hesap ekleme - birden fazla indeksi günceller
    public void Add(T account)
    {
        _accounts.Add(account);
        
        // Hesap numarasına göre indeksleme
        _accountNumberIndex[account.AccountNumber] = account;
        
        // Müşteri ID'sine göre indeksleme
        if (!_customerIdIndex.ContainsKey(account.CustomerId))
            _customerIdIndex[account.CustomerId] = new List<T>();
        
        _customerIdIndex[account.CustomerId].Add(account);
    }
    
    // Hesap numarasına göre hızlı arama (O(1) karmaşıklık)
    public T FindAccount(string accountNumber)
    {
        if (_accountNumberIndex.TryGetValue(accountNumber, out T account))
            return account;
        
        return null;
    }
    
    // Müşteri ID'sine göre hesapları getirme
    public List<T> GetCustomerAccounts(string customerId)
    {
        if (_customerIdIndex.TryGetValue(customerId, out List<T> accounts))
            return accounts;
        
        return new List<T>();
    }
    
    // Belirli bir bakiyenin üzerindeki hesapları filtreleme
    public List<T> FilterByBalance(decimal minBalance)
    {
        return _accounts.Where(a => a.Balance >= minBalance).ToList();
    }
    
    // IEnumerable<T> arayüzünü uygulama
    public IEnumerator<T> GetEnumerator()
    {
        return _accounts.GetEnumerator();
    }
    
    // IEnumerable arayüzünü uygulama
    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}

// Kullanım örneği
public class BankApplication
{
    public void AccountManagement()
    {
        AccountCollection<BankAccount> accounts = new AccountCollection<BankAccount>();
        
        // Hesapları ekleme
        accounts.Add(new BankAccount { AccountNumber = "123456", CustomerId = "C001", Balance = 1000 });
        accounts.Add(new BankAccount { AccountNumber = "789012", CustomerId = "C001", Balance = 2500 });
        accounts.Add(new BankAccount { AccountNumber = "345678", CustomerId = "C002", Balance = 5000 });
        
        // Hesap numarasına göre hızlı arama
        BankAccount account = accounts.FindAccount("123456");
        
        // Müşterinin tüm hesaplarını getirme
        List<BankAccount> customerAccounts = accounts.GetCustomerAccounts("C001");
        
        // Belirli bir bakiyenin üzerindeki hesapları filtreleme
        List<BankAccount> highBalanceAccounts = accounts.FilterByBalance(2000);
    }
}
```

## 4. Generic Collection Constraints

Generic koleksiyonlarda da kısıtlamalar kullanarak, koleksiyonun içereceği elemanların belirli özelliklere sahip olmasını sağlayabilirsiniz.

### Banka Uygulaması için Kısıtlamalı Koleksiyonlar

```csharp
// Temel hesap sınıfı
public abstract class BankAccountBase
{
    public string AccountNumber { get; set; }
    public decimal Balance { get; set; }
    public string CustomerId { get; set; }
    
    public abstract void CalculateInterest();
}

// Vadeli hesap
public class SavingsAccount : BankAccountBase
{
    public decimal InterestRate { get; set; }
    public DateTime MaturityDate { get; set; }
    
    public override void CalculateInterest()
    {
        // Vadeli hesap için faiz hesaplama
        Console.WriteLine($"Vadeli hesap faizi: {Balance * InterestRate / 100}");
    }
}

// Vadesiz hesap
public class CheckingAccount : BankAccountBase
{
    public bool IsAutomaticPaymentActive { get; set; }
    
    public override void CalculateInterest()
    {
        // Vadesiz hesap için faiz hesaplama (genellikle düşük veya sıfır)
        Console.WriteLine("Vadesiz hesap için faiz uygulanmaz");
    }
}

// Kısıtlamalı generic koleksiyon - sadece BankAccountBase'den türeyen sınıfları kabul eder
public class AccountManager<T> where T : BankAccountBase
{
    private List<T> _accounts = new List<T>();
    
    public void AddAccount(T account)
    {
        _accounts.Add(account);
    }
    
    public void CalculateInterestForAllAccounts()
    {
        foreach (T account in _accounts)
        {
            account.CalculateInterest(); // BankAccountBase'den gelen metot
        }
    }
    
    public decimal CalculateTotalBalance()
    {
        return _accounts.Sum(a => a.Balance);
    }
}

// Kullanım örneği
public class InterestCalculationService
{
    public void InterestOperations()
    {
        // Vadeli hesaplar için yönetici
        AccountManager<SavingsAccount> savingsAccountManager = new AccountManager<SavingsAccount>();
        savingsAccountManager.AddAccount(new SavingsAccount { AccountNumber = "S001", Balance = 10000, InterestRate = 15 });
        savingsAccountManager.AddAccount(new SavingsAccount { AccountNumber = "S002", Balance = 25000, InterestRate = 12 });
        
        // Tüm vadeli hesaplar için faiz hesaplama
        savingsAccountManager.CalculateInterestForAllAccounts();
        
        // Vadesiz hesaplar için yönetici
        AccountManager<CheckingAccount> checkingAccountManager = new AccountManager<CheckingAccount>();
        checkingAccountManager.AddAccount(new CheckingAccount { AccountNumber = "C001", Balance = 5000 });
        
        // Tüm vadesiz hesaplar için faiz hesaplama
        checkingAccountManager.CalculateInterestForAllAccounts();
    }
}
```

## 5. Collection Initialization

C#'ta koleksiyon başlatıcıları, koleksiyonları daha kolay ve okunabilir bir şekilde oluşturmanıza olanak tanır.

###  Koleksiyon Başlatma

```csharp
public class CustomerService
{
    public void LoadCustomers()
    {
        // Koleksiyon başlatıcı kullanarak müşteri listesi oluşturma
        List<Customer> customers = new List<Customer>
        {
            new Customer { Id = "C001", FirstName = "Ahmet", LastName = "Yılmaz", Email = "ahmet@ornek.com" },
            new Customer { Id = "C002", FirstName = "Ayşe", LastName = "Kaya", Email = "ayse@ornek.com" },
            new Customer { Id = "C003", FirstName = "Mehmet", LastName = "Demir", Email = "mehmet@ornek.com" }
        };
        
        // Dictionary başlatıcı kullanarak müşteri indeksi oluşturma
        Dictionary<string, Customer> customerIndex = new Dictionary<string, Customer>
        {
            { "C001", new Customer { Id = "C001", FirstName = "Ahmet", LastName = "Yılmaz" } },
            { "C002", new Customer { Id = "C002", FirstName = "Ayşe", LastName = "Kaya" } }
        };
        
        // Alternatif Dictionary başlatıcı sözdizimi
        Dictionary<string, List<BankAccount>> customerAccounts = new Dictionary<string, List<BankAccount>>
        {
            ["C001"] = new List<BankAccount>
            {
                new BankAccount { AccountNumber = "123456", Balance = 1000 },
                new BankAccount { AccountNumber = "789012", Balance = 2500 }
            },
            ["C002"] = new List<BankAccount>
            {
                new BankAccount { AccountNumber = "345678", Balance = 5000 }
            }
        };
    }
}

public class Customer
{
    public string Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public string Email { get; set; }
}
```

## 6. Collection Serialization

Generic koleksiyonları serileştirmek, verileri depolamak veya ağ üzerinden iletmek için önemlidir. C#'ta çeşitli serileştirme yöntemleri bulunur.

###  Koleksiyon Serileştirme

```csharp
// Serileştirilebilir sınıflar
[Serializable]
public class TransactionReport
{
    public string ReportId { get; set; }
    public DateTime ReportDate { get; set; }
    public List<TransactionSummary> Transactions { get; set; }
}

[Serializable]
public class TransactionSummary
{
    public string TransactionId { get; set; }
    public string AccountNumber { get; set; }
    public decimal Amount { get; set; }
    public DateTime TransactionDate { get; set; }
}

// Serileştirme işlemleri
public class ReportService
{
    // JSON serileştirme
    public string SerializeReportAsJson(TransactionReport report)
    {
        return System.Text.Json.JsonSerializer.Serialize(report);
    }
    
    public TransactionReport DeserializeReportFromJson(string json)
    {
        return System.Text.Json.JsonSerializer.Deserialize<TransactionReport>(json);
    }
    
    // XML serileştirme
    public string SerializeReportAsXml(TransactionReport report)
    {
        XmlSerializer serializer = new XmlSerializer(typeof(TransactionReport));
        using (StringWriter writer = new StringWriter())
        {
            serializer.Serialize(writer, report);
            return writer.ToString();
        }
    }
    
    public TransactionReport DeserializeReportFromXml(string xml)
    {
        XmlSerializer serializer = new XmlSerializer(typeof(TransactionReport));
        using (StringReader reader = new StringReader(xml))
        {
            return (TransactionReport)serializer.Deserialize(reader);
        }
    }
    
    // Örnek kullanım
    public void ReportSerializationExample()
    {
        // Örnek rapor oluşturma
        TransactionReport report = new TransactionReport
        {
            ReportId = "R12345",
            ReportDate = DateTime.Now,
            Transactions = new List<TransactionSummary>
            {
                new TransactionSummary { TransactionId = "T001", AccountNumber = "123456", Amount = 1000, TransactionDate = DateTime.Now.AddHours(-2) },
                new TransactionSummary { TransactionId = "T002", AccountNumber = "789012", Amount = -500, TransactionDate = DateTime.Now.AddHours(-1) }
            }
        };
        
        // JSON serileştirme
        string json = SerializeReportAsJson(report);
        Console.WriteLine($"JSON: {json}");
        
        // JSON deserileştirme
        TransactionReport deserializedReport = DeserializeReportFromJson(json);
        Console.WriteLine($"Deserileştirilen rapor ID: {deserializedReport.ReportId}");
        
        // XML serileştirme
        string xml = SerializeReportAsXml(report);
        Console.WriteLine($"XML: {xml}");
    }
}
```

## Özet

Generic koleksiyonlar, C#'ta tip güvenliği, performans ve esneklik sağlayan güçlü veri yapılarıdır. Bu bölümde, generic koleksiyonların özelliklerini ve banka uygulamalarındaki kullanım örneklerini inceledik:

- **Generic vs Non-Generic**: Generic koleksiyonlar tip güvenliği sağlar ve performans avantajları sunar.
- **Collection Performance**: Generic koleksiyonlar, boxing/unboxing işlemlerini önleyerek ve tip dönüşümlerini ortadan kaldırarak daha iyi performans sağlar.
- **Custom Generic Collections**: Özel ihtiyaçlar için kendi generic koleksiyonlarınızı oluşturabilirsiniz.
- **Generic Collection Constraints**: Koleksiyonun içereceği elemanların belirli özelliklere sahip olmasını sağlayabilirsiniz.
- **Collection Initialization**: Koleksiyon başlatıcıları, koleksiyonları daha kolay ve okunabilir bir şekilde oluşturmanıza olanak tanır.
- **Collection Serialization**: Generic koleksiyonları serileştirerek verileri depolayabilir veya ağ üzerinden iletebilirsiniz.

Generic koleksiyonlar, modern C# uygulamalarının temel yapı taşlarından biridir ve özellikle banka gibi performans ve güvenliğin kritik olduğu uygulamalarda büyük avantajlar sağlar. 