# Middleware Pipeline

ASP.NET Core uygulamalarında middleware, HTTP istek-yanıt döngüsünü işleyen yazılım bileşenleridir. Bu bölümde, middleware kavramını, çalışma prensibini ve ASP.NET Core uygulamalarında nasıl kullanıldığını inceleyeceğiz.

## Middleware Kavramı ve Çalışma Prensibi

Middleware, HTTP isteklerini ve yanıtlarını işleyen, pipeline (boru hattı) şeklinde düzenlenmiş yazılım bileşenleridir. Her middleware, kendisinden sonra gelen middleware'i çağırma veya yanıt döndürme kararı verebilir.

### Middleware Pipeline Yapısı

```
İstek → [Middleware 1] → [Middleware 2] → [Middleware 3] → Uygulama
Yanıt ← [Middleware 1] ← [Middleware 2] ← [Middleware 3] ← Uygulama
```

Middleware pipeline'ı şu şekilde çalışır:

1. İstek, pipeline'ın başından girer ve sırayla her middleware tarafından işlenir.
2. Her middleware, isteği işleyebilir, değiştirebilir veya sonraki middleware'e geçebilir.
3. İstek, pipeline'ın sonuna ulaştığında (genellikle endpoint middleware), uygulama isteği işler ve yanıt oluşturur.
4. Yanıt, pipeline'ı ters sırada geçerek istemciye döner.
5. Her middleware, yanıtı işleyebilir veya değiştirebilir.

### Temel Middleware Yapısı

```csharp
public class SimpleMiddleware
{
    private readonly RequestDelegate _next;

    public SimpleMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        // İstek işleme (middleware'den önce)
        Console.WriteLine($"İstek alındı: {context.Request.Path}");

        // Sonraki middleware'i çağır
        await _next(context);

        // Yanıt işleme (middleware'den sonra)
        Console.WriteLine($"Yanıt gönderildi: {context.Response.StatusCode}");
    }
}

// Extension metodu ile middleware'i kaydetme
public static class SimpleMiddlewareExtensions
{
    public static IApplicationBuilder UseSimpleMiddleware(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<SimpleMiddleware>();
    }
}
```

### Program.cs'de Middleware Kullanımı

```csharp
var builder = WebApplication.CreateBuilder(args);

// Servis kayıtları
builder.Services.AddControllers();

var app = builder.Build();

// Middleware pipeline yapılandırması
app.UseSimpleMiddleware(); // Özel middleware
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

## Yerleşik Middleware Bileşenleri

ASP.NET Core, çeşitli yerleşik middleware bileşenleri sunar. Bu bileşenler, yaygın web uygulaması işlevlerini gerçekleştirir.

### UseExceptionHandler

Uygulama hatalarını yakalayıp işler:

```csharp
// Geliştirme ortamında detaylı hata sayfaları göster
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    // Üretim ortamında özel hata sayfası göster
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}
```

### UseStaticFiles

Statik dosyaları (HTML, CSS, JavaScript, resimler vb.) sunmak için kullanılır:

```csharp
app.UseStaticFiles();

// Özel dizin ve seçeneklerle
app.UseStaticFiles(new StaticFileOptions
{
    FileProvider = new PhysicalFileProvider(
        Path.Combine(Directory.GetCurrentDirectory(), "MyStaticFiles")),
    RequestPath = "/files"
});
```

### UseRouting ve UseEndpoints

Gelen istekleri uygun endpoint'lere yönlendirir:

```csharp
app.UseRouting();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.MapRazorPages();

app.MapGet("/hello", () => "Hello World!");
```

### UseAuthentication ve UseAuthorization

Kimlik doğrulama ve yetkilendirme işlemlerini gerçekleştirir:

```csharp
app.UseAuthentication();
app.UseAuthorization();
```

### UseCors

Cross-Origin Resource Sharing (CORS) politikalarını uygular:

```csharp
// Servis kaydı
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowSpecificOrigin",
        builder => builder
            .WithOrigins("https://example.com")
            .AllowAnyMethod()
            .AllowAnyHeader());
});

// Middleware kullanımı
app.UseCors("AllowSpecificOrigin");
```

### UseResponseCompression

HTTP yanıtlarını sıkıştırır:

```csharp
// Servis kaydı
builder.Services.AddResponseCompression(options =>
{
    options.Providers.Add<GzipCompressionProvider>();
    options.Providers.Add<BrotliCompressionProvider>();
    options.EnableForHttps = true;
});

