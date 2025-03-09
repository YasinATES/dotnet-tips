# .NET Core Mimarisi

.NET Core, Microsoft tarafından geliştirilen açık kaynaklı, çapraz platform destekli bir uygulama geliştirme platformudur. Bu bölümde .NET Core'un temel mimarisi, bileşenleri ve diğer .NET implementasyonlarıyla karşılaştırması ele alınacaktır.

## .NET Core vs .NET Framework Karşılaştırması

.NET Core ve .NET Framework arasındaki temel farklar şunlardır:

### Platform Desteği
- **.NET Framework**: Sadece Windows işletim sisteminde çalışır.
- **.NET Core**: Windows, Linux, macOS gibi farklı işletim sistemlerinde çalışabilir.

### Açık Kaynak ve Topluluk
- **.NET Framework**: Kapalı kaynak kodlu, Microsoft tarafından geliştirilir.
- **.NET Core**: Açık kaynak kodlu, GitHub üzerinde geliştiriliyor ve topluluk katkılarına açık.

### Modülerlik
- **.NET Framework**: Monolitik yapıda, tüm framework bileşenleri birlikte kurulur.
- **.NET Core**: Modüler yapıda, NuGet paketleri aracılığıyla sadece ihtiyaç duyulan bileşenler uygulamaya dahil edilir.

### Dağıtım Modeli
- **.NET Framework**: Global Assembly Cache (GAC) üzerinden merkezi dağıtım.
- **.NET Core**: Uygulama ile birlikte dağıtım (self-contained) veya çalışma ortamına bağlı dağıtım (framework-dependent).

### Performans
- **.NET Core**: Daha yüksek performans, özellikle asenkron I/O operasyonları ve mikroservis mimarileri için optimize edilmiş.
- **.NET Framework**: Geleneksel Windows uygulamaları için optimize edilmiş.

### API Kapsamı
- **.NET Framework**: Daha geniş API yüzeyi, Windows'a özgü birçok özellik içerir.
- **.NET Core**: Daha küçük API yüzeyi, ancak çapraz platform çalışabilecek şekilde tasarlanmış.

### Geliştirme Modeli
- **.NET Framework**: Daha yavaş güncelleme döngüsü, Windows güncellemeleriyle birlikte gelir.
- **.NET Core**: Daha hızlı güncelleme döngüsü, bağımsız olarak güncellenebilir.

## .NET Standard Nedir?

.NET Standard, farklı .NET implementasyonları arasında kod paylaşımını sağlayan bir API spesifikasyonudur.

### Temel Özellikleri
- Farklı .NET implementasyonları (.NET Framework, .NET Core, Xamarin) arasında ortak bir API seti tanımlar.
- Kütüphane geliştiricilerin birden fazla .NET platformunda çalışabilen kod yazmasını sağlar.
- Sürüm numaraları (1.0, 1.1, 2.0, 2.1) ile ifade edilir ve her sürüm önceki sürümleri kapsar.

### .NET Standard Sürümleri ve Uyumluluk
| .NET Standard | .NET Core | .NET Framework | Xamarin |
|---------------|-----------|----------------|---------|
| 1.0           | 1.0+      | 4.5+           | Tümü    |
| 1.1           | 1.0+      | 4.5+           | Tümü    |
| 1.2           | 1.0+      | 4.5.1+         | Tümü    |
| 1.3           | 1.0+      | 4.6+           | Tümü    |
| 1.4           | 1.0+      | 4.6.1+         | Tümü    |
| 1.5           | 1.0+      | 4.6.1+         | Tümü    |
| 1.6           | 1.0+      | 4.6.1+         | Tümü    |
| 2.0           | 2.0+      | 4.6.1+         | Tümü    |
| 2.1           | 3.0+      | N/A            | Tümü    |

### .NET Standard'ın Geleceği
.NET 5 ve sonrası ile birlikte, .NET Standard'ın yerini tek bir birleşik .NET platformu almıştır. Ancak mevcut .NET Standard kütüphaneleri desteklenmeye devam etmektedir.

## Çapraz Platform Desteği

.NET Core'un en önemli özelliklerinden biri, farklı işletim sistemlerinde çalışabilmesidir.

### Desteklenen İşletim Sistemleri
- **Windows**: Windows 7 SP1+, Windows 8.1, Windows 10, Windows Server 2012 R2+
- **Linux**: Ubuntu, Debian, Fedora, CentOS, RHEL, Alpine ve diğer birçok dağıtım
- **macOS**: macOS 10.13 (High Sierra) ve üzeri

### Çapraz Platform Geliştirme Araçları
- **Visual Studio**: Windows ve macOS için tam entegre geliştirme ortamı
- **Visual Studio Code**: Tüm platformlarda çalışan hafif kod editörü
- **JetBrains Rider**: Çapraz platform .NET IDE'si
- **dotnet CLI**: Komut satırı arayüzü, tüm platformlarda kullanılabilir

### Çapraz Platform Derleme ve Dağıtım
- **Framework-Dependent Deployment (FDD)**: Uygulama, hedef sistemde .NET Core runtime'ın kurulu olmasını gerektirir.
- **Self-Contained Deployment (SCD)**: Uygulama, .NET Core runtime'ı da içerecek şekilde paketlenir.
- **Single-File Deployment**: Tüm uygulama ve bağımlılıkları tek bir çalıştırılabilir dosyada paketlenir.
- **Native AOT Compilation**: Uygulamayı doğrudan makine koduna derleyerek daha hızlı başlatma süresi sağlar.

## Runtime Components (CoreCLR, CoreFX)

.NET Core, iki ana bileşenden oluşur: CoreCLR ve CoreFX.

### CoreCLR (Common Language Runtime)
CoreCLR, .NET Core'un çalışma zamanı bileşenidir ve aşağıdaki özellikleri içerir:

- **JIT (Just-In-Time) Compiler**: IL (Intermediate Language) kodunu makine koduna dönüştürür.
- **GC (Garbage Collector)**: Otomatik bellek yönetimi sağlar.
- **Type System**: Tip güvenliği ve nesne yönelimli programlama desteği sağlar.
- **Exception Handling**: Hata yakalama ve işleme mekanizması.
- **Thread Management**: İş parçacığı yönetimi.
- **Security**: Kod erişim güvenliği ve doğrulama.

### CoreFX (.NET Core Framework)
CoreFX, .NET Core'un sınıf kütüphaneleri koleksiyonudur:

- **System.Collections**: Liste, sözlük, kuyruk gibi veri yapıları.
- **System.IO**: Dosya ve akış işlemleri.
- **System.Net**: Ağ iletişimi ve protokolleri.
- **System.Linq**: Veri sorgulama yetenekleri.
- **System.Text**: Metin işleme ve kodlama.
- **System.Threading**: Asenkron programlama ve iş parçacığı yönetimi.
- **System.Xml**: XML işleme.
- **Microsoft.Extensions**: Yapılandırma, loglama, DI gibi çekirdek altyapı bileşenleri.

### .NET Core SDK
.NET Core SDK, uygulama geliştirmek için gereken araçları içerir:

- **dotnet CLI**: Komut satırı arayüzü.
- **MSBuild**: Derleme sistemi.
- **Roslyn**: C# ve VB.NET derleyicileri.
- **NuGet**: Paket yöneticisi.

## .NET 5 ve Sonrası

.NET 5 ile birlikte, Microsoft .NET platformlarını birleştirme stratejisini başlatmıştır.

### .NET 5
- .NET Core 3.1'in devamı olarak geliştirilmiştir.
- .NET Framework, .NET Core ve Xamarin'i tek bir platform altında birleştirme sürecinin ilk adımıdır.
- Windows Forms ve WPF için geliştirilmiş destek (sadece Windows'ta).
- C# 9.0 ve F# 5.0 dil özellikleri.
- Performans iyileştirmeleri ve yeni GC özellikleri.

### .NET 6
- İlk Long-Term Support (LTS) sürümü.
- Minimal API desteği.
- Hot Reload özelliği.
- C# 10 dil özellikleri.
- .NET MAUI (Multi-platform App UI) için ilk önizleme.
- Apple Silicon (ARM64) desteği.

### .NET 7
- Performans iyileştirmeleri.
- .NET MAUI'nin genel kullanıma sunulması.
- C# 11 dil özellikleri.
- Native AOT derlemede iyileştirmeler.
- Minimal API'lerde zenginleştirmeler.

### .NET 8
- İkinci LTS sürümü.
- Blazor SSR (Server-Side Rendering) ve Blazor United.
- Native AOT desteğinde genişleme.
- C# 12 dil özellikleri.
- Kepler projesi ile enerji verimliliği iyileştirmeleri.
- ASP.NET Core Identity'nin yeniden tasarımı.

### .NET 9
- Non-LTS sürümü (Standard Term Support).
- C# 13 dil özellikleri.
- Geliştirilmiş Native AOT desteği ve daha geniş uygulama senaryoları.
- Blazor ekosisteminde önemli iyileştirmeler.
- ASP.NET Core'da yeni performans optimizasyonları.
- Daha hızlı derleme süreleri ve geliştirme deneyimi iyileştirmeleri.
- Daha kapsamlı AI ve ML entegrasyonu.
- Daha gelişmiş konteyner desteği ve bulut-yerel özellikler.

### .NET 10
- Üçüncü LTS sürümü olması bekleniyor.
- Şu anda preview aşamasında ve geliştirme devam ediyor.
- C# 14 dil özellikleri bekleniyor.
- Quantum computing için genişletilmiş destek.
- WebAssembly (Wasm) desteğinde önemli iyileştirmeler.
- Daha kapsamlı çapraz platform geliştirme yetenekleri.
- Daha gelişmiş AI ve ML entegrasyonu ve araçları.
- Daha fazla performans optimizasyonu ve kaynak kullanımında verimlilik.
- Mikroservis mimarileri için geliştirilmiş araçlar ve kütüphaneler.

## Özet

.NET Core, modern, açık kaynaklı ve çapraz platform bir uygulama geliştirme platformudur. .NET Framework'e göre daha modüler, daha performanslı ve daha esnek bir yapıya sahiptir. .NET 5 ve sonrası ile birlikte, Microsoft tüm .NET platformlarını birleştirerek tek bir birleşik platform oluşturma stratejisini izlemektedir.

.NET Core mimarisi, CoreCLR ve CoreFX olmak üzere iki ana bileşenden oluşur. CoreCLR, çalışma zamanı ortamını sağlarken, CoreFX temel sınıf kütüphanelerini içerir. .NET Core SDK ise geliştirme araçlarını sağlar.

.NET Standard, farklı .NET implementasyonları arasında kod paylaşımını sağlayan bir API spesifikasyonudur. .NET 5 ve sonrası ile birlikte, .NET Standard'ın yerini tek bir birleşik .NET platformu almıştır. 