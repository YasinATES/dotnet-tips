# Mapping Techniques

NHibernate, nesne yönelimli programlama modeli ile ilişkisel veritabanı modeli arasındaki uyumsuzluğu gidermek için çeşitli mapping teknikleri sunar. Bu bölümde, NHibernate'in desteklediği farklı mapping yaklaşımlarını inceleyeceğiz.

## XML Mapping

XML mapping, NHibernate'in geleneksel ve en yaygın kullanılan mapping yaklaşımıdır. Bu yaklaşımda, her entity sınıfı için bir `.hbm.xml` uzantılı XML dosyası oluşturulur.

### Temel XML Mapping

Aşağıda, basit bir `Product` sınıfı için XML mapping örneği verilmiştir:

```xml
<?xml version="1.0" encoding="utf-8" ?>
<hibernate-mapping xmlns="urn:nhibernate-mapping-2.2">
  <class name="MyApp.Domain.Product, MyApp.Domain" table="Products">
    <!-- ID mapping -->
    <id name="Id" column="Id">
      <generator class="native" />
    </id>
    
    <!-- Basit özellikler -->
    <property name="Name" column="Name" not-null="true" length="100" />
    <property name="Price" column="Price" not-null="true" />
    <property name="Stock" column="Stock" not-null="true" />
    <property name="CreatedAt" column="CreatedAt" />
    
    <!-- Kategori ilişkisi -->
    <many-to-one name="Category" column="CategoryId" class="MyApp.Domain.Category, MyApp.Domain" />
  </class>
</hibernate-mapping>
```

### XML Mapping Dosyalarını Yapılandırma

XML mapping dosyalarını NHibernate yapılandırmasına eklemek için aşağıdaki yöntemler kullanılabilir:

```csharp
// XML mapping dosyalarını ekleme
var configuration = new Configuration();

// Yöntem 1: Tek bir dosya ekleme
configuration.AddFile("Product.hbm.xml");

// Yöntem 2: Bir dizindeki tüm dosyaları ekleme
configuration.AddDirectory(new DirectoryInfo("Mappings"));

// Yöntem 3: Bir assembly'deki gömülü kaynakları ekleme
configuration.AddAssembly(typeof(Product).Assembly);
```

### İlişki Mapping

XML mapping ile farklı ilişki türlerini tanımlamak mümkündür:

```xml
<!-- One-to-Many ilişkisi -->
<class name="MyApp.Domain.Category, MyApp.Domain" table="Categories">
  <id name="Id" column="Id">
    <generator class="native" />
  </id>
  <property name="Name" column="Name" not-null="true" />
  
  <!-- Ürünler koleksiyonu -->
  <bag name="Products" inverse="true" cascade="all-delete-orphan">
    <key column="CategoryId" />
    <one-to-many class="MyApp.Domain.Product, MyApp.Domain" />
  </bag>
</class>

<!-- Many-to-Many ilişkisi -->
<class name="MyApp.Domain.Product, MyApp.Domain" table="Products">
  <!-- Diğer mapping'ler... -->
  
  <!-- Etiketler koleksiyonu -->
  <bag name="Tags" table="ProductTags">
    <key column="ProductId" />
    <many-to-many class="MyApp.Domain.Tag, MyApp.Domain" column="TagId" />
  </bag>
</class>
```

### Kalıtım Mapping

XML mapping ile kalıtım hiyerarşilerini farklı stratejilerle eşlemek mümkündür:

```xml
<!-- Table-per-class-hierarchy stratejisi -->
<class name="MyApp.Domain.Person, MyApp.Domain" table="People" discriminator-value="P">
  <id name="Id" column="Id">
    <generator class="native" />
  </id>
  <discriminator column="Type" type="string" />
  <property name="Name" column="Name" not-null="true" />
  
  <subclass name="MyApp.Domain.Employee, MyApp.Domain" discriminator-value="E">
    <property name="Salary" column="Salary" />
    <property name="Department" column="Department" />
  </subclass>
  
  <subclass name="MyApp.Domain.Customer, MyApp.Domain" discriminator-value="C">
    <property name="CompanyName" column="CompanyName" />
    <property name="ContactNumber" column="ContactNumber" />
  </subclass>
</class>

<!-- Table-per-subclass stratejisi -->
<class name="MyApp.Domain.Person, MyApp.Domain" table="People">
  <id name="Id" column="Id">
    <generator class="native" />
  </id>
  <property name="Name" column="Name" not-null="true" />
  
  <joined-subclass name="MyApp.Domain.Employee, MyApp.Domain" table="Employees">
    <key column="PersonId" />
    <property name="Salary" column="Salary" />
    <property name="Department" column="Department" />
  </joined-subclass>
  
  <joined-subclass name="MyApp.Domain.Customer, MyApp.Domain" table="Customers">
    <key column="PersonId" />
    <property name="CompanyName" column="CompanyName" />
    <property name="ContactNumber" column="ContactNumber" />
  </joined-subclass>
</class>
```

