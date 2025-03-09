# OOP Temelleri

Nesne Yönelimli Programlama (Object-Oriented Programming - OOP), yazılım geliştirmenin temel paradigmalarından biridir. C#, OOP prensiplerini tam olarak destekleyen ve bu prensipleri etkin bir şekilde kullanmanıza olanak tanıyan bir dildir. Bu bölümde, OOP'nin temel kavramlarını ve C#'ta nasıl uygulandıklarını gerçek dünya örnekleriyle inceleyeceğiz.

## 1. Sınıf ve Nesne Kavramları

Sınıflar, nesnelerin şablonlarıdır ve nesneler bu şablonlardan oluşturulan somut örneklerdir. Gerçek dünyadan bir örnek olarak, "Müşteri" bir sınıf olabilir ve her bir müşteri bu sınıfın bir nesnesidir.

### Sınıf Tanımlama

```csharp
// Banka müşterisi sınıfı
public class Customer
{
    // Alanlar (Fields)
    private int _id;
    private string _name;
    
    // Özellikler (Properties)
    public int Id { get { return _id; } }
    public string Name { get; set; }
    public string Email { get; set; }
    public decimal Balance { get; private set; }
    
    // Constructor (Yapıcı metot)
    public Customer(int id, string name, string email)
    {
        _id = id;
        Name = name;
        Email = email;
        Balance = 0;
    }
    
    // Metotlar
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Yatırılan miktar pozitif olmalıdır.");
        
        Balance += amount;
    }
    
    public bool Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Çekilen miktar pozitif olmalıdır.");
        
        if (Balance < amount)
            return false; // Yetersiz bakiye
        
        Balance -= amount;
        return true;
    }
}
```

### Nesne Oluşturma ve Kullanma

```csharp
// Müşteri nesnesi oluşturma
Customer customer1 = new Customer(1, "Ahmet Yılmaz", "ahmet@example.com");
Customer customer2 = new Customer(2, "Ayşe Demir", "ayse@example.com");

// Nesne özelliklerine erişim ve değiştirme
Console.WriteLine(customer1.Name); // Ahmet Yılmaz
customer1.Name = "Ahmet Y. Yılmaz";

// Nesne metotlarını çağırma
customer1.Deposit(1000);
bool success = customer1.Withdraw(250);
Console.WriteLine(customer1.Balance); // 750
```

## 2. Encapsulation (Kapsülleme)

Kapsülleme, bir sınıfın iç detaylarını gizleyerek, dış dünyaya sadece gerekli arayüzü sunma prensibidir. Bu, veri güvenliğini sağlar ve sınıfın iç yapısının değişmesine rağmen dış arayüzün sabit kalmasına olanak tanır.

### Kapsülleme Örneği

```csharp
public class BankAccount
{
    // Private alanlar - dışarıdan doğrudan erişilemez
    private string _accountNumber;
    private decimal _balance;
    private readonly decimal _minimumBalance = 100;
    
    // Public özellikler - kontrollü erişim sağlar
    public string AccountNumber { get { return _accountNumber; } }
    public decimal Balance { get { return _balance; } }
    
    public BankAccount(string accountNumber, decimal initialDeposit)
    {
        if (initialDeposit < _minimumBalance)
            throw new ArgumentException($"Başlangıç bakiyesi en az {_minimumBalance:C} olmalıdır.");
        
        _accountNumber = accountNumber;
        _balance = initialDeposit;
    }
    
    // Public metotlar - dış dünyaya sunulan arayüz
    public void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Yatırılan miktar pozitif olmalıdır.");
        
        _balance += amount;
    }
    
    public bool Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Çekilen miktar pozitif olmalıdır.");
        
        if (_balance - amount < _minimumBalance)
            return false; // Minimum bakiye koruması
        
        _balance -= amount;
        return true;
    }
}
```

Kapsülleme sayesinde:
- `_balance` alanına doğrudan erişim engellenmiştir, sadece `Deposit` ve `Withdraw` metotları üzerinden değiştirilebilir.
- Minimum bakiye kontrolü gibi iş kuralları uygulanabilir.
- Hesap numarası oluşturulduktan sonra değiştirilemez.

