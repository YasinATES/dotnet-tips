# Type Information (Tür Bilgisi)

.NET'te reflection, çalışma zamanında tür bilgilerine erişmeyi ve bu bilgileri kullanarak dinamik olarak kod çalıştırmayı sağlayan güçlü bir özelliktir. Bu bölümde, Type sınıfı ve tür bilgilerine erişim konularını inceleyeceğiz.

## 1. Type Class (Type Sınıfı)

`System.Type` sınıfı, .NET'te tür bilgilerine erişmenin temel yoludur.

```csharp
// Type sınıfına erişim yöntemleri
public void TypeClassExamples()
{
    // 1. typeof operatörü ile
    Type stringType = typeof(string);
    
    // 2. GetType() metodu ile
    string text = "Hello";
    Type textType = text.GetType();
    
    // 3. Type.GetType() metodu ile
    Type intType = Type.GetType("System.Int32");
    
    // 4. Assembly üzerinden
    Assembly assembly = Assembly.GetExecutingAssembly();
    Type[] types = assembly.GetTypes();
    
    // Type bilgilerini yazdırma
    Console.WriteLine($"String türü: {stringType.FullName}");
    Console.WriteLine($"Text değişkeninin türü: {textType.FullName}");
    Console.WriteLine($"Int türü: {intType.FullName}");
    Console.WriteLine($"Assembly'deki tür sayısı: {types.Length}");
}
```

### Type Sınıfının Özellikleri

`Type` sınıfının temel özellikleri:

```csharp
// Type sınıfının özellikleri
public void TypeProperties()
{
    Type type = typeof(DateTime);
    
    // İsim özellikleri
    Console.WriteLine($"Name: {type.Name}");
    Console.WriteLine($"FullName: {type.FullName}");
    Console.WriteLine($"Namespace: {type.Namespace}");
    
    // Tür kategorisi
    Console.WriteLine($"IsClass: {type.IsClass}");
    Console.WriteLine($"IsValueType: {type.IsValueType}");
    Console.WriteLine($"IsEnum: {type.IsEnum}");
    Console.WriteLine($"IsInterface: {type.IsInterface}");
    Console.WriteLine($"IsAbstract: {type.IsAbstract}");
    Console.WriteLine($"IsSealed: {type.IsSealed}");
    Console.WriteLine($"IsGenericType: {type.IsGenericType}");
    
    // Assembly bilgisi
    Console.WriteLine($"Assembly: {type.Assembly.FullName}");
    
    // Temel tür
    Console.WriteLine($"BaseType: {type.BaseType?.FullName}");
    
    // Generic tür bilgileri
    Type listType = typeof(List<int>);
    Console.WriteLine($"IsGenericType: {listType.IsGenericType}");
    Console.WriteLine($"GenericTypeDefinition: {listType.GetGenericTypeDefinition().FullName}");
    Console.WriteLine($"GenericArguments: {string.Join(", ", listType.GetGenericArguments().Select(t => t.Name))}");
}
```

## 2. Type Inspection (Tür İncelemesi)

Reflection ile türlerin içeriğini inceleyebilirsiniz.

```csharp
// Tür incelemesi
public void TypeInspection()
{
    Type personType = typeof(Person);
    
    // Üyeleri listeleme
    Console.WriteLine("--- Alanlar ---");
    foreach (var field in personType.GetFields(BindingFlags.Public | BindingFlags.NonPublic | BindingFlags.Instance))
    {
        Console.WriteLine($"{field.FieldType.Name} {field.Name}");
    }
    
    Console.WriteLine("--- Özellikler ---");
    foreach (var property in personType.GetProperties())
    {
        Console.WriteLine($"{property.PropertyType.Name} {property.Name}");
    }
    
    Console.WriteLine("--- Metotlar ---");
    foreach (var method in personType.GetMethods(BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly))
    {
        Console.WriteLine($"{method.ReturnType.Name} {method.Name}({string.Join(", ", method.GetParameters().Select(p => $"{p.ParameterType.Name} {p.Name}"))})");
    }
    
    Console.WriteLine("--- Yapıcılar ---");
    foreach (var constructor in personType.GetConstructors())
    {
        Console.WriteLine($"{constructor.Name}({string.Join(", ", constructor.GetParameters().Select(p => $"{p.ParameterType.Name} {p.Name}"))})");
    }
}

// Örnek sınıf
public class Person
{
    private string _name;
    public int Age { get; set; }
    
    public Person()
    {
    }
    
    public Person(string name, int age)
    {
        _name = name;
        Age = age;
    }
    
    public string GetName()
    {
        return _name;
    }
    
    public void SetName(string name)
    {
        _name = name;
    }
    
    public string GetInfo()
    {
        return $"{_name}, {Age} yaşında";
    }
}
```

