# NHibernate Fundamentals

NHibernate, .NET platformu için geliştirilmiş, olgun ve güçlü bir ORM (Object-Relational Mapping) çözümüdür. Java dünyasındaki Hibernate'in .NET uyarlaması olan NHibernate, nesne yönelimli programlama ile ilişkisel veritabanları arasındaki uyumsuzluğu gidermek için tasarlanmıştır. Bu bölümde, NHibernate'in temel kavramlarını ve kullanımını inceleyeceğiz.

## Session Factory ve Session

NHibernate'in çalışma mantığı, `ISessionFactory` ve `ISession` arayüzleri etrafında şekillenir. Bu iki bileşen, NHibernate'in veritabanı işlemlerini yönetmesini sağlar.

### Session Factory

`ISessionFactory`, NHibernate uygulamasının merkezi bileşenidir. Uygulama başlangıcında bir kez oluşturulur ve tüm uygulama yaşam döngüsü boyunca kullanılır. Session Factory, veritabanı bağlantılarını ve mapping bilgilerini yönetir.

```csharp
// Session Factory oluşturma
var configuration = new Configuration();
configuration.Configure(); // hibernate.cfg.xml dosyasını kullanır
configuration.AddAssembly(typeof(Product).Assembly); // Entity sınıflarının bulunduğu assembly

// Session Factory oluşturma
ISessionFactory sessionFactory = configuration.BuildSessionFactory();
```

Session Factory oluşturma işlemi maliyetli olduğundan, genellikle uygulama başlangıcında bir kez oluşturulur ve singleton olarak kullanılır:

```csharp
// Singleton Session Factory örneği
public static class NHibernateHelper
{
    private static ISessionFactory _sessionFactory;
    private static readonly object _lock = new object();
    
    public static ISessionFactory GetSessionFactory()
    {
        if (_sessionFactory == null)
        {
            lock (_lock)
            {
                if (_sessionFactory == null)
                {
                    var configuration = new Configuration();
                    configuration.Configure();
                    configuration.AddAssembly(typeof(Product).Assembly);
                    
                    _sessionFactory = configuration.BuildSessionFactory();
                }
            }
        }
        
        return _sessionFactory;
    }
}
```

### Session

`ISession`, veritabanı işlemlerini gerçekleştirmek için kullanılan birincil arayüzdür. Session, veritabanı bağlantısını temsil eder ve CRUD (Create, Read, Update, Delete) işlemlerini gerçekleştirmek için kullanılır. Session, kısa ömürlüdür ve her işlem için yeni bir session oluşturulması önerilir.

```csharp
// Session oluşturma ve kullanma
using (var session = sessionFactory.OpenSession())
{
    // Session ile işlemler
    var product = session.Get<Product>(1); // ID'ye göre ürün getirme
    
    // Ürün güncelleme
    product.Price = 99.99m;
    session.Update(product);
    
    // Değişiklikleri kaydetme
    session.Flush();
}
```

Session, birinci seviye önbellek (first-level cache) sağlar. Bu, aynı session içinde aynı nesne için yapılan sorguların veritabanına gitmeden önbellekten döndürülmesini sağlar:

```csharp
// Birinci seviye önbellek örneği
using (var session = sessionFactory.OpenSession())
{
    // İlk sorgu veritabanına gider
    var product1 = session.Get<Product>(1);
    
    // İkinci sorgu önbellekten döner (veritabanına gitmez)
    var product2 = session.Get<Product>(1);
    
    // product1 ve product2 aynı nesne referansıdır
    Console.WriteLine(ReferenceEquals(product1, product2)); // True
}
```

### Transaction Yönetimi

NHibernate, transaction yönetimi için `ITransaction` arayüzünü sağlar. Transaction, veritabanı işlemlerinin atomik olarak gerçekleştirilmesini sağlar:

```csharp
// Transaction kullanımı
using (var session = sessionFactory.OpenSession())
using (var transaction = session.BeginTransaction())
{
    try
    {
        // İşlemler
        var product = new Product
        {
            Name = "Yeni Ürün",
            Price = 99.99m,
            Stock = 100
        };
        
        session.Save(product);
        
        // Başka işlemler...
        
        // Transaction'ı commit et
        transaction.Commit();
    }
    catch (Exception)
    {
        // Hata durumunda transaction'ı geri al
        transaction.Rollback();
        throw;
    }
}
```

