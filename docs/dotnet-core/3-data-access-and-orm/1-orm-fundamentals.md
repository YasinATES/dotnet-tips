# ORM Temelleri

Object-Relational Mapping (ORM), nesne yönelimli programlama dilleri ile ilişkisel veritabanları arasında bir köprü görevi gören bir programlama tekniğidir. Bu bölümde, ORM kavramını, avantajlarını, temel prensiplerini ve .NET ekosistemindeki uygulamalarını inceleyeceğiz.

## ORM Kavramı ve Avantajları

ORM, ilişkisel veritabanı tablolarını nesne yönelimli sınıflara dönüştüren bir tekniktir. Bu dönüşüm, geliştiricilerin SQL sorguları yazmak yerine nesne yönelimli programlama dilinin gücünü kullanarak veritabanı işlemlerini gerçekleştirmesine olanak tanır.

### ORM'nin Temel Amacı

ORM'nin temel amacı, iki farklı paradigma arasındaki "impedance mismatch" (empedans uyumsuzluğu) sorununu çözmektir:

1. **İlişkisel Model**: Tablolar, satırlar ve sütunlar kullanır; normalizasyon ve ilişkiler önemlidir.
2. **Nesne Modeli**: Sınıflar, nesneler ve kalıtım kullanır; kapsülleme ve polimorfizm önemlidir.

### ORM'nin Avantajları

1. **Veritabanı Bağımsızlığı**: Uygulamanızı farklı veritabanı sistemleri arasında taşımanızı kolaylaştırır.

2. **Geliştirme Hızı**: SQL sorguları yazmak yerine nesne yönelimli kod yazarak geliştirme sürecini hızlandırır.

3. **Kod Okunabilirliği**: SQL sorgularının karmaşıklığını azaltarak kodun daha okunabilir olmasını sağlar.

4. **Güvenlik**: SQL enjeksiyon saldırılarına karşı koruma sağlar.

5. **Bakım Kolaylığı**: Veritabanı şeması değiştiğinde, sadece model sınıflarını güncellemeniz yeterlidir.

6. **Performans Optimizasyonu**: Birçok ORM, sorgu önbellekleme ve lazy loading gibi performans optimizasyonları sunar.

7. **Test Edilebilirlik**: Veritabanı işlemlerini taklit etmek (mock) daha kolaydır, bu da birim testlerini kolaylaştırır.

### ORM'nin Dezavantajları

1. **Performans Sorunları**: Karmaşık sorgularda doğrudan SQL kullanmak bazen daha verimli olabilir.

2. **Öğrenme Eğrisi**: ORM araçlarının etkin kullanımı için belirli bir öğrenme süreci gerektirir.

3. **Kontrol Kaybı**: Arka planda oluşturulan SQL sorgularının tam kontrolü olmayabilir.

4. **Karmaşık Sorgular**: Çok karmaşık sorgular için ORM yetersiz kalabilir.

## Object-Relational Mapping Prensipleri

ORM sistemleri, aşağıdaki temel prensiplere dayanır:

### 1. Nesne-İlişki Eşleştirmesi

Veritabanı tabloları sınıflara, tablo sütunları özelliklere (properties) ve tablo satırları nesnelere eşlenir.

```csharp
// Database table: Customers
// Columns: Id, FirstName, LastName, Email
public class Customer
{
    // Maps to the primary key column
    public int Id { get; set; }
    
    // Maps to the FirstName column
    public string FirstName { get; set; }
    
    // Maps to the LastName column
    public string LastName { get; set; }
    
    // Maps to the Email column
    public string Email { get; set; }
}
```

### 2. İlişki Yönetimi

ORM, veritabanındaki ilişkileri (bire-bir, bire-çok, çoka-çok) nesne modeline yansıtır.

```csharp
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    
    // One-to-many relationship: One customer can have many orders
    public List<Order> Orders { get; set; }
}

public class Order
{
    public int Id { get; set; }
    public DateTime OrderDate { get; set; }
    
    // Many-to-one relationship: Many orders belong to one customer
    public int CustomerId { get; set; }
    public Customer Customer { get; set; }
}
```

### 3. CRUD İşlemleri

ORM, temel CRUD (Create, Read, Update, Delete) işlemlerini nesne yönelimli bir şekilde gerçekleştirmenizi sağlar.

```csharp
// Create
var customer = new Customer { Name = "Ahmet Yılmaz", Email = "ahmet@example.com" };
context.Customers.Add(customer);
await context.SaveChangesAsync();

// Read
var customers = await context.Customers.ToListAsync();
var customer = await context.Customers.FindAsync(1);

// Update
customer.Name = "Ahmet Mehmet Yılmaz";
await context.SaveChangesAsync();

// Delete
context.Customers.Remove(customer);
await context.SaveChangesAsync();
```

