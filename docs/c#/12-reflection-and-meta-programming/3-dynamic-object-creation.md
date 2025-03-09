# Dynamic Object Creation (Dinamik Nesne Oluşturma)

.NET'te dinamik nesne oluşturma, çalışma zamanında tip bilgisine dayanarak nesnelerin oluşturulmasını sağlayan güçlü bir özelliktir. Bu bölümde, çalışma zamanında dinamik olarak nesne oluşturmanın çeşitli yöntemlerini inceleyeceğiz.

## 1. Activator.CreateInstance

`Activator.CreateInstance` metodu, .NET'te çalışma zamanında dinamik olarak nesne oluşturmanın en temel ve yaygın yoludur.

```csharp
// Temel kullanım
public void BasicActivatorExample()
{
    // Tip bilgisi ile nesne oluşturma
    Type stringType = typeof(string);
    object emptyString = Activator.CreateInstance(stringType);
    Console.WriteLine($"Boş string oluşturuldu: '{emptyString}'");
    
    // Parametreli constructor çağırma
    Type listType = typeof(List<int>);
    object list = Activator.CreateInstance(listType);
    Console.WriteLine($"Liste oluşturuldu: {list.GetType().Name}");
    
    // Jenerik tip oluşturma
    Type genericDictType = typeof(Dictionary<,>);
    Type concreteType = genericDictType.MakeGenericType(typeof(string), typeof(int));
    object dictionary = Activator.CreateInstance(concreteType);
    Console.WriteLine($"Sözlük oluşturuldu: {dictionary.GetType().Name}");
}
```

### Parametreli Constructor Kullanımı

`Activator.CreateInstance` metodu, constructor parametrelerini de kabul edebilir:

```csharp
// Parametreli constructor örneği
public void ParameterizedConstructorExample()
{
    // Person sınıfı için örnek
    Type personType = typeof(Person);
    
    // Constructor parametreleri
    object[] constructorParams = new object[] { "Ahmet", 30 };
    
    // Nesne oluşturma
    object person = Activator.CreateInstance(personType, constructorParams);
    
    // Oluşturulan nesneyi kullanma
    Console.WriteLine($"Kişi oluşturuldu: {person}");
}

// Örnek sınıf
public class Person
{
    public string Name { get; }
    public int Age { get; }
    
    public Person(string name, int age)
    {
        Name = name;
        Age = age;
    }
    
    public override string ToString()
    {
        return $"{Name}, {Age} yaşında";
    }
}
```

### Jenerik Activator.CreateInstance Kullanımı

Jenerik versiyonu tip güvenliği sağlar:

```csharp
// Jenerik Activator.CreateInstance
public void GenericActivatorExample()
{
    // Doğrudan tip ile çağırma
    var customer = Activator.CreateInstance<Customer>();
    Console.WriteLine($"Müşteri oluşturuldu: {customer.GetType().Name}");
    
    // Jenerik metot ile çağırma
    var product = CreateInstance<Product>();
    Console.WriteLine($"Ürün oluşturuldu: {product.GetType().Name}");
}

// Jenerik yardımcı metot
public T CreateInstance<T>() where T : new()
{
    return Activator.CreateInstance<T>();
}

// Örnek sınıflar
public class Customer
{
    public string Id { get; set; } = Guid.NewGuid().ToString();
}

public class Product
{
    public string Code { get; set; } = "PRD-" + new Random().Next(1000, 9999);
}
```

## 2. Expression Trees (İfade Ağaçları)

Expression Trees, çalışma zamanında kod oluşturmak ve çalıştırmak için kullanılan güçlü bir mekanizmadır. Dinamik nesne oluşturma için Activator.CreateInstance'dan daha hızlı bir alternatif sunarlar.

