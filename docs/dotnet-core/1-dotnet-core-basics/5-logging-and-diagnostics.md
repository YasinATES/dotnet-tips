# Logging ve Diagnostik

.NET uygulamalarında loglama ve diagnostik, uygulamanın davranışını izlemek, hataları tespit etmek ve performans sorunlarını belirlemek için kritik öneme sahiptir. Bu bölümde, .NET'in loglama ve diagnostik yeteneklerini inceleyeceğiz.

## ILogger Arayüzü

`ILogger` arayüzü, .NET'in yerleşik loglama sisteminin temelidir ve uygulamanızda tutarlı bir loglama yaklaşımı sağlar.

### Temel Kullanım

```csharp
public class WeatherController : ControllerBase
{
    private readonly ILogger<WeatherController> _logger;

    public WeatherController(ILogger<WeatherController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IActionResult Get()
    {
        _logger.LogInformation("Hava durumu verileri alınıyor.");
        
        try
        {
            // İş mantığı
            var result = GetWeatherData();
            
            _logger.LogDebug("Alınan veri sayısı: {Count}", result.Count);
            
            return Ok(result);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Hava durumu verileri alınırken hata oluştu.");
            return StatusCode(500);
        }
    }
}
```

### Yapılandırma

Program.cs dosyasında loglama yapılandırması:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Loglama yapılandırması
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();
builder.Logging.AddEventSourceLogger();

// Konfigürasyondan loglama ayarlarını alma
builder.Logging.AddConfiguration(builder.Configuration.GetSection("Logging"));

var app = builder.Build();
```

appsettings.json dosyasında loglama yapılandırması:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "MyApp.Controllers": "Debug"
    },
    "Console": {
      "LogLevel": {
        "Default": "Information",
        "Microsoft": "Warning"
      },
      "FormatterName": "json",
      "FormatterOptions": {
        "SingleLine": true,
        "IncludeScopes": true,
        "TimestampFormat": "HH:mm:ss ",
        "UseUtcTimestamp": true,
        "JsonWriterOptions": {
          "Indented": true
        }
      }
    }
  }
}
```

### Structured Logging

Structured logging, log mesajlarını düz metin yerine yapılandırılmış veriler olarak kaydeder, bu da daha sonra sorgulama ve analiz için daha kullanışlıdır.

```csharp
// Basit mesaj
_logger.LogInformation("Kullanıcı giriş yaptı.");

// Yapılandırılmış log
_logger.LogInformation("Kullanıcı {UserId} {Action} işlemini gerçekleştirdi.", userId, "giriş");

// Nesne ile log
var user = new { Id = 123, Name = "Ahmet", Role = "Admin" };
_logger.LogInformation("Kullanıcı bilgileri: {@User}", user);
```

### Log Scopes

Log scope'ları, bir dizi log mesajı için ortak bağlam bilgisi sağlar:

```csharp
public IActionResult ProcessOrder(int orderId)
{
    using (_logger.BeginScope("OrderProcessing {OrderId}", orderId))
    {
        _logger.LogInformation("Sipariş işlemi başlatıldı.");
        
        // İş mantığı
        
        _logger.LogInformation("Ödeme işlemi başlatıldı.");
        
        // Ödeme işlemi
        
        _logger.LogInformation("Sipariş işlemi tamamlandı.");
    }
    
    return Ok();
}
```

## Log Seviyeleri ve Filtreleme

.NET, farklı önem derecelerine sahip log mesajları için çeşitli log seviyeleri sunar.

### Log Seviyeleri

```csharp
// Trace - En ayrıntılı seviye, genellikle sadece geliştirme sırasında kullanılır
_logger.LogTrace("Çok ayrıntılı tanılama bilgisi.");

// Debug - Geliştirme sırasında hata ayıklama için kullanışlı bilgiler
_logger.LogDebug("Hata ayıklama bilgisi.");

// Information - Uygulamanın normal akışını izlemek için genel bilgiler
_logger.LogInformation("Uygulama başlatıldı.");

// Warning - Potansiyel sorunlar veya beklenmeyen durumlar
_logger.LogWarning("Dosya bulunamadı: {FilePath}", filePath);

// Error - Hata durumları, işlem başarısız oldu ancak uygulama çalışmaya devam edebilir
_logger.LogError(exception, "Veritabanı bağlantısı başarısız oldu.");

// Critical - Kritik hatalar, uygulamanın çökmesine veya çalışamamasına neden olabilir
_logger.LogCritical(exception, "Uygulama başlatılamadı.");
```

