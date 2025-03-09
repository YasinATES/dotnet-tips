# Querying

NHibernate, veritabanı sorgulama işlemleri için çeşitli API'ler sunar. Bu API'ler, farklı karmaşıklık seviyelerindeki sorguları oluşturmanıza ve çalıştırmanıza olanak tanır. Bu bölümde, NHibernate'in sunduğu sorgulama seçeneklerini inceleyeceğiz.

## HQL (Hibernate Query Language)

HQL, NHibernate'in kendi sorgu dilidir. SQL'e benzer bir sözdizimi vardır, ancak tablolar ve sütunlar yerine entity sınıfları ve özellikleri üzerinde çalışır. HQL, veritabanı bağımsız olduğundan, farklı veritabanı sistemleri arasında taşınabilir sorgular yazmanıza olanak tanır.

### Temel HQL Sorguları

```csharp
// Basit HQL sorgusu
using (var session = sessionFactory.OpenSession())
{
    // Tüm ürünleri getirme
    var products = session.CreateQuery("from Product")
                         .List<Product>();
    
    // Filtreleme ile ürünleri getirme
    var expensiveProducts = session.CreateQuery("from Product p where p.Price > :minPrice")
                                  .SetParameter("minPrice", 100m)
                                  .List<Product>();
    
    // Sıralama ile ürünleri getirme
    var sortedProducts = session.CreateQuery("from Product p order by p.Name asc")
                               .List<Product>();
}
```

### Projection ve Aggregation

HQL, projection (belirli sütunları seçme) ve aggregation (toplama işlemleri) destekler:

```csharp
// Projection örneği
using (var session = sessionFactory.OpenSession())
{
    // Sadece belirli özellikleri getirme
    var productNames = session.CreateQuery("select p.Name from Product p")
                             .List<string>();
    
    // Anonim nesneler oluşturma
    var productSummaries = session.CreateQuery("select new map(p.Id as Id, p.Name as Name, p.Price as Price) from Product p")
                                 .SetResultTransformer(Transformers.AliasToBean<ProductSummary>())
                                 .List<ProductSummary>();
    
    // Aggregation fonksiyonları
    var averagePrice = session.CreateQuery("select avg(p.Price) from Product p")
                             .UniqueResult<decimal>();
    
    var categoryCounts = session.CreateQuery("select p.Category.Name, count(p) from Product p group by p.Category.Name")
                               .List<object[]>();
}
```

### Join İşlemleri

HQL, farklı join türlerini destekler:

```csharp
// Join örnekleri
using (var session = sessionFactory.OpenSession())
{
    // Inner join
    var productsWithCategories = session.CreateQuery("from Product p join p.Category c")
                                       .List<object[]>();
    
    // Left outer join
    var productsWithOptionalSuppliers = session.CreateQuery("from Product p left join p.Supplier s")
                                              .List<object[]>();
    
    // Fetch join (N+1 sorunu çözümü)
    var productsWithCategoriesFetched = session.CreateQuery("from Product p join fetch p.Category")
                                              .List<Product>();
}
```

### Subquery ve Exists

HQL, subquery ve exists ifadelerini destekler:

```csharp
// Subquery ve exists örnekleri
using (var session = sessionFactory.OpenSession())
{
    // Subquery
    var productsInTopCategories = session.CreateQuery(@"
        from Product p 
        where p.Category.Id in (
            select c.Id from Category c where c.Featured = true
        )")
        .List<Product>();
    
    // Exists
    var productsWithOrders = session.CreateQuery(@"
        from Product p 
        where exists (
            select o from Order o join o.Items i where i.Product = p
        )")
        .List<Product>();
}
```

### Named Queries

Sık kullanılan sorguları entity mapping dosyalarında tanımlayabilir ve isimle çağırabilirsiniz:

```xml
<!-- XML mapping dosyasında named query tanımlama -->
<hibernate-mapping>
  <class name="Product" table="Products">
    <!-- Mapping tanımları... -->
    
    <query name="ProductsByCategory"><![CDATA[
      from Product p
      where p.Category.Id = :categoryId
      order by p.Name
    ]]></query>
    
    <query name="ProductsWithLowStock"><![CDATA[
      from Product p
      where p.Stock < :threshold
      order by p.Stock
    ]]></query>
  </class>
</hibernate-mapping>
```

