# Konfigürasyon Sistemi

.NET uygulamalarında konfigürasyon sistemi, uygulamanın davranışını ve ayarlarını dış kaynaklardan yönetmeyi sağlayan güçlü bir mekanizmadır. Bu bölümde, .NET konfigürasyon sisteminin temel bileşenlerini ve kullanım senaryolarını inceleyeceğiz.

## appsettings.json Yapısı

`appsettings.json` dosyası, .NET uygulamalarında konfigürasyon ayarlarını depolamak için kullanılan temel JSON dosyasıdır.

### Temel Yapı

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=localhost;Database=MyDatabase;User Id=sa;Password=YourPassword;"
  },
  "AppSettings": {
    "ApiKey": "your-api-key",
    "MaxItemCount": 100,
    "EnableFeature": true
  }
}
```

### Ortama Özgü Konfigürasyon

Farklı ortamlar için farklı konfigürasyon dosyaları oluşturabilirsiniz:

- `appsettings.json`: Temel konfigürasyon
- `appsettings.Development.json`: Geliştirme ortamı için konfigürasyon
- `appsettings.Production.json`: Üretim ortamı için konfigürasyon
- `appsettings.Staging.json`: Test ortamı için konfigürasyon

Ortama özgü dosyalar, temel dosyadaki ayarları geçersiz kılar (override).

### Hiyerarşik Yapı

JSON formatı, hiyerarşik konfigürasyon yapısı oluşturmaya olanak tanır:

```json
{
  "FeatureManagement": {
    "AdvancedSearch": true,
    "UserManagement": {
      "EnableRegistration": true,
      "RequireEmailConfirmation": true,
      "PasswordPolicy": {
        "MinimumLength": 8,
        "RequireDigit": true,
        "RequireUppercase": true
      }
    }
  }
}
```

### Diziler

JSON dizileri kullanarak liste şeklinde konfigürasyon değerleri tanımlayabilirsiniz:

```json
{
  "AllowedOrigins": [
    "https://example.com",
    "https://api.example.com",
    "https://admin.example.com"
  ],
  "EmailSettings": {
    "AdminEmails": [
      "admin1@example.com",
      "admin2@example.com"
    ]
  }
}
```

## Ortam Değişkenleri

Ortam değişkenleri, özellikle konteyner ortamlarında ve bulut platformlarında konfigürasyon değerlerini ayarlamak için yaygın olarak kullanılır.

### Ortam Değişkenlerini Kullanma

.NET konfigürasyon sistemi, ortam değişkenlerini otomatik olarak yükler ve `appsettings.json` dosyasındaki değerleri geçersiz kılabilir.

Örnek ortam değişkenleri:

```
ASPNETCORE_ENVIRONMENT=Production
ConnectionStrings__DefaultConnection=Server=prod-server;Database=ProdDb;User Id=prod-user;Password=prod-password;
AppSettings__ApiKey=production-api-key
Logging__LogLevel__Default=Warning
```

### Hiyerarşik Yapı

JSON yapısındaki iç içe geçmiş özellikler, ortam değişkenlerinde çift alt çizgi (`__`) ile ayrılır:

- JSON: `{ "Logging": { "LogLevel": { "Default": "Information" } } }`
- Ortam Değişkeni: `Logging__LogLevel__Default=Warning`

### Öncelik Sırası

Ortam değişkenleri, genellikle diğer konfigürasyon kaynaklarından daha yüksek önceliğe sahiptir. Bu, güvenlik açısından hassas bilgileri (bağlantı dizesi, API anahtarları vb.) ortam değişkenleri aracılığıyla sağlamak için kullanışlıdır.

## User Secrets

User Secrets, geliştirme ortamında hassas bilgileri güvenli bir şekilde depolamak için kullanılan bir mekanizmadır. Bu bilgiler, kaynak kontrol sistemine gönderilmez.

### User Secrets Başlatma

```bash
dotnet user-secrets init --project MyProject.csproj
```

Bu komut, proje dosyasına bir `UserSecretsId` ekler:

```xml
<PropertyGroup>
  <UserSecretsId>aspnet-MyProject-12345678-1234-1234-1234-1234567890AB</UserSecretsId>
