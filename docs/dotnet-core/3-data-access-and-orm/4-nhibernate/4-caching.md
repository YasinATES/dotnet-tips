# Caching

NHibernate, performansı artırmak için çeşitli önbellek (cache) mekanizmaları sunar. Bu mekanizmalar, veritabanı erişimini azaltarak uygulamanızın daha hızlı çalışmasını sağlar. Bu bölümde, NHibernate'in sunduğu önbellek seçeneklerini inceleyeceğiz.

## First-level Cache

First-level cache (birinci seviye önbellek), NHibernate'in otomatik olarak sağladığı ve session seviyesinde çalışan bir önbellektir. Bu önbellek, bir session içinde aynı entity'nin tekrar tekrar yüklenmesini önler.

### First-level Cache Özellikleri

- Session kapsamında çalışır
- Otomatik olarak etkindir ve kapatılamaz
- Session kapandığında önbellek de temizlenir
- Aynı session içinde bir entity'nin tekrar yüklenmesini önler

### First-level Cache Örneği

```csharp
// First-level cache örneği
using (var session = sessionFactory.OpenSession())
{
    // İlk sorgu - veritabanından yüklenir
    var product1 = session.Get<Product>(1);
    
    // İkinci sorgu - önbellekten gelir, veritabanına sorgu gönderilmez
    var product2 = session.Get<Product>(1);
    
    // product1 ve product2 aynı nesne referansıdır
    Console.WriteLine(ReferenceEquals(product1, product2)); // True
}

// Farklı session'larda first-level cache paylaşılmaz
using (var session1 = sessionFactory.OpenSession())
using (var session2 = sessionFactory.OpenSession())
{
    // Her iki sorgu da veritabanına gider
    var product1 = session1.Get<Product>(1);
    var product2 = session2.Get<Product>(1);
    
    // product1 ve product2 farklı nesne referanslarıdır
    Console.WriteLine(ReferenceEquals(product1, product2)); // False
}
```

### First-level Cache ve Entity Durumları

First-level cache, entity'lerin durumlarını (state) takip eder:

```csharp
// Entity durumları ve first-level cache
using (var session = sessionFactory.OpenSession())
using (var transaction = session.BeginTransaction())
{
    // Transient durum - henüz session ile ilişkilendirilmemiş
    var newProduct = new Product { Name = "Yeni Ürün", Price = 100m };
    
    // Persistent durum - session ile ilişkilendirilmiş ve önbellekte
    session.Save(newProduct);
    
    // Değişiklikler otomatik olarak takip edilir
    newProduct.Price = 120m;
    
    // Flush çağrıldığında değişiklikler veritabanına yazılır
    session.Flush();
    
    // Detached durum - session kapatıldığında veya Clear çağrıldığında
    session.Clear(); // Önbelleği temizler
    
    transaction.Commit();
}
```

## Second-level Cache

Second-level cache (ikinci seviye önbellek), session'lar arasında paylaşılan bir önbellektir. Bu önbellek, farklı session'larda aynı entity'nin tekrar tekrar yüklenmesini önler.

### Second-level Cache Özellikleri

- SessionFactory seviyesinde çalışır
- Varsayılan olarak devre dışıdır, açıkça yapılandırılması gerekir
- Farklı session'lar arasında paylaşılır
- Entity'lerin yanı sıra koleksiyonlar ve sorgular için de kullanılabilir

### Second-level Cache Yapılandırma

Second-level cache'i etkinleştirmek için bir cache provider seçmeniz ve yapılandırmanız gerekir:

```csharp
// Second-level cache yapılandırma
var configuration = new Configuration();
configuration.SetProperty(NHibernate.Cfg.Environment.UseSecondLevelCache, "true");
configuration.SetProperty(NHibernate.Cfg.Environment.CacheProvider, 
    typeof(NHibernate.Cache.HashtableCacheProvider).AssemblyQualifiedName);

// Sorgu önbelleğini etkinleştirme
configuration.SetProperty(NHibernate.Cfg.Environment.UseQueryCache, "true");
```

### Entity'leri Önbelleklenebilir Yapma

Entity sınıflarını önbelleklenebilir yapmak için mapping dosyalarında veya attribute'lar ile yapılandırabilirsiniz:

```xml
<!-- XML mapping ile önbellekleme -->
<class name="Product" table="Products">
  <cache usage="read-write"/>
  <!-- Diğer mapping tanımları... -->
  
  <!-- Koleksiyon önbellekleme -->
  <set name="Categories" table="ProductCategories">
    <cache usage="read-write"/>
    <!-- Diğer koleksiyon tanımları... -->
  </set>
</class>
```