### Tür Hiyerarşisi İncelemesi

Bir türün hiyerarşisini ve uyguladığı arayüzleri inceleyebilirsiniz.

```csharp
// Tür hiyerarşisi incelemesi
public void TypeHierarchyInspection()
{
    Type type = typeof(List<string>);
    
    // Temel tür hiyerarşisi
    Console.WriteLine("Temel tür hiyerarşisi:");
    Type currentType = type;
    while (currentType != null)
    {
        Console.WriteLine($"- {currentType.FullName}");
        currentType = currentType.BaseType;
    }
    
    // Uygulanan arayüzler
    Console.WriteLine("\nUygulanan arayüzler:");
    foreach (var interfaceType in type.GetInterfaces())
    {
        Console.WriteLine($"- {interfaceType.FullName}");
    }
    
    // Generic tür parametreleri
    if (type.IsGenericType)
    {
        Console.WriteLine("\nGeneric tür parametreleri:");
        foreach (var genericArg in type.GetGenericArguments())
        {
            Console.WriteLine($"- {genericArg.FullName}");
        }
    }
}
```

## 3. Member Information (Üye Bilgileri)

Reflection ile bir türün üyelerine (alanlar, özellikler, metotlar vb.) erişebilirsiniz.

```csharp
// Üye bilgileri
public void MemberInformation()
{
    Type type = typeof(Person);
    
    // Alan bilgileri
    FieldInfo nameField = type.GetField("_name", BindingFlags.NonPublic | BindingFlags.Instance);
    Console.WriteLine($"Alan: {nameField.Name}, Tür: {nameField.FieldType.Name}, Erişim: {GetAccessibility(nameField)}");
    
    // Özellik bilgileri
    PropertyInfo ageProperty = type.GetProperty("Age");
    Console.WriteLine($"Özellik: {ageProperty.Name}, Tür: {ageProperty.PropertyType.Name}, Okuma: {ageProperty.CanRead}, Yazma: {ageProperty.CanWrite}");
    
    // Metot bilgileri
    MethodInfo getInfoMethod = type.GetMethod("GetInfo");
    Console.WriteLine($"Metot: {getInfoMethod.Name}, Dönüş Türü: {getInfoMethod.ReturnType.Name}, Parametre Sayısı: {getInfoMethod.GetParameters().Length}");
    
    // Yapıcı bilgileri
    ConstructorInfo constructor = type.GetConstructor(new[] { typeof(string), typeof(int) });
    Console.WriteLine($"Yapıcı: {constructor.Name}, Parametre Sayısı: {constructor.GetParameters().Length}");
    
    // Parametre bilgileri
    Console.WriteLine("\nYapıcı parametreleri:");
    foreach (var parameter in constructor.GetParameters())
    {
        Console.WriteLine($"- {parameter.Name}: {parameter.ParameterType.Name}, Pozisyon: {parameter.Position}");
    }
}

// Erişim belirleyicisini alma
private string GetAccessibility(FieldInfo field)
{
    if (field.IsPublic) return "public";
    if (field.IsPrivate) return "private";
    if (field.IsFamily) return "protected";
    if (field.IsAssembly) return "internal";
    if (field.IsFamilyOrAssembly) return "protected internal";
    return "unknown";
}
```

### Dinamik Üye Erişimi

Reflection ile çalışma zamanında dinamik olarak üyelere erişebilirsiniz.

