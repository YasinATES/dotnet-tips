# Kalıtım (Inheritance)

Kalıtım, nesne yönelimli programlamanın temel prensiplerinden biridir ve bir sınıfın başka bir sınıfın özelliklerini ve davranışlarını miras almasını sağlar. Bu bölümde, C#'ta kalıtım kavramını ve ilgili konuları detaylı olarak inceleyeceğiz.

## 1. Base ve Derived Sınıflar

Kalıtımda, bir sınıf (derived/child class) başka bir sınıftan (base/parent class) türetilir ve onun özelliklerini ve davranışlarını miras alır.

### Temel Kalıtım Örneği

```csharp
// Base sınıf (temel sınıf)
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime DateOfBirth { get; set; }
    
    public Person(string firstName, string lastName, DateTime dateOfBirth)
    {
        FirstName = firstName;
        LastName = lastName;
        DateOfBirth = dateOfBirth;
    }
    
    public string GetFullName()
    {
        return $"{FirstName} {LastName}";
    }
    
    public int CalculateAge()
    {
        var today = DateTime.Today;
        var age = today.Year - DateOfBirth.Year;
        
        if (DateOfBirth.Date > today.AddYears(-age))
            age--;
            
        return age;
    }
}

// Derived sınıf (türetilmiş sınıf)
public class Employee : Person
{
    public int EmployeeId { get; set; }
    public string Department { get; set; }
    public decimal Salary { get; set; }
    
    public Employee(string firstName, string lastName, DateTime dateOfBirth, 
                   int employeeId, string department, decimal salary)
        : base(firstName, lastName, dateOfBirth) // Base sınıf constructor'ını çağırır
    {
        EmployeeId = employeeId;
        Department = department;
        Salary = salary;
    }
    
    public decimal CalculateAnnualSalary()
    {
        return Salary * 12;
    }
}
```

### Kalıtım Kullanımı

```csharp
// Person nesnesi oluşturma
Person person = new Person("Ahmet", "Yılmaz", new DateTime(1985, 5, 15));
Console.WriteLine($"İsim: {person.GetFullName()}, Yaş: {person.CalculateAge()}");

// Employee nesnesi oluşturma
Employee employee = new Employee("Ayşe", "Demir", new DateTime(1990, 10, 20), 
                                1001, "IT", 10000);
Console.WriteLine($"Çalışan: {employee.GetFullName()}, Departman: {employee.Department}");
Console.WriteLine($"Yaş: {employee.CalculateAge()}, Yıllık Maaş: {employee.CalculateAnnualSalary():C}");

// Polimorfizm - base sınıf referansı ile derived sınıf nesnesi
Person personRef = new Employee("Mehmet", "Kaya", new DateTime(1988, 3, 10), 
                               1002, "Finance", 12000);
Console.WriteLine($"İsim: {personRef.GetFullName()}, Yaş: {personRef.CalculateAge()}");
// personRef.CalculateAnnualSalary(); // Derleme hatası - Person sınıfında bu metot yok
```

## 2. virtual, override ve new Anahtar Kelimeleri

Bu anahtar kelimeler, kalıtım hiyerarşisinde metotların davranışını kontrol etmek için kullanılır.

### virtual ve override

```csharp
public class Shape
{
    public string Name { get; set; }
    
    public Shape(string name)
    {
        Name = name;
    }
    
    // Virtual metot - alt sınıflar tarafından override edilebilir
    public virtual double CalculateArea()
    {
        return 0;
    }
    
    public virtual string GetInfo()
    {
        return $"Bu bir {Name} şeklidir.";
    }
}

public class Circle : Shape
{
    public double Radius { get; set; }
    
    public Circle(double radius) : base("Daire")
    {
        Radius = radius;
    }
    
    // Base sınıftaki virtual metodu override eder
    public override double CalculateArea()
    {
        return Math.PI * Radius * Radius;
    }
    
    // Base sınıftaki virtual metodu override eder ve genişletir
    public override string GetInfo()
    {
        return $"{base.GetInfo()} Yarıçapı {Radius} birimdir.";
    }
}

public class Rectangle : Shape
{
    public double Width { get; set; }
    public double Height { get; set; }
    
    public Rectangle(double width, double height) : base("Dikdörtgen")
    {
        Width = width;
        Height = height;
    }
    
    // Base sınıftaki virtual metodu override eder
    public override double CalculateArea()
    {
        return Width * Height;
    }
    
    // Base sınıftaki virtual metodu override eder ve genişletir
    public override string GetInfo()
    {
        return $"{base.GetInfo()} Boyutları {Width}x{Height} birimdir.";
    }
}
```