```csharp
// Named query kullanımı
using (var session = sessionFactory.OpenSession())
{
    // Named query çağırma
    var products = session.GetNamedQuery("ProductsByCategory")
                         .SetParameter("categoryId", 1)
                         .List<Product>();
    
    var lowStockProducts = session.GetNamedQuery("ProductsWithLowStock")
                                 .SetParameter("threshold", 10)
                                 .List<Product>();
}
```

## Criteria API

Criteria API, sorguları programatik olarak oluşturmanıza olanak tanıyan bir API'dir. String tabanlı HQL sorgularının aksine, Criteria API tip güvenliği sağlar ve refactoring'i kolaylaştırır.

### Temel Criteria Sorguları

```csharp
// Temel Criteria sorguları
using (var session = sessionFactory.OpenSession())
{
    // Tüm ürünleri getirme
    var products = session.CreateCriteria<Product>()
                         .List<Product>();
    
    // Filtreleme ile ürünleri getirme
    var expensiveProducts = session.CreateCriteria<Product>()
                                  .Add(Restrictions.Gt("Price", 100m))
                                  .List<Product>();
    
    // Sıralama ile ürünleri getirme
    var sortedProducts = session.CreateCriteria<Product>()
                               .AddOrder(Order.Asc("Name"))
                               .List<Product>();
}
```

### Projection ve Aggregation

Criteria API, projection ve aggregation işlemleri için `Projections` sınıfını kullanır:

```csharp
// Projection ve aggregation örnekleri
using (var session = sessionFactory.OpenSession())
{
    // Sadece belirli özellikleri getirme
    var productNames = session.CreateCriteria<Product>()
                             .SetProjection(Projections.Property("Name"))
                             .List<string>();
    
    // Birden fazla özellik getirme
    var productSummaries = session.CreateCriteria<Product>()
                                 .SetProjection(Projections.ProjectionList()
                                     .Add(Projections.Property("Id"), "Id")
                                     .Add(Projections.Property("Name"), "Name")
                                     .Add(Projections.Property("Price"), "Price"))
                                 .SetResultTransformer(Transformers.AliasToBean<ProductSummary>())
                                 .List<ProductSummary>();
    
    // Aggregation fonksiyonları
    var averagePrice = session.CreateCriteria<Product>()
                             .SetProjection(Projections.Avg("Price"))
                             .UniqueResult<decimal>();
    
    // Group by
    var categoryCounts = session.CreateCriteria<Product>()
                               .CreateAlias("Category", "c")
                               .SetProjection(Projections.ProjectionList()
                                   .Add(Projections.GroupProperty("c.Name"))
                                   .Add(Projections.RowCount()))
                               .List<object[]>();
}
```

### Join İşlemleri

Criteria API, join işlemleri için `CreateAlias` veya `CreateCriteria` metodlarını kullanır:

```csharp
// Join örnekleri
using (var session = sessionFactory.OpenSession())
{
    // Inner join
    var productsWithCategories = session.CreateCriteria<Product>()
                                       .CreateAlias("Category", "c")
                                       .List<Product>();
    
    // Left outer join
    var productsWithOptionalSuppliers = session.CreateCriteria<Product>()
                                              .CreateAlias("Supplier", "s", JoinType.LeftOuterJoin)
                                              .List<Product>();
    
    // Fetch join
    var productsWithCategoriesFetched = session.CreateCriteria<Product>()
                                              .Fetch(SelectMode.Fetch, "Category")
                                              .List<Product>();
}
```

### Detached Criteria

Detached Criteria, sorgu tanımını session'dan bağımsız olarak oluşturmanıza olanak tanır:

```csharp
// Detached Criteria örneği
// Sorgu tanımını oluşturma
var productsByCategoryQuery = DetachedCriteria.For<Product>()
    .Add(Restrictions.Eq("Category.Id", 0)) // Placeholder değer
    .AddOrder(Order.Asc("Name"));

// Sorguyu çalıştırma
using (var session = sessionFactory.OpenSession())
{
    // Placeholder değeri gerçek değerle değiştirme
    var products = productsByCategoryQuery
        .GetExecutableCriteria(session)
        .SetParameter(0, 1) // Gerçek kategori ID'si
        .List<Product>();
}
```

