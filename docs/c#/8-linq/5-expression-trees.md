# Expression Trees (İfade Ağaçları)

Expression Trees (İfade Ağaçları), C# kodunu çalışma zamanında temsil eden veri yapılarıdır. Bu yapılar, kodun çalıştırılması yerine incelenmesine, değiştirilmesine ve dinamik olarak oluşturulmasına olanak tanır. Bu bölümde, Expression Trees'in nasıl çalıştığını ve bunların LINQ ve dinamik sorgu oluşturma gibi senaryolarda nasıl kullanıldığını inceleyeceğiz.

## 1. Expression<T> vs Func<T>

C#'ta, lambda ifadeleri iki farklı şekilde temsil edilebilir: `Expression<T>` ve `Func<T>`. Bu iki tip arasındaki farkı anlamak, Expression Trees'i etkili bir şekilde kullanmak için önemlidir.

### Func<T>

`Func<T>` delegeleri, çalıştırılabilir kod bloklarını temsil eder. Lambda ifadesi bir `Func<T>` olarak derlendiğinde, derleyici bu ifadeyi doğrudan çalıştırılabilir IL (Intermediate Language) koduna dönüştürür.

```csharp
// Func<T> örneği
Func<int, int, int> add = (a, b) => a + b;

// Fonksiyonu çalıştırma
int result = add(5, 3); // Sonuç: 8
Console.WriteLine($"5 + 3 = {result}");
```

### Expression<T>

`Expression<T>` ise, lambda ifadesini bir kod bloğu olarak değil, bir veri yapısı olarak temsil eder. Bu veri yapısı, lambda ifadesinin sözdizimsel yapısını korur ve çalışma zamanında incelenebilir, değiştirilebilir veya dönüştürülebilir.

```csharp
// Expression<T> örneği
Expression<Func<int, int, int>> addExpr = (a, b) => a + b;

// İfade ağacını inceleme
Console.WriteLine($"İfade türü: {addExpr.Body.NodeType}");
Console.WriteLine($"Sol operand: {((BinaryExpression)addExpr.Body).Left}");
Console.WriteLine($"Sağ operand: {((BinaryExpression)addExpr.Body).Right}");

// İfadeyi derleyip çalıştırma
Func<int, int, int> compiledAdd = addExpr.Compile();
int exprResult = compiledAdd(5, 3); // Sonuç: 8
Console.WriteLine($"5 + 3 = {exprResult}");
```

### Karşılaştırma

```csharp
// Veri modelleri
public class Customer
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal TotalBalance { get; set; }
    public bool IsActive { get; set; }
}

// Örnek veri
List<Customer> customers = new List<Customer>
{
    new Customer { Id = 1, Name = "Ali Yılmaz", TotalBalance = 15000, IsActive = true },
    new Customer { Id = 2, Name = "Ayşe Demir", TotalBalance = 28500, IsActive = true },
    new Customer { Id = 3, Name = "Mehmet Kaya", TotalBalance = 7200, IsActive = false },
    new Customer { Id = 4, Name = "Zeynep Çelik", TotalBalance = 42000, IsActive = true }
};

// Func<T> ile filtreleme - bellek içinde çalışır
Func<Customer, bool> activeFilter = c => c.IsActive;
var activeCustomers = customers.Where(activeFilter);

// Expression<T> ile filtreleme - veritabanı sorgusuna dönüştürülebilir
Expression<Func<Customer, bool>> activeExprFilter = c => c.IsActive;

// Entity Framework kullanımı
using (var context = new CustomerContext())
{
    // IQueryable<T> sorgusu - filtreleme veritabanında yapılır
    var dbActiveCustomers = context.Customers.Where(activeExprFilter);
    
    // SQL sorgusu şuna benzer:
    // SELECT * FROM Customers WHERE IsActive = 1
}
```

## 2. Building Expression Trees (İfade Ağaçları Oluşturma)

İfade ağaçları, lambda sözdizimi kullanılarak derleme zamanında oluşturulabilir veya Expression sınıfının statik metotları kullanılarak programlı bir şekilde çalışma zamanında oluşturulabilir.

### Lambda Sözdizimi ile Oluşturma