### 4. Sorgu Dili

ORM'ler genellikle SQL'e benzer bir sorgu dili veya LINQ gibi dile entegre sorgu özellikleri sunar.

```csharp
// LINQ query
var highValueCustomers = await context.Customers
    .Where(c => c.Orders.Sum(o => o.TotalAmount) > 10000)
    .OrderByDescending(c => c.Orders.Sum(o => o.TotalAmount))
    .Select(c => new { c.Name, TotalSpent = c.Orders.Sum(o => o.TotalAmount) })
    .ToListAsync();
```

### 5. Değişiklik Takibi

ORM'ler, nesnelerdeki değişiklikleri izler ve veritabanına yansıtır.

```csharp
// Entity is tracked after being retrieved from the database
var customer = await context.Customers.FindAsync(1);
customer.Email = "new.email@example.com";

// No need to explicitly update, changes are tracked
await context.SaveChangesAsync();
```

### 6. Lazy ve Eager Loading

ORM'ler, ilişkili verilerin ne zaman yükleneceğini kontrol etmenize olanak tanır.

```csharp
// Lazy loading - related data is loaded only when accessed
var customer = await context.Customers.FindAsync(1);
// Orders are loaded only when accessed
var orderCount = customer.Orders.Count;

// Eager loading - related data is loaded immediately
var customerWithOrders = await context.Customers
    .Include(c => c.Orders)
    .FirstOrDefaultAsync(c => c.Id == 1);
```

## ORM vs ADO.NET Karşılaştırması

ADO.NET, .NET'in doğrudan veritabanı erişimi için sunduğu bir kütüphanedir. ORM'ler ise ADO.NET üzerine inşa edilmiş daha yüksek seviyeli bir soyutlamadır.

### ADO.NET Yaklaşımı

```csharp
// ADO.NET approach
public List<Customer> GetCustomers()
{
    var customers = new List<Customer>();
    
    using (var connection = new SqlConnection(_connectionString))
    {
        connection.Open();
        
        using (var command = new SqlCommand("SELECT Id, Name, Email FROM Customers", connection))
        {
            using (var reader = command.ExecuteReader())
            {
                while (reader.Read())
                {
                    customers.Add(new Customer
                    {
                        Id = reader.GetInt32(0),
                        Name = reader.GetString(1),
                        Email = reader.GetString(2)
                    });
                }
            }
        }
    }
    
    return customers;
}
```

### ORM Yaklaşımı (Entity Framework Core)

```csharp
// ORM (Entity Framework Core) approach
public async Task<List<Customer>> GetCustomers()
{
    using (var context = new ApplicationDbContext())
    {
        return await context.Customers.ToListAsync();
    }
}
```

### Karşılaştırma Tablosu

| Özellik | ADO.NET | ORM |
|---------|---------|-----|
| Kod Miktarı | Daha fazla | Daha az |
| Performans | Genellikle daha hızlı | Ek yük nedeniyle biraz daha yavaş olabilir |
| Öğrenme Eğrisi | Daha düşük | Daha yüksek |
| Veritabanı Bağımsızlığı | Düşük | Yüksek |
| SQL Kontrolü | Tam kontrol | Sınırlı kontrol |
| Bakım | Daha zor | Daha kolay |
| Güvenlik | Manuel önlemler gerektirir | Otomatik koruma sağlar |

## ORM Seçim Kriterleri

Bir ORM seçerken aşağıdaki kriterleri göz önünde bulundurmalısınız:

### 1. Proje Gereksinimleri

- **Proje Büyüklüğü**: Küçük projeler için daha hafif ORM'ler yeterli olabilir.
- **Performans Gereksinimleri**: Yüksek performans gerektiren uygulamalar için daha optimize edilmiş ORM'ler veya hibrit yaklaşımlar düşünülebilir.
- **Veritabanı Desteği**: Kullanmak istediğiniz veritabanını destekleyen bir ORM seçin.

### 2. Ekosistem ve Topluluk

- **Dokümantasyon**: İyi belgelenmiş bir ORM, geliştirme sürecini hızlandırır.
- **Topluluk Desteği**: Aktif bir topluluğa sahip ORM'ler, sorunlarınızı çözmenize yardımcı olabilir.
- **Güncellik**: Düzenli olarak güncellenen bir ORM seçin.

### 3. Özellikler

- **Sorgu Yetenekleri**: Karmaşık sorguları desteklemesi önemlidir.
- **Performans Optimizasyonları**: Önbellekleme, lazy loading gibi özellikler.
- **Migrasyon Desteği**: Veritabanı şemasını kod üzerinden yönetme yeteneği.

### 4. Öğrenme Eğrisi

- **Ekip Deneyimi**: Ekibinizin belirli bir ORM ile deneyimi varsa, bu ORM'i seçmek avantajlı olabilir.
- **Dokümantasyon Kalitesi**: İyi belgelenmiş bir ORM, öğrenme sürecini kolaylaştırır.

### .NET Ekosistemindeki Popüler ORM'ler

1. **Entity Framework Core**: Microsoft'un resmi ORM'i, en yaygın kullanılan çözümdür.
2. **Dapper**: Hafif, yüksek performanslı bir micro-ORM'dir.
3. **NHibernate**: Olgun ve özellik açısından zengin bir ORM'dir.
4. **ServiceStack.OrmLite**: Hafif ve hızlı bir ORM çözümüdür.
5. **LLBLGen Pro**: Ticari, özellik açısından zengin bir ORM'dir.

## N+1 Sorunu ve Çözümleri

N+1 sorunu, ORM'lerin en yaygın performans sorunlarından biridir. Bu sorun, bir koleksiyonu yükledikten sonra, koleksiyondaki her öğe için ayrı bir sorgu çalıştırıldığında ortaya çıkar.

### Sorunun Açıklaması

Örneğin, tüm müşterileri ve siparişlerini yüklemek istediğinizi düşünün:

```csharp
// This can lead to N+1 problem
var customers = context.Customers.ToList(); // 1 query to get all customers

foreach (var customer in customers)
{
    // N queries, one for each customer's orders
    var orders = customer.Orders.ToList();
}
```

Bu kod, aşağıdaki sorguları çalıştırır:
1. Tüm müşterileri getiren 1 sorgu
2. Her müşteri için siparişleri getiren N sorgu (müşteri sayısı kadar)

Toplam: N+1 sorgu

### Çözümler

#### 1. Eager Loading (Include)

```csharp
// Eager loading with Include
var customers = context.Customers
    .Include(c => c.Orders) // Load orders along with customers
    .ToList();

// Now you can access orders without additional queries
foreach (var customer in customers)
{
    var orders = customer.Orders;
}
```

#### 2. Explicit Loading

```csharp
// Explicit loading
var customers = context.Customers.ToList();

// Load all orders for all customers in one query
context.Entry(customers)
    .Collection(c => c.Orders)
    .Load();
```

#### 3. Projection (Select)

```csharp
// Projection with Select
var customerData = context.Customers
    .Select(c => new
    {
        Customer = c,
        Orders = c.Orders
    })
    .ToList();

foreach (var data in customerData)
{
    var customer = data.Customer;
    var orders = data.Orders;
}
```

#### 4. Future Queries (NHibernate)

```csharp
// Future queries (NHibernate specific)
var customersQuery = session.Query<Customer>().Future();
var ordersQuery = session.Query<Order>().Future();

// Both queries are executed in a single roundtrip
var customers = customersQuery.ToList();
var orders = ordersQuery.ToList();
```

#### 5. BatchSize Yapılandırması

```csharp
// Configure batch size in Entity Framework Core
modelBuilder.Entity<Customer>()
    .Navigation(c => c.Orders)
    .UsePropertyAccessMode(PropertyAccessMode.Property)
    .AutoInclude();

// In NHibernate
mapping.Bag(x => x.Orders, map =>
{
    map.BatchSize(25);
});
```

### Performans Karşılaştırması

| Yaklaşım | Sorgu Sayısı | Avantajları | Dezavantajları |
|----------|--------------|-------------|----------------|
| N+1 Sorunu | N+1 | Kod basitliği | Çok sayıda sorgu, düşük performans |
| Eager Loading | 1 | Tek sorgu, yüksek performans | Gereksiz veri yüklenebilir |
| Explicit Loading | 2 | Kontrollü veri yükleme | Biraz daha karmaşık kod |
| Projection | 1 | Yüksek performans, özelleştirilebilir | Nesne izleme kaybı |
| Future Queries | 1 | Yüksek performans | ORM'e özgü özellik |

## Özet

ORM, nesne yönelimli programlama ile ilişkisel veritabanları arasında bir köprü görevi görür. Geliştirme sürecini hızlandırır, kod okunabilirliğini artırır ve bakımı kolaylaştırır. Ancak, performans sorunları ve öğrenme eğrisi gibi dezavantajları da vardır.

Doğru ORM seçimi, projenizin gereksinimlerine, ekibinizin deneyimine ve performans beklentilerinize bağlıdır. N+1 sorunu gibi yaygın performans sorunlarının farkında olmak ve bunları çözmek için uygun teknikleri kullanmak, ORM'leri etkili bir şekilde kullanmanın önemli bir parçasıdır.

.NET ekosisteminde, Entity Framework Core, Dapper, NHibernate gibi çeşitli ORM seçenekleri bulunmaktadır. Her birinin kendi güçlü yönleri ve zayıf yönleri vardır, bu nedenle projeniz için en uygun olanı seçmek önemlidir. 