### new Anahtar Kelimesi

```csharp
public class BaseClass
{
    public void Display()
    {
        Console.WriteLine("BaseClass.Display() metodu çağrıldı");
    }
}

public class DerivedClass : BaseClass
{
    // new anahtar kelimesi ile base sınıftaki metodu gizler
    public new void Display()
    {
        Console.WriteLine("DerivedClass.Display() metodu çağrıldı");
    }
}

// Kullanım
BaseClass baseObj = new BaseClass();
baseObj.Display(); // "BaseClass.Display() metodu çağrıldı"

DerivedClass derivedObj = new DerivedClass();
derivedObj.Display(); // "DerivedClass.Display() metodu çağrıldı"

// Polimorfizm ile
BaseClass polymorphicObj = new DerivedClass();
polymorphicObj.Display(); // "BaseClass.Display() metodu çağrıldı" - new kullanıldığı için!
```

`new` anahtar kelimesi, base sınıftaki bir üyeyi gizler, ancak polimorfik davranışı değiştirmez. Bu nedenle, genellikle `virtual` ve `override` kullanmak daha iyidir.

## 3. Abstract Sınıflar ve Metotlar

Abstract sınıflar, doğrudan örneklenemeyen ve genellikle türetilmiş sınıflar için bir şablon görevi gören sınıflardır. Abstract metotlar ise implementasyonu olmayan ve türetilmiş sınıflar tarafından override edilmesi gereken metotlardır.

### Abstract Sınıf ve Metot Örneği

```csharp
// Abstract sınıf
public abstract class Vehicle
{
    public string Brand { get; set; }
    public string Model { get; set; }
    public int Year { get; set; }
    
    // Normal constructor
    protected Vehicle(string brand, string model, int year)
    {
        Brand = brand;
        Model = model;
        Year = year;
    }
    
    // Normal metot
    public string GetVehicleInfo()
    {
        return $"{Year} {Brand} {Model}";
    }
    
    // Abstract metot - implementasyonu yok
    public abstract double CalculateFuelEfficiency();
    
    // Virtual metot - default implementasyonu var
    public virtual void StartEngine()
    {
        Console.WriteLine("Motor çalıştırıldı.");
    }
}

// Abstract sınıftan türetilen concrete sınıf
public class Car : Vehicle
{
    public int NumberOfDoors { get; set; }
    public double EngineSize { get; set; }
    
    public Car(string brand, string model, int year, int numberOfDoors, double engineSize)
        : base(brand, model, year)
    {
        NumberOfDoors = numberOfDoors;
        EngineSize = engineSize;
    }
    
    // Abstract metodu implement etmek zorunludur
    public override double CalculateFuelEfficiency()
    {
        // Basit bir hesaplama
        return 100 / (EngineSize * 5);
    }
    
    // Virtual metodu override etmek opsiyoneldir
    public override void StartEngine()
    {
        Console.WriteLine($"{Brand} {Model} aracının motoru çalıştırıldı.");
    }
}

// Başka bir türetilmiş sınıf
public class Motorcycle : Vehicle
{
    public bool HasSidecar { get; set; }
    
    public Motorcycle(string brand, string model, int year, bool hasSidecar)
        : base(brand, model, year)
    {
        HasSidecar = hasSidecar;
    }
    
    // Abstract metodu implement etmek zorunludur
    public override double CalculateFuelEfficiency()
    {
        // Basit bir hesaplama
        return HasSidecar ? 20 : 30;
    }
}
```

### Abstract Sınıf Kullanımı