```csharp
// Dinamik üye erişimi
public void DynamicMemberAccess()
{
    // Örnek nesne oluşturma
    Person person = new Person("Ahmet", 30);
    Type personType = person.GetType();
    
    // Alan değerine erişim
    FieldInfo nameField = personType.GetField("_name", BindingFlags.NonPublic | BindingFlags.Instance);
    string name = (string)nameField.GetValue(person);
    Console.WriteLine($"Alan değeri: {name}");
    
    // Alan değerini değiştirme
    nameField.SetValue(person, "Mehmet");
    Console.WriteLine($"Değiştirilen alan değeri: {person.GetName()}");
    
    // Özellik değerine erişim
    PropertyInfo ageProperty = personType.GetProperty("Age");
    int age = (int)ageProperty.GetValue(person);
    Console.WriteLine($"Özellik değeri: {age}");
    
    // Özellik değerini değiştirme
    ageProperty.SetValue(person, 35);
    Console.WriteLine($"Değiştirilen özellik değeri: {person.Age}");
    
    // Metot çağırma
    MethodInfo getInfoMethod = personType.GetMethod("GetInfo");
    string info = (string)getInfoMethod.Invoke(person, null);
    Console.WriteLine($"Metot sonucu: {info}");
    
    // Parametreli metot çağırma
    MethodInfo setNameMethod = personType.GetMethod("SetName");
    setNameMethod.Invoke(person, new object[] { "Ali" });
    Console.WriteLine($"Metot çağrısı sonrası: {person.GetName()}");
}
```

## 4. Custom Attributes (Özel Nitelikler)

Reflection ile özel niteliklere erişebilirsiniz.

```csharp
// Özel nitelikler
[AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = true)]
public class InfoAttribute : Attribute
{
    public string Description { get; }
    public int Version { get; }
    
    public InfoAttribute(string description, int version = 1)
    {
        Description = description;
        Version = version;
    }
}

[Info("Kişi bilgilerini temsil eden sınıf", Version = 2)]
public class Employee
{
    [Info("Çalışanın adı")]
    public string Name { get; set; }
    
    [Info("Çalışanın departmanı")]
    public string Department { get; set; }
    
    [Info("Çalışan bilgilerini döndürür", Version = 3)]
    public string GetEmployeeInfo()
    {
        return $"{Name}, {Department} departmanında çalışıyor";
    }
}

// Özel niteliklere erişim
public void CustomAttributeAccess()
{
    Type employeeType = typeof(Employee);
    
    // Sınıf üzerindeki nitelikler
    Console.WriteLine("Sınıf nitelikleri:");
    foreach (var attribute in employeeType.GetCustomAttributes(true))
    {
        if (attribute is InfoAttribute infoAttr)
        {
            Console.WriteLine($"- Açıklama: {infoAttr.Description}, Versiyon: {infoAttr.Version}");
        }
    }
    
    // Özellikler üzerindeki nitelikler
    Console.WriteLine("\nÖzellik nitelikleri:");
    foreach (var property in employeeType.GetProperties())
    {
        var attributes = property.GetCustomAttributes<InfoAttribute>(true);
        foreach (var attribute in attributes)
        {
            Console.WriteLine($"- {property.Name}: {attribute.Description}, Versiyon: {attribute.Version}");
        }
    }
    
    // Metotlar üzerindeki nitelikler
    Console.WriteLine("\nMetot nitelikleri:");
    foreach (var method in employeeType.GetMethods(BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly))
    {
        var attributes = method.GetCustomAttributes<InfoAttribute>(true);
        foreach (var attribute in attributes)
        {
            Console.WriteLine($"- {method.Name}: {attribute.Description}, Versiyon: {attribute.Version}");
        }
    }
    
    // Belirli bir niteliğe sahip üyeleri bulma
    Console.WriteLine("\nVersion 3 olan nitelikler:");
    var members = employeeType.GetMembers()
        .Where(m => m.GetCustomAttributes<InfoAttribute>().Any(a => a.Version == 3));
    
    foreach (var member in members)
    {
        Console.WriteLine($"- {member.Name}");
    }
}
```

## 5. Assembly Loading (Assembly Yükleme)

