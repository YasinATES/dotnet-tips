# Fail Fast (Hızlı Başarısızlık)

Fail Fast prensibi, hataların mümkün olan en erken aşamada tespit edilmesi ve raporlanması gerektiğini savunan bir yazılım geliştirme yaklaşımıdır. Bu prensip, hataların gizlenmesi veya görmezden gelinmesi yerine, hemen ortaya çıkarılmasını ve ele alınmasını önerir.

## 1. Fail Fast Prensibinin Temel Kavramı

Fail Fast prensibinin özü, bir sorun tespit edildiğinde programın hemen hata vermesi ve çalışmayı durdurmasıdır. Bu yaklaşım, hataların erken aşamada tespit edilmesini sağlayarak, daha büyük sorunların ortaya çıkmasını önler.

### Fail Fast İhlali Örneği

```csharp
// Fail Fast ihlali - Hataları gizleme
public class UserService
{
    public User GetUser(int id)
    {
        try
        {
            // Veritabanından kullanıcı çekme
            var user = _repository.GetById(id);
            
            // Kullanıcı bulunamadığında null dönme (hata gizleme)
            return user;
        }
        catch (Exception ex)
        {
            // Hatayı yutma
            Console.WriteLine($"Hata oluştu: {ex.Message}");
            return null;
        }
    }
    
    public void ProcessUserData(int userId)
    {
        var user = GetUser(userId);
        
        // Null kontrolü yapılmadığı için NullReferenceException riski
        var userName = user.Name;
        var email = user.Email;
        
        // İşlem devam ediyor...
    }
}
```

Bu örnekte, `GetUser` metodu hataları yutuyor ve kullanıcı bulunamadığında null dönüyor. Bu durum, `ProcessUserData` metodunda NullReferenceException'a neden olabilir ve hatanın gerçek kaynağını bulmayı zorlaştırır.

### Fail Fast Uyumlu Tasarım

```csharp
// Fail Fast uyumlu tasarım
public class UserService
{
    public User GetUser(int id)
    {
        // Geçersiz id kontrolü
        if (id <= 0)
        {
            throw new ArgumentException("Kullanıcı ID'si pozitif olmalıdır.", nameof(id));
        }
        
        var user = _repository.GetById(id);
        
        // Kullanıcı bulunamadığında anlamlı bir exception fırlatma
        if (user == null)
        {
            throw new UserNotFoundException($"ID: {id} olan kullanıcı bulunamadı.");
        }
        
        return user;
    }
    
    public void ProcessUserData(int userId)
    {
        try
        {
            var user = GetUser(userId);
            
            // Kullanıcı null olmayacağı için güvenle devam edebiliriz
            var userName = user.Name;
            var email = user.Email;
            
            // İşlem devam ediyor...
        }
        catch (UserNotFoundException ex)
        {
            // Özel exception'ı ele alma
            _logger.LogWarning(ex.Message);
            throw; // Yeniden fırlatma
        }
    }
}

// Özel exception sınıfı
public class UserNotFoundException : Exception
{
    public UserNotFoundException(string message) : base(message) { }
}
```

Bu tasarımda, hatalar hemen tespit edilip raporlanıyor. Geçersiz parametreler ve bulunamayan kullanıcılar için anlamlı exception'lar fırlatılıyor, böylece sorunlar erken aşamada ele alınabiliyor.

## 2. Fail Fast'in Uygulama Alanları

### 1. Giriş Parametrelerinin Doğrulanması

```csharp
public void ProcessOrder(Order order, Customer customer, PaymentMethod paymentMethod)
{
    // Parametrelerin null kontrolü
    if (order == null)
        throw new ArgumentNullException(nameof(order));
    
    if (customer == null)
        throw new ArgumentNullException(nameof(customer));
    
    if (paymentMethod == null)
        throw new ArgumentNullException(nameof(paymentMethod));
    
    // Değer kontrolü
    if (order.Items.Count == 0)
        throw new ArgumentException("Sipariş en az bir ürün içermelidir.", nameof(order));
    
    if (order.TotalAmount <= 0)
        throw new ArgumentException("Sipariş tutarı pozitif olmalıdır.", nameof(order));
    
    // İşlem devam ediyor...
}
```