```csharp
// Lambda sözdizimi ile ifade ağacı oluşturma
Expression<Func<Customer, bool>> isVipCustomer = c => c.TotalBalance > 25000 && c.IsActive;

// İfade ağacını inceleme
Console.WriteLine($"İfade türü: {isVipCustomer.Body.NodeType}");
Console.WriteLine($"İfade: {isVipCustomer}");
```

### Programlı Oluşturma

```csharp
// Programlı olarak ifade ağacı oluşturma
ParameterExpression customerParam = Expression.Parameter(typeof(Customer), "c");

// c.TotalBalance > 25000
MemberExpression balanceProp = Expression.Property(customerParam, "TotalBalance");
ConstantExpression threshold = Expression.Constant(25000m);
BinaryExpression balanceCheck = Expression.GreaterThan(balanceProp, threshold);

// c.IsActive
MemberExpression activeProp = Expression.Property(customerParam, "IsActive");
ConstantExpression trueValue = Expression.Constant(true);
BinaryExpression activeCheck = Expression.Equal(activeProp, trueValue);

// c.TotalBalance > 25000 && c.IsActive
BinaryExpression combinedCheck = Expression.AndAlso(balanceCheck, activeCheck);

// Oluşturulan ifade ağacını lambda ifadesine dönüştürme
Expression<Func<Customer, bool>> programmaticExpr = Expression.Lambda<Func<Customer, bool>>(
    combinedCheck,
    customerParam
);

Console.WriteLine($"Programlı oluşturulan ifade: {programmaticExpr}");
```

### Karmaşık İfade Ağaçları Oluşturma

```csharp
// Karmaşık filtreleme senaryosu
public class TransactionFilter
{
    public DateTime? StartDate { get; set; }
    public DateTime? EndDate { get; set; }
    public decimal? MinAmount { get; set; }
    public decimal? MaxAmount { get; set; }
    public string TransactionType { get; set; }
    public int? CustomerId { get; set; }
}

public class Transaction
{
    public int Id { get; set; }
    public int CustomerId { get; set; }
    public DateTime Date { get; set; }
    public decimal Amount { get; set; }
    public string TransactionType { get; set; }
}

// Filtre nesnesinden dinamik sorgu oluşturma
public static Expression<Func<Transaction, bool>> BuildFilterExpression(TransactionFilter filter)
{
    ParameterExpression transactionParam = Expression.Parameter(typeof(Transaction), "t");
    Expression combinedExpression = null;
    
    // Tarih aralığı filtresi
    if (filter.StartDate.HasValue)
    {
        MemberExpression dateProp = Expression.Property(transactionParam, "Date");
        ConstantExpression startDateValue = Expression.Constant(filter.StartDate.Value);
        BinaryExpression dateCheck = Expression.GreaterThanOrEqual(dateProp, startDateValue);
        
        combinedExpression = combinedExpression == null 
            ? dateCheck 
            : Expression.AndAlso(combinedExpression, dateCheck);
    }
    
    if (filter.EndDate.HasValue)
    {
        MemberExpression dateProp = Expression.Property(transactionParam, "Date");
        ConstantExpression endDateValue = Expression.Constant(filter.EndDate.Value);
        BinaryExpression dateCheck = Expression.LessThanOrEqual(dateProp, endDateValue);
        
        combinedExpression = combinedExpression == null 
            ? dateCheck 
            : Expression.AndAlso(combinedExpression, dateCheck);
    }
    
    // Tutar aralığı filtresi
    if (filter.MinAmount.HasValue)
    {
        MemberExpression amountProp = Expression.Property(transactionParam, "Amount");
        ConstantExpression minAmountValue = Expression.Constant(filter.MinAmount.Value);
        BinaryExpression amountCheck = Expression.GreaterThanOrEqual(amountProp, minAmountValue);
        
        combinedExpression = combinedExpression == null 
            ? amountCheck 
            : Expression.AndAlso(combinedExpression, amountCheck);
    }
    
    if (filter.MaxAmount.HasValue)
    {
        MemberExpression amountProp = Expression.Property(transactionParam, "Amount");
        ConstantExpression maxAmountValue = Expression.Constant(filter.MaxAmount.Value);
        BinaryExpression amountCheck = Expression.LessThanOrEqual(amountProp, maxAmountValue);
        
        combinedExpression = combinedExpression == null 
            ? amountCheck 
            : Expression.AndAlso(combinedExpression, amountCheck);
    }
    
    // İşlem tipi filtresi
    if (!string.IsNullOrEmpty(filter.TransactionType))
    {
        MemberExpression typeProp = Expression.Property(transactionParam, "TransactionType");
        ConstantExpression typeValue = Expression.Constant(filter.TransactionType);
        BinaryExpression typeCheck = Expression.Equal(typeProp, typeValue);
        
        combinedExpression = combinedExpression == null 
            ? typeCheck 
            : Expression.AndAlso(combinedExpression, typeCheck);
    }
    
    // Müşteri ID filtresi
    if (filter.CustomerId.HasValue)
    {
        MemberExpression customerIdProp = Expression.Property(transactionParam, "CustomerId");
        ConstantExpression customerIdValue = Expression.Constant(filter.CustomerId.Value);
        BinaryExpression customerIdCheck = Expression.Equal(customerIdProp, customerIdValue);
        
        combinedExpression = combinedExpression == null 
            ? customerIdCheck 
            : Expression.AndAlso(combinedExpression, customerIdCheck);
    }
    
    // Hiçbir filtre belirtilmemişse, tüm işlemleri döndür
    if (combinedExpression == null)
    {
        combinedExpression = Expression.Constant(true);
    }
    
    return Expression.Lambda<Func<Transaction, bool>>(combinedExpression, transactionParam);
}

// Kullanım örneği
TransactionFilter filter = new TransactionFilter
{
    StartDate = new DateTime(2023, 1, 1),
    EndDate = new DateTime(2023, 1, 31),
    MinAmount = 1000,
    TransactionType = "Deposit"
};

Expression<Func<Transaction, bool>> filterExpr = BuildFilterExpression(filter);
Console.WriteLine($"Oluşturulan filtre ifadesi: {filterExpr}");

// Entity Framework ile kullanım
using (var context = new TransactionContext())
{
    var filteredTransactions = context.Transactions.Where(filterExpr).ToList();
}
```