```csharp
// Fluent mapping ile önbellekleme
public class ProductMap : ClassMap<Product>
{
    public ProductMap()
    {
        Table("Products");
        Id(x => x.Id).GeneratedBy.Native();
        Map(x => x.Name);
        Map(x => x.Price);
        
        // Entity önbellekleme
        Cache.ReadWrite();
        
        // Koleksiyon önbellekleme
        HasMany(x => x.Categories)
            .Table("ProductCategories")
            .Cache.ReadWrite();
    }
}
```

### Second-level Cache Kullanımı

Second-level cache etkinleştirildiğinde, entity'ler otomatik olarak önbelleklenir:

```csharp
// Second-level cache kullanımı
// İlk session
using (var session1 = sessionFactory.OpenSession())
{
    // Veritabanından yüklenir ve second-level cache'e kaydedilir
    var product = session1.Get<Product>(1);
}

// İkinci session
using (var session2 = sessionFactory.OpenSession())
{
    // Second-level cache'den yüklenir, veritabanına sorgu gönderilmez
    var product = session2.Get<Product>(1);
}
```

### Cache Concurrency Stratejileri

NHibernate, farklı önbellek eşzamanlılık stratejileri sunar:

- **Read-only**: Sadece okunabilen, asla değişmeyen veriler için (örn. referans verileri)
- **Read-write**: Okunabilen ve yazılabilen veriler için, optimistik kilitleme kullanır
- **Nonstrict-read-write**: Nadiren güncellenen veriler için, eşzamanlılık garantisi düşüktür
- **Transactional**: Tam işlem güvenliği gerektiren veriler için, JTA işlemleri gerektirir

```csharp
// Cache stratejisi belirleme
public class ProductMap : ClassMap<Product>
{
    public ProductMap()
    {
        // Diğer mapping tanımları...
        
        // Read-only stratejisi
        Cache.ReadOnly();
        
        // Read-write stratejisi
        // Cache.ReadWrite();
        
        // Nonstrict-read-write stratejisi
        // Cache.NonStrictReadWrite();
        
        // Transactional stratejisi
        // Cache.Transactional();
    }
}
```

## Query Cache

Query cache (sorgu önbelleği), sorgu sonuçlarını önbelleklemenize olanak tanır. Bu, aynı parametrelerle sık sık çalıştırılan sorgular için faydalıdır.

### Query Cache Etkinleştirme

Sorgu önbelleğini etkinleştirmek için yapılandırma ayarlarını değiştirmeniz gerekir:

```csharp
// Sorgu önbelleğini etkinleştirme
configuration.SetProperty(NHibernate.Cfg.Environment.UseQueryCache, "true");
```

### Query Cache Kullanımı

Sorgu önbelleğini kullanmak için, sorguları çalıştırırken `SetCacheable` metodunu çağırmanız gerekir:

```csharp
// Sorgu önbelleği kullanımı
using (var session = sessionFactory.OpenSession())
{
    // HQL sorgusu önbellekleme
    var products = session.CreateQuery("from Product p where p.Price > :minPrice")
                         .SetParameter("minPrice", 100m)
                         .SetCacheable(true) // Sorguyu önbelleklenebilir yapar
                         .List<Product>();
    
    // Criteria sorgusu önbellekleme
    var expensiveProducts = session.CreateCriteria<Product>()
                                  .Add(Restrictions.Gt("Price", 100m))
                                  .SetCacheable(true) // Sorguyu önbelleklenebilir yapar
                                  .List<Product>();
    
    // LINQ sorgusu önbellekleme
    var linqProducts = session.Query<Product>()
                             .Where(p => p.Price > 100m)
                             .Cacheable() // Sorguyu önbelleklenebilir yapar
                             .ToList();
}
```

### Query Cache Bölgeleri

Sorgu önbelleğini bölgelere ayırarak daha iyi yönetebilirsiniz:

```csharp
// Sorgu önbellek bölgeleri
using (var session = sessionFactory.OpenSession())
{
    // Belirli bir bölgede önbellekleme
    var products = session.CreateQuery("from Product p where p.Price > :minPrice")
                         .SetParameter("minPrice", 100m)
                         .SetCacheable(true)
                         .SetCacheRegion("product.queries") // Önbellek bölgesi belirleme
                         .List<Product>();
    
    // Farklı bir bölgede önbellekleme
    var categories = session.CreateQuery("from Category c")
                           .SetCacheable(true)
                           .SetCacheRegion("category.queries") // Farklı bir önbellek bölgesi
                           .List<Category>();
}
```

