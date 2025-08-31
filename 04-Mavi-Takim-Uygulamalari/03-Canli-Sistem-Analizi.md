# 4.3. Canlı Sistem Analizi

[↩️ Geri: İçindekiler](../README.md)

Olay günlükleri bize geçmişte ne olduğunu söyler, ancak bazen bir anormalliği tespit etmek için o anda sistemde "canlı" olarak neyin çalıştığını görmemiz gerekir. Canlı sistem analizi, çalışan işlemleri, ağ bağlantılarını, sistemdeki kalıcılık mekanizmalarını ve diğer anlık durumları inceleyerek şüpheli aktiviteleri tespit etme sürecidir.

PowerShell, bu analizleri yapmak için zengin bir komut seti sunar.

## Çalışan İşlemleri (Processes) İnceleme

Saldırganların çalıştırdığı zararlı yazılımlar veya araçlar genellikle kendilerini işlemler listesinde belli eder.

**Aranacak Şeyler:**
-   Beklenmedik veya tuhaf isimli işlemler (`svch0st.exe`, `1a2b3c.exe` gibi).
-   Meşru bir işlem adını (`svchost.exe` gibi) kullanan ama beklenmedik bir yolda (`C:\Users\Public\` gibi) çalışan işlemler.
-   Aşırı CPU veya bellek tüketen, açıklaması (Description) olmayan işlemler.

```powershell
# Çalışan tüm işlemleri, çalıştıkları yolu ve açıklamasını listele
Get-Process | Select-Object Name, Id, Path, Description

# 'svchost.exe' gibi meşru bir işlemin nerede çalıştığını kontrol et
Get-Process svchost | Select-Object -Property Path -Unique

# Açıklaması veya şirket adı olmayan tüm işlemleri bul (potansiyel olarak şüpheli)
Get-Process | Where-Object { -not $_.Description -or -not $_.Company }
```

`Get-CimInstance` kullanarak bir işlemin ebeveyn (parent) sürecini de bulabiliriz. Örneğin, `powershell.exe`'yi `winword.exe`'nin başlatması (bir Word belgesindeki makrodan tetiklenmesi) oldukça şüphelidir.
```powershell
Get-CimInstance Win32_Process | Select-Object Name, ProcessId, ParentProcessId, CommandLine
```

## Ağ Bağlantılarını Analiz Etme

Bir sistemdeki ağ bağlantıları, komuta-kontrol (C2) sunucularıyla iletişimi veya veri sızıntısını tespit etmek için en kritik yerlerden biridir.

**Aranacak Şeyler:**
-   Bilinmeyen veya şüpheli IP adreslerine yapılan giden bağlantılar.
-   Standart olmayan portlar (örn: 4444, 8080, 1337) üzerinden iletişim.
-   `powershell.exe`, `rundll32.exe` gibi normalde yoğun ağ trafiği oluşturmaması gereken işlemlerin kurduğu bağlantılar.

```powershell
# 'Established' durumundaki tüm TCP bağlantılarını ve sahibi olan işlemi listele
Get-NetTCPConnection -State Established | Select-Object LocalAddress, RemoteAddress, State, OwningProcess

# Belirli bir işlem (örneğin powershell) tarafından yapılan ağ bağlantılarını bul
$psProc = Get-Process -Name powershell
Get-NetTCPConnection -OwningProcess $psProc.Id
```

## Kalıcılık (Persistence) Mekanizmalarını Kontrol Etme

Saldırganlar, bir sistemi ele geçirdikten sonra bilgisayar yeniden başlasa bile erişimlerini korumak için kalıcılık mekanizmaları bırakırlar. Bunları periyodik olarak kontrol etmek, bir sızmanın erken tespiti için hayati önem taşır.

### 1. Zamanlanmış Görevler (Scheduled Tasks)
```powershell
# Microsoft tarafından oluşturulmamış tüm zamanlanmış görevleri bul
Get-ScheduledTask | Where-Object { $_.Author -ne "Microsoft Corporation" } |
  Select-Object TaskName, Author, State, @{N='Action'; E={$_.Actions.Execute}}
```

### 2. Servisler (Services)
Saldırganlar, zararlı yazılımlarını bir Windows servisi olarak yükleyebilirler.
```powershell
# Otomatik başlayan ve bilinen Windows servisleri dışında olanları bul
# Bu sorgu, 'DisplayName' veya 'PathName' alanlarında şüpheli anahtar kelimeler arayarak geliştirilebilir
Get-CimInstance -ClassName Win32_Service -Filter "StartMode = 'Auto' AND State = 'Running'" |
  Where-Object { $_.PathName -notlike 'C:\Windows\*' }
```

### 3. Kayıt Defteri Başlangıç Noktaları (Registry Run Keys)
En yaygın kalıcılık yöntemlerinden biridir.
```powershell
# Mevcut kullanıcı için başlangıçta çalışan programlar
Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Run'

# Tüm kullanıcılar için başlangıçta çalışan programlar
Get-ItemProperty 'HKLM:\Software\Microsoft\Windows\CurrentVersion\Run'
```
Bu anahtarlarda bulunan bilinmeyen veya şüpheli girdiler incelenmelidir.

### 4. WMI Olay Abonelikleri (WMI Event Subscriptions)
Gelişmiş ve gizli bir kalıcılık tekniğidir. Belirli bir olay (örn: her saat başı) gerçekleştiğinde bir komutun veya script'in çalıştırılmasını sağlar.
```powershell
# WMI kalıcılığını tespit etmek için özel sorgular
Get-WmiObject -Namespace root\subscription -Class __EventFilter
Get-WmiObject -Namespace root\subscription -Class __EventConsumer
Get-WmiObject -Namespace root\subscription -Class __FilterToConsumerBinding
```
Bu komutların herhangi birinin beklenmedik bir sonuç döndürmesi, derinlemesine incelenmelidir.

---
[➡️ Sonraki Bölüm: Kırmızı Takım Uygulamaları](../05-Kirmizi-Takim-Uygulamalari/01-Kesif-ve-Bilgi-Toplama.md)
