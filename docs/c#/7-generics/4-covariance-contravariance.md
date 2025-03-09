# Covariance ve Contravariance

C#'ta covariance ve contravariance, generic tiplerin ve delegelerin tip uyumluluğunu genişleten güçlü kavramlardır. Bu bölümde, covariance ve contravariance kavramlarını, bunların C#'taki uygulamalarını ve banka projelerindeki gerçek dünya örneklerini inceleyeceğiz.

## 1. in ve out Anahtar Kelimeleri

C# 4.0 ile birlikte, generic arayüzlerde ve delegelerde covariance ve contravariance'ı belirtmek için `in` ve `out` anahtar kelimeleri tanıtılmıştır.

- **out (Covariance)**: Bir tip parametresinin yalnızca çıkış (return) pozisyonunda kullanılabileceğini belirtir.
- **in (Contravariance)**: Bir tip parametresinin yalnızca giriş (parametre) pozisyonunda kullanılabileceğini belirtir.

### Covariance (out) Örneği

```csharp
// Covariance örneği - out anahtar kelimesi
public interface IAccountInfoProvider<out T>
{
    T GetAccountInfo();
    // T UseAsParameter(T param); // Hata! out parametresi giriş pozisyonunda kullanılamaz
}

// Banka hesabı hiyerarşisi
public class BankAccount
{
    public string AccountNumber { get; set; }
    public decimal Balance { get; set; }
}

public class CheckingAccount : BankAccount
{
    public bool IsAutomaticPaymentActive { get; set; }
}

// Covariance kullanımı
public class AccountService
{
    public void ShowAccountInfo()
    {
        // CheckingAccount, BankAccount'tan türediği için covariance sayesinde aşağıdaki atama geçerlidir
        IAccountInfoProvider<CheckingAccount> checkingAccountProvider = new CheckingAccountInfoProvider();
        IAccountInfoProvider<BankAccount> accountProvider = checkingAccountProvider; // Covariance sayesinde geçerli
        
        BankAccount account = accountProvider.GetAccountInfo();
        Console.WriteLine($"Hesap No: {account.AccountNumber}, Bakiye: {account.Balance}");
    }
}

public class CheckingAccountInfoProvider : IAccountInfoProvider<CheckingAccount>
{
    public CheckingAccount GetAccountInfo()
    {
        return new CheckingAccount { AccountNumber = "123456", Balance = 1000, IsAutomaticPaymentActive = true };
    }
}
```

### Contravariance (in) Örneği

```csharp
// Contravariance örneği - in anahtar kelimesi
public interface IAccountProcessor<in T>
{
    void ProcessAccount(T account);
    // T ReturnAccount(); // Hata! in parametresi çıkış pozisyonunda kullanılamaz
}

// Contravariance kullanımı
public class AccountOperations
{
    public void PerformOperation()
    {
        // BankAccount, CheckingAccount'ın üst sınıfı olduğu için contravariance sayesinde aşağıdaki atama geçerlidir
        IAccountProcessor<BankAccount> generalAccountProcessor = new GeneralAccountProcessor();
        IAccountProcessor<CheckingAccount> checkingAccountProcessor = generalAccountProcessor; // Contravariance sayesinde geçerli
        
        checkingAccountProcessor.ProcessAccount(new CheckingAccount { AccountNumber = "123456", Balance = 1000 });
    }
}

public class GeneralAccountProcessor : IAccountProcessor<BankAccount>
{
    public void ProcessAccount(BankAccount account)
    {
        Console.WriteLine($"Hesap işleniyor: {account.AccountNumber}");
        // Genel hesap işlemleri
    }
}
```

## 2. Delege Varyansı (Delegate Variance)

C#'ta delegeler de covariance ve contravariance özelliklerini destekler. Bu, delegelerin daha esnek bir şekilde kullanılmasını sağlar.

### Delege Covariance