## Cache Providers

NHibernate, çeşitli önbellek sağlayıcıları (cache providers) destekler. Her sağlayıcının kendi avantajları ve dezavantajları vardır.

### Yerleşik Cache Providers

NHibernate, bazı yerleşik önbellek sağlayıcıları sunar:

- **HashtableCacheProvider**: Basit, bellek içi önbellek, sadece geliştirme için uygundur
- **PrevalenceCache**: Prevalence framework tabanlı önbellek
- **SysCache**: ASP.NET Cache tabanlı önbellek
- **SysCache2**: ASP.NET Cache tabanlı, SQL Server bildirimlerini destekleyen önbellek

```csharp
// HashtableCacheProvider yapılandırma
configuration.SetProperty(NHibernate.Cfg.Environment.CacheProvider, 
    typeof(NHibernate.Cache.HashtableCacheProvider).AssemblyQualifiedName);

// SysCache yapılandırma
configuration.SetProperty(NHibernate.Cfg.Environment.CacheProvider, 
    typeof(NHibernate.Caches.SysCache.SysCacheProvider).AssemblyQualifiedName);
```

### Üçüncü Parti Cache Providers

NHibernate için çeşitli üçüncü parti önbellek sağlayıcıları da mevcuttur:

- **NHibernate.Caches.Redis**: Redis tabanlı dağıtık önbellek
- **NHibernate.Caches.MemCache**: Memcached tabanlı dağıtık önbellek
- **NHibernate.Caches.RtMemoryCache**: .NET Core Memory Cache tabanlı önbellek

```csharp
// Redis cache provider yapılandırma
// Önce NuGet paketi yüklenir: Install-Package NHibernate.Caches.Redis
configuration.SetProperty(NHibernate.Cfg.Environment.CacheProvider, 
    typeof(NHibernate.Caches.Redis.RedisCacheProvider).AssemblyQualifiedName);
configuration.SetProperty("redis.config", "localhost:6379");

// MemCache provider yapılandırma
// Önce NuGet paketi yüklenir: Install-Package NHibernate.Caches.MemCache
configuration.SetProperty(NHibernate.Cfg.Environment.CacheProvider, 
    typeof(NHibernate.Caches.MemCache.MemCacheProvider).AssemblyQualifiedName);
configuration.SetProperty("memcache.servers", "localhost:11211");
```

### Cache Provider Seçimi

Önbellek sağlayıcısı seçerken şu faktörleri göz önünde bulundurmalısınız:

- Uygulama ölçeği (tek sunucu veya çoklu sunucu)
- Performans gereksinimleri
- Bellek kullanımı
- Dağıtık önbellek ihtiyacı
- Önbellek bildirim mekanizmaları

## Cache Invalidation

Önbellek geçersiz kılma (cache invalidation), önbellekteki verilerin güncel olmadığında temizlenmesi işlemidir. NHibernate, bazı durumlarda önbelleği otomatik olarak geçersiz kılar, ancak bazen manuel olarak geçersiz kılmanız gerekebilir.

### Otomatik Cache Invalidation

NHibernate, aşağıdaki durumlarda önbelleği otomatik olarak geçersiz kılar:

- Entity güncellendiğinde veya silindiğinde
- İlişkili entity'ler güncellendiğinde (ilişki türüne bağlı olarak)
- Transaction commit edildiğinde (cache stratejisine bağlı olarak)

### Manuel Cache Invalidation

Bazen önbelleği manuel olarak geçersiz kılmanız gerekebilir:

```csharp
// Manuel önbellek geçersiz kılma
using (var session = sessionFactory.OpenSession())
{
    // Belirli bir entity'yi önbellekten temizleme
    session.Evict(typeof(Product), 1);
    
    // Bir entity türünün tüm örneklerini önbellekten temizleme
    session.SessionFactory.Evict(typeof(Product));
    
    // Belirli bir koleksiyonu önbellekten temizleme
    session.SessionFactory.Evict(typeof(Product), "Categories");
    
    // Tüm second-level cache'i temizleme
    session.SessionFactory.EvictQueries();
    session.SessionFactory.Evict(typeof(object));
    
    // Belirli bir sorgu önbellek bölgesini temizleme
    session.SessionFactory.EvictQueries("product.queries");
}
```

### Cache Invalidation Stratejileri