// Middleware kullanımı
app.UseResponseCompression();
```

### UseRequestLocalization

Çoklu dil desteği için istek yerelleştirme:

```csharp
// Servis kaydı
builder.Services.AddLocalization(options => options.ResourcesPath = "Resources");

// Middleware kullanımı
var supportedCultures = new[] { "tr-TR", "en-US", "fr-FR" };
app.UseRequestLocalization(options =>
{
    options.SetDefaultCulture(supportedCultures[0])
        .AddSupportedCultures(supportedCultures)
        .AddSupportedUICultures(supportedCultures);
});
```

### UseSession

Oturum durumunu yönetir:

```csharp
// Servis kaydı
builder.Services.AddDistributedMemoryCache();
builder.Services.AddSession(options =>
{
    options.IdleTimeout = TimeSpan.FromMinutes(20);
    options.Cookie.HttpOnly = true;
    options.Cookie.IsEssential = true;
});

// Middleware kullanımı
app.UseSession();
```

### UseHealthChecks

Uygulama sağlık durumunu kontrol eder:

```csharp
// Servis kaydı
builder.Services.AddHealthChecks()
    .AddCheck("Database", () => HealthCheckResult.Healthy())
    .AddCheck("ExternalService", () => HealthCheckResult.Healthy());

// Middleware kullanımı
app.MapHealthChecks("/health");
```

## Özel Middleware Geliştirme

Özel ihtiyaçlar için kendi middleware bileşenlerinizi geliştirebilirsiniz. Middleware geliştirmenin birkaç yolu vardır.

### Middleware Sınıfı Oluşturma

```csharp
public class RequestLoggingMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<RequestLoggingMiddleware> _logger;

    public RequestLoggingMiddleware(RequestDelegate next, ILogger<RequestLoggingMiddleware> logger)
    {
        _next = next;
        _logger = logger;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        try
        {
            // İstek başlangıç zamanı
            var startTime = DateTime.UtcNow;
            
            // İstek bilgilerini logla
            _logger.LogInformation(
                "İstek başladı: {Method} {Path}", 
                context.Request.Method, 
                context.Request.Path);
            
            // Sonraki middleware'i çağır
            await _next(context);
            
            // İşlem süresini hesapla
            var duration = DateTime.UtcNow - startTime;
            
            // Yanıt bilgilerini logla
            _logger.LogInformation(
                "İstek tamamlandı: {Method} {Path} => {StatusCode} ({Duration}ms)",
                context.Request.Method,
                context.Request.Path,
                context.Response.StatusCode,
                duration.TotalMilliseconds);
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "İstek işlenirken hata oluştu: {Method} {Path}",
                context.Request.Method, context.Request.Path);
            throw;
        }
    }
}

// Extension metodu
public static class RequestLoggingMiddlewareExtensions
{
    public static IApplicationBuilder UseRequestLogging(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<RequestLoggingMiddleware>();
    }
}
```

Kullanımı:

```csharp
app.UseRequestLogging();
```

### Inline Middleware Kullanımı

Basit middleware'ler için inline (satır içi) tanımlama yapılabilir:

```csharp
app.Use(async (context, next) =>
{
    // İstek işleme
    Console.WriteLine($"İstek alındı: {context.Request.Path}");
    
    // Sonraki middleware'i çağır
    await next();
    
    // Yanıt işleme
    Console.WriteLine($"Yanıt gönderildi: {context.Response.StatusCode}");
});
```

### Middleware Factory Kullanımı

Dependency Injection ile middleware oluşturmak için factory kullanılabilir:

```csharp
app.UseMiddleware<RequestLoggingMiddleware>();
```

## Use, Run, Map Metotları

ASP.NET Core, middleware pipeline'ını yapılandırmak için çeşitli extension metotları sunar.

### Use Metodu

`Use` metodu, pipeline'a middleware ekler ve sonraki middleware'i çağırma imkanı sağlar:

```csharp
app.Use(async (context, next) =>
{
    // İstek işleme
    Console.WriteLine("Middleware 1: İstek alındı");
    
    // Sonraki middleware'i çağır
    await next();
    
    // Yanıt işleme
    Console.WriteLine("Middleware 1: Yanıt gönderildi");
});
```

### Run Metodu

`Run` metodu, pipeline'ın sonunu belirtir ve sonraki middleware'leri çağırmaz:

```csharp
app.Run(async context =>
{
    await context.Response.WriteAsync("Hello from terminal middleware!");
});
```

### Map Metodu

`Map` metodu, belirli bir yol için ayrı bir middleware pipeline'ı oluşturur:

```csharp
app.Map("/admin", adminApp =>
{
    adminApp.Use(async (context, next) =>
    {
        // Admin yetki kontrolü
        if (!context.User.IsInRole("Admin"))
        {
            context.Response.StatusCode = 403;
            await context.Response.WriteAsync("Yetkisiz erişim");
            return;
        }
        
        await next();
    });
    
    adminApp.Run(async context =>
    {
        await context.Response.WriteAsync("Admin paneli");
    });
});
```

### MapWhen Metodu

`MapWhen` metodu, belirli bir koşul için ayrı bir middleware pipeline'ı oluşturur:

```csharp
app.MapWhen(
    context => context.Request.Query.ContainsKey("api-key"),
    apiApp =>
    {
        apiApp.Run(async context =>
        {
            var apiKey = context.Request.Query["api-key"];
            await context.Response.WriteAsync($"API isteği: {apiKey}");
        });
    });