```csharp
using System;
using System.Linq.Expressions;
using System.Reflection;

// Expression Trees ile nesne oluşturma
public class ExpressionTreeFactory
{
    // Delegate tipi tanımı
    public delegate T ObjectActivator<T>(params object[] args);
    
    // Expression Trees kullanarak factory oluşturma
    public static ObjectActivator<T> CreateActivator<T>(ConstructorInfo ctor)
    {
        // Constructor parametreleri
        ParameterInfo[] paramsInfo = ctor.GetParameters();
        
        // Expression metoduna geçirilecek parametre
        ParameterExpression param = Expression.Parameter(typeof(object[]), "args");
        
        // Constructor'a geçirilecek parametreleri oluşturma
        Expression[] argsExpressions = new Expression[paramsInfo.Length];
        
        // Her parametre için dönüşüm ifadesi oluşturma
        for (int i = 0; i < paramsInfo.Length; i++)
        {
            Expression index = Expression.Constant(i);
            Type paramType = paramsInfo[i].ParameterType;
            
            // args[i]
            Expression paramAccessor = Expression.ArrayIndex(param, index);
            
            // (ParamType)args[i]
            Expression paramCast = Expression.Convert(paramAccessor, paramType);
            
            argsExpressions[i] = paramCast;
        }
        
        // new T(args)
        NewExpression newExpression = Expression.New(ctor, argsExpressions);
        
        // Lambda ifadesi: (object[] args) => new T((T0)args[0], (T1)args[1], ...)
        LambdaExpression lambda = Expression.Lambda(typeof(ObjectActivator<T>), newExpression, param);
        
        // Compile ederek çalıştırılabilir kodu oluşturma
        ObjectActivator<T> activator = (ObjectActivator<T>)lambda.Compile();
        
        return activator;
    }
    
    // Kullanım örneği
    public static void ExpressionTreeExample()
    {
        // Person sınıfının constructor'ını alma
        ConstructorInfo ctor = typeof(Person).GetConstructor(new[] { typeof(string), typeof(int) });
        
        // Activator oluşturma
        var activator = CreateActivator<Person>(ctor);
        
        // Nesne oluşturma
        Person person = activator("Mehmet", 35);
        
        Console.WriteLine($"Expression Trees ile oluşturulan kişi: {person}");
    }
}
```

## 3. Emit Dynamic Types (Dinamik Tipler Oluşturma)

`System.Reflection.Emit` namespace'i, çalışma zamanında tamamen yeni tipler oluşturmanıza olanak tanır.

