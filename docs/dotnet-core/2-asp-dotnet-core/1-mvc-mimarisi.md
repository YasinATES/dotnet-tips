# MVC Mimarisi

ASP.NET Core MVC, Model-View-Controller (MVC) tasarım desenini uygulayan bir web uygulama çerçevesidir. Bu mimari, web uygulamalarını üç ana bileşene ayırarak daha modüler, test edilebilir ve sürdürülebilir kod yazmanızı sağlar.

## Controller ve Action Yapısı

Controller'lar, kullanıcı isteklerini işleyen ve uygun yanıtları döndüren sınıflardır. Her controller, belirli bir URL yolu ile ilişkilendirilir ve çeşitli HTTP isteklerini (GET, POST, PUT, DELETE vb.) işleyebilir.

### Temel Controller Yapısı

```csharp
using Microsoft.AspNetCore.Mvc;

namespace WebApp.Controllers
{
    public class HomeController : Controller
    {
        // GET: /Home/Index
        public IActionResult Index()
        {
            return View();
        }

        // GET: /Home/About
        public IActionResult About()
        {
            ViewData["Message"] = "Uygulama hakkında bilgi sayfası.";
            return View();
        }

        // GET: /Home/Contact
        public IActionResult Contact()
        {
            ViewData["Message"] = "İletişim sayfası.";
            return View();
        }
    }
}
```

### Action Metodları

Action metodları, controller sınıfları içinde tanımlanan ve HTTP isteklerini işleyen metodlardır. Her action metodu, belirli bir URL yolu ile ilişkilendirilir ve çeşitli dönüş türlerine sahip olabilir.

```csharp
// Temel action metodu - View döndürür
public IActionResult Index()
{
    return View();
}

// Belirli bir view döndürür
public IActionResult Custom()
{
    return View("CustomView");
}

// JSON döndürür
public IActionResult GetData()
{
    var data = new { Name = "Ahmet", Age = 30 };
    return Json(data);
}

// Dosya döndürür
public IActionResult Download()
{
    byte[] fileBytes = System.IO.File.ReadAllBytes("path/to/file.pdf");
    return File(fileBytes, "application/pdf", "document.pdf");
}

// Yönlendirme yapar
public IActionResult RedirectToHome()
{
    return RedirectToAction("Index", "Home");
}

// HTTP durum kodu döndürür
public IActionResult NotFound()
{
    return NotFound();
}
```

### Action Sonuç Türleri

ASP.NET Core MVC, çeşitli action sonuç türleri sunar:

- **ViewResult**: Razor view'ı render eder (`return View()`)
- **PartialViewResult**: Kısmi view'ı render eder (`return PartialView()`)
- **JsonResult**: JSON formatında veri döndürür (`return Json(data)`)
- **FileResult**: Dosya içeriği döndürür (`return File()`)
- **RedirectResult**: Başka bir URL'ye yönlendirir (`return Redirect()`)
- **RedirectToActionResult**: Başka bir action'a yönlendirir (`return RedirectToAction()`)
- **StatusCodeResult**: HTTP durum kodu döndürür (`return StatusCode()`)
- **ContentResult**: Düz metin içeriği döndürür (`return Content()`)

### Action Filtreleri

Action filtreleri, action metodlarının çalışmasını özelleştirmek için kullanılır. Bunlar, action metodunun çalışmasından önce veya sonra çalışan kod parçalarıdır.

```csharp
// Controller seviyesinde filtre
[Authorize]
public class AdminController : Controller
{
    // Action seviyesinde filtre
    [HttpPost]
    [ValidateAntiForgeryToken]
    public IActionResult Create(ProductViewModel model)
    {
        if (ModelState.IsValid)
        {
            // İş mantığı
            return RedirectToAction("Index");
        }
        return View(model);
    }
}
```

Yaygın kullanılan filtreler:

- **[Authorize]**: Kimlik doğrulama gerektirir
- **[AllowAnonymous]**: Kimlik doğrulama gerektirmez
- **[HttpGet]**, **[HttpPost]**: HTTP metodunu belirtir
- **[ValidateAntiForgeryToken]**: CSRF koruması sağlar
- **[Route]**: Özel rota tanımlar
- **[ApiController]**: API controller'ı belirtir

## Routing Mekanizması

Routing, gelen HTTP isteklerini uygun controller ve action'lara yönlendiren mekanizmadır. ASP.NET Core, hem geleneksel rota tanımlarını hem de öznitelik tabanlı rotaları destekler.

### Geleneksel Routing

Program.cs dosyasında geleneksel rota tanımı:

```csharp
var builder = WebApplication.CreateBuilder(args);

// MVC servislerini ekle
builder.Services.AddControllersWithViews();

var app = builder.Build();

// Rota yapılandırması
app.UseRouting();

app.MapControllerRoute(
    name: "default",
    pattern: "{controller=Home}/{action=Index}/{id?}");

app.Run();
```