## LINQ to NHibernate

LINQ to NHibernate, LINQ (Language Integrated Query) sözdizimini kullanarak NHibernate sorguları yazmanıza olanak tanır. Bu, C# dilinin tüm gücünü ve IntelliSense desteğini sorgu yazarken kullanmanızı sağlar.

### Kurulum

LINQ to NHibernate'i kullanmak için NHibernate.Linq paketini kurmanız gerekir:

```csharp
// NuGet üzerinden kurulum
Install-Package NHibernate.Linq
```

### Temel LINQ Sorguları

```csharp
// Temel LINQ sorguları
using (var session = sessionFactory.OpenSession())
{
    // IQueryable<T> elde etme
    var query = session.Query<Product>();
    
    // Tüm ürünleri getirme
    var products = query.ToList();
    
    // Filtreleme ile ürünleri getirme
    var expensiveProducts = query.Where(p => p.Price > 100m).ToList();
    
    // Sıralama ile ürünleri getirme
    var sortedProducts = query.OrderBy(p => p.Name).ToList();
    
    // Filtreleme ve sıralama birlikte
    var filteredAndSorted = query
        .Where(p => p.Stock > 0)
        .OrderByDescending(p => p.Price)
        .ThenBy(p => p.Name)
        .ToList();
}
```

### Projection ve Aggregation

LINQ, projection ve aggregation işlemleri için zengin bir sözdizimi sunar:

```csharp
// Projection ve aggregation örnekleri
using (var session = sessionFactory.OpenSession())
{
    var query = session.Query<Product>();
    
    // Sadece belirli özellikleri getirme
    var productNames = query.Select(p => p.Name).ToList();
    
    // Anonim nesneler oluşturma
    var productSummaries = query
        .Select(p => new ProductSummary
        {
            Id = p.Id,
            Name = p.Name,
            Price = p.Price
        })
        .ToList();
    
    // Aggregation fonksiyonları
    var averagePrice = query.Average(p => p.Price);
    var totalProducts = query.Count();
    var maxPrice = query.Max(p => p.Price);
    
    // Group by
    var categoryCounts = query
        .GroupBy(p => p.Category.Name)
        .Select(g => new
        {
            CategoryName = g.Key,
            ProductCount = g.Count()
        })
        .ToList();
}
```

### Join İşlemleri

LINQ, join işlemleri için çeşitli yöntemler sunar:

```csharp
// Join örnekleri
using (var session = sessionFactory.OpenSession())
{
    // Navigation property kullanarak join
    var productsWithCategories = session.Query<Product>()
        .Select(p => new
        {
            Product = p,
            CategoryName = p.Category.Name
        })
        .ToList();
    
    // Explicit join
    var productCategoryJoin = session.Query<Product>()
        .Join(
            session.Query<Category>(),
            p => p.Category.Id,
            c => c.Id,
            (p, c) => new
            {
                Product = p,
                Category = c
            })
        .ToList();
    
    // Left join
    var productSupplierLeftJoin = session.Query<Product>()
        .GroupJoin(
            session.Query<Supplier>(),
            p => p.SupplierId,
            s => s.Id,
            (p, suppliers) => new
            {
                Product = p,
                Suppliers = suppliers
            })
        .SelectMany(
            x => x.Suppliers.DefaultIfEmpty(),
            (x, s) => new
            {
                Product = x.Product,
                Supplier = s
            })
        .ToList();
}
```

### Eager Loading

LINQ to NHibernate, ilişkili nesneleri eager loading ile yüklemenize olanak tanır:

```csharp
// Eager loading örnekleri
using (var session = sessionFactory.OpenSession())
{
    // Fetch ile eager loading
    var productsWithCategories = session.Query<Product>()
        .Fetch(p => p.Category)
        .ToList();
    
    // Çoklu fetch
    var ordersWithDetails = session.Query<Order>()
        .Fetch(o => o.Customer)
        .FetchMany(o => o.Items)
        .ThenFetch(i => i.Product)
        .ToList();
}
```

## QueryOver API