## Fluent Mapping

Fluent mapping, XML dosyaları yerine C# kodu kullanarak mapping tanımlamayı sağlayan bir yaklaşımdır. Bu yaklaşım, Fluent NHibernate kütüphanesi aracılığıyla kullanılır.

### Kurulum

Fluent NHibernate'i kullanmak için öncelikle NuGet üzerinden kurmanız gerekir:

```csharp
// NuGet üzerinden kurulum
Install-Package FluentNHibernate
```

### Temel Fluent Mapping

Fluent mapping için, her entity sınıfı için bir mapping sınıfı oluşturulur:

```csharp
// Entity sınıfı
public class Product
{
    public virtual int Id { get; set; }
    public virtual string Name { get; set; }
    public virtual decimal Price { get; set; }
    public virtual int Stock { get; set; }
    public virtual DateTime CreatedAt { get; set; }
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
        Map(x => x.CreatedAt);
        
        // İlişkileri tanımlama
        References(x => x.Category).Column("CategoryId");
    }
}
```

### Fluent Mapping ile Session Factory Oluşturma

Fluent mapping ile session factory oluşturmak için `FluentConfiguration` kullanılır:

```csharp
// Fluent mapping ile session factory oluşturma
public static ISessionFactory CreateSessionFactory()
{
    return Fluently.Configure()
        .Database(MsSqlConfiguration.MsSql2012
            .ConnectionString("Server=localhost;Database=MyDatabase;User Id=sa;Password=P@ssw0rd;")
            .ShowSql())
        .Mappings(m => m.FluentMappings
            .AddFromAssemblyOf<ProductMap>())
        .BuildSessionFactory();
}
```

### İlişki Mapping

Fluent mapping ile farklı ilişki türlerini tanımlamak mümkündür:

```csharp
// One-to-Many ilişkisi
public class CategoryMap : ClassMap<Category>
{
    public CategoryMap()
    {
        Table("Categories");
        Id(x => x.Id).GeneratedBy.Native();
        Map(x => x.Name).Length(50).Not.Nullable();
        
        // Ürünler koleksiyonu
        HasMany(x => x.Products)
            .KeyColumn("CategoryId")
            .Inverse()
            .Cascade.AllDeleteOrphan();
    }
}

// Many-to-Many ilişkisi
public class ProductMap : ClassMap<Product>
{
    public ProductMap()
    {
        // Diğer mapping'ler...
        
        // Etiketler koleksiyonu
        HasManyToMany(x => x.Tags)
            .Table("ProductTags")
            .ParentKeyColumn("ProductId")
            .ChildKeyColumn("TagId")
            .Cascade.All();
    }
}
```

### Kalıtım Mapping

Fluent mapping ile kalıtım hiyerarşilerini farklı stratejilerle eşlemek mümkündür:

```csharp
// Table-per-class-hierarchy stratejisi
public class PersonMap : ClassMap<Person>
{
    public PersonMap()
    {
        Table("People");
        Id(x => x.Id).GeneratedBy.Native();
        Map(x => x.Name).Not.Nullable();
        
        // Discriminator sütunu
        DiscriminateSubClassesOnColumn("Type")
            .AlwaysSelectWithValue();
    }
}

public class EmployeeMap : SubclassMap<Employee>
{
    public EmployeeMap()
    {
        DiscriminatorValue("E");
        Map(x => x.Salary);
        Map(x => x.Department);
    }
}

public class CustomerMap : SubclassMap<Customer>
{
    public CustomerMap()
    {
        DiscriminatorValue("C");
        Map(x => x.CompanyName);
        Map(x => x.ContactNumber);
    }
}

// Table-per-subclass stratejisi
public class PersonMap : ClassMap<Person>
{
    public PersonMap()
    {
        Table("People");
        Id(x => x.Id).GeneratedBy.Native();
        Map(x => x.Name).Not.Nullable();
    }
}

public class EmployeeMap : SubclassMap<Employee>
{
    public EmployeeMap()
    {
        Table("Employees");
        KeyColumn("PersonId");
        Map(x => x.Salary);
        Map(x => x.Department);
    }
}

public class CustomerMap : SubclassMap<Customer>
{
    public CustomerMap()
    {
        Table("Customers");
        KeyColumn("PersonId");
        Map(x => x.CompanyName);
        Map(x => x.ContactNumber);
    }
}
```

## Attribute-based Mapping

Attribute-based mapping, entity sınıflarını öznitelikler (attributes) kullanarak doğrudan işaretlemeyi sağlayan bir yaklaşımdır. Bu yaklaşım, NHibernate.Mapping.Attributes kütüphanesi aracılığıyla kullanılır.

### Kurulum

NHibernate.Mapping.Attributes'u kullanmak için öncelikle NuGet üzerinden kurmanız gerekir:

```csharp
// NuGet üzerinden kurulum
Install-Package NHibernate.Mapping.Attributes
```

### Temel Attribute Mapping

Attribute mapping için, entity sınıfları özniteliklerle işaretlenir:

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
    
    [Property(Column = "CreatedAt")]
    public virtual DateTime CreatedAt { get; set; }
    
    [ManyToOne(Column = "CategoryId", ClassType = typeof(Category))]
    public virtual Category Category { get; set; }
}
```

### Attribute Mapping ile Configuration

Attribute mapping'leri NHibernate yapılandırmasına eklemek için `HbmSerializer` kullanılır:

```csharp
// Attribute mapping ile configuration
var configuration = new Configuration();
configuration.AddInputStream(
    HbmSerializer.Default.Serialize(
        Assembly.GetExecutingAssembly()
    )
);
```

### İlişki Mapping

Attribute mapping ile farklı ilişki türlerini tanımlamak mümkündür:

```csharp
// One-to-Many ilişkisi
[Class(Table = "Categories")]
public class Category
{
    [Id(Name = "Id", Column = "Id")]
    [Generator(Class = "native")]
    public virtual int Id { get; set; }
    
    [Property(Column = "Name", NotNull = true)]
    public virtual string Name { get; set; }
    
    [Bag(0, Inverse = true, Cascade = "all-delete-orphan")]
    [Key(1, Column = "CategoryId")]
    [OneToMany(2, ClassType = typeof(Product))]
    public virtual IList<Product> Products { get; set; }
}

// Many-to-Many ilişkisi
[Class(Table = "Products")]
public class Product
{
    // Diğer öznitelikler...
    
    [Bag(0, Table = "ProductTags")]
    [Key(1, Column = "ProductId")]
    [ManyToMany(2, ClassType = typeof(Tag), Column = "TagId")]
    public virtual IList<Tag> Tags { get; set; }
}
```

## Auto Mapping

Auto mapping, convention-over-configuration prensibine dayalı olarak, minimal yapılandırma ile entity sınıflarını otomatik olarak eşlemeyi sağlayan bir yaklaşımdır. Bu yaklaşım, Fluent NHibernate'in AutoMapping özelliği aracılığıyla kullanılır.

### Temel Auto Mapping

Auto mapping için, entity sınıflarınızın belirli kurallara uyması gerekir:

```csharp
// Auto mapping için entity sınıfları
public class Entity
{
    public virtual int Id { get; set; }
}

public class Product : Entity
{
    public virtual string Name { get; set; }
    public virtual decimal Price { get; set; }
    public virtual int Stock { get; set; }
    public virtual DateTime CreatedAt { get; set; }
    public virtual Category Category { get; set; }
}

