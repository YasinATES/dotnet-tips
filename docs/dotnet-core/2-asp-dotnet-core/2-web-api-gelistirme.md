# Web API Geliştirme

ASP.NET Core Web API, HTTP protokolü üzerinden veri alışverişi yapan servisler geliştirmek için kullanılan güçlü bir çerçevedir. Bu bölümde, modern web API'leri geliştirmenin temel kavramlarını ve tekniklerini inceleyeceğiz.

## RESTful API Tasarımı

REST (Representational State Transfer), web servisleri tasarlamak için kullanılan bir mimari stildir. RESTful API'ler, kaynakları URL'ler aracılığıyla temsil eder ve HTTP metodlarını kullanarak bu kaynaklar üzerinde işlemler gerçekleştirir.

### RESTful Tasarım İlkeleri

1. **Kaynak Odaklı**: Her şey bir kaynaktır ve her kaynak benzersiz bir URL ile tanımlanır.
2. **HTTP Metodlarının Doğru Kullanımı**: CRUD işlemleri için uygun HTTP metodları kullanılır.
3. **Durumsuzluk**: Sunucu, istemci durumunu saklamaz.
4. **Önbelleğe Alınabilirlik**: Yanıtlar önbelleğe alınabilir olmalıdır.
5. **Katmanlı Sistem**: İstemci, doğrudan son sunucuya bağlı olduğunu bilmez.
6. **Uniform Interface**: Kaynaklar için standart bir arayüz kullanılır.

### HTTP Metodları ve CRUD İşlemleri

| HTTP Metodu | CRUD İşlemi | Açıklama |
|-------------|-------------|----------|
| GET | Read | Kaynak veya kaynak koleksiyonu alır |
| POST | Create | Yeni bir kaynak oluşturur |
| PUT | Update/Replace | Bir kaynağı tamamen günceller/değiştirir |
| PATCH | Update/Modify | Bir kaynağı kısmen günceller |
| DELETE | Delete | Bir kaynağı siler |

### URL Tasarımı

İyi tasarlanmış RESTful API URL'leri:

```
# Koleksiyon alma
GET /api/products

# Belirli bir öğe alma
GET /api/products/5

# İlişkili kaynakları alma
GET /api/products/5/reviews

# Filtreleme
GET /api/products?category=electronics&minPrice=100

# Sıralama
GET /api/products?sort=price_desc

# Sayfalama
GET /api/products?page=2&pageSize=10
```

### Temel Controller Yapısı

```csharp
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    private readonly IProductRepository _repository;

    public ProductsController(IProductRepository repository)
    {
        _repository = repository;
    }

    // GET: api/products
    [HttpGet]
    public async Task<ActionResult<IEnumerable<Product>>> GetProducts()
    {
        var products = await _repository.GetAllAsync();
        return Ok(products);
    }

    // GET: api/products/5
    [HttpGet("{id}")]
    public async Task<ActionResult<Product>> GetProduct(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        
        if (product == null)
        {
            return NotFound();
        }
        
        return Ok(product);
    }

    // POST: api/products
    [HttpPost]
    public async Task<ActionResult<Product>> CreateProduct(ProductCreateDto productDto)
    {
        var product = new Product
        {
            Name = productDto.Name,
            Price = productDto.Price,
            Description = productDto.Description
        };
        
        await _repository.AddAsync(product);
        
        return CreatedAtAction(nameof(GetProduct), new { id = product.Id }, product);
    }

    // PUT: api/products/5
    [HttpPut("{id}")]
    public async Task<IActionResult> UpdateProduct(int id, ProductUpdateDto productDto)
    {
        var product = await _repository.GetByIdAsync(id);
        
        if (product == null)
        {
            return NotFound();
        }
        
        product.Name = productDto.Name;
        product.Price = productDto.Price;
        product.Description = productDto.Description;
        
        await _repository.UpdateAsync(product);
        
        return NoContent();
    }

    // DELETE: api/products/5
    [HttpDelete("{id}")]
    public async Task<IActionResult> DeleteProduct(int id)
    {
        var product = await _repository.GetByIdAsync(id);
        
        if (product == null)
        {
            return NotFound();
        }
        
        await _repository.DeleteAsync(product);
        
        return NoContent();
    }
}
```

### Durum Kodları

RESTful API'lerde doğru HTTP durum kodlarını kullanmak önemlidir:

| Durum Kodu | Açıklama |
|------------|----------|
| 200 OK | İstek başarılı |
| 201 Created | Kaynak başarıyla oluşturuldu |
| 204 No Content | İstek başarılı, ancak içerik yok (genellikle PUT/DELETE sonrası) |
| 400 Bad Request | İstek hatalı veya eksik |
| 401 Unauthorized | Kimlik doğrulama gerekli |
| 403 Forbidden | Kimlik doğrulandı ancak yetki yok |
| 404 Not Found | Kaynak bulunamadı |
| 409 Conflict | Kaynak durumu ile çakışma |
| 500 Internal Server Error | Sunucu hatası |

