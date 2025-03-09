# Code Generation (Kod Üretimi)

.NET ekosisteminde kod üretimi, çalışma zamanında veya derleme zamanında programatik olarak kod oluşturma yeteneğidir. Bu bölümde, .NET'te kod üretimi için kullanılan çeşitli teknikleri ve araçları inceleyeceğiz.

## 1. CodeDOM (Code Document Object Model)

CodeDOM, .NET Framework'ün bir parçası olarak sunulan ve programatik olarak kod üretmeyi sağlayan bir API'dir. Dil bağımsız bir şekilde kod üretimi yapmanıza olanak tanır.

```csharp
using System.CodeDom;
using System.CodeDom.Compiler;
using System.IO;
using Microsoft.CSharp;

// CodeDOM ile basit bir sınıf oluşturma
public void GenerateClassWithCodeDOM()
{
    // Kod üretimi için CodeDOM graph oluşturma
    CodeCompileUnit compileUnit = new CodeCompileUnit();
    
    // Namespace oluşturma
    CodeNamespace codeNamespace = new CodeNamespace("GeneratedCode");
    codeNamespace.Imports.Add(new CodeNamespaceImport("System"));
    compileUnit.Namespaces.Add(codeNamespace);
    
    // Sınıf oluşturma
    CodeTypeDeclaration customerClass = new CodeTypeDeclaration("Customer");
    customerClass.IsClass = true;
    customerClass.TypeAttributes = System.Reflection.TypeAttributes.Public;
    codeNamespace.Types.Add(customerClass);
    
    // Property ekleme
    CodeMemberField idField = new CodeMemberField(typeof(string), "_id");
    idField.Attributes = MemberAttributes.Private;
    customerClass.Members.Add(idField);
    
    // Id property'si
    CodeMemberProperty idProperty = new CodeMemberProperty();
    idProperty.Name = "Id";
    idProperty.Type = new CodeTypeReference(typeof(string));
    idProperty.Attributes = MemberAttributes.Public;
    
    // Getter
    idProperty.GetStatements.Add(
        new CodeMethodReturnStatement(
            new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "_id")));
    
    // Setter
    idProperty.SetStatements.Add(
        new CodeAssignStatement(
            new CodeFieldReferenceExpression(
                new CodeThisReferenceExpression(), "_id"),
            new CodePropertySetValueReferenceExpression()));
    
    customerClass.Members.Add(idProperty);
    
    // ToString metodu ekleme
    CodeMemberMethod toStringMethod = new CodeMemberMethod();
    toStringMethod.Name = "ToString";
    toStringMethod.ReturnType = new CodeTypeReference(typeof(string));
    toStringMethod.Attributes = MemberAttributes.Public | MemberAttributes.Override;
    
    toStringMethod.Statements.Add(
        new CodeMethodReturnStatement(
            new CodeBinaryOperatorExpression(
                new CodePrimitiveExpression("Customer ID: "),
                CodeBinaryOperatorType.Add,
                new CodePropertyReferenceExpression(
                    new CodeThisReferenceExpression(), "Id"))));
    
    customerClass.Members.Add(toStringMethod);
    
    // Kodu C# olarak üretme
    CSharpCodeProvider provider = new CSharpCodeProvider();
    
    using (StringWriter writer = new StringWriter())
    {
        CodeGeneratorOptions options = new CodeGeneratorOptions();
        options.BracingStyle = "C";
        
        provider.GenerateCodeFromCompileUnit(compileUnit, writer, options);
        
        string sourceCode = writer.ToString();
        Console.WriteLine(sourceCode);
        
        // Dosyaya yazma
        File.WriteAllText("Customer.cs", sourceCode);
    }
}
```

CodeDOM'un avantajları:
- Dil bağımsız kod üretimi
- .NET Framework ile entegre
- Farklı dillerde kod üretebilme

Dezavantajları:
- Karmaşık API
- Modern C# özelliklerini tam desteklememe
- Daha yeni alternatiflerle karşılaştırıldığında sınırlı yetenekler