Reflection ile assembly'leri dinamik olarak yükleyebilirsiniz.

```csharp
// Assembly yükleme
public void AssemblyLoading()
{
    // Çalışan assembly'yi alma
    Assembly currentAssembly = Assembly.GetExecutingAssembly();
    Console.WriteLine($"Çalışan assembly: {currentAssembly.FullName}");
    
    // Assembly'deki türleri listeleme
    Console.WriteLine("\nAssembly'deki türler:");
    foreach (var type in currentAssembly.GetTypes().Take(5)) // İlk 5 tür
    {
        Console.WriteLine($"- {type.FullName}");
    }
    
    // Dosyadan assembly yükleme
    try
    {
        Assembly loadedAssembly = Assembly.LoadFrom("System.Text.Json.dll");
        Console.WriteLine($"\nYüklenen assembly: {loadedAssembly.FullName}");
        
        // Assembly'deki türleri listeleme
        Console.WriteLine("\nYüklenen assembly'deki türler:");
        foreach (var type in loadedAssembly.GetTypes().Take(5)) // İlk 5 tür
        {
            Console.WriteLine($"- {type.FullName}");
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine($"Assembly yükleme hatası: {ex.Message}");
    }
    
    // GAC'den assembly yükleme
    try
    {
        Assembly systemAssembly = Assembly.Load("System, Version=4.0.0.0, Culture=neutral, PublicKeyToken=b77a5c561934e089");
        Console.WriteLine($"\nGAC'den yüklenen assembly: {systemAssembly.FullName}");
    }
    catch (Exception ex)
    {
        Console.WriteLine($"GAC'den assembly yükleme hatası: {ex.Message}");
    }
}
```

### Dinamik Tür Oluşturma

Reflection.Emit ile çalışma zamanında dinamik olarak türler oluşturabilirsiniz.

```csharp
// Dinamik tür oluşturma
public void DynamicTypeCreation()
{
    // Dinamik assembly oluşturma
    AssemblyName assemblyName = new AssemblyName("DynamicAssembly");
    AssemblyBuilder assemblyBuilder = AssemblyBuilder.DefineDynamicAssembly(
        assemblyName, AssemblyBuilderAccess.Run);
    
    // Dinamik modül oluşturma
    ModuleBuilder moduleBuilder = assemblyBuilder.DefineDynamicModule("DynamicModule");
    
    // Dinamik tür oluşturma
    TypeBuilder typeBuilder = moduleBuilder.DefineType(
        "DynamicType", TypeAttributes.Public | TypeAttributes.Class);
    
    // Alan ekleme
    FieldBuilder fieldBuilder = typeBuilder.DefineField(
        "_value", typeof(int), FieldAttributes.Private);
    
    // Özellik ekleme
    PropertyBuilder propertyBuilder = typeBuilder.DefineProperty(
        "Value", PropertyAttributes.None, typeof(int), null);
    
    // Getter metodu
    MethodBuilder getterBuilder = typeBuilder.DefineMethod(
        "get_Value", MethodAttributes.Public | MethodAttributes.SpecialName | MethodAttributes.HideBySig,
        typeof(int), Type.EmptyTypes);
    
    ILGenerator getterIL = getterBuilder.GetILGenerator();
    getterIL.Emit(OpCodes.Ldarg_0);
    getterIL.Emit(OpCodes.Ldfld, fieldBuilder);
    getterIL.Emit(OpCodes.Ret);
    
    // Setter metodu
    MethodBuilder setterBuilder = typeBuilder.DefineMethod(
        "set_Value", MethodAttributes.Public | MethodAttributes.SpecialName | MethodAttributes.HideBySig,
        null, new[] { typeof(int) });
    
    ILGenerator setterIL = setterBuilder.GetILGenerator();
    setterIL.Emit(OpCodes.Ldarg_0);
    setterIL.Emit(OpCodes.Ldarg_1);
    setterIL.Emit(OpCodes.Stfld, fieldBuilder);
    setterIL.Emit(OpCodes.Ret);
    
    // Getter ve setter'ı özelliğe bağlama
    propertyBuilder.SetGetMethod(getterBuilder);
    propertyBuilder.SetSetMethod(setterBuilder);
    
    // Türü oluşturma
    Type dynamicType = typeBuilder.CreateType();
    
    // Dinamik türü kullanma
    object instance = Activator.CreateInstance(dynamicType);
    PropertyInfo property = dynamicType.GetProperty("Value");
    
    property.SetValue(instance, 42);
    int value = (int)property.GetValue(instance);
    
    Console.WriteLine($"Dinamik tür: {dynamicType.FullName}");
    Console.WriteLine($"Özellik değeri: {value}");
}
```

