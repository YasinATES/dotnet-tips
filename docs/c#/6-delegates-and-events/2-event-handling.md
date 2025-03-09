# Event Handling

Bu bölümde, C#'ın olay (event) mekanizmasını inceleyeceğiz. Olaylar, delegelerin özel bir kullanımıdır ve nesne yönelimli programlamada önemli bir rol oynar. Olaylar, bir nesnenin durumundaki değişiklikleri diğer nesnelere bildirmek için kullanılır.

## 1. Event Declaration

C#'ta olaylar, `event` anahtar kelimesi kullanılarak tanımlanır. Olaylar, delege türünde olmalıdır ve genellikle `EventHandler` veya `EventHandler<TEventArgs>` delegeleri kullanılır.

### Temel Olay Tanımlama

```csharp
public class Button
{
    // Standart olay tanımlama
    public event EventHandler Click;
    
    // Olay tetikleme metodu
    protected virtual void OnClick(EventArgs e)
    {
        // Olay null kontrolü ve çağrısı
        Click?.Invoke(this, e);
    }
    
    // Kullanıcı etkileşimini simüle eden metot
    public void PerformClick()
    {
        Console.WriteLine("Düğmeye tıklandı");
        OnClick(EventArgs.Empty);
    }
}

public class Program
{
    public static void Main()
    {
        Button button = new Button();
        
        // Olaya abone olma
        button.Click += Button_Click;
        
        // Olayı tetikleme
        button.PerformClick();
    }
    
    private static void Button_Click(object sender, EventArgs e)
    {
        Console.WriteLine("Düğme tıklama olayı işlendi");
    }
}
```

### Özel Olay Argümanları ile Olay Tanımlama

```csharp
// Özel olay argümanları sınıfı
public class ProductEventArgs : EventArgs
{
    public string ProductName { get; }
    public decimal Price { get; }
    
    public ProductEventArgs(string productName, decimal price)
    {
        ProductName = productName;
        Price = price;
    }
}

public class ShoppingCart
{
    // Özel olay argümanları ile olay tanımlama
    public event EventHandler<ProductEventArgs> ProductAdded;
    
    private List<string> _products = new List<string>();
    private Dictionary<string, decimal> _prices = new Dictionary<string, decimal>();
    
    // Olay tetikleme metodu
    protected virtual void OnProductAdded(ProductEventArgs e)
    {
        ProductAdded?.Invoke(this, e);
    }
    
    // İş mantığı metodu
    public void AddProduct(string productName, decimal price)
    {
        _products.Add(productName);
        _prices[productName] = price;
        
        Console.WriteLine($"Ürün eklendi: {productName}, Fiyat: {price:C}");
        OnProductAdded(new ProductEventArgs(productName, price));
    }
}
```

## 2. Event Subscription

Olaylara abone olmak, bir olayın tetiklendiğinde çalıştırılacak metotları belirtmek anlamına gelir. C#'ta olaylara abone olmak için `+=` operatörü kullanılır.

### Olay Aboneliği Yöntemleri

```csharp
public class SubscriptionExample
{
    public static void Main()
    {
        ShoppingCart cart = new ShoppingCart();
        
        // 1. İsimli metot ile abonelik
        cart.ProductAdded += HandleProductAdded;
        
        // 2. Anonim metot ile abonelik
        cart.ProductAdded += delegate(object sender, ProductEventArgs e)
        {
            Console.WriteLine($"Anonim metot: {e.ProductName} eklendi, fiyat: {e.Price:C}");
        };
        
        // 3. Lambda ifadesi ile abonelik
        cart.ProductAdded += (sender, e) =>
        {
            Console.WriteLine($"Lambda: {e.ProductName} eklendi, fiyat: {e.Price:C}");
        };
        
        // Ürün ekleme
        cart.AddProduct("Laptop", 5000m);
    }
    
    // İsimli olay işleyici metot
    private static void HandleProductAdded(object sender, ProductEventArgs e)
    {
        Console.WriteLine($"İsimli metot: {e.ProductName} eklendi, fiyat: {e.Price:C}");
    }
}
```

## 3. Event Unsubscription

Olaylardan aboneliği kaldırmak, bellek sızıntılarını önlemek ve gereksiz işlemleri engellemek için önemlidir. C#'ta olay aboneliğini kaldırmak için `-=` operatörü kullanılır.

```csharp
public class UnsubscriptionExample
{
    public static void Main()
    {
        Button button = new Button();
        
        // Olaya abone olma
        EventHandler handler = (sender, e) => Console.WriteLine("Düğme tıklandı!");
        button.Click += handler;
        
        // Olayı tetikleme
        button.PerformClick(); // "Düğme tıklandı!" yazdırır
        
        // Aboneliği kaldırma
        button.Click -= handler;
        
        // Olayı tekrar tetikleme
        button.PerformClick(); // Artık hiçbir şey yazdırmaz
    }
}
```

### Abonelik Yönetimi