</PropertyGroup>
```

### Değer Ekleme

```bash
dotnet user-secrets set "ConnectionStrings:DefaultConnection" "Server=localhost;Database=MyDatabase;User Id=dev-user;Password=dev-password;"
dotnet user-secrets set "AppSettings:ApiKey" "development-api-key"
```

### Birden Fazla Değer Ekleme

```bash
dotnet user-secrets set "EmailSettings:AdminEmails:0" "admin1@example.com"
dotnet user-secrets set "EmailSettings:AdminEmails:1" "admin2@example.com"
```

### Değerleri Listeleme

```bash
dotnet user-secrets list
```

### Değer Kaldırma

```bash
dotnet user-secrets remove "AppSettings:ApiKey"
```

### Tüm Değerleri Temizleme

```bash
dotnet user-secrets clear
```

### Depolama Konumu

User Secrets, işletim sistemine bağlı olarak aşağıdaki konumlarda saklanır:

- Windows: `%APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json`
- macOS/Linux: `~/.microsoft/usersecrets/<user_secrets_id>/secrets.json`

## Options Pattern

Options Pattern, konfigürasyon değerlerini güçlü bir şekilde tiplendirilmiş sınıflara bağlamak için kullanılan bir tasarım desenidir.

### Options Sınıfı Tanımlama

```csharp
public class AppSettings
{
    public string ApiKey { get; set; }
    public int MaxItemCount { get; set; }
    public bool EnableFeature { get; set; }
}

public class ConnectionStrings
{
    public string DefaultConnection { get; set; }
}

public class EmailSettings
{
    public List<string> AdminEmails { get; set; }
    public string SmtpServer { get; set; }
    public int SmtpPort { get; set; }
    public string SmtpUsername { get; set; }
    public string SmtpPassword { get; set; }
}
```

### Servis Kaydı

```csharp
// Program.cs
builder.Services.Configure<AppSettings>(
    builder.Configuration.GetSection("AppSettings"));

builder.Services.Configure<ConnectionStrings>(
    builder.Configuration.GetSection("ConnectionStrings"));

builder.Services.Configure<EmailSettings>(
    builder.Configuration.GetSection("EmailSettings"));
```

### Kullanım

```csharp
// Constructor Injection
public class EmailService
{
    private readonly AppSettings _appSettings;
    private readonly EmailSettings _emailSettings;
    
    public EmailService(
        IOptions<AppSettings> appSettings,
        IOptions<EmailSettings> emailSettings)
    {
        _appSettings = appSettings.Value;
        _emailSettings = emailSettings.Value;
    }
    
