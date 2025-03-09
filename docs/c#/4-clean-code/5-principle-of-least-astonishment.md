# Principle of Least Astonishment (POLA)

Principle of Least Astonishment (En Az Şaşırtma Prensibi), bir sistemin kullanıcılarını (hem son kullanıcılar hem de geliştiriciler) mümkün olduğunca az şaşırtacak şekilde davranması gerektiğini savunur. Bu prensip, kodun ve API'lerin beklenen ve tahmin edilebilir şekilde çalışması gerektiğini vurgular.

## 1. POLA'nın Temel Kavramı

POLA'nın özü, yazılımın kullanıcıların beklentileriyle uyumlu olmasıdır. Metot isimleri, parametre sıralamaları, dönüş değerleri ve davranışlar mantıklı ve öngörülebilir olmalıdır.

### POLA İhlali Örneği

```csharp
// POLA ihlali - Şaşırtıcı davranış
public class StringHelper
{
    // Şaşırtıcı: Boş string için exception fırlatıyor
    public static string Capitalize(string input)
    {
        if (string.IsNullOrEmpty(input))
            throw new ArgumentException("Input cannot be empty");
            
        return char.ToUpper(input[0]) + input.Substring(1);
    }
    
    // Şaşırtıcı: Parametre sırası mantıksız
    public static string Replace(string newValue, string oldValue, string input)
    {
        return input.Replace(oldValue, newValue);
    }
    
    // Şaşırtıcı: İsmiyle çelişen davranış
    public static bool IsEmpty(string input)
    {
        return input == null || input.Trim().Length == 0; // IsNullOrWhiteSpace ile karıştırılabilir
    }
}
```

### POLA Uyumlu Tasarım

```csharp
public static class StringExtensions
{
    // Öngörülebilir: Null için null, boş string için boş string döndürür
    public static string Capitalize(this string input)
    {
        if (string.IsNullOrEmpty(input))
            return input;
            
        return char.ToUpper(input[0]) + input.Substring(1);
    }
    
    // Öngörülebilir: Parametre sırası mantıklı
    public static string Replace(this string input, string oldValue, string newValue)
    {
        return input.Replace(oldValue, newValue);
    }
    
    // Öngörülebilir: İsmiyle tutarlı davranış
    public static bool IsEmpty(this string input)
    {
        return string.IsNullOrEmpty(input);
    }
    
    // Öngörülebilir: Farklı boşluk kontrolü için farklı metot
    public static bool IsBlank(this string input)
    {
        return string.IsNullOrWhiteSpace(input);
    }
}
```

## 2. POLA'nın Uygulama Alanları

### 1. API Tasarımı

```csharp
// POLA ihlali
public interface IUserService
{
    // Şaşırtıcı: Silme işlemi bool döndürüyor
    bool DeleteUser(int userId);
    
    // Şaşırtıcı: Güncelleme işlemi void
    void UpdateUser(User user);
    
    // Şaşırtıcı: Aynı işlem farklı isimle
    User GetUserById(int id);
    User FetchUser(int userId);
}

// POLA uyumlu
public interface IUserService
{
    // Öngörülebilir: Exception ile hata durumu bildirimi
    void DeleteUser(int userId);
    
    // Öngörülebilir: Güncel kullanıcıyı dönme
    User UpdateUser(User user);
    
    // Öngörülebilir: Tutarlı isimlendirme
    User GetUserById(int userId);
}
```

### 2. Exception Handling

```csharp
// POLA ihlali
public class DataService
{
    public void SaveData(string data)
    {
        try
        {
            // Veri kaydetme işlemi
        }
        catch (Exception)
        {
            // Şaşırtıcı: Hatayı yutuyor
            return;
        }
    }
}

// POLA uyumlu
public class DataService
{
    public void SaveData(string data)
    {
        try
        {
            // Veri kaydetme işlemi
        }
        catch (IOException ex)
        {
            // Öngörülebilir: Spesifik hata ve açıklayıcı mesaj
            throw new DataServiceException("Failed to save data", ex);
        }
    }
}
```

## 3. En İyi Pratikler

1. **Tutarlı İsimlendirme Kullanın**
   - Metot isimleri davranışı açıkça belirtmeli
   - Benzer işlemler için benzer isimlendirmeler kullanın
   - Domain dilini yansıtan isimler seçin

2. **Parametreleri Mantıklı Sıralayın**
   - En önemli parametreleri önce koyun
   - Opsiyonel parametreleri sona bırakın
   - Benzer metotlarda tutarlı parametre sırası kullanın

3. **Dönüş Değerlerini Standardize Edin**
   - Benzer işlemler benzer tipte değerler döndürmeli
   - Null dönüş değerlerini dokümante edin
   - Exception kullanımında tutarlı olun

4. **Açık ve Net Dokümantasyon Yapın**
   - Beklenmeyen davranışları belgelendirin
   - Örneklerle kullanımı gösterin
   - XML dokümantasyonu kullanın

5. **Platform Konvansiyonlarına Uyun**
   - Dilin standart kütüphanesindeki desenleri takip edin
   - Framework'ün naming convention'larına uyun
   - Yaygın tasarım desenlerini kullanın

## 4. Yaygın Hatalar ve Kaçınılması Gerekenler

1. **Belirsiz İsimlendirme**
   ```csharp
   // Kötü - belirsiz isimler
   public class Helper
   {
       public void Process(string input) { }
       public void Handle(string data) { }
       public void Manage(string content) { }
   }
   
   // İyi - açık ve net isimler
   public class DocumentProcessor
   {
       public void ValidateDocument(string content) { }
       public void SaveDocument(string content) { }
       public void ArchiveDocument(string content) { }
   }
   ```

2. **Tutarsız Davranış**
   ```csharp
   // Kötü - tutarsız null handling
   public class UserService
   {
       public User GetUserById(int id)
       {
           return null; // Kullanıcı bulunamadığında null
       }
       
       public User GetUserByEmail(string email)
       {
           throw new NotFoundException(); // Kullanıcı bulunamadığında exception
       }
   }
   
   // İyi - tutarlı davranış
   public class UserService
   {
       public User GetUserById(int id)
       {
           var user = _repository.FindById(id);
           if (user == null)
               throw new UserNotFoundException(id);
           return user;
       }
       
       public User GetUserByEmail(string email)
       {
           var user = _repository.FindByEmail(email);
           if (user == null)
               throw new UserNotFoundException(email);
           return user;
       }
   }
   ```

3. **Yanlış Exception Kullanımı**
   ```csharp
   // Kötü - yanlış exception tipleri
   public class Calculator
   {
       public int Divide(int a, int b)
       {
           if (b == 0)
               throw new Exception("Cannot divide by zero"); // Genel exception
           
           return a / b;
       }
   }
   
   // İyi - uygun exception tipleri
   public class Calculator
   {
       public int Divide(int a, int b)
       {
           if (b == 0)
               throw new DivideByZeroException("Division by zero is not allowed");
           
           return a / b;
       }
   }
   ```

POLA, yazılım geliştirmede kullanıcı deneyimini ve kod kalitesini artıran temel bir prensiptir. Bu prensibi doğru uygulayarak, daha anlaşılır, bakımı kolay ve güvenilir sistemler geliştirebilirsiniz. 