### Filtreleme

appsettings.json dosyasında log filtreleme:

```json
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information",
      "MyApp.Services": "Debug",
      "MyApp.Controllers.UserController": "Debug"
    }
  }
}
```

Kod içinde filtreleme:

```csharp
builder.Logging.AddFilter("Microsoft", LogLevel.Warning);
builder.Logging.AddFilter("System", LogLevel.Warning);
builder.Logging.AddFilter("MyApp.Services", LogLevel.Debug);
```

Kategori bazlı filtreleme:

```csharp
builder.Logging.AddFilter<ConsoleLoggerProvider>("Microsoft", LogLevel.Warning);
builder.Logging.AddFilter<DebugLoggerProvider>("MyApp", LogLevel.Trace);
```

## Serilog, NLog Entegrasyonu

.NET'in yerleşik loglama sistemi temel ihtiyaçları karşılar, ancak daha gelişmiş senaryolar için Serilog ve NLog gibi üçüncü taraf loglama kütüphaneleri kullanılabilir.

### Serilog Entegrasyonu

Paket kurulumu:

```bash
dotnet add package Serilog.AspNetCore
dotnet add package Serilog.Sinks.Console
dotnet add package Serilog.Sinks.File
dotnet add package Serilog.Settings.Configuration
```

Program.cs dosyasında yapılandırma:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Serilog yapılandırması
builder.Host.UseSerilog((context, services, configuration) => configuration
    .ReadFrom.Configuration(context.Configuration)
    .ReadFrom.Services(services)
    .Enrich.FromLogContext()
    .WriteTo.Console()
    .WriteTo.File("logs/app.log", rollingInterval: RollingInterval.Day));

var app = builder.Build();
```

appsettings.json dosyasında Serilog yapılandırması:

```json
{
  "Serilog": {
    "MinimumLevel": {
      "Default": "Information",
      "Override": {
        "Microsoft": "Warning",
        "System": "Warning"
      }
    },
    "WriteTo": [
      {
        "Name": "Console",
        "Args": {
          "outputTemplate": "[{Timestamp:HH:mm:ss} {Level:u3}] {Message:lj}{NewLine}{Exception}"
        }
      },
      {
        "Name": "File",
        "Args": {
          "path": "logs/app.log",
          "rollingInterval": "Day",
          "outputTemplate": "{Timestamp:yyyy-MM-dd HH:mm:ss.fff zzz} [{Level:u3}] {Message:lj}{NewLine}{Exception}"
        }
      }
    ],
    "Enrich": [ "FromLogContext", "WithMachineName", "WithThreadId" ]
  }
}
```

Kullanım:

```csharp
public class UserService
{
    private readonly ILogger<UserService> _logger;
    
    public UserService(ILogger<UserService> logger)
    {
        _logger = logger;
    }
    
    public async Task<User> GetUserAsync(int userId)
    {
        _logger.LogInformation("Kullanıcı bilgileri alınıyor: {UserId}", userId);
        
        // İş mantığı
    }
}
```

### NLog Entegrasyonu

Paket kurulumu:

```bash
dotnet add package NLog.Web.AspNetCore
```

Program.cs dosyasında yapılandırma:

```csharp
var builder = WebApplication.CreateBuilder(args);

// NLog yapılandırması
builder.Logging.ClearProviders();
builder.Host.UseNLog();