```csharp
using System.Reflection;
using System.Reflection.Emit;

// Dinamik tip oluşturma
public class DynamicTypeGenerator
{
    public static Type CreateDynamicType(string typeName, string[] propertyNames, Type[] propertyTypes)
    {
        // Tip oluşturmak için AssemblyBuilder
        AssemblyName assemblyName = new AssemblyName("DynamicAssembly");
        AssemblyBuilder assemblyBuilder = AssemblyBuilder.DefineDynamicAssembly(
            assemblyName, AssemblyBuilderAccess.Run);
        
        // ModuleBuilder
        ModuleBuilder moduleBuilder = assemblyBuilder.DefineDynamicModule("DynamicModule");
        
        // TypeBuilder
        TypeBuilder typeBuilder = moduleBuilder.DefineType(
            typeName, TypeAttributes.Public | TypeAttributes.Class);
        
        // Her özellik için field ve property oluşturma
        for (int i = 0; i < propertyNames.Length; i++)
        {
            string propertyName = propertyNames[i];
            Type propertyType = propertyTypes[i];
            
            // Field tanımlama
            FieldBuilder fieldBuilder = typeBuilder.DefineField(
                "_" + propertyName.ToLower(), propertyType, FieldAttributes.Private);
            
            // Property tanımlama
            PropertyBuilder propertyBuilder = typeBuilder.DefineProperty(
                propertyName, PropertyAttributes.HasDefault, propertyType, null);
            
            // Getter metodu
            MethodBuilder getMethodBuilder = typeBuilder.DefineMethod(
                "get_" + propertyName,
                MethodAttributes.Public | MethodAttributes.SpecialName | MethodAttributes.HideBySig,
                propertyType,
                Type.EmptyTypes);
            
            // Getter IL kodu
            ILGenerator getIL = getMethodBuilder.GetILGenerator();
            getIL.Emit(OpCodes.Ldarg_0);
            getIL.Emit(OpCodes.Ldfld, fieldBuilder);
            getIL.Emit(OpCodes.Ret);
            
            // Setter metodu
            MethodBuilder setMethodBuilder = typeBuilder.DefineMethod(
                "set_" + propertyName,
                MethodAttributes.Public | MethodAttributes.SpecialName | MethodAttributes.HideBySig,
                null,
                new[] { propertyType });
            
            // Setter IL kodu
            ILGenerator setIL = setMethodBuilder.GetILGenerator();
            setIL.Emit(OpCodes.Ldarg_0);
            setIL.Emit(OpCodes.Ldarg_1);
            setIL.Emit(OpCodes.Stfld, fieldBuilder);
            setIL.Emit(OpCodes.Ret);
            
            // Property'ye getter ve setter atama
            propertyBuilder.SetGetMethod(getMethodBuilder);
            propertyBuilder.SetSetMethod(setMethodBuilder);
        }
        
        // Tipi oluşturma
        Type dynamicType = typeBuilder.CreateType();
        
        return dynamicType;
    }
    
    // Kullanım örneği
    public static void EmitExample()
    {
        // Dinamik tip oluşturma
        string[] propertyNames = { "Id", "Name", "CreatedDate" };
        Type[] propertyTypes = { typeof(int), typeof(string), typeof(DateTime) };
        
        Type dynamicType = CreateDynamicType("DynamicProduct", propertyNames, propertyTypes);
        
        // Dinamik tipten nesne oluşturma
        object instance = Activator.CreateInstance(dynamicType);
        
        // Özellikleri ayarlama
        dynamicType.GetProperty("Id").SetValue(instance, 101);
        dynamicType.GetProperty("Name").SetValue(instance, "Dinamik Ürün");
        dynamicType.GetProperty("CreatedDate").SetValue(instance, DateTime.Now);
        
        // Özellikleri okuma
        Console.WriteLine($"Dinamik Tip: {dynamicType.Name}");
        Console.WriteLine($"Id: {dynamicType.GetProperty("Id").GetValue(instance)}");
        Console.WriteLine($"Name: {dynamicType.GetProperty("Name").GetValue(instance)}");
        Console.WriteLine($"CreatedDate: {dynamicType.GetProperty("CreatedDate").GetValue(instance)}");
    }
}
```

## 4. Factory Patterns (Fabrika Desenleri)

Factory pattern, nesne oluşturma mantığını istemci kodundan ayırmak için kullanılan bir tasarım desenidir. Reflection ile birleştirildiğinde, çok esnek ve genişletilebilir fabrikalar oluşturabilirsiniz.

```csharp
// Basit bir ürün hiyerarşisi
public interface IProduct
{
    string GetDescription();
}

public class ConcreteProductA : IProduct
{
    public string GetDescription() => "Ürün A";
}

public class ConcreteProductB : IProduct
{
    public string GetDescription() => "Ürün B";
}

// Reflection tabanlı fabrika
public class ReflectionBasedFactory
{
    private readonly Dictionary<string, Type> _registeredTypes = new Dictionary<string, Type>();
    
    // Tip kaydetme
    public void RegisterType(string key, Type type)
    {
        if (!typeof(IProduct).IsAssignableFrom(type))
        {
            throw new ArgumentException($"{type.Name} tipi IProduct arayüzünü uygulamıyor.");
        }
        
        _registeredTypes[key] = type;
    }
    
    // Nesne oluşturma
    public IProduct CreateProduct(string key)
    {
        if (!_registeredTypes.ContainsKey(key))
        {
            throw new KeyNotFoundException($"'{key}' anahtarı ile kayıtlı bir tip bulunamadı.");
        }
        
        Type type = _registeredTypes[key];
        return (IProduct)Activator.CreateInstance(type);
    }
    
    // Kullanım örneği
    public static void FactoryExample()
    {
        var factory = new ReflectionBasedFactory();
        
        // Tipleri kaydetme
        factory.RegisterType("A", typeof(ConcreteProductA));
        factory.RegisterType("B", typeof(ConcreteProductB));
        
        // Nesneleri oluşturma
        IProduct productA = factory.CreateProduct("A");
        IProduct productB = factory.CreateProduct("B");
        
        Console.WriteLine(productA.GetDescription());
        Console.WriteLine(productB.GetDescription());
        
        // Dinamik tip bulma ve kaydetme
        Assembly assembly = Assembly.GetExecutingAssembly();
        var productTypes = assembly.GetTypes()
            .Where(t => typeof(IProduct).IsAssignableFrom(t) && !t.IsInterface && !t.IsAbstract);
        
        foreach (var type in productTypes)
        {
            factory.RegisterType(type.Name, type);
            Console.WriteLine($"Tip kaydedildi: {type.Name}");
        }
    }
}
```