## 2. Roslyn API (Compiler as a Service)

Roslyn, Microsoft'un .NET derleyicilerini API olarak sunduğu bir platformdur. Kod analizi, dönüştürme ve üretimi için güçlü araçlar sağlar.

```csharp
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.CSharp.Syntax;
using static Microsoft.CodeAnalysis.CSharp.SyntaxFactory;

// Roslyn ile kod üretimi
public void GenerateCodeWithRoslyn()
{
    // Namespace oluşturma
    NamespaceDeclarationSyntax namespaceDeclaration = NamespaceDeclaration(
        QualifiedName(
            IdentifierName("GeneratedCode"),
            IdentifierName("Roslyn")))
        .WithUsings(
            SingletonList<UsingDirectiveSyntax>(
                UsingDirective(IdentifierName("System"))));
    
    // Sınıf oluşturma
    ClassDeclarationSyntax classDeclaration = ClassDeclaration("Product")
        .WithModifiers(
            TokenList(
                Token(SyntaxKind.PublicKeyword)));
    
    // Property ekleme
    PropertyDeclarationSyntax idProperty = PropertyDeclaration(
        PredefinedType(Token(SyntaxKind.StringKeyword)),
        Identifier("Id"))
        .WithModifiers(
            TokenList(
                Token(SyntaxKind.PublicKeyword)))
        .WithAccessorList(
            AccessorList(
                List(new[]
                {
                    AccessorDeclaration(SyntaxKind.GetAccessorDeclaration)
                        .WithSemicolonToken(Token(SyntaxKind.SemicolonToken)),
                    AccessorDeclaration(SyntaxKind.SetAccessorDeclaration)
                        .WithSemicolonToken(Token(SyntaxKind.SemicolonToken))
                })));
    
    PropertyDeclarationSyntax nameProperty = PropertyDeclaration(
        PredefinedType(Token(SyntaxKind.StringKeyword)),
        Identifier("Name"))
        .WithModifiers(
            TokenList(
                Token(SyntaxKind.PublicKeyword)))
        .WithAccessorList(
            AccessorList(
                List(new[]
                {
                    AccessorDeclaration(SyntaxKind.GetAccessorDeclaration)
                        .WithSemicolonToken(Token(SyntaxKind.SemicolonToken)),
                    AccessorDeclaration(SyntaxKind.SetAccessorDeclaration)
                        .WithSemicolonToken(Token(SyntaxKind.SemicolonToken))
                })));
    
    // Sınıfa property'leri ekleme
    classDeclaration = classDeclaration.AddMembers(idProperty, nameProperty);
    
    // Namespace'e sınıfı ekleme
    namespaceDeclaration = namespaceDeclaration.AddMembers(classDeclaration);
    
    // Compilation unit oluşturma
    CompilationUnitSyntax compilationUnit = CompilationUnit()
        .WithUsings(
            SingletonList(
                UsingDirective(IdentifierName("System"))))
        .WithMembers(
            SingletonList<MemberDeclarationSyntax>(namespaceDeclaration));
    
    // Kodu formatlama
    string sourceCode = compilationUnit
        .NormalizeWhitespace()
        .ToFullString();
    
    Console.WriteLine(sourceCode);
    
    // Dosyaya yazma
    File.WriteAllText("Product.cs", sourceCode);
}
```

Roslyn'in avantajları:
- Modern C# özelliklerini tam destekleme
- Kod analizi ve dönüştürme yetenekleri
- Zengin API
- Microsoft tarafından aktif geliştirme

Dezavantajları:
- Öğrenme eğrisi dik olabilir
- Bazı senaryolarda karmaşık API kullanımı

## 3. Source Generators (Kaynak Kod Üreticileri)

Source Generators, C# 9.0 ile tanıtılan ve derleme zamanında kod üretmeyi sağlayan bir özelliktir. Mevcut kodu analiz ederek yeni kod üretebilirler.