var app = builder.Build();
```

nlog.config dosyası:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true"
      internalLogLevel="Info"
      internalLogFile="logs/internal-nlog.log">

  <!-- Hedefler -->
  <targets>
    <!-- Dosya hedefi -->
    <target xsi:type="File" name="allfile" fileName="logs/app-${shortdate}.log"
            layout="${longdate}|${event-properties:item=EventId_Id:whenEmpty=0}|${uppercase:${level}}|${logger}|${message} ${exception:format=tostring}" />

    <!-- Konsol hedefi -->
    <target xsi:type="Console" name="lifetimeConsole" 
            layout="${MicrosoftConsoleLayout}" />
  </targets>

  <!-- Kurallar -->
  <rules>
    <!-- Tüm logları dosyaya yaz -->
    <logger name="*" minlevel="Trace" writeTo="allfile" />

    <!-- Microsoft loglarını konsola yaz -->
    <logger name="Microsoft.*" minlevel="Info" writeTo="lifetimeConsole" final="true" />
    
    <!-- Diğer tüm logları konsola yaz -->
    <logger name="*" minlevel="Info" writeTo="lifetimeConsole" />
  </rules>
</nlog>
```

## Application Insights

Application Insights, Azure'un uygulama performans yönetimi (APM) hizmetidir ve .NET uygulamalarıyla kolayca entegre edilebilir.

### Kurulum ve Yapılandırma

Paket kurulumu:

```bash
dotnet add package Microsoft.ApplicationInsights.AspNetCore
```

Program.cs dosyasında yapılandırma:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Application Insights ekleme
builder.Services.AddApplicationInsightsTelemetry();

var app = builder.Build();
```

appsettings.json dosyasında yapılandırma:

```json
{
  "ApplicationInsights": {
    "ConnectionString": "InstrumentationKey=your-instrumentation-key;IngestionEndpoint=https://your-region.in.applicationinsights.azure.com/",
    "EnableAdaptiveSampling": true,
    "EnablePerformanceCounterCollectionModule": true
  }
}
```

### Özel Telemetri

```csharp
public class OrderController : ControllerBase
{
    private readonly ILogger<OrderController> _logger;
    private readonly TelemetryClient _telemetryClient;
    
    public OrderController(
        ILogger<OrderController> logger,
        TelemetryClient telemetryClient)
    {
        _logger = logger;
        _telemetryClient = telemetryClient;
    }
    
    [HttpPost]
    public async Task<IActionResult> CreateOrder(OrderRequest request)
    {
        // İşlem başlangıç zamanı
        var startTime = DateTime.UtcNow;
        
        // Özel özellikler
        var properties = new Dictionary<string, string>
        {
            { "OrderType", request.Type },
            { "CustomerId", request.CustomerId.ToString() }
        };
        
        // Özel metrikler
        var metrics = new Dictionary<string, double>
        {
            { "OrderAmount", request.Amount }
        };
        
        try
        {
            // İş mantığı
            var result = await _orderService.CreateOrderAsync(request);
            
            // Başarılı işlem telemetrisi
            _telemetryClient.TrackEvent("OrderCreated", properties, metrics);
            
            // İşlem süresini ölçme
            var duration = DateTime.UtcNow - startTime;
            _telemetryClient.TrackMetric("OrderCreationTime", duration.TotalMilliseconds);
            
            return Ok(result);
        }
        catch (Exception ex)
        {
            // Hata telemetrisi
            _telemetryClient.TrackException(ex, properties, metrics);
            _logger.LogError(ex, "Sipariş oluşturulurken hata oluştu.");
            
            return StatusCode(500);
        }
    }
}
```

### Dependency Tracking

Application Insights, HTTP çağrıları, SQL sorguları ve diğer bağımlılıkları otomatik olarak izler. Özel bağımlılıkları izlemek için:

```csharp
public async Task<ExternalData> GetExternalDataAsync(string id)
{
    // Bağımlılık izleme başlat
    using (var operation = _telemetryClient.StartOperation<DependencyTelemetry>("ExternalService"))
    {
        operation.Telemetry.Type = "External";
        operation.Telemetry.Target = "ExternalService";
        operation.Telemetry.Data = $"GetData/{id}";
        
        try
        {
            // Dış servis çağrısı
            var result = await _externalService.GetDataAsync(id);
            
            // Başarılı sonuç
            operation.Telemetry.Success = true;
            return result;
        }
        catch (Exception ex)
        {
            // Başarısız sonuç
            operation.Telemetry.Success = false;
            _telemetryClient.TrackException(ex);
            throw;
        }
    }
}
```

## Health Checks

Health checks, uygulamanızın ve bağımlılıklarının durumunu izlemek için kullanılır.

### Temel Health Checks

Servis kaydı:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Health checks ekleme
builder.Services.AddHealthChecks()
    // Basit health check
    .AddCheck("Self", () => HealthCheckResult.Healthy())
    // Veritabanı health check
    .AddDbContextCheck<ApplicationDbContext>()
    // URL health check
    .AddUrlGroup(new Uri("https://example.com/health"), name: "external-api")
    // Disk alanı health check
    .AddDiskStorageHealthCheck(setup =>
    {
        setup.AddDrive("C:\\", 1024); // En az 1 GB boş alan gerekli
    })
    // Bellek health check
    .AddProcessAllocatedMemoryHealthCheck(512); // En fazla 512 MB bellek kullanımı

var app = builder.Build();

// Health check endpoint'i
app.MapHealthChecks("/health");
```