```csharp
// Banka işlemleri için delegeler
public delegate TResult AccountOperationDelegate<out TResult>();
public delegate void AccountUpdateDelegate<in TAccount>(TAccount account);

// Banka işlemleri servisi
public class BankOperationsService
{
    public void DelegateVarianceExample()
    {
        // Delegate covariance örneği
        AccountOperationDelegate<CheckingAccount> getCheckingAccount = CreateCheckingAccount;
        AccountOperationDelegate<BankAccount> getAnyAccount = getCheckingAccount; // Covariance sayesinde geçerli
        
        BankAccount account = getAnyAccount();
        Console.WriteLine($"Hesap oluşturuldu: {account.AccountNumber}");
        
        // Delegate contravariance örneği
        AccountUpdateDelegate<BankAccount> updateGeneralAccount = UpdateGeneralAccount;
        AccountUpdateDelegate<CheckingAccount> updateCheckingAccount = updateGeneralAccount; // Contravariance sayesinde geçerli
        
        updateCheckingAccount(new CheckingAccount { AccountNumber = "789012", Balance = 2500 });
    }
    
    private CheckingAccount CreateCheckingAccount()
    {
        return new CheckingAccount { AccountNumber = "V" + DateTime.Now.Ticks.ToString().Substring(0, 6), Balance = 0 };
    }
    
    private void UpdateGeneralAccount(BankAccount account)
    {
        Console.WriteLine($"Hesap güncelleniyor: {account.AccountNumber}");
        // Hesap güncelleme işlemleri
    }
}
```

### Func ve Action Delegelerinde Varyans

```csharp
// Func ve Action delegelerinde varyans
public class CustomerOperationsService
{
    public void FuncActionVarianceExample()
    {
        // Func delegesi covariance örneği (dönüş tipi için)
        Func<CheckingAccount> checkingAccountCreator = () => new CheckingAccount { AccountNumber = "V001", Balance = 0 };
        Func<BankAccount> accountCreator = checkingAccountCreator; // Covariance sayesinde geçerli
        
        BankAccount newAccount = accountCreator();
        
        // Action delegesi contravariance örneği (parametre tipi için)
        Action<BankAccount> processAccount = account => Console.WriteLine($"Hesap işleniyor: {account.AccountNumber}");
        Action<CheckingAccount> processCheckingAccount = processAccount; // Contravariance sayesinde geçerli
        
        processCheckingAccount(new CheckingAccount { AccountNumber = "V002", Balance = 100 });
    }
}
```

## 3. Arayüz Varyansı (Interface Variance)

Generic arayüzlerde covariance ve contravariance, tip parametrelerinin nasıl kullanıldığına bağlıdır.

###  Arayüz Varyansı

