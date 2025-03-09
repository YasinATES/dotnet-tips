# Dapper Extensions

Dapper, temel işlevselliğinin yanı sıra, çeşitli eklentiler ve uzantılar aracılığıyla daha da genişletilebilir. Bu eklentiler, Dapper'ın basitliğini ve performansını korurken, daha karmaşık senaryolarda kullanılabilecek ek özellikler sunar. Bu bölümde, Dapper ekosistemindeki popüler uzantıları inceleyeceğiz.

## Dapper.Contrib

Dapper.Contrib, Dapper'ın resmi bir uzantısıdır ve temel CRUD (Create, Read, Update, Delete) işlemleri için basit bir API sağlar. Bu uzantı, SQL sorgularını manuel olarak yazmak zorunda kalmadan, entity sınıfları üzerinden veritabanı işlemlerini gerçekleştirmenize olanak tanır.

### Kurulum

```csharp
// NuGet üzerinden kurulum
Install-Package Dapper.Contrib
```

### Temel Kullanım

Dapper.Contrib kullanımı için, entity sınıflarınızı özel niteliklerle işaretlemeniz gerekir:

```csharp
// Entity sınıfı tanımlama
using Dapper.Contrib.Extensions;

[Table("Products")] // Tablo adını belirtme
public class Product
{
    [Key] // Birincil anahtar
    public int Id { get; set; }
    
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
    
    [Write(false)] // Veritabanına yazılmayacak özellik
    public string DisplayName => $"{Name} ({Price:C})";
    
    [Computed] // Veritabanı tarafından hesaplanan alan (INSERT işlemlerinde hariç tutulur)
    public DateTime CreatedAt { get; set; }
}
```

### CRUD İşlemleri

Dapper.Contrib, aşağıdaki temel CRUD işlemlerini destekler:

```csharp
// Bağlantı oluşturma
using var connection = new SqlConnection(connectionString);

// Tüm ürünleri getirme
var allProducts = connection.GetAll<Product>();

// ID'ye göre ürün getirme
var product = connection.Get<Product>(1);

// Yeni ürün ekleme
var newProduct = new Product
{
    Name = "Yeni Ürün",
    Price = 99.99m,
    CategoryId = 1
};
var id = connection.Insert(newProduct); // Yeni ID döner

// Ürün güncelleme
product.Price = 149.99m;
bool updated = connection.Update(product);

// Ürün silme
bool deleted = connection.Delete(product);

// ID ile silme
bool deleted = connection.Delete<Product>(1);

// Tüm ürünleri silme
bool deleted = connection.DeleteAll<Product>();
```

### Özel Sorgular

Dapper.Contrib, temel CRUD işlemlerinin yanı sıra, özel sorgular için standart Dapper yöntemlerini de destekler:

```csharp
// Özel sorgu ile ürünleri getirme
var products = connection.Query<Product>("SELECT * FROM Products WHERE Price > @Price", new { Price = 100 });
```

### Repository Pattern Örneği

```csharp
// Repository sınıfı
public class ProductRepository : IProductRepository
{
    private readonly string _connectionString;
    
    public ProductRepository(string connectionString)
    {
        _connectionString = connectionString;
    }
    
    public IEnumerable<Product> GetAll()
    {
        using var connection = new SqlConnection(_connectionString);
        return connection.GetAll<Product>();
    }
    
    public Product GetById(int id)
    {
        using var connection = new SqlConnection(_connectionString);
        return connection.Get<Product>(id);
    }
    
    public int Add(Product product)
    {
        using var connection = new SqlConnection(_connectionString);
        return (int)connection.Insert(product);
    }
    
    public bool Update(Product product)
    {
        using var connection = new SqlConnection(_connectionString);
        return connection.Update(product);
    }
    
    public bool Delete(int id)
    {
        using var connection = new SqlConnection(_connectionString);
        return connection.Delete(new Product { Id = id });
    }
}
```

## Dapper.FastCrud

Dapper.FastCrud, entity tabanlı CRUD işlemleri için daha kapsamlı bir API sağlar. Bu uzantı, daha karmaşık sorgular oluşturmanıza ve filtreleme, sıralama gibi işlemleri kolayca gerçekleştirmenize olanak tanır.

### Kurulum

```csharp
// NuGet üzerinden kurulum
Install-Package Dapper.FastCrud
```

### Temel Kullanım

Dapper.FastCrud kullanımı için, entity sınıflarınızı özel niteliklerle işaretlemeniz gerekir:

```csharp
// Entity sınıfı tanımlama
using Dapper.FastCrud;
using Dapper.FastCrud.Mappings;

[Table("Products")]
public class Product
{
    [Key]
    public int Id { get; set; }
    
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
    
    [NotMapped] // Veritabanında olmayan alan
    public string DisplayName => $"{Name} ({Price:C})";
}
```