## 3. Inheritance (Kalıtım)

Kalıtım, bir sınıfın başka bir sınıfın özelliklerini ve davranışlarını miras almasını sağlar. Bu, kod tekrarını azaltır ve hiyerarşik ilişkiler kurmaya olanak tanır.

### Kalıtım Örneği

```csharp
// Temel sınıf
public class BankAccount
{
    public string AccountNumber { get; }
    public decimal Balance { get; protected set; }
    
    public BankAccount(string accountNumber, decimal initialBalance)
    {
        AccountNumber = accountNumber;
        Balance = initialBalance;
    }
    
    public virtual void Deposit(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Yatırılan miktar pozitif olmalıdır.");
        
        Balance += amount;
    }
    
    public virtual bool Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Çekilen miktar pozitif olmalıdır.");
        
        if (Balance < amount)
            return false;
        
        Balance -= amount;
        return true;
    }
}

// Türetilmiş sınıf - Tasarruf Hesabı
public class SavingsAccount : BankAccount
{
    public decimal InterestRate { get; }
    
    public SavingsAccount(string accountNumber, decimal initialBalance, decimal interestRate)
        : base(accountNumber, initialBalance)
    {
        InterestRate = interestRate;
    }
    
    public void ApplyInterest()
    {
        decimal interest = Balance * InterestRate;
        Deposit(interest);
    }
}

// Türetilmiş sınıf - Çek Hesabı
public class CheckingAccount : BankAccount
{
    public decimal OverdraftLimit { get; }
    
    public CheckingAccount(string accountNumber, decimal initialBalance, decimal overdraftLimit)
        : base(accountNumber, initialBalance)
    {
        OverdraftLimit = overdraftLimit;
    }
    
    // Metot override - çek hesabı için özel çekme kuralları
    public override bool Withdraw(decimal amount)
    {
        if (amount <= 0)
            throw new ArgumentException("Çekilen miktar pozitif olmalıdır.");
        
        // Overdraft limitini de hesaba katarak çekim yapabilir
        if (Balance + OverdraftLimit < amount)
            return false;
        
        Balance -= amount;
        return true;
    }
}
```

Kalıtım sayesinde:
- `SavingsAccount` ve `CheckingAccount` sınıfları, `BankAccount` sınıfının tüm özelliklerini ve metotlarını miras alır.
- Her hesap türü kendi özel davranışlarını ekleyebilir (örn. faiz uygulama).
- `CheckingAccount` sınıfı, `Withdraw` metodunu kendi ihtiyaçlarına göre override edebilir.

## 4. Polymorphism (Çok Biçimlilik)

Çok biçimlilik, aynı arayüzü kullanarak farklı türdeki nesnelerin farklı davranışlar sergilemesini sağlar. Bu, kodun daha esnek ve genişletilebilir olmasını sağlar.

### Çok Biçimlilik Örneği

```csharp
// Banka işlemi temel sınıfı
public abstract class BankTransaction
{
    public DateTime Date { get; }
    public decimal Amount { get; }
    
    protected BankTransaction(decimal amount)
    {
        Date = DateTime.Now;
        Amount = amount;
    }
    
    // Soyut metot - tüm işlemler bunu uygulamalıdır
    public abstract void Execute(BankAccount account);
    
    // Sanal metot - isteğe bağlı override edilebilir
    public virtual string GetDescription()
    {
        return $"{Date:dd/MM/yyyy HH:mm}: İşlem tutarı {Amount:C}";
    }
}

// Para yatırma işlemi
public class DepositTransaction : BankTransaction
{
    public DepositTransaction(decimal amount) : base(amount) { }
    
    public override void Execute(BankAccount account)
    {
        account.Deposit(Amount);
    }
    
    public override string GetDescription()
    {
        return $"{base.GetDescription()} - Para Yatırma";
    }
}

// Para çekme işlemi
public class WithdrawalTransaction : BankTransaction
{
    public WithdrawalTransaction(decimal amount) : base(amount) { }
    
    public override void Execute(BankAccount account)
    {
        if (!account.Withdraw(Amount))
            throw new InvalidOperationException("Yetersiz bakiye");
    }
    
    public override string GetDescription()
    {
        return $"{base.GetDescription()} - Para Çekme";
    }
}

// Kullanım
public void ProcessTransactions(BankAccount account, List<BankTransaction> transactions)
{
    foreach (var transaction in transactions)
    {
        try
        {
            transaction.Execute(account);
            Console.WriteLine(transaction.GetDescription());
        }
        catch (Exception ex)
        {
            Console.WriteLine($"İşlem hatası: {ex.Message}");
        }
    }
}
```

