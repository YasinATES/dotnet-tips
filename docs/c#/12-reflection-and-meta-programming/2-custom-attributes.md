# Custom Attributes (Özel Nitelikler)

.NET'te özel nitelikler (custom attributes), kodunuza bildirimsel meta veriler eklemenizi sağlayan güçlü bir özelliktir. Bu bölümde, özel niteliklerin tanımlanması, kullanımı ve reflection ile erişilmesi konularını inceleyeceğiz.

## 1. Attribute Definition (Nitelik Tanımı)

Özel bir nitelik oluşturmak için `System.Attribute` sınıfından türetilmiş bir sınıf tanımlamanız gerekir.

```csharp
// Temel nitelik tanımı
[AttributeUsage(AttributeTargets.All, AllowMultiple = false, Inherited = true)]
public class DescriptionAttribute : Attribute
{
    // Nitelik özellikleri
    public string Text { get; }
    public int Version { get; set; }
    
    // Yapıcı metot
    public DescriptionAttribute(string text)
    {
        Text = text;
        Version = 1; // Varsayılan değer
    }
}

// Kullanım örneği
[Description("Bu bir örnek sınıftır", Version = 2)]
public class ExampleClass
{
    [Description("Örnek bir metot")]
    public void ExampleMethod()
    {
    }
}
```

### Nitelik Parametreleri

Nitelikler, yapıcı parametreleri (zorunlu) ve adlandırılmış parametreler (isteğe bağlı) alabilir.

```csharp
// Nitelik parametreleri
[AttributeUsage(AttributeTargets.All)]
public class InfoAttribute : Attribute
{
    // Yapıcı parametreleri (zorunlu)
    public string Name { get; }
    public string Category { get; }
    
    // Adlandırılmış parametreler (isteğe bağlı)
    public int Priority { get; set; }
    public bool IsDeprecated { get; set; }
    public string[] Tags { get; set; }
    
    public InfoAttribute(string name, string category)
    {
        Name = name;
        Category = category;
        Priority = 0; // Varsayılan değer
        IsDeprecated = false; // Varsayılan değer
        Tags = Array.Empty<string>(); // Varsayılan değer
    }
}

// Kullanım örneği
[Info("UserService", "Services", 
      Priority = 1, 
      IsDeprecated = false, 
      Tags = new[] { "User", "Authentication" })]
public class UserService
{
}
```

## 2. Attribute Usage (Nitelik Kullanımı)

Niteliklerin hangi program öğelerine uygulanabileceğini `AttributeUsage` niteliği ile belirleyebilirsiniz.

```csharp
// AttributeUsage örnekleri
[AttributeUsage(AttributeTargets.Class)]
public class ClassOnlyAttribute : Attribute
{
}

[AttributeUsage(AttributeTargets.Method | AttributeTargets.Property)]
public class MethodOrPropertyAttribute : Attribute
{
}

[AttributeUsage(AttributeTargets.All, AllowMultiple = true)]
public class MultipleUsageAttribute : Attribute
{
    public string Value { get; }
    
    public MultipleUsageAttribute(string value)
    {
        Value = value;
    }
}

// Kullanım örnekleri
[ClassOnly]
[MultipleUsage("First")]
[MultipleUsage("Second")]
public class AttributeUsageExample
{
    [MethodOrProperty]
    public string Name { get; set; }
    
    [MethodOrProperty]
    public void DoSomething()
    {
    }
    
    // [ClassOnly] // Hata: ClassOnlyAttribute sadece sınıflara uygulanabilir
    public void InvalidUsage()
    {
    }
}
```

### Nitelik Hedefleri

`AttributeTargets` enum'ı, niteliklerin uygulanabileceği tüm program öğelerini tanımlar.

```csharp
// Nitelik hedefleri
public void AttributeTargetsExample()
{
    // AttributeTargets değerleri:
    // - Assembly: Assembly'lere
    // - Module: Modüllere
    // - Class: Sınıflara
    // - Struct: Struct'lara
    // - Enum: Enum'lara
    // - Constructor: Yapıcılara
    // - Method: Metotlara
    // - Property: Özelliklere
    // - Field: Alanlara
    // - Event: Olaylara
    // - Interface: Arayüzlere
    // - Parameter: Parametrelere
    // - Delegate: Delegate'lere
    // - ReturnValue: Dönüş değerlerine
    // - GenericParameter: Generic parametrelere
    // - All: Tüm hedeflere
    
    // Örnek: Parametre niteliği
    [AttributeUsage(AttributeTargets.Parameter)]
    public class NotNullAttribute : Attribute
    {
    }
    
    // Kullanım
    public void ProcessData([NotNull] string data)
    {
        if (data == null)
        {
            throw new ArgumentNullException(nameof(data));
        }
        
        // İşlem...
    }
}
```