## 6. Version Compatibility (Versiyon Uyumluluğu)

Reflection ile farklı versiyonlardaki assembly'leri ve türleri yönetebilirsiniz.

```csharp
// Versiyon uyumluluğu
public void VersionCompatibility()
{
    // Assembly yükleme politikası
    AppDomain currentDomain = AppDomain.CurrentDomain;
    
    // Assembly çözümleme olayını dinleme
    currentDomain.AssemblyResolve += (sender, args) =>
    {
        Console.WriteLine($"Assembly çözümleme isteği: {args.Name}");
        
        // Özel assembly çözümleme mantığı
        // Burada belirli bir assembly için özel bir versiyon dönebilirsiniz
        if (args.Name.StartsWith("MyLibrary"))
        {
            // Özel bir konumdan assembly yükleme
            string assemblyPath = Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "Libraries", "MyLibrary.dll");
            if (File.Exists(assemblyPath))
            {
                return Assembly.LoadFrom(assemblyPath);
            }
        }
        
        return null;
    };
    
    // Tür yükleme güvenliği
    try
    {
        // Güvenli tür yükleme
        Type safeType = Type.GetType("System.String, mscorlib", false, true);
        Console.WriteLine($"Güvenli tür yükleme: {safeType?.FullName ?? "Bulunamadı"}");
        
        // Güvensiz tür yükleme (istisna fırlatabilir)
        Type unsafeType = Type.GetType("NonExistentType", true);
    }
    catch (TypeLoadException ex)
    {
        Console.WriteLine($"Tür yükleme hatası: {ex.Message}");
    }
    
    // Assembly versiyon bilgisi
    Assembly currentAssembly = Assembly.GetExecutingAssembly();
    AssemblyName assemblyName = currentAssembly.GetName();
    
    Console.WriteLine($"\nAssembly adı: {assemblyName.Name}");
    Console.WriteLine($"Versiyon: {assemblyName.Version}");
    Console.WriteLine($"Kültür: {assemblyName.CultureName ?? "nötr"}");
    Console.WriteLine($"Public Key Token: {BitConverter.ToString(assemblyName.GetPublicKeyToken() ?? new byte[0]).Replace("-", "")}");
}
```

## 7. Performance Considerations (Performans Değerlendirmeleri)

Reflection kullanırken performans etkilerini göz önünde bulundurmalısınız.

```csharp
// Performans değerlendirmeleri
public void PerformanceConsiderations()
{
    const int iterations = 1_000_000;
    
    // 1. Doğrudan erişim
    var sw1 = Stopwatch.StartNew();
    
    Person person = new Person("Test", 30);
    for (int i = 0; i < iterations; i++)
    {
        string name = person.GetName();
    }
    
    sw1.Stop();
    Console.WriteLine($"Doğrudan erişim: {sw1.ElapsedMilliseconds} ms");
    
    // 2. Reflection ile erişim
    var sw2 = Stopwatch.StartNew();
    
    Type personType = typeof(Person);
    MethodInfo getNameMethod = personType.GetMethod("GetName");
    
    for (int i = 0; i < iterations; i++)
    {
        string name = (string)getNameMethod.Invoke(person, null);
    }
    
    sw2.Stop();
    Console.WriteLine($"Reflection ile erişim: {sw2.ElapsedMilliseconds} ms");
    
    // 3. Delegate ile erişim (daha hızlı reflection)
    var sw3 = Stopwatch.StartNew();
    
    var getNameDelegate = (Func<Person, string>)Delegate.CreateDelegate(
        typeof(Func<Person, string>), getNameMethod);
    
    for (int i = 0; i < iterations; i++)
    {
        string name = getNameDelegate(person);
    }
    
    sw3.Stop();
    Console.WriteLine($"Delegate ile erişim: {sw3.ElapsedMilliseconds} ms");
    
    // Performans karşılaştırması
    Console.WriteLine($"Reflection/Doğrudan oranı: {sw2.ElapsedMilliseconds / (double)sw1.ElapsedMilliseconds:F2}x");
    Console.WriteLine($"Delegate/Doğrudan oranı: {sw3.ElapsedMilliseconds / (double)sw1.ElapsedMilliseconds:F2}x");
}
```

