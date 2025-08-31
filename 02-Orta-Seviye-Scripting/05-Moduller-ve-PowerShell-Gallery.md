# 2.5. Modüller ve PowerShell Gallery

[↩️ Geri: İçindekiler](../README.md)

PowerShell, kutudan çıktığı haliyle güçlü bir araçtır, ancak asıl gücü **genişletilebilir** olmasından gelir. **Modüller**, belirli bir göreve veya teknolojiye (örneğin, Active Directory, Microsoft Azure, AWS, SQL Server) yönelik cmdlet'leri, fonksiyonları, değişkenleri ve diğer kodları bir araya toplayan paketlerdir.

Bu bölümde, bu güç paketlerini nasıl bulacağımızı, kuracağımızı ve yöneteceğimizi öğreneceğiz.

## Modül Nedir?

Bir modül, PowerShell'e yeni komutlar ekleyen bir pakettir. Sisteminizde yüklü olan modülleri listelemek için:
```powershell
Get-Module -ListAvailable
```
PowerShell 3.0 ve sonrası, **modül otomatik yükleme** (module auto-loading) özelliğine sahiptir. Yani, bir modülün içindeki bir komutu ilk kez çağırdığınızda, PowerShell o modülü otomatik olarak mevcut oturuma yükler (`Import-Module`).

## PowerShell Gallery: Komut Evreniniz

**[PowerShell Gallery](https://www.powershellgallery.com/)**, Microsoft tarafından yönetilen, binlerce modül ve script içeren resmi ve güvenilir bir depodur. İhtiyacınız olan hemen her görev için topluluk tarafından yazılmış bir modülü burada bulabilirsiniz.

PowerShell Gallery ile etkileşim kurmak için `PowerShellGet` modülündeki cmdlet'leri kullanırız.

### 1. Modül Bulma: `Find-Module`
PowerShell Gallery'de modül aramak için kullanılır.
```powershell
# İsminde "sql" geçen popüler modülleri ara
Find-Module -Name *sql*

# Belirli bir modül hakkında detaylı bilgi al
Find-Module -Name Pester | Format-List
```
> **⚠️ Sorun Giderme: TLS 1.2 Hatası**
> Özellikle eski sistemlerde `Find-Module` komutu bir ağ hatası verebilir. Bunun nedeni, PowerShell Gallery'nin artık daha güvenli olan TLS 1.2 protokolünü zorunlu kılmasıdır. Çözüm olarak, mevcut oturumunuz için bu protokolü etkinleştirin:
> ```powershell
> [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
> ```

### 2. Modül Kurma: `Install-Module`
Beğendiğiniz bir modülü sisteminize kurar.
```powershell
# dbatools modülünü kuralım
Install-Module -Name dbatools
```

**Önemli Parametreler:**
-   `-Scope CurrentUser`: Modülü sadece kendi kullanıcı profilinize kurar. Yönetici izni gerektirmez ve en güvenli yöntemdir.
    ```powershell
    Install-Module -Name Pester -Scope CurrentUser
    ```
-   `-Scope AllUsers`: Modülü sistemdeki tüm kullanıcılar için kurar. Yönetici olarak çalıştırılması gerekir.
-   `-Force`: Eğer modülün daha eski bir sürümü zaten yüklüyse, üzerine yazarak en güncel sürümü kurar.
-   `-AllowClobber`: Kurulan modüldeki bir komutun adı, sistemde zaten var olan başka bir komutla çakışıyorsa, bu parametre kurulumun devam etmesine izin verir.

### 3. Modülü Güncelleme: `Update-Module`
Yüklü bir modülü PowerShell Gallery'deki en son sürümüne günceller.
```powershell
Update-Module -Name dbatools
```

### 4. Modülü Kullanma: `Import-Module`
Daha önce de belirtildiği gibi, bu işlem genellikle otomatiktir. Ancak bazen, özellikle de isim çakışmaları olduğunda, bir modülü manuel olarak yüklemek isteyebilirsiniz.
```powershell
Import-Module -Name Pester
```

### 5. Modülü Kaldırma
-   **Oturumdan Kaldırma: `Remove-Module`**
    Bir modülü sadece mevcut PowerShell oturumundan kaldırır. Komutları artık kullanılamaz hale gelir, ancak dosyaları silinmez. PowerShell'i yeniden başlattığınızda modül tekrar kullanılabilir.
    ```powershell
    Remove-Module -Name Pester
    ```
-   **Sistemden Silme: `Uninstall-Module`**
    Bir modülü ve tüm dosyalarını sistemden kalıcı olarak siler.
    ```powershell
    Uninstall-Module -Name Pester
    ```

## Alias'lar: Komut Kısayolları

Modüllerle doğrudan ilgili olmasa da, PowerShell'in genişletilebilirliğinin bir başka parçası da **Alias**'lardır. Alias'lar, uzun cmdlet isimleri için tanımlanmış takma isimler veya kısayollardır.

-   **Mevcut Alias'ları Listeleme:**
    ```powershell
    Get-Alias
    ```
    Örneğin, `gci`'nin `Get-ChildItem` için, `gps`'in `Get-Process` için bir alias olduğunu göreceksiniz. Linux'tan gelenler için `ls`, `cat`, `ps` gibi birçok tanıdık alias da mevcuttur.

-   **Yeni Alias Oluşturma:**
    Sık kullandığınız uzun bir komut için kendi kısayolunuzu oluşturabilirsiniz.
    ```powershell
    # Get-ServiceStatus fonksiyonu için 'gss' alias'ını oluştur
    New-Alias -Name gss -Value Get-ServiceStatus
    ```
    Bu alias sadece mevcut oturum için geçerlidir. Kalıcı hale getirmek için PowerShell profilinize eklemeniz gerekir.

---
[➡️ Sonraki Bölüm: İleri Seviye Teknikler](../03-Ileri-Seviye-Teknikler/01-Uzak-Yonetim-Remoting.md)