```

### UseWhen Metodu

`UseWhen` metodu, belirli bir koşul için middleware çalıştırır, ancak ana pipeline'a geri döner:

```csharp
app.UseWhen(
    context => context.Request.Path.StartsWithSegments("/api"),
    apiApp =>
    {
        apiApp.Use(async (context, next) =>
        {
            // API istekleri için özel işlemler
            Console.WriteLine("API isteği alındı");
            await next();
        });
    });
```

## Middleware Sıralaması

Middleware'lerin sırası çok önemlidir, çünkü her middleware isteği ve yanıtı farklı şekillerde işleyebilir. Doğru sıralama, uygulamanızın düzgün çalışması için kritiktir.

### Önerilen Middleware Sıralaması

```csharp
// Hata işleme - en dıştaki middleware olmalı
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}

// HTTPS yönlendirme
app.UseHttpsRedirection();

// Statik dosyalar
app.UseStaticFiles();

// Routing
app.UseRouting();

// CORS
app.UseCors();

// Kimlik doğrulama ve yetkilendirme
app.UseAuthentication();
app.UseAuthorization();

// Oturum
app.UseSession();

// Endpoint'ler
app.MapControllers();
app.MapRazorPages();
```

### Sıralama Nedenleri

1. **Exception Handler**: En dıştaki middleware olmalıdır, böylece tüm hataları yakalayabilir.
2. **HTTPS Redirection**: Güvenli olmayan istekleri güvenli olanlara yönlendirmek için erken çalışmalıdır.
3. **Static Files**: Statik dosya istekleri, gereksiz işlemleri önlemek için erken işlenmelidir.
4. **Routing**: İsteğin hangi endpoint'e gideceğini belirler.
5. **CORS**: CORS başlıkları, kimlik doğrulama ve yetkilendirmeden önce eklenmelidir.
6. **Authentication/Authorization**: Kimlik doğrulama ve yetkilendirme, endpoint çalıştırılmadan önce yapılmalıdır.
7. **Session**: Oturum bilgileri, endpoint çalıştırılmadan önce erişilebilir olmalıdır.
8. **Endpoints**: Pipeline'ın sonunda, istekleri işleyecek endpoint'ler çalıştırılır.

### Yanlış Sıralama Sorunları

Yanlış middleware sıralaması çeşitli sorunlara yol açabilir:

- Kimlik doğrulama middleware'i routing'den önce gelirse, tüm istekler için kimlik doğrulama gerekebilir.
- Exception handler middleware'i pipeline'ın sonunda olursa, önceki middleware'lerdeki hataları yakalayamaz.
- CORS middleware'i kimlik doğrulamadan sonra gelirse, kimlik doğrulama hataları için CORS başlıkları eklenemez.

## Özel Middleware Örnekleri

### IP Adresi Filtreleme Middleware

```csharp
public class IpFilterMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<IpFilterMiddleware> _logger;
    private readonly List<string> _allowedIps;

    public IpFilterMiddleware(
        RequestDelegate next,
        ILogger<IpFilterMiddleware> logger,
        IConfiguration configuration)
    {
        _next = next;
        _logger = logger;
        _allowedIps = configuration.GetSection("AllowedIps").Get<List<string>>() ?? new List<string>();
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var remoteIp = context.Connection.RemoteIpAddress?.ToString();
        
        if (remoteIp != null && !_allowedIps.Contains(remoteIp))
        {
            _logger.LogWarning("Engellenen IP adresi: {IpAddress}", remoteIp);
            context.Response.StatusCode = StatusCodes.Status403Forbidden;
            await context.Response.WriteAsync("Erişim engellendi");
            return;
        }
        
        await _next(context);
    }
}

