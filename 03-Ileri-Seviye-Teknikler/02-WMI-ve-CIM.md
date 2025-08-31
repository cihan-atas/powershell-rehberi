# 3.2. WMI ve CIM ile Sistem Yönetimi

[↩️ Geri: İçindeler](../README.md)

PowerShell cmdlet'lerinin yetersiz kaldığı durumlarda, bir Windows sisteminin en derinliklerindeki donanım ve yazılım bilgilerine erişmek için daha alt seviye yönetim arayüzlerine ihtiyaç duyarız. Windows'ta bu arayüzler **WMI (Windows Management Instrumentation)** ve onun modern hali olan **CIM (Common Information Model)**'dir.

Bu teknolojiler, işletim sistemindeki yönetilebilir her şeyi (CPU, BIOS, kurulu programlar, kullanıcı hesapları, kayıt defteri ayarları vb.) standart bir yapıda sorgulamanıza olanak tanır.

## WMI vs. CIM: Neden CIM Kullanmalıyız?

-   **WMI (`Get-WmiObject`):** Eski, DCOM protokolünü kullanan ve sadece Windows'a özgü olan teknolojidir. Artık geliştirilmemektedir ve PowerShell'in modern (7+) sürümlerinde varsayılan olarak bulunmaz.
-   **CIM (`Get-CimInstance`):** Modern, WS-Man protokolünü (PowerShell Remoting'in de kullandığı protokol) kullanan, standartlara dayalı ve platformlar arası bir teknolojidir. `Get-CimInstance` cmdlet'i daha hızlı, daha güvenli ve ağ üzerinde daha verimlidir.

> **🚀 En İyi Pratik:** Mümkün olan her zaman `Get-WmiObject` yerine `Get-CimInstance` kullanın. Bu rehberde `Get-CimInstance` üzerine odaklanacağız.

## CIM Sınıflarını (Classes) Keşfetme

CIM, tüm bilgileri **sınıflar** halinde düzenler. Örneğin, `Win32_Processor` sınıfı işlemci hakkında bilgi tutarken, `Win32_OperatingSystem` sınıfı işletim sistemi hakkında bilgi tutar.

İhtiyacınız olan sınıfı bulmak için `Get-CimClass` cmdlet'ini joker karakterlerle kullanabilirsiniz.

```powershell
# Adında "disk" geçen tüm CIM sınıflarını bul
Get-CimClass -ClassName *disk*

# Adında "network" veya "adapter" geçen sınıfları bul
Get-CimClass -ClassName *network*
Get-CimClass -ClassName *adapter*
```
Binlerce sınıf mevcuttur. En yaygın kullanılanlar genellikle `Win32_` önekiyle başlar.

## Veri Sorgulama: `Get-CimInstance`

Bir sınıftaki bilgileri (örnekleri/instances) almak için `Get-CimInstance` kullanılır.

**Örnekler:**

1.  **İşletim Sistemi Bilgilerini Alma:**
    ```powershell
    # Win32_OperatingSystem sınıfının tek bir örneği vardır.
    Get-CimInstance -ClassName Win32_OperatingSystem | Select-Object Caption, OSArchitecture, LastBootUpTime
    ```

2.  **BIOS Seri Numarasını Öğrenme:**
    ```powershell
    Get-CimInstance -ClassName Win32_BIOS | Select-Object SerialNumber
    ```

3.  **Mantıksal Disk (Sürücü) Bilgilerini Listeleme:**
    Bu sınıf, sistemdeki her bir sürücü için birden çok örnek döndürür.
    ```powershell
    Get-CimInstance -ClassName Win32_LogicalDisk | Format-Table DeviceID, VolumeName, Size, FreeSpace
    ```
    `Size` ve `FreeSpace` özelliklerinin byte cinsinden olduğuna dikkat edin. Bunları GB'a çevirebiliriz:
    ```powershell
    Get-CimInstance -ClassName Win32_LogicalDisk |
      Select-Object DeviceID,
                    @{Name="Size(GB)"; Expression={$_.Size / 1GB -as [int]}},
                    @{Name="FreeSpace(GB)"; Expression={$_.FreeSpace / 1GB -as [int]}}
    ```

## Filtreleme ve Uzak Sistemlerde Kullanım

`Get-CimInstance`, veriyi **kaynakta** filtrelemek için `-Filter` parametresini kullanır. Bu, tüm veriyi çekip sonra `Where-Object` ile filtrelemekten çok daha verimlidir. Filtreleme dili **WQL (WMI Query Language)** olarak bilinir ve SQL'e çok benzer.

**Örnek:** Sadece C: sürücüsü hakkındaki bilgiyi alalım.
```powershell
# WQL sözdiziminde string'ler tek tırnak içinde olmalıdır.
Get-CimInstance -ClassName Win32_LogicalDisk -Filter "DeviceID = 'C:'"
```

CIM, PowerShell Remoting ile aynı protokolü kullandığı için, uzak sistemlerden bilgi almak son derece kolaydır.
```powershell
# Uzak bir sunucudaki otomatik başlayan servisleri listele
Get-CimInstance -ClassName Win32_Service -Filter "StartMode = 'Auto'" -ComputerName WEB01
```

## Eylem Gerçekleştirme: `Invoke-CimMethod`

CIM sınıfları sadece veri sağlamakla kalmaz, aynı zamanda eylemleri tetikleyebilen **metotlara** da sahiptir. `Invoke-CimMethod` cmdlet'i ile bu metotları çağırabiliriz.

**Örnek:** Bir işlemi sonlandıralım.
```powershell
# 1. Sonlandırmak istediğimiz Notepad işlemini bulalım
$notepadProcess = Get-CimInstance -ClassName Win32_Process -Filter "Name = 'notepad.exe'"

# 2. Bu nesne üzerinde 'Terminate' metodunu çağıralım
Invoke-CimMethod -InputObject $notepadProcess -MethodName Terminate
```
Bu yöntem, `Stop-Process` cmdlet'inin yapamadığı durumlarda veya daha alt seviye kontrol gerektiğinde kullanışlı olabilir. Örneğin, uzak bir sistemde yeni bir işlem başlatmak (`Win32_Process` sınıfının `Create` metodu ile) bu yolla mümkündür.

---
[➡️ Sonraki Konu: .NET Framework Entegrasyonu](./03-NET-Framework-Entegrasyonu.md)