QueryOver API, Criteria API'nin daha modern ve tip güvenli bir versiyonudur. LINQ benzeri bir sözdizimi kullanır, ancak NHibernate'in kendi sorgu altyapısını kullanır.

### Temel QueryOver Sorguları

```csharp
// Temel QueryOver sorguları
using (var session = sessionFactory.OpenSession())
{
    // Tüm ürünleri getirme
    var products = session.QueryOver<Product>()
                         .List();
    
    // Filtreleme ile ürünleri getirme
    var expensiveProducts = session.QueryOver<Product>()
                                  .Where(p => p.Price > 100m)
                                  .List();
    
    // Sıralama ile ürünleri getirme
    var sortedProducts = session.QueryOver<Product>()
                               .OrderBy(p => p.Name).Asc
                               .List();
}
```

### Lambda Expressions ile Filtreleme

QueryOver API, lambda expressions kullanarak tip güvenli filtreleme sağlar:

```csharp
// Lambda expressions ile filtreleme
using (var session = sessionFactory.OpenSession())
{
    // Lambda ile filtreleme
    var products = session.QueryOver<Product>()
                         .Where(p => p.Price > 100m && p.Stock > 0)
                         .List();
    
    // Alias kullanarak filtreleme
    Product productAlias = null;
    var productsWithAlias = session.QueryOver<Product>(() => productAlias)
                                  .Where(() => productAlias.Price > 100m)
                                  .List();
    
    // Restrictions kullanarak filtreleme
    var productsWithRestrictions = session.QueryOver<Product>()
                                         .Where(Restrictions.Gt("Price", 100m))
                                         .And(Restrictions.Gt("Stock", 0))
                                         .List();
}
```

### Join İşlemleri

QueryOver API, join işlemleri için çeşitli yöntemler sunar:

```csharp
// Join örnekleri
using (var session = sessionFactory.OpenSession())
{
    // Inner join
    Category categoryAlias = null;
    var productsWithCategories = session.QueryOver<Product>()
                                       .JoinAlias(p => p.Category, () => categoryAlias)
                                       .List();
    
    // Left outer join
    Supplier supplierAlias = null;
    var productsWithOptionalSuppliers = session.QueryOver<Product>()
                                              .Left.JoinAlias(p => p.Supplier, () => supplierAlias)
                                              .List();
    
    // Fetch join
    var productsWithCategoriesFetched = session.QueryOver<Product>()
                                              .Fetch(p => p.Category).Eager
                                              .List();
}
```

### Projection ve Aggregation

QueryOver API, projection ve aggregation işlemleri için `Projections` sınıfını kullanır:

```csharp
// Projection ve aggregation örnekleri
using (var session = sessionFactory.OpenSession())
{
    // Sadece belirli özellikleri getirme
    var productNames = session.QueryOver<Product>()
                             .Select(p => p.Name)
                             .List<string>();
    
    // Birden fazla özellik getirme
    var productSummaries = session.QueryOver<Product>()
                                 .SelectList(list => list
                                     .Select(p => p.Id).WithAlias(() => productSummaryAlias.Id)
                                     .Select(p => p.Name).WithAlias(() => productSummaryAlias.Name)
                                     .Select(p => p.Price).WithAlias(() => productSummaryAlias.Price))
                                 .TransformUsing(Transformers.AliasToBean<ProductSummary>())
                                 .List<ProductSummary>();
    
    // Aggregation fonksiyonları
    var averagePrice = session.QueryOver<Product>()
                             .Select(Projections.Avg<Product>(p => p.Price))
                             .SingleOrDefault<decimal>();
    
    // Group by
    Category categoryAlias = null;
    var categoryCounts = session.QueryOver<Product>()
                               .JoinAlias(p => p.Category, () => categoryAlias)
                               .SelectList(list => list
                                   .SelectGroup(() => categoryAlias.Name)
                                   .SelectCount(p => p.Id))
                               .List<object[]>();
}
```

## Native SQL

Bazen, NHibernate'in sağladığı sorgulama API'leri yeterli olmayabilir veya veritabanına özgü özellikleri kullanmanız gerekebilir. Bu durumlarda, native SQL sorguları kullanabilirsiniz.

### Temel SQL Sorguları