### 2. Nesne Durumunun Doğrulanması

```csharp
public class ShoppingCart
{
    private readonly List<Item> _items = new List<Item>();
    private bool _isCheckedOut;
    
    public void AddItem(Item item)
    {
        // Nesne durumunun kontrolü
        if (_isCheckedOut)
            throw new InvalidOperationException("Ödeme yapılmış sepete ürün eklenemez.");
        
        // Parametre kontrolü
        if (item == null)
            throw new ArgumentNullException(nameof(item));
        
        _items.Add(item);
    }
    
    public void Checkout()
    {
        // Nesne durumunun kontrolü
        if (_isCheckedOut)
            throw new InvalidOperationException("Sepet zaten ödenmiş durumda.");
        
        if (_items.Count == 0)
            throw new InvalidOperationException("Boş sepet için ödeme yapılamaz.");
        
        // Ödeme işlemi...
        _isCheckedOut = true;
    }
}
```

### 3. Savunmacı Programlama

```csharp
public class FileProcessor
{
    public void ProcessFile(string filePath)
    {
        // Dosya yolu kontrolü
        if (string.IsNullOrEmpty(filePath))
            throw new ArgumentException("Dosya yolu boş olamaz.", nameof(filePath));
        
        // Dosyanın varlığını kontrol etme
        if (!File.Exists(filePath))
            throw new FileNotFoundException("Dosya bulunamadı.", filePath);
        
        try
        {
            // Dosya erişim iznini kontrol etme
            using (var fs = File.OpenRead(filePath))
            {
                // Dosya işleme...
            }
        }
        catch (UnauthorizedAccessException ex)
        {
            throw new AccessDeniedException("Dosyaya erişim izni yok.", ex);
        }
    }
}

public class AccessDeniedException : Exception
{
    public AccessDeniedException(string message, Exception innerException)
        : base(message, innerException) { }
}
```

## 3. Fail Fast'in Faydaları

1. **Hataların Erken Tespiti**
   - Sorunlar, daha büyük problemlere dönüşmeden tespit edilir
   - Hata ayıklama süreci kolaylaşır
   - Hatanın kaynağı daha net anlaşılır

2. **Daha Güvenilir Kod**
   - Beklenmeyen durumlar açıkça ele alınır
   - Varsayımlar açıkça doğrulanır
   - Yan etkiler azalır

3. **Daha İyi Dokümantasyon**
   - Metot sözleşmeleri açıkça belirtilir
   - Gereksinimler koda yansıtılır
   - Kullanım şartları netleşir

## 4. En İyi Pratikler

1. **Anlamlı Exception'lar Kullanın**
   - Özel exception sınıfları oluşturun
   - Hata mesajlarını açıklayıcı yapın
   - Exception hiyerarşisini doğru kullanın

2. **Giriş Noktalarını Koruyun**
   - Public API'lerde kapsamlı doğrulama yapın
   - Dış kaynaklardan gelen verileri doğrulayın
   - Sınır koşullarını kontrol edin

3. **Savunmacı Programlama Yapın**
   - Varsayımlarınızı doğrulayın
   - Dış sistemlerle etkileşimde hata senaryolarını düşünün
   - "Happy path" dışındaki durumları ele alın

4. **Assertion'ları Kullanın**
   - Kritik varsayımları assertion'larla doğrulayın
   - Debug modunda ek kontroller ekleyin
   - İç tutarlılığı kontrol edin

5. **Null Kontrollerini Optimize Edin**
   - C# 8.0+ için nullable reference types kullanın
   - Null-conditional operatörleri (?.) kullanın
   - Null-coalescing operatörleri (??) kullanın