### Performans İyileştirme Teknikleri

Reflection kullanırken performansı iyileştirmek için çeşitli teknikler kullanabilirsiniz.

```csharp
// Performans iyileştirme teknikleri
public void PerformanceOptimizationTechniques()
{
    // 1. Reflection sonuçlarını önbelleğe alma
    Type type = typeof(Person);
    
    // Metot bilgilerini önbelleğe alma
    Dictionary<string, MethodInfo> methodCache = type.GetMethods()
        .ToDictionary(m => m.Name);
    
    // Önbellekten metot alma
    MethodInfo cachedMethod = methodCache["GetName"];
    
    // 2. Delegate kullanma
    MethodInfo getNameMethod = type.GetMethod("GetName");
    var getNameDelegate = (Func<Person, string>)Delegate.CreateDelegate(
        typeof(Func<Person, string>), getNameMethod);
    
    Person person = new Person("Test", 30);
    string name = getNameDelegate(person); // Hızlı çağrı
    
    // 3. Expression trees kullanma
    PropertyInfo ageProperty = type.GetProperty("Age");
    
    // Getter için expression tree
    ParameterExpression paramExpr = Expression.Parameter(typeof(Person), "p");
    MemberExpression propExpr = Expression.Property(paramExpr, ageProperty);
    var getterLambda = Expression.Lambda<Func<Person, int>>(propExpr, paramExpr);
    var getterFunc = getterLambda.Compile();
    
    // Setter için expression tree
    ParameterExpression valueExpr = Expression.Parameter(typeof(int), "v");
    var setterLambda = Expression.Lambda<Action<Person, int>>(
        Expression.Call(paramExpr, ageProperty.GetSetMethod(), valueExpr),
        paramExpr, valueExpr);
    var setterAction = setterLambda.Compile();
    
    // Kullanım
    int age = getterFunc(person);
    setterAction(person, 35);
    
    // 4. DynamicMethod kullanma
    DynamicMethod dynamicGetter = new DynamicMethod(
        "GetName", typeof(string), new[] { typeof(Person) },
        typeof(Person).Module);
    
    ILGenerator il = dynamicGetter.GetILGenerator();
    il.Emit(OpCodes.Ldarg_0);
    il.Emit(OpCodes.Call, getNameMethod);
    il.Emit(OpCodes.Ret);
    
    var fastGetter = (Func<Person, string>)dynamicGetter.CreateDelegate(typeof(Func<Person, string>));
    string dynamicName = fastGetter(person);
}
```

## Özet

Bu bölümde, .NET'te reflection ve tür bilgilerine erişim konularını inceledik:

1. **Type Class**: Tür bilgilerine erişmenin temel yolu.

2. **Type Inspection**: Türlerin içeriğini ve hiyerarşisini inceleme.

3. **Member Information**: Türlerin üyelerine erişim ve dinamik çağrılar.

4. **Custom Attributes**: Özel niteliklere erişim ve kullanım.

5. **Assembly Loading**: Assembly'leri dinamik olarak yükleme ve tür oluşturma.

6. **Version Compatibility**: Farklı versiyonlardaki assembly'leri ve türleri yönetme.

7. **Performance Considerations**: Reflection kullanırken performans etkilerini değerlendirme ve iyileştirme teknikleri.

Reflection, dinamik ve genişletilebilir uygulamalar geliştirmek için güçlü bir araçtır, ancak performans etkilerini göz önünde bulundurarak dikkatli kullanılmalıdır. 