Bu yapılandırma, aşağıdaki URL desenlerini destekler:
- `/` → `HomeController.Index()`
- `/Home` → `HomeController.Index()`
- `/Home/About` → `HomeController.About()`
- `/Products/Details/5` → `ProductsController.Details(id: 5)`

### Öznitelik Tabanlı Routing

Controller ve action seviyesinde rota tanımları:

```csharp
[Route("api/[controller]")]
[ApiController]
public class ProductsController : ControllerBase
{
    // GET: api/products
    [HttpGet]
    public IActionResult GetAll()
    {
        // Tüm ürünleri getir
        return Ok(products);
    }

    // GET: api/products/5
    [HttpGet("{id}")]
    public IActionResult GetById(int id)
    {
        // ID'ye göre ürün getir
        return Ok(product);
    }

    // POST: api/products
    [HttpPost]
    public IActionResult Create([FromBody] ProductModel model)
    {
        // Yeni ürün oluştur
        return CreatedAtAction(nameof(GetById), new { id = newProduct.Id }, newProduct);
    }

    // PUT: api/products/5
    [HttpPut("{id}")]
    public IActionResult Update(int id, [FromBody] ProductModel model)
    {
        // Ürünü güncelle
        return NoContent();
    }

    // DELETE: api/products/5
    [HttpDelete("{id}")]
    public IActionResult Delete(int id)
    {
        // Ürünü sil
        return NoContent();
    }
}
```

### Rota Kısıtlamaları

Rota parametrelerini kısıtlamak için çeşitli seçenekler mevcuttur:

```csharp
// Sayısal değer kısıtlaması
app.MapControllerRoute(
    name: "product",
    pattern: "products/{id:int}",
    defaults: new { controller = "Products", action = "Details" });

// Özel rota kısıtlaması
[Route("users/{username:alpha:minlength(5)}")]
public IActionResult GetUserByName(string username)
{
    // Kullanıcı adına göre kullanıcı getir
    return View(user);
}
```

Yaygın rota kısıtlamaları:
- **int**: Tam sayı değeri
- **bool**: Boolean değeri
- **datetime**: Tarih-saat değeri
- **decimal**: Ondalık sayı değeri
- **guid**: GUID değeri
- **minlength(value)**: Minimum uzunluk
- **maxlength(value)**: Maksimum uzunluk
- **length(min,max)**: Uzunluk aralığı
- **alpha**: Sadece alfabetik karakterler
- **regex(expression)**: Düzenli ifade

## Model Binding

Model binding, HTTP isteklerinden gelen verileri .NET nesnelerine otomatik olarak dönüştüren mekanizmadır. Bu, form verileri, URL parametreleri, JSON veya XML içeriği gibi çeşitli kaynaklardan gelen verileri işlemeyi kolaylaştırır.

### Temel Model Binding

```csharp
// URL'den parametre alma: /products/details/5
public IActionResult Details(int id)
{
    // id parametresi URL'den otomatik olarak alınır
    return View(GetProductById(id));
}

// Form verilerini alma
[HttpPost]
public IActionResult Create(ProductViewModel model)
{
    if (ModelState.IsValid)
    {
        // model nesnesi form verilerinden otomatik olarak doldurulur
        SaveProduct(model);
        return RedirectToAction("Index");
    }
    return View(model);
}

// Kompleks model binding
public IActionResult Search(ProductSearchModel search)
{
    // search nesnesi query string parametrelerinden doldurulur
    // örnek: /products/search?name=laptop&minPrice=1000&maxPrice=5000
    var results = SearchProducts(search);
    return View(results);
}
```

### Binding Kaynakları

Model binding, çeşitli kaynaklardan veri alabilir:

- **[FromQuery]**: Query string parametreleri
- **[FromRoute]**: URL rota değerleri
- **[FromForm]**: Form verileri
- **[FromBody]**: İstek gövdesi (JSON, XML)
- **[FromHeader]**: HTTP başlıkları
- **[FromServices]**: DI servislerinden

```csharp
// Farklı kaynaklardan veri alma
public IActionResult ProcessOrder(
    [FromRoute] int id,
    [FromQuery] string coupon,
    [FromBody] OrderDetails details,
    [FromHeader(Name = "User-Agent")] string userAgent,
    [FromServices] IOrderService orderService)
{
    // İş mantığı
    return Ok();
}
```

### Model Doğrulama

Model binding ile birlikte, gelen verilerin doğruluğunu kontrol etmek için model doğrulama mekanizması kullanılır. Bu, veri anotasyonları veya özel doğrulama mantığı ile yapılabilir.

