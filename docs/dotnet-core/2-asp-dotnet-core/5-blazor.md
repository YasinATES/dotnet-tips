# Blazor

Blazor, C# ve .NET kullanarak etkileşimli web uygulamaları geliştirmenize olanak tanıyan bir web çerçevesidir. JavaScript yerine C# kullanarak hem sunucu tarafında hem de istemci tarafında çalışabilen uygulamalar oluşturabilirsiniz.

## Server-side Blazor

Server-side Blazor (Blazor Server), UI güncellemelerini ve olay işlemlerini gerçek zamanlı olarak sunucu üzerinden gerçekleştiren bir hosting modelidir. SignalR bağlantısı üzerinden istemci ve sunucu arasında iletişim sağlanır.

### Temel Yapılandırma

Program.cs dosyasında Blazor Server yapılandırması:

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container
builder.Services.AddRazorPages();
builder.Services.AddServerSideBlazor();
builder.Services.AddSingleton<WeatherForecastService>();

var app = builder.Build();

// Configure the HTTP request pipeline
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.MapBlazorHub();
app.MapFallbackToPage("/_Host");

app.Run();
```

### _Host.cshtml

Blazor Server uygulamasının başlangıç noktası olan _Host.cshtml sayfası:

```cshtml
@page "/"
@namespace BlazorApp.Pages
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@{
    Layout = "_Layout";
}

<component type="typeof(App)" render-mode="ServerPrerendered" />

<script src="_framework/blazor.server.js"></script>
```

### Avantajları

- Daha küçük indirme boyutu
- Anında başlatma
- Sunucu kaynaklarına doğrudan erişim
- Daha geniş tarayıcı desteği

### Dezavantajları

- Sürekli sunucu bağlantısı gerektirir
- Daha yüksek gecikme süresi
- Ölçeklenebilirlik zorlukları
- Çevrimdışı çalışma desteği yok

## WebAssembly Blazor

WebAssembly Blazor (Blazor WASM), .NET uygulamasını WebAssembly formatında tarayıcıya indirerek istemci tarafında çalıştıran bir hosting modelidir. Uygulama, tarayıcıda doğrudan çalışır ve sunucuya sadece veri erişimi için bağlanır.

### Temel Yapılandırma

Program.cs dosyasında Blazor WASM yapılandırması:

```csharp
var builder = WebAssemblyHostBuilder.CreateDefault(args);
builder.RootComponents.Add<App>("#app");
builder.RootComponents.Add<HeadOutlet>("head::after");

// Add services to the container
builder.Services.AddScoped(sp => new HttpClient { BaseAddress = new Uri(builder.HostEnvironment.BaseAddress) });
builder.Services.AddScoped<IWeatherForecastService, WeatherForecastService>();

await builder.Build().RunAsync();
```

### index.html

Blazor WASM uygulamasının başlangıç noktası olan index.html sayfası:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Blazor WASM App</title>
    <base href="/" />
    <link href="css/bootstrap/bootstrap.min.css" rel="stylesheet" />
    <link href="css/app.css" rel="stylesheet" />
    <link href="BlazorApp.styles.css" rel="stylesheet" />
</head>
<body>
    <div id="app">
        <div class="loading">Uygulama yükleniyor...</div>
    </div>

    <script src="_framework/blazor.webassembly.js"></script>
</body>
</html>
```

### Avantajları

- Sunucu bağlantısı gerektirmez
- Düşük gecikme süresi
- Ölçeklenebilirlik
- Çevrimdışı çalışma desteği

### Dezavantajları

- Daha büyük indirme boyutu
- Başlatma gecikmesi
- Sınırlı tarayıcı desteği
- Sınırlı donanım kaynakları

## Component Yapısı

Blazor, bileşen tabanlı bir mimari kullanır. Her bileşen, kullanıcı arayüzünün bir parçasını temsil eder ve kendi mantığını içerebilir.

### Temel Bileşen Yapısı

```razor
@page "/counter"

<PageTitle>Sayaç</PageTitle>

<h1>Sayaç</h1>

<p>Mevcut sayı: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Artır</button>

@code {
    private int currentCount = 0;

    // Increment the counter when button is clicked
    private void IncrementCount()
    {
        currentCount++;
    }
}
```

### Bileşen Parametreleri

Bileşenler, parametreler aracılığıyla dışarıdan veri alabilir:

```razor
<h3>Ürün Kartı</h3>

<div class="card">
    <div class="card-header">
        <h4>@Title</h4>
    </div>
    <div class="card-body">
        <p>@Description</p>
        <p>Fiyat: @Price.ToString("C")</p>
    </div>
    <div class="card-footer">
        <button class="btn btn-primary" @onclick="OnAddToCart">Sepete Ekle</button>
    </div>
</div>

@code {
    // Parameters passed from parent component
    [Parameter]
    public string Title { get; set; } = "";

    [Parameter]
    public string Description { get; set; } = "";

    [Parameter]
    public decimal Price { get; set; }

    [Parameter]
    public EventCallback<string> OnAddToCartCallback { get; set; }

    // Method to handle button click
    private async Task OnAddToCart()
    {
        await OnAddToCartCallback.InvokeAsync(Title);
    }
}
```

### Bileşen Kullanımı

Bileşenler, diğer bileşenler içinde kullanılabilir:

```razor
@page "/products"

<h1>Ürünler</h1>

<div class="row">
    @foreach (var product in products)
    {
        <div class="col-md-4 mb-4">
            <ProductCard 
                Title="@product.Name"
                Description="@product.Description"
                Price="@product.Price"
                OnAddToCartCallback="AddToCart" />
        </div>
    }
</div>

<div class="mt-4">
    <h3>Sepet</h3>
    <ul class="list-group">
        @foreach (var item in cartItems)
        {
            <li class="list-group-item">@item</li>
        }
    </ul>
</div>

@code {
    private List<Product> products = new List<Product>
    {
        new Product { Id = 1, Name = "Laptop", Description = "Güçlü işlemci", Price = 5000 },
        new Product { Id = 2, Name = "Telefon", Description = "Yüksek çözünürlük", Price = 3000 },
        new Product { Id = 3, Name = "Tablet", Description = "İnce tasarım", Price = 2000 }
    };

    private List<string> cartItems = new List<string>();

    // Add product to cart
    private void AddToCart(string productName)
    {
        cartItems.Add(productName);
    }

    private class Product
    {
        public int Id { get; set; }
        public string Name { get; set; } = "";
        public string Description { get; set; } = "";
        public decimal Price { get; set; }
    }
}
```

### Yaşam Döngüsü Metotları

Blazor bileşenleri, çeşitli yaşam döngüsü olaylarına yanıt verebilir:

```razor
@page "/lifecycle"
@implements IDisposable

<h1>Yaşam Döngüsü Örneği</h1>

<p>Mevcut sayı: @currentCount</p>
<button class="btn btn-primary" @onclick="IncrementCount">Artır</button>

@code {
    private int currentCount = 0;
    private System.Threading.Timer? timer;

    // Called when the component is initialized
    protected override void OnInitialized()
    {
        Console.WriteLine("OnInitialized çağrıldı");
    }

    // Called after the component is initialized and parameters are set
    protected override async Task OnInitializedAsync()
    {
        Console.WriteLine("OnInitializedAsync çağrıldı");
        await Task.Delay(1000);
    }

    // Called when parameters are set
    protected override void OnParametersSet()
    {
        Console.WriteLine("OnParametersSet çağrıldı");
    }

    // Called after parameters are set
    protected override async Task OnParametersSetAsync()
    {
        Console.WriteLine("OnParametersSetAsync çağrıldı");
        await Task.Delay(1000);
    }

    // Called after the component is rendered
    protected override void OnAfterRender(bool firstRender)
    {
        Console.WriteLine($"OnAfterRender çağrıldı, firstRender: {firstRender}");
        
        if (firstRender)
        {
            // Initialize timer on first render
            timer = new System.Threading.Timer(_ =>
            {
                IncrementCount();
                StateHasChanged();
            }, null, 5000, 5000);
        }
    }

    // Called after the component is rendered asynchronously
    protected override async Task OnAfterRenderAsync(bool firstRender)
    {
        Console.WriteLine($"OnAfterRenderAsync çağrıldı, firstRender: {firstRender}");
        await Task.Delay(1000);
    }

    // Increment the counter
    private void IncrementCount()
    {
        currentCount++;
    }

    // Called when the component is disposed
    public void Dispose()
    {
        Console.WriteLine("Dispose çağrıldı");
        timer?.Dispose();
    }
}
```

## State Management

Blazor uygulamalarında durum yönetimi, uygulama verilerinin nasıl saklanacağını ve bileşenler arasında nasıl paylaşılacağını belirler.

### Bileşen Durumu

En basit durum yönetimi, bileşen içinde özel alanlar veya özellikler kullanmaktır:

```razor
@page "/counter"

<h1>Sayaç</h1>

<p>Mevcut sayı: @currentCount</p>

<button class="btn btn-primary" @onclick="IncrementCount">Artır</button>

@code {
    // Component state
    private int currentCount = 0;

    private void IncrementCount()
    {
        currentCount++;
    }
}
```

### Bileşenler Arası Durum Paylaşımı

#### Parametre Geçişi

Üst bileşenden alt bileşene parametre geçişi:

```razor
<!-- ParentComponent.razor -->
<h1>Üst Bileşen</h1>

<ChildComponent Count="@currentCount" OnCountChanged="@UpdateCount" />

<button class="btn btn-secondary" @onclick="IncrementCount">Üst Bileşenden Artır</button>

@code {
    private int currentCount = 0;

    private void IncrementCount()
    {
        currentCount++;
    }

    private void UpdateCount(int newCount)
    {
        currentCount = newCount;
    }
}

<!-- ChildComponent.razor -->
<div class="card">
    <div class="card-body">
        <h3>Alt Bileşen</h3>
        <p>Sayı: @Count</p>
        <button class="btn btn-primary" @onclick="IncrementCount">Alt Bileşenden Artır</button>
    </div>
</div>

@code {
    [Parameter]
    public int Count { get; set; }

    [Parameter]
    public EventCallback<int> OnCountChanged { get; set; }

    private async Task IncrementCount()
    {
        await OnCountChanged.InvokeAsync(Count + 1);
    }
}
```

#### Cascading Values

Cascading values, bileşen ağacında derinlere veri iletmek için kullanılır:

```razor
<!-- MainLayout.razor -->
<CascadingValue Value="@theme">
    <div class="@theme">
        @Body
    </div>
</CascadingValue>

<div class="theme-selector">
    <button class="btn btn-sm btn-outline-primary" @onclick="() => theme = 'light-theme'">Açık Tema</button>
    <button class="btn btn-sm btn-outline-dark" @onclick="() => theme = 'dark-theme'">Koyu Tema</button>
</div>

@code {
    private string theme = "light-theme";
}

<!-- DeepChildComponent.razor -->
<div class="card">
    <div class="card-body">
        <h4>Derin Alt Bileşen</h4>
        <p>Mevcut tema: @Theme</p>
    </div>
</div>

@code {
    [CascadingParameter]
    public string Theme { get; set; } = "";
}
```

### Servis Tabanlı Durum Yönetimi

Bileşenler arasında durum paylaşımı için servisler kullanılabilir:

```csharp
// CounterState.cs
public class CounterState
{
    private int count = 0;
    
    public int Count => count;
    
    public event Action? OnChange;
    
    public void IncrementCount()
    {
        count++;
        NotifyStateChanged();
    }
    
    private void NotifyStateChanged() => OnChange?.Invoke();
}
```

Servis kaydı:

```csharp
// Program.cs
builder.Services.AddSingleton<CounterState>();
```

Servis kullanımı:

```razor
@page "/counter-1"
@inject CounterState CounterState
@implements IDisposable

<h1>Sayaç 1</h1>

<p>Mevcut sayı: @CounterState.Count</p>

<button class="btn btn-primary" @onclick="IncrementCount">Artır</button>

@code {
    protected override void OnInitialized()
    {
        // Subscribe to state changes
        CounterState.OnChange += StateHasChanged;
    }

    private void IncrementCount()
    {
        CounterState.IncrementCount();
    }

    public void Dispose()
    {
        // Unsubscribe from state changes
        CounterState.OnChange -= StateHasChanged;
    }
}
```

### Blazor Server için Oturum Durumu

Blazor Server uygulamalarında, CircuitHandler kullanarak oturum durumunu yönetebilirsiniz:

```csharp
// CircuitHandlerService.cs
public class CircuitHandlerService : CircuitHandler
{
    private readonly ILogger<CircuitHandlerService> _logger;
    private readonly UserState _userState;

    public CircuitHandlerService(ILogger<CircuitHandlerService> logger, UserState userState)
    {
        _logger = logger;
        _userState = userState;
    }

    public override Task OnConnectionUpAsync(Circuit circuit, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Circuit {CircuitId} connected at {Time}", 
            circuit.Id, DateTime.UtcNow);
        
        _userState.IsConnected = true;
        return Task.CompletedTask;
    }

    public override Task OnConnectionDownAsync(Circuit circuit, CancellationToken cancellationToken)
    {
        _logger.LogInformation("Circuit {CircuitId} disconnected at {Time}", 
            circuit.Id, DateTime.UtcNow);
        
        _userState.IsConnected = false;
        return Task.CompletedTask;
    }
}

// UserState.cs
public class UserState
{
    public bool IsConnected { get; set; }
    public string Username { get; set; } = "";
    
    public event Action? OnChange;
    
    public void SetUsername(string username)
    {
        Username = username;
        NotifyStateChanged();
    }
    
    private void NotifyStateChanged() => OnChange?.Invoke();
}
```