## Content Negotiation

Content negotiation, istemci ve sunucu arasında hangi veri formatının kullanılacağına karar verme sürecidir. ASP.NET Core, varsayılan olarak JSON formatını destekler, ancak XML ve diğer formatlar da eklenebilir.

### Accept Header Kullanımı

İstemci, `Accept` başlığını kullanarak tercih ettiği formatı belirtebilir:

```
Accept: application/json
Accept: application/xml
```

### Formatters Yapılandırması

Program.cs dosyasında formatters yapılandırması:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
        options.JsonSerializerOptions.WriteIndented = true;
    })
    .AddXmlSerializerFormatters(); // XML desteği ekler

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Özel Formatter Ekleme

Özel bir veri formatı eklemek için:

```csharp
public class CsvOutputFormatter : TextOutputFormatter
{
    public CsvOutputFormatter()
    {
        SupportedMediaTypes.Add(MediaTypeHeaderValue.Parse("text/csv"));
        SupportedEncodings.Add(Encoding.UTF8);
        SupportedEncodings.Add(Encoding.Unicode);
    }

    protected override bool CanWriteType(Type type)
    {
        if (typeof(IEnumerable<object>).IsAssignableFrom(type) || 
            typeof(object).IsAssignableFrom(type))
        {
            return true;
        }
        return false;
    }

    public override async Task WriteResponseBodyAsync(
        OutputFormatterWriteContext context, Encoding selectedEncoding)
    {
        var response = context.HttpContext.Response;
        var buffer = new StringBuilder();

        if (context.Object is IEnumerable<object> enumerable)
        {
            foreach (var obj in enumerable)
            {
                FormatCsv(buffer, obj);
            }
        }
        else
        {
            FormatCsv(buffer, context.Object);
        }

        await response.WriteAsync(buffer.ToString(), selectedEncoding);
    }

    private static void FormatCsv(StringBuilder buffer, object obj)
    {
        var properties = obj.GetType().GetProperties();
        
        for (var i = 0; i < properties.Length; i++)
        {
            var value = properties[i].GetValue(obj);
            buffer.Append(value);
            
            if (i < properties.Length - 1)
            {
                buffer.Append(",");
            }
        }
        
        buffer.AppendLine();
    }
}
```

Formatter'ı kaydetme:

```csharp
builder.Services.AddControllers(options =>
{
    options.OutputFormatters.Add(new CsvOutputFormatter());
});
```

### Controller'da Format Belirtme

Belirli bir action için format belirtme:

```csharp
[HttpGet]
[Produces("application/json", "application/xml", "text/csv")]
public async Task<ActionResult<IEnumerable<Product>>> GetProducts()
{
    var products = await _repository.GetAllAsync();
    return Ok(products);
}
```

## API Versiyonlama

API versiyonlama, API'nizin değişen gereksinimlerle birlikte evrimleşmesine olanak tanırken, mevcut istemcilerin çalışmaya devam etmesini sağlar.

### Versiyonlama Stratejileri

1. **URL Yolu Versiyonlama**: `/api/v1/products`, `/api/v2/products`
2. **Query String Versiyonlama**: `/api/products?api-version=1.0`
3. **Header Versiyonlama**: `X-API-Version: 1.0` başlığı kullanma
4. **Media Type Versiyonlama**: `Accept: application/vnd.company.product+json;version=1.0`

### API Versiyonlama Paketi

Paket kurulumu:

```bash
dotnet add package Microsoft.AspNetCore.Mvc.Versioning
dotnet add package Microsoft.AspNetCore.Mvc.Versioning.ApiExplorer
```

### Temel Yapılandırma

Program.cs dosyasında versiyonlama yapılandırması:

```csharp
var builder = WebApplication.CreateBuilder(args);

// API versiyonlama servislerini ekle
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ReportApiVersions = true;
    options.ApiVersionReader = ApiVersionReader.Combine(
        new UrlSegmentApiVersionReader(),
        new QueryStringApiVersionReader("api-version"),
        new HeaderApiVersionReader("X-API-Version"));
});

// API Explorer için versiyonlama desteği
builder.Services.AddVersionedApiExplorer(options =>
{
    options.GroupNameFormat = "'v'VVV";
    options.SubstituteApiVersionInUrl = true;
});

builder.Services.AddControllers();

var app = builder.Build();

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Controller'da Versiyonlama

```csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsV1Controller : ControllerBase
{
    // V1 API implementasyonu
}

