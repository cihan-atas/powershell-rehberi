# 5.4. Tespit Edilmekten Kaçınma (Evasion)

[↩️ Geri: İçindekiler](../../README.md)

Kırmızı Takım operasyonunun en zorlu kısmı, Mavi Takım'ın ve güvenlik ürünlerinin (EDR, Antivirüs) radarına yakalanmadan hedeflere ulaşmaktır. Bu bölümde, saldırganların PowerShell kullanırken tespit edilmekten kaçınmak için kullandığı bazı gelişmiş teknikler ele alınmaktadır.

### Loglamayı Devre Dışı Bırakma (T1562.002: Disable Windows Event Logs)

Yeterli yetkilere (genellikle Administrator) sahip bir saldırgan, Mavi Takım'ın en önemli silahı olan PowerShell loglamasını devre dışı bırakmaya çalışabilir.
```powershell
# Script Block Logging'i kayıt defterinden kapatma denemesi
$path = "HKLM:\Software\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging"
Set-ItemProperty -Path $path -Name "EnableScriptBlockLogging" -Value 0
```
Bu eylemin kendisi de genellikle EDR çözümleri tarafından şüpheli olarak işaretlenir.

### AMSI Bypass (T1562.001: Disable or Modify Tools)

AMSI (Antimalware Scan Interface), PowerShell kodunu çalıştırmadan önce bellekte tarayan modern bir savunma katmanıdır. Kırmızı Takım operasyonlarında, operasyonun başında genellikle AMSI'ı devre dışı bırakan bir "bypass" script'i çalıştırılır. Bu bypass'lar genellikle PowerShell'in kendi içindeki `amsi.dll`'in bellek adreslerini manipüle ederek çalışır ve internette birçok örneği bulunur.
```powershell
# Örnek konsept (Gerçek bypass kodu değildir)
# [Ref].Assembly.GetType('System.Management.Automation.AmsiUtils').GetField('amsiInitFailed','NonPublic,Static').SetValue($null,$true)
```
Bu tür bir kodun çalıştırılması, genellikle bir sızmanın ilk ve en net işaretlerinden biridir.

### PowerShell Sürümünü Düşürme (Downgrade Attack)

PowerShell v2, AMSI, Script Block Logging gibi modern güvenlik özelliklerinin hiçbirine sahip değildir. Bir sistemde hala PowerShell v2 motoru etkinse, saldırgan komutlarını bu eski ve güvensiz motor üzerinden çalıştırarak loglamadan kaçınabilir.
```powershell
# PowerShell'i versiyon 2 modunda başlat
powershell.exe -Version 2
```
Bu nedenle, sistem yöneticilerinin Group Policy üzerinden PowerShell v2 özelliğini tamamen kaldırması önerilir.

### Bellekte Yansıtma (In-Memory Reflection)

Saldırganlar, PowerSploit gibi araç setlerini diske hiç yazmadan kullanabilirler. `Invoke-Expression` ile indirilen script, `System.Reflection.Assembly` sınıfı kullanılarak doğrudan belleğe yüklenir ve cmdlet'leri kullanılabilir hale gelir. Bu, disk tabanlı taramaları tamamen atlatır.

Bu bölüm, bir saldırganın PowerShell'i nasıl bir "hayalet" gibi kullanabileceğini göstermektedir. Mavi Takım için bu teknikleri bilmek, doğru logları toplamanın, anormal davranışları izlemenin ve savunma mekanizmalarını sürekli güncel tutmanın ne kadar kritik olduğunu bir kez daha vurgular.

---
[↩️ Geri: İçindekiler](../../README.md)