```csharp
// Covariant arayüz - müşteri bilgisi sağlayıcı
public interface ICustomerInfoProvider<out T> where T : Customer
{
    T GetCustomer(string customerId);
    IEnumerable<T> GetAllCustomers();
}

// Contravariant arayüz - müşteri işleyici
public interface ICustomerProcessor<in T> where T : Customer
{
    void AddCustomer(T customer);
    void UpdateCustomer(T customer);
    void DeleteCustomer(T customer);
}

// Müşteri sınıf hiyerarşisi
public class Customer
{
    public string Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
}

public class IndividualCustomer : Customer
{
    public string NationalId { get; set; }
}

public class CorporateCustomer : Customer
{
    public string TaxNumber { get; set; }
    public string CompanyName { get; set; }
}

// Covariant arayüz uygulaması
public class IndividualCustomerInfoProvider : ICustomerInfoProvider<IndividualCustomer>
{
    public IndividualCustomer GetCustomer(string customerId)
    {
        // Veritabanından bireysel müşteri getirme işlemi
        return new IndividualCustomer { Id = customerId, FirstName = "Ahmet", LastName = "Yılmaz", NationalId = "12345678901" };
    }
    
    public IEnumerable<IndividualCustomer> GetAllCustomers()
    {
        // Tüm bireysel müşterileri getirme işlemi
        return new List<IndividualCustomer>
        {
            new IndividualCustomer { Id = "B001", FirstName = "Ahmet", LastName = "Yılmaz", NationalId = "12345678901" },
            new IndividualCustomer { Id = "B002", FirstName = "Ayşe", LastName = "Kaya", NationalId = "98765432109" }
        };
    }
}

// Contravariant arayüz uygulaması
public class GeneralCustomerProcessor : ICustomerProcessor<Customer>
{
    public void AddCustomer(Customer customer)
    {
        Console.WriteLine($"Müşteri ekleniyor: {customer.FirstName} {customer.LastName}");
        // Müşteri ekleme işlemi
    }
    
    public void UpdateCustomer(Customer customer)
    {
        Console.WriteLine($"Müşteri güncelleniyor: {customer.FirstName} {customer.LastName}");
        // Müşteri güncelleme işlemi
    }
    
    public void DeleteCustomer(Customer customer)
    {
        Console.WriteLine($"Müşteri siliniyor: {customer.FirstName} {customer.LastName}");
        // Müşteri silme işlemi
    }
}

// Interface variance kullanımı
public class CustomerService
{
    public void CustomerOperations()
    {
        // Covariance örneği
        ICustomerInfoProvider<IndividualCustomer> individualProvider = new IndividualCustomerInfoProvider();
        ICustomerInfoProvider<Customer> generalProvider = individualProvider; // Covariance sayesinde geçerli
        
        Customer customer = generalProvider.GetCustomer("B001");
        Console.WriteLine($"Müşteri: {customer.FirstName} {customer.LastName}");
        
        // Contravariance örneği
        ICustomerProcessor<Customer> generalProcessor = new GeneralCustomerProcessor();
        ICustomerProcessor<IndividualCustomer> individualProcessor = generalProcessor; // Contravariance sayesinde geçerli
        
        individualProcessor.AddCustomer(new IndividualCustomer { Id = "B003", FirstName = "Mehmet", LastName = "Demir", NationalId = "45678901234" });
    }
}
```

## 4. Dizi Covariance (Array Covariance)

C#'ta diziler doğal olarak covariant'tır, ancak bu tip güvenliği açısından bazı sorunlara yol açabilir.

```csharp
// Dizi covariance örneği
public class ArrayVarianceExample
{
    public void ArrayCovariance()
    {
        // Dizi covariance - doğal olarak desteklenir
        IndividualCustomer[] individualCustomers = new IndividualCustomer[3];
        individualCustomers[0] = new IndividualCustomer { Id = "B001", FirstName = "Ahmet", LastName = "Yılmaz" };
        
        Customer[] customers = individualCustomers; // Covariance sayesinde geçerli
        
        // Ancak bu, çalışma zamanında hatalara yol açabilir
        try
        {
            // Bu satır çalışma zamanında ArrayTypeMismatchException fırlatır
            // customers[1] = new CorporateCustomer { Id = "K001", FirstName = "ABC", LastName = "Ltd. Şti." };
        }
        catch (ArrayTypeMismatchException ex)
        {
            Console.WriteLine($"Hata: {ex.Message}");
        }
    }
}
```

## 5. Generic Varyans Kuralları (Generic Variance Rules)

Generic variance kullanırken dikkat edilmesi gereken bazı kurallar vardır:

### Temel Kurallar

```csharp
// Generic varyans kuralları
public class VarianceRules
{
    public void ShowRules()
    {
        // 1. Covariance (out) yalnızca dönüş değerleri için kullanılabilir
        ICustomerInfoProvider<IndividualCustomer> individualProvider = new IndividualCustomerInfoProvider();
        ICustomerInfoProvider<Customer> generalProvider = individualProvider; // Geçerli
        
        // 2. Contravariance (in) yalnızca parametreler için kullanılabilir
        ICustomerProcessor<Customer> generalProcessor = new GeneralCustomerProcessor();
        ICustomerProcessor<IndividualCustomer> individualProcessor = generalProcessor; // Geçerli
        
        // 3. Değer tipleri için varyans kullanılamaz
        // IComparable<in T> için T değer tipi olamaz
        
        // 4. Aynı tip parametresi hem in hem de out olarak işaretlenemez
        // public interface IInvalid<in out T> { } // Geçersiz
        
        // 5. Generic sınıflar varyans desteklemez, yalnızca arayüzler ve delegeler destekler
        // public class Covariant<out T> { } // Geçersiz
    }
}
```

