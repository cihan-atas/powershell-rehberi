# 4.1. Güvenlik Loglama ve Auditing

[↩️ Geri: İçindeler](../README.md)

PowerShell, saldırganlar için güçlü bir araç olabileceği gibi, savunmacılar (Mavi Takım) için de paha biçilmez bir izleme ve tespit aracıdır. Microsoft, PowerShell'i daha güvenli hale getirmek ve "kim, ne zaman, ne yaptı?" sorusuna cevap verebilmek için PowerShell 5.0 ile birlikte bir dizi güçlü loglama ve denetim (auditing) özelliği sunmuştur. Bu özellikler, Group Policy (GPO) üzerinden etkinleştirildiğinde, bir saldırganın PowerShell üzerinden yaptığı hemen her eylemi görünür kılar.

## PowerShell'in Temel Güvenlik Mekanizmaları

### 1. Script Block Logging (Olay ID: 4104)

Bu, Mavi Takım için **en önemli** loglama özelliğidir. Etkinleştirildiğinde, PowerShell motoru tarafından işlenen **her bir komut bloğunu** olay günlüğüne kaydeder.

-   **Neyi Loglar?** Bir kullanıcının interaktif olarak girdiği komutları, bir script dosyasının (`.ps1`) içeriğini, `Invoke-Expression` gibi dinamik olarak çalıştırılan kodları.
-   **Neden Kritik?** Saldırganlar, kodlarını gizlemek için genellikle karmaşıklaştırma (obfuscation) teknikleri kullanır. Script Block Logging, bu karmaşıklaştırılmış kodun, PowerShell motoru tarafından çalıştırılmadan hemen önceki **orijinal, temiz halini** loglar. Bu, saldırganın gerçek niyetini ortaya çıkarır.

**Etkinleştirme (Group Policy):**
`Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell > Turn on PowerShell Script Block Logging`

**Olay Günlüğü Konumu:**
`Applications and Services Logs > Microsoft > Windows > PowerShell > Operational`

### 2. Module Logging (Olay ID: 4103)

Bu özellik, belirli PowerShell modüllerinden komutlar çalıştırıldığında olay günlüğüne detaylı loglar bırakır. Pipeline'dan geçen her komut ve parametreleri loglanır.
- **Neyi Loglar?** Hangi komutun hangi parametrelerle çalıştırıldığı gibi detayları.
- **Nasıl Kullanılır?** Saldırı araçlarında sıkça kullanılan modülleri (`NetTCPIP`, `ActiveDirectory` vb.) veya şüpheli modülleri izlemek için kullanılır. Modül adlarını joker karakterlerle (`*`) belirterek tüm modülleri izlemek de mümkündür.

**Etkinleştirme (Group Policy):**
`Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell > Turn on Module Logging`

### 3. Transcription (Oturum Kaydı)

Transcription, bir kullanıcının tüm PowerShell oturumunu (girdiği her komut ve aldığı her çıktı) belirtilen bir merkezi dosya sunucusundaki bir metin dosyasına kaydeder.
- **Neyi Kaydeder?** Tam bir adli kanıt niteliğinde, oturumun baştan sona dökümünü.
- **Avantajı:** Bir saldırgan yerel olay günlüklerini silse bile, merkezi sunucudaki bu transkript kayıtları güvende kalır.

**Etkinleştirme (Group Policy):**
`Computer Configuration > Administrative Templates > Windows Components > Windows PowerShell > Turn on PowerShell Transcription`
Burada, transkriptlerin kaydedileceği bir ağ yolu (`\\FileServer\PSTranscripts`) belirtmeniz gerekir.

## AMSI (Antimalware Scan Interface)

AMSI, PowerShell'in yerleşik bir güvenlik özelliğidir ve bir antivirüs (AV) veya EDR (Endpoint Detection and Response) çözümünün, script'ler **çalıştırılmadan hemen önce bellekteki içeriğini** taramasına olanak tanıyan bir arayüzdür.

- **Nasıl Çalışır?** `Invoke-Expression` ile çalıştırılan kodlar, base64 ile şifrelenmiş komutlar veya tamamen bellekte çalışan "dosyasız" (fileless) saldırılar gibi geleneksel disk tabanlı taramalardan kaçan tehditler, AMSI sayesinde güvenlik yazılımı tarafından analiz edilebilir.
- **Savunmacı İçin Önemi:** Modern bir EDR çözümü ile birlikte AMSI, PowerShell tabanlı zararlı yazılımların büyük bir kısmını proaktif olarak engelleme potansiyeline sahiptir.

## Constrained Language Mode (Kısıtlı Dil Modu)

Bu, PowerShell'in yeteneklerini ciddi şekilde kısıtlayan çok güvenli bir çalışma modudur. Bu modda, kullanıcılar sadece temel cmdlet'leri çalıştırabilir.
- **Ne Engellenir?** `.NET` sınıflarına erişim, `Add-Type` kullanımı, COM nesneleri ve Windows API'lerine erişim gibi bir saldırganın sistemi manipüle etmek için kullanacağı neredeyse tüm gelişmiş özellikler engellenir.
- **Ne Zaman Kullanılır?** AppLocker veya Device Guard gibi uygulama beyaz listeleme (whitelisting) çözümleriyle birlikte kullanıldığında, PowerShell'i sadece yöneticinin izin verdiği script'leri çalıştırabilen, son derece güvenli bir kabuğa dönüştürür.

Bu güvenlik ve loglama mekanizmaları doğru bir şekilde yapılandırıldığında, PowerShell, bir saldırı yüzeyi olmaktan çıkıp, kurum ağındaki en iyi dostunuz olan bir "kamera sistemine" dönüşür.

---
[➡️ Sonraki Konu: Olay Günlükleri ile Tehidit Avcılığı](./02-Olay-Gunlukleri-ile-Tehdit-Avciligi.md)