## 3. Attribute Inheritance (Nitelik Kalıtımı)

Nitelikler, `AttributeUsage` niteliğinin `Inherited` özelliği ile kalıtım davranışını kontrol edebilir.

```csharp
// Nitelik kalıtımı
[AttributeUsage(AttributeTargets.Class, Inherited = true)]
public class InheritedAttribute : Attribute
{
}

[AttributeUsage(AttributeTargets.Class, Inherited = false)]
public class NotInheritedAttribute : Attribute
{
}

// Temel sınıf
[Inherited]
[NotInherited]
public class BaseClass
{
}

// Türetilmiş sınıf
public class DerivedClass : BaseClass
{
    // InheritedAttribute kalıtım yoluyla burada da geçerlidir
    // NotInheritedAttribute kalıtım yoluyla burada geçerli değildir
}

// Kalıtım kontrolü
public void InheritanceCheck()
{
    Type baseType = typeof(BaseClass);
    Type derivedType = typeof(DerivedClass);
    
    // BaseClass üzerindeki nitelikler
    Console.WriteLine("BaseClass nitelikleri:");
    foreach (var attribute in baseType.GetCustomAttributes(true))
    {
        Console.WriteLine($"- {attribute.GetType().Name}");
    }
    
    // DerivedClass üzerindeki nitelikler
    Console.WriteLine("\nDerivedClass nitelikleri:");
    foreach (var attribute in derivedType.GetCustomAttributes(true))
    {
        Console.WriteLine($"- {attribute.GetType().Name}");
    }
}
```

### Nitelik Kalıtımı ve Reflection

Reflection ile niteliklere erişirken kalıtım davranışını kontrol edebilirsiniz.

```csharp
// Nitelik kalıtımı ve reflection
public void AttributeInheritanceReflection()
{
    Type derivedType = typeof(DerivedClass);
    
    // Kalıtılan nitelikleri dahil et
    var allAttributes = derivedType.GetCustomAttributes(true);
    Console.WriteLine("Tüm nitelikler (kalıtılanlar dahil):");
    foreach (var attribute in allAttributes)
    {
        Console.WriteLine($"- {attribute.GetType().Name}");
    }
    
    // Sadece doğrudan uygulanan nitelikler
    var declaredAttributes = derivedType.GetCustomAttributes(false);
    Console.WriteLine("\nSadece doğrudan uygulanan nitelikler:");
    foreach (var attribute in declaredAttributes)
    {
        Console.WriteLine($"- {attribute.GetType().Name}");
    }
    
    // Belirli bir nitelik türünü kontrol etme
    bool hasInherited = derivedType.IsDefined(typeof(InheritedAttribute), true);
    bool hasNotInherited = derivedType.IsDefined(typeof(NotInheritedAttribute), true);
    
    Console.WriteLine($"\nInheritedAttribute var mı: {hasInherited}");
    Console.WriteLine($"NotInheritedAttribute var mı: {hasNotInherited}");
}
```

## 4. Conditional Attributes (Koşullu Nitelikler)

`Conditional` niteliği, belirli bir önişlemci sembolü tanımlandığında niteliğin uygulanmasını sağlar.

```csharp
// Koşullu nitelikler
[AttributeUsage(AttributeTargets.Method)]
[Conditional("DEBUG")]
public class DebugOnlyAttribute : Attribute
{
}

[AttributeUsage(AttributeTargets.Method)]
[Conditional("TRACE")]
public class TraceOnlyAttribute : Attribute
{
}

// Kullanım örneği
public class ConditionalAttributeExample
{
    [DebugOnly] // Sadece DEBUG sembolü tanımlandığında derlenir
    public void DebugMethod()
    {
        Console.WriteLine("Debug metodu");
    }
    
    [TraceOnly] // Sadece TRACE sembolü tanımlandığında derlenir
    public void TraceMethod()
    {
        Console.WriteLine("Trace metodu");
    }
    
    [DebugOnly]
    [TraceOnly] // Her iki sembol de tanımlandığında derlenir
    public void DebugAndTraceMethod()
    {
        Console.WriteLine("Debug ve Trace metodu");
    }
}
```

### Özel Koşullu Nitelikler

Kendi koşullu niteliklerinizi oluşturabilirsiniz.

