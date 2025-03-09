# IL Emission (IL Yayımı)

IL Emission, .NET'te çalışma zamanında dinamik olarak IL (Intermediate Language) kodu oluşturma ve yürütme yeteneğidir. Bu bölümde, IL Emission'ın temel kavramlarını ve kullanım alanlarını inceleyeceğiz.

## 1. ILGenerator Usage (ILGenerator Kullanımı)

`ILGenerator`, IL komutlarını programatik olarak oluşturmak için kullanılan temel sınıftır.

```csharp
using System;
using System.Reflection;
using System.Reflection.Emit;

// Temel ILGenerator kullanımı
public void BasicILGeneratorUsage()
{
    // DynamicMethod oluşturma
    DynamicMethod addMethod = new DynamicMethod(
        "Add",                  // Metot adı
        typeof(int),            // Dönüş tipi
        new[] { typeof(int), typeof(int) }, // Parametre tipleri
        typeof(Program).Module); // Modül
    
    // IL üreteci alma
    ILGenerator il = addMethod.GetILGenerator();
    
    // IL komutları ekleme
    il.Emit(OpCodes.Ldarg_0);  // İlk parametreyi yükle
    il.Emit(OpCodes.Ldarg_1);  // İkinci parametreyi yükle
    il.Emit(OpCodes.Add);      // Topla
    il.Emit(OpCodes.Ret);      // Sonucu döndür
    
    // Metodu bir delegate'e dönüştürme
    var addDelegate = (Func<int, int, int>)addMethod.CreateDelegate(typeof(Func<int, int, int>));
    
    // Metodu çağırma
    int result = addDelegate(10, 20);
    Console.WriteLine($"10 + 20 = {result}");
}
```

### Temel IL Komutları

```csharp
// Temel IL komutları örneği
public void CommonILInstructions()
{
    DynamicMethod method = new DynamicMethod(
        "Calculate",
        typeof(int),
        new[] { typeof(int), typeof(int) },
        typeof(Program).Module);
    
    ILGenerator il = method.GetILGenerator();
    
    // Yerel değişken tanımlama
    LocalBuilder localVar = il.DeclareLocal(typeof(int));
    
    // Parametreleri yükleme
    il.Emit(OpCodes.Ldarg_0);  // İlk parametre
    il.Emit(OpCodes.Ldarg_1);  // İkinci parametre
    
    // Çarpma işlemi
    il.Emit(OpCodes.Mul);
    
    // Sonucu yerel değişkene kaydetme
    il.Emit(OpCodes.Stloc, localVar);
    
    // Sabit değer yükleme
    il.Emit(OpCodes.Ldc_I4, 5);
    
    // Yerel değişkeni yükleme ve toplama
    il.Emit(OpCodes.Ldloc, localVar);
    il.Emit(OpCodes.Add);
    
    // Sonucu döndürme
    il.Emit(OpCodes.Ret);
    
    // Metodu bir delegate'e dönüştürme
    var calculateDelegate = (Func<int, int, int>)method.CreateDelegate(typeof(Func<int, int, int>));
    
    // Metodu çağırma: (3 * 4) + 5 = 17
    int result = calculateDelegate(3, 4);
    Console.WriteLine($"(3 * 4) + 5 = {result}");
}
```

### Dallanma ve Etiketler