## 3. Compiling Expressions (İfadeleri Derleme)

İfade ağaçları, `Compile` metodu kullanılarak çalıştırılabilir delegelere dönüştürülebilir. Bu, dinamik olarak oluşturulan kodun çalıştırılmasına olanak tanır.

```csharp
// İfade ağacını derleme
Expression<Func<Customer, bool>> isVipExpr = c => c.TotalBalance > 25000 && c.IsActive;
Func<Customer, bool> isVipFunc = isVipExpr.Compile();

// Derlenen fonksiyonu kullanma
foreach (var customer in customers)
{
    if (isVipFunc(customer))
    {
        Console.WriteLine($"VIP Müşteri: {customer.Name} - {customer.TotalBalance:C}");
    }
}
```

### Performans Değerlendirmeleri

```csharp
// Performans karşılaştırması
public static void ComparePerformance()
{
    // Test verileri
    List<Customer> testCustomers = Enumerable.Range(1, 1000000)
        .Select(i => new Customer
        {
            Id = i,
            Name = $"Customer {i}",
            TotalBalance = i * 100,
            IsActive = i % 10 != 0
        })
        .ToList();
    
    // Doğrudan lambda
    Func<Customer, bool> directLambda = c => c.TotalBalance > 25000 && c.IsActive;
    
    // Derlenen ifade
    Expression<Func<Customer, bool>> expr = c => c.TotalBalance > 25000 && c.IsActive;
    Func<Customer, bool> compiledExpr = expr.Compile();
    
    // Doğrudan lambda performansı
    var sw1 = System.Diagnostics.Stopwatch.StartNew();
    int count1 = testCustomers.Count(directLambda);
    sw1.Stop();
    
    // Derlenen ifade performansı
    var sw2 = System.Diagnostics.Stopwatch.StartNew();
    int count2 = testCustomers.Count(compiledExpr);
    sw2.Stop();
    
    Console.WriteLine($"Doğrudan lambda: {sw1.ElapsedMilliseconds} ms, Sonuç: {count1}");
    Console.WriteLine($"Derlenen ifade: {sw2.ElapsedMilliseconds} ms, Sonuç: {count2}");
}
```

## 4. Expression Tree Visitors (İfade Ağacı Ziyaretçileri)