Çok biçimlilik sayesinde:
- Farklı işlem türleri (`DepositTransaction`, `WithdrawalTransaction`) aynı arayüzü (`BankTransaction`) uygular.
- `ProcessTransactions` metodu, işlemin türünü bilmeden herhangi bir işlemi çalıştırabilir.
- Yeni işlem türleri eklemek için mevcut kodu değiştirmek gerekmez.

## 5. Abstraction (Soyutlama)

Soyutlama, karmaşık sistemleri daha basit arayüzler arkasında gizleyerek, kullanıcının sadece gerekli detaylarla ilgilenmesini sağlar. Soyut sınıflar ve arayüzler, soyutlama için temel mekanizmalardır.

# ❤️ Aşk ve Abstraction ❤️

> Eşim Seda'nın gözlerine baktığımda sadece aşk görürüm;  
> karaciğerinin veya pankreasının nasıl çalıştığını bilmem,  
> başka şeyleri önemsemeden onu olduğu gibi severim.  

*abstraction* da aynen böyledir:  
Bir nesnenin iç detayları yerine, bize gerekli olan görünür ve onla ilgileniriz.


### Soyutlama Örneği

```csharp
// Ödeme işlemi arayüzü
public interface IPaymentProcessor
{
    bool ProcessPayment(decimal amount);
    void RefundPayment(string transactionId, decimal amount);
    string GetPaymentMethod();
}

// Kredi kartı ödeme işlemcisi
public class CreditCardProcessor : IPaymentProcessor
{
    private readonly string _cardNumber;
    private readonly string _cardHolderName;
    
    public CreditCardProcessor(string cardNumber, string cardHolderName)
    {
        _cardNumber = cardNumber;
        _cardHolderName = cardHolderName;
    }
    
    public bool ProcessPayment(decimal amount)
    {
        // Kredi kartı ödeme işlemi gerçekleştirme
        Console.WriteLine($"Kredi kartı ile {amount:C} ödeme işlemi gerçekleştiriliyor...");
        // Gerçek uygulamada burada kart doğrulama, ödeme ağı ile iletişim vb. olur
        return true;
    }
    
    public void RefundPayment(string transactionId, decimal amount)
    {
        Console.WriteLine($"Kredi kartına {amount:C} iade işlemi gerçekleştiriliyor...");
    }
    
    public string GetPaymentMethod()
    {
        return $"Kredi Kartı: {MaskCardNumber(_cardNumber)}";
    }
    
    private string MaskCardNumber(string cardNumber)
    {
        // Kart numarasının son 4 hanesi hariç maskeleme
        return "XXXX-XXXX-XXXX-" + cardNumber.Substring(cardNumber.Length - 4);
    }
}

// Banka havalesi ödeme işlemcisi
public class BankTransferProcessor : IPaymentProcessor
{
    private readonly string _accountNumber;
    private readonly string _bankName;
    
    public BankTransferProcessor(string accountNumber, string bankName)
    {
        _accountNumber = accountNumber;
        _bankName = bankName;
    }
    
    public bool ProcessPayment(decimal amount)
    {
        Console.WriteLine($"Banka havalesi ile {amount:C} ödeme işlemi gerçekleştiriliyor...");
        return true;
    }
    
    public void RefundPayment(string transactionId, decimal amount)
    {
        Console.WriteLine($"Banka hesabına {amount:C} iade işlemi gerçekleştiriliyor...");
    }
    
    public string GetPaymentMethod()
    {
        return $"Banka Havalesi: {_bankName} / {_accountNumber}";
    }
}

// Ödeme işlemi gerçekleştiren sınıf
public class PaymentService
{
    public bool MakePayment(IPaymentProcessor processor, decimal amount)
    {
        Console.WriteLine($"Ödeme yöntemi: {processor.GetPaymentMethod()}");
        return processor.ProcessPayment(amount);
    }
}
```