```csharp
// Dallanma ve etiketler örneği
public void BranchingAndLabels()
{
    DynamicMethod method = new DynamicMethod(
        "Max",
        typeof(int),
        new[] { typeof(int), typeof(int) },
        typeof(Program).Module);
    
    ILGenerator il = method.GetILGenerator();
    
    // Etiket tanımlama
    Label returnSecond = il.DefineLabel();
    Label returnFirst = il.DefineLabel();
    
    // İlk parametreyi yükle
    il.Emit(OpCodes.Ldarg_0);
    
    // İkinci parametreyi yükle
    il.Emit(OpCodes.Ldarg_1);
    
    // Karşılaştırma: ilk parametre < ikinci parametre
    il.Emit(OpCodes.Blt, returnSecond);
    
    // İlk parametre >= ikinci parametre ise buraya gelir
    il.Emit(OpCodes.Ldarg_0);
    il.Emit(OpCodes.Br, returnFirst);
    
    // İkinci parametreyi döndür
    il.MarkLabel(returnSecond);
    il.Emit(OpCodes.Ldarg_1);
    
    // Dönüş
    il.MarkLabel(returnFirst);
    il.Emit(OpCodes.Ret);
    
    // Metodu bir delegate'e dönüştürme
    var maxDelegate = (Func<int, int, int>)method.CreateDelegate(typeof(Func<int, int, int>));
    
    // Metodu çağırma
    Console.WriteLine($"Max(5, 10) = {maxDelegate(5, 10)}");
    Console.WriteLine($"Max(20, 10) = {maxDelegate(20, 10)}");
}
```

## 2. Dynamic Methods (Dinamik Metotlar)

`DynamicMethod`, çalışma zamanında oluşturulan ve çalıştırılan metotlardır.

```csharp
// Dinamik metot örneği
public void DynamicMethodExample()
{
    // Faktöriyel hesaplayan dinamik metot
    DynamicMethod factorial = new DynamicMethod(
        "Factorial",
        typeof(int),
        new[] { typeof(int) },
        typeof(Program).Module);
    
    ILGenerator il = factorial.GetILGenerator();
    
    // Yerel değişkenler
    LocalBuilder result = il.DeclareLocal(typeof(int)); // Sonuç
    LocalBuilder i = il.DeclareLocal(typeof(int));      // Döngü değişkeni
    
    // Etiketler
    Label startLoop = il.DefineLabel();
    Label endLoop = il.DefineLabel();
    
    // result = 1
    il.Emit(OpCodes.Ldc_I4_1);
    il.Emit(OpCodes.Stloc, result);
    
    // i = 1
    il.Emit(OpCodes.Ldc_I4_1);
    il.Emit(OpCodes.Stloc, i);
    
    // Döngü başlangıcı
    il.MarkLabel(startLoop);
    
    // i > n ise döngüden çık
    il.Emit(OpCodes.Ldloc, i);
    il.Emit(OpCodes.Ldarg_0);
    il.Emit(OpCodes.Bgt, endLoop);
    
    // result *= i
    il.Emit(OpCodes.Ldloc, result);
    il.Emit(OpCodes.Ldloc, i);
    il.Emit(OpCodes.Mul);
    il.Emit(OpCodes.Stloc, result);
    
    // i++
    il.Emit(OpCodes.Ldloc, i);
    il.Emit(OpCodes.Ldc_I4_1);
    il.Emit(OpCodes.Add);
    il.Emit(OpCodes.Stloc, i);
    
    // Döngü başına dön
    il.Emit(OpCodes.Br, startLoop);
    
    // Döngü sonu
    il.MarkLabel(endLoop);
    
    // Sonucu döndür
    il.Emit(OpCodes.Ldloc, result);
    il.Emit(OpCodes.Ret);
    
    // Metodu bir delegate'e dönüştürme
    var factorialDelegate = (Func<int, int>)factorial.CreateDelegate(typeof(Func<int, int>));
    
    // Metodu çağırma
    Console.WriteLine($"5! = {factorialDelegate(5)}");
    Console.WriteLine($"10! = {factorialDelegate(10)}");
}
```

### Dinamik Metotların Avantajları

- Çalışma zamanında oluşturulabilir ve değiştirilebilir
- Reflection'a göre daha yüksek performans
- Geçici kullanım için uygundur (assembly oluşturmaya gerek yoktur)
- Erişim kısıtlamalarını aşabilir (private metotlara erişim gibi)

## 3. Type Building (Tip Oluşturma)

`TypeBuilder`, çalışma zamanında tamamen yeni tipler oluşturmak için kullanılır.