```csharp
// Özel koşullu nitelikler
[AttributeUsage(AttributeTargets.Method)]
public class ConditionalLogAttribute : Attribute
{
    public string Level { get; }
    
    public ConditionalLogAttribute(string level)
    {
        Level = level;
    }
}

// Koşullu nitelik işleyici
public static class LoggingService
{
    public static bool IsDebugEnabled { get; set; } = false;
    public static bool IsInfoEnabled { get; set; } = true;
    public static bool IsErrorEnabled { get; set; } = true;
    
    public static void LogMethod(MethodBase method)
    {
        var logAttribute = method.GetCustomAttribute<ConditionalLogAttribute>();
        if (logAttribute == null)
        {
            return;
        }
        
        bool shouldLog = logAttribute.Level switch
        {
            "Debug" => IsDebugEnabled,
            "Info" => IsInfoEnabled,
            "Error" => IsErrorEnabled,
            _ => false
        };
        
        if (shouldLog)
        {
            Console.WriteLine($"[{logAttribute.Level}] {method.DeclaringType.Name}.{method.Name} çağrıldı");
        }
    }
}

// Kullanım örneği
public class ConditionalLogExample
{
    [ConditionalLog("Debug")]
    public void DebugOperation()
    {
        // İşlem öncesi log
        LoggingService.LogMethod(MethodBase.GetCurrentMethod());
        
        // İşlem...
    }
    
    [ConditionalLog("Info")]
    public void InfoOperation()
    {
        // İşlem öncesi log
        LoggingService.LogMethod(MethodBase.GetCurrentMethod());
        
        // İşlem...
    }
}
```

## 5. Assembly Attributes (Assembly Nitelikleri)

Assembly düzeyinde nitelikler, assembly meta verilerini tanımlamak için kullanılır.

```csharp
// Assembly nitelikleri (AssemblyInfo.cs dosyasında veya doğrudan kaynak dosyada)
[assembly: AssemblyTitle("MyLibrary")]
[assembly: AssemblyDescription("Örnek kütüphane")]
[assembly: AssemblyCompany("My Company")]
[assembly: AssemblyProduct("My Product")]
[assembly: AssemblyCopyright("Copyright © 2023")]
[assembly: AssemblyVersion("1.0.0.0")]
[assembly: AssemblyFileVersion("1.0.0.0")]

// .NET Core/5+ için
[assembly: AssemblyMetadata("GitHash", "abc123")]
[assembly: AssemblyMetadata("BuildDate", "2023-01-01")]

// Özel assembly nitelikleri
[AttributeUsage(AttributeTargets.Assembly)]
public class ApiVersionAttribute : Attribute
{
    public string Version { get; }
    
    public ApiVersionAttribute(string version)
    {
        Version = version;
    }
}

[assembly: ApiVersion("1.0")]
```

### Assembly Niteliklerine Erişim

Reflection ile assembly niteliklerine erişebilirsiniz.

```csharp
// Assembly niteliklerine erişim
public void AccessAssemblyAttributes()
{
    // Çalışan assembly'yi alma
    Assembly assembly = Assembly.GetExecutingAssembly();
    
    // Temel assembly bilgileri
    AssemblyName assemblyName = assembly.GetName();
    Console.WriteLine($"Assembly adı: {assemblyName.Name}");
    Console.WriteLine($"Versiyon: {assemblyName.Version}");
    
    // Assembly niteliklerine erişim
    var titleAttribute = assembly.GetCustomAttribute<AssemblyTitleAttribute>();
    var descriptionAttribute = assembly.GetCustomAttribute<AssemblyDescriptionAttribute>();
    var versionAttribute = assembly.GetCustomAttribute<AssemblyVersionAttribute>();
    
    Console.WriteLine($"Başlık: {titleAttribute?.Title}");
    Console.WriteLine($"Açıklama: {descriptionAttribute?.Description}");
    Console.WriteLine($"Versiyon: {versionAttribute?.Version}");
    
    // Özel niteliklere erişim
    var apiVersionAttribute = assembly.GetCustomAttribute<ApiVersionAttribute>();
    Console.WriteLine($"API Versiyonu: {apiVersionAttribute?.Version}");
    
    // AssemblyMetadata niteliklerine erişim (.NET Core/5+)
    var metadataAttributes = assembly.GetCustomAttributes<AssemblyMetadataAttribute>();
    foreach (var metadata in metadataAttributes)
    {
        Console.WriteLine($"Metadata: {metadata.Key} = {metadata.Value}");
    }
}
```

## 6. Reflection Performance (Reflection Performansı)

Niteliklere reflection ile erişmek performans açısından maliyetli olabilir.