Soyutlama sayesinde:
- `PaymentService` sınıfı, ödeme işlemcisinin iç detaylarını bilmeden ödeme işlemi gerçekleştirebilir.
- Farklı ödeme yöntemleri (`CreditCardProcessor`, `BankTransferProcessor`) aynı arayüzü uygular.
- Yeni ödeme yöntemleri eklemek için mevcut kodu değiştirmek gerekmez.

## 6. Association, Aggregation, Composition

Bu kavramlar, sınıflar arasındaki ilişkileri tanımlar ve nesne yönelimli tasarımın temel yapı taşlarıdır.

### Association (İlişki)

Association, iki sınıf arasındaki genel bir ilişkiyi ifade eder. Sınıflar birbirinden bağımsızdır ve birbirlerini kullanırlar.

```csharp
// Müşteri sınıfı
public class Customer
{
    public int Id { get; }
    public string Name { get; set; }
    
    public Customer(int id, string name)
    {
        Id = id;
        Name = name;
    }
    
    // Banka ile ilişki - müşteri bankayı kullanır
    public void MakeTransaction(Bank bank, decimal amount, TransactionType type)
    {
        bank.ProcessTransaction(this, amount, type);
    }
}

// Banka sınıfı
public class Bank
{
    public string Name { get; }
    
    public Bank(string name)
    {
        Name = name;
    }
    
    // Müşteri ile ilişki - banka müşteriyi tanır
    public void ProcessTransaction(Customer customer, decimal amount, TransactionType type)
    {
        Console.WriteLine($"{Name} bankası, {customer.Name} müşterisi için {amount:C} tutarında " +
                         $"{(type == TransactionType.Deposit ? "yatırma" : "çekme")} işlemi gerçekleştiriyor.");
    }
}

public enum TransactionType { Deposit, Withdrawal }
```

### Aggregation (Birleştirme)

Aggregation, "sahiptir" ilişkisini ifade eder, ancak parçalar bağımsız olarak var olabilir.

```csharp
// Departman sınıfı
public class Department
{
    public string Name { get; }
    private List<Employee> _employees;
    
    public Department(string name)
    {
        Name = name;
        _employees = new List<Employee>();
    }
    
    // Çalışan ekleme (aggregation)
    public void AddEmployee(Employee employee)
    {
        _employees.Add(employee);
    }
    
    // Çalışan çıkarma
    public void RemoveEmployee(Employee employee)
    {
        _employees.Remove(employee);
    }
}

// Çalışan sınıfı
public class Employee
{
    public int Id { get; }
    public string Name { get; set; }
    
    public Employee(int id, string name)
    {
        Id = id;
        Name = name;
    }
}

// Kullanım
Employee emp1 = new Employee(1, "Mehmet Kaya");
Department itDepartment = new Department("Bilgi Teknolojileri");

// Departman çalışanlara sahiptir (aggregation)
itDepartment.AddEmployee(emp1);

// Çalışan departmandan ayrılabilir ve bağımsız olarak var olmaya devam eder
itDepartment.RemoveEmployee(emp1);
```

### Composition (Bileşim)

Composition, "parçasıdır" ilişkisini ifade eder. Parçalar, bütünün yaşam döngüsüne sıkı sıkıya bağlıdır.

