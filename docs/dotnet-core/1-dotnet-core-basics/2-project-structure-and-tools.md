# Proje Yapısı ve Araçlar

.NET ekosisteminde proje yapısı ve geliştirme araçları, verimli ve etkili uygulama geliştirme sürecinin temelini oluşturur. Bu bölümde, .NET projelerinin yapısını ve kullanılan temel araçları inceleyeceğiz.

## SDK ve Runtime Farkları

.NET ekosisteminde SDK (Software Development Kit) ve Runtime arasındaki farkları anlamak önemlidir.

### SDK (Software Development Kit)
SDK, .NET uygulamaları geliştirmek için gereken tüm araçları içerir:

- **Derleyiciler**: C#, F# ve Visual Basic için derleyiciler
- **CLI Araçları**: `dotnet` komut satırı arayüzü
- **MSBuild**: Derleme sistemi
- **NuGet**: Paket yöneticisi
- **Runtime Kütüphaneleri**: Temel sınıf kütüphaneleri
- **Şablonlar**: Proje ve öğe şablonları

SDK, geliştirme makinelerinde kurulu olmalıdır ve uygulama geliştirme, derleme ve paketleme işlemleri için kullanılır.

### Runtime
Runtime, .NET uygulamalarının çalışması için gereken minimum bileşenleri içerir:

- **CoreCLR**: Common Language Runtime, JIT derleyici, garbage collector
- **Temel Kütüphaneler**: Uygulamanın çalışması için gereken minimum kütüphaneler

Runtime, uygulamanın çalıştırılacağı hedef sistemlerde kurulu olmalıdır (framework-dependent deployment durumunda).

### Dağıtım Modelleri

- **Framework-Dependent Deployment (FDD)**: Uygulama, hedef sistemde .NET Runtime'ın kurulu olmasını gerektirir. Daha küçük dağıtım boyutu sağlar.
  
  ```bash
  dotnet publish -c Release
  ```

- **Self-Contained Deployment (SCD)**: Uygulama, .NET Runtime'ı da içerecek şekilde paketlenir. Hedef sistemde .NET kurulu olması gerekmez.
  
  ```bash
  dotnet publish -c Release -r win-x64 --self-contained
  ```

- **Single-File Application**: Tüm uygulama ve bağımlılıkları tek bir çalıştırılabilir dosyada paketlenir.
  
  ```bash
  dotnet publish -c Release -r win-x64 --self-contained -p:PublishSingleFile=true
  ```

- **Native AOT**: Uygulamayı doğrudan makine koduna derleyerek daha hızlı başlatma süresi sağlar.
  
  ```bash
  dotnet publish -c Release -r win-x64 -p:PublishAot=true
  ```

## dotnet CLI Komutları ve Kullanımı

`dotnet` CLI (Command Line Interface), .NET uygulamalarını geliştirmek, derlemek, test etmek ve yayınlamak için kullanılan komut satırı aracıdır.

### Temel Komutlar

#### Proje Oluşturma ve Yönetim
```bash
# Yeni bir konsol uygulaması oluşturma
dotnet new console -n MyConsoleApp

# Yeni bir web uygulaması oluşturma
dotnet new web -n MyWebApp

# Yeni bir sınıf kütüphanesi oluşturma
dotnet new classlib -n MyLibrary

# Mevcut şablonları listeleme
dotnet new list

# Çözüm dosyası oluşturma
dotnet new sln -n MySolution

# Projeyi çözüme ekleme
dotnet sln MySolution.sln add MyConsoleApp/MyConsoleApp.csproj
```

#### Derleme ve Çalıştırma
```bash
# Projeyi derleme
dotnet build

# Projeyi çalıştırma
dotnet run

# Belirli bir yapılandırma ile derleme
dotnet build -c Release

# Projeyi yayınlama
dotnet publish -c Release
```

#### Paket Yönetimi
```bash
# Paket ekleme
dotnet add package Newtonsoft.Json

# Paket kaldırma
dotnet remove package Newtonsoft.Json

# Paketleri geri yükleme
dotnet restore

# Paket referanslarını listeleme
dotnet list package
```

#### Test
```bash
# Testleri çalıştırma
dotnet test

# Belirli bir test projesini çalıştırma
dotnet test MyTests/MyTests.csproj

# Belirli bir filtre ile testleri çalıştırma
dotnet test --filter "Category=UnitTest"
```

#### Araçlar
```bash
# Global araç kurma
dotnet tool install -g dotnet-ef

# Yerel araç kurma
dotnet tool install dotnet-ef

# Kurulu araçları listeleme
dotnet tool list

# Aracı güncelleme
dotnet tool update dotnet-ef

# Aracı kaldırma
dotnet tool uninstall dotnet-ef
```