```csharp
// Reflection performansı
public void AttributeReflectionPerformance()
{
    const int iterations = 1_000_000;
    Type type = typeof(ExampleClass);
    
    // 1. Her seferinde nitelikleri alma
    var sw1 = Stopwatch.StartNew();
    
    for (int i = 0; i < iterations; i++)
    {
        var attributes = type.GetCustomAttributes(typeof(DescriptionAttribute), false);
        if (attributes.Length > 0)
        {
            var attribute = (DescriptionAttribute)attributes[0];
            string text = attribute.Text;
        }
    }
    
    sw1.Stop();
    Console.WriteLine($"Her seferinde nitelikleri alma: {sw1.ElapsedMilliseconds} ms");
    
    // 2. Nitelikleri önbelleğe alma
    var sw2 = Stopwatch.StartNew();
    
    var cachedAttributes = type.GetCustomAttributes(typeof(DescriptionAttribute), false);
    var cachedAttribute = cachedAttributes.Length > 0 ? (DescriptionAttribute)cachedAttributes[0] : null;
    
    for (int i = 0; i < iterations; i++)
    {
        if (cachedAttribute != null)
        {
            string text = cachedAttribute.Text;
        }
    }
    
    sw2.Stop();
    Console.WriteLine($"Nitelikleri önbelleğe alma: {sw2.ElapsedMilliseconds} ms");
    
    // Performans farkı
    Console.WriteLine($"Performans farkı: {sw1.ElapsedMilliseconds / (double)sw2.ElapsedMilliseconds:F2}x");
}
```

### Nitelik Önbelleğe Alma Stratejileri

Niteliklere erişim performansını artırmak için çeşitli önbelleğe alma stratejileri kullanabilirsiniz.

```csharp
// Nitelik önbelleğe alma stratejileri
public static class AttributeCache
{
    // Tür -> Nitelik eşlemesi
    private static readonly ConcurrentDictionary<Type, object> _typeAttributes = new ConcurrentDictionary<Type, object>();
    
    // Üye -> Nitelik eşlemesi
    private static readonly ConcurrentDictionary<MemberInfo, Dictionary<Type, object>> _memberAttributes = 
        new ConcurrentDictionary<MemberInfo, Dictionary<Type, object>>();
    
    // Tür için nitelik alma
    public static T GetAttribute<T>(Type type) where T : Attribute
    {
        return (T)_typeAttributes.GetOrAdd(type, t => 
        {
            var attributes = t.GetCustomAttributes(typeof(T), false);
            return attributes.Length > 0 ? attributes[0] : null;
        });
    }
    
    // Üye için nitelik alma
    public static T GetAttribute<T>(MemberInfo member) where T : Attribute
    {
        var memberCache = _memberAttributes.GetOrAdd(member, m => new Dictionary<Type, object>());
        
        if (!memberCache.TryGetValue(typeof(T), out var attribute))
        {
            var attributes = member.GetCustomAttributes(typeof(T), false);
            attribute = attributes.Length > 0 ? attributes[0] : null;
            memberCache[typeof(T)] = attribute;
        }
        
        return (T)attribute;
    }
    
    // Önbelleği temizleme
    public static void ClearCache()
    {
        _typeAttributes.Clear();
        _memberAttributes.Clear();
    }
}

// Kullanım örneği
public void AttributeCacheExample()
{
    Type type = typeof(ExampleClass);
    MethodInfo method = type.GetMethod("ExampleMethod");
    
    // Önbelleğe alınmış niteliklere erişim
    var typeDescription = AttributeCache.GetAttribute<DescriptionAttribute>(type);
    var methodDescription = AttributeCache.GetAttribute<DescriptionAttribute>(method);
    
    Console.WriteLine($"Sınıf açıklaması: {typeDescription?.Text}");
    Console.WriteLine($"Metot açıklaması: {methodDescription?.Text}");
}
```

## 7. Best Practices (En İyi Uygulamalar)

Özel nitelikler oluştururken ve kullanırken izlemeniz gereken en iyi uygulamalar.

```csharp
// En iyi uygulamalar
public class AttributeBestPractices
{
    // 1. Nitelik adlarını "Attribute" son eki ile bitirin
    [AttributeUsage(AttributeTargets.All)]
    public class GoodNameAttribute : Attribute
    {
    }
    
    // 2. AttributeUsage niteliğini her zaman belirtin
    [AttributeUsage(AttributeTargets.Class | AttributeTargets.Method, AllowMultiple = false, Inherited = true)]
    public class WellDefinedAttribute : Attribute
    {
    }
    
    // 3. Nitelik parametrelerini doğrulayın
    [AttributeUsage(AttributeTargets.Property)]
    public class RangeAttribute : Attribute
    {
        public int Min { get; }
        public int Max { get; }
        
        public RangeAttribute(int min, int max)
        {
            if (min > max)
            {
                throw new ArgumentException("Min değeri Max değerinden büyük olamaz", nameof(min));
            }
            
            Min = min;
            Max = max;
        }
    }
    
    // 4. Nitelik kullanımını belgelendirin
    /// <summary>
    /// Bir özelliğin gerekli olduğunu belirtir.
    /// </summary>
    [AttributeUsage(AttributeTargets.Property)]
    public class RequiredAttribute : Attribute
    {
    }
    
    // 5. Nitelik doğrulama mantığını ayrı bir sınıfta tutun
    public static class ValidationHelper
    {
        public static bool ValidateRange(object value, RangeAttribute rangeAttr)
        {
            if (value is int intValue)
            {
                return intValue >= rangeAttr.Min && intValue <= rangeAttr.Max;
            }
            
            return false;
        }
        
        public static bool ValidateRequired(object value, RequiredAttribute requiredAttr)
        {
            return value != null && !string.IsNullOrEmpty(value.ToString());
        }
    }
}
```

### Nitelik Tabanlı Programlama Örnekleri

Nitelik tabanlı programlama, kodunuzu daha bildirimsel ve genişletilebilir hale getirebilir.

```csharp
// Nitelik tabanlı programlama örnekleri
// 1. Doğrulama nitelikleri
[AttributeUsage(AttributeTargets.Property)]
public class ValidateAttribute : Attribute
{
    public virtual bool IsValid(object value)
    {
        return true;
    }
    
    public virtual string GetErrorMessage()
    {
        return "Doğrulama hatası";
    }
}

[AttributeUsage(AttributeTargets.Property)]
public class RequiredAttribute : ValidateAttribute
{
    public override bool IsValid(object value)
    {
        return value != null && !string.IsNullOrEmpty(value.ToString());
    }
    
    public override string GetErrorMessage()
    {
        return "Bu alan gereklidir";
    }
}

[AttributeUsage(AttributeTargets.Property)]
public class EmailAttribute : ValidateAttribute
{
    public override bool IsValid(object value)
    {
        if (value is string email)
        {
            // Basit e-posta doğrulama
            return email.Contains("@") && email.Contains(".");
        }
        
        return false;
    }
    
    public override string GetErrorMessage()
    {
        return "Geçerli bir e-posta adresi giriniz";
    }
}

// Model sınıfı
public class UserModel
{
    [Required]
    public string Username { get; set; }
    
    [Required]
    [Email]
    public string Email { get; set; }
    
    public string Phone { get; set; }
}

// Doğrulama işleyici
public static class Validator
{
    public static List<string> Validate(object model)
    {
        var errors = new List<string>();
        Type type = model.GetType();
        
        foreach (var property in type.GetProperties())
        {
            object value = property.GetValue(model);
            
            foreach (var attribute in property.GetCustomAttributes<ValidateAttribute>())
            {
                if (!attribute.IsValid(value))
                {
                    errors.Add($"{property.Name}: {attribute.GetErrorMessage()}");
                }
            }
        }
        
        return errors;
    }
}

// Kullanım örneği
public void ValidationExample()
{
    var user = new UserModel
    {
        Username = "",
        Email = "invalid-email",
        Phone = "555-1234"
    };
    
    var errors = Validator.Validate(user);
    
    if (errors.Count > 0)
    {
        Console.WriteLine("Doğrulama hataları:");
        foreach (var error in errors)
        {
            Console.WriteLine($"- {error}");
        }
    }
    else
    {
        Console.WriteLine("Doğrulama başarılı");
    }
}
```

## Özet

Bu bölümde, .NET'te özel nitelikler ve bunların kullanımı hakkında bilgi edindik:

1. **Attribute Definition**: Özel niteliklerin tanımlanması ve parametrelerinin belirlenmesi.

2. **Attribute Usage**: Niteliklerin hangi program öğelerine uygulanabileceğinin kontrolü.

3. **Attribute Inheritance**: Niteliklerin kalıtım davranışı ve reflection ile erişim.

4. **Conditional Attributes**: Belirli koşullarda derlenen nitelikler.

5. **Assembly Attributes**: Assembly düzeyinde meta verilerin tanımlanması.

6. **Reflection Performance**: Niteliklere erişimde performans değerlendirmeleri ve iyileştirme teknikleri.

7. **Best Practices**: Özel nitelikler oluştururken ve kullanırken izlenmesi gereken en iyi uygulamalar.

Özel nitelikler, kodunuza bildirimsel meta veriler ekleyerek daha esnek, genişletilebilir ve bakımı kolay uygulamalar geliştirmenize yardımcı olur. 