```csharp
// Banka sınıfı
public class Bank
{
    public string Name { get; }
    private List<BankAccount> _accounts;
    
    public Bank(string name)
    {
        Name = name;
        _accounts = new List<BankAccount>();
    }
    
    // Yeni hesap oluşturma (composition)
    public BankAccount CreateAccount(Customer owner, decimal initialDeposit)
    {
        string accountNumber = GenerateAccountNumber();
        var account = new BankAccount(accountNumber, owner, initialDeposit, this);
        _accounts.Add(account);
        return account;
    }
    
    // Hesap kapatma
    public void CloseAccount(BankAccount account)
    {
        if (_accounts.Remove(account))
        {
            account.Close();
        }
    }
    
    // Hesap numarası üretme
    private string GenerateAccountNumber()
    {
        return "ACC-" + Guid.NewGuid().ToString().Substring(0, 8).ToUpper();
    }
    
    // İç sınıf: Banka Hesabı (composition)
    public class BankAccount
    {
        public string AccountNumber { get; }
        public Customer Owner { get; }
        public decimal Balance { get; private set; }
        private Bank _bank;
        private bool _isClosed;
        
        // Constructor - sadece Bank sınıfı tarafından çağrılabilir
        internal BankAccount(string accountNumber, Customer owner, decimal initialDeposit, Bank bank)
        {
            AccountNumber = accountNumber;
            Owner = owner;
            Balance = initialDeposit;
            _bank = bank;
            _isClosed = false;
        }
        
        public void Deposit(decimal amount)
        {
            if (_isClosed)
                throw new InvalidOperationException("Kapalı hesaba işlem yapılamaz.");
            
            Balance += amount;
        }
        
        public bool Withdraw(decimal amount)
        {
            if (_isClosed)
                throw new InvalidOperationException("Kapalı hesaba işlem yapılamaz.");
            
            if (Balance < amount)
                return false;
            
            Balance -= amount;
            return true;
        }
        
        internal void Close()
        {
            _isClosed = true;
        }
    }
}
```

Bu örnekte:
- `BankAccount` nesneleri sadece `Bank` sınıfı tarafından oluşturulabilir.
- Hesap kapatıldığında, hesap nesnesi artık kullanılamaz.
- Hesaplar, bankanın yaşam döngüsüne bağlıdır.

## En İyi Pratikler

1. **Sınıf Tasarımı**
   - Her sınıf tek bir sorumluluğa sahip olmalıdır (Single Responsibility Principle).
   - Sınıflar, gerektiğinde genişletilebilir ancak değiştirilmesi zor olmalıdır (Open/Closed Principle).
   - Sınıf isimleri, sınıfın amacını açıkça belirtmelidir.

2. **Encapsulation (Kapsülleme)**
   - Sınıf üyelerini mümkün olan en kısıtlayıcı erişim belirleyicisiyle tanımlayın.
   - Veri doğrulama ve iş kurallarını property'lerin setter'larında uygulayın.
   - İç durumu korumak için immutable nesneler kullanmayı düşünün.

3. **Inheritance (Kalıtım)**
   - Kalıtımı sadece gerçek "is-a" ilişkisi olduğunda kullanın.
   - Kalıtım hiyerarşisini çok derin yapmaktan kaçının (genellikle 2-3 seviye yeterlidir).
   - Davranış paylaşımı için interface'leri tercih edin.

4. **Polymorphism (Çok Biçimlilik)**
   - Ortak davranışları tanımlamak için interface'leri kullanın.
   - Metotları gerektiğinde `virtual` olarak işaretleyin.
   - Farklı tipleri ortak bir interface üzerinden işleyin.

5. **Abstraction (Soyutlama)**
   - Karmaşık sistemleri daha basit arayüzler arkasında gizleyin.
   - İmplementasyon detaylarını değil, davranışı ve yetenekleri açığa çıkarın.
   - Soyutlama için interface'leri ve abstract sınıfları kullanın.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Aşırı Karmaşık Sınıflar**
   ```csharp
   // Kötü - çok fazla sorumluluğa sahip sınıf
   public class BankAccount
   {
       public void DepositMoney() { /* ... */ }
       public void WithdrawMoney() { /* ... */ }
       public void CalculateInterest() { /* ... */ }
       public void SendEmail() { /* ... */ }
       public void GenerateStatement() { /* ... */ }
       public void ValidateUser() { /* ... */ }
       public void ConnectToDatabase() { /* ... */ }
   }
   
   // İyi - tek sorumluluğa sahip sınıflar
   public class BankAccount
   {
       public void DepositMoney() { /* ... */ }
       public void WithdrawMoney() { /* ... */ }
       public void CalculateInterest() { /* ... */ }
   }
   
   public class EmailService
   {
       public void SendEmail() { /* ... */ }
   }
   
   public class StatementGenerator
   {
       public void GenerateStatement() { /* ... */ }
   }
   ```