    public void SendEmail()
    {
        // Konfigürasyon değerlerini kullanma
        if (_appSettings.EnableFeature)
        {
            // _emailSettings.SmtpServer, _emailSettings.AdminEmails vb. kullanarak e-posta gönderme
        }
    }
}
```

### IOptions, IOptionsSnapshot ve IOptionsMonitor

- **IOptions<T>**: Singleton yaşam döngüsüne sahiptir ve uygulama başlatıldıktan sonra değişiklikleri algılamaz.
  
  ```csharp
  public class MyService
  {
      private readonly AppSettings _appSettings;
      
      public MyService(IOptions<AppSettings> appSettings)
      {
          _appSettings = appSettings.Value;
      }
  }
  ```

- **IOptionsSnapshot<T>**: Scoped yaşam döngüsüne sahiptir ve her istek için yeniden değerlendirilir.
  
  ```csharp
  public class MyService
  {
      private readonly IOptionsSnapshot<AppSettings> _appSettings;
      
      public MyService(IOptionsSnapshot<AppSettings> appSettings)
      {
          // Her kullanımda .Value özelliğine erişim
      }
      
      public void DoSomething()
      {
          var apiKey = _appSettings.Value.ApiKey;
      }
  }
  ```

- **IOptionsMonitor<T>**: Singleton yaşam döngüsüne sahiptir, ancak konfigürasyon değişikliklerini algılar ve bildirim mekanizması sağlar.
  
  ```csharp
  public class MyService
  {
      private readonly IOptionsMonitor<AppSettings> _appSettings;
      private readonly IDisposable _optionsChangeListener;
      
      public MyService(IOptionsMonitor<AppSettings> appSettings)
      {
          _appSettings = appSettings;
          
          // Değişiklikleri dinleme
          _optionsChangeListener = _appSettings.OnChange(settings =>
          {
              Console.WriteLine($"AppSettings değişti. Yeni ApiKey: {settings.ApiKey}");
              // Değişikliğe göre işlem yapma
          });
      }
      
      public void DoSomething()
      {
          var apiKey = _appSettings.CurrentValue.ApiKey;
      }
      
      public void Dispose()
      {
          _optionsChangeListener?.Dispose();
      }
  }
  ```

### Doğrulama

Options sınıflarınızı doğrulamak için `Validate` metodunu kullanabilirsiniz:

```csharp
builder.Services.AddOptions<EmailSettings>()
    .Bind(builder.Configuration.GetSection("EmailSettings"))
    .ValidateDataAnnotations()
    .Validate(settings =>
    {
        if (string.IsNullOrEmpty(settings.SmtpServer))
            return false;
        if (settings.SmtpPort <= 0)
            return false;
        return true;
    }, "SMTP ayarları geçersiz.");
```

Data Annotations ile doğrulama:

```csharp
public class EmailSettings
{
    [Required]
    public List<string> AdminEmails { get; set; }
    
    [Required]
    [MinLength(3)]
    public string SmtpServer { get; set; }
    
    [Range(1, 65535)]
    public int SmtpPort { get; set; }
    
    [Required]
    public string SmtpUsername { get; set; }
    
    [Required]
    public string SmtpPassword { get; set; }
}
```

## IConfiguration Arayüzü

`IConfiguration` arayüzü, konfigürasyon sisteminin temel bileşenidir ve farklı kaynaklardan gelen konfigürasyon değerlerine erişim sağlar.

### Temel Kullanım

```csharp
public class ConfigurationExample
{
    private readonly IConfiguration _configuration;
    
    public ConfigurationExample(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    
    public void ReadConfiguration()
    {
        // Basit değer okuma
        string apiKey = _configuration["AppSettings:ApiKey"];
        
        // GetValue ile tip dönüşümü
        int maxItemCount = _configuration.GetValue<int>("AppSettings:MaxItemCount");
        bool enableFeature = _configuration.GetValue<bool>("AppSettings:EnableFeature");
        
        // Varsayılan değer ile okuma
        int timeout = _configuration.GetValue<int>("AppSettings:Timeout", 30);
        
        // Bağlantı dizesi okuma
        string connectionString = _configuration.GetConnectionString("DefaultConnection");
        
        // Section okuma
        IConfigurationSection appSettingsSection = _configuration.GetSection("AppSettings");
        string apiKeyFromSection = appSettingsSection["ApiKey"];
        
        // Dizi okuma
        string[] allowedOrigins = _configuration.GetSection("AllowedOrigins").Get<string[]>();
    }
}
```

### Konfigürasyon Oluşturma

```csharp
// Konsol uygulaması veya test için manuel konfigürasyon oluşturma
IConfiguration configuration = new ConfigurationBuilder()
    .SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("appsettings.json", optional: false, reloadOnChange: true)
    .AddJsonFile($"appsettings.{Environment.GetEnvironmentVariable("ASPNETCORE_ENVIRONMENT") ?? "Production"}.json", optional: true)
    .AddEnvironmentVariables()
    .AddUserSecrets<Program>()
    .AddCommandLine(args)
    .Build();
```

### Konfigürasyon Kaynakları

.NET konfigürasyon sistemi, çeşitli kaynaklardan konfigürasyon yükleyebilir:

```csharp
IConfiguration configuration = new ConfigurationBuilder()
    // JSON dosyaları
    .AddJsonFile("appsettings.json")
    .AddJsonFile("config.json")
    
