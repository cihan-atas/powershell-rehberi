# Ek B: En İyi Pratikler ve Stil Rehberi

[↩️ Geri: İçindekiler](./README.md)

Çalışan bir kod yazmak işin sadece yarısıdır. Profesyonel PowerShell script'leri, başkaları tarafından okunabilir, kolayca yönetilebilir ve sürdürülebilir olmalıdır. Bu bölümde, kodunuzun kalitesini artıracak bazı en iyi pratikler ve stil kuralları listelenmiştir.

## 1. İsimlendirme Standartları

-   **Fiil-İsim Standardına Uyun:** Kendi fonksiyonlarınızı yazarken, PowerShell'in yerleşik `Fiil-İsim` standardını takip edin. Bu, kodunuzu daha sezgisel ve tutarlı hale getirir. Onaylanmış fiillerin listesi için `Get-Verb` komutunu kullanın.
    ```powershell
    # Kötü:
    function ProcessData {}
    function CheckServer {}

    # İyi:
    function Invoke-DataProcessing {}
    function Test-ServerConnection {}
    ```

-   **Açıklayıcı Değişken Adları Kullanın:** Değişkenlerinize, neyi sakladıklarını açıkça belirten isimler verin. Bu, kodunuzu aylar sonra okuduğunuzda ne yaptığını anlamanıza yardımcı olur.
    ```powershell
    # Kötü:
    $x = Get-Process
    $i = 10

    # İyi:
    $runningProcesses = Get-Process
    $maxRetryCount = 10
    ```

-   **Parametre İsimlerini Tutarlı Tutun:** Standart cmdlet'lerde kullanılan parametre isimlerini (örn: `-ComputerName`, `-Path`, `-Credential`) kendi fonksiyonlarınızda da kullanmaya çalışın. Bu, fonksiyonunuzu kullanan kişilerin alışkın olduğu bir deneyim sunar.

## 2. Okunabilirlik ve Formatlama

-   **Yorum Satırları Ekleyin:** Kodunuzun "neden" belirli bir şekilde çalıştığını açıklamak için `#` ile başlayan yorum satırları kullanın. Karmaşık mantıkları veya önemli kararları açıklamak için özellikle faydalıdır.
    ```powershell
    # Sunucuya bağlanmadan önce portun açık olduğundan emin oluyoruz,
    # çünkü Test-Connection zaman aşımına uğrayabilir.
    if (Test-NetConnection -ComputerName $server -Port 3389) {
        # ...
    }
    ```

-   **Kodunuzu Hizalayın ve Girintileyin:** `if`, `foreach`, `switch` gibi blokların içindeki kodları girintili yazın. Bu, kodun mantıksal yapısını görsel olarak anlamayı çok kolaylaştırır. VS Code gibi modern editörler bunu sizin için otomatik olarak yapabilir.

-   **Satır Uzunluğunu Sınırlayın:** Uzun komut satırlarını, okunabilirliği artırmak için `|` (pipeline) veya `` ` `` (backtick - ters tırnak) karakterlerinden sonra bölerek birden çok satıra yayın. Pipeline'dan sonra bölmek daha çok tercih edilir.
    ```powershell
    # Tercih edilen yöntem
    Get-ChildItem -Path $logPath -Recurse |
        Where-Object { $_.LastWriteTime -lt (Get-Date).AddDays(-30) } |
        Remove-Item -Force -Verbose
    ```

## 3. Sağlamlık ve Hata Yönetimi

-   **Hata Yönetimini İhmal Etmeyin:** Kritik komutları `Try/Catch` blokları içine alarak script'inizin beklenmedik hatalarda çökmesini önleyin. Özellikle dosya/ağ işlemleri ve dış sistemlerle etkileşimler hata vermeye müsaittir.

-   **`-ErrorAction Stop` Kullanımını Bilin:** `Try/Catch` bloklarının sonlandırıcı olmayan hataları (non-terminating errors) yakalaması için, hata potansiyeli olan komutta `-ErrorAction Stop` kullanmanız gerektiğini unutmayın.

-   **Gereksiz Yere Alias Kullanmaktan Kaçının:** Script'lerde, interaktif kullanımın aksine, cmdlet'lerin tam adlarını (`Get-ChildItem` gibi) kullanmak, alias'larını (`gci` gibi) kullanmaktan daha iyidir. Bu, kodun okunabilirliğini ve anlaşılırlığını artırır. Alias'lar, komut satırında hız kazanmak için harikadır, ancak kalıcı script'lerde açıklık daha önemlidir.

## 4. Gelişmiş Fonksiyon Özelliklerini Kullanın

-   **`[CmdletBinding()]` Ekleyin:** Fonksiyonlarınızı, `-Verbose`, `-Debug` gibi ortak parametreleri destekleyen ve daha profesyonel hissettiren gelişmiş fonksiyonlara dönüştürmek için en başına `[CmdletBinding()]` ekleyin.

-   **Kullanıcıya Geri Bildirim Sağlayın:**
    -   `Write-Host`: Sadece ekrana renkli metin yazdırmak için kullanılır. Çıktısı pipeline'a gönderilmez.
    -   `Write-Output`: Fonksiyonunuzun asıl çıktısıdır. Bu, pipeline'daki bir sonraki komuta gönderilecek olan veridir.
    -   `Write-Verbose`: `-Verbose` parametresi kullanıldığında gösterilecek detaylı işlem adımlarını yazmak için kullanılır.
    -   `Write-Warning`: Potansiyel bir sorunu belirtmek için kullanılır.
    -   `Write-Error`: Sonlandırıcı olmayan bir hata oluşturmak için kullanılır.

-   **Onay İsteği Ekleyin (`SupportsShouldProcess`):** Sisteme zarar verebilecek veya kalıcı değişiklik yapacak (dosya silme, servis durdurma vb.) fonksiyonlarınıza `[CmdletBinding(SupportsShouldProcess=$true)]` ekleyin. Bu, fonksiyonunuza otomatik olarak `-WhatIf` ve `-Confirm` parametrelerini kazandırır.
    ```powershell
    function Remove-OldLogFiles {
        [CmdletBinding(SupportsShouldProcess=$true)]
        param($Path)

        if ($pscmdlet.ShouldProcess($Path, "Eski log dosyaları silinecek")) {
            Remove-Item -Path $Path
        }
    }
    ```

---
[➡️ Sonraki Konu: Kaynaklar ve Topluluk](./EK-C-Kaynaklar-ve-Topluluk.md)