```csharp
// Vehicle vehicle = new Vehicle(); // Derleme hatası - abstract sınıflar örneklenemez

Car car = new Car("Toyota", "Corolla", 2020, 4, 1.6);
Console.WriteLine($"Araç: {car.GetVehicleInfo()}");
Console.WriteLine($"Yakıt verimliliği: {car.CalculateFuelEfficiency()} km/lt");
car.StartEngine();

Motorcycle motorcycle = new Motorcycle("Honda", "CBR", 2021, false);
Console.WriteLine($"Araç: {motorcycle.GetVehicleInfo()}");
Console.WriteLine($"Yakıt verimliliği: {motorcycle.CalculateFuelEfficiency()} km/lt");
motorcycle.StartEngine(); // Base sınıftaki implementasyonu kullanır

// Polimorfizm
Vehicle vehicleRef = new Car("BMW", "X5", 2022, 5, 3.0);
Console.WriteLine($"Araç: {vehicleRef.GetVehicleInfo()}");
Console.WriteLine($"Yakıt verimliliği: {vehicleRef.CalculateFuelEfficiency()} km/lt");
vehicleRef.StartEngine();
```

## 4. Çoklu Kalıtım ve Interface Kullanımı

C#, sınıflar için çoklu kalıtımı doğrudan desteklemez, ancak interface'ler aracılığıyla çoklu kalıtım benzeri bir yapı sağlar.

### Interface Tanımlama ve Kullanma

```csharp
// Interface tanımlama
public interface IPayable
{
    decimal CalculatePayment();
    void ProcessPayment();
}

public interface ITaxable
{
    decimal CalculateTax();
}

// Birden fazla interface'i implement eden sınıf
public class Invoice : IPayable, ITaxable
{
    public string InvoiceNumber { get; set; }
    public decimal Amount { get; set; }
    public decimal TaxRate { get; set; }
    
    public Invoice(string invoiceNumber, decimal amount, decimal taxRate)
    {
        InvoiceNumber = invoiceNumber;
        Amount = amount;
        TaxRate = taxRate;
    }
    
    // IPayable interface'inden
    public decimal CalculatePayment()
    {
        return Amount + CalculateTax();
    }
    
    public void ProcessPayment()
    {
        Console.WriteLine($"Fatura {InvoiceNumber} için {CalculatePayment():C} ödeme işlendi.");
    }
    
    // ITaxable interface'inden
    public decimal CalculateTax()
    {
        return Amount * TaxRate;
    }
}

// Kalıtım ve interface'i birlikte kullanma
public class Employee : Person, IPayable
{
    public decimal Salary { get; set; }
    
    public Employee(string firstName, string lastName, decimal salary)
        : base(firstName, lastName)
    {
        Salary = salary;
    }
    
    // IPayable interface'inden
    public decimal CalculatePayment()
    {
        return Salary;
    }
    
    public void ProcessPayment()
    {
        Console.WriteLine($"{GetFullName()} için {CalculatePayment():C} maaş ödemesi yapıldı.");
    }
}
```

### Interface ve Polimorfizm

```csharp
// Interface referansı ile farklı nesneleri kullanma
IPayable payment1 = new Invoice("INV-001", 1000, 0.18m);
IPayable payment2 = new Employee("Ahmet", "Yılmaz", 5000);

// Aynı interface metotlarını çağırma
Console.WriteLine($"Ödeme 1: {payment1.CalculatePayment():C}");
payment1.ProcessPayment();

Console.WriteLine($"Ödeme 2: {payment2.CalculatePayment():C}");
payment2.ProcessPayment();

// Interface listesi
List<IPayable> payments = new List<IPayable>
{
    new Invoice("INV-002", 2000, 0.18m),
    new Invoice("INV-003", 3000, 0.18m),
    new Employee("Ayşe", "Demir", 6000),
    new Employee("Mehmet", "Kaya", 7000)
};

// Tüm ödemeleri işleme
foreach (var payment in payments)
{
    payment.ProcessPayment();
}
```

## 5. Sealed Methods ve Override Sealing