Servis kaydı:

```csharp
// Program.cs
builder.Services.AddSingleton<UserState>();
builder.Services.AddScoped<CircuitHandler, CircuitHandlerService>();
```

## JavaScript Interop

Blazor, JavaScript ile etkileşim kurmanıza olanak tanır. Bu, tarayıcı API'lerine erişmek veya mevcut JavaScript kütüphanelerini kullanmak için gereklidir.

### JavaScript'ten C#'a Çağrı

JavaScript dosyası (wwwroot/js/interop.js):

```javascript
// Define a namespace for our JavaScript functions
window.interopFunctions = {
    // Show a browser alert
    showAlert: function (message) {
        alert(message);
        return "Alert displayed";
    },
    
    // Get browser information
    getBrowserInfo: function () {
        return {
            userAgent: navigator.userAgent,
            language: navigator.language,
            screenWidth: window.screen.width,
            screenHeight: window.screen.height
        };
    },
    
    // Add event listener to document
    addDocumentEventListener: function (dotnetHelper) {
        function documentClickHandler() {
            // Call .NET method
            dotnetHelper.invokeMethodAsync('HandleDocumentClick');
        }
        
        // Add the event listener
        document.addEventListener('click', documentClickHandler);
        
        // Return a function to remove the event listener
        return function () {
            document.removeEventListener('click', documentClickHandler);
        };
    }
};
```

C# bileşeni:

```razor
@page "/js-interop"
@inject IJSRuntime JSRuntime
@implements IAsyncDisposable

<h1>JavaScript Interop</h1>

<div class="mb-3">
    <button class="btn btn-primary" @onclick="ShowAlert">Alert Göster</button>
</div>

<div class="mb-3">
    <button class="btn btn-info" @onclick="GetBrowserInfo">Tarayıcı Bilgisi Al</button>
    @if (browserInfo != null)
    {
        <div class="mt-2">
            <p>User Agent: @browserInfo.UserAgent</p>
            <p>Language: @browserInfo.Language</p>
            <p>Screen: @browserInfo.ScreenWidth x @browserInfo.ScreenHeight</p>
        </div>
    }
</div>

<div class="mb-3">
    <button class="btn btn-success" @onclick="SetupDocumentClickListener">Döküman Tıklama Dinleyicisi Ekle</button>
    <p>Döküman tıklamaları: @documentClicks</p>
</div>

@code {
    private BrowserInfo? browserInfo;
    private int documentClicks = 0;
    private IJSObjectReference? module;
    private DotNetObjectReference<JsInteropPage>? dotNetHelper;
    private IJSObjectReference? documentClickListenerRemover;

    protected override async Task OnInitializedAsync()
    {
        // Load the JavaScript module
        module = await JSRuntime.InvokeAsync<IJSObjectReference>("import", "./js/interop.js");
        
        // Create a reference to this component for JS to call
        dotNetHelper = DotNetObjectReference.Create(this);
    }

    private async Task ShowAlert()
    {
        if (module != null)
        {
            string result = await module.InvokeAsync<string>("interopFunctions.showAlert", "Hello from Blazor!");
            Console.WriteLine($"Result from JS: {result}");
        }
    }

    private async Task GetBrowserInfo()
    {
        if (module != null)
        {
            browserInfo = await module.InvokeAsync<BrowserInfo>("interopFunctions.getBrowserInfo");
        }
    }

    private async Task SetupDocumentClickListener()
    {
        if (module != null && dotNetHelper != null && documentClickListenerRemover == null)
        {
            documentClickListenerRemover = await module.InvokeAsync<IJSObjectReference>(
                "interopFunctions.addDocumentEventListener", dotNetHelper);
        }
    }

    // This method will be called from JavaScript
    [JSInvokable]
    public void HandleDocumentClick()
    {
        documentClicks++;
        StateHasChanged();
    }

    async ValueTask IAsyncDisposable.DisposeAsync()
    {
        // Remove the event listener
        if (documentClickListenerRemover != null)
        {
            await documentClickListenerRemover.InvokeVoidAsync("invoke");
            await documentClickListenerRemover.DisposeAsync();
        }
        
        // Dispose the .NET object reference
        dotNetHelper?.Dispose();
        
        // Dispose the JS module
        if (module != null)
        {
            await module.DisposeAsync();
        }
    }

    private class BrowserInfo
    {
        public string UserAgent { get; set; } = "";
        public string Language { get; set; } = "";
        public int ScreenWidth { get; set; }
        public int ScreenHeight { get; set; }
    }
}
```

