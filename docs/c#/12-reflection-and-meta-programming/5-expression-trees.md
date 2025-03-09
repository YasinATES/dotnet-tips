# Expression Trees (İfade Ağaçları)

Expression Trees, .NET'te kod yapısını çalışma zamanında temsil etmek, incelemek ve değiştirmek için kullanılan güçlü bir özelliktir. Bu bölümde, Expression Trees'in temel kavramlarını ve kullanım alanlarını inceleyeceğiz.

## 1. Building Expressions (İfadelerin Oluşturulması)

Expression Trees, kod ifadelerini veri yapıları olarak temsil eder. Bu ifadeler manuel olarak oluşturulabilir veya lambda ifadelerinden otomatik olarak dönüştürülebilir.

### Lambda İfadelerinden Expression Trees Oluşturma

```csharp
using System;
using System.Linq.Expressions;

// Lambda ifadesinden Expression Tree oluşturma
public void CreateFromLambda()
{
    // Lambda ifadesi
    Expression<Func<int, int, int>> addExpr = (x, y) => x + y;
    
    // Expression Tree bileşenlerine erişim
    BinaryExpression body = (BinaryExpression)addExpr.Body;
    ParameterExpression left = (ParameterExpression)body.Left;
    ParameterExpression right = (ParameterExpression)body.Right;
    
    Console.WriteLine($"İşlem: {body.NodeType}");
    Console.WriteLine($"Sol parametre: {left.Name}, Tip: {left.Type}");
    Console.WriteLine($"Sağ parametre: {right.Name}, Tip: {right.Type}");
    
    // Daha karmaşık bir örnek
    Expression<Func<int, bool>> isPositive = n => n > 0;
    
    Console.WriteLine($"İfade: {isPositive}");
    Console.WriteLine($"Dönüş tipi: {isPositive.ReturnType}");
    Console.WriteLine($"Parametre sayısı: {isPositive.Parameters.Count}");
}
```

### Manuel Expression Tree Oluşturma

```csharp
// Manuel Expression Tree oluşturma
public void CreateManually()
{
    // x ve y parametreleri
    ParameterExpression x = Expression.Parameter(typeof(int), "x");
    ParameterExpression y = Expression.Parameter(typeof(int), "y");
    
    // x + y ifadesi
    BinaryExpression body = Expression.Add(x, y);
    
    // (x, y) => x + y lambda ifadesi
    Expression<Func<int, int, int>> addExpr = 
        Expression.Lambda<Func<int, int, int>>(body, x, y);
    
    Console.WriteLine($"Oluşturulan ifade: {addExpr}");
    
    // Daha karmaşık bir ifade: (x, y) => x * y + 5
    BinaryExpression multiply = Expression.Multiply(x, y);
    ConstantExpression constant = Expression.Constant(5, typeof(int));
    BinaryExpression addConstant = Expression.Add(multiply, constant);
    
    Expression<Func<int, int, int>> complexExpr = 
        Expression.Lambda<Func<int, int, int>>(addConstant, x, y);
    
    Console.WriteLine($"Karmaşık ifade: {complexExpr}");
}
```

### Koşullu İfadeler Oluşturma

```csharp
// Koşullu ifadeler oluşturma
public void CreateConditionalExpressions()
{
    // n parametresi
    ParameterExpression n = Expression.Parameter(typeof(int), "n");
    
    // n > 0 koşulu
    BinaryExpression condition = Expression.GreaterThan(n, Expression.Constant(0));
    
    // "Positive" değeri
    ConstantExpression trueResult = Expression.Constant("Positive");
    
    // "Non-positive" değeri
    ConstantExpression falseResult = Expression.Constant("Non-positive");
    
    // n > 0 ? "Positive" : "Non-positive" ifadesi
    ConditionalExpression conditional = Expression.Condition(condition, trueResult, falseResult);
    
    // n => n > 0 ? "Positive" : "Non-positive" lambda ifadesi
    Expression<Func<int, string>> expr = 
        Expression.Lambda<Func<int, string>>(conditional, n);
    
    Console.WriteLine($"Koşullu ifade: {expr}");
    
    // Compile edip çalıştırma
    Func<int, string> func = expr.Compile();
    
    Console.WriteLine($"5 için sonuç: {func(5)}");
    Console.WriteLine($"-3 için sonuç: {func(-3)}");
}
```

## 2. Expression Compilation (İfade Derleme)

Expression Trees, `Compile` metodu ile çalıştırılabilir kod parçalarına dönüştürülebilir.