```csharp
public class Subscriber : IDisposable
{
    private readonly Button _button;
    
    public Subscriber(Button button)
    {
        _button = button;
        _button.Click += HandleButtonClick;
    }
    
    private void HandleButtonClick(object sender, EventArgs e)
    {
        Console.WriteLine("Abone: Düğme tıklandı!");
    }
    
    public void Dispose()
    {
        // Aboneliği kaldır
        _button.Click -= HandleButtonClick;
    }
}

public class SubscriptionManagementExample
{
    public static void Main()
    {
        Button button = new Button();
        
        // using bloğu ile otomatik abonelik kaldırma
        using (Subscriber subscriber = new Subscriber(button))
        {
            button.PerformClick(); // "Abone: Düğme tıklandı!" yazdırır
        } // Dispose çağrılır, abonelik kaldırılır
        
        button.PerformClick(); // Artık hiçbir şey yazdırmaz
    }
}
```

## 4. Event Arguments

Olay argümanları, olay hakkında ek bilgi sağlamak için kullanılır. C#'ta tüm olay argümanları `EventArgs` sınıfından türetilmelidir.

### Standart EventArgs

```csharp
public class StandardEventArgsExample
{
    public static void Main()
    {
        Button button = new Button();
        
        button.Click += (sender, e) =>
        {
            // sender: olayı tetikleyen nesne
            Button sourceButton = (Button)sender;
            
            // e: olay argümanları (bu durumda boş)
            // EventArgs.Empty kullanılır
            Console.WriteLine("Düğme tıklandı!");
        };
        
        button.PerformClick();
    }
}
```

### Özel EventArgs

```csharp
public class MouseEventArgs : EventArgs
{
    public int X { get; }
    public int Y { get; }
    public MouseButton Button { get; }
    public bool Handled { get; set; }
    
    public MouseEventArgs(int x, int y, MouseButton button)
    {
        X = x;
        Y = y;
        Button = button;
        Handled = false;
    }
}

public enum MouseButton
{
    Left,
    Middle,
    Right
}

public class AdvancedButton
{
    public event EventHandler<MouseEventArgs> Click;
    
    protected virtual void OnClick(MouseEventArgs e)
    {
        Click?.Invoke(this, e);
        
        if (!e.Handled)
        {
            Console.WriteLine("Tıklama olayı işlenmedi");
        }
    }
    
    public void SimulateClick(int x, int y, MouseButton button)
    {
        Console.WriteLine($"{button} düğmesi ile ({x},{y}) konumuna tıklandı");
        OnClick(new MouseEventArgs(x, y, button));
    }
}
```

## 5. Event Threading Considerations

Çok iş parçacıklı uygulamalarda, olayların güvenli bir şekilde tetiklenmesi ve işlenmesi önemlidir.

### Thread-Safe Olay Tetikleme

```csharp
public class ThreadSafePublisher
{
    private readonly object _lockObject = new object();
    
    // Olay tanımlama
    public event EventHandler<string> MessageReceived;
    
    // Thread-safe olay tetikleme
    protected virtual void OnMessageReceived(string message)
    {
        EventHandler<string> handler;
        
        // Olay işleyicisinin yerel bir kopyasını al
        lock (_lockObject)
        {
            handler = MessageReceived;
        }
        
        // Null kontrolü yap ve çağır
        handler?.Invoke(this, message);
    }
    
    public void SendMessage(string message)
    {
        Console.WriteLine($"Thread {Thread.CurrentThread.ManagedThreadId}: Mesaj gönderiliyor: {message}");
        OnMessageReceived(message);
    }
}
```

## 6. Weak Event Patterns

Zayıf olay desenleri, olay abonelikleri nedeniyle oluşabilecek bellek sızıntılarını önlemek için kullanılır.

### WeakEventManager Örneği

```csharp
public class WeakEventSubscriber
{
    private readonly WeakReference<EventHandler<EventArgs>> _weakHandler;
    private readonly Button _source;
    
    public WeakEventSubscriber(Button source, EventHandler<EventArgs> handler)
    {
        _weakHandler = new WeakReference<EventHandler<EventArgs>>(handler);
        _source = source;
        
        // Proxy işleyici ile abone ol
        _source.Click += ProxyHandler;
    }
    
    private void ProxyHandler(object sender, EventArgs e)
    {
        // Zayıf referanstan gerçek işleyiciyi al
        if (_weakHandler.TryGetTarget(out var handler))
        {
            // Gerçek işleyiciyi çağır
            handler(sender, e);
        }
        else
        {
            // İşleyici GC tarafından toplandı, aboneliği kaldır
            _source.Click -= ProxyHandler;
        }
    }
}
```

Olaylar, C#'ta nesneler arasında gevşek bağlı iletişim sağlayan güçlü bir mekanizmadır. Doğru şekilde kullanıldığında, olaylar modüler ve bakımı kolay kod oluşturmanıza yardımcı olur. 