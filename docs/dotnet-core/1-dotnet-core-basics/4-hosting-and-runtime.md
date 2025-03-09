# Hosting ve Çalışma Zamanı

.NET uygulamalarında hosting ve çalışma zamanı, uygulamanın nasıl başlatılacağını, çalışacağını ve sonlandırılacağını belirleyen temel bileşenlerdir. Bu bölümde, .NET hosting modellerini ve çalışma zamanı bileşenlerini inceleyeceğiz.

## Generic Host vs Web Host

.NET uygulamaları için iki ana hosting modeli bulunmaktadır: Generic Host ve Web Host.

### Web Host

Web Host, ASP.NET Core web uygulamaları için tasarlanmış orijinal hosting modelidir. Temel olarak HTTP isteklerini işlemek için optimize edilmiştir.

```csharp
public static void Main(string[] args)
{
    CreateWebHostBuilder(args).Build().Run();
}

public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>();
```

Web Host'un özellikleri:
- HTTP istekleri için optimize edilmiş
- Kestrel web sunucusu entegrasyonu
- IIS entegrasyonu
- Web uygulamaları için özel yapılandırma

### Generic Host

Generic Host, tüm .NET uygulama türleri için tasarlanmış genel amaçlı bir hosting modelidir. Web uygulamaları, arka plan hizmetleri, konsol uygulamaları ve daha fazlası için kullanılabilir.

```csharp
public static void Main(string[] args)
{
    CreateHostBuilder(args).Build().Run();
}

public static IHostBuilder CreateHostBuilder(string[] args) =>
    Host.CreateDefaultBuilder(args)
        .ConfigureWebHostDefaults(webBuilder =>
        {
            webBuilder.UseStartup<Startup>();
        });
```

Generic Host'un özellikleri:
- Tüm uygulama türleri için genel amaçlı hosting
- Bağımlılık enjeksiyonu
- Yapılandırma
- Loglama
- Uygulama yaşam döngüsü yönetimi
- Hosted Service desteği

### Minimal API ile Hosting

Modern .NET uygulamalarında, Minimal API yaklaşımı ile daha basit bir hosting modeli kullanılabilir:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Servis kayıtları
builder.Services.AddControllers();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

// Middleware yapılandırması
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

// Endpoint tanımları
app.MapGet("/hello", () => "Hello World!");
app.MapPost("/items", (Item item) => Results.Created($"/items/{item.Id}", item));

app.Run();
```

Bu yaklaşımın avantajları:
- Daha az kod
- Daha basit yapı
- Startup sınıfına gerek yok
- Program.cs içinde tüm yapılandırma

## Host Builder Yapılandırması

Host Builder, uygulamanın çalışma zamanı ortamını yapılandırmak için kullanılır.

### Temel Yapılandırma

```csharp
var builder = Host.CreateDefaultBuilder(args)
    .ConfigureAppConfiguration((hostContext, config) =>
    {
        // Konfigürasyon kaynaklarını ekleme
        config.AddJsonFile("appsettings.json", optional: false);
        config.AddJsonFile($"appsettings.{hostContext.HostingEnvironment.EnvironmentName}.json", optional: true);
        config.AddEnvironmentVariables();
        
        if (hostContext.HostingEnvironment.IsDevelopment())
        {
            config.AddUserSecrets<Program>();
        }
    })
    .ConfigureLogging((hostContext, logging) =>
    {
        // Loglama yapılandırması
        logging.ClearProviders();
        logging.AddConsole();
        logging.AddDebug();
        logging.AddEventSourceLogger();
        
        // Log seviyelerini konfigürasyondan alma
        logging.AddConfiguration(hostContext.Configuration.GetSection("Logging"));
    })
    .ConfigureServices((hostContext, services) =>
    {
        // Servis kayıtları
        services.AddHostedService<MyBackgroundService>();
        services.AddSingleton<IMyService, MyService>();
        services.AddScoped<IMyRepository, MyRepository>();
    });

var host = builder.Build();
await host.RunAsync();
```

### Web Uygulaması Yapılandırması

```csharp
var builder = WebApplication.CreateBuilder(args);

// Servis kayıtları
builder.Services.AddControllers();
builder.Services.AddRazorPages();
builder.Services.AddDbContext<ApplicationDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("DefaultConnection")));