2. **Yetersiz Kapsülleme**
   ```csharp
   // Kötü - public alanlar
   public class Customer
   {
       public string Name;
       public decimal Balance;
       
       public void UpdateBalance(decimal amount)
       {
           Balance += amount;
       }
   }
   
   // İyi - private alanlar ve property'ler
   public class Customer
   {
       private string _name;
       private decimal _balance;
       
       public string Name
       {
           get { return _name; }
           set { _name = value; }
       }
       
       public decimal Balance
       {
           get { return _balance; }
           private set { _balance = value; }
       }
       
       public void UpdateBalance(decimal amount)
       {
           if (amount + Balance < 0)
               throw new InvalidOperationException("Yetersiz bakiye");
               
           Balance += amount;
       }
   }
   ```

3. **Kalıtımın Yanlış Kullanımı**
   ```csharp
   // Kötü - uygunsuz kalıtım
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
       private int _size;
       
       public override int Width
       {
           get { return _size; }
           set { _size = value; }
       }
       
       public override int Height
       {
           get { return _size; }
           set { _size = value; }
       }
   }
   
   // İyi - composition kullanımı
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
       public int Size { get; set; }
       
       public int CalculateArea()
       {
           return Size * Size;
       }
   }
   ```

4. **Soyutlamanın Yetersiz Kullanımı**
   ```csharp
   // Kötü - implementasyon detaylarına bağımlılık
   public class OrderProcessor
   {
       public void ProcessOrder(Order order)
       {
           SqlConnection connection = new SqlConnection("connection_string");
           connection.Open();
           
           // SQL Server'a özgü işlemler...
           
           connection.Close();
       }
   }
   
   // İyi - soyutlama kullanımı
   public interface IOrderRepository
   {
       void SaveOrder(Order order);
   }
   
   public class SqlOrderRepository : IOrderRepository
   {
       public void SaveOrder(Order order)
       {
           // SQL Server implementasyonu
       }
   }
   
   public class OrderProcessor
   {
       private readonly IOrderRepository _repository;
       
       public OrderProcessor(IOrderRepository repository)
       {
           _repository = repository;
       }
       
       public void ProcessOrder(Order order)
       {
           // İş mantığı...
           _repository.SaveOrder(order);
       }
   }
   ```

5. **Association, Aggregation ve Composition'ın Karıştırılması**
   ```csharp
   // Kötü - ilişki türlerinin karıştırılması
   public class Department
   {
       private List<Employee> _employees = new List<Employee>();
       
       public Department()
       {
           // Composition gibi davranış - Department oluşturulduğunda Employee'ler de oluşturuluyor
           _employees.Add(new Employee("John"));
           _employees.Add(new Employee("Jane"));
       }
       
       public void AddEmployee(Employee employee)
       {
           // Aggregation gibi davranış - dışarıdan Employee ekleniyor
           _employees.Add(employee);
       }
       
       // Department silindiğinde Employee'ler de siliniyor (Composition)
   }
   
   // İyi - açık ilişki tanımları
   // Composition örneği
   public class Car
   {
       private Engine _engine;
       
       public Car()
       {
           _engine = new Engine(); // Car, Engine'i oluşturur ve kontrol eder
       }
   }
   
   // Aggregation örneği
   public class Department
   {
       private List<Employee> _employees = new List<Employee>();
       
       public void AddEmployee(Employee employee)
       {
           _employees.Add(employee);
       }
       
       public void RemoveEmployee(Employee employee)
       {
           _employees.Remove(employee);
       }
   }
   
   // Association örneği
   public class Doctor
   {
       public void TreatPatient(Patient patient)
       {
           // Doctor, Patient ile çalışır ama onu kontrol etmez
       }
   }
   ```

OOP prensiplerini doğru şekilde uygulamak, daha modüler, bakımı kolay ve genişletilebilir kod yazmanıza yardımcı olur. Bu prensipler, gerçek dünya problemlerini yazılım çözümlerine dönüştürmenin etkili bir yolunu sunar. 