```csharp
// Dinamik tip oluşturma
public void DynamicTypeBuilding()
{
    // Assembly ve modül oluşturma
    AssemblyName assemblyName = new AssemblyName("DynamicAssembly");
    AssemblyBuilder assemblyBuilder = AssemblyBuilder.DefineDynamicAssembly(
        assemblyName, AssemblyBuilderAccess.Run);
    
    ModuleBuilder moduleBuilder = assemblyBuilder.DefineDynamicModule("DynamicModule");
    
    // Tip oluşturma
    TypeBuilder typeBuilder = moduleBuilder.DefineType(
        "Person",
        TypeAttributes.Public | TypeAttributes.Class);
    
    // Alan tanımlama
    FieldBuilder nameField = typeBuilder.DefineField(
        "_name",
        typeof(string),
        FieldAttributes.Private);
    
    FieldBuilder ageField = typeBuilder.DefineField(
        "_age",
        typeof(int),
        FieldAttributes.Private);
    
    // Constructor oluşturma
    ConstructorBuilder constructor = typeBuilder.DefineConstructor(
        MethodAttributes.Public,
        CallingConventions.Standard,
        new[] { typeof(string), typeof(int) });
    
    ILGenerator ctorIL = constructor.GetILGenerator();
    
    // this._name = name;
    ctorIL.Emit(OpCodes.Ldarg_0);
    ctorIL.Emit(OpCodes.Ldarg_1);
    ctorIL.Emit(OpCodes.Stfld, nameField);
    
    // this._age = age;
    ctorIL.Emit(OpCodes.Ldarg_0);
    ctorIL.Emit(OpCodes.Ldarg_2);
    ctorIL.Emit(OpCodes.Stfld, ageField);
    
    ctorIL.Emit(OpCodes.Ret);
    
    // Name property oluşturma
    PropertyBuilder nameProperty = typeBuilder.DefineProperty(
        "Name",
        PropertyAttributes.HasDefault,
        typeof(string),
        null);
    
    // Name getter
    MethodBuilder getNameMethod = typeBuilder.DefineMethod(
        "get_Name",
        MethodAttributes.Public | MethodAttributes.SpecialName | MethodAttributes.HideBySig,
        typeof(string),
        Type.EmptyTypes);
    
    ILGenerator getNameIL = getNameMethod.GetILGenerator();
    getNameIL.Emit(OpCodes.Ldarg_0);
    getNameIL.Emit(OpCodes.Ldfld, nameField);
    getNameIL.Emit(OpCodes.Ret);
    
    // Name setter
    MethodBuilder setNameMethod = typeBuilder.DefineMethod(
        "set_Name",
        MethodAttributes.Public | MethodAttributes.SpecialName | MethodAttributes.HideBySig,
        null,
        new[] { typeof(string) });
    
    ILGenerator setNameIL = setNameMethod.GetILGenerator();
    setNameIL.Emit(OpCodes.Ldarg_0);
    setNameIL.Emit(OpCodes.Ldarg_1);
    setNameIL.Emit(OpCodes.Stfld, nameField);
    setNameIL.Emit(OpCodes.Ret);
    
    // Property'ye getter ve setter atama
    nameProperty.SetGetMethod(getNameMethod);
    nameProperty.SetSetMethod(setNameMethod);
    
    // Age property oluşturma (benzer şekilde)
    PropertyBuilder ageProperty = typeBuilder.DefineProperty(
        "Age",
        PropertyAttributes.HasDefault,
        typeof(int),
        null);
    
    // Age getter
    MethodBuilder getAgeMethod = typeBuilder.DefineMethod(
        "get_Age",
        MethodAttributes.Public | MethodAttributes.SpecialName | MethodAttributes.HideBySig,
        typeof(int),
        Type.EmptyTypes);
    
    ILGenerator getAgeIL = getAgeMethod.GetILGenerator();
    getAgeIL.Emit(OpCodes.Ldarg_0);
    getAgeIL.Emit(OpCodes.Ldfld, ageField);
    getAgeIL.Emit(OpCodes.Ret);
    
    // Age setter
    MethodBuilder setAgeMethod = typeBuilder.DefineMethod(
        "set_Age",
        MethodAttributes.Public | MethodAttributes.SpecialName | MethodAttributes.HideBySig,
        null,
        new[] { typeof(int) });
    
    ILGenerator setAgeIL = setAgeMethod.GetILGenerator();
    setAgeIL.Emit(OpCodes.Ldarg_0);
    setAgeIL.Emit(OpCodes.Ldarg_1);
    setAgeIL.Emit(OpCodes.Stfld, ageField);
    setAgeIL.Emit(OpCodes.Ret);
    
    // Property'ye getter ve setter atama
    ageProperty.SetGetMethod(getAgeMethod);
    ageProperty.SetSetMethod(setAgeMethod);
    
    // Tipi oluşturma
    Type personType = typeBuilder.CreateType();
    
    // Oluşturulan tipi kullanma
    object person = Activator.CreateInstance(personType, "Ahmet", 30);
    
    // Property'lere erişim
    Console.WriteLine($"Name: {personType.GetProperty("Name").GetValue(person)}");
    Console.WriteLine($"Age: {personType.GetProperty("Age").GetValue(person)}");
    
    // Property değerini değiştirme
    personType.GetProperty("Name").SetValue(person, "Mehmet");
    Console.WriteLine($"New Name: {personType.GetProperty("Name").GetValue(person)}");
}
```