[ApiController]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsV2Controller : ControllerBase
{
    // V2 API implementasyonu
}
```

Aynı controller içinde farklı versiyonlar:

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    [HttpGet]
    [ApiVersion("1.0")]
    [MapToApiVersion("1.0")]
    public ActionResult<IEnumerable<ProductV1Dto>> GetProductsV1()
    {
        // V1 implementasyonu
        return Ok(productsV1);
    }

    [HttpGet]
    [ApiVersion("2.0")]
    [MapToApiVersion("2.0")]
    public ActionResult<IEnumerable<ProductV2Dto>> GetProductsV2()
    {
        // V2 implementasyonu
        return Ok(productsV2);
    }
}
```

### API Sürüm Kullanım Dışı Bırakma

```csharp
[ApiController]
[ApiVersion("1.0", Deprecated = true)]
[ApiVersion("2.0")]
[Route("api/v{version:apiVersion}/products")]
public class ProductsController : ControllerBase
{
    // API implementasyonu
}
```

## OpenAPI (Swagger) Entegrasyonu

OpenAPI (eski adıyla Swagger), API'lerinizi belgelendirmek ve test etmek için kullanılan bir standarttır. ASP.NET Core, Swashbuckle paketi ile OpenAPI desteği sağlar.

### Paket Kurulumu

```bash
dotnet add package Swashbuckle.AspNetCore
```

### Temel Yapılandırma

Program.cs dosyasında Swagger yapılandırması:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers();

// Swagger/OpenAPI yapılandırması
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API",
        Version = "v1",
        Description = "An API to perform operations",
        Contact = new OpenApiContact
        {
            Name = "Contact Name",
            Email = "contact@example.com",
            Url = new Uri("https://example.com/contact")
        },
        License = new OpenApiLicense
        {
            Name = "License",
            Url = new Uri("https://example.com/license")
        }
    });
    
    // XML belgeleri ekle
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
});

var app = builder.Build();

// Swagger middleware'i yapılandır
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(options =>
    {
        options.SwaggerEndpoint("/swagger/v1/swagger.json", "v1");
        options.RoutePrefix = string.Empty; // Kök URL'de Swagger UI'ı göster
    });
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### XML Belgeleri Etkinleştirme

.csproj dosyasında XML belgeleri etkinleştirme:

```xml
<PropertyGroup>
  <GenerateDocumentationFile>true</GenerateDocumentationFile>
  <NoWarn>$(NoWarn);1591</NoWarn>
</PropertyGroup>
```

### Controller ve Action Belgelendirme

```csharp
/// <summary>
/// Ürünleri yönetmek için API.
/// </summary>
[ApiController]
[Route("api/[controller]")]
public class ProductsController : ControllerBase
{
    /// <summary>
    /// Tüm ürünleri getirir.
    /// </summary>
    /// <returns>Ürün listesi</returns>
    /// <response code="200">Ürünler başarıyla getirildi</response>
    [HttpGet]
    [ProducesResponseType(StatusCodes.Status200OK)]
    public async Task<ActionResult<IEnumerable<Product>>> GetProducts()
    {
        // İmplementasyon
    }

    /// <summary>
    /// Belirli bir ürünü ID'ye göre getirir.
    /// </summary>
    /// <param name="id">Ürün ID'si</param>
    /// <returns>Ürün</returns>
    /// <response code="200">Ürün başarıyla getirildi</response>
    /// <response code="404">Ürün bulunamadı</response>
    [HttpGet("{id}")]
    [ProducesResponseType(StatusCodes.Status200OK)]
    [ProducesResponseType(StatusCodes.Status404NotFound)]
    public async Task<ActionResult<Product>> GetProduct(int id)
    {
        // İmplementasyon
    }
}
```

### Versiyonlu API için Swagger

Versiyonlu API'ler için Swagger yapılandırması:

```csharp
builder.Services.AddSwaggerGen(options =>
{
    // V1 için belgelendirme
    options.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "My API V1",
        Version = "v1",
        Description = "V1 API"
    });
    
    // V2 için belgelendirme
    options.SwaggerDoc("v2", new OpenApiInfo
    {
        Title = "My API V2",
        Version = "v2",
        Description = "V2 API"
    });
    
    // Versiyonları ayırt etmek için
    options.OperationFilter<SwaggerDefaultValues>();
    
    // XML belgeleri ekle
    var xmlFile = $"{Assembly.GetExecutingAssembly().GetName().Name}.xml";
    var xmlPath = Path.Combine(AppContext.BaseDirectory, xmlFile);
    options.IncludeXmlComments(xmlPath);
});

// Swagger UI yapılandırması
app.UseSwaggerUI(options =>
{
    options.SwaggerEndpoint("/swagger/v1/swagger.json", "v1");
    options.SwaggerEndpoint("/swagger/v2/swagger.json", "v2");
    options.RoutePrefix = string.Empty;
});
```