Expression Tree Visitors, ifade ağaçlarını dolaşmak, incelemek ve değiştirmek için kullanılan güçlü bir mekanizmadır. `ExpressionVisitor` sınıfını genişleterek, özel ziyaretçiler oluşturabilir ve ifade ağaçları üzerinde karmaşık dönüşümler gerçekleştirebilirsiniz.

### Temel Ziyaretçi

```csharp
// Temel ifade ağacı ziyaretçisi
public class ExpressionPrinter : ExpressionVisitor
{
    private int _indentLevel = 0;
    
    public void PrintExpression(Expression expression)
    {
        Visit(expression);
    }
    
    protected override Expression VisitBinary(BinaryExpression node)
    {
        Console.WriteLine($"{new string(' ', _indentLevel * 2)}Binary: {node.NodeType}");
        
        _indentLevel++;
        Console.Write($"{new string(' ', _indentLevel * 2)}Left: ");
        Visit(node.Left);
        
        Console.Write($"{new string(' ', _indentLevel * 2)}Right: ");
        Visit(node.Right);
        _indentLevel--;
        
        return node;
    }
    
    protected override Expression VisitConstant(ConstantExpression node)
    {
        Console.WriteLine($"Constant: {node.Value}");
        return node;
    }
    
    protected override Expression VisitMember(MemberExpression node)
    {
        Console.WriteLine($"Member: {node.Member.Name}");
        return node;
    }
    
    protected override Expression VisitParameter(ParameterExpression node)
    {
        Console.WriteLine($"Parameter: {node.Name}");
        return node;
    }
}

// Kullanım örneği
Expression<Func<Customer, bool>> expr = c => c.TotalBalance > 25000 && c.IsActive;
var printer = new ExpressionPrinter();
printer.PrintExpression(expr.Body);
```

### İfade Ağacı Dönüştürme

```csharp
// İfade ağacı dönüştürücü
public class ParameterReplacer : ExpressionVisitor
{
    private readonly ParameterExpression _oldParameter;
    private readonly ParameterExpression _newParameter;
    
    public ParameterReplacer(ParameterExpression oldParameter, ParameterExpression newParameter)
    {
        _oldParameter = oldParameter;
        _newParameter = newParameter;
    }
    
    protected override Expression VisitParameter(ParameterExpression node)
    {
        return node == _oldParameter ? _newParameter : base.VisitParameter(node);
    }
    
    public Expression Replace(Expression expression)
    {
        return Visit(expression);
    }
}

// Kullanım örneği
Expression<Func<Customer, bool>> customerExpr = c => c.TotalBalance > 25000;
ParameterExpression oldParam = customerExpr.Parameters[0];
ParameterExpression newParam = Expression.Parameter(typeof(Customer), "customer");

var replacer = new ParameterReplacer(oldParam, newParam);
Expression newBody = replacer.Replace(customerExpr.Body);

Expression<Func<Customer, bool>> newExpr = Expression.Lambda<Func<Customer, bool>>(newBody, newParam);
Console.WriteLine($"Orijinal ifade: {customerExpr}");
Console.WriteLine($"Dönüştürülmüş ifade: {newExpr}");
```

## 5. Dynamic Query Building (Dinamik Sorgu Oluşturma)

İfade ağaçları, kullanıcı girdisine veya çalışma zamanı koşullarına dayalı olarak dinamik sorgular oluşturmak için kullanılabilir. Bu, özellikle karmaşık arama formları veya raporlama araçları için yararlıdır.