```csharp
public class ProductViewModel
{
    [Required(ErrorMessage = "Ürün adı zorunludur.")]
    [StringLength(100, MinimumLength = 3, ErrorMessage = "Ürün adı 3-100 karakter arasında olmalıdır.")]
    public string Name { get; set; }

    [Required(ErrorMessage = "Fiyat zorunludur.")]
    [Range(0.01, 10000, ErrorMessage = "Fiyat 0.01 ile 10000 arasında olmalıdır.")]
    public decimal Price { get; set; }

    [Display(Name = "Stok Miktarı")]
    [Range(0, 1000, ErrorMessage = "Stok miktarı 0-1000 arasında olmalıdır.")]
    public int StockQuantity { get; set; }

    [DataType(DataType.Date)]
    [Display(Name = "Üretim Tarihi")]
    public DateTime ManufactureDate { get; set; }

    [Required(ErrorMessage = "Kategori zorunludur.")]
    public int CategoryId { get; set; }
}
```

Controller'da model doğrulama:

```csharp
[HttpPost]
public IActionResult Create(ProductViewModel model)
{
    if (!ModelState.IsValid)
    {
        // Doğrulama hatalarını görüntüle
        return View(model);
    }

    // Model geçerli, işleme devam et
    _productService.Create(model);
    return RedirectToAction("Index");
}
```

## View Engine (Razor)

Razor, ASP.NET Core MVC'de kullanılan varsayılan view engine'dir. HTML içinde C# kodu yazmanıza olanak tanır ve dinamik web sayfaları oluşturmanızı sağlar.

### Temel Razor Sözdizimi

```cshtml
@{
    ViewData["Title"] = "Ana Sayfa";
}

<h1>Hoş Geldiniz</h1>

@if (User.Identity.IsAuthenticated)
{
    <p>Merhaba, @User.Identity.Name!</p>
}
else
{
    <p>Lütfen <a asp-controller="Account" asp-action="Login">giriş yapın</a>.</p>
}

<div class="row">
    @foreach (var product in Model)
    {
        <div class="col-md-4">
            <h2>@product.Name</h2>
            <p>@product.Description</p>
            <p>Fiyat: @product.Price.ToString("C")</p>
            <a asp-controller="Products" asp-action="Details" asp-route-id="@product.Id" class="btn btn-primary">Detaylar</a>
        </div>
    }
</div>
```

### Layout Kullanımı

_Layout.cshtml dosyası (Views/Shared/_Layout.cshtml):

```cshtml
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>@ViewData["Title"] - Web Uygulaması</title>
    <link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" />
    <link rel="stylesheet" href="~/css/site.css" />
</head>
<body>
    <header>
        <nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3">
            <div class="container">
                <a class="navbar-brand" asp-controller="Home" asp-action="Index">Web Uygulaması</a>
                <button class="navbar-toggler" type="button" data-toggle="collapse" data-target=".navbar-collapse">
                    <span class="navbar-toggler-icon"></span>
                </button>
                <div class="navbar-collapse collapse d-sm-inline-flex justify-content-between">
                    <ul class="navbar-nav flex-grow-1">
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-controller="Home" asp-action="Index">Ana Sayfa</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-controller="Home" asp-action="About">Hakkında</a>
                        </li>
                        <li class="nav-item">
                            <a class="nav-link text-dark" asp-controller="Home" asp-action="Contact">İletişim</a>
                        </li>
                    </ul>
                </div>
            </div>
        </nav>
    </header>
    
    <div class="container">
        <main role="main" class="pb-3">
            @RenderBody()
        </main>
    </div>
    
    <footer class="border-top footer text-muted">
        <div class="container">
            &copy; @DateTime.Now.Year - Web Uygulaması - <a asp-controller="Home" asp-action="Privacy">Gizlilik</a>
        </div>
    </footer>
    
    <script src="~/lib/jquery/dist/jquery.min.js"></script>
    <script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script>
    <script src="~/js/site.js" asp-append-version="true"></script>
    
    @await RenderSectionAsync("Scripts", required: false)
</body>
</html>
```

View'da layout kullanımı:

```cshtml
@{
    Layout = "_Layout";
    ViewData["Title"] = "Ürün Detayları";
}

<h1>@Model.Name</h1>

<div>
    <p>@Model.Description</p>
    <p>Fiyat: @Model.Price.ToString("C")</p>
</div>
```

### Partial View Kullanımı

Partial view (_ProductCard.cshtml):

```cshtml
@model ProductViewModel

<div class="card">
    <div class="card-header">
        <h3>@Model.Name</h3>
    </div>
    <div class="card-body">
        <p>@Model.Description</p>
        <p>Fiyat: @Model.Price.ToString("C")</p>
    </div>
    <div class="card-footer">
        <a asp-controller="Products" asp-action="Details" asp-route-id="@Model.Id" class="btn btn-primary">Detaylar</a>
    </div>
</div>
```