## 5. Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Hataları Yutma**
   ```csharp
   // Kötü - hataları yutma
   public void SaveData(Data data)
   {
       try
       {
           _repository.Save(data);
       }
       catch (Exception)
       {
           // Hatayı yutma, hiçbir şey yapmama
       }
   }
   
   // İyi - hataları uygun şekilde ele alma
   public void SaveData(Data data)
   {
       try
       {
           _repository.Save(data);
       }
       catch (DbException ex)
       {
           _logger.LogError(ex, "Veri kaydedilirken hata oluştu.");
           throw new DataPersistenceException("Veri kaydedilemedi.", ex);
       }
   }
   ```

2. **Genel Exception Yakalama**
   ```csharp
   // Kötü - çok genel exception yakalama
   try
   {
       ProcessData(data);
   }
   catch (Exception ex)
   {
       // Tüm hataları aynı şekilde ele alma
       _logger.LogError(ex.Message);
   }
   
   // İyi - spesifik exception'ları yakalama
   try
   {
       ProcessData(data);
   }
   catch (ValidationException ex)
   {
       _logger.LogWarning(ex.Message);
       return BadRequest(ex.Message);
   }
   catch (NotFoundException ex)
   {
       _logger.LogInformation(ex.Message);
       return NotFound(ex.Message);
   }
   catch (Exception ex) when (IsOperationalError(ex))
   {
       _logger.LogError(ex, "Operasyonel hata oluştu.");
       return StatusCode(500, "Bir hata oluştu. Lütfen daha sonra tekrar deneyin.");
   }
   ```

3. **Aşırı Doğrulama**
   ```csharp
   // Kötü - aşırı doğrulama
   public void ProcessUserData(User user)
   {
       if (user == null)
           throw new ArgumentNullException(nameof(user));
       
       if (string.IsNullOrEmpty(user.Name))
           throw new ArgumentException("İsim boş olamaz.", nameof(user));
       
       if (user.Age < 0)
           throw new ArgumentException("Yaş negatif olamaz.", nameof(user));
       
       if (string.IsNullOrEmpty(user.Email))
           throw new ArgumentException("E-posta boş olamaz.", nameof(user));
       
       if (!IsValidEmail(user.Email))
           throw new ArgumentException("Geçersiz e-posta formatı.", nameof(user));
       
       // Daha birçok kontrol...
   }
   
   // İyi - doğrulama mantığını ayırma
   public void ProcessUserData(User user)
   {
       ValidateUser(user);
       
       // İşlem mantığı...
   }
   
   private void ValidateUser(User user)
   {
       if (user == null)
           throw new ArgumentNullException(nameof(user));
       
       var validator = new UserValidator();
       var result = validator.Validate(user);
       
       if (!result.IsValid)
           throw new ValidationException(result.Errors);
   }
   ```

4. **Sessiz Başarısızlık**
   ```csharp
   // Kötü - sessiz başarısızlık
   public bool TryDeleteFile(string path)
   {
       try
       {
           File.Delete(path);
           return true;
       }
       catch
       {
           return false; // Başarısızlık nedenini gizleme
       }
   }
   
   // İyi - başarısızlık nedenini belirtme
   public (bool Success, string ErrorMessage) TryDeleteFile(string path)
   {
       try
       {
           File.Delete(path);
           return (true, null);
       }
       catch (FileNotFoundException)
       {
           return (false, "Dosya bulunamadı.");
       }
       catch (UnauthorizedAccessException)
       {
           return (false, "Dosyayı silme izniniz yok.");
       }
       catch (IOException ex)
       {
           return (false, $"Dosya silinirken I/O hatası: {ex.Message}");
       }
   }
   ```

Fail Fast prensibi, yazılım geliştirmede hataların erken tespit edilmesini ve ele alınmasını sağlayan önemli bir yaklaşımdır. Bu prensibi doğru uygulayarak, daha güvenilir ve bakımı kolay sistemler geliştirebilirsiniz. 