### Gelişmiş Komutlar

#### Entity Framework Core
```bash
# Migration oluşturma
dotnet ef migrations add InitialCreate

# Veritabanını güncelleme
dotnet ef database update

# Migrations'ları listeleme
dotnet ef migrations list
```

#### Performans Profili
```bash
# Performans profili oluşturma
dotnet-trace collect --process-id <PID>

# Bellek dökümü alma
dotnet-dump collect --process-id <PID>
```

## csproj Dosya Yapısı ve Özellikleri

`.csproj` dosyası, .NET projelerinin yapılandırmasını tanımlayan XML tabanlı bir dosyadır. MSBuild tarafından kullanılır ve projenin nasıl derleneceğini belirtir.

### Temel Yapı
```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>

</Project>
```

### Önemli Özellikler

#### SDK Referansı
```xml
<Project Sdk="Microsoft.NET.Sdk">
```

Farklı SDK türleri:
- `Microsoft.NET.Sdk`: Standart .NET uygulamaları
- `Microsoft.NET.Sdk.Web`: Web uygulamaları
- `Microsoft.NET.Sdk.Worker`: Worker servisleri
- `Microsoft.NET.Sdk.Razor`: Razor kütüphaneleri

#### Hedef Framework
```xml
<TargetFramework>net8.0</TargetFramework>
```

Birden fazla framework hedeflemek için:
```xml
<TargetFrameworks>net8.0;net7.0;netstandard2.1</TargetFrameworks>
```

#### Çıktı Tipi
```xml
<OutputType>Exe</OutputType>
```

Değerler:
- `Exe`: Konsol uygulaması
- `Library`: Sınıf kütüphanesi
- `WinExe`: Windows uygulaması

#### Dil Özellikleri
```xml
<LangVersion>latest</LangVersion>
<Nullable>enable</Nullable>
<ImplicitUsings>enable</ImplicitUsings>
```

#### Assembly Bilgileri
```xml
<AssemblyName>MyCustomName</AssemblyName>
<RootNamespace>MyCompany.MyProject</RootNamespace>
<Version>1.0.0</Version>
<Authors>Your Name</Authors>
<Company>Your Company</Company>
```

#### Derleme Yapılandırması
```xml
<Configuration Condition=" '$(Configuration)' == '' ">Debug</Configuration>
<Platform Condition=" '$(Platform)' == '' ">AnyCPU</Platform>
```

#### Koşullu Derleme
```xml
<PropertyGroup Condition="'$(Configuration)|$(Platform)'=='Debug|AnyCPU'">
  <DefineConstants>DEBUG;TRACE</DefineConstants>
</PropertyGroup>
```

#### Paket Referansları
```xml
<ItemGroup>
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  <PackageReference Include="Microsoft.Extensions.Logging" Version="8.0.0" />
</ItemGroup>
```

#### Proje Referansları
```xml
<ItemGroup>
  <ProjectReference Include="..\MyLibrary\MyLibrary.csproj" />
</ItemGroup>
```

#### Dosya İşlemleri
```xml
<ItemGroup>
  <Content Include="appsettings.json">
    <CopyToOutputDirectory>PreserveNewest</CopyToOutputDirectory>
  </Content>
  <None Include="README.md" />
  <Compile Remove="OldCode.cs" />
</ItemGroup>
```

#### Özel Derleme Adımları
```xml
<Target Name="PreBuild" BeforeTargets="PreBuildEvent">
  <Exec Command="echo Pre-build event" />
</Target>

<Target Name="PostBuild" AfterTargets="PostBuildEvent">
  <Exec Command="echo Post-build event" />
</Target>
```

## NuGet Paket Yönetimi

NuGet, .NET ekosisteminin paket yöneticisidir ve kütüphanelerin dağıtımını, paylaşımını ve kullanımını kolaylaştırır.

### Paket Ekleme

#### CLI ile
```bash
dotnet add package Newtonsoft.Json
dotnet add package Microsoft.EntityFrameworkCore --version 8.0.0
```

#### csproj ile
```xml
<ItemGroup>
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  <PackageReference Include="Microsoft.EntityFrameworkCore" Version="8.0.0" />
</ItemGroup>
```

### Paket Kaynakları

#### Yapılandırma
`nuget.config` dosyası ile paket kaynaklarını yapılandırabilirsiniz:

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="MyCompany" value="https://packages.mycompany.com/nuget/v3/index.json" />
  </packageSources>
  <packageSourceCredentials>
    <MyCompany>
      <add key="Username" value="username" />
      <add key="ClearTextPassword" value="password" />
    </MyCompany>
  </packageSourceCredentials>