// Extension metodu
public static class IpFilterMiddlewareExtensions
{
    public static IApplicationBuilder UseIpFilter(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<IpFilterMiddleware>();
    }
}
```

### API Key Doğrulama Middleware

```csharp
public class ApiKeyMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<ApiKeyMiddleware> _logger;
    private readonly string _apiKey;

    public ApiKeyMiddleware(
        RequestDelegate next,
        ILogger<ApiKeyMiddleware> logger,
        IConfiguration configuration)
    {
        _next = next;
        _logger = logger;
        _apiKey = configuration["ApiKey"] ?? "default-api-key";
    }

    public async Task InvokeAsync(HttpContext context)
    {
        if (!context.Request.Headers.TryGetValue("X-API-Key", out var extractedApiKey))
        {
            _logger.LogWarning("API Key eksik");
            context.Response.StatusCode = StatusCodes.Status401Unauthorized;
            await context.Response.WriteAsync("API Key gerekli");
            return;
        }

        if (!_apiKey.Equals(extractedApiKey))
        {
            _logger.LogWarning("Geçersiz API Key: {ApiKey}", extractedApiKey);
            context.Response.StatusCode = StatusCodes.Status401Unauthorized;
            await context.Response.WriteAsync("Geçersiz API Key");
            return;
        }

        await _next(context);
    }
}

// Extension metodu
public static class ApiKeyMiddlewareExtensions
{
    public static IApplicationBuilder UseApiKey(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<ApiKeyMiddleware>();
    }
}
```

### Performans İzleme Middleware

```csharp
public class PerformanceMiddleware
{
    private readonly RequestDelegate _next;
    private readonly ILogger<PerformanceMiddleware> _logger;
    private readonly string _applicationName;

    public PerformanceMiddleware(
        RequestDelegate next,
        ILogger<PerformanceMiddleware> logger,
        IConfiguration configuration)
    {
        _next = next;
        _logger = logger;
        _applicationName = configuration["ApplicationName"] ?? "MyApp";
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var sw = Stopwatch.StartNew();
        
        try
        {
            await _next(context);
        }
        finally
        {
            sw.Stop();
            var elapsed = sw.ElapsedMilliseconds;
            
            if (elapsed > 500) // 500ms'den uzun süren istekler için uyarı
            {
                _logger.LogWarning(
                    "Yavaş istek: {Method} {Path} ({ElapsedMilliseconds}ms)",
                    context.Request.Method,
                    context.Request.Path,
                    elapsed);
                
                // Telemetri için header ekle
                context.Response.Headers.Add("X-Performance", elapsed.ToString());
            }
            
            // Tüm istekler için performans metriği kaydet
            _logger.LogInformation(
                "Performans: {Application} {Method} {Path} {StatusCode} {ElapsedMilliseconds}ms",
                _applicationName,
                context.Request.Method,
                context.Request.Path,
                context.Response.StatusCode,
                elapsed);
        }
    }
}

// Extension metodu
public static class PerformanceMiddlewareExtensions
{
    public static IApplicationBuilder UsePerformanceMonitoring(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<PerformanceMiddleware>();
    }
}
```

## Özet

Middleware, ASP.NET Core uygulamalarında HTTP istek-yanıt döngüsünü işleyen temel bileşenlerdir. Pipeline şeklinde düzenlenen middleware'ler, istekleri ve yanıtları işleyerek uygulamanızın davranışını şekillendirir.

ASP.NET Core, çeşitli yerleşik middleware bileşenleri sunar ve özel ihtiyaçlar için kendi middleware'lerinizi geliştirmenize olanak tanır. `Use`, `Run`, `Map` gibi metotlar, middleware pipeline'ını yapılandırmanıza yardımcı olur.

Middleware'lerin sırası, uygulamanızın düzgün çalışması için kritik öneme sahiptir. Doğru sıralama, isteklerin ve yanıtların doğru şekilde işlenmesini sağlar.

Özel middleware geliştirerek, loglama, kimlik doğrulama, performans izleme gibi çeşitli işlevleri uygulamanıza ekleyebilirsiniz. Bu, uygulamanızın modüler ve bakımı kolay olmasını sağlar. 