```csharp
// Expression Tree derleme
public void CompileExpressions()
{
    // (x, y) => x + y ifadesi
    ParameterExpression x = Expression.Parameter(typeof(int), "x");
    ParameterExpression y = Expression.Parameter(typeof(int), "y");
    BinaryExpression body = Expression.Add(x, y);
    
    Expression<Func<int, int, int>> addExpr = 
        Expression.Lambda<Func<int, int, int>>(body, x, y);
    
    // İfadeyi derleyerek çalıştırılabilir bir delegate oluşturma
    Func<int, int, int> addFunc = addExpr.Compile();
    
    // Delegate'i çağırma
    int result = addFunc(10, 20);
    Console.WriteLine($"10 + 20 = {result}");
    
    // Daha karmaşık bir ifade: (x, y, z) => (x * y) + z
    ParameterExpression z = Expression.Parameter(typeof(int), "z");
    BinaryExpression multiply = Expression.Multiply(x, y);
    BinaryExpression addZ = Expression.Add(multiply, z);
    
    Expression<Func<int, int, int, int>> complexExpr = 
        Expression.Lambda<Func<int, int, int, int>>(addZ, x, y, z);
    
    Func<int, int, int, int> complexFunc = complexExpr.Compile();
    
    int complexResult = complexFunc(5, 4, 3);  // (5 * 4) + 3 = 23
    Console.WriteLine($"(5 * 4) + 3 = {complexResult}");
    
    // Performans karşılaştırması
    ComparePerformance();
}

// Derleme performansı karşılaştırması
private void ComparePerformance()
{
    const int iterations = 1000000;
    
    // 1. Normal metot
    Func<int, int, int> normalAdd = (a, b) => a + b;
    
    // 2. Derlenmiş Expression
    Expression<Func<int, int, int>> addExpr = (a, b) => a + b;
    Func<int, int, int> compiledAdd = addExpr.Compile();
    
    // Normal metot çağrısı
    var sw = System.Diagnostics.Stopwatch.StartNew();
    
    for (int i = 0; i < iterations; i++)
    {
        int result = normalAdd(5, 10);
    }
    
    sw.Stop();
    Console.WriteLine($"Normal metot: {sw.ElapsedMilliseconds} ms");
    
    // Derlenmiş Expression çağrısı
    sw.Restart();
    
    for (int i = 0; i < iterations; i++)
    {
        int result = compiledAdd(5, 10);
    }
    
    sw.Stop();
    Console.WriteLine($"Derlenmiş Expression: {sw.ElapsedMilliseconds} ms");
}
```

## 3. Dynamic LINQ (Dinamik LINQ)

Expression Trees, dinamik olarak LINQ sorgularını oluşturmak için kullanılabilir.

```csharp
using System.Linq;
using System.Linq.Expressions;

// Dinamik LINQ sorguları
public void DynamicLinqQueries()
{
    // Örnek veri
    var products = new List<Product>
    {
        new Product { Id = 1, Name = "Laptop", Price = 1200, Category = "Electronics" },
        new Product { Id = 2, Name = "Phone", Price = 800, Category = "Electronics" },
        new Product { Id = 3, Name = "Desk", Price = 350, Category = "Furniture" },
        new Product { Id = 4, Name = "Chair", Price = 150, Category = "Furniture" },
        new Product { Id = 5, Name = "Tablet", Price = 600, Category = "Electronics" }
    };
    
    // Dinamik filtreleme
    string propertyName = "Category";
    string propertyValue = "Electronics";
    
    // Ürün parametresi
    ParameterExpression parameter = Expression.Parameter(typeof(Product), "p");
    
    // p.Category
    MemberExpression property = Expression.Property(parameter, propertyName);
    
    // "Electronics"
    ConstantExpression value = Expression.Constant(propertyValue);
    
    // p.Category == "Electronics"
    BinaryExpression equality = Expression.Equal(property, value);
    
    // p => p.Category == "Electronics"
    Expression<Func<Product, bool>> lambda = 
        Expression.Lambda<Func<Product, bool>>(equality, parameter);
    
    // Sorguyu çalıştırma
    var filteredProducts = products.AsQueryable().Where(lambda);
    
    Console.WriteLine($"Filtreleme kriteri: {lambda}");
    Console.WriteLine("Filtrelenmiş ürünler:");
    
    foreach (var product in filteredProducts)
    {
        Console.WriteLine($"- {product.Name} ({product.Category})");
    }
    
    // Dinamik sıralama
    string sortProperty = "Price";
    bool ascending = false;
    
    // Sıralama için lambda ifadesi oluşturma
    ParameterExpression paramSort = Expression.Parameter(typeof(Product), "p");
    MemberExpression propSort = Expression.Property(paramSort, sortProperty);
    
    // Dönüş tipini object olarak belirtme (generic olmayan sıralama için)
    UnaryExpression conversion = Expression.Convert(propSort, typeof(object));
    
    // p => (object)p.Price
    Expression<Func<Product, object>> sortLambda = 
        Expression.Lambda<Func<Product, object>>(conversion, paramSort);
    
    // Sorguyu çalıştırma
    IQueryable<Product> sortedProducts = ascending 
        ? products.AsQueryable().OrderBy(sortLambda)
        : products.AsQueryable().OrderByDescending(sortLambda);
    
    Console.WriteLine($"\nSıralama kriteri: {sortProperty} ({(ascending ? "artan" : "azalan")})");
    Console.WriteLine("Sıralanmış ürünler:");
    
    foreach (var product in sortedProducts)
    {
        Console.WriteLine($"- {product.Name}: ${product.Price}");
    }
}

// Örnek ürün sınıfı
public class Product
{
    public int Id { get; set; }
    public string Name { get; set; }
    public decimal Price { get; set; }
    public string Category { get; set; }
    
    public override string ToString() => $"{Name} (${Price})";
}
```

