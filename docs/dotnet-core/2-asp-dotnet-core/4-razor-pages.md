# Razor Pages

Razor Pages, ASP.NET Core'da sayfa tabanlı web uygulamaları geliştirmek için kullanılan bir programlama modelidir. MVC'ye benzer ancak daha basit bir yapı sunar ve özellikle CRUD işlemleri için daha az kod yazmanızı sağlar.

## Page Model Yapısı

Razor Pages, Page Model desenini kullanır. Her Razor sayfası (.cshtml) bir model sınıfı (.cshtml.cs) ile eşleştirilir. Bu model sınıfı, sayfanın davranışını ve veri işleme mantığını içerir.

### Temel Sayfa Yapısı

Razor sayfası (Index.cshtml):

```cshtml
@page
@model IndexModel
@{
    ViewData["Title"] = "Ana Sayfa";
}

<h1>Hoş Geldiniz</h1>
<p>Merhaba, @Model.Message</p>

<form method="post">
    <button type="submit">Selamla</button>
</form>
```

Page Model sınıfı (Index.cshtml.cs):

```csharp
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace WebApp.Pages
{
    public class IndexModel : PageModel
    {
        public string Message { get; set; } = "";

        public void OnGet()
        {
            Message = "Ziyaretçi";
        }

        public void OnPost()
        {
            Message = "Dünya";
        }
    }
}
```

### Program.cs'de Razor Pages Yapılandırması

```csharp
var builder = WebApplication.CreateBuilder(args);

// Razor Pages servislerini ekle
builder.Services.AddRazorPages();

var app = builder.Build();

// Middleware yapılandırması
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();
app.UseAuthorization();

// Razor Pages endpoint'lerini yapılandır
app.MapRazorPages();

app.Run();
```

### PageModel Özellikleri

PageModel sınıfı, sayfa işlevselliğini sağlayan çeşitli özellikler sunar:

```csharp
public class ProductDetailModel : PageModel
{
    private readonly IProductService _productService;
    
    // Constructor Dependency Injection
    public ProductDetailModel(IProductService productService)
    {
        _productService = productService;
    }
    
    // Sayfa verileri için property'ler
    [BindProperty]
    public ProductViewModel Product { get; set; } = default!;
    
    // Sayfa parametreleri
    public string ErrorMessage { get; set; } = "";
    
    // HTTP context'e erişim
    public IActionResult OnGet(int id)
    {
        // PageContext, HttpContext, Request, Response, RouteData, User, ModelState
        // gibi PageModel'den miras alınan özelliklere erişim
        var user = User.Identity?.Name;
        var path = Request.Path;
        
        var product = _productService.GetById(id);
        
        if (product == null)
        {
            ErrorMessage = "Ürün bulunamadı";
            return Page(); // Mevcut sayfayı döndür
        }
        
        Product = new ProductViewModel
        {
            Id = product.Id,
            Name = product.Name,
            Price = product.Price
        };
        
        return Page();
    }
}
```

## Routing

Razor Pages, dosya sistemi tabanlı routing kullanır. Sayfalar, `Pages` klasörü altında bulunur ve URL yolları dosya yollarına karşılık gelir.

### Temel Routing

| Dosya Yolu | URL |
|------------|-----|
| /Pages/Index.cshtml | / veya /Index |
| /Pages/Products/List.cshtml | /Products/List |
| /Pages/Blog/Posts/Recent.cshtml | /Blog/Posts/Recent |

### Route Parametreleri

Sayfa yolunda parametreler tanımlanabilir:

```cshtml
@page "{id:int}"
@model ProductDetailModel
@{
    ViewData["Title"] = "Ürün Detayı";
}

<h1>@Model.Product.Name</h1>
<p>Fiyat: @Model.Product.Price.ToString("C")</p>
```

```csharp
public class ProductDetailModel : PageModel
{
    private readonly IProductService _productService;
    
    public ProductDetailModel(IProductService productService)
    {
        _productService = productService;
    }
    
    public ProductViewModel Product { get; set; } = default!;
    
    public IActionResult OnGet(int id)
    {
        var product = _productService.GetById(id);
        
        if (product == null)
        {
            return NotFound();
        }
        
        Product = new ProductViewModel
        {
            Id = product.Id,
            Name = product.Name,
            Price = product.Price
        };
        
        return Page();
    }
}
```

### Özel Route Tanımlama

```cshtml
@page "/urunler/{categoryName}/{id:int?}"
@model ProductsByCategoryModel
```

```csharp
public class ProductsByCategoryModel : PageModel
{
    public void OnGet(string categoryName, int? id)
    {
        // categoryName ve id parametrelerini kullan
    }
}
```

### Route Kısıtlamaları

```cshtml
@page "{id:int:min(1)}/{slug:alpha:minlength(5)}"
@model ProductDetailModel
```

Yaygın route kısıtlamaları:
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

### Sayfa Yönlendirme