## 5. Performance Comparison (Performans Karşılaştırması)

Farklı dinamik nesne oluşturma yöntemlerinin performans karşılaştırması:

```csharp
using System.Diagnostics;

public class PerformanceComparison
{
    private const int IterationCount = 1000000;
    
    public static void ComparePerformance()
    {
        Console.WriteLine("Performans Karşılaştırması Başlıyor...");
        Console.WriteLine($"Her yöntem için {IterationCount:N0} iterasyon");
        Console.WriteLine("----------------------------------------");
        
        // 1. Doğrudan constructor çağrısı (baseline)
        Stopwatch sw = Stopwatch.StartNew();
        
        for (int i = 0; i < IterationCount; i++)
        {
            var obj = new TestClass();
        }
        
        sw.Stop();
        Console.WriteLine($"Doğrudan constructor: {sw.ElapsedMilliseconds} ms");
        
        // 2. Activator.CreateInstance
        sw.Restart();
        
        for (int i = 0; i < IterationCount; i++)
        {
            var obj = Activator.CreateInstance<TestClass>();
        }
        
        sw.Stop();
        Console.WriteLine($"Activator.CreateInstance: {sw.ElapsedMilliseconds} ms");
        
        // 3. Expression Trees
        var ctor = typeof(TestClass).GetConstructor(Type.EmptyTypes);
        var activator = ExpressionTreeFactory.CreateActivator<TestClass>(ctor);
        
        sw.Restart();
        
        for (int i = 0; i < IterationCount; i++)
        {
            var obj = activator();
        }
        
        sw.Stop();
        Console.WriteLine($"Expression Trees: {sw.ElapsedMilliseconds} ms");
        
        // 4. Compiled Lambda
        Func<TestClass> factory = () => new TestClass();
        
        sw.Restart();
        
        for (int i = 0; i < IterationCount; i++)
        {
            var obj = factory();
        }
        
        sw.Stop();
        Console.WriteLine($"Compiled Lambda: {sw.ElapsedMilliseconds} ms");
    }
    
    // Test sınıfı
    public class TestClass
    {
        public int Id { get; set; }
        public string Name { get; set; }
    }
}
```

## 6. Error Handling (Hata Yönetimi)

Dinamik nesne oluşturma sırasında oluşabilecek hataları yönetmek önemlidir:

```csharp
public class ErrorHandlingExamples
{
    public static void DemonstrateErrorHandling()
    {
        Console.WriteLine("Hata Yönetimi Örnekleri");
        Console.WriteLine("------------------------");
        
        // 1. Var olmayan tip
        try
        {
            Type nonExistentType = Type.GetType("NonExistentNamespace.NonExistentType");
            object instance = Activator.CreateInstance(nonExistentType);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Var olmayan tip hatası: {ex.Message}");
        }
        
        // 2. Parametresiz constructor olmayan sınıf
        try
        {
            Type type = typeof(NoDefaultConstructor);
            object instance = Activator.CreateInstance(type);
        }
        catch (MissingMethodException ex)
        {
            Console.WriteLine($"Constructor hatası: {ex.Message}");
        }
        
        // 3. Soyut sınıf veya arayüz
        try
        {
            Type abstractType = typeof(AbstractClass);
            object instance = Activator.CreateInstance(abstractType);
        }
        catch (MemberAccessException ex)
        {
            Console.WriteLine($"Soyut sınıf hatası: {ex.Message}");
        }
        
        // 4. Güvenli nesne oluşturma
        object safeInstance = SafeCreateInstance(typeof(NoDefaultConstructor), new object[] { "Test" });
        Console.WriteLine($"Güvenli oluşturma sonucu: {safeInstance != null}");
    }
    
    // Güvenli nesne oluşturma yardımcı metodu
    public static object SafeCreateInstance(Type type, object[] args = null)
    {
        try
        {
            return Activator.CreateInstance(type, args ?? new object[0]);
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Nesne oluşturma hatası ({type.Name}): {ex.Message}");
            return null;
        }
    }
    
    // Örnek sınıflar
    public class NoDefaultConstructor
    {
        public string Name { get; }
        
        public NoDefaultConstructor(string name)
        {
            Name = name;
        }
    }
    
    public abstract class AbstractClass
    {
        public abstract void DoSomething();
    }
}
```