// Loglama yapılandırması
builder.Logging.ClearProviders();
builder.Logging.AddConsole();
builder.Logging.AddDebug();

// Konfigürasyon kaynakları ekleme
builder.Configuration.AddJsonFile("customsettings.json", optional: true);
builder.Configuration.AddEnvironmentVariables(prefix: "MYAPP_");

// Host yapılandırması
builder.Host.UseWindowsService();
builder.Host.UseSerilog((context, config) =>
{
    config.ReadFrom.Configuration(context.Configuration);
});

var app = builder.Build();

// Middleware yapılandırması
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

app.MapControllers();
app.MapRazorPages();

await app.RunAsync();
```

### Özel Host Yapılandırması

```csharp
var host = Host.CreateDefaultBuilder(args)
    // Özel içerik kök dizini
    .UseContentRoot(Directory.GetCurrentDirectory())
    
    // Özel ortam adı
    .UseEnvironment("Production")
    
    // Özel uygulama adı
    .ConfigureHostConfiguration(config =>
    {
        config.AddInMemoryCollection(new Dictionary<string, string>
        {
            { HostDefaults.ApplicationKey, "MyCustomApp" }
        });
    })
    
    // Özel DI konteyner
    .UseServiceProviderFactory(new AutofacServiceProviderFactory())
    .ConfigureContainer<ContainerBuilder>(builder =>
    {
        builder.RegisterModule(new MyApplicationModule());
    })
    
    // Özel yaşam döngüsü
    .ConfigureHostOptions(options =>
    {
        options.ShutdownTimeout = TimeSpan.FromSeconds(30);
        options.BackgroundServiceExceptionBehavior = BackgroundServiceExceptionBehavior.StopHost;
    })
    
    .Build();

await host.RunAsync();
```

## Background Services

Background Services, uygulamanın arka planında sürekli çalışan ve belirli görevleri yerine getiren hizmetlerdir. `BackgroundService` sınıfını genişleterek oluşturulurlar.

### Temel Background Service

```csharp
public class TimedBackgroundService : BackgroundService
{
    private readonly ILogger<TimedBackgroundService> _logger;
    private readonly PeriodicTimer _timer;
    
    public TimedBackgroundService(ILogger<TimedBackgroundService> logger)
    {
        _logger = logger;
        _timer = new PeriodicTimer(TimeSpan.FromSeconds(10));
    }
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("Timed Background Service başlatıldı.");
        
        try
        {
            while (await _timer.WaitForNextTickAsync(stoppingToken))
            {
                await DoWorkAsync(stoppingToken);
            }
        }
        catch (OperationCanceledException)
        {
            _logger.LogInformation("Timed Background Service durduruldu.");
        }
    }
    
    private async Task DoWorkAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("Timed Background Service çalışıyor: {Time}", DateTimeOffset.Now);
        
        // Uzun süren işlem simülasyonu
        await Task.Delay(1000, stoppingToken);
    }
    
    public override async Task StopAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("Timed Background Service durduruluyor...");
        
        // Temizlik işlemleri
        _timer.Dispose();
        
        await base.StopAsync(stoppingToken);
    }
}
```

### Scoped Service Kullanımı

Background Service'ler singleton yaşam döngüsüne sahiptir, ancak scoped servisleri kullanmak için `IServiceScopeFactory` kullanılabilir:

```csharp
public class ScopedBackgroundService : BackgroundService
{
    private readonly ILogger<ScopedBackgroundService> _logger;
    private readonly IServiceScopeFactory _scopeFactory;
    
    public ScopedBackgroundService(
        ILogger<ScopedBackgroundService> logger,
        IServiceScopeFactory scopeFactory)
    {
        _logger = logger;
        _scopeFactory = scopeFactory;
    }
    
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _logger.LogInformation("Scoped Background Service başlatıldı.");
        