</configuration>
```

#### Özel Paket Kaynağı Kullanımı
```bash
dotnet nuget add source https://packages.mycompany.com/nuget/v3/index.json -n MyCompany
dotnet nuget add source https://packages.mycompany.com/nuget/v3/index.json -n MyCompany -u username -p password
```

### Paket Oluşturma

#### Paket Yapılandırması
```xml
<PropertyGroup>
  <PackageId>MyCompany.MyLibrary</PackageId>
  <Version>1.0.0</Version>
  <Authors>Your Name</Authors>
  <Company>Your Company</Company>
  <Description>A description of the package</Description>
  <PackageTags>tag1;tag2;tag3</PackageTags>
  <PackageLicenseExpression>MIT</PackageLicenseExpression>
  <PackageProjectUrl>https://github.com/yourusername/yourproject</PackageProjectUrl>
  <RepositoryUrl>https://github.com/yourusername/yourproject.git</RepositoryUrl>
  <RepositoryType>git</RepositoryType>
  <PackageReadmeFile>README.md</PackageReadmeFile>
  <PackageIcon>icon.png</PackageIcon>
</PropertyGroup>

<ItemGroup>
  <None Include="README.md" Pack="true" PackagePath="\" />
  <None Include="icon.png" Pack="true" PackagePath="\" />
</ItemGroup>
```

#### Paket Oluşturma
```bash
dotnet pack -c Release
```

#### Paket Yayınlama
```bash
dotnet nuget push bin/Release/MyCompany.MyLibrary.1.0.0.nupkg --api-key YOUR_API_KEY --source https://api.nuget.org/v3/index.json
```

### Paket Geri Yükleme
```bash
dotnet restore
```

### Paket Önbelleği Yönetimi
```bash
# Önbelleği temizleme
dotnet nuget locals all --clear

# Önbellek bilgilerini görüntüleme
dotnet nuget locals all --list
```

## Global ve Yerel Araçlar

.NET CLI, global ve yerel olarak kullanılabilen araçları destekler.

### Global Araçlar

Global araçlar, tüm projeler için kullanılabilir ve PATH üzerinden erişilebilir.

#### Kurulum
```bash
dotnet tool install -g dotnet-ef
dotnet tool install -g dotnet-format
```

#### Listeleme
```bash
dotnet tool list -g
```

#### Güncelleme
```bash
dotnet tool update -g dotnet-ef
```

#### Kaldırma
```bash
dotnet tool uninstall -g dotnet-ef
```

### Yerel Araçlar

Yerel araçlar, belirli bir proje veya çözüm için kullanılır ve sürüm kontrolü ile takip edilebilir.

#### Manifest Oluşturma
```bash
dotnet new tool-manifest
```

Bu komut, `.config/dotnet-tools.json` dosyasını oluşturur.

#### Kurulum
```bash
dotnet tool install dotnet-ef
```

#### Listeleme
```bash
dotnet tool list
```

#### Güncelleme
```bash
dotnet tool update dotnet-ef
```

#### Kaldırma
```bash
dotnet tool uninstall dotnet-ef
```

#### Restore
```bash
dotnet tool restore
```

### Yaygın Kullanılan Araçlar

- **dotnet-ef**: Entity Framework Core için komut satırı aracı
  ```bash
  dotnet tool install -g dotnet-ef
  ```

- **dotnet-format**: Kod formatlamak için araç
  ```bash
  dotnet tool install -g dotnet-format
  ```

- **dotnet-outdated**: Güncel olmayan paketleri bulmak için araç
  ```bash
  dotnet tool install -g dotnet-outdated-tool
  ```

- **dotnet-trace**: Performans izleme aracı
  ```bash
  dotnet tool install -g dotnet-trace
  ```

- **dotnet-dump**: Bellek dökümü alma aracı
  ```bash
  dotnet tool install -g dotnet-dump
  ```

- **dotnet-counters**: Performans sayaçları izleme aracı
  ```bash
  dotnet tool install -g dotnet-counters
  ```

## Özet

.NET ekosisteminde proje yapısı ve araçlar, verimli ve etkili uygulama geliştirme sürecinin temelini oluşturur. SDK ve Runtime arasındaki farkları anlamak, dotnet CLI komutlarını etkin kullanmak, csproj dosya yapısını ve özelliklerini bilmek, NuGet paket yönetimini anlamak ve global/yerel araçları kullanmak, .NET geliştiricileri için temel becerilerdir.

Bu bölümde, .NET projelerinin yapısını ve kullanılan temel araçları inceledik. Bu bilgiler, .NET uygulamaları geliştirirken daha verimli ve etkili olmanıza yardımcı olacaktır. 