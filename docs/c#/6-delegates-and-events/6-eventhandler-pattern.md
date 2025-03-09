# EventHandler Pattern

Bu bölümde, C#'ta yaygın olarak kullanılan EventHandler desenini inceleyeceğiz. EventHandler deseni, .NET Framework'te olayları tutarlı bir şekilde tanımlamak ve kullanmak için standart bir yaklaşım sunar.

## 1. Standard Event Pattern

.NET'te standart olay deseni, `EventHandler` ve `EventHandler<TEventArgs>` delegelerini kullanır. Bu desen, olay yayıncısı (publisher) ve olay abonesi (subscriber) arasında tutarlı bir iletişim sağlar.

```csharp
public class Button
{
    // EventHandler delegesi ile olay tanımlama
    public event EventHandler Click;
    
    // Olayı tetikleme metodu
    protected virtual void OnClick(EventArgs e)
    {
        Click?.Invoke(this, e);
    }
    
    public void PerformClick()
    {
        Console.WriteLine("Düğmeye tıklandı");
        OnClick(EventArgs.Empty);
    }
}

// Kullanım
Button button = new Button();
button.Click += (sender, e) => Console.WriteLine("Düğme tıklama olayı işlendi");
button.PerformClick();
```

## 2. Custom EventArgs

Özel olay argümanları, olaylar hakkında daha fazla bilgi taşımak için kullanılır. `EventArgs` sınıfından türetilen özel sınıflar, olay verilerini taşımak için standart bir yol sağlar.

```csharp
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
    // Özel EventArgs ile olay tanımlama
    public event EventHandler<ProductEventArgs> ProductAdded;
    
    protected virtual void OnProductAdded(ProductEventArgs e)
    {
        ProductAdded?.Invoke(this, e);
    }
    
    public void AddProduct(string productName, decimal price)
    {
        Console.WriteLine($"Ürün eklendi: {productName}, Fiyat: {price:C}");
        OnProductAdded(new ProductEventArgs(productName, price));
    }
}
```

## 3. Event Accessors

Olay erişimcileri (event accessors), olayların nasıl abone olunacağını ve abonelikten çıkılacağını özelleştirmenize olanak tanır. Bu, özel olay davranışları uygulamak için kullanılabilir.

```csharp
public class CustomEventAccessorsExample
{
    private EventHandler _click;
    
    // Özel olay erişimcileri
    public event EventHandler Click
    {
        add
        {
            Console.WriteLine("Click olayına abone olundu");
            _click += value;
        }
        remove
        {
            Console.WriteLine("Click olayından abonelik kaldırıldı");
            _click -= value;
        }
    }
    
    protected virtual void OnClick(EventArgs e)
    {
        _click?.Invoke(this, e);
    }
    
    public void PerformClick()
    {
        Console.WriteLine("Düğmeye tıklandı");
        OnClick(EventArgs.Empty);
    }
}
```

## 4. Thread-Safe Events

Çok iş parçacıklı uygulamalarda, olayların güvenli bir şekilde tetiklenmesi önemlidir. Thread-safe olaylar, eşzamanlı erişim sorunlarını önler.

```csharp
public class ThreadSafeEventExample
{
    private readonly object _lockObject = new object();
    private EventHandler _dataReceived;
    
    // Thread-safe olay
    public event EventHandler DataReceived
    {
        add
        {
            lock (_lockObject)
            {
                _dataReceived += value;
            }
        }
        remove
        {
            lock (_lockObject)
            {
                _dataReceived -= value;
            }
        }
    }
    
    protected virtual void OnDataReceived(EventArgs e)
    {
        EventHandler handler;
        
        // Olay işleyicisinin yerel bir kopyasını al
        lock (_lockObject)
        {
            handler = _dataReceived;
        }
        
        // Null kontrolü yap ve çağır
        handler?.Invoke(this, e);
    }
}
```

## 5. Event Design Guidelines

Olay tasarımında izlenmesi gereken bazı önemli yönergeler vardır:

1. **Standart İsimlendirme**: Olay isimleri fiil veya fiil+isim şeklinde olmalıdır (örn. `Click`, `PropertyChanged`).
2. **EventArgs Kullanımı**: Özel veri için `EventArgs`'tan türetilen sınıflar kullanın.
3. **Olay Tetikleme Metotları**: `OnEventName` şeklinde isimlendirin ve `protected virtual` olarak tanımlayın.
4. **Null Kontrolü**: Olay tetiklemeden önce null kontrolü yapın (`event?.Invoke(this, e)`).
5. **İptal Edilebilir Olaylar**: İptal edilebilir olaylar için `CancelEventArgs` kullanın.

```csharp
public class FileOperations
{
    // İptal edilebilir olay
    public event EventHandler<CancelEventArgs> FileDeleting;
    
    protected virtual void OnFileDeleting(CancelEventArgs e)
    {
        FileDeleting?.Invoke(this, e);
    }
    
    public bool DeleteFile(string path)
    {
        CancelEventArgs args = new CancelEventArgs();
        OnFileDeleting(args);
        
        if (args.Cancel)
        {
            return false;
        }
        
        // Dosya silme işlemi...
        return true;
    }
}
```

## 6. Event Performance

Olayların performansını optimize etmek için bazı teknikler:

1. **Null Kontrolü Optimizasyonu**: `?.` operatörünü kullanın.
2. **Olay Aboneliklerini Yönetin**: Abonelikleri takip edin ve gerektiğinde kaldırın.
3. **Gereksiz Olay Tetiklemelerinden Kaçının**: Sadece gerektiğinde olayları tetikleyin.
4. **Yerel Kopya Kullanımı**: Thread-safe olaylar için yerel kopya kullanın.

```csharp
// Performans optimizasyonu örneği
public event EventHandler ValueChanged;

private int _value;
public int Value
{
    get => _value;
    set
    {
        if (_value != value) // Sadece değer değiştiğinde tetikle
        {
            _value = value;
            ValueChanged?.Invoke(this, EventArgs.Empty); // ?. operatörü ile kısa null kontrolü
        }
    }
}
```

EventHandler deseni, C#'ta olayları tutarlı ve güvenli bir şekilde kullanmanın standart yoludur. Bu deseni doğru şekilde uygulamak, kodunuzu daha okunabilir, bakımı daha kolay ve daha güvenli hale getirir. 