## 4. Query Providers (Sorgu Sağlayıcıları)

Expression Trees, özel sorgu sağlayıcıları oluşturmak için kullanılabilir. Bu, LINQ sorgularını farklı veri kaynaklarına çevirmek için kullanılır.

```csharp
// Basit bir sorgu sağlayıcısı örneği
public class SimpleQueryProvider : IQueryProvider
{
    public IQueryable CreateQuery(Expression expression)
    {
        Type elementType = TypeSystem.GetElementType(expression.Type);
        return (IQueryable)Activator.CreateInstance(
            typeof(SimpleQueryable<>).MakeGenericType(elementType), 
            new object[] { this, expression });
    }

    public IQueryable<TElement> CreateQuery<TElement>(Expression expression)
    {
        return new SimpleQueryable<TElement>(this, expression);
    }

    public object Execute(Expression expression)
    {
        Console.WriteLine($"Executing expression: {expression}");
        return null;
    }

    public TResult Execute<TResult>(Expression expression)
    {
        Console.WriteLine($"Executing expression: {expression}");
        return default(TResult);
    }
}

// Yardımcı tip sistemi
internal static class TypeSystem
{
    internal static Type GetElementType(Type seqType)
    {
        Type ienum = FindIEnumerable(seqType);
        if (ienum == null) return seqType;
        return ienum.GetGenericArguments()[0];
    }

    private static Type FindIEnumerable(Type seqType)
    {
        if (seqType == null || seqType == typeof(string))
            return null;

        if (seqType.IsArray)
            return typeof(IEnumerable<>).MakeGenericType(seqType.GetElementType());

        if (seqType.IsGenericType)
        {
            foreach (Type arg in seqType.GetGenericArguments())
            {
                Type ienum = typeof(IEnumerable<>).MakeGenericType(arg);
                if (ienum.IsAssignableFrom(seqType))
                {
                    return ienum;
                }
            }
        }

        Type[] ifaces = seqType.GetInterfaces();
        if (ifaces != null && ifaces.Length > 0)
        {
            foreach (Type iface in ifaces)
            {
                Type ienum = FindIEnumerable(iface);
                if (ienum != null) return ienum;
            }
        }

        if (seqType.BaseType != null && seqType.BaseType != typeof(object))
        {
            return FindIEnumerable(seqType.BaseType);
        }

        return null;
    }
}

// Basit bir sorgu sınıfı
public class SimpleQueryable<T> : IQueryable<T>
{
    private readonly Expression _expression;
    private readonly IQueryProvider _provider;

    public SimpleQueryable(IQueryProvider provider)
    {
        _provider = provider;
        _expression = Expression.Constant(this);
    }

    public SimpleQueryable(IQueryProvider provider, Expression expression)
    {
        _provider = provider;
        _expression = expression;
    }

    public Type ElementType => typeof(T);

    public Expression Expression => _expression;

    public IQueryProvider Provider => _provider;

    public IEnumerator<T> GetEnumerator()
    {
        return Provider.Execute<IEnumerable<T>>(Expression).GetEnumerator();
    }

    IEnumerator IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}
```

## 5. Tree Manipulation (Ağaç Manipülasyonu)

Expression Trees, çalışma zamanında değiştirilebilir ve dönüştürülebilir.