```csharp
// Temel SQL sorguları
using (var session = sessionFactory.OpenSession())
{
    // Entity sınıfına eşleme
    var products = session.CreateSQLQuery("SELECT * FROM Products")
                         .AddEntity(typeof(Product))
                         .List<Product>();
    
    // Parametreli sorgu
    var expensiveProducts = session.CreateSQLQuery("SELECT * FROM Products WHERE Price > :minPrice")
                                  .AddEntity(typeof(Product))
                                  .SetParameter("minPrice", 100m)
                                  .List<Product>();
    
    // Scalar sonuçlar
    var productCount = session.CreateSQLQuery("SELECT COUNT(*) FROM Products")
                             .UniqueResult<long>();
}
```

### Karmaşık SQL Sorguları

```csharp
// Karmaşık SQL sorguları
using (var session = sessionFactory.OpenSession())
{
    // Join ile sorgu
    var productsWithCategories = session.CreateSQLQuery(@"
        SELECT p.*, c.* 
        FROM Products p
        INNER JOIN Categories c ON p.CategoryId = c.Id
        WHERE p.Price > :minPrice")
        .AddEntity("p", typeof(Product))
        .AddEntity("c", typeof(Category))
        .SetParameter("minPrice", 100m)
        .List<object[]>();
    
    // Stored procedure çağırma
    var lowStockProducts = session.CreateSQLQuery("EXEC GetLowStockProducts :threshold")
                                 .AddEntity(typeof(Product))
                                 .SetParameter("threshold", 10)
                                 .List<Product>();
    
    // Özel sonuç dönüştürme
    var productSummaries = session.CreateSQLQuery(@"
        SELECT p.Id, p.Name, p.Price, c.Name AS CategoryName
        FROM Products p
        INNER JOIN Categories c ON p.CategoryId = c.Id")
        .SetResultTransformer(Transformers.AliasToBean<ProductSummary>())
        .List<ProductSummary>();
}
```

### SQL Result Transformers

NHibernate, SQL sorgu sonuçlarını çeşitli şekillerde dönüştürmenize olanak tanır:

```csharp
// SQL result transformers
using (var session = sessionFactory.OpenSession())
{
    // AliasToBean transformer
    var productSummaries = session.CreateSQLQuery(@"
        SELECT 
            p.Id AS Id, 
            p.Name AS Name, 
            p.Price AS Price, 
            c.Name AS CategoryName
        FROM Products p
        INNER JOIN Categories c ON p.CategoryId = c.Id")
        .SetResultTransformer(Transformers.AliasToBean<ProductSummary>())
        .List<ProductSummary>();
    
    // AliasToBeanConstructor transformer
    var constructor = typeof(ProductSummary).GetConstructor(new[] { typeof(int), typeof(string), typeof(decimal), typeof(string) });
    var productSummariesWithConstructor = session.CreateSQLQuery(@"
        SELECT 
            p.Id AS Id, 
            p.Name AS Name, 
            p.Price AS Price, 
            c.Name AS CategoryName
        FROM Products p
        INNER JOIN Categories c ON p.CategoryId = c.Id")
        .SetResultTransformer(Transformers.AliasToBeanConstructor(constructor))
        .List<ProductSummary>();
}
```

## Özet

Bu bölümde, NHibernate'in sunduğu çeşitli sorgulama seçeneklerini inceledik:

- **HQL (Hibernate Query Language)**: NHibernate'in kendi sorgu dili, SQL'e benzer ancak entity sınıfları üzerinde çalışır
- **Criteria API**: Sorguları programatik olarak oluşturmanıza olanak tanıyan API
- **LINQ to NHibernate**: LINQ sözdizimini kullanarak NHibernate sorguları yazmanıza olanak tanır
- **QueryOver API**: Criteria API'nin daha modern ve tip güvenli bir versiyonu
- **Native SQL**: Veritabanına özgü SQL sorguları yazmanıza olanak tanır

Her sorgulama API'sinin kendi avantajları ve dezavantajları vardır. Projenizin gereksinimlerine ve ekibinizin tercihlerine göre en uygun API'yi seçebilirsiniz. Genellikle, modern NHibernate uygulamalarında LINQ to NHibernate tercih edilir, çünkü C# dilinin tüm gücünü ve IntelliSense desteğini sorgu yazarken kullanmanızı sağlar. 