```csharp
public IActionResult OnPost()
{
    if (!ModelState.IsValid)
    {
        return Page();
    }
    
    // İşlem başarılı, başka bir sayfaya yönlendir
    return RedirectToPage("Success");
}

// Parametreli yönlendirme
public IActionResult OnPost(int id)
{
    // İşlem başarılı, parametreli sayfaya yönlendir
    return RedirectToPage("Detail", new { id });
}

// Farklı bir klasördeki sayfaya yönlendirme
public IActionResult OnPost()
{
    return RedirectToPage("/Products/List");
}

// Başka bir controller/action'a yönlendirme
public IActionResult OnPost()
{
    return RedirectToAction("Index", "Home");
}
```

## Handler Metotları

Razor Pages, HTTP isteklerini işlemek için handler metotları kullanır. Bu metotlar, HTTP metoduna göre adlandırılır ve sayfa yaşam döngüsünü yönetir.

### Temel Handler Metotları

```csharp
public class ContactModel : PageModel
{
    [BindProperty]
    public ContactViewModel Contact { get; set; } = default!;
    
    // GET isteği için
    public void OnGet()
    {
        // Sayfa ilk yüklendiğinde çalışır
    }
    
    // POST isteği için
    public IActionResult OnPost()
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }
        
        // Form verilerini işle
        // Contact property'si otomatik olarak form verilerinden doldurulur
        
        return RedirectToPage("ThankYou");
    }
    
    // PUT isteği için
    public IActionResult OnPut()
    {
        // PUT isteğini işle
        return Page();
    }
    
    // DELETE isteği için
    public IActionResult OnDelete()
    {
        // DELETE isteğini işle
        return Page();
    }
}
```

### Asenkron Handler Metotları

```csharp
public class ProductsModel : PageModel
{
    private readonly IProductService _productService;
    
    public ProductsModel(IProductService productService)
    {
        _productService = productService;
    }
    
    public List<ProductViewModel> Products { get; set; } = new();
    
    public async Task<IActionResult> OnGetAsync()
    {
        var products = await _productService.GetAllAsync();
        
        Products = products.Select(p => new ProductViewModel
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price
        }).ToList();
        
        return Page();
    }
    
    public async Task<IActionResult> OnPostDeleteAsync(int id)
    {
        await _productService.DeleteAsync(id);
        return RedirectToPage();
    }
}
```

### Named Handler Metotları

Aynı HTTP metodu için birden fazla handler tanımlanabilir:

```cshtml
<form method="post" asp-page-handler="Save">
    <!-- Form alanları -->
    <button type="submit">Kaydet</button>
</form>

<form method="post" asp-page-handler="Delete">
    <input type="hidden" name="id" value="@Model.Product.Id" />
    <button type="submit">Sil</button>
</form>
```

```csharp
public class EditModel : PageModel
{
    private readonly IProductService _productService;
    
    public EditModel(IProductService productService)
    {
        _productService = productService;
    }
    
    [BindProperty]
    public ProductViewModel Product { get; set; } = default!;
    
    public async Task<IActionResult> OnGetAsync(int id)
    {
        var product = await _productService.GetByIdAsync(id);
        
        if (product == null)
        {
            return NotFound();
        }
        
        Product = new ProductViewModel
        {
            Id = product.Id,
            Name = product.Name,
            Price = product.Price
        };
        
        return Page();
    }
    
    public async Task<IActionResult> OnPostSaveAsync()
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }
        
        await _productService.UpdateAsync(Product);
        
        return RedirectToPage("Index");
    }
    
    public async Task<IActionResult> OnPostDeleteAsync(int id)
    {
        await _productService.DeleteAsync(id);
        
        return RedirectToPage("Index");
    }
}
```

### Handler Metotlarında Model Binding

```csharp
public class CreateModel : PageModel
{
    private readonly IProductService _productService;
    
    public CreateModel(IProductService productService)
    {
        _productService = productService;
    }
    
    // [BindProperty] ile işaretlenen property'ler POST isteklerinde otomatik olarak doldurulur
    [BindProperty]
    public ProductViewModel Product { get; set; } = default!;
    
    // [BindProperty(SupportsGet = true)] ile GET isteklerinde de doldurulabilir
    [BindProperty(SupportsGet = true)]
    public string Category { get; set; } = "";
    
    public void OnGet()
    {
        // Category property'si query string'den otomatik olarak doldurulur
        // örnek: /Create?category=Electronics
    }
    
    public async Task<IActionResult> OnPostAsync()
    {
        if (!ModelState.IsValid)
        {
            return Page();
        }
        
        await _productService.CreateAsync(Product);
        
        return RedirectToPage("Index");
    }
}
```

## Partial Views

Partial view'lar, Razor Pages'te tekrar kullanılabilir UI bileşenleri oluşturmak için kullanılır. Bunlar, sayfanın belirli bölümlerini modüler hale getirmenizi sağlar.

### Partial View Oluşturma

Partial view (_ProductCard.cshtml):