```csharp
// Source Generator örneği
using Microsoft.CodeAnalysis;
using Microsoft.CodeAnalysis.CSharp;
using Microsoft.CodeAnalysis.CSharp.Syntax;
using Microsoft.CodeAnalysis.Text;
using System.Text;

[Generator]
public class AutoNotifyGenerator : ISourceGenerator
{
    public void Initialize(GeneratorInitializationContext context)
    {
        // Register a syntax receiver that will be created for each generation pass
        context.RegisterForSyntaxNotifications(() => new SyntaxReceiver());
    }

    public void Execute(GeneratorExecutionContext context)
    {
        // Retrieve the populated receiver 
        if (!(context.SyntaxContextReceiver is SyntaxReceiver receiver))
            return;

        // Create the source code
        string attributeText = @"
using System;
namespace AutoNotify
{
    [AttributeUsage(AttributeTargets.Field, Inherited = false, AllowMultiple = false)]
    sealed class AutoNotifyPropertyAttribute : Attribute
    {
        public AutoNotifyPropertyAttribute()
        {
        }
        public string PropertyName { get; set; }
    }
}
";
        // Add the attribute text
        context.AddSource("AutoNotifyAttribute", SourceText.From(attributeText, Encoding.UTF8));

        // Process each field with the attribute
        foreach (IFieldSymbol fieldSymbol in receiver.Fields)
        {
            string namespaceName = fieldSymbol.ContainingNamespace.ToDisplayString();
            string className = fieldSymbol.ContainingType.Name;
            string fieldName = fieldSymbol.Name;
            string fieldType = fieldSymbol.Type.ToDisplayString();
            
            // Get the AutoNotifyPropertyAttribute from the field
            AttributeData attributeData = fieldSymbol.GetAttributes()
                .Single(ad => ad.AttributeClass.ToDisplayString() == "AutoNotify.AutoNotifyPropertyAttribute");
            
            string propertyName = attributeData.NamedArguments
                .SingleOrDefault(kvp => kvp.Key == "PropertyName").Value.Value?.ToString() 
                ?? fieldName.TrimStart('_').Substring(0, 1).ToUpper() + fieldName.TrimStart('_').Substring(1);
            
            string propertyText = $@"
namespace {namespaceName}
{{
    partial class {className}
    {{
        public {fieldType} {propertyName} 
        {{ 
            get => this.{fieldName};
            set
            {{
                this.{fieldName} = value;
                this.PropertyChanged?.Invoke(this, new System.ComponentModel.PropertyChangedEventArgs(nameof({propertyName})));
            }}
        }}
    }}
}}
";
            context.AddSource($"{className}_{propertyName}", SourceText.From(propertyText, Encoding.UTF8));
        }
    }

    class SyntaxReceiver : ISyntaxContextReceiver
    {
        public List<IFieldSymbol> Fields { get; } = new List<IFieldSymbol>();

        public void OnVisitSyntaxNode(GeneratorSyntaxContext context)
        {
            // Look for fields with the [AutoNotifyProperty] attribute
            if (context.Node is FieldDeclarationSyntax fieldDeclaration 
                && fieldDeclaration.AttributeLists.Count > 0)
            {
                foreach (VariableDeclaratorSyntax variable in fieldDeclaration.Declaration.Variables)
                {
                    // Get the symbol for the field
                    IFieldSymbol fieldSymbol = context.SemanticModel.GetDeclaredSymbol(variable) as IFieldSymbol;
                    
                    // Check if the field has the attribute
                    if (fieldSymbol.GetAttributes()
                        .Any(ad => ad.AttributeClass.ToDisplayString() == "AutoNotify.AutoNotifyPropertyAttribute"))
                    {
                        Fields.Add(fieldSymbol);
                    }
                }
            }
        }
    }
}
```

Kullanım örneği:

```csharp
using System.ComponentModel;
using AutoNotify;

namespace MyApp
{
    // Partial class gerekli
    partial class Person : INotifyPropertyChanged
    {
        // AutoNotifyProperty attribute ile işaretlenen alanlar
        [AutoNotifyProperty]
        private string _name;
        
        [AutoNotifyProperty]
        private int _age;
        
        // INotifyPropertyChanged implementasyonu
        public event PropertyChangedEventHandler PropertyChanged;
    }
}
```

Source Generators'ın avantajları:
- Derleme zamanında çalışır, çalışma zamanı maliyeti yoktur
- Mevcut kodu analiz edebilir
- Reflection kullanımını azaltabilir
- Performans avantajı sağlar

Dezavantajları:
- Sadece derleme zamanında çalışır, çalışma zamanında dinamik kod üretemez
- Hata ayıklama zor olabilir

## 4. T4 Templates (Text Template Transformation Toolkit)

T4, Visual Studio ile entegre olan ve metin tabanlı kod üretimi sağlayan bir şablonlama sistemidir.

```
<#@ template debug="false" hostspecific="false" language="C#" #>
<#@ assembly name="System.Core" #>
<#@ import namespace="System.Linq" #>
<#@ import namespace="System.Text" #>
<#@ import namespace="System.Collections.Generic" #>
<#@ output extension=".cs" #>

using System;

namespace GeneratedCode
{
<#
    var entities = new[] { "Customer", "Product", "Order" };
    foreach (var entity in entities)
    {
#>
    public class <#= entity #>
    {
        public Guid Id { get; set; } = Guid.NewGuid();
        public string Name { get; set; }
        public DateTime CreatedDate { get; set; } = DateTime.Now;
        
        public override string ToString()
        {
            return $"<#= entity #>: {Name} (ID: {Id})";
        }
    }
    
<#
    }
#>
}
```

T4 şablonları iki şekilde kullanılabilir:
1. **Tasarım Zamanı Şablonları**: Derleme öncesinde çalıştırılır ve çıktı projeye dahil edilir.
2. **Çalışma Zamanı Şablonları**: Uygulama çalışırken dinamik olarak işlenir.

T4'ün avantajları:
- Visual Studio ile entegrasyon
- Esnek şablonlama
- Hem tasarım zamanında hem de çalışma zamanında kullanılabilir

Dezavantajları:
- Hata ayıklama zor olabilir
- Şablon sözdizimi karmaşık olabilir

## 5. Runtime Code Generation (Çalışma Zamanı Kod Üretimi)

Çalışma zamanında kod üretimi, dinamik olarak C# kodu oluşturup derleyerek çalıştırmanızı sağlar.

```csharp
using Microsoft.CSharp;
using System.CodeDom.Compiler;
using System.Reflection;

// Çalışma zamanında kod derleme
public object CompileAndExecute(string sourceCode, string typeName, string methodName, params object[] args)
{
    // Derleyici parametrelerini ayarlama
    CompilerParameters parameters = new CompilerParameters();
    parameters.GenerateInMemory = true;
    parameters.ReferencedAssemblies.Add("System.dll");
    parameters.ReferencedAssemblies.Add("System.Core.dll");
    parameters.ReferencedAssemblies.Add(Assembly.GetExecutingAssembly().Location);
    
    // Kodu derleme
    using (CSharpCodeProvider provider = new CSharpCodeProvider())
    {
        CompilerResults results = provider.CompileAssemblyFromSource(parameters, sourceCode);
        
        // Derleme hatalarını kontrol etme
        if (results.Errors.HasErrors)
        {
            StringBuilder sb = new StringBuilder();
            sb.AppendLine("Derleme hataları:");
            
            foreach (CompilerError error in results.Errors)
            {
                sb.AppendLine($"Satır {error.Line}: {error.ErrorText}");
            }
            
            throw new Exception(sb.ToString());
        }
        
        // Derlenen assembly'den tipi alma
        Assembly assembly = results.CompiledAssembly;
        Type type = assembly.GetType(typeName);
        
        if (type == null)
        {
            throw new Exception($"Tip bulunamadı: {typeName}");
        }
        
        // Metodu çağırma
        object instance = Activator.CreateInstance(type);
        MethodInfo method = type.GetMethod(methodName);
        
        if (method == null)
        {
            throw new Exception($"Metot bulunamadı: {methodName}");
        }
        
        return method.Invoke(instance, args);
    }
}

// Kullanım örneği
public void RuntimeCodeGenExample()
{
    string sourceCode = @"
using System;

namespace DynamicCode
{
    public class Calculator
    {
        public int Add(int a, int b)
        {
            return a + b;
        }
        
        public int Multiply(int a, int b)
        {
            return a * b;
        }
    }
}";
    
    // Kodu derleme ve çalıştırma
    int result1 = (int)CompileAndExecute(sourceCode, "DynamicCode.Calculator", "Add", 5, 3);
    int result2 = (int)CompileAndExecute(sourceCode, "DynamicCode.Calculator", "Multiply", 5, 3);
    
    Console.WriteLine($"Toplama sonucu: {result1}");
    Console.WriteLine($"Çarpma sonucu: {result2}");
}
```

