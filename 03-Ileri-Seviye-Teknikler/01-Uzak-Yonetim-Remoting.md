# 3.1. Uzak Yönetim (Remoting)

[↩️ Geri: İçindekiler](../README.md)

PowerShell'in bir sistem yöneticisi veya siber güvenlik uzmanı için en dönüştürücü özelliklerinden biri, şüphesiz Uzak Yönetim (Remoting) yeteneğidir. Bu özellik, tek bir makineden ağınızdaki diğer tüm Windows (ve PowerShell yüklü Linux/macOS) makinelerinde komutları güvenli bir şekilde çalıştırmanıza olanak tanır. Artık her sunucuya RDP ile tek tek bağlanmanıza gerek yok.

PowerShell Remoting, arka planda **WinRM (Windows Remote Management)** hizmetini kullanır. Modern Windows Server işletim sistemlerinde WinRM varsayılan olarak etkindir. İstemci (Client) işletim sistemlerinde ise etkinleştirilmesi gerekebilir.

```powershell
# Uzak yönetim için mevcut makineyi yapılandırır (Yönetici olarak çalıştırın)
Enable-PSRemoting -Force
```

## Uzak Yönetim Modelleri

İki temel uzak yönetim modeli vardır:

### 1. Bire Bir (One-to-One): İnteraktif Oturum

Bu model, `ssh` ile bir Linux sunucusuna bağlanmaya çok benzer. Uzak bir makinenin PowerShell konsoluna doğrudan bağlanırsınız ve o andan itibaren yazdığınız her komut, yerel makinenizde değil, uzak makinede çalışır. Bu model için `Enter-PSSession` cmdlet'i kullanılır.

**Sözdizimi:**
```powershell
Enter-PSSession -ComputerName <UzakSunucuAdı>
```

**Örnek:**
```powershell
Enter-PSSession -ComputerName DC01
```
Bu komutu çalıştırdıktan sonra komut isteminiz (prompt) şu şekilde değişir:
`[DC01]: PS C:\Users\Kullanici\Documents>`

Artık `Get-Process`, `Get-Service` gibi komutlar `DC01` sunucusundaki işlemleri ve servisleri listeleyecektir. Oturumdan çıkmak için `Exit-PSSession` veya kısaca `exit` yazmanız yeterlidir.

### 2. Bire Çok (One-to-Many): Komut Gönderme

Bu, PowerShell Remoting'in en güçlü kullanım şeklidir. Tek bir komutu veya bütün bir script'i aynı anda birden çok uzak makinede çalıştırmanıza olanak tanır. Bu model için `Invoke-Command` cmdlet'i kullanılır.

**Sözdizimi:**
```powershell
Invoke-Command -ComputerName <Sunucu1, Sunucu2, ...> -ScriptBlock { # Çalıştırılacak kodlar }
```

**Örnekler:**

1.  **Birden çok sunucudaki disk alanını kontrol etme:**
    ```powershell
    $servers = "WEB01", "SQL01", "DC01"
    Invoke-Command -ComputerName $servers -ScriptBlock {
        Get-Volume | Where-Object DriveLetter
    }
    ```
    Çıktı, hangi sonucun hangi sunucudan geldiğini gösteren `PSComputerName` adlı bir özellik ile birlikte geri döner.

2.  **Tüm domain'deki sunucuları yeniden başlatma (DİKKATLİ KULLANIN!):**
    Active Directory modülünü kullanarak tüm sunucuları alıp `Invoke-Command`'e gönderebiliriz.
    ```powershell
    # Önce sunucuları bir değişkene atayalım
    $allServers = Get-ADComputer -Filter 'OperatingSystem -like "*Server*"' | Select-Object -ExpandProperty Name

    # Şimdi tüm bu sunucularda yeniden başlatma komutunu çalıştıralım
    # -WhatIf parametresi, komutun gerçekten çalışmadan ne yapacağını gösterir. GÜVENLİK İÇİN KULLANIN!
    Invoke-Command -ComputerName $allServers -ScriptBlock {
        Restart-Computer -Force
    } -WhatIf
    ```

## Uzak Oturumda Değişken Kullanımı

Uzak bir oturumda yerel bir değişkeni kullanmak istediğinizde, PowerShell'in bu değişkenin değerini uzak makineye göndermesi gerekir. Bu, `$using:` kapsam değiştiricisi (scope modifier) ile yapılır.

**Örnek:** Belirli bir adı taşıyan servisleri birden çok sunucuda arayalım.
```powershell
# Bu değişken yerel makinemizde tanımlı
$serviceNameToFind = "Spooler"

$servers = "WEB01", "SQL01"

Invoke-Command -ComputerName $servers -ScriptBlock {
    # $using: ile yerel değişkene erişiyoruz
    Get-Service -Name $using:serviceNameToFind
}
```
Eğer `$using:` kullanmasaydık, script bloğu uzak sunucularda `$serviceNameToFind` adında bir değişken arayacak ve bulamadığı için hata verecekti.

## Kalıcı Oturumlar (PSSessions)

Her `Invoke-Command` çağrısı yeni bir bağlantı kurar ve işi bitince kapatır. Eğer aynı sunucular üzerinde art arda birden çok komut çalıştıracaksanız, **kalıcı bir oturum (PSSession)** oluşturmak çok daha verimlidir.

```powershell
$servers = "WEB01", "SQL01"

# 1. Uzak sunucularla kalıcı bir oturum oluştur ve bir değişkende sakla
$session = New-PSSession -ComputerName $servers

# 2. Komutları -Session parametresi ile bu oturum üzerinden çalıştır
Invoke-Command -Session $session -ScriptBlock {
    # İlk komut: Disk durumunu al
    Get-Volume C
}

Invoke-Command -Session $session -ScriptBlock {
    # İkinci komut: Windows Update servisini bul
    Get-Service -Name wuauserv
}

# 3. İşin bittiğinde oturumu kapatarak kaynakları serbest bırak
Remove-PSSession -Session $session
```

---
[➡️ Sonraki Konu: WMI ve CIM ile Sistem Yönetimi](./02-WMI-ve-CIM.md)
