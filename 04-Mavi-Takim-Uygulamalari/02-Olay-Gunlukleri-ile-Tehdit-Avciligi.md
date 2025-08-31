# 4.2. Olay Günlükleri ile Tehdit Avcılığı

[↩️ Geri: İçindekiler](../README.md)

**Tehdit Avcılığı (Threat Hunting)**, antivirüs veya EDR gibi otomatik sistemlerin atladığı tehditleri, ağ içerisinde proaktif bir şekilde arama sürecidir. Bu süreçte varsayım şudur: "Saldırgan zaten içerde, ben onun izlerini bulmalıyım." PowerShell, bu izleri aramak için Windows'un zengin olay günlüklerini (Event Logs) sorgulamak için mükemmel bir araçtır.

Windows'taki olay günlüklerini sorgulamak için modern ve en güçlü cmdlet `Get-WinEvent`'tir. Eski `Get-EventLog` cmdlet'ine göre çok daha hızlıdır ve yapılandırılmış XML sorgularını destekler.

## `Get-WinEvent` ile Sorgulama

`Get-WinEvent`'in en verimli kullanım şekli, tüm logları çekip `Where-Object` ile filtrelemek yerine, veriyi **kaynakta** filtreleyen `-FilterHashtable` veya `-FilterXPath` parametrelerini kullanmaktır.

**`-FilterHashtable` Sözdizimi:**
```powershell
$filter = @{
    LogName = '<LogAdı>'
    ID = <OlayID>
    StartTime = (Get-Date).AddDays(-1)
    # ... diğer filtreler
}
Get-WinEvent -FilterHashtable $filter
```

## Yaygın Tehdit Avcılığı Senaryoları

Aşağıda, bir saldırganın bırakabileceği yaygın dijital izleri aramak için kullanılabilecek pratik PowerShell sorguları bulunmaktadır.

### 1. Başarısız ve Başarılı Oturum Açma Denemeleri

-   **Başarısız Oturum Açma (Olay ID 4625):** Kaba kuvvet (brute-force) saldırılarının veya parola tahmin denemelerinin en net göstergesidir.
    ```powershell
    # Son 24 saatteki tüm başarısız oturum açma denemelerini bul
    Get-WinEvent -FilterHashtable @{LogName='Security'; ID=4625; StartTime=(Get-Date).AddDays(-1)} |
      Select-Object TimeCreated, @{N='AccountName';E={$_.Properties[5].Value}}, @{N='SourceIp';E={$_.Properties[19].Value}}
    ```

-   **Başarılı RDP Oturumları:** Bir sisteme kimlerin uzaktan bağlandığını izlemek kritiktir.
    - Olay ID `4624` (Başarılı Oturum Açma) ve Logon Type `10` (RemoteInteractive)
    ```powershell
    $filter = @{
        LogName = 'Security'
        ID = 4624
        Data = '10' # Logon Type 10
    }
    Get-WinEvent -FilterHashtable $filter |
      Select-Object TimeCreated, @{N='User';E={$_.Properties[5].Value}}, @{N='SourceIp';E={$_.Properties[18].Value}}
    ```

### 2. Yeni Süreç (Process) Oluşturma (Olay ID 4688)

Bir saldırganın komut çalıştırdığı, zararlı yazılım indirdiği veya keşif yaptığı anlar, yeni bir process oluşturduğunda loglanır. Bu loglamanın **Group Policy üzerinden etkinleştirilmesi gerekir.**

`Computer Configuration > Policies > Windows Settings > Security Settings > Advanced Audit Policy Configuration > Detailed Tracking > Audit Process Creation`

**Senaryo:** `powershell.exe`'nin şüpheli parametrelerle çalıştırılıp çalıştırılmadığını kontrol edelim.
```powershell
# Son 1 günde "EncodedCommand" veya "NonInteractive" gibi parametrelerle çalıştırılan PowerShell işlemlerini bul
$filter = @{
    LogName = 'Security'
    ID = 4688
    Data = 'powershell.exe'
}
Get-WinEvent -FilterHashtable $filter | Where-Object {
    $_.Message -like '*EncodedCommand*' -or $_.Message -like '*NonInteractive*'
} | Select-Object TimeCreated, @{N='CommandLine';E={$_.Properties[5].Value}}, @{N='ParentProcess';E={$_.Properties[12].Value}}
```
Bu sorgu, "dosyasız" saldırıların veya uzaktan kod yürütme girişimlerinin izlerini bulabilir.

### 3. Zamanlanmış Görev Oluşturma veya Değiştirme (Olay ID 4698, 4702)

Saldırganlar, sistemde kalıcılık (persistence) sağlamak için genellikle zamanlanmış görevler oluşturur.
```powershell
# Son 7 gün içinde oluşturulan veya güncellenen tüm zamanlanmış görevleri listele
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=@(4698, 4702); StartTime=(Get-Date).AddDays(-7)} |
  Select-Object TimeCreated, Id, @{N='TaskName';E={$_.Properties[1].Value}}, @{N='User';E={$_.Properties[5].Value}}
```

### 4. PowerShell Loglarını İnceleme (Olay ID 4104)

Bir önceki bölümde etkinleştirdiğimiz Script Block Logging, tehdit avcılığının altın madenidir.

**Senaryo:** `Invoke-Expression`, `IEX` veya `DownloadString` gibi saldırganların sıkça kullandığı tehlikeli komutları içeren script bloklarını bulalım.
```powershell
$filter = @{
    LogName = 'Microsoft-Windows-PowerShell/Operational'
    ID = 4104
}
Get-WinEvent -FilterHashtable $filter | Where-Object {
    $_.Message -match '(Invoke-Expression|IEX|DownloadString|EncodedCommand)'
} | Select-Object TimeCreated, Message
```
Bu sorgu, karmaşıklaştırılmış (obfuscated) bir kodun arkasındaki gerçek niyeti ortaya çıkarabilir.

### 5. Olay Günlüklerinin Silinmesi (Olay ID 1102)

Bir saldırganın yapacağı ilk işlerden biri, arkasındaki izleri temizlemek için olay günlüklerini silmektir. Bu eylemin kendisi de bir log bırakır. Bu logu görmek, sistemde kesinlikle bir anormallik olduğunun en büyük işaretidir.
```powershell
# Güvenlik olay günlüğünün silinip silinmediğini kontrol et
Get-WinEvent -FilterHashtable @{LogName='Security'; ID=1102}
```
Bu komutun bir sonuç döndürmesi, acil bir olay müdahalesi gerektirir.

---
[➡️ Sonraki Konu: Canlı Sistem Analizi](./03-Canli-Sistem-Analizi.md)
