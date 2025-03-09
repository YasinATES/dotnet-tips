# Advanced Features

NHibernate, temel ORM işlevlerinin yanı sıra, uygulamanızın esnekliğini ve gücünü artıran çeşitli gelişmiş özellikler sunar. Bu bölümde, NHibernate'in sunduğu gelişmiş özellikleri inceleyeceğiz.

## Interceptors

Interceptor'lar (araya giriciler), NHibernate'in normal işlem akışına müdahale etmenize ve özelleştirmenize olanak tanır. Bu, veritabanı işlemlerini denetleme, değiştirme veya zenginleştirme için güçlü bir mekanizmadır.

### IInterceptor Arayüzü

NHibernate'in `IInterceptor` arayüzü, çeşitli olaylara tepki vermenizi sağlar:

```csharp
// Temel interceptor örneği
public class AuditInterceptor : EmptyInterceptor
{
    // Entity kaydedilmeden önce çağrılır
    public override bool OnSave(object entity, object id, object[] state, string[] propertyNames, IType[] types)
    {
        // Eğer entity bir IAuditable ise
        if (entity is IAuditable auditable)
        {
            auditable.CreatedDate = DateTime.Now;
            auditable.CreatedBy = CurrentUser.Identity;
            
            // State dizisini güncelle
            for (int i = 0; i < propertyNames.Length; i++)
            {
                if (propertyNames[i] == "CreatedDate")
                    state[i] = auditable.CreatedDate;
                else if (propertyNames[i] == "CreatedBy")
                    state[i] = auditable.CreatedBy;
            }
            
            // True döndürerek state dizisinin değiştiğini belirtiyoruz
            return true;
        }
        
        return false;
    }
    
    // Entity güncellenmeden önce çağrılır
    public override bool OnFlushDirty(object entity, object id, object[] currentState, object[] previousState, string[] propertyNames, IType[] types)
    {
        // Eğer entity bir IAuditable ise
        if (entity is IAuditable auditable)
        {
            auditable.ModifiedDate = DateTime.Now;
            auditable.ModifiedBy = CurrentUser.Identity;
            
            // State dizisini güncelle
            for (int i = 0; i < propertyNames.Length; i++)
            {
                if (propertyNames[i] == "ModifiedDate")
                    currentState[i] = auditable.ModifiedDate;
                else if (propertyNames[i] == "ModifiedBy")
                    currentState[i] = auditable.ModifiedBy;
            }
            
            // True döndürerek state dizisinin değiştiğini belirtiyoruz
            return true;
        }
        
        return false;
    }
}
```

### Interceptor Kullanımı

Interceptor'ları session veya session factory seviyesinde kullanabilirsiniz:

```csharp
// Session seviyesinde interceptor kullanımı
var interceptor = new AuditInterceptor();
using (var session = sessionFactory.OpenSession(interceptor))
{
    // Bu session'da yapılan tüm işlemler interceptor tarafından denetlenir
    var product = new Product { Name = "Yeni Ürün", Price = 100m };
    session.Save(product); // AuditInterceptor.OnSave çağrılır
}

// Session factory seviyesinde interceptor kullanımı
var configuration = new Configuration();
// Diğer yapılandırmalar...
var interceptor = new AuditInterceptor();
var sessionFactory = configuration.BuildSessionFactory(interceptor);
```

### Yaygın Interceptor Kullanım Senaryoları

1. **Denetim (Audit) Kaydı**: Entity'lerin oluşturulma ve değiştirilme zamanlarını otomatik olarak kaydetme
2. **Veri Doğrulama**: Entity'lerin kaydedilmeden veya güncellenmeden önce doğrulanması
3. **Şifreleme/Şifre Çözme**: Hassas verilerin otomatik olarak şifrelenmesi ve çözülmesi
4. **Soft Delete**: Entity'lerin silinmek yerine işaretlenmesi
5. **Performans İzleme**: Sorguların ve işlemlerin performansını izleme

## Event Listeners

Event listener'lar (olay dinleyiciler), NHibernate'in çeşitli olaylarına tepki vermenizi sağlar. Interceptor'lara benzer, ancak daha spesifik olaylara odaklanır ve daha modüler bir yaklaşım sunar.

### Event Listener Türleri

NHibernate, çeşitli olay türleri için listener'lar sunar:

- **IPreInsertEventListener**: Entity eklenmeden önce
- **IPostInsertEventListener**: Entity eklendikten sonra
- **IPreUpdateEventListener**: Entity güncellenmeden önce
- **IPostUpdateEventListener**: Entity güncellendikten sonra
- **IPreDeleteEventListener**: Entity silinmeden önce
- **IPostDeleteEventListener**: Entity silindikten sonra
- **IPreLoadEventListener**: Entity yüklenmeden önce
- **IPostLoadEventListener**: Entity yüklendikten sonra
- **IFlushEventListener**: Session flush edildiğinde
- **IDirtyCheckEventListener**: Kirli kontrol yapıldığında