###  Varyans Kuralları

```csharp
//  varyans kuralları
public interface ITransactionRecord<out T> where T : TransactionDetail
{
    T GetTransactionDetail();
    IEnumerable<T> GetTransactionHistory();
}

public interface ITransactionProcessor<in T> where T : TransactionDetail
{
    void SaveTransaction(T transaction);
    bool IsTransactionValid(T transaction);
}

public class TransactionDetail
{
    public string TransactionId { get; set; }
    public DateTime TransactionDate { get; set; }
    public decimal Amount { get; set; }
}

public class MoneyTransferDetail : TransactionDetail
{
    public string SenderAccountNumber { get; set; }
    public string ReceiverAccountNumber { get; set; }
    public string Description { get; set; }
}

// Varyans kullanımı
public class TransactionService
{
    public void TransactionManagement()
    {
        // Covariance örneği
        ITransactionRecord<MoneyTransferDetail> transferRecord = new MoneyTransferRecord();
        ITransactionRecord<TransactionDetail> generalRecord = transferRecord; // Covariance sayesinde geçerli
        
        TransactionDetail transaction = generalRecord.GetTransactionDetail();
        
        // Contravariance örneği
        ITransactionProcessor<TransactionDetail> generalProcessor = new GeneralTransactionProcessor();
        ITransactionProcessor<MoneyTransferDetail> transferProcessor = generalProcessor; // Contravariance sayesinde geçerli
        
        transferProcessor.SaveTransaction(new MoneyTransferDetail 
        { 
            TransactionId = "T001", 
            TransactionDate = DateTime.Now, 
            Amount = 1000, 
            SenderAccountNumber = "123456", 
            ReceiverAccountNumber = "789012",
            Description = "Kira ödemesi"
        });
    }
}

public class MoneyTransferRecord : ITransactionRecord<MoneyTransferDetail>
{
    public MoneyTransferDetail GetTransactionDetail()
    {
        return new MoneyTransferDetail
        {
            TransactionId = "T001",
            TransactionDate = DateTime.Now,
            Amount = 1000,
            SenderAccountNumber = "123456",
            ReceiverAccountNumber = "789012",
            Description = "Kira ödemesi"
        };
    }
    
    public IEnumerable<MoneyTransferDetail> GetTransactionHistory()
    {
        // İşlem geçmişini getirme
        return new List<MoneyTransferDetail>();
    }
}

public class GeneralTransactionProcessor : ITransactionProcessor<TransactionDetail>
{
    public void SaveTransaction(TransactionDetail transaction)
    {
        Console.WriteLine($"İşlem kaydediliyor: {transaction.TransactionId}, Tutar: {transaction.Amount}");
        // İşlem kaydetme
    }
    
    public bool IsTransactionValid(TransactionDetail transaction)
    {
        // İşlem geçerlilik kontrolü
        return transaction.Amount > 0;
    }
}
```

## 6. Varyans En İyi Uygulamaları (Variance Best Practices)

Covariance ve contravariance kullanırken izlenmesi gereken bazı en iyi uygulamalar vardır.

###  Varyans En İyi Uygulamaları