`sealed` anahtar kelimesi, bir sınıfın miras alınmasını veya bir metotun daha fazla override edilmesini engellemek için kullanılır.

### Sealed Sınıf

```csharp
// Sealed sınıf - miras alınamaz
public sealed class SecurityManager
{
    public void Authenticate(string username, string password)
    {
        // Kimlik doğrulama işlemleri
        Console.WriteLine($"{username} kimlik doğrulaması yapıldı.");
    }
    
    public void Authorize(string username, string resource)
    {
        // Yetkilendirme işlemleri
        Console.WriteLine($"{username} kullanıcısı {resource} kaynağına erişim yetkisi aldı.");
    }
}

// Aşağıdaki sınıf derleme hatası verir
// public class EnhancedSecurityManager : SecurityManager { }
```

### Sealed Override

```csharp
public class BaseClass
{
    public virtual void Method1()
    {
        Console.WriteLine("BaseClass.Method1");
    }
    
    public virtual void Method2()
    {
        Console.WriteLine("BaseClass.Method2");
    }
}

public class DerivedClass : BaseClass
{
    // Normal override
    public override void Method1()
    {
        Console.WriteLine("DerivedClass.Method1");
    }
    
    // Sealed override - daha fazla override edilemez
    public sealed override void Method2()
    {
        Console.WriteLine("DerivedClass.Method2 (sealed)");
    }
}

public class FurtherDerivedClass : DerivedClass
{
    // Bu geçerli - Method1 sealed değil
    public override void Method1()
    {
        Console.WriteLine("FurtherDerivedClass.Method1");
    }
    
    // Bu derleme hatası verir - Method2 sealed
    // public override void Method2() { }
}
```

## 6. Covariance ve Contravariance

Covariance ve contravariance, generic tiplerde ve delegate'lerde tip uyumluluğunu sağlayan kavramlardır.

### Covariance (out)

Covariance, bir metotun dönüş tipinin daha türetilmiş bir tip olabilmesini sağlar.

```csharp
// Covariance örneği - generic interface
public interface IProducer<out T>
{
    T Produce();
}

public class Animal { }
public class Dog : Animal { }

public class DogProducer : IProducer<Dog>
{
    public Dog Produce()
    {
        return new Dog();
    }
}

// Kullanım
IProducer<Dog> dogProducer = new DogProducer();
IProducer<Animal> animalProducer = dogProducer; // Covariance sayesinde geçerli
Animal animal = animalProducer.Produce(); // Aslında bir Dog döner
```

### Contravariance (in)

Contravariance, bir metotun parametre tipinin daha base bir tip olabilmesini sağlar.

```csharp
// Contravariance örneği - generic interface
public interface IConsumer<in T>
{
    void Consume(T item);
}

public class AnimalConsumer : IConsumer<Animal>
{
    public void Consume(Animal animal)
    {
        Console.WriteLine($"Consuming an animal: {animal.GetType().Name}");
    }
}

// Kullanım
IConsumer<Animal> animalConsumer = new AnimalConsumer();
IConsumer<Dog> dogConsumer = animalConsumer; // Contravariance sayesinde geçerli
dogConsumer.Consume(new Dog()); // AnimalConsumer.Consume metodu çağrılır
```

### Delegate'lerde Covariance ve Contravariance

```csharp
// Delegate tanımları
public delegate TResult Func<in T, out TResult>(T arg);

// Metotlar
public static Animal ConvertDogToAnimal(Dog dog)
{
    return dog; // Dog zaten bir Animal
}

public static void ProcessAnimal(Animal animal)
{
    Console.WriteLine($"Processing animal: {animal.GetType().Name}");
}

// Kullanım
Func<Dog, Animal> dogToAnimal = ConvertDogToAnimal;
Func<Dog, object> dogToObject = dogToAnimal; // Return type covariance

Action<Animal> animalAction = ProcessAnimal;
Action<Dog> dogAction = animalAction; // Parameter type contravariance
```

## En İyi Pratikler