### Event Listener Örneği

```csharp
// Post-load event listener örneği
public class AuditPostLoadEventListener : IPostLoadEventListener
{
    public void OnPostLoad(PostLoadEvent @event)
    {
        // Yüklenen entity
        var entity = @event.Entity;
        
        // Eğer entity bir IAuditable ise
        if (entity is IAuditable auditable)
        {
            // Yükleme zamanını kaydet
            auditable.LastLoadedTime = DateTime.Now;
            
            // Yükleme sayacını artır
            auditable.LoadCount++;
            
            Console.WriteLine($"Entity yüklendi: {entity.GetType().Name}, ID: {@event.Id}");
        }
    }
}
```

### Event Listener Kaydetme

Event listener'ları NHibernate yapılandırmasına kaydetmeniz gerekir:

```csharp
// Event listener kaydetme
var configuration = new Configuration();
// Diğer yapılandırmalar...

// Post-load event listener kaydetme
var postLoadListener = new AuditPostLoadEventListener();
configuration.EventListeners.PostLoadEventListeners = 
    new IPostLoadEventListener[] { postLoadListener };

// Pre-update event listener kaydetme
var preUpdateListener = new ValidationPreUpdateEventListener();
configuration.EventListeners.PreUpdateEventListeners = 
    new IPreUpdateEventListener[] { preUpdateListener };

var sessionFactory = configuration.BuildSessionFactory();
```

### Çoklu Event Listener

Aynı olay türü için birden fazla listener kaydedebilirsiniz:

```csharp
// Çoklu event listener kaydetme
configuration.EventListeners.PostLoadEventListeners = 
    new IPostLoadEventListener[] 
    { 
        new AuditPostLoadEventListener(),
        new CachePostLoadEventListener(),
        new LoggingPostLoadEventListener() 
    };
```

## Custom User Types

NHibernate, yerleşik veri türlerinin yanı sıra, özel veri türleri tanımlamanıza olanak tanır. Bu, veritabanında doğrudan desteklenmeyen türleri eşlemenize veya karmaşık veri dönüşümleri yapmanıza yardımcı olur.

### IUserType Arayüzü

Özel bir tür oluşturmak için `IUserType` arayüzünü uygulamanız gerekir:

```csharp
// Özel para birimi türü örneği
public class MoneyType : IUserType
{
    // Türün SQL türü
    public SqlType[] SqlTypes => new[] { SqlTypeFactory.GetDecimal(19, 4) };
    
    // .NET türü
    public Type ReturnedType => typeof(Money);
    
    // Değer eşitliği kontrolü
    public bool Equals(object x, object y)
    {
        if (x == null && y == null) return true;
        if (x == null || y == null) return false;
        return x.Equals(y);
    }
    
    // Hash kodu hesaplama
    public int GetHashCode(object x) => x.GetHashCode();
    
    // Veritabanından değer okuma
    public object NullSafeGet(IDataReader rs, string[] names, object owner)
    {
        var amount = NHibernateUtil.Decimal.NullSafeGet(rs, names[0]) as decimal?;
        if (!amount.HasValue) return null;
        return new Money(amount.Value);
    }
    
    // Veritabanına değer yazma
    public void NullSafeSet(IDbCommand cmd, object value, int index)
    {
        if (value == null)
        {
            NHibernateUtil.Decimal.NullSafeSet(cmd, null, index);
        }
        else
        {
            var money = (Money)value;
            NHibernateUtil.Decimal.NullSafeSet(cmd, money.Amount, index);
        }
    }
    
    // Derin kopya oluşturma
    public object DeepCopy(object value)
    {
        if (value == null) return null;
        var money = (Money)value;
        return new Money(money.Amount);
    }
    
    // Diğer gerekli metodlar...
    public bool IsMutable => false;
    public object Replace(object original, object target, object owner) => original;
    public object Assemble(object cached, object owner) => cached;
    public object Disassemble(object value) => value;
}
```

### Özel Tür Kullanımı

Özel türleri entity mapping'de kullanabilirsiniz:

```csharp
// XML mapping ile özel tür kullanımı
<property name="Price" type="MyNamespace.MoneyType, MyAssembly">
    <column name="Price" sql-type="decimal(19,4)" />
</property>

// Fluent mapping ile özel tür kullanımı
Map(x => x.Price)
    .CustomType<MoneyType>()
    .Column("Price");
```

### Kompozit Özel Türler

Birden fazla sütunu tek bir özelliğe eşlemek için `ICompositeUserType` arayüzünü kullanabilirsiniz:

```csharp
// Kompozit adres türü örneği
public class AddressType : ICompositeUserType
{
    // Özellik adları
    public string[] PropertyNames => new[] { "Street", "City", "PostalCode", "Country" };
    
    // Özellik türleri
    public IType[] PropertyTypes => new IType[] 
    { 
        NHibernateUtil.String, 
        NHibernateUtil.String,
        NHibernateUtil.String,
        NHibernateUtil.String
    };
    
    // Özellik değeri alma
    public object GetPropertyValue(object component, int property)
    {
        var address = (Address)component;
        switch (property)
        {
            case 0: return address.Street;
            case 1: return address.City;
            case 2: return address.PostalCode;
            case 3: return address.Country;
            default: throw new ArgumentOutOfRangeException(nameof(property));
        }
    }
    
    // Özellik değeri ayarlama
    public void SetPropertyValue(object component, int property, object value)
    {
        var address = (Address)component;
        switch (property)
        {
            case 0: address.Street = (string)value; break;
            case 1: address.City = (string)value; break;
            case 2: address.PostalCode = (string)value; break;
            case 3: address.Country = (string)value; break;
            default: throw new ArgumentOutOfRangeException(nameof(property));
        }
    }
    
    // Diğer gerekli metodlar...
}
```

## Filters

Filter'lar (filtreler), sorguları otomatik olarak filtrelemenize olanak tanır. Bu, belirli koşulları tüm sorgulara uygulamak için kullanışlıdır, örneğin soft-deleted entity'leri gizlemek veya çok kiracılı uygulamalarda kiracıya özgü verileri filtrelemek için.

### Filter Tanımlama

Filter'ları mapping dosyalarında veya programatik olarak tanımlayabilirsiniz:

```csharp
// XML mapping ile filter tanımlama
<hibernate-mapping>
    <filter-def name="activeOnly">
        <filter-param name="isActive" type="boolean"/>
    </filter-def>
    
    <class name="Product" table="Products">
        <!-- Diğer mapping tanımları... -->
        <filter name="activeOnly" condition="IsActive = :isActive"/>
    </class>
</hibernate-mapping>

// Fluent mapping ile filter tanımlama
public class ProductMap : ClassMap<Product>
{
    public ProductMap()
    {
        // Diğer mapping tanımları...
        
        // Filter tanımlama
        Filter("activeOnly", f => f.Condition("IsActive = :isActive"));
    }
}
```

### Filter Etkinleştirme ve Kullanma

Filter'ları session seviyesinde etkinleştirebilir ve parametrelerini ayarlayabilirsiniz:

```csharp
// Filter kullanımı
using (var session = sessionFactory.OpenSession())
{
    // Filter'ı etkinleştirme
    session.EnableFilter("activeOnly")
           .SetParameter("isActive", true);
    
    // Tüm sorgular otomatik olarak filtrelenir
    var products = session.CreateQuery("from Product")
                         .List<Product>();
    
    // Filter'ı devre dışı bırakma
    session.DisableFilter("activeOnly");
    
    // Artık tüm ürünler gelir
    var allProducts = session.CreateQuery("from Product")
                            .List<Product>();
}
```

### Yaygın Filter Kullanım Senaryoları

1. **Soft Delete**: Silinmiş olarak işaretlenen entity'leri gizleme
2. **Çok Kiracılık**: Belirli bir kiracıya ait verileri filtreleme
3. **Veri Erişim Kontrolü**: Kullanıcının erişim yetkisi olan verileri filtreleme
4. **Aktif/Pasif Durumu**: Sadece aktif entity'leri gösterme
5. **Geçici Veri**: Geçerliliğini yitirmiş verileri gizleme

## Multi-tenancy

Multi-tenancy (çok kiracılık), tek bir uygulama örneğinin birden fazla kiracıya (tenant) hizmet vermesini sağlayan bir mimaridir. NHibernate, çeşitli çok kiracılık stratejilerini destekler.

### Çok Kiracılık Stratejileri

NHibernate, üç temel çok kiracılık stratejisi sunar:

1. **Ayrı Veritabanı**: Her kiracı için ayrı bir veritabanı
2. **Ayrı Şema**: Aynı veritabanında her kiracı için ayrı bir şema
3. **Ayrı Sütun**: Aynı tablolarda kiracı kimliği sütunu

### Ayrı Veritabanı Stratejisi

Her kiracı için ayrı bir veritabanı kullanıldığında, kiracıya göre bağlantı dizesini değiştirmeniz gerekir:

```csharp
// Ayrı veritabanı stratejisi
public class TenantConnectionProvider : DriverConnectionProvider
{
    private readonly IDictionary<string, string> tenantConnectionStrings;
    
    public TenantConnectionProvider()
    {
        // Kiracı bağlantı dizelerini yükle
        tenantConnectionStrings = new Dictionary<string, string>();
        tenantConnectionStrings["tenant1"] = "Server=localhost;Database=Tenant1DB;...";
        tenantConnectionStrings["tenant2"] = "Server=localhost;Database=Tenant2DB;...";
    }
    
    public override IDbConnection GetConnection()
    {
        // Mevcut kiracıyı al
        var tenantId = TenantContext.CurrentTenant;
        
        // Kiracıya özgü bağlantı dizesini kullan
        if (tenantConnectionStrings.TryGetValue(tenantId, out var connectionString))
        {
            var connection = Driver.CreateConnection();
            connection.ConnectionString = connectionString;
            return connection;
        }
        
        // Varsayılan bağlantıyı kullan
        return base.GetConnection();
    }
}
```

### Ayrı Şema Stratejisi

Aynı veritabanında her kiracı için ayrı bir şema kullanıldığında, şema adını dinamik olarak değiştirmeniz gerekir:

```csharp
// Ayrı şema stratejisi
public class TenantSchemaInterceptor : EmptyInterceptor
{
    public override string OnPrepareStatement(string sql)
    {
        // Mevcut kiracıyı al
        var tenantId = TenantContext.CurrentTenant;
        
        // SQL sorgusundaki şema adını değiştir
        return sql.Replace("[dbo].", $"[{tenantId}].");
    }
}
```

### Ayrı Sütun Stratejisi

Aynı tablolarda kiracı kimliği sütunu kullanıldığında, filter'lar ile otomatik filtreleme yapabilirsiniz:

```csharp
// Ayrı sütun stratejisi
// Filter tanımlama
configuration.AddFilterDefinition(new FilterDefinition(
    "tenantFilter",
    "TenantId = :tenantId",
    new Dictionary<string, IType> { { "tenantId", NHibernateUtil.String } }
));

// Entity mapping
public class ProductMap : ClassMap<Product>
{
    public ProductMap()
    {
        // Diğer mapping tanımları...
        
        // Kiracı kimliği sütunu
        Map(x => x.TenantId);
        
        // Filter uygulama
        Filter("tenantFilter");
    }
}

// Filter etkinleştirme
using (var session = sessionFactory.OpenSession())
{
    // Mevcut kiracıyı al
    var tenantId = TenantContext.CurrentTenant;
    
    // Filter'ı etkinleştirme
    session.EnableFilter("tenantFilter")
           .SetParameter("tenantId", tenantId);
    
    // Tüm sorgular otomatik olarak filtrelenir
    var products = session.Query<Product>().ToList();
}
```

### Çok Kiracılık İçin Özel Çözümler

Bazen, NHibernate'in yerleşik çok kiracılık desteği yeterli olmayabilir. Bu durumda, özel çözümler geliştirebilirsiniz:

```csharp
// Özel çok kiracılık çözümü
public class TenantSessionFactory
{
    private readonly IDictionary<string, ISessionFactory> sessionFactories;
    
    public TenantSessionFactory()
    {
        sessionFactories = new Dictionary<string, ISessionFactory>();
    }
    
    public ISession OpenSession()
    {
        // Mevcut kiracıyı al
        var tenantId = TenantContext.CurrentTenant;
        
        // Kiracıya özgü session factory'yi al veya oluştur
        if (!sessionFactories.TryGetValue(tenantId, out var sessionFactory))
        {
            var configuration = new Configuration();
            // Kiracıya özgü yapılandırma
            configuration.SetProperty(Environment.ConnectionString, GetConnectionString(tenantId));
            sessionFactory = configuration.BuildSessionFactory();
            sessionFactories[tenantId] = sessionFactory;
        }
        
        // Kiracıya özgü session'ı aç
        return sessionFactory.OpenSession();
    }
    
    private string GetConnectionString(string tenantId)
    {
        // Kiracıya özgü bağlantı dizesini döndür
        return $"Server=localhost;Database={tenantId}DB;...";
    }
}
```

## Özet

Bu bölümde, NHibernate'in sunduğu gelişmiş özellikleri inceledik:

- **Interceptor'lar**: NHibernate'in normal işlem akışına müdahale etme
- **Event Listener'lar**: Belirli olaylara tepki verme
- **Custom User Types**: Özel veri türleri tanımlama
- **Filter'lar**: Sorguları otomatik olarak filtreleme
- **Multi-tenancy**: Çok kiracılı uygulamalar için destek

Bu gelişmiş özellikler, NHibernate'i daha esnek ve güçlü bir ORM çözümü haline getirir. Uygulamanızın gereksinimlerine göre bu özellikleri kullanarak, daha temiz, daha bakımı kolay ve daha performanslı uygulamalar geliştirebilirsiniz. 