## Problem Details (RFC 7807)

Problem Details, HTTP API'lerde hataları standart bir formatta döndürmek için kullanılan bir RFC (7807) standardıdır. ASP.NET Core, bu standardı destekler ve hata yanıtlarını otomatik olarak bu formatta döndürebilir.

### Problem Details Formatı

```json
{
  "type": "https://tools.ietf.org/html/rfc7231#section-6.5.4",
  "title": "Not Found",
  "status": 404,
  "detail": "The product with ID 5 was not found.",
  "traceId": "00-9a4ec3d0b5d36e4f8066d4a73e67af00-56740bdf9c2a764d-00"
}
```

### Yapılandırma

Program.cs dosyasında Problem Details yapılandırması:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Problem Details'ı etkinleştir
builder.Services.AddProblemDetails(options =>
{
    // Özel problem detayları için yapılandırma
    options.CustomizeProblemDetails = context =>
    {
        context.ProblemDetails.Extensions.Add("serverId", Environment.MachineName);
        
        if (context.HttpContext.Request.Path.StartsWithSegments("/api"))
        {
            context.ProblemDetails.Type = "https://example.com/api/errors";
        }
    };
});

builder.Services.AddControllers()
    .AddJsonOptions(options =>
    {
        options.JsonSerializerOptions.PropertyNamingPolicy = JsonNamingPolicy.CamelCase;
    });

var app = builder.Build();

// Geliştirme ortamında detaylı hata sayfaları göster
if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}
else
{
    // Üretim ortamında Problem Details kullan
    app.UseExceptionHandler();
    app.UseStatusCodePages();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

### Özel Problem Details Oluşturma

```csharp
[HttpGet("{id}")]
public async Task<ActionResult<Product>> GetProduct(int id)
{
    var product = await _repository.GetByIdAsync(id);
    
    if (product == null)
    {
        var problemDetails = new ProblemDetails
        {
            Status = StatusCodes.Status404NotFound,
            Title = "Product Not Found",
            Detail = $"The product with ID {id} was not found.",
            Type = "https://example.com/errors/not-found",
            Instance = HttpContext.Request.Path
        };
        
        return NotFound(problemDetails);
    }
    
    return Ok(product);
}
```

### Özel Hata Tipleri

Özel hata tipleri tanımlama:

```csharp
public class ValidationProblemDetails : ProblemDetails
{
    public ValidationProblemDetails(ActionContext context)
    {
        Title = "Validation errors occurred";
        Status = StatusCodes.Status400BadRequest;
        Type = "https://example.com/errors/validation";
        Instance = context.HttpContext.Request.Path;
        
        var errors = new Dictionary<string, string[]>();
        
        foreach (var keyModelStatePair in context.ModelState)
        {
            var key = keyModelStatePair.Key;
            var errorMessages = keyModelStatePair.Value.Errors
                .Select(x => x.ErrorMessage)
                .ToArray();
            
            errors.Add(key, errorMessages);
        }
        
        Extensions.Add("errors", errors);
    }
}
```

Özel filtreleme ile kullanma:

```csharp
public class ValidationProblemDetailsFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
        {
            context.Result = new BadRequestObjectResult(
                new ValidationProblemDetails(context));
        }
    }

    public void OnActionExecuted(ActionExecutedContext context)
    {
    }
}
```

Filtreyi kaydetme:

```csharp
builder.Services.AddControllers(options =>
{
    options.Filters.Add<ValidationProblemDetailsFilter>();
});
```

## Özet

Bu bölümde, ASP.NET Core ile modern web API'leri geliştirmenin temel kavramlarını ve tekniklerini inceledik. RESTful API tasarımı, kaynakları URL'ler aracılığıyla temsil etmeyi ve HTTP metodlarını kullanarak bu kaynaklar üzerinde işlemler gerçekleştirmeyi sağlar. Content negotiation, istemci ve sunucu arasında hangi veri formatının kullanılacağına karar verme sürecidir. API versiyonlama, API'nizin değişen gereksinimlerle birlikte evrimleşmesine olanak tanırken, mevcut istemcilerin çalışmaya devam etmesini sağlar. OpenAPI (Swagger) entegrasyonu, API'lerinizi belgelendirmek ve test etmek için kullanılır. Problem Details (RFC 7807), HTTP API'lerde hataları standart bir formatta döndürmek için kullanılır.

Bu bileşenler bir araya gelerek, modern, ölçeklenebilir ve bakımı kolay web API'leri geliştirmenizi sağlar. 