## 7. Security Considerations (Güvenlik Konuları)

Dinamik nesne oluşturma, güvenlik açısından dikkatli kullanılması gereken bir özelliktir:

```csharp
public class SecurityConsiderations
{
    public static void DemonstrateSecurityIssues()
    {
        Console.WriteLine("Güvenlik Konuları");
        Console.WriteLine("-----------------");
        
        // 1. Kullanıcı girdisine dayalı tip oluşturma riskleri
        string userInput = "System.IO.File"; // Potansiyel tehlikeli girdi
        
        Console.WriteLine("Kullanıcı girdisine dayalı tip oluşturma:");
        Console.WriteLine($"Girdi: {userInput}");
        
        // Güvensiz yaklaşım
        try
        {
            Type unsafeType = Type.GetType(userInput);
            if (unsafeType != null)
            {
                Console.WriteLine($"Güvensiz: Tip bulundu - {unsafeType.FullName}");
                // Burada nesne oluşturmak tehlikeli olabilir
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Hata: {ex.Message}");
        }
        
        // Güvenli yaklaşım - izin verilen tipler listesi
        List<string> allowedTypes = new List<string>
        {
            "System.String",
            "System.Collections.Generic.List`1[[System.String]]",
            "YourNamespace.SafeClass"
        };
        
        if (allowedTypes.Contains(userInput))
        {
            Type safeType = Type.GetType(userInput);
            Console.WriteLine($"Güvenli: İzin verilen tip - {safeType?.FullName}");
        }
        else
        {
            Console.WriteLine("Güvenli: Tip oluşturmaya izin verilmedi");
        }
        
        // 2. Sandbox içinde çalıştırma
        Console.WriteLine("\nSandbox içinde çalıştırma:");
        AppDomain.CurrentDomain.AssemblyResolve += (sender, args) => {
            Console.WriteLine($"Assembly yükleme girişimi: {args.Name}");
            // Burada izin verilen assembly'leri kontrol edebilirsiniz
            return null;
        };
        
        // 3. Kod erişim güvenliği (CAS) - .NET Core'da sınırlı destek var
        Console.WriteLine("\nKod erişim güvenliği (CAS):");
        Console.WriteLine("Not: .NET Core'da tam CAS desteği bulunmamaktadır.");
        Console.WriteLine("Güvenlik için izin verilen tipler listesi ve AppDomain izolasyonu kullanın.");
    }
}
```

## Özet

Dinamik nesne oluşturma, .NET'te güçlü bir özelliktir ve çeşitli senaryolarda kullanılabilir:

1. **Activator.CreateInstance**: En temel ve yaygın yöntem, basit senaryolar için idealdir.
2. **Expression Trees**: Yüksek performans gerektiren senaryolar için Activator.CreateInstance'a göre daha hızlı bir alternatiftir.
3. **Emit Dynamic Types**: Çalışma zamanında tamamen yeni tipler oluşturmak için kullanılır.
4. **Factory Patterns**: Nesne oluşturma mantığını istemci kodundan ayırmak için kullanılan tasarım desenleridir.

Dinamik nesne oluşturma kullanırken dikkat edilmesi gereken konular:
- Performans etkileri
- Hata yönetimi
- Güvenlik riskleri

Doğru yöntemi seçerken, uygulamanızın gereksinimlerini ve kısıtlamalarını göz önünde bulundurmanız önemlidir. 