## 4. Method Building (Metot Oluşturma)

`MethodBuilder`, dinamik tiplere metotlar eklemek için kullanılır.

```csharp
// Dinamik metot oluşturma
public void DynamicMethodBuilding()
{
    // Assembly ve modül oluşturma
    AssemblyName assemblyName = new AssemblyName("CalculatorAssembly");
    AssemblyBuilder assemblyBuilder = AssemblyBuilder.DefineDynamicAssembly(
        assemblyName, AssemblyBuilderAccess.Run);
    
    ModuleBuilder moduleBuilder = assemblyBuilder.DefineDynamicModule("CalculatorModule");
    
    // Tip oluşturma
    TypeBuilder typeBuilder = moduleBuilder.DefineType(
        "Calculator",
        TypeAttributes.Public | TypeAttributes.Class);
    
    // Add metodu oluşturma
    MethodBuilder addMethod = typeBuilder.DefineMethod(
        "Add",
        MethodAttributes.Public | MethodAttributes.Static,
        typeof(int),
        new[] { typeof(int), typeof(int) });
    
    ILGenerator addIL = addMethod.GetILGenerator();
    addIL.Emit(OpCodes.Ldarg_0);
    addIL.Emit(OpCodes.Ldarg_1);
    addIL.Emit(OpCodes.Add);
    addIL.Emit(OpCodes.Ret);
    
    // Multiply metodu oluşturma
    MethodBuilder multiplyMethod = typeBuilder.DefineMethod(
        "Multiply",
        MethodAttributes.Public | MethodAttributes.Static,
        typeof(int),
        new[] { typeof(int), typeof(int) });
    
    ILGenerator multiplyIL = multiplyMethod.GetILGenerator();
    multiplyIL.Emit(OpCodes.Ldarg_0);
    multiplyIL.Emit(OpCodes.Ldarg_1);
    multiplyIL.Emit(OpCodes.Mul);
    multiplyIL.Emit(OpCodes.Ret);
    
    // Tipi oluşturma
    Type calculatorType = typeBuilder.CreateType();
    
    // Metotları çağırma
    int sum = (int)calculatorType.GetMethod("Add").Invoke(null, new object[] { 10, 20 });
    int product = (int)calculatorType.GetMethod("Multiply").Invoke(null, new object[] { 10, 20 });
    
    Console.WriteLine($"10 + 20 = {sum}");
    Console.WriteLine($"10 * 20 = {product}");
}
```