```csharp
// Expression Tree manipülasyonu
public void ManipulateExpressionTrees()
{
    // Örnek Expression Tree: x => x + 1
    ParameterExpression parameter = Expression.Parameter(typeof(int), "x");
    ConstantExpression constant = Expression.Constant(1, typeof(int));
    BinaryExpression addition = Expression.Add(parameter, constant);
    
    Expression<Func<int, int>> original = 
        Expression.Lambda<Func<int, int>>(addition, parameter);
    
    Console.WriteLine($"Orijinal ifade: {original}");
    
    // İfadeyi değiştirme: x => x + 1 -> x => x * 2
    ExpressionModifier modifier = new ExpressionModifier();
    Expression<Func<int, int>> modified = (Expression<Func<int, int>>)modifier.Modify(original);
    
    Console.WriteLine($"Değiştirilmiş ifade: {modified}");
    
    // İfadeleri çalıştırma
    Func<int, int> originalFunc = original.Compile();
    Func<int, int> modifiedFunc = modified.Compile();
    
    Console.WriteLine($"Orijinal sonuç (5): {originalFunc(5)}");
    Console.WriteLine($"Değiştirilmiş sonuç (5): {modifiedFunc(5)}");
}

// Expression Tree değiştirici
public class ExpressionModifier : ExpressionVisitor
{
    public Expression Modify(Expression expression)
    {
        return Visit(expression);
    }
    
    protected override Expression VisitBinary(BinaryExpression node)
    {
        // + işlemini * işlemi ile değiştirme
        if (node.NodeType == ExpressionType.Add)
        {
            // Sağ taraftaki sabiti 2 ile değiştirme
            if (node.Right is ConstantExpression)
            {
                return Expression.Multiply(
                    Visit(node.Left),
                    Expression.Constant(2, typeof(int)));
            }
        }
        
        return base.VisitBinary(node);
    }
}
```

## 6. Optimization (Optimizasyon)

Expression Trees, çalışma zamanında optimize edilebilir.

```csharp
// Expression Tree optimizasyonu
public void OptimizeExpressionTrees()
{
    // Optimize edilecek ifade: x => 2 * 3 + x
    ParameterExpression parameter = Expression.Parameter(typeof(int), "x");
    ConstantExpression two = Expression.Constant(2, typeof(int));
    ConstantExpression three = Expression.Constant(3, typeof(int));
    
    BinaryExpression multiply = Expression.Multiply(two, three);
    BinaryExpression addition = Expression.Add(multiply, parameter);
    
    Expression<Func<int, int>> original = 
        Expression.Lambda<Func<int, int>>(addition, parameter);
    
    Console.WriteLine($"Orijinal ifade: {original}");
    
    // İfadeyi optimize etme
    ExpressionOptimizer optimizer = new ExpressionOptimizer();
    Expression<Func<int, int>> optimized = (Expression<Func<int, int>>)optimizer.Optimize(original);
    
    Console.WriteLine($"Optimize edilmiş ifade: {optimized}");
    
    // İfadeleri çalıştırma
    Func<int, int> originalFunc = original.Compile();
    Func<int, int> optimizedFunc = optimized.Compile();
    
    Console.WriteLine($"Orijinal sonuç (5): {originalFunc(5)}");
    Console.WriteLine($"Optimize edilmiş sonuç (5): {optimizedFunc(5)}");
    
    // Performans karşılaştırması
    CompareOptimizationPerformance(originalFunc, optimizedFunc);
}

// Sabit ifadeleri hesaplayan optimizer
public class ExpressionOptimizer : ExpressionVisitor
{
    public Expression Optimize(Expression expression)
    {
        return Visit(expression);
    }
    
    protected override Expression VisitBinary(BinaryExpression node)
    {
        // Alt ifadeleri ziyaret etme
        Expression left = Visit(node.Left);
        Expression right = Visit(node.Right);
        
        // İki taraf da sabit ise, ifadeyi hesapla
        if (left.NodeType == ExpressionType.Constant && 
            right.NodeType == ExpressionType.Constant)
        {
            ConstantExpression leftConst = (ConstantExpression)left;
            ConstantExpression rightConst = (ConstantExpression)right;
            
            switch (node.NodeType)
            {
                case ExpressionType.Add:
                    return Expression.Constant((int)leftConst.Value + (int)rightConst.Value);
                case ExpressionType.Multiply:
                    return Expression.Constant((int)leftConst.Value * (int)rightConst.Value);
                // Diğer işlemler için benzer hesaplamalar eklenebilir
            }
        }
        
        // Değişiklik yoksa veya hesaplanamıyorsa, yeni bir binary expression oluştur
        if (left != node.Left || right != node.Right)
        {
            return Expression.MakeBinary(node.NodeType, left, right);
        }
        
        return node;
    }
}

// Optimizasyon performans karşılaştırması
private void CompareOptimizationPerformance(Func<int, int> original, Func<int, int> optimized)
{
    const int iterations = 10000000;
    
    // Orijinal ifade
    var sw = System.Diagnostics.Stopwatch.StartNew();
    
    for (int i = 0; i < iterations; i++)
    {
        int result = original(5);
    }
    
    sw.Stop();
    Console.WriteLine($"Orijinal ifade: {sw.ElapsedMilliseconds} ms");
    
    // Optimize edilmiş ifade
    sw.Restart();
    
    for (int i = 0; i < iterations; i++)
    {
        int result = optimized(5);
    }
    
    sw.Stop();
    Console.WriteLine($"Optimize edilmiş ifade: {sw.ElapsedMilliseconds} ms");
}
```