    // XML dosyaları
    .AddXmlFile("settings.xml")
    
    // INI dosyaları
    .AddIniFile("config.ini")
    
    // Ortam değişkenleri
    .AddEnvironmentVariables()
    
    // Komut satırı argümanları
    .AddCommandLine(args)
    
    // User Secrets
    .AddUserSecrets<Program>()
    
    // Azure Key Vault
    .AddAzureKeyVault(new Uri("https://myvault.vault.azure.net/"), new DefaultAzureCredential())
    
    // In-memory koleksiyon
    .AddInMemoryCollection(new Dictionary<string, string>
    {
        {"Key1", "Value1"},
        {"Key2", "Value2"}
    })
    
    // Özel konfigürasyon sağlayıcısı
    .Add(new CustomConfigurationSource())
    
    .Build();
```

### Değişiklikleri İzleme

```csharp
IChangeToken changeToken = configuration.GetReloadToken();
changeToken.RegisterChangeCallback(state =>
{
    Console.WriteLine("Konfigürasyon değişti!");
    // Değişikliğe göre işlem yapma
    
    // Yeni değişiklikleri izlemeye devam etme
    IChangeToken newToken = configuration.GetReloadToken();
    // ...
}, null);
```

## Konfigürasyon Sistemi Best Practices

### 1. Hassas Bilgileri Güvenli Şekilde Saklama

- Geliştirme ortamında User Secrets kullanın
- Üretim ortamında ortam değişkenleri veya Azure Key Vault gibi güvenli depolama alanları kullanın
- Bağlantı dizelerini ve API anahtarlarını asla kaynak kontrole göndermeyin

### 2. Hiyerarşik Yapı Kullanma

- İlgili ayarları mantıksal bölümlerde gruplayın
- Düz bir yapı yerine hiyerarşik bir yapı tercih edin

### 3. Güçlü Tipli Konfigürasyon

- Options Pattern kullanarak konfigürasyonu sınıflara bağlayın
- Data Annotations ile doğrulama ekleyin
- IOptionsSnapshot veya IOptionsMonitor kullanarak değişiklikleri izleyin

### 4. Ortama Özgü Konfigürasyon

- Farklı ortamlar için ayrı konfigürasyon dosyaları kullanın
- ASPNETCORE_ENVIRONMENT ortam değişkenini ayarlayın
- Geliştirme ortamında daha ayrıntılı loglama, üretim ortamında daha az loglama yapın

### 5. Varsayılan Değerler

- Konfigürasyon değerleri eksik olduğunda kullanılacak varsayılan değerler belirleyin
- GetValue<T> metodunu varsayılan değerlerle kullanın

## Özet

.NET konfigürasyon sistemi, uygulamanın davranışını ve ayarlarını dış kaynaklardan yönetmeyi sağlayan güçlü bir mekanizmadır. Bu bölümde, appsettings.json yapısını, ortam değişkenlerini, user secrets'ı, options pattern'i ve IConfiguration arayüzünü inceledik.

Konfigürasyon sistemi, farklı ortamlarda (geliştirme, test, üretim) farklı ayarlar kullanmanıza, hassas bilgileri güvenli bir şekilde saklamanıza ve konfigürasyon değerlerini güçlü tipli sınıflara bağlamanıza olanak tanır. Bu özellikler, .NET uygulamalarının esnekliğini ve güvenliğini artırır. 