## 5. Performance Optimization (Performans Optimizasyonu)

IL Emission, yüksek performanslı kod oluşturmak için kullanılabilir.

```csharp
// Performans karşılaştırması
public void PerformanceComparison()
{
    const int iterations = 10000000;
    
    Console.WriteLine("Performans Karşılaştırması");
    Console.WriteLine("-------------------------");
    
    // 1. Normal metot çağrısı
    Func<int, int, int> normalAdd = (a, b) => a + b;
    
    var sw = System.Diagnostics.Stopwatch.StartNew();
    
    for (int i = 0; i < iterations; i++)
    {
        int result = normalAdd(5, 10);
    }
    
    sw.Stop();
    Console.WriteLine($"Normal metot: {sw.ElapsedMilliseconds} ms");
    
    // 2. Reflection ile çağrı
    MethodInfo reflectionMethod = typeof(Math).GetMethod("Max", new[] { typeof(int), typeof(int) });
    
    sw.Restart();
    
    for (int i = 0; i < iterations; i++)
    {
        int result = (int)reflectionMethod.Invoke(null, new object[] { 5, 10 });
    }
    
    sw.Stop();
    Console.WriteLine($"Reflection: {sw.ElapsedMilliseconds} ms");
    
    // 3. DynamicMethod ile IL
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
    Console.WriteLine($"DynamicMethod: {sw.ElapsedMilliseconds} ms");
}
```

## 6. Security Concerns (Güvenlik Konuları)

IL Emission, güçlü bir özellik olduğu için güvenlik riskleri taşır.

```csharp
// Güvenlik konuları
public void SecurityConsiderations()
{
    Console.WriteLine("IL Emission Güvenlik Konuları");
    Console.WriteLine("----------------------------");
    
    // 1. Erişim kısıtlamalarını aşma
    Console.WriteLine("1. Erişim kısıtlamalarını aşma:");
    
    // Private alana erişim örneği
    Type stringType = typeof(string);
    FieldInfo privateField = stringType.GetField("_firstChar", 
        BindingFlags.NonPublic | BindingFlags.Instance);
    
    if (privateField != null)
    {
        Console.WriteLine($"Private alan bulundu: {privateField.Name}");
        
        // DynamicMethod ile private alana erişim
        DynamicMethod getPrivateField = new DynamicMethod(
            "GetPrivateField",
            typeof(char),
            new[] { typeof(string) },
            typeof(Program).Module,
            true); // Skip visibility checks
        
        ILGenerator il = getPrivateField.GetILGenerator();
        il.Emit(OpCodes.Ldarg_0);
        il.Emit(OpCodes.Ldfld, privateField);
        il.Emit(OpCodes.Ret);
        
        var fieldAccessor = (Func<string, char>)getPrivateField.CreateDelegate(typeof(Func<string, char>));
        
        try
        {
            string testString = "Hello";
            char firstChar = fieldAccessor(testString);
            Console.WriteLine($"İlk karakter: {firstChar}");
        }
        catch (Exception ex)
        {
            Console.WriteLine($"Erişim hatası: {ex.Message}");
        }
    }
    
    // 2. Kod doğrulama atlatma
    Console.WriteLine("\n2. Kod doğrulama atlatma:");
    Console.WriteLine("IL Emission, CLR'nin kod doğrulama mekanizmalarını atlayabilir.");
    Console.WriteLine("Bu, güvenlik açıklarına yol açabilir.");
    
    // 3. Güvenli kullanım önerileri
    Console.WriteLine("\n3. Güvenli kullanım önerileri:");
    Console.WriteLine("- Kullanıcı girdisine dayalı dinamik kod oluşturmaktan kaçının");
    Console.WriteLine("- Erişim kısıtlamalarını aşarken dikkatli olun");
    Console.WriteLine("- Kod doğrulama ve güvenlik kontrollerini atlamamaya çalışın");
    Console.WriteLine("- Güvenilmeyen kaynaklardan gelen kodu çalıştırmayın");
}
```