Çalışma zamanı kod üretiminin avantajları:
- Tamamen dinamik kod oluşturma
- Çalışma zamanında değişen gereksinimlere uyum sağlama
- Eklenti (plugin) sistemleri için uygun

Dezavantajları:
- Performans maliyeti
- Güvenlik riskleri
- Hata ayıklama zorluğu

## 6. IL Generation (IL Üretimi)

IL (Intermediate Language) üretimi, doğrudan .NET ara dilinde kod oluşturmanızı sağlar. Bu, en düşük seviyeli ve en performanslı kod üretim yöntemidir.

```csharp
using System.Reflection;
using System.Reflection.Emit;

// IL ile dinamik metot oluşturma
public void ILGenerationExample()
{
    // DynamicMethod oluşturma
    DynamicMethod dynamicMethod = new DynamicMethod(
        "AddNumbers",           // Metot adı
        typeof(int),            // Dönüş tipi
        new[] { typeof(int), typeof(int) }, // Parametre tipleri
        typeof(ILGenerationExample).Module); // Modül
    
    // IL üreteci alma
    ILGenerator il = dynamicMethod.GetILGenerator();
    
    // IL komutları ekleme
    il.Emit(OpCodes.Ldarg_0);  // İlk parametreyi yükle
    il.Emit(OpCodes.Ldarg_1);  // İkinci parametreyi yükle
    il.Emit(OpCodes.Add);      // Topla
    il.Emit(OpCodes.Ret);      // Sonucu döndür
    
    // Delegate oluşturma
    var addDelegate = (Func<int, int, int>)dynamicMethod.CreateDelegate(typeof(Func<int, int, int>));
    
    // Metodu çağırma
    int result = addDelegate(10, 20);
    Console.WriteLine($"10 + 20 = {result}");
    
    // Daha karmaşık bir örnek: Çarpma ve toplama
    DynamicMethod complexMethod = new DynamicMethod(
        "MultiplyAndAdd",
        typeof(int),
        new[] { typeof(int), typeof(int), typeof(int) },
        typeof(ILGenerationExample).Module);
    
    il = complexMethod.GetILGenerator();
    
    // (a * b) + c
    il.Emit(OpCodes.Ldarg_0);  // a
    il.Emit(OpCodes.Ldarg_1);  // b
    il.Emit(OpCodes.Mul);      // a * b
    il.Emit(OpCodes.Ldarg_2);  // c
    il.Emit(OpCodes.Add);      // (a * b) + c
    il.Emit(OpCodes.Ret);
    
    var complexDelegate = (Func<int, int, int, int>)complexMethod.CreateDelegate(typeof(Func<int, int, int, int>));
    
    int complexResult = complexDelegate(5, 4, 3);  // (5 * 4) + 3 = 23
    Console.WriteLine($"(5 * 4) + 3 = {complexResult}");
}
```

IL üretiminin avantajları:
- Maksimum performans
- Düşük seviyeli kontrol
- Reflection'dan daha hızlı