        await DoWorkAsync(stoppingToken);
    }
    
    private async Task DoWorkAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            // Yeni bir scope oluşturma
            using (var scope = _scopeFactory.CreateScope())
            {
                // Scope'tan servis alma
                var scopedService = scope.ServiceProvider.GetRequiredService<IScopedService>();
                
                // Servis ile işlem yapma
                await scopedService.DoWorkAsync();
            }
            
            await Task.Delay(TimeSpan.FromMinutes(1), stoppingToken);
        }
    }
}
```

### Kayıt ve Kullanım

```csharp
// Servis kaydı
builder.Services.AddHostedService<TimedBackgroundService>();
builder.Services.AddHostedService<ScopedBackgroundService>();

// Scoped servis kaydı
builder.Services.AddScoped<IScopedService, ScopedService>();
```

## Worker Services

Worker Service, arka planda çalışan uzun süreli işlemler için tasarlanmış bir uygulama türüdür. Background Service'leri barındırmak için kullanılır ve Windows Service veya Linux daemon olarak çalıştırılabilir.

### Worker Service Oluşturma

```bash
dotnet new worker -n MyWorkerService
```

Bu komut, aşağıdaki temel dosyaları içeren bir Worker Service projesi oluşturur:
- Program.cs
- Worker.cs
- appsettings.json
- appsettings.Development.json

### Temel Worker Sınıfı

```csharp
public class Worker : BackgroundService
{
    private readonly ILogger<Worker> _logger;

    public Worker(ILogger<Worker> logger)
    {
        _logger = logger;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            _logger.LogInformation("Worker running at: {time}", DateTimeOffset.Now);
            await Task.Delay(1000, stoppingToken);
        }
    }
}
```

### Windows Service Olarak Yapılandırma

```csharp
// Program.cs
var builder = Host.CreateDefaultBuilder(args)
    .UseWindowsService(options =>
    {
        options.ServiceName = "MyWorkerService";
    })
    .ConfigureServices(services =>
    {
        services.AddHostedService<Worker>();
    });

var host = builder.Build();
host.Run();
```

Windows Service olarak yükleme:

```bash
sc create MyWorkerService binPath="C:\path\to\MyWorkerService.exe"
sc start MyWorkerService
```

### Linux Daemon Olarak Yapılandırma

```csharp
// Program.cs
var builder = Host.CreateDefaultBuilder(args)
    .UseSystemd()
    .ConfigureServices(services =>
    {
        services.AddHostedService<Worker>();
    });

var host = builder.Build();
host.Run();
```

Linux systemd service dosyası (/etc/systemd/system/myworkerservice.service):

```
[Unit]
Description=My Worker Service

[Service]
Type=notify
ExecStart=/usr/share/myworkerservice/MyWorkerService

[Install]
WantedBy=multi-user.target
```

Linux service komutları:

```bash
sudo systemctl enable myworkerservice
sudo systemctl start myworkerservice
sudo systemctl status myworkerservice
```

## Hosted Services

Hosted Service, uygulamanın yaşam döngüsüne bağlı olarak çalışan ve `IHostedService` arayüzünü uygulayan hizmetlerdir. Background Service'ler de Hosted Service'lerin özel bir türüdür.

### IHostedService Arayüzü

```csharp
public interface IHostedService
{
    Task StartAsync(CancellationToken cancellationToken);
    Task StopAsync(CancellationToken cancellationToken);
}
```

### Temel Hosted Service

```csharp
public class MyHostedService : IHostedService
{
    private readonly ILogger<MyHostedService> _logger;
    private Timer _timer;
    
    public MyHostedService(ILogger<MyHostedService> logger)
    {
        _logger = logger;
    }
    
    public Task StartAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Hosted Service başlatıldı.");
        
        _timer = new Timer(DoWork, null, TimeSpan.Zero, TimeSpan.FromSeconds(5));
        
        return Task.CompletedTask;
    }
    
    private void DoWork(object state)
    {
        _logger.LogInformation("Hosted Service çalışıyor: {Time}", DateTimeOffset.Now);
    }
    
    public Task StopAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Hosted Service durduruluyor...");
        
        _timer?.Change(Timeout.Infinite, 0);
        
        return Task.CompletedTask;
    }
    
    public void Dispose()
    {
        _timer?.Dispose();
    }
}
```

### Kayıt ve Kullanım

```csharp
// Servis kaydı
builder.Services.AddHostedService<MyHostedService>();
```

### Startup Task Olarak Hosted Service

Uygulama başlatıldığında bir kez çalışan ve tamamlanan görevler için Hosted Service kullanılabilir:

```csharp
public class MigrationHostedService : IHostedService
{
    private readonly IServiceProvider _serviceProvider;
    private readonly ILogger<MigrationHostedService> _logger;
    
