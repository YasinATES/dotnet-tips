# Nullable Reference Types (Null Atanabilir Referans Türleri)

C# 8.0 ile tanıtılan Nullable Reference Types (NRT) özelliği, referans türlerinin null değer alabilirliğini daha açık bir şekilde belirtmenize olanak tanır. Bu özellik, en yaygın çalışma zamanı istisnalarından biri olan `NullReferenceException`'ları daha kolay tespit etmenize ve önlemenize yardımcı olur.

## 1. Null Safety (Null Güvenliği)

Null güvenliği, programınızın null referanslarla ilgili hataları derleme zamanında tespit etme yeteneğidir.

```csharp
// Null güvenliği olmadan (C# 8.0 öncesi)
public void NullSafetyBeforeExample()
{
    string name = null;
    int length = name.Length; // Çalışma zamanında NullReferenceException
}

// Null güvenliği ile (C# 8.0 ve sonrası)
public void NullSafetyAfterExample()
{
    string name = null; // Uyarı: Converting null literal or possible null value to non-nullable type.
    // int length = name.Length; // Uyarı: Derleme zamanında tespit edilir
}
```

### Null Referans Hatalarının Maliyeti

Null referans hataları, yazılım geliştirmede en yaygın hatalardan biridir ve önemli maliyetlere neden olabilir.

```csharp
// Gerçek dünya senaryosu
public class UserService
{
    private readonly IUserRepository _repository;
    
    public UserService(IUserRepository repository)
    {
        _repository = repository; // repository null olabilir
    }
    
    public User GetUserById(int id)
    {
        // _repository null ise, burada NullReferenceException oluşur
        return _repository.GetById(id);
    }
}

// Null güvenliği ile düzeltilmiş versiyon
public class SafeUserService
{
    private readonly IUserRepository _repository;
    
    public SafeUserService(IUserRepository repository)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
    }
    
    public User? GetUserById(int id)
    {
        return _repository.GetById(id);
    }
}
```

## 2. Nullable Contexts (Null Atanabilir Bağlamlar)

C# 8.0, iki farklı nullable bağlam modu sunar: nullable aware (farkında) ve nullable oblivious (farkında olmayan).

```csharp
// Nullable bağlamları etkinleştirme yöntemleri

// 1. Proje dosyasında (.csproj):
// <PropertyGroup>
//   <Nullable>enable</Nullable>
// </PropertyGroup>

// 2. Dosya başında pragma direktifi:
// #nullable enable

// 3. Kod bloğu için pragma direktifi:
// #nullable enable
// ... kod ...
// #nullable disable
```

### Nullable Aware Bağlam

Nullable aware bağlamda, tüm referans türleri varsayılan olarak null atanamaz kabul edilir.

```csharp
#nullable enable

// Nullable aware bağlam
public class NullableAwareExample
{
    public string NonNullableString { get; set; } // null olamaz
    public string? NullableString { get; set; } // null olabilir
    
    public void ProcessString(string text) // text null olamaz
    {
        Console.WriteLine(text.Length); // Güvenli erişim
    }
    
    public void ProcessNullableString(string? text) // text null olabilir
    {
        if (text != null)
        {
            Console.WriteLine(text.Length); // Null kontrolü sonrası güvenli erişim
        }
    }
}
```

### Nullable Oblivious Bağlam

Nullable oblivious bağlamda, referans türleri C# 8.0 öncesindeki gibi davranır.

```csharp
#nullable disable

// Nullable oblivious bağlam
public class NullableObliviousExample
{
    public string MightBeNullString { get; set; } // Eski davranış, null olabilir
    
    public void ProcessString(string text) // Eski davranış, text null olabilir
    {
        // Derleme zamanında uyarı yok, ancak çalışma zamanında hata olabilir
        Console.WriteLine(text.Length);
    }
}
```

## 3. Null-forgiving Operator (Null-Görmezden Gelme Operatörü)

Null-forgiving operatörü (`!`), derleyiciye bir ifadenin null olmadığını bildirmek için kullanılır.

```csharp
#nullable enable

// Null-forgiving operatörü
public class NullForgettingExample
{
    public void ProcessData(string? text)
    {
        // text null olabilir, ancak burada null olmadığından eminiz
        string nonNullText = text!;
        
        // Artık nonNullText üzerinde güvenle işlem yapabiliriz
        Console.WriteLine(nonNullText.Length);
    }
    
    public void InitializeData()
    {
        string? name = GetNameFromSomewhere();
        
        // name'in null olmadığını biliyoruz, ancak derleyici bunu anlayamıyor
        ProcessNonNullableName(name!);
    }
    
    private string? GetNameFromSomewhere()
    {
        // Bir yerden isim al
        return "John";
    }
    
    private void ProcessNonNullableName(string name)
    {
        Console.WriteLine(name.Length);
    }
}
```

### Null-forgiving Operatörünün Doğru Kullanımı

Null-forgiving operatörünü dikkatli kullanmak önemlidir, çünkü yanlış kullanım çalışma zamanı hatalarına neden olabilir.

```csharp
#nullable enable

// Null-forgiving operatörünün doğru ve yanlış kullanımı
public class NullForgettingUsage
{
    // DOĞRU: Başlatıcı metot içinde değer atandığını biliyoruz
    private string _name = null!;
    
    public NullForgettingUsage()
    {
        Initialize();
    }
    
    private void Initialize()
    {
        _name = "John";
    }
    
    // YANLIŞ: Gerçekten null olabilecek bir değeri zorla null olmayan olarak işaretlemek
    public void DangerousMethod(string? input)
    {
        // input gerçekten null ise, bu satır çalışma zamanında hata verecektir
        int length = input!.Length;
    }
    
    // DOĞRU: Null kontrolü yaparak güvenli erişim
    public void SafeMethod(string? input)
    {
        if (input != null)
        {
            int length = input.Length;
        }
    }
}
```

## 4. Null Analysis (Null Analizi)

C# derleyicisi, kodunuzdaki null değerlerin akışını analiz eder ve potansiyel null referans hatalarını tespit eder.

```csharp
#nullable enable

// Null akış analizi
public class NullFlowAnalysis
{
    public void ProcessData(string? text)
    {
        // Derleyici, text'in null olabileceğini bilir
        // text.Length; // Derleme hatası
        
        // Null kontrolü sonrası, derleyici text'in artık null olmadığını anlar
        if (text != null)
        {
            Console.WriteLine(text.Length); // Güvenli erişim
        }
        
        // Alternatif null kontrolü
        if (text is null)
        {
            return;
        }
        
        Console.WriteLine(text.Length); // Güvenli erişim
    }
    
    public void ConditionalNullCheck(string? a, string? b)
    {
        // Mantıksal operatörlerle null kontrolü
        if (a != null && b != null)
        {
            // Burada a ve b'nin null olmadığını biliyoruz
            Console.WriteLine(a.Length + b.Length);
        }
        
        // Erken dönüş ile null kontrolü
        if (a == null || b == null)
        {
            return;
        }
        
        // Burada a ve b'nin null olmadığını biliyoruz
        Console.WriteLine(a.Length + b.Length);
    }
}
```

### Gelişmiş Null Analizi

Derleyici, daha karmaşık null kontrol desenlerini de anlayabilir.

```csharp
#nullable enable

// Gelişmiş null analizi
public class AdvancedNullAnalysis
{
    public void PatternMatching(object? obj)
    {
        // Pattern matching ile null kontrolü
        if (obj is string text)
        {
            // text null olamaz, çünkü obj string ise ve null değilse buraya girer
            Console.WriteLine(text.Length);
        }
    }
    
    public void NullCoalescingOperator(string? text)
    {
        // Null birleştirme operatörü ile varsayılan değer atama
        string nonNullText = text ?? "Default";
        
        // nonNullText artık null olamaz
        Console.WriteLine(nonNullText.Length);
    }
    
    public void NullConditionalOperator(string? text)
    {
        // Null koşullu operatör ile güvenli erişim
        int? length = text?.Length;
        
        // length null olabilir
        Console.WriteLine(length ?? 0);
    }
    
    public void ThrowExpression(string? text)
    {
        // throw ifadesi ile null kontrolü
        string nonNullText = text ?? throw new ArgumentNullException(nameof(text));
        
        // nonNullText artık null olamaz
        Console.WriteLine(nonNullText.Length);
    }
}
```

## 5. Migration Strategies (Geçiş Stratejileri)

Mevcut bir projeyi nullable reference types özelliğine geçirmek için çeşitli stratejiler vardır.

```csharp
// Geçiş stratejileri

// 1. Aşamalı geçiş: Proje dosyasında
// <PropertyGroup>
//   <Nullable>warnings</Nullable>
// </PropertyGroup>

// 2. Dosya bazında geçiş
// #nullable enable

// 3. Yeni kod için nullable, eski kod için nullable oblivious
// <PropertyGroup>
//   <Nullable>annotations</Nullable>
// </PropertyGroup>
```

### Aşamalı Geçiş Örneği

Büyük bir projeyi aşamalı olarak nullable reference types özelliğine geçirme.

```csharp
// Aşama 1: Uyarıları etkinleştir, ancak hata verme
// <Nullable>warnings</Nullable>

// Aşama 2: Yeni dosyalarda tam nullable desteği etkinleştir
#nullable enable

// Aşama 3: Mevcut dosyaları birer birer güncelle
public class LegacyClass
{
    // Eski kod, nullable bağlamı farkında değil
    public string MightBeNull { get; set; }
}

#nullable enable
public class ModernClass
{
    // Yeni kod, nullable bağlamı farkında
    public string NonNullableString { get; set; } = "";
    public string? NullableString { get; set; }
}
```

### Geçiş Sırasında Karşılaşılan Zorluklar

Nullable reference types özelliğine geçiş sırasında karşılaşılabilecek zorluklar ve çözümleri.

```csharp
#nullable enable

// Zorluk 1: Başlatılmamış alanlar
public class InitializationChallenge
{
    // Hata: Non-nullable property must contain a non-null value when exiting constructor
    public string Name { get; set; }
    
    // Çözüm 1: Varsayılan değer atama
    public string Name1 { get; set; } = "";
    
    // Çözüm 2: Yapıcıda değer atama
    public string Name2 { get; set; }
    
    public InitializationChallenge()
    {
        Name2 = "Default";
    }
    
    // Çözüm 3: Nullable yapma
    public string? Name3 { get; set; }
}

// Zorluk 2: Üçüncü taraf kütüphaneler
public class ThirdPartyChallenge
{
    // Üçüncü taraf kütüphane null döndürebilir, ancak nullable olarak işaretlenmemiş
    public void UseThirdPartyLibrary()
    {
        // ThirdPartyLib.GetValue() null döndürebilir
        string value = ThirdPartyLib.GetValue()!; // Null-forgiving operatörü kullanmak zorunda kalabiliriz
    }
}
```

## 6. Best Practices (En İyi Uygulamalar)

Nullable reference types özelliğini en etkili şekilde kullanmak için en iyi uygulamalar.

```csharp
#nullable enable

// En iyi uygulamalar
public class NullableBestPractices
{
    // 1. Varsayılan olarak non-nullable kullanın
    public string Name { get; set; } = "";
    
    // 2. Null olabilecek değerler için nullable kullanın
    public string? MiddleName { get; set; }
    
    // 3. Parametrelerde null kontrolü yapın
    public void ProcessUser(User user, string? optionalData = null)
    {
        // Zorunlu parametreler için null kontrolü
        if (user == null)
        {
            throw new ArgumentNullException(nameof(user));
        }
        
        // İsteğe bağlı parametreler için null kontrolü
        if (optionalData != null)
        {
            Console.WriteLine(optionalData);
        }
    }
    
    // 4. Null döndürebilen metotları nullable olarak işaretleyin
    public User? FindUserById(int id)
    {
        // Kullanıcı bulunamazsa null döndür
        return id > 0 ? new User { Id = id } : null;
    }
    
    // 5. Koleksiyonlar için boş koleksiyon kullanın, null değil
    public List<string> GetTags()
    {
        // Null yerine boş liste döndür
        return new List<string>();
    }
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
}
```

### Null Kontrolü Desenleri

Null kontrolü için çeşitli desenler ve yaklaşımlar.

```csharp
#nullable enable

// Null kontrolü desenleri
public class NullCheckPatterns
{
    // 1. Throw ifadesi
    public void ProcessWithThrow(string? input)
    {
        string nonNullInput = input ?? throw new ArgumentNullException(nameof(input));
        Console.WriteLine(nonNullInput.Length);
    }
    
    // 2. Null birleştirme operatörü
    public void ProcessWithCoalescing(string? input)
    {
        string nonNullInput = input ?? "Default";
        Console.WriteLine(nonNullInput.Length);
    }
    
    // 3. Null koşullu operatör
    public void ProcessWithConditional(string? input)
    {
        int length = input?.Length ?? 0;
        Console.WriteLine(length);
    }
    
    // 4. Pattern matching
    public void ProcessWithPatternMatching(object? input)
    {
        if (input is string text)
        {
            Console.WriteLine(text.Length);
        }
    }
    
    // 5. Guard clause
    public void ProcessWithGuardClause(string? input)
    {
        if (input == null)
        {
            return;
        }
        
        Console.WriteLine(input.Length);
    }
}
```

## Gerçek Dünya Uygulamaları

Nullable reference types özelliğinin gerçek dünya uygulamalarında nasıl kullanılabileceğine dair örnekler.

```csharp
#nullable enable

// Web API Controller örneği
public class UserController
{
    private readonly IUserRepository _repository;
    
    public UserController(IUserRepository repository)
    {
        _repository = repository ?? throw new ArgumentNullException(nameof(repository));
    }
    
    public async Task<UserDto?> GetUserById(int id)
    {
        var user = await _repository.GetByIdAsync(id);
        
        if (user == null)
        {
            return null;
        }
        
        return new UserDto
        {
            Id = user.Id,
            Name = user.Name,
            Email = user.Email ?? "N/A" // Email null olabilir
        };
    }
    
    public async Task<Result> CreateUser(CreateUserRequest request)
    {
        if (request == null)
        {
            return Result.Failure("Request cannot be null");
        }
        
        if (string.IsNullOrEmpty(request.Name))
        {
            return Result.Failure("Name is required");
        }
        
        var user = new User
        {
            Name = request.Name,
            Email = request.Email // Email nullable olabilir
        };
        
        await _repository.CreateAsync(user);
        
        return Result.Success();
    }
}

public interface IUserRepository
{
    Task<User?> GetByIdAsync(int id);
    Task CreateAsync(User user);
}

public class User
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string? Email { get; set; }
}

public class UserDto
{
    public int Id { get; set; }
    public string Name { get; set; } = "";
    public string Email { get; set; } = "";
}

public class CreateUserRequest
{
    public string Name { get; set; } = "";
    public string? Email { get; set; }
}

public class Result
{
    public bool IsSuccess { get; private set; }
    public string? ErrorMessage { get; private set; }
    
    public static Result Success() => new Result { IsSuccess = true };
    public static Result Failure(string errorMessage) => new Result { IsSuccess = false, ErrorMessage = errorMessage };
}
```

## Özet

Bu bölümde, C# 8.0 ile tanıtılan Nullable Reference Types özelliğini inceledik:

1. **Null Safety**: Null referans hatalarını derleme zamanında tespit etme yeteneği.

2. **Nullable Contexts**: Nullable aware ve nullable oblivious bağlamlar arasındaki farklar.

3. **Null-forgiving Operator**: Derleyiciye bir ifadenin null olmadığını bildirmek için kullanılan `!` operatörü.

4. **Null Analysis**: Derleyicinin kodunuzdaki null değerlerin akışını analiz etme yeteneği.

5. **Migration Strategies**: Mevcut projeleri nullable reference types özelliğine geçirmek için stratejiler.

6. **Best Practices**: Nullable reference types özelliğini en etkili şekilde kullanmak için en iyi uygulamalar.

Nullable Reference Types, kodunuzdaki null referans hatalarını azaltmanıza ve daha güvenli, daha sağlam uygulamalar geliştirmenize yardımcı olur. Bu özelliği kullanarak, çalışma zamanı hatalarını derleme zamanında yakalayabilir ve kodunuzun kalitesini artırabilirsiniz. 