Dezavantajları:
- Karmaşık API
- IL bilgisi gerektirir
- Hata ayıklama zorluğu

## 7. Performance Impact (Performans Etkisi)

Farklı kod üretim tekniklerinin performans karşılaştırması:

```csharp
using System.Diagnostics;

public void ComparePerformance()
{
    const int iterations = 1000000;
    
    Console.WriteLine("Kod Üretim Teknikleri Performans Karşılaştırması");
    Console.WriteLine("------------------------------------------------");
    
    // 1. Normal metot çağrısı (baseline)
    Func<int, int, int> normalAdd = (a, b) => a + b;
    
    Stopwatch sw = Stopwatch.StartNew();
    
    for (int i = 0; i < iterations; i++)
    {
        int result = normalAdd(5, 10);
    }
    
    sw.Stop();
    Console.WriteLine($"Normal metot çağrısı: {sw.ElapsedMilliseconds} ms");
    
    // 2. Reflection ile çağrı
    MethodInfo reflectionMethod = typeof(Math).GetMethod("Max", new[] { typeof(int), typeof(int) });
    
    sw.Restart();
    
    for (int i = 0; i < iterations; i++)
    {
        int result = (int)reflectionMethod.Invoke(null, new object[] { 5, 10 });
    }
    
    sw.Stop();
    Console.WriteLine($"Reflection ile çağrı: {sw.ElapsedMilliseconds} ms");
    
    // 3. DynamicMethod ile IL üretimi
    DynamicMethod dynamicMethod = new DynamicMethod(
        "AddDynamic",
        typeof(int),
        new[] { typeof(int), typeof(int) },
        typeof(Program).Module);
    
    ILGenerator il = dynamicMethod.GetILGenerator();
    il.Emit(OpCodes.Ldarg_0);
    il.Emit(OpCodes.Ldarg_1);
    il.Emit(OpCodes.Add);
    il.Emit(OpCodes.Ret);
    
    var dynamicDelegate = (Func<int, int, int>)dynamicMethod.CreateDelegate(typeof(Func<int, int, int>));
    
    sw.Restart();
    
    for (int i = 0; i < iterations; i++)
    {
        int result = dynamicDelegate(5, 10);
    }
    
    sw.Stop();
    Console.WriteLine($"DynamicMethod ile IL üretimi: {sw.ElapsedMilliseconds} ms");
    
    // 4. Expression Trees
    ParameterExpression paramA = Expression.Parameter(typeof(int), "a");
    ParameterExpression paramB = Expression.Parameter(typeof(int), "b");
    BinaryExpression body = Expression.Add(paramA, paramB);
    
    var expressionDelegate = Expression.Lambda<Func<int, int, int>>(body, paramA, paramB).Compile();
    
    sw.Restart();
    
    for (int i = 0; i < iterations; i++)
    {
        int result = expressionDelegate(5, 10);
    }
    
    sw.Stop();
    Console.WriteLine($"Expression Trees: {sw.ElapsedMilliseconds} ms");
}
```

## Özet

.NET'te kod üretimi için çeşitli teknikler ve araçlar bulunmaktadır:

1. **CodeDOM**: Dil bağımsız kod üretimi için eski ama güvenilir bir API.
2. **Roslyn API**: Modern C# özellikleri için tam destek sunan güçlü bir kod analizi ve üretim platformu.
3. **Source Generators**: Derleme zamanında kod üretimi için yeni ve verimli bir yöntem.
4. **T4 Templates**: Metin tabanlı şablonlar kullanarak kod üretimi.
5. **Runtime Code Generation**: Çalışma zamanında dinamik olarak kod oluşturma ve derleme.
6. **IL Generation**: En düşük seviyeli ve en performanslı kod üretim yöntemi.

Her tekniğin kendi avantajları ve dezavantajları vardır. Kullanım senaryonuza ve gereksinimlerinize göre en uygun tekniği seçmelisiniz. 