1. **Kalıtım Kullanımı**
   - "is-a" ilişkisi varsa kalıtım kullanın (örn. Dog is an Animal).
   - "has-a" ilişkisi varsa composition kullanın (örn. Car has an Engine).
   - Kalıtım hiyerarşisini çok derin yapmaktan kaçının (genellikle 2-3 seviye yeterlidir).

2. **Interface vs Abstract Sınıf**
   - Sadece davranış tanımlamak istiyorsanız interface kullanın.
   - Ortak implementasyon paylaşmak istiyorsanız abstract sınıf kullanın.
   - Çoklu kalıtım gerekiyorsa interface'leri tercih edin.

3. **virtual, override ve new**
   - Türetilmiş sınıflarda davranış değişikliği gerekiyorsa `virtual` ve `override` kullanın.
   - `new` anahtar kelimesini sadece base sınıf davranışını tamamen değiştirmek istediğinizde kullanın.

4. **Sealed Kullanımı**
   - Güvenlik açısından kritik sınıfları veya performans optimizasyonu gerektiren sınıfları `sealed` olarak işaretleyin.
   - Bir metotun davranışının daha fazla değiştirilmemesi gerekiyorsa `sealed override` kullanın.

## Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Aşırı Derin Kalıtım Hiyerarşisi**
   ```csharp
   // Kötü - çok derin kalıtım hiyerarşisi
   public class A { }
   public class B : A { }
   public class C : B { }
   public class D : C { }
   public class E : D { }
   
   // İyi - daha düz hiyerarşi ve composition
   public class Base { }
   public class Derived1 : Base { }
   public class Derived2 : Base { }
   public class Composite
   {
       private Derived1 _component;
   }
   ```

2. **Kalıtım Yerine Composition Kullanmamak**
   ```csharp
   // Kötü - gereksiz kalıtım
   public class Engine { }
   public class Car : Engine { } // Araba bir motor değildir!
   
   // İyi - composition
   public class Car
   {
       private Engine _engine; // Arabanın motoru vardır
       
       public Car(Engine engine)
       {
           _engine = engine;
       }
   }
   ```

3. **Interface'leri Yanlış Kullanmak**
   ```csharp
   // Kötü - çok genel interface
   public interface IDoEverything
   {
       void SaveToDatabase();
       void SendEmail();
       void GenerateReport();
       void ProcessPayment();
   }
   
   // İyi - ayrı, odaklanmış interface'ler
   public interface IRepository
   {
       void Save(object entity);
   }
   
   public interface IEmailSender
   {
       void SendEmail(string to, string subject, string body);
   }
   ```

4. **new Anahtar Kelimesini Gereksiz Kullanmak**
   ```csharp
   // Kötü - new kullanımı
   public class Base
   {
       public virtual void Method()
       {
           Console.WriteLine("Base.Method");
       }
   }
   
   public class Derived : Base
   {
       public new void Method() // virtual metodu gizler
       {
           Console.WriteLine("Derived.Method");
       }
   }
   
   // İyi - override kullanımı
   public class BetterDerived : Base
   {
       public override void Method()
       {
           Console.WriteLine("BetterDerived.Method");
       }
   }
   ```

5. **Abstract Sınıfları Yanlış Kullanmak**
   ```csharp
   // Kötü - tüm metotları abstract olan abstract sınıf
   public abstract class Repository
   {
       public abstract void Add(object entity);
       public abstract void Update(object entity);
       public abstract void Delete(object entity);
       public abstract object GetById(int id);
   }
   
   // İyi - interface kullanımı
   public interface IRepository
   {
       void Add(object entity);
       void Update(object entity);
       void Delete(object entity);
       object GetById(int id);
   }
   
   // veya ortak implementasyon içeren abstract sınıf
   public abstract class RepositoryBase
   {
       public virtual void Add(object entity)
       {
           // Temel implementasyon
       }
       
       public abstract object GetById(int id);
   }
   ```

Kalıtım, C# programlamanın güçlü bir özelliğidir, ancak doğru kullanılması önemlidir. Uygun şekilde kullanıldığında, kod tekrarını azaltır, kodun bakımını kolaylaştırır ve daha esnek, genişletilebilir uygulamalar oluşturmanıza yardımcı olur. 