## 7. Debugging Techniques (Hata Ayıklama Teknikleri)

IL Emission ile oluşturulan kodun hata ayıklaması zor olabilir.

```csharp
// Hata ayıklama teknikleri
public void DebuggingTechniques()
{
    Console.WriteLine("IL Emission Hata Ayıklama Teknikleri");
    Console.WriteLine("----------------------------------");
    
    // 1. IL kodunu yazdırma
    Console.WriteLine("1. IL kodunu yazdırma:");
    
    DynamicMethod method = new DynamicMethod(
        "DebugExample",
        typeof(int),
        new[] { typeof(int), typeof(int) },
        typeof(Program).Module);
    
    ILGenerator il = method.GetILGenerator();
    il.Emit(OpCodes.Ldarg_0);
    il.Emit(OpCodes.Ldarg_1);
    il.Emit(OpCodes.Add);
    il.Emit(OpCodes.Ret);
    
    // IL kodunu yazdırma yardımcı metodu
    PrintILCode(method);
    
    // 2. Try-catch blokları kullanma
    Console.WriteLine("\n2. Try-catch blokları kullanma:");
    
    DynamicMethod debugMethod = new DynamicMethod(
        "DebugWithTryCatch",
        typeof(int),
        new[] { typeof(int), typeof(int) },
        typeof(Program).Module);
    
    ILGenerator debugIL = debugMethod.GetILGenerator();
    
    // Exception handler için etiketler
    Label tryStart = debugIL.BeginExceptionBlock();
    
    // Try bloğu
    debugIL.Emit(OpCodes.Ldarg_0);
    debugIL.Emit(OpCodes.Ldarg_1);
    debugIL.Emit(OpCodes.Div); // Potansiyel sıfıra bölme hatası
    
    // Catch bloğu
    debugIL.BeginCatchBlock(typeof(DivideByZeroException));
    debugIL.Emit(OpCodes.Pop); // Exception nesnesini stack'ten çıkar
    debugIL.Emit(OpCodes.Ldc_I4_0); // Hata durumunda 0 döndür
    
    // Try-catch bloğunu sonlandır
    debugIL.EndExceptionBlock();
    
    debugIL.Emit(OpCodes.Ret);
    
    var debugDelegate = (Func<int, int, int>)debugMethod.CreateDelegate(typeof(Func<int, int, int>));
    
    Console.WriteLine($"10 / 5 = {debugDelegate(10, 5)}");
    Console.WriteLine($"10 / 0 = {debugDelegate(10, 0)}"); // Hata yakalanır
}

// IL kodunu yazdırma yardımcı metodu
private void PrintILCode(DynamicMethod method)
{
    Console.WriteLine($"Metot: {method.Name}");
    Console.WriteLine("IL Komutları:");
    
    // Not: Gerçek uygulamada, IL kodunu yazdırmak için özel bir kütüphane
    // veya reflection kullanmak gerekebilir. Bu örnek basitleştirilmiştir.
    Console.WriteLine("  ldarg.0");
    Console.WriteLine("  ldarg.1");
    Console.WriteLine("  add");
    Console.WriteLine("  ret");
}
```

## Özet

IL Emission, .NET'te çalışma zamanında dinamik olarak IL kodu oluşturma ve yürütme yeteneğidir. Temel kullanım alanları:

1. **Yüksek Performanslı Kod**: Reflection'a göre daha hızlı çalışan dinamik kod.
2. **Dinamik Tip Oluşturma**: Çalışma zamanında yeni tipler ve metotlar oluşturma.
3. **Erişim Kısıtlamalarını Aşma**: Normalde erişilemeyen alanlara ve metotlara erişim.
4. **Özel Dil Özellikleri**: Dil seviyesinde desteklenmeyen özellikleri uygulama.

IL Emission güçlü bir özellik olduğu için dikkatli kullanılmalıdır. Güvenlik riskleri taşır ve hata ayıklaması zor olabilir. 