Önbellek geçersiz kılma için çeşitli stratejiler kullanabilirsiniz:

1. **Zaman Tabanlı Geçersiz Kılma**: Önbellekteki verilerin belirli bir süre sonra otomatik olarak geçersiz kılınması

```csharp
// Zaman tabanlı geçersiz kılma (SysCache örneği)
<property name="expiration" value="300" /> <!-- 5 dakika -->
```

2. **Olay Tabanlı Geçersiz Kılma**: Belirli olaylar gerçekleştiğinde önbelleğin geçersiz kılınması

```csharp
// Olay tabanlı geçersiz kılma
public class ProductService
{
    private readonly ISessionFactory sessionFactory;
    
    public void UpdateProductPrice(int productId, decimal newPrice)
    {
        using (var session = sessionFactory.OpenSession())
        using (var transaction = session.BeginTransaction())
        {
            var product = session.Get<Product>(productId);
            product.Price = newPrice;
            session.Update(product);
            transaction.Commit();
            
            // İlgili sorgu önbelleğini geçersiz kılma
            sessionFactory.EvictQueries("product.queries");
        }
    }
}
```

3. **SQL Bildirim Tabanlı Geçersiz Kılma**: Veritabanı değişikliklerini izleyerek önbelleği geçersiz kılma (SysCache2 gibi sağlayıcılar için)

```xml
<!-- SysCache2 SQL bildirim yapılandırması -->
<syscache2>
  <cacheRegion name="product.queries" priority="5">
    <dependencies>
      <tables>
        <add name="Products" />
      </tables>
    </dependencies>
  </cacheRegion>
</syscache2>
```

## Cache Monitoring ve Performans

Önbellek performansını izlemek ve optimize etmek, uygulamanızın genel performansını artırmak için önemlidir.

### Cache İstatistikleri

NHibernate, önbellek kullanımı hakkında istatistikler sağlar:

```csharp
// Önbellek istatistiklerini etkinleştirme
configuration.SetProperty(NHibernate.Cfg.Environment.GenerateStatistics, "true");

// İstatistikleri alma
var statistics = sessionFactory.Statistics;
Console.WriteLine($"Second-level cache hit count: {statistics.SecondLevelCacheHitCount}");
Console.WriteLine($"Second-level cache miss count: {statistics.SecondLevelCacheMissCount}");
Console.WriteLine($"Second-level cache put count: {statistics.SecondLevelCachePutCount}");
Console.WriteLine($"Query cache hit count: {statistics.QueryCacheHitCount}");
Console.WriteLine($"Query cache miss count: {statistics.QueryCacheMissCount}");
Console.WriteLine($"Query cache put count: {statistics.QueryCachePutCount}");
```

### Cache Optimizasyon İpuçları

Önbellek performansını optimize etmek için şu ipuçlarını göz önünde bulundurabilirsiniz:

1. **Doğru Cache Stratejisi Seçimi**: Verilerinizin kullanım desenine uygun önbellek stratejisi seçin
2. **Önbellek Bölgeleri Kullanımı**: İlgili entity'leri ve sorguları aynı önbellek bölgelerinde gruplandırın
3. **Önbellek Boyutu Yönetimi**: Önbellek boyutunu uygulamanızın ihtiyaçlarına göre ayarlayın
4. **Önbellek Ömrü Ayarı**: Verilerin değişim sıklığına göre önbellek ömrünü ayarlayın
5. **Seçici Önbellekleme**: Sadece sık erişilen ve nadiren değişen verileri önbellekleyin

## Özet

Bu bölümde, NHibernate'in sunduğu önbellek mekanizmalarını inceledik:

- **First-level Cache**: Session seviyesinde otomatik olarak çalışan önbellek
- **Second-level Cache**: SessionFactory seviyesinde, session'lar arasında paylaşılan önbellek
- **Query Cache**: Sorgu sonuçlarını önbelleklemeye olanak tanıyan mekanizma
- **Cache Providers**: Farklı önbellek sağlayıcıları ve özellikleri
- **Cache Invalidation**: Önbellekteki verilerin geçersiz kılınması yöntemleri

Doğru önbellek stratejilerini kullanarak, uygulamanızın performansını önemli ölçüde artırabilirsiniz. Ancak, önbellekleme her zaman bir ödünleşim gerektirir: bellek kullanımı ve veri tutarlılığı karşılığında performans kazanımı. Bu nedenle, uygulamanızın gereksinimlerine göre en uygun önbellek stratejisini seçmek önemlidir. 