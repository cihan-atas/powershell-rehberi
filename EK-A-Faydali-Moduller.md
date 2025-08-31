# Ek A: Faydalı PowerShell Modülleri

[↩️ Geri: İçindekiler](./README.md)

PowerShell'in gücü, topluluk tarafından geliştirilen ve belirli görevleri otomatikleştiren binlerce modülden gelir. İşte her PowerShell kullanıcısının bilmesi gereken, hayat kurtaran bazı popüler modüller.

### Pester
**Ne İşe Yarar?** Pester, PowerShell için endüstri standardı haline gelmiş bir test framework'üdür. Yazdığınız script'lerin ve fonksiyonların beklendiği gibi çalıştığını doğrulamak için testler yazmanızı sağlar. Kodunuzun kalitesini ve güvenilirliğini artırmak için vazgeçilmezdir.
```powershell
Install-Module -Name Pester -Scope CurrentUser -Force
```
**Örnek Kullanım (Pester Test Dosyası - `Get-MyFunction.Tests.ps1`):**
```powershell
Describe "Get-MyFunction Testleri" {
    It "Doğru bir değer döndürmeli" {
        $result = Get-MyFunction
        $result | Should -Be "BeklenenDeğer"
    }
}
```

### dbatools
**Ne İşe Yarar?** SQL Server yönetimi ve otomasyonu için yazılmış, 500'den fazla komut içeren devasa bir modüldür. Yedekleme, geri yükleme, taşıma ve sorgulama gibi aklınıza gelebilecek her türlü SQL Server görevini PowerShell üzerinden yapmanızı sağlar.
```powershell
Install-Module -Name dbatools -Scope CurrentUser
```
**Örnek Kullanım:**
```powershell
# SQL01 sunucusundaki tüm veritabanlarının son yedekleme durumunu kontrol et
Get-DbaLastBackup -SqlInstance SQL01
```

### ImportExcel
**Ne İşe Yarar?** Sisteminize Microsoft Excel kurulmasına **gerek kalmadan** `.xlsx` dosyaları oluşturmanızı, okumanızı ve düzenlemenizi sağlar. Raporlama otomasyonları için paha biçilmezdir.
```powershell
Install-Module -Name ImportExcel -Scope CurrentUser
```
**Örnek Kullanım:**
```powershell
# Çalışan tüm işlemleri bir Excel dosyasına, tablo ve grafiklerle birlikte aktar
Get-Process | Export-Excel -Path "C:\Temp\ProcessReport.xlsx" -AutoSize -AutoFilter -Show
```

### PSWriteHTML
**Ne İşe Yarar?** PowerShell çıktılarından görsel olarak zengin ve interaktif HTML raporları oluşturur. Tablolar, grafikler, takvimler ve daha fazlasını içeren profesyonel raporlar hazırlamak için harikadır.
```powershell
Install-Module -Name PSWriteHTML -Scope CurrentUser
```
**Örnek Kullanım:**
```powershell
# Servis durumlarını içeren bir HTML raporu oluştur
New-Html -FilePath "C:\Temp\ServiceReport.html" -Content {
    Get-Service | New-HtmlTable
} -ShowHtml
```

### Az (Azure PowerShell)
**Ne İşe Yarar?** Microsoft Azure kaynaklarını yönetmek için resmi PowerShell modülüdür. Sanal makinelerden depolama hesaplarına, veritabanlarından ağ yapılandırmalarına kadar Azure üzerindeki her şeyi otomatikleştirebilirsiniz.
```powershell
Install-Module -Name Az -Scope CurrentUser -Repository PSGallery -Force
```
**Örnek Kullanım:**
```powershell
# Azure hesabınıza giriş yapın
Connect-AzAccount

# Tüm sanal makineleri listele
Get-AzVM
```

### BurntToast
**Ne İşe Yarar?** Windows 10/11'de görünen bildirimler (toast notifications) oluşturmanızı sağlayan eğlenceli ve kullanışlı bir modüldür. Uzun süren bir script bittiğinde size haber vermesi gibi senaryolar için idealdir.
```powershell
Install-Module -Name BurntToast -Scope CurrentUser
```
**Örnek Kullanım:**
```powershell
# Script bittiğinde bir bildirim gönder
New-BurntToastNotification -Text "Yedekleme tamamlandı!", "Tüm sunucuların yedeklemesi başarıyla alındı."
```

---
[➡️ Sonraki Konu: En İyi Pratikler ve Stil Rehberi](./EK-B-En-Iyi-Pratikler.md)