public class Category : Entity
{
    public virtual string Name { get; set; }
    public virtual IList<Product> Products { get; set; }
}
```

### Auto Mapping Yapılandırma

Auto mapping'i yapılandırmak için `AutoPersistenceModel` kullanılır:

```csharp
// Auto mapping yapılandırma
public static ISessionFactory CreateSessionFactory()
{
    return Fluently.Configure()
        .Database(MsSqlConfiguration.MsSql2012
            .ConnectionString("Server=localhost;Database=MyDatabase;User Id=sa;Password=P@ssw0rd;"))
        .Mappings(m => m.AutoMappings.Add(
            AutoMap.AssemblyOf<Entity>()
                // Temel sınıfı hariç tutma
                .Where(t => t != typeof(Entity))
                // ID convention'ı
                .Override<Entity>(map => map.Id(x => x.Id).GeneratedBy.Native())
                // Özel entity override'ları
                .Override<Product>(map =>
                {
                    map.Map(x => x.Name).Length(100).Not.Nullable();
                    map.Map(x => x.Price).Not.Nullable();
                    map.References(x => x.Category).Column("CategoryId");
                })
        ))
        .BuildSessionFactory();
}
```

### Auto Mapping Convention'ları

Auto mapping'in davranışını özelleştirmek için convention'lar kullanılabilir:

```csharp
// Auto mapping convention'ları
public class StoreConfiguration : DefaultAutomappingConfiguration
{
    public override bool ShouldMap(Type type)
    {
        // Entity sınıflarını belirleme
        return type.Namespace == "MyApp.Domain" && type != typeof(Entity);
    }
    
    public override bool IsId(Member member)
    {
        // ID özelliklerini belirleme
        return member.Name == "Id";
    }
    
    public override bool IsComponent(Type type)
    {
        // Component türlerini belirleme
        return type.Namespace == "MyApp.Domain.Components";
    }
}

// Convention'ları kullanma
public static ISessionFactory CreateSessionFactory()
{
    return Fluently.Configure()
        .Database(MsSqlConfiguration.MsSql2012
            .ConnectionString("Server=localhost;Database=MyDatabase;User Id=sa;Password=P@ssw0rd;"))
        .Mappings(m => m.AutoMappings.Add(
            AutoMap.AssemblyOf<Entity>(new StoreConfiguration())
        ))
        .BuildSessionFactory();
}
```

## Component Mapping

Component mapping, değer nesnelerini (value objects) entity'lerin içinde gömülü olarak eşlemeyi sağlayan bir yaklaşımdır. Bu, domain-driven design prensiplerini uygulamak için özellikle faydalıdır.

### XML Component Mapping

XML ile component mapping örneği:

```xml
<class name="MyApp.Domain.Order, MyApp.Domain" table="Orders">
  <id name="Id" column="Id">
    <generator class="native" />
  </id>
  <property name="OrderDate" column="OrderDate" not-null="true" />
  
  <!-- Adres component'i -->
  <component name="ShippingAddress" class="MyApp.Domain.Address, MyApp.Domain">
    <property name="Street" column="ShippingStreet" not-null="true" />
    <property name="City" column="ShippingCity" not-null="true" />
    <property name="PostalCode" column="ShippingPostalCode" not-null="true" />
    <property name="Country" column="ShippingCountry" not-null="true" />
  </component>
  
  <!-- Para birimi component'i -->
  <component name="TotalAmount" class="MyApp.Domain.Money, MyApp.Domain">
    <property name="Amount" column="TotalAmount" not-null="true" />
    <property name="Currency" column="Currency" not-null="true" length="3" />
  </component>
</class>
```

### Fluent Component Mapping

Fluent API ile component mapping örneği:

```csharp
// Component sınıfları
public class Address
{
    public virtual string Street { get; set; }
    public virtual string City { get; set; }
    public virtual string PostalCode { get; set; }
    public virtual string Country { get; set; }
}

public class Money
{
    public virtual decimal Amount { get; set; }
    public virtual string Currency { get; set; }
}

// Entity sınıfı
public class Order
{
    public virtual int Id { get; set; }
    public virtual DateTime OrderDate { get; set; }
    public virtual Address ShippingAddress { get; set; }
    public virtual Money TotalAmount { get; set; }
}