### C#'tan JavaScript'e Çağrı

JavaScript dosyası (wwwroot/js/chart.js):

```javascript
// Chart library wrapper
window.chartInterop = {
    // Create a chart using a third-party library
    createChart: function (elementId, data) {
        // In a real app, you would use a library like Chart.js
        const element = document.getElementById(elementId);
        if (!element) return false;
        
        // For demo purposes, we'll just create a simple visualization
        element.innerHTML = '';
        
        const maxValue = Math.max(...data.values);
        
        data.values.forEach((value, index) => {
            const bar = document.createElement('div');
            bar.style.height = `${(value / maxValue) * 100}px`;
            bar.style.width = '30px';
            bar.style.backgroundColor = data.colors[index] || '#007bff';
            bar.style.display = 'inline-block';
            bar.style.margin = '0 5px';
            bar.title = `${data.labels[index]}: ${value}`;
            
            element.appendChild(bar);
        });
        
        return true;
    }
};
```

C# bileşeni:

```razor
@page "/chart"
@inject IJSRuntime JSRuntime

<h1>Chart Örneği</h1>

<div class="mb-3">
    <button class="btn btn-primary" @onclick="CreateChart">Grafik Oluştur</button>
    <button class="btn btn-secondary" @onclick="UpdateData">Verileri Güncelle</button>
</div>

<div id="chartContainer" style="height: 200px; border: 1px solid #ccc; padding: 10px;"></div>

@code {
    private ChartData chartData = new ChartData
    {
        Labels = new[] { "Ocak", "Şubat", "Mart", "Nisan", "Mayıs" },
        Values = new[] { 10, 20, 15, 25, 30 },
        Colors = new[] { "#007bff", "#28a745", "#dc3545", "#ffc107", "#17a2b8" }
    };
    
    private IJSObjectReference? module;
    
    protected override async Task OnInitializedAsync()
    {
        // Load the JavaScript module
        module = await JSRuntime.InvokeAsync<IJSObjectReference>("import", "./js/chart.js");
    }
    
    private async Task CreateChart()
    {
        if (module != null)
        {
            await module.InvokeVoidAsync("chartInterop.createChart", "chartContainer", chartData);
        }
    }
    
    private async Task UpdateData()
    {
        // Generate new random data
        var random = new Random();
        chartData.Values = Enumerable.Range(0, 5).Select(_ => random.Next(10, 50)).ToArray();
        
        await CreateChart();
    }
    
    private class ChartData
    {
        public string[] Labels { get; set; } = Array.Empty<string>();
        public int[] Values { get; set; } = Array.Empty<int>();
        public string[] Colors { get; set; } = Array.Empty<string>();
    }
}
```

## Özet

Blazor, C# ve .NET kullanarak etkileşimli web uygulamaları geliştirmenize olanak tanıyan güçlü bir web çerçevesidir. İki ana hosting modeli sunar: Server-side Blazor ve WebAssembly Blazor.

Server-side Blazor, UI güncellemelerini ve olay işlemlerini gerçek zamanlı olarak sunucu üzerinden gerçekleştirir. WebAssembly Blazor ise, .NET uygulamasını WebAssembly formatında tarayıcıya indirerek istemci tarafında çalıştırır.

Blazor, bileşen tabanlı bir mimari kullanır ve her bileşen, kullanıcı arayüzünün bir parçasını temsil eder. Durum yönetimi, bileşen durumu, bileşenler arası durum paylaşımı ve servis tabanlı durum yönetimi gibi çeşitli yöntemlerle gerçekleştirilebilir.

JavaScript Interop, Blazor uygulamalarının JavaScript ile etkileşim kurmasına olanak tanır. Bu, tarayıcı API'lerine erişmek veya mevcut JavaScript kütüphanelerini kullanmak için gereklidir.

Blazor, modern web uygulamaları geliştirmek için güçlü bir alternatif sunar ve C# geliştiricilerinin becerilerini web geliştirme alanında kullanmalarına olanak tanır. 