## 7. Debugging (Hata Ayıklama)

Expression Trees'in hata ayıklaması için çeşitli teknikler kullanılabilir.

```csharp
// Expression Tree hata ayıklama
public void DebugExpressionTrees()
{
    // Hata ayıklanacak ifade: (x, y) => x * y + 5
    ParameterExpression x = Expression.Parameter(typeof(int), "x");
    ParameterExpression y = Expression.Parameter(typeof(int), "y");
    
    BinaryExpression multiply = Expression.Multiply(x, y);
    ConstantExpression constant = Expression.Constant(5, typeof(int));
    BinaryExpression add = Expression.Add(multiply, constant);
    
    Expression<Func<int, int, int>> expression = 
        Expression.Lambda<Func<int, int, int>>(add, x, y);
    
    // Expression Tree yapısını yazdırma
    ExpressionPrinter printer = new ExpressionPrinter();
    printer.Print(expression);
    
    // Expression Tree'i çalıştırma
    Func<int, int, int> func = expression.Compile();
    int result = func(3, 4);
    
    Console.WriteLine($"Sonuç: {result}");
}

// Expression Tree yazdırıcı
public class ExpressionPrinter : ExpressionVisitor
{
    private int _indent = 0;
    
    public void Print(Expression expression)
    {
        Console.WriteLine("Expression Tree Yapısı:");
        Visit(expression);
    }
    
    private void WriteLine(string text)
    {
        Console.WriteLine(new string(' ', _indent * 2) + text);
    }
    
    public override Expression Visit(Expression node)
    {
        if (node == null)
        {
            WriteLine("null");
            return null;
        }
        
        WriteLine($"Tip: {node.NodeType}, .NET Tipi: {node.Type.Name}");
        _indent++;
        Expression result = base.Visit(node);
        _indent--;
        return result;
    }
    
    protected override Expression VisitBinary(BinaryExpression node)
    {
        WriteLine($"İkili İşlem: {node.NodeType}");
        
        WriteLine("Sol:");
        _indent++;
        Visit(node.Left);
        _indent--;
        
        WriteLine("Sağ:");
        _indent++;
        Visit(node.Right);
        _indent--;
        
        return node;
    }
    
    protected override Expression VisitConstant(ConstantExpression node)
    {
        WriteLine($"Sabit: {node.Value}");
        return node;
    }
    
    protected override Expression VisitParameter(ParameterExpression node)
    {
        WriteLine($"Parametre: {node.Name}");
        return node;
    }
    
    protected override Expression VisitLambda<T>(Expression<T> node)
    {
        WriteLine($"Lambda: {node}");
        
        WriteLine("Parametreler:");
        _indent++;
        foreach (var param in node.Parameters)
        {
            Visit(param);
        }
        _indent--;
        
        WriteLine("Gövde:");
        _indent++;
        Visit(node.Body);
        _indent--;
        
        return node;
    }
}
```

## Özet

Expression Trees, .NET'te kod yapısını çalışma zamanında temsil etmek, incelemek ve değiştirmek için kullanılan güçlü bir özelliktir. Temel kullanım alanları:

1. **Dinamik Kod Oluşturma**: Çalışma zamanında kod oluşturma ve derleme.
2. **LINQ Sorgu Sağlayıcıları**: Farklı veri kaynaklarına LINQ sorgularını çevirme.
3. **Dinamik Sorgular**: Çalışma zamanında oluşturulan sorgular.
4. **Kod Analizi ve Dönüştürme**: Kod yapısını inceleme ve değiştirme.
5. **Optimizasyon**: Çalışma zamanında kod optimizasyonu.

Expression Trees, özellikle ORM'ler, dinamik sorgu oluşturma, meta-programlama ve dil entegrasyonu gibi senaryolarda yaygın olarak kullanılır. 