    public MigrationHostedService(
        IServiceProvider serviceProvider,
        ILogger<MigrationHostedService> logger)
    {
        _serviceProvider = serviceProvider;
        _logger = logger;
    }
    
    public async Task StartAsync(CancellationToken cancellationToken)
    {
        _logger.LogInformation("Veritabanı migrasyonu başlatılıyor...");
        
        using (var scope = _serviceProvider.CreateScope())
        {
            var dbContext = scope.ServiceProvider.GetRequiredService<ApplicationDbContext>();
            
            try
            {
                await dbContext.Database.MigrateAsync(cancellationToken);
                _logger.LogInformation("Veritabanı migrasyonu tamamlandı.");
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, "Veritabanı migrasyonu sırasında hata oluştu.");
                throw;
            }
        }
    }
    
    public Task StopAsync(CancellationToken cancellationToken)
    {
        return Task.CompletedTask;
    }
}
```

## Uygulama Yaşam Döngüsü

.NET uygulamalarının yaşam döngüsü, host tarafından yönetilir ve çeşitli olaylar aracılığıyla izlenebilir.

### IHostApplicationLifetime

`IHostApplicationLifetime` arayüzü, uygulamanın yaşam döngüsü olaylarını izlemek ve kontrol etmek için kullanılır:

```csharp
public class LifetimeEventsHostedService : IHostedService
{
    private readonly IHostApplicationLifetime _appLifetime;
    private readonly ILogger<LifetimeEventsHostedService> _logger;
    
    public LifetimeEventsHostedService(
        IHostApplicationLifetime appLifetime,
        ILogger<LifetimeEventsHostedService> logger)
    {
        _appLifetime = appLifetime;
        _logger = logger;
    }
    
    public Task StartAsync(CancellationToken cancellationToken)
    {
        // Başlatma olayı
        _appLifetime.ApplicationStarted.Register(() =>
        {
            _logger.LogInformation("Uygulama başlatıldı.");
            
            // Başlatma işlemleri
        });
        
        // Durdurma olayı
        _appLifetime.ApplicationStopping.Register(() =>
        {
            _logger.LogInformation("Uygulama durduruluyor...");
            
            // Temizlik işlemleri
        });
        
        // Durdurulma olayı
        _appLifetime.ApplicationStopped.Register(() =>
        {
            _logger.LogInformation("Uygulama durduruldu.");
        });
        
        return Task.CompletedTask;
    }
    
    public Task StopAsync(CancellationToken cancellationToken)
    {
        return Task.CompletedTask;
    }
}
```

### Graceful Shutdown

Uygulamanın düzgün bir şekilde kapatılması için graceful shutdown mekanizması kullanılabilir:

```csharp
public static async Task Main(string[] args)
{
    var host = CreateHostBuilder(args).Build();
    
    // Graceful shutdown için sinyal işleyicileri
    Console.CancelKeyPress += (sender, e) =>
    {
        e.Cancel = true; // Uygulamanın hemen sonlandırılmasını engelle
        host.StopAsync().Wait(); // Host'u düzgün bir şekilde durdur
    };
    
    await host.RunAsync();
}
```

## Özet

.NET uygulamalarında hosting ve çalışma zamanı, uygulamanın nasıl başlatılacağını, çalışacağını ve sonlandırılacağını belirleyen temel bileşenlerdir. Bu bölümde, Generic Host ve Web Host arasındaki farkları, Host Builder yapılandırmasını, Background Services, Worker Services ve Hosted Services kavramlarını inceledik.

Modern .NET uygulamalarında, Generic Host modeli tüm uygulama türleri için standart bir hosting çözümü sunar. Background Services ve Worker Services, arka planda çalışan uzun süreli işlemler için ideal çözümlerdir. Hosted Services ise uygulamanın yaşam döngüsüne bağlı olarak çalışan hizmetler oluşturmak için kullanılır.

Bu bileşenler, .NET uygulamalarının güvenilir, ölçeklenebilir ve yönetilebilir olmasını sağlar. 