```csharp
// Varyans için en iyi uygulamalar
public class VarianceBestPractices
{
    public void BestPractices()
    {
        // 1. Arayüzleri tasarlarken, tip parametrelerinin rolünü düşünün
        // - Yalnızca üretilen (çıkış) değerler için 'out' kullanın
        // - Yalnızca tüketilen (giriş) değerler için 'in' kullanın
        
        // 2. Arayüz hiyerarşilerinde varyansı tutarlı bir şekilde kullanın
        
        // İyi tasarlanmış arayüz hiyerarşisi örneği
        IReportGenerator<CustomerReport> customerReportGenerator = new CustomerReportGenerator();
        IReportGenerator<Report> generalReportGenerator = customerReportGenerator; // Covariance sayesinde geçerli
        
        IReportProcessor<Report> generalReportProcessor = new GeneralReportProcessor();
        IReportProcessor<CustomerReport> customerReportProcessor = generalReportProcessor; // Contravariance sayesinde geçerli
        
        // 3. Generic koleksiyonlarda varyansı dikkatli kullanın
        // IEnumerable<T> covariant'tır, ancak IList<T> değildir
        
        IEnumerable<IndividualCustomer> individualCustomers = new List<IndividualCustomer>();
        IEnumerable<Customer> customers = individualCustomers; // Geçerli, IEnumerable<out T> covariant'tır
        
        // List<IndividualCustomer> individualCustomerList = new List<IndividualCustomer>();
        // List<Customer> customerList = individualCustomerList; // Geçersiz, List<T> invariant'tır
        
        // 4. Varyansı yalnızca gerektiğinde kullanın
        // Çoğu durumda, generic tiplerin invariant olması yeterlidir
    }
}

// Rapor sınıf hiyerarşisi
public class Report
{
    public string ReportId { get; set; }
    public DateTime CreationDate { get; set; }
}

public class CustomerReport : Report
{
    public string CustomerId { get; set; }
    public string CustomerName { get; set; }
}

// Covariant rapor oluşturucu arayüzü
public interface IReportGenerator<out T> where T : Report
{
    T GenerateReport();
    IEnumerable<T> GetAllReports();
}

// Contravariant rapor işleyici arayüzü
public interface IReportProcessor<in T> where T : Report
{
    void SaveReport(T report);
    void SendReport(T report, string email);
}

// Uygulama sınıfları
public class CustomerReportGenerator : IReportGenerator<CustomerReport>
{
    public CustomerReport GenerateReport()
    {
        return new CustomerReport { ReportId = "MR001", CreationDate = DateTime.Now, CustomerId = "M001", CustomerName = "Ahmet Yılmaz" };
    }
    
    public IEnumerable<CustomerReport> GetAllReports()
    {
        return new List<CustomerReport>();
    }
}

public class GeneralReportProcessor : IReportProcessor<Report>
{
    public void SaveReport(Report report)
    {
        Console.WriteLine($"Rapor kaydediliyor: {report.ReportId}");
    }
    
    public void SendReport(Report report, string email)
    {
        Console.WriteLine($"Rapor gönderiliyor: {report.ReportId}, Alıcı: {email}");
    }
}
```

## Özet

Covariance ve contravariance, C#'ta generic tiplerin ve delegelerin tip uyumluluğunu genişleten güçlü kavramlardır. Bu bölümde, covariance ve contravariance kavramlarını ve bunların banka uygulamalarındaki kullanımını inceledik:

- **in ve out Anahtar Kelimeleri**: Generic arayüzlerde ve delegelerde covariance ve contravariance'ı belirtmek için kullanılır.
- **Delege Varyansı**: Delegelerin daha esnek bir şekilde kullanılmasını sağlar.
- **Arayüz Varyansı**: Generic arayüzlerde tip parametrelerinin nasıl kullanıldığına bağlıdır.
- **Dizi Covariance**: Diziler doğal olarak covariant'tır, ancak bu tip güvenliği açısından sorunlara yol açabilir.
- **Generic Varyans Kuralları**: Generic varyans kullanırken dikkat edilmesi gereken kurallar.
- **Varyans En İyi Uygulamaları**: Covariance ve contravariance kullanırken izlenmesi gereken en iyi uygulamalar.

Covariance ve contravariance, özellikle banka gibi karmaşık iş mantığına sahip uygulamalarda, kodun daha esnek ve yeniden kullanılabilir olmasını sağlar. Ancak, bu kavramları doğru bir şekilde anlamak ve uygulamak önemlidir. 