// Mapping sınıfı
public class OrderMap : ClassMap<Order>
{
    public OrderMap()
    {
        Table("Orders");
        Id(x => x.Id).GeneratedBy.Native();
        Map(x => x.OrderDate).Not.Nullable();
        
        // Adres component'i
        Component(x => x.ShippingAddress, c =>
        {
            c.Map(x => x.Street, "ShippingStreet").Not.Nullable();
            c.Map(x => x.City, "ShippingCity").Not.Nullable();
            c.Map(x => x.PostalCode, "ShippingPostalCode").Not.Nullable();
            c.Map(x => x.Country, "ShippingCountry").Not.Nullable();
        });
        
        // Para birimi component'i
        Component(x => x.TotalAmount, c =>
        {
            c.Map(x => x.Amount, "TotalAmount").Not.Nullable();
            c.Map(x => x.Currency, "Currency").Not.Nullable().Length(3);
        });
    }
}
```

### Attribute Component Mapping

Attribute ile component mapping örneği:

```csharp
[Class(Table = "Orders")]
public class Order
{
    [Id(Name = "Id", Column = "Id")]
    [Generator(Class = "native")]
    public virtual int Id { get; set; }
    
    [Property(Column = "OrderDate", NotNull = true)]
    public virtual DateTime OrderDate { get; set; }
    
    [Component(Name = "ShippingAddress")]
    public virtual Address ShippingAddress { get; set; }
    
    [Component(Name = "TotalAmount")]
    public virtual Money TotalAmount { get; set; }
}

public class Address
{
    [Property(Column = "ShippingStreet", NotNull = true)]
    public virtual string Street { get; set; }
    
    [Property(Column = "ShippingCity", NotNull = true)]
    public virtual string City { get; set; }
    
    [Property(Column = "ShippingPostalCode", NotNull = true)]
    public virtual string PostalCode { get; set; }
    
    [Property(Column = "ShippingCountry", NotNull = true)]
    public virtual string Country { get; set; }
}

public class Money
{
    [Property(Column = "TotalAmount", NotNull = true)]
    public virtual decimal Amount { get; set; }
    
    [Property(Column = "Currency", NotNull = true, Length = 3)]
    public virtual string Currency { get; set; }
}
```

### Component Koleksiyonları

Component'lerin koleksiyonlarını da eşlemek mümkündür:

```csharp
// Component koleksiyonu
public class OrderLine
{
    public virtual string ProductName { get; set; }
    public virtual int Quantity { get; set; }
    public virtual Money UnitPrice { get; set; }
}

// Entity sınıfı
public class Order
{
    public virtual int Id { get; set; }
    public virtual DateTime OrderDate { get; set; }
    public virtual IList<OrderLine> Lines { get; set; }
}

// Fluent mapping
public class OrderMap : ClassMap<Order>
{
    public OrderMap()
    {
        Table("Orders");
        Id(x => x.Id).GeneratedBy.Native();
        Map(x => x.OrderDate).Not.Nullable();
        
        // Component koleksiyonu
        HasMany(x => x.Lines).Component(c =>
        {
            c.Map(x => x.ProductName).Column("ProductName").Not.Nullable();
            c.Map(x => x.Quantity).Column("Quantity").Not.Nullable();
            c.Component(x => x.UnitPrice, m =>
            {
                m.Map(x => x.Amount).Column("UnitPrice").Not.Nullable();
                m.Map(x => x.Currency).Column("Currency").Not.Nullable().Length(3);
            });
        }).Table("OrderLines").KeyColumn("OrderId");
    }
}
```

## Özet

Bu bölümde, NHibernate'in desteklediği farklı mapping tekniklerini inceledik:

- **XML Mapping**: Geleneksel ve en yaygın kullanılan mapping yaklaşımı
- **Fluent Mapping**: C# kodu kullanarak mapping tanımlamayı sağlayan yaklaşım
- **Attribute-based Mapping**: Entity sınıflarını özniteliklerle işaretlemeyi sağlayan yaklaşım
- **Auto Mapping**: Convention-over-configuration prensibine dayalı otomatik mapping yaklaşımı
- **Component Mapping**: Değer nesnelerini entity'lerin içinde gömülü olarak eşlemeyi sağlayan yaklaşım

Her mapping tekniğinin kendi avantajları ve dezavantajları vardır. Projenizin gereksinimlerine ve ekibinizin tercihlerine göre en uygun tekniği seçebilirsiniz. Genellikle, modern NHibernate uygulamalarında Fluent Mapping tercih edilir, çünkü tip güvenliği sağlar ve refactoring'i kolaylaştırır. 