### Gelişmiş Health Checks

Özel health check sınıfı:

```csharp
public class ExternalServiceHealthCheck : IHealthCheck
{
    private readonly IExternalService _externalService;
    
    public ExternalServiceHealthCheck(IExternalService externalService)
    {
        _externalService = externalService;
    }
    
    public async Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context, 
        CancellationToken cancellationToken = default)
    {
        try
        {
            // Dış servisin durumunu kontrol et
            var isHealthy = await _externalService.IsHealthyAsync();
            
            if (isHealthy)
            {
                return HealthCheckResult.Healthy("External service is healthy.");
            }
            
            return HealthCheckResult.Degraded("External service is experiencing issues.");
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("External service is unhealthy.", ex);
        }
    }
}
```

Servis kaydı:

```csharp
// Özel health check kaydı
builder.Services.AddHealthChecks()
    .AddCheck<ExternalServiceHealthCheck>("external-service");
```

### Health Check UI

Health Check UI, health check sonuçlarını görselleştirmek için kullanılır.

Paket kurulumu:

```bash
dotnet add package AspNetCore.HealthChecks.UI
dotnet add package AspNetCore.HealthChecks.UI.InMemory.Storage
dotnet add package AspNetCore.HealthChecks.UI.Client
```

Servis kaydı:

```csharp
// Health Check UI ekleme
builder.Services.AddHealthChecksUI(options =>
{
    options.SetEvaluationTimeInSeconds(60); // Her 60 saniyede bir kontrol et
    options.MaximumHistoryEntriesPerEndpoint(10); // Her endpoint için en fazla 10 kayıt tut
    options.AddHealthCheckEndpoint("API", "/health"); // Health check endpoint'i
})
.AddInMemoryStorage(); // Sonuçları bellekte sakla

var app = builder.Build();

// Health check endpoint'i
app.MapHealthChecks("/health", new HealthCheckOptions
{
    Predicate = _ => true,
    ResponseWriter = UIResponseWriter.WriteHealthCheckUIResponse
});

// Health check UI endpoint'i
app.MapHealthChecksUI(options =>
{
    options.UIPath = "/health-ui"; // UI endpoint'i
    options.ApiPath = "/health-api"; // API endpoint'i
});
```

## Özet

.NET uygulamalarında loglama ve diagnostik, uygulamanın davranışını izlemek, hataları tespit etmek ve performans sorunlarını belirlemek için kritik öneme sahiptir. Bu bölümde, ILogger arayüzünü, log seviyelerini ve filtrelemeyi, Serilog ve NLog entegrasyonunu, Application Insights'ı ve health checks'i inceledik.

Etkili bir loglama ve diagnostik stratejisi, uygulamanızın üretim ortamında sorunsuz çalışmasını sağlamak ve sorunları hızlı bir şekilde tespit etmek için önemlidir. Doğru log seviyelerini kullanmak, yapılandırılmış loglama yapmak ve uygun loglama hedeflerini seçmek, uygulamanızın izlenebilirliğini artırır.

Application Insights gibi APM araçları, uygulamanızın performansını ve kullanımını izlemek için değerli bilgiler sağlar. Health checks ise uygulamanızın ve bağımlılıklarının durumunu izlemek için kullanışlıdır.

Bu bileşenler, .NET uygulamalarının güvenilirliğini ve yönetilebilirliğini artırır. 