## Mapping Stratejileri

NHibernate, nesneleri veritabanı tablolarına eşlemek için çeşitli mapping stratejileri sunar. Bu stratejiler, XML dosyaları, öznitelikler (attributes) veya Fluent API kullanılarak uygulanabilir.

### XML Mapping

Geleneksel olarak, NHibernate mapping'leri XML dosyaları kullanılarak tanımlanır. Her entity sınıfı için bir `.hbm.xml` dosyası oluşturulur:

```xml
<!-- Product.hbm.xml -->
<?xml version="1.0" encoding="utf-8" ?>
<hibernate-mapping xmlns="urn:nhibernate-mapping-2.2">
  <class name="MyApp.Domain.Product, MyApp.Domain" table="Products">
    <id name="Id" column="Id">
      <generator class="native" />
    </id>
    <property name="Name" column="Name" not-null="true" length="100" />
    <property name="Price" column="Price" not-null="true" />
    <property name="Stock" column="Stock" not-null="true" />
    <many-to-one name="Category" column="CategoryId" class="MyApp.Domain.Category, MyApp.Domain" />
  </class>
</hibernate-mapping>
```

XML mapping dosyaları, Configuration nesnesine eklenir:

```csharp
// XML mapping dosyalarını ekleme
var configuration = new Configuration();
configuration.Configure();
configuration.AddFile("Product.hbm.xml");
configuration.AddFile("Category.hbm.xml");
```

### Attribute Mapping

NHibernate, öznitelikler kullanarak mapping tanımlamayı da destekler. Bu yaklaşım, mapping bilgilerini doğrudan entity sınıflarına yerleştirir:

```csharp
// Attribute mapping örneği
[Class(Table = "Products")]
public class Product
{
    [Id(Name = "Id", Column = "Id")]
    [Generator(Class = "native")]
    public virtual int Id { get; set; }
    
    [Property(Column = "Name", NotNull = true, Length = 100)]
    public virtual string Name { get; set; }
    
    [Property(Column = "Price", NotNull = true)]
    public virtual decimal Price { get; set; }
    
    [Property(Column = "Stock", NotNull = true)]
    public virtual int Stock { get; set; }
    
    [ManyToOne(Column = "CategoryId", ClassType = typeof(Category))]
    public virtual Category Category { get; set; }
}
```

### Fluent Mapping

Fluent NHibernate, kod tabanlı mapping tanımlamayı sağlayan bir kütüphanedir. Bu yaklaşım, XML dosyalarına veya özniteliklere gerek kalmadan, C# kodu kullanarak mapping tanımlamayı mümkün kılar. Fluent mapping hakkında daha fazla bilgi, "Fluent NHibernate" bölümünde verilecektir.

## Configuration

NHibernate'i yapılandırmak için çeşitli yöntemler bulunmaktadır. En yaygın yöntem, `hibernate.cfg.xml` dosyası kullanmaktır.

### hibernate.cfg.xml

Bu dosya, veritabanı bağlantı bilgileri, dialect, mapping dosyaları ve diğer NHibernate ayarlarını içerir:

```xml
<!-- hibernate.cfg.xml -->
<?xml version="1.0" encoding="utf-8" ?>
<hibernate-configuration xmlns="urn:nhibernate-configuration-2.2">
  <session-factory>
    <!-- Veritabanı bağlantı ayarları -->
    <property name="connection.provider">NHibernate.Connection.DriverConnectionProvider</property>
    <property name="connection.driver_class">NHibernate.Driver.SqlClientDriver</property>
    <property name="connection.connection_string">Server=localhost;Database=MyDatabase;User Id=sa;Password=P@ssw0rd;</property>
    <property name="dialect">NHibernate.Dialect.MsSql2012Dialect</property>
    
    <!-- SQL çıktısını gösterme -->
    <property name="show_sql">true</property>
    
    <!-- Otomatik şema güncelleme -->
    <property name="hbm2ddl.auto">update</property>
    
    <!-- Mapping dosyaları -->
    <mapping assembly="MyApp.Domain" />
  </session-factory>
</hibernate-configuration>
```

### Programatik Yapılandırma

NHibernate'i kod içinde de yapılandırabilirsiniz:

```csharp
// Programatik yapılandırma
var configuration = new Configuration();

// Veritabanı bağlantı ayarları
configuration.SetProperty(NHibernate.Cfg.Environment.ConnectionProvider, "NHibernate.Connection.DriverConnectionProvider");
configuration.SetProperty(NHibernate.Cfg.Environment.ConnectionDriver, "NHibernate.Driver.SqlClientDriver");
configuration.SetProperty(NHibernate.Cfg.Environment.ConnectionString, "Server=localhost;Database=MyDatabase;User Id=sa;Password=P@ssw0rd;");
configuration.SetProperty(NHibernate.Cfg.Environment.Dialect, "NHibernate.Dialect.MsSql2012Dialect");

// SQL çıktısını gösterme
configuration.SetProperty(NHibernate.Cfg.Environment.ShowSql, "true");

// Otomatik şema güncelleme
configuration.SetProperty(NHibernate.Cfg.Environment.Hbm2ddlAuto, "update");

// Mapping dosyaları
configuration.AddAssembly("MyApp.Domain");
```

### Önemli Yapılandırma Ayarları

NHibernate'in davranışını kontrol eden bazı önemli yapılandırma ayarları şunlardır:

- **connection.provider**: Bağlantı sağlayıcısı
- **connection.driver_class**: Veritabanı sürücüsü
- **connection.connection_string**: Veritabanı bağlantı dizesi
- **dialect**: Veritabanı dialect'i (SQL dilini belirler)
- **show_sql**: SQL sorgularını konsola yazdırma
- **format_sql**: SQL sorgularını biçimlendirme
- **hbm2ddl.auto**: Otomatik şema güncelleme (create, update, validate, none)
- **cache.use_second_level_cache**: İkinci seviye önbelleği kullanma
- **cache.provider_class**: Önbellek sağlayıcısı

## Fluent NHibernate

Fluent NHibernate, NHibernate için kod tabanlı mapping tanımlamayı sağlayan bir kütüphanedir. XML dosyalarına veya özniteliklere gerek kalmadan, C# kodu kullanarak mapping tanımlamayı mümkün kılar.

### Kurulum

Fluent NHibernate'i NuGet üzerinden kurabilirsiniz:

```csharp
// NuGet üzerinden kurulum
Install-Package FluentNHibernate
```

### Entity Mapping

Fluent NHibernate ile entity mapping tanımlamak için, `ClassMap<T>` sınıfından türetilen mapping sınıfları oluşturulur:

```csharp
// Entity sınıfı
public class Product
{
    public virtual int Id { get; set; }
    public virtual string Name { get; set; }
    public virtual decimal Price { get; set; }
    public virtual int Stock { get; set; }
    public virtual Category Category { get; set; }
}

// Mapping sınıfı
public class ProductMap : ClassMap<Product>
{
    public ProductMap()
    {
        // Tablo adını belirtme
        Table("Products");
        
        // ID ve generator tanımlama
        Id(x => x.Id).GeneratedBy.Native();
        
        // Özellikleri eşleme
        Map(x => x.Name).Length(100).Not.Nullable();
        Map(x => x.Price).Not.Nullable();
        Map(x => x.Stock).Not.Nullable();
        
        // İlişkileri tanımlama
        References(x => x.Category).Column("CategoryId");
    }
}
```

### Session Factory Oluşturma

Fluent NHibernate ile session factory oluşturmak için `FluentConfiguration` kullanılır:

```csharp
// Fluent NHibernate ile session factory oluşturma
public static ISessionFactory CreateSessionFactory()
{
    return Fluently.Configure()
        .Database(MsSqlConfiguration.MsSql2012
            .ConnectionString("Server=localhost;Database=MyDatabase;User Id=sa;Password=P@ssw0rd;")
            .ShowSql())
        .Mappings(m => m.FluentMappings
            .AddFromAssemblyOf<ProductMap>())
        .ExposeConfiguration(cfg => new SchemaUpdate(cfg).Execute(false, true))
        .BuildSessionFactory();
}
```

### İlişki Mapping

Fluent NHibernate, çeşitli ilişki türlerini tanımlamak için zengin bir API sunar:

```csharp
// One-to-Many ilişkisi
public class CategoryMap : ClassMap<Category>
{
    public CategoryMap()
    {
        Table("Categories");
        Id(x => x.Id).GeneratedBy.Native();
        Map(x => x.Name).Length(50).Not.Nullable();
        
        // One-to-Many ilişkisi
        HasMany(x => x.Products)
            .KeyColumn("CategoryId")
            .Inverse()
            .Cascade.All();
    }
}

// Many-to-Many ilişkisi
public class ProductMap : ClassMap<Product>
{
    public ProductMap()
    {
        Table("Products");
        Id(x => x.Id).GeneratedBy.Native();
        Map(x => x.Name).Length(100).Not.Nullable();
        
        // Many-to-Many ilişkisi
        HasManyToMany(x => x.Tags)
            .Table("ProductTags")
            .ParentKeyColumn("ProductId")
            .ChildKeyColumn("TagId")
            .Cascade.All();
    }
}
```

## Conventions

Fluent NHibernate, tekrarlayan mapping kodunu azaltmak için convention'ları destekler. Convention'lar, belirli kurallara göre otomatik mapping tanımlamayı sağlar.

### Temel Convention'lar

Fluent NHibernate, çeşitli yerleşik convention'lar sunar:

```csharp
// Temel convention'lar
public static ISessionFactory CreateSessionFactory()
{
    return Fluently.Configure()
        .Database(MsSqlConfiguration.MsSql2012
            .ConnectionString("Server=localhost;Database=MyDatabase;User Id=sa;Password=P@ssw0rd;"))
        .Mappings(m => m.FluentMappings
            .AddFromAssemblyOf<ProductMap>()
            .Conventions.Add(
                // ID convention'ı
                ForeignKey.EndsWith("Id"),
                // Tablo adı convention'ı
                Table.Is(x => Inflector.Pluralize(x.EntityType.Name)),
                // Sütun adı convention'ı
                DefaultLazy.Always()
            ))
        .BuildSessionFactory();
}
```

### Özel Convention'lar

Özel convention'lar oluşturarak, kendi mapping kurallarınızı tanımlayabilirsiniz:

```csharp
// Özel convention örneği
public class CustomForeignKeyConvention : ForeignKeyConvention
{
    protected override string GetKeyName(Member property, Type type)
    {
        return property.Name + "Id";
    }
}

// Özel convention kullanımı
public static ISessionFactory CreateSessionFactory()
{
    return Fluently.Configure()
        .Database(MsSqlConfiguration.MsSql2012
            .ConnectionString("Server=localhost;Database=MyDatabase;User Id=sa;Password=P@ssw0rd;"))
        .Mappings(m => m.FluentMappings
            .AddFromAssemblyOf<ProductMap>()
            .Conventions.Add<CustomForeignKeyConvention>())
        .BuildSessionFactory();
}
```

### Convention Grupları

İlgili convention'ları gruplamak için convention grupları kullanabilirsiniz:

```csharp
// Convention grubu örneği
public class MyConventions : ConventionGroup
{
    public MyConventions()
    {
        // ID convention'ı
        Add(ForeignKey.EndsWith("Id"));
        
        // Tablo adı convention'ı
        Add(Table.Is(x => Inflector.Pluralize(x.EntityType.Name)));
        
        // Sütun adı convention'ı
        Add(DefaultLazy.Always());
        
        // Özel convention
        Add<CustomForeignKeyConvention>();
    }
}

// Convention grubu kullanımı
public static ISessionFactory CreateSessionFactory()
{
    return Fluently.Configure()
        .Database(MsSqlConfiguration.MsSql2012
            .ConnectionString("Server=localhost;Database=MyDatabase;User Id=sa;Password=P@ssw0rd;"))
        .Mappings(m => m.FluentMappings
            .AddFromAssemblyOf<ProductMap>()
            .Conventions.Add<MyConventions>())
        .BuildSessionFactory();
}
```

## Özet

Bu bölümde, NHibernate'in temel kavramlarını ve kullanımını inceledik:

- **Session Factory ve Session**: NHibernate'in çalışma mantığı ve veritabanı işlemlerini yönetme
- **Mapping Stratejileri**: Nesneleri veritabanı tablolarına eşleme yöntemleri
- **Configuration**: NHibernate'i yapılandırma seçenekleri
- **Fluent NHibernate**: Kod tabanlı mapping tanımlama
- **Conventions**: Tekrarlayan mapping kodunu azaltma

NHibernate, güçlü ve esnek bir ORM çözümüdür. Doğru şekilde kullanıldığında, veritabanı işlemlerini büyük ölçüde basitleştirir ve nesne yönelimli programlama ile ilişkisel veritabanları arasındaki uyumsuzluğu giderir. 