Ana view'da partial view kullanımı:

```cshtml
<div class="row">
    @foreach (var product in Model)
    {
        <div class="col-md-4">
            <partial name="_ProductCard" model="product" />
        </div>
    }
</div>
```

### View Component Kullanımı

View component sınıfı:

```csharp
public class CartSummaryViewComponent : ViewComponent
{
    private readonly ICartService _cartService;

    public CartSummaryViewComponent(ICartService cartService)
    {
        _cartService = cartService;
    }

    public async Task<IViewComponentResult> InvokeAsync()
    {
        var items = await _cartService.GetCartItemsAsync(HttpContext.User.Identity.Name);
        return View(items);
    }
}
```

View component view'ı (Views/Shared/Components/CartSummary/Default.cshtml):

```cshtml
@model IEnumerable<CartItemViewModel>

<div class="dropdown">
    <button class="btn btn-secondary dropdown-toggle" type="button" id="cartDropdown" data-toggle="dropdown">
        Sepet (@Model.Count())
    </button>
    <div class="dropdown-menu">
        @foreach (var item in Model)
        {
            <a class="dropdown-item">@item.ProductName - @item.Quantity x @item.Price.ToString("C")</a>
        }
        <div class="dropdown-divider"></div>
        <a class="dropdown-item" asp-controller="Cart" asp-action="Index">Sepete Git</a>
    </div>
</div>
```

View component kullanımı:

```cshtml
<div class="navbar-nav ml-auto">
    @await Component.InvokeAsync("CartSummary")
</div>
```

## Tag Helpers

Tag Helper'lar, HTML etiketlerini sunucu tarafında işleyerek dinamik içerik oluşturmanızı sağlar. Razor view'larında HTML benzeri bir sözdizimi ile kullanılırlar.

### Temel Tag Helper'lar

```cshtml
<!-- Link oluşturma -->
<a asp-controller="Home" asp-action="Index" asp-route-id="5">Ana Sayfa</a>

<!-- Form oluşturma -->
<form asp-controller="Account" asp-action="Login" method="post">
    <!-- Form içeriği -->
</form>

<!-- Input alanları -->
<input asp-for="Username" class="form-control" />
<span asp-validation-for="Username" class="text-danger"></span>

<!-- Select listesi -->
<select asp-for="CountryId" asp-items="@(new SelectList(Model.Countries, "Id", "Name"))" class="form-control">
    <option value="">-- Ülke Seçin --</option>
</select>

<!-- Validation summary -->
<div asp-validation-summary="All" class="text-danger"></div>

<!-- Image tag -->
<img asp-append-version="true" src="~/images/logo.png" alt="Logo" />

<!-- Environment tag -->
<environment include="Development">
    <link rel="stylesheet" href="~/css/site.css" />
</environment>
<environment exclude="Development">
    <link rel="stylesheet" href="~/css/site.min.css" asp-append-version="true" />
</environment>
```

### Özel Tag Helper Oluşturma

```csharp
[HtmlTargetElement("email-link")]
public class EmailTagHelper : TagHelper
{
    [HtmlAttributeName("email-address")]
    public string EmailAddress { get; set; }

    [HtmlAttributeName("display-text")]
    public string DisplayText { get; set; }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagName = "a";
        output.Attributes.SetAttribute("href", $"mailto:{EmailAddress}");
        output.Content.SetContent(string.IsNullOrEmpty(DisplayText) ? EmailAddress : DisplayText);
    }
}
```

_ViewImports.cshtml dosyasında tag helper'ı kaydetme:

```cshtml
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, YourAssemblyName
```

Özel tag helper kullanımı:

```cshtml
<email-link email-address="info@example.com" display-text="Bize Ulaşın"></email-link>
```

## Özet

ASP.NET Core MVC, web uygulamaları geliştirmek için güçlü ve esnek bir mimari sunar. Controller ve action yapısı, HTTP isteklerini işlemek için temiz bir yol sağlar. Routing mekanizması, URL'leri uygun controller ve action'lara yönlendirir. Model binding, HTTP isteklerinden gelen verileri .NET nesnelerine dönüştürür. Razor view engine, dinamik web sayfaları oluşturmak için güçlü bir şablon sistemi sunar. Tag Helper'lar ise HTML etiketlerini sunucu tarafında işleyerek daha temiz ve okunabilir view'lar yazmanızı sağlar.

Bu bileşenler bir araya gelerek, modern web uygulamaları geliştirmek için kapsamlı bir çerçeve oluşturur. MVC mimarisi, uygulamanızı modüler ve test edilebilir parçalara ayırarak, büyük ve karmaşık web uygulamalarının geliştirilmesini ve bakımını kolaylaştırır. 