### CRUD İşlemleri

Dapper.FastCrud, aşağıdaki temel CRUD işlemlerini destekler:

```csharp
// Bağlantı oluşturma
using var connection = new SqlConnection(connectionString);

// Tüm ürünleri getirme
var allProducts = connection.Find<Product>();

// ID'ye göre ürün getirme
var product = connection.Get<Product>(new Product { Id = 1 });

// Yeni ürün ekleme
var newProduct = new Product
{
    Name = "Yeni Ürün",
    Price = 99.99m,
    CategoryId = 1
};
connection.Insert(newProduct);

// Ürün güncelleme
product.Price = 149.99m;
connection.Update(product);

// Ürün silme
connection.Delete(product);

// Koşullu silme
connection.BulkDelete<Product>(statement => statement
    .Where($"{nameof(Product.CategoryId)} = @CategoryId")
    .WithParameters(new { CategoryId = 1 }));
```

### Filtreleme ve Sıralama

Dapper.FastCrud, filtreleme ve sıralama işlemleri için güçlü bir API sunar:

```csharp
// Filtreleme ile ürünleri getirme
var products = connection.Find<Product>(statement => statement
    .Where($"{nameof(Product.Price)} > @MinPrice AND {nameof(Product.CategoryId)} = @CategoryId")
    .WithParameters(new { MinPrice = 100, CategoryId = 1 })
    .OrderBy($"{nameof(Product.Name)}"));

// Sayfalama ile ürünleri getirme
var pagedProducts = connection.Find<Product>(statement => statement
    .Where($"{nameof(Product.CategoryId)} = @CategoryId")
    .WithParameters(new { CategoryId = 1 })
    .OrderBy($"{nameof(Product.Price)} DESC")
    .Skip(10)
    .Take(10));
```

### İlişkisel Veri

Dapper.FastCrud, ilişkisel verileri getirmek için JOIN işlemlerini destekler:

```csharp
// JOIN ile ürünleri ve kategorileri getirme
var sql = $@"
    SELECT p.*, c.*
    FROM {OrmConfiguration.GetSqlBuilder().GetTableName(typeof(Product))} p
    INNER JOIN {OrmConfiguration.GetSqlBuilder().GetTableName(typeof(Category))} c
        ON p.{nameof(Product.CategoryId)} = c.{nameof(Category.Id)}
    WHERE p.{nameof(Product.Price)} > @MinPrice";

var productDictionary = new Dictionary<int, Product>();

var products = connection.Query<Product, Category, Product>(
    sql,
    (product, category) => {
        if (!productDictionary.TryGetValue(product.Id, out var existingProduct))
        {
            existingProduct = product;
            existingProduct.Category = category;
            productDictionary.Add(existingProduct.Id, existingProduct);
        }
        return existingProduct;
    },
    new { MinPrice = 100 },
    splitOn: nameof(Category.Id));
```

## Dapper.SimpleCRUD

Dapper.SimpleCRUD, basit CRUD işlemleri için kolay bir API sağlar. Bu uzantı, SQL sorgularını otomatik olarak oluşturarak, veritabanı işlemlerini daha basit hale getirir.

### Kurulum

```csharp
// NuGet üzerinden kurulum
Install-Package Dapper.SimpleCRUD
```

### Temel Kullanım

Dapper.SimpleCRUD kullanımı için, entity sınıflarınızı özel niteliklerle işaretlemeniz gerekir:

```csharp
// Entity sınıfı tanımlama
using Dapper;
using Dapper.SimpleCRUD;

[Table("Products")]
public class Product
{
    [Key] // Birincil anahtar
    public int Id { get; set; }
    
    public string Name { get; set; }
    public decimal Price { get; set; }
    
    [Column("CategoryId")] // Farklı bir sütun adı belirtme
    public int CategoryId { get; set; }
    
    [Editable(false)] // Düzenlenemez alan
    public DateTime CreatedAt { get; set; }
    
    [NotMapped] // Veritabanında olmayan alan
    public string DisplayName => $"{Name} ({Price:C})";
}
```

### CRUD İşlemleri

Dapper.SimpleCRUD, aşağıdaki temel CRUD işlemlerini destekler:

```csharp
// Başlangıç yapılandırması
SimpleCRUD.SetDialect(SimpleCRUD.Dialect.SQLServer);

// Bağlantı oluşturma
using var connection = new SqlConnection(connectionString);

// Tüm ürünleri getirme
var allProducts = connection.GetList<Product>();

// ID'ye göre ürün getirme
var product = connection.Get<Product>(1);

// Filtreleme ile ürünleri getirme
var products = connection.GetList<Product>(new { CategoryId = 1, Price = 99.99m });

// Dinamik koşullar ile ürünleri getirme
var whereCondition = "Price > @Price AND CategoryId = @CategoryId";
var parameters = new { Price = 100, CategoryId = 1 };
var filteredProducts = connection.GetList<Product>(whereCondition, parameters);

// Sayfalama ile ürünleri getirme
var pagedProducts = connection.GetListPaged<Product>(1, 10, "Price DESC", "CategoryId = @CategoryId", new { CategoryId = 1 });

// Yeni ürün ekleme
var newProduct = new Product
{
    Name = "Yeni Ürün",
    Price = 99.99m,
    CategoryId = 1
};
var id = connection.Insert(newProduct);

// Ürün güncelleme
product.Price = 149.99m;
connection.Update(product);

// Ürün silme
connection.Delete(product);

// ID ile silme
connection.Delete<Product>(1);

// Koşullu silme
connection.DeleteList<Product>("CategoryId = @CategoryId", new { CategoryId = 1 });
```

## DapperExtensions

DapperExtensions, daha gelişmiş sorgulama yetenekleri sağlar. Bu uzantı, predicate tabanlı sorgulama, sayfalama ve ilişkisel veri getirme gibi işlemleri destekler.

### Kurulum

```csharp
// NuGet üzerinden kurulum
Install-Package DapperExtensions
```

### Temel Kullanım

DapperExtensions kullanımı için, entity sınıflarınızı ve mapping sınıflarını tanımlamanız gerekir:

```csharp
// Entity sınıfı tanımlama
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
    public DateTime CreatedAt { get; set; }
}

// Mapping sınıfı tanımlama
public class ProductMap : ClassMapper<Product>
{
    public ProductMap()
    {
        // Tablo adını belirtme
        Table("Products");
        
        // Birincil anahtar
        Map(p => p.Id).Key(KeyType.Identity);
        
        // Diğer alanlar
        Map(p => p.Name);
        Map(p => p.Price);
        Map(p => p.CategoryId);
        Map(p => p.CreatedAt);
        
        // Otomatik eşleştirme
        AutoMap();
    }
}
```

### CRUD İşlemleri

DapperExtensions, aşağıdaki temel CRUD işlemlerini destekler:

```csharp
// Bağlantı oluşturma
using var connection = new SqlConnection(connectionString);

// Tüm ürünleri getirme
var allProducts = connection.GetList<Product>();

// ID'ye göre ürün getirme
var product = connection.Get<Product>(1);

// Yeni ürün ekleme
var newProduct = new Product
{
    Name = "Yeni Ürün",
    Price = 99.99m,
    CategoryId = 1,
    CreatedAt = DateTime.Now
};
var id = connection.Insert(newProduct);

// Ürün güncelleme
product.Price = 149.99m;
connection.Update(product);

// Ürün silme
connection.Delete(product);
```

### Predicate Tabanlı Sorgulama

DapperExtensions, predicate tabanlı sorgulama için güçlü bir API sunar:

```csharp
// Predicate ile filtreleme
var predicate = Predicates.Field<Product>(p => p.CategoryId, Operator.Eq, 1);
var products = connection.GetList<Product>(predicate);

// Çoklu predicate ile filtreleme
var predicateGroup = new PredicateGroup
{
    Operator = GroupOperator.And,
    Predicates = new List<IPredicate>
    {
        Predicates.Field<Product>(p => p.CategoryId, Operator.Eq, 1),
        Predicates.Field<Product>(p => p.Price, Operator.Gt, 100)
    }
};
var filteredProducts = connection.GetList<Product>(predicateGroup);

// Sıralama ile filtreleme
var sort = new List<ISort>
{
    Predicates.Sort<Product>(p => p.Name),
    Predicates.Sort<Product>(p => p.Price, false) // Azalan sıralama
};
var sortedProducts = connection.GetList<Product>(predicateGroup, sort);

// Sayfalama ile filtreleme
var pagedProducts = connection.GetPage<Product>(
    predicateGroup,
    sort,
    1,  // Sayfa numarası
    10); // Sayfa boyutu
```

## Dapper.Rainbow

Dapper.Rainbow, tablo başına tek bir nesne ile çalışmak için basit bir API sağlar. Bu uzantı, özellikle basit veritabanı işlemleri için idealdir.

### Kurulum

```csharp
// NuGet üzerinden kurulum
Install-Package Dapper.Rainbow
```

### Temel Kullanım

Dapper.Rainbow kullanımı için, bir veritabanı sınıfı tanımlamanız gerekir:

```csharp
// Veritabanı sınıfı tanımlama
public class MyDatabase : Database
{
    public MyDatabase(string connectionString) : base(connectionString) { }
    
    // Tablo tanımlamaları
    public Table<Product> Products { get; set; }
    public Table<Category> Categories { get; set; }
}

// Entity sınıfları
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public int CategoryId { get; set; }
}

public class Category
{
    public int Id { get; set; }
    public string Name { get; set; }
}
```

### CRUD İşlemleri

Dapper.Rainbow, aşağıdaki temel CRUD işlemlerini destekler:

```csharp
// Veritabanı örneği oluşturma
var db = new MyDatabase(connectionString);

// ID'ye göre ürün getirme
var product = db.Products.Get(1);

// Koşullu ürün getirme
var products = db.Products.All("WHERE CategoryId = @CategoryId", new { CategoryId = 1 });

// Yeni ürün ekleme
var newProduct = new Product
{
    Name = "Yeni Ürün",
    Price = 99.99m,
    CategoryId = 1
};
var id = db.Products.Insert(newProduct);

// Ürün güncelleme
product.Price = 149.99m;
db.Products.Update(product);

// Ürün silme
db.Products.Delete(product.Id);

// Koşullu silme
db.Products.Delete("WHERE CategoryId = @CategoryId", new { CategoryId = 1 });
```

### İlişkisel Veri

Dapper.Rainbow, ilişkisel verileri getirmek için özel sorgular kullanmanızı gerektirir:

```csharp
// JOIN ile ürünleri ve kategorileri getirme
var sql = @"
    SELECT p.*, c.*
    FROM Products p
    INNER JOIN Categories c ON p.CategoryId = c.Id
    WHERE p.Price > @MinPrice";

var productDictionary = new Dictionary<int, Product>();

var products = db.Query<Product, Category, Product>(
    sql,
    (product, category) => {
        if (!productDictionary.TryGetValue(product.Id, out var existingProduct))
        {
            existingProduct = product;
            existingProduct.Category = category;
            productDictionary.Add(existingProduct.Id, existingProduct);
        }
        return existingProduct;
    },
    new { MinPrice = 100 },
    splitOn: "Id");
```

## Karşılaştırma ve Seçim Kriterleri

Dapper uzantıları arasında seçim yaparken, aşağıdaki faktörleri göz önünde bulundurmanız gerekir:

### Özellik Karşılaştırması

| Özellik | Dapper.Contrib | Dapper.FastCrud | Dapper.SimpleCRUD | DapperExtensions | Dapper.Rainbow |
|---------|----------------|-----------------|-------------------|------------------|----------------|
| Temel CRUD | ✓ | ✓ | ✓ | ✓ | ✓ |
| Otomatik SQL | ✓ | ✓ | ✓ | ✓ | ✓ |
| Filtreleme | Sınırlı | Gelişmiş | Orta | Gelişmiş | Sınırlı |
| Sayfalama | ✗ | ✓ | ✓ | ✓ | ✗ |
| İlişkisel Veri | ✗ | Sınırlı | ✗ | Sınırlı | ✗ |
| Predicate API | ✗ | ✗ | ✗ | ✓ | ✗ |
| Performans | Yüksek | Orta | Orta | Orta | Yüksek |
| Bakım | Aktif | Aktif | Aktif | Sınırlı | Sınırlı |

### Seçim Kriterleri

1. **Proje Karmaşıklığı**: Basit projeler için Dapper.Contrib veya Dapper.Rainbow, karmaşık projeler için Dapper.FastCrud veya DapperExtensions tercih edilebilir.

2. **Performans Gereksinimleri**: Yüksek performans gerektiren projeler için Dapper.Contrib veya doğrudan Dapper kullanımı daha uygundur.

3. **Sorgulama İhtiyaçları**: Karmaşık sorgulama ihtiyaçları için DapperExtensions veya Dapper.FastCrud daha uygun olabilir.

4. **Kod Basitliği**: Daha az kod yazmak için Dapper.SimpleCRUD veya Dapper.Rainbow tercih edilebilir.

5. **Bakım ve Güncellik**: Aktif olarak geliştirilen ve bakımı yapılan kütüphaneleri tercih etmek önemlidir.

## Özet

Bu bölümde, Dapper ekosistemindeki popüler uzantıları inceledik:

- **Dapper.Contrib**: Temel CRUD işlemleri için basit bir API sağlar.
- **Dapper.FastCrud**: Entity tabanlı CRUD işlemleri için daha kapsamlı bir API sunar.
- **Dapper.SimpleCRUD**: Basit CRUD işlemleri için kolay bir API sağlar.
- **DapperExtensions**: Daha gelişmiş sorgulama yetenekleri sunar.
- **Dapper.Rainbow**: Tablo başına tek bir nesne ile çalışmak için basit bir API sağlar.

Her bir uzantı, farklı senaryolar için farklı avantajlar sunar. Projenizin gereksinimlerine ve karmaşıklığına bağlı olarak, en uygun uzantıyı seçebilirsiniz. Dapper'ın temel felsefesi olan basitlik ve performans, tüm bu uzantılarda da korunmuştur. 