```cshtml
@model ProductViewModel

<div class="card">
    <div class="card-header">
        <h3>@Model.Name</h3>
    </div>
    <div class="card-body">
        <p>Fiyat: @Model.Price.ToString("C")</p>
    </div>
    <div class="card-footer">
        <a asp-page="Detail" asp-route-id="@Model.Id" class="btn btn-primary">Detaylar</a>
    </div>
</div>
```

### Partial View Kullanımı

```cshtml
@page
@model ProductsModel
@{
    ViewData["Title"] = "Ürünler";
}

<h1>Ürünler</h1>

<div class="row">
    @foreach (var product in Model.Products)
    {
        <div class="col-md-4">
            <partial name="_ProductCard" model="product" />
        </div>
    }
</div>
```

### Partial Tag Helper

```cshtml
<!-- Model ile kullanım -->
<partial name="_ProductCard" model="product" />

<!-- For ile kullanım (model koleksiyonu için) -->
<partial name="_ProductCard" for="Products[0]" />

<!-- View data ile kullanım -->
<partial name="_Notification" view-data="ViewData" />
```

### Farklı Klasördeki Partial View

```cshtml
<!-- Tam yol belirterek -->
<partial name="/Pages/Shared/_ProductCard.cshtml" model="product" />

<!-- Göreceli yol belirterek -->
<partial name="../Shared/_ProductCard" model="product" />
```

### Dinamik Partial View

```cshtml
@{
    var partialName = Model.IsAdmin ? "_AdminActions" : "_UserActions";
}

<partial name="@partialName" />
```

## View Components

View Component'ler, partial view'lardan daha güçlü ve bağımsız bileşenlerdir. Kendi mantıklarını içerebilir ve sayfa akışından bağımsız olarak çalışabilirler.

### View Component Sınıfı

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
        var userId = HttpContext.User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
        
        if (string.IsNullOrEmpty(userId))
        {
            return View("Empty");
        }
        
        var cartItems = await _cartService.GetCartItemsAsync(userId);
        
        return View(cartItems);
    }
}
```

### View Component View'ı

View component view'ları, `Pages/Shared/Components/{ViewComponentName}/{ViewName}.cshtml` yolunda bulunur.

Default view (Pages/Shared/Components/CartSummary/Default.cshtml):

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
        <a class="dropdown-item" asp-page="/Cart/Index">Sepete Git</a>
    </div>
</div>
```

Empty view (Pages/Shared/Components/CartSummary/Empty.cshtml):

```cshtml
<a asp-page="/Cart/Index" class="btn btn-outline-secondary">
    Sepet (0)
</a>
```

### View Component Kullanımı

```cshtml
<!-- Tag helper ile kullanım -->
<vc:cart-summary></vc:cart-summary>

<!-- Invoke ile kullanım -->
@await Component.InvokeAsync("CartSummary")

<!-- Parametreli kullanım -->
<vc:product-list category="Electronics" count="5"></vc:product-list>
```

### Parametreli View Component

```csharp
public class ProductListViewComponent : ViewComponent
{
    private readonly IProductService _productService;
    
    public ProductListViewComponent(IProductService productService)
    {
        _productService = productService;
    }
    
    public async Task<IViewComponentResult> InvokeAsync(string category, int count = 10)
    {
        var products = await _productService.GetByCategoryAsync(category, count);
        
        return View(products);
    }
}
```

```cshtml
<!-- Parametreli kullanım -->
<vc:product-list category="Electronics" count="5"></vc:product-list>
```

### View Component'te Farklı View Döndürme

```csharp
public async Task<IViewComponentResult> InvokeAsync(string viewType)
{
    var products = await _productService.GetFeaturedProductsAsync();
    
    // viewType parametresine göre farklı view döndür
    return View(viewType, products);
}
```

```cshtml
<!-- "Grid" view'ını kullanarak çağırma -->
<vc:featured-products view-type="Grid"></vc:featured-products>

<!-- "List" view'ını kullanarak çağırma -->
<vc:featured-products view-type="List"></vc:featured-products>
```

## Özet

Razor Pages, ASP.NET Core'da sayfa tabanlı web uygulamaları geliştirmek için kullanılan bir programlama modelidir. Page Model yapısı, her sayfanın kendi model sınıfı ile eşleştirilmesini sağlar ve bu model sınıfı, sayfanın davranışını ve veri işleme mantığını içerir.

Routing, dosya sistemi tabanlıdır ve URL yolları dosya yollarına karşılık gelir. Handler metotları, HTTP isteklerini işlemek için kullanılır ve sayfa yaşam döngüsünü yönetir. Partial view'lar, tekrar kullanılabilir UI bileşenleri oluşturmak için kullanılırken, view component'ler daha güçlü ve bağımsız bileşenlerdir.

Razor Pages, özellikle form tabanlı CRUD işlemleri için MVC'ye göre daha basit bir yapı sunar ve daha az kod yazmanızı sağlar. Bu nedenle, küçük ve orta ölçekli web uygulamaları için ideal bir seçimdir. 