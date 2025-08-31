# 3.3. .NET Entegrasyonu

[↩️ Geri: İçindekiler](../README.md)

PowerShell, aslında .NET platformu (.NET Framework veya modern .NET) üzerine inşa edilmiş bir kabuktur. Bu, PowerShell'in sadece bir script dili olmasının ötesinde, .NET'in sunduğu on binlerce sınıfa (class), metoda (method) ve tipe doğrudan erişebildiği anlamına gelir. Bir görev için hazır bir cmdlet bulamadığınızda, .NET'in gücünü kullanarak kendi çözümünüzü yaratabilirsiniz.

## .NET Sınıflarına (Classes) Nasıl Erişilir?

.NET sınıfları, belirli bir işlevi yerine getiren kod kalıplarıdır. PowerShell'de bir .NET sınıfına erişmek için, sınıfın adını köşeli parantezler `[]` içine yazarız.

**Örnek:** Matematiksel işlemler için kullanılan `[System.Math]` sınıfı.
```powershell
# Pi sayısının değerini alalım
[System.Math]::PI

# Bir sayının karekökünü hesaplayalım
[System.Math]::Sqrt(64)

# İki sayıdan büyük olanı bulalım
[System.Math]::Max(10, 20)
```
`::` operatörü, bir sınıfa ait **statik** üyelere (metotlar veya özellikler) erişmek için kullanılır. Statik üyeler, sınıfın bir örneğini oluşturmadan doğrudan çağrılabilir.

## Yeni Nesneler Oluşturma: `New-Object`

Bazı .NET sınıflarını kullanabilmek için, o sınıftan yeni bir **örnek (instance)** oluşturmanız gerekir. Bu işlem `New-Object` cmdlet'i ile yapılır.

**Örnek 1: Bir Web Sitesinden Veri Çekme**
Hazırda `Invoke-WebRequest` cmdlet'i olsa da, .NET'in `WebClient` sınıfını kullanarak bu işin arka planda nasıl yapıldığını görelim.
```powershell
# System.Net.WebClient sınıfından yeni bir nesne oluştur
$webClient = New-Object System.Net.WebClient

# DownloadString metodu ile bir web sayfasının içeriğini indir
$htmlContent = $webClient.DownloadString("https://example.com")

# İçeriğin ilk 200 karakterini göster
$htmlContent.Substring(0, 200)
```

**Örnek 2: Zamanlayıcı (Stopwatch) Kullanımı**
Bir kod bloğunun ne kadar sürede çalıştığını hassas bir şekilde ölçmek için `Stopwatch` sınıfını kullanabiliriz.
```powerserv
# Yeni bir zamanlayıcı nesnesi oluştur
$stopwatch = New-Object System.Diagnostics.Stopwatch

# Zamanlayıcıyı başlat
$stopwatch.Start()

# Ölçmek istediğimiz uzun süren işlem
Start-Sleep -Seconds 2

# Zamanlayıcıyı durdur
$stopwatch.Stop()

# Geçen süreyi ekrana yazdır
Write-Host "İşlem $($stopwatch.Elapsed.TotalSeconds) saniye sürdü."
```

## Statik Metotlar vs. Örnek Metotları

-   **Statik Metotlar:** `[ClassName]::Method()` şeklinde, nesne oluşturmadan doğrudan sınıf üzerinden çağrılır. `[System.Math]::Sqrt()` gibi.
-   **Örnek Metotları:** `New-Object` ile bir nesne oluşturduktan sonra, `$object.Method()` şeklinde o nesne üzerinden çağrılır. `$stopwatch.Start()` gibi.

## Pratik Kullanım Alanları

.NET'e erişim, PowerShell'in yeteneklerini sonsuz bir şekilde genişletir.

### Rasgele Sayı ve Parola Üretme
```powershell
# 0 ile 100 arasında rasgele bir sayı üret
Get-Random -Minimum 0 -Maximum 100

# Daha karmaşık senaryolar için .NET'in Random sınıfı
$random = New-Object System.Random
$random.Next(0, 101) # 0 dahil, 101 hariç
```

### Dosya ve Dizin İşlemleri
PowerShell'in `*-Item` cmdlet'leri çoğu işi görse de, `[System.IO.Path]` sınıfı yol manipülasyonları için çok kullanışlı statik metotlar sunar.
```powershell
$filePath = "C:\Users\Zeus\Documents\script.ps1"

# Bir dosya yolundan sadece dosya adını al
[System.IO.Path]::GetFileName($filePath)
# Çıktı: script.ps1

# Sadece dizin adını al
[System.IO.Path]::GetDirectoryName($filePath)
# Çıktı: C:\Users\Zeus\Documents

# Sadece uzantıyı al
[System.IO.Path]::GetExtension($filePath)
# Çıktı: .ps1
```

### Gelişmiş API Entegrasyonları

Birçok uygulama ve servis, .NET kütüphaneleri (DLL dosyaları) aracılığıyla API'ler sunar. `Add-Type` cmdlet'i ile bu kütüphaneleri PowerShell oturumunuza yükleyebilir ve doğrudan o uygulamanın nesnelerini ve metotlarını kullanabilirsiniz. Bu, PowerShell'i neredeyse her türlü kurumsal uygulama ile entegre etmenin anahtarıdır.

```powershell
# Örnek: Özel bir DLL dosyasını yükleme
Add-Type -Path "C:\Path\To\MyCustomLibrary.dll"

# Artık o kütüphanedeki sınıfları kullanabiliriz
$myObject = New-Object MyCustomLibrary.MyClass
$myObject.DoSomething()
```

---
[➡️ Sonraki Konu: Arka Plan Görevleri (Jobs)](./04-Arka-Plan-Gorevleri-Jobs.md)