```csharp
// Dinamik sorgu oluşturucu
public class QueryBuilder<T>
{
    private readonly List<Expression<Func<T, bool>>> _filters = new List<Expression<Func<T, bool>>>();
    
    public void AddFilter(Expression<Func<T, bool>> filter)
    {
        _filters.Add(filter);
    }
    
    public Expression<Func<T, bool>> Build()
    {
        if (_filters.Count == 0)
        {
            // Hiç filtre yoksa, tüm öğeleri döndür
            return x => true;
        }
        
        // İlk filtreyi al
        Expression<Func<T, bool>> result = _filters[0];
        
        // Diğer filtreleri AND operatörü ile birleştir
        for (int i = 1; i < _filters.Count; i++)
        {
            result = CombineExpressions(result, _filters[i]);
        }
        
        return result;
    }
    
    private Expression<Func<T, bool>> CombineExpressions(
        Expression<Func<T, bool>> expr1,
        Expression<Func<T, bool>> expr2)
    {
        // expr2'nin parametresini expr1'in parametresi ile değiştir
        var parameter = expr1.Parameters[0];
        var visitor = new ParameterReplacer(expr2.Parameters[0], parameter);
        var expr2Body = visitor.Replace(expr2.Body);
        
        // İki ifadeyi AND operatörü ile birleştir
        var combinedBody = Expression.AndAlso(expr1.Body, expr2Body);
        
        // Yeni bir lambda ifadesi oluştur
        return Expression.Lambda<Func<T, bool>>(combinedBody, parameter);
    }
}

// Kullanım örneği
public class SearchCriteria
{
    public string CustomerName { get; set; }
    public decimal? MinBalance { get; set; }
    public decimal? MaxBalance { get; set; }
    public bool? IsActive { get; set; }
}

public static Expression<Func<Customer, bool>> BuildSearchQuery(SearchCriteria criteria)
{
    var builder = new QueryBuilder<Customer>();
    
    // İsim filtresi
    if (!string.IsNullOrEmpty(criteria.CustomerName))
    {
        builder.AddFilter(c => c.Name.Contains(criteria.CustomerName));
    }
    
    // Minimum bakiye filtresi
    if (criteria.MinBalance.HasValue)
    {
        builder.AddFilter(c => c.TotalBalance >= criteria.MinBalance.Value);
    }
    
    // Maksimum bakiye filtresi
    if (criteria.MaxBalance.HasValue)
    {
        builder.AddFilter(c => c.TotalBalance <= criteria.MaxBalance.Value);
    }
    
    // Aktiflik filtresi
    if (criteria.IsActive.HasValue)
    {
        builder.AddFilter(c => c.IsActive == criteria.IsActive.Value);
    }
    
    return builder.Build();
}

// Arama örneği
SearchCriteria search = new SearchCriteria
{
    CustomerName = "Ali",
    MinBalance = 10000,
    IsActive = true
};

Expression<Func<Customer, bool>> searchQuery = BuildSearchQuery(search);
Console.WriteLine($"Oluşturulan arama sorgusu: {searchQuery}");

// Entity Framework ile kullanım
using (var context = new CustomerContext())
{
    var searchResults = context.Customers.Where(searchQuery).ToList();
}
```

## 6. Custom Expression Providers (Özel İfade Sağlayıcıları)

Özel ifade sağlayıcıları, belirli bir domain için özelleştirilmiş ifade ağaçları oluşturmak için kullanılabilir. Bu, domain-specific language (DSL) oluşturmak veya karmaşık sorgu senaryolarını basitleştirmek için yararlıdır.

```csharp
// Özel ifade sağlayıcısı
public static class CustomerExpressions
{
    // VIP müşterileri filtreleme
    public static Expression<Func<Customer, bool>> IsVip(decimal threshold = 25000)
    {
        return c => c.TotalBalance > threshold && c.IsActive;
    }
    
    // Belirli bir bakiye aralığındaki müşterileri filtreleme
    public static Expression<Func<Customer, bool>> BalanceBetween(decimal min, decimal max)
    {
        return c => c.TotalBalance >= min && c.TotalBalance <= max;
    }
    
    // İsim araması
    public static Expression<Func<Customer, bool>> NameContains(string searchText)
    {
        return c => c.Name.Contains(searchText);
    }
    
    // İfadeleri birleştirme
    public static Expression<Func<T, bool>> And<T>(
        this Expression<Func<T, bool>> expr1,
        Expression<Func<T, bool>> expr2)
    {
        var parameter = expr1.Parameters[0];
        var visitor = new ParameterReplacer(expr2.Parameters[0], parameter);
        var expr2Body = visitor.Replace(expr2.Body);
        
        var combinedBody = Expression.AndAlso(expr1.Body, expr2Body);
        
        return Expression.Lambda<Func<T, bool>>(combinedBody, parameter);
    }
    
    // İfadeleri birleştirme (OR)
    public static Expression<Func<T, bool>> Or<T>(
        this Expression<Func<T, bool>> expr1,
        Expression<Func<T, bool>> expr2)
    {
        var parameter = expr1.Parameters[0];
        var visitor = new ParameterReplacer(expr2.Parameters[0], parameter);
        var expr2Body = visitor.Replace(expr2.Body);
        
        var combinedBody = Expression.OrElse(expr1.Body, expr2Body);
        
        return Expression.Lambda<Func<T, bool>>(combinedBody, parameter);
    }
}

// Kullanım örneği
var vipQuery = CustomerExpressions.IsVip(30000);
var nameQuery = CustomerExpressions.NameContains("Ali");
var combinedQuery = vipQuery.Or(nameQuery);

Console.WriteLine($"Birleştirilmiş sorgu: {combinedQuery}");

// Entity Framework ile kullanım
using (var context = new CustomerContext())
{
    var results = context.Customers.Where(combinedQuery).ToList();
}
```

