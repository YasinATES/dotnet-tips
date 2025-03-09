# Kalıtım Yerine Kompozisyon (Composition Over Inheritance)

Kalıtım Yerine Kompozisyon prensibi, nesne yönelimli programlamada karmaşık işlevselliği oluşturmak için kalıtım yerine kompozisyonu tercih etmeyi öneren bir tasarım prensibidir. Bu prensip, "is-a" (bir tür) ilişkisi yerine "has-a" (sahiplik) ilişkisini kullanmayı önerir.

## 1. Kompozisyon Nedir?

Kompozisyon, bir sınıfın diğer sınıfları bileşen olarak kullanarak oluşturulmasıdır. Bu yaklaşım, daha esnek ve yönetilebilir kod yapıları oluşturmamızı sağlar.

### Kalıtım vs Kompozisyon Örneği

```csharp
// Kalıtım yaklaşımı - Problemli
public class Animal
{
    public virtual void Move()
    {
        Console.WriteLine("Hareket ediyor...");
    }
    
    public virtual void MakeSound()
    {
        Console.WriteLine("Ses çıkarıyor...");
    }
}

public class Bird : Animal
{
    public override void Move()
    {
        Console.WriteLine("Uçuyor...");
    }
    
    public override void MakeSound()
    {
        Console.WriteLine("Ötüyor...");
    }
}

public class Penguin : Bird  // Problem: Penguenler uçamaz!
{
    public override void Move()
    {
        // Bird sınıfından gelen uçma davranışı penguenler için uygun değil
        Console.WriteLine("Yürüyor...");
    }
}

// Kompozisyon yaklaşımı - Esnek
public interface IMovementBehavior
{
    void Move();
}

public interface ISoundBehavior
{
    void MakeSound();
}

public class WalkingBehavior : IMovementBehavior
{
    public void Move()
    {
        Console.WriteLine("Yürüyor...");
    }
}

public class FlyingBehavior : IMovementBehavior
{
    public void Move()
    {
        Console.WriteLine("Uçuyor...");
    }
}

public class ChirpingBehavior : ISoundBehavior
{
    public void MakeSound()
    {
        Console.WriteLine("Ötüyor...");
    }
}

public class Animal
{
    private readonly IMovementBehavior _movementBehavior;
    private readonly ISoundBehavior _soundBehavior;
    
    public Animal(IMovementBehavior movementBehavior, ISoundBehavior soundBehavior)
    {
        _movementBehavior = movementBehavior;
        _soundBehavior = soundBehavior;
    }
    
    public void Move() => _movementBehavior.Move();
    public void MakeSound() => _soundBehavior.MakeSound();
}

// Kullanım
var bird = new Animal(new FlyingBehavior(), new ChirpingBehavior());
var penguin = new Animal(new WalkingBehavior(), new ChirpingBehavior());
```

## 2. Kompozisyonun Avantajları

1. **Daha Fazla Esneklik**
   - Davranışlar çalışma zamanında değiştirilebilir
   - Yeni davranışlar kolayca eklenebilir
   - Mevcut kodu değiştirmeden genişletilebilir

2. **Daha İyi Test Edilebilirlik**
   ```csharp
   public class AnimalTests
   {
       [Fact]
       public void Move_ShouldUseMovementBehavior()
       {
           // Düzenleme
           var mockMovementBehavior = new Mock<IMovementBehavior>();
           var animal = new Animal(
               mockMovementBehavior.Object,
               Mock.Of<ISoundBehavior>());

           // İşlem
           animal.Move();

           // Doğrulama
           mockMovementBehavior.Verify(x => x.Move(), Times.Once);
       }
   }
   ```

3. **Daha Az Bağımlılık**
   ```csharp
   // Kalıtım - Sıkı bağlı
   public class ElectricVehicle : Vehicle
   {
       public override void StartEngine()
       {
           // Base class'a sıkı bağımlılık
       }
   }

   // Kompozisyon - Gevşek bağlı
   public class Vehicle
   {
       private readonly IEngine _engine;
       
       public Vehicle(IEngine engine)
       {
           _engine = engine;
       }
       
       public void Start()
       {
           _engine.Start();
       }
   }
   ```

## 3. En İyi Pratikler

1. **Davranışları Arayüzlerle Tanımlayın**
   - Her davranış için ayrı bir arayüz oluşturun
   - Arayüzleri küçük ve odaklanmış tutun
   - Interface Segregation prensibini uygulayın

2. **Dependency Injection Kullanın**
   - Bağımlılıkları constructor üzerinden enjekte edin
   - IoC container kullanmayı düşünün
   - Bağımlılıkları açıkça belirtin

3. **Strategy Pattern'i Uygulayın**
   - Değiştirilebilir algoritmaları strateji olarak tanımlayın
   - Stratejileri çalışma zamanında değiştirilebilir yapın
   - Her strateji için ayrı bir sınıf oluşturun

4. **Kompozisyonu Doğru Yerde Kullanın**
   - Gerçek "is-a" ilişkilerinde kalıtımı kullanın
   - Davranış paylaşımı için kompozisyonu tercih edin
   - Derin kalıtım hiyerarşilerinden kaçının

## 4. Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Gereksiz Kalıtım Kullanımı**
   ```csharp
   // Kötü - Gereksiz kalıtım
   public class Logger
   {
       protected void Log(string message) { }
   }

   public class UserService : Logger  // Yanlış: Kalıtım yerine kompozisyon kullanılmalı
   {
       public void CreateUser()
       {
           Log("User created");
       }
   }

   // İyi - Kompozisyon kullanımı
   public class UserService
   {
       private readonly ILogger _logger;
       
       public UserService(ILogger logger)
       {
           _logger = logger;
       }
       
       public void CreateUser()
       {
           _logger.Log("User created");
       }
   }
   ```

2. **Aşırı Kompozisyon**
   ```csharp
   // Kötü - Çok fazla bileşen
   public class Order
   {
       private readonly IValidator _validator;
       private readonly ILogger _logger;
       private readonly IMapper _mapper;
       private readonly INotifier _notifier;
       private readonly ICalculator _calculator;
       private readonly IFormatter _formatter;
       // Çok fazla bağımlılık...
   }

   // İyi - Mantıklı gruplandırma
   public class Order
   {
       private readonly IOrderProcessor _orderProcessor;
       private readonly IOrderValidator _orderValidator;
       
       public Order(IOrderProcessor processor, IOrderValidator validator)
       {
           _orderProcessor = processor;
           _orderValidator = validator;
       }
   }
   ```

3. **Yanlış Soyutlama Seviyesi**
   ```csharp
   // Kötü - Çok genel soyutlama
   public interface IProcessor
   {
       void Process(object data);
   }

   // İyi - Doğru soyutlama seviyesi
   public interface IOrderProcessor
   {
       void ProcessOrder(Order order);
   }
   ```

Kalıtım yerine kompozisyon kullanmak, kodunuzu daha esnek, test edilebilir ve bakımı kolay hale getirir. Bu prensibi doğru uygulayarak, daha modüler ve değişime açık sistemler geliştirebilirsiniz. 