### Özel Sorgu Dili Oluşturma

```csharp
// Özel sorgu dili
public class QueryLanguage
{
    // Basit bir sorgu dili ayrıştırıcı
    public static Expression<Func<Customer, bool>> ParseQuery(string query)
    {
        // Örnek sorgu: "balance > 10000 AND active = true"
        // Gerçek bir uygulamada, daha karmaşık bir ayrıştırıcı kullanılmalıdır
        
        var parts = query.Split(new[] { " AND " }, StringSplitOptions.None);
        Expression<Func<Customer, bool>> result = null;
        
        foreach (var part in parts)
        {
            Expression<Func<Customer, bool>> partExpr = ParseCondition(part);
            
            if (result == null)
            {
                result = partExpr;
            }
            else
            {
                result = result.And(partExpr);
            }
        }
        
        return result ?? (c => true);
    }
    
    private static Expression<Func<Customer, bool>> ParseCondition(string condition)
    {
        condition = condition.Trim();
        
        if (condition.Contains(" > "))
        {
            var parts = condition.Split(new[] { " > " }, StringSplitOptions.None);
            var field = parts[0].Trim();
            var value = parts[1].Trim();
            
            if (field == "balance")
            {
                decimal decimalValue = decimal.Parse(value);
                return c => c.TotalBalance > decimalValue;
            }
        }
        else if (condition.Contains(" = "))
        {
            var parts = condition.Split(new[] { " = " }, StringSplitOptions.None);
            var field = parts[0].Trim();
            var value = parts[1].Trim();
            
            if (field == "active")
            {
                bool boolValue = bool.Parse(value);
                return c => c.IsActive == boolValue;
            }
            else if (field == "name")
            {
                string stringValue = value.Trim('"');
                return c => c.Name == stringValue;
            }
        }
        
        throw new ArgumentException($"Geçersiz koşul: {condition}");
    }
}

// Kullanım örneği
string queryString = "balance > 20000 AND active = true";
var parsedQuery = QueryLanguage.ParseQuery(queryString);
Console.WriteLine($"Ayrıştırılan sorgu: {parsedQuery}");

// Entity Framework ile kullanım
using (var context = new CustomerContext())
{
    var results = context.Customers.Where(parsedQuery).ToList();
}
```

## Özet

Bu bölümde, Expression Trees'in (İfade Ağaçları) temel kavramlarını ve kullanım alanlarını inceledik:

1. **Expression<T> vs Func<T>**: İfade ağaçları ile çalıştırılabilir delegeler arasındaki farkları anladık.

2. **Building Expression Trees**: Lambda sözdizimi veya programlı yaklaşım kullanarak ifade ağaçları oluşturmayı öğrendik.

3. **Compiling Expressions**: İfade ağaçlarını çalıştırılabilir koda dönüştürmeyi ve performans etkilerini inceledik.

4. **Expression Tree Visitors**: İfade ağaçlarını dolaşmak, incelemek ve değiştirmek için ziyaretçi desenini kullandık.

5. **Dynamic Query Building**: Çalışma zamanında dinamik sorgular oluşturmak için ifade ağaçlarını kullandık.

6. **Custom Expression Providers**: Belirli bir domain için özelleştirilmiş ifade sağlayıcıları oluşturduk.

Expression Trees, özellikle ORM'ler, dinamik sorgu oluşturma, meta-programlama ve domain-specific language (DSL) oluşturma gibi senaryolarda güçlü bir araçtır. Bu yapıları anlamak ve etkili bir şekilde kullanmak, daha esnek ve güçlü uygulamalar geliştirmenize yardımcı olacaktır. 