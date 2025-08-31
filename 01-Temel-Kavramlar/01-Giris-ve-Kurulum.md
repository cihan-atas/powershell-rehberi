# 1.1. PowerShell'e Giriş ve Kurulum

[↩️ Geri: İçindekiler](../README.md)

PowerShell dünyasına hoş geldiniz! Bu bölümde, PowerShell'in ne olduğunu, geleneksel komut satırlarından farkını, versiyonlarını ve çalışma ortamınızı nasıl hazırlayacağınızı öğreneceksiniz.

## PowerShell Nedir?

PowerShell, sadece bir komut satırı kabuğu (shell) değildir. Microsoft tarafından geliştirilmiş, .NET platformu üzerine inşa edilmiş, platformlar arası (Windows, Linux, macOS) çalışabilen güçlü bir **otomasyon ve yapılandırma yönetimi aracıdır**.

### PowerShell vs. CMD / Bash

| Özellik | Geleneksel Shell (CMD, Bash) | PowerShell |
| :--- | :--- | :--- |
| **Veri Tipi** | Metin (String) | **Nesne (Object)** |
| **İşlem** | Metni işlemek için `grep`, `awk` gibi araçlar gerekir. | Nesnelerin özelliklerine (`.Property`) ve metotlarına (`.Method()`) doğrudan erişilir. |
| **Yapı** | Komutlar ve parametreler standart değildir. | Tüm komutlar (`Cmdlet`) `Fiil-İsim` yapısındadır, bu da tahmin edilebilirliği artırır. |
| **Platform** | Genellikle tek bir platforma odaklıdır. | Windows, Linux ve macOS'ta tutarlı bir deneyim sunar. |

PowerShell'in en büyük gücü, komutlar arasında metin yerine **nesne** taşımasıdır. Bu konuya [Pipeline ve Nesneler](./03-Pipeline-ve-Nesneler.md) bölümünde derinlemesine dalacağız.

## PowerShell Sürümleri

İki ana PowerShell sürümü bulunmaktadır ve ikisi de sisteminizde aynı anda bulunabilir:

1.  **Windows PowerShell (Sürüm 5.1'e kadar):**
    - Windows işletim sistemi ile bütünleşik gelir.
    - Sadece Windows üzerinde çalışır ve .NET Framework kullanır.
    - Artık yeni özellikler eklenmemektedir, sadece güvenlik güncellemeleri alır.
    - **Konumu:** `C:\Windows\System2\WindowsPowerShell\v1.0\powershell.exe`

2.  **PowerShell (Sürüm 7+):**
    - Modern, açık kaynaklı ve platformlar arasıdır.
    - Windows, Linux ve macOS üzerinde çalışır.
    - .NET (eskiden .NET Core) platformunu kullanır ve sürekli olarak yeni özellikler kazanır.
    - **Bu rehberdeki tüm modern uygulamalar için kullanılması şiddetle tavsiye edilir.**
    - **Kurulum:** [Microsoft'un resmi GitHub sayfasından](https://github.com/PowerShell/PowerShell/releases) işletim sisteminize uygun sürümü indirebilirsiniz. Windows kullanıcıları için en kolay kurulum yöntemi `winget` paket yöneticisidir:
      ```powershell
      winget install --id Microsoft.PowerShell --source winget
      ```

### Sürümünüzü Nasıl Kontrol Edersiniz?

Kullandığınız PowerShell sürümünü ve detaylarını görmek için en güvenilir komut `$PSVersionTable` özel değişkenidir:

```powershell
$PSVersionTable
```

**Örnek Çıktı:**
```
Name                           Value
----                           -----
PSVersion                      7.2.1
PSEdition                      Core
GitCommitId                    7.2.1
OS                             Microsoft Windows 10.0.19044
Platform                       Win32NT
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0…}
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
WSManStackVersion              3.0
```
Bu çıktıda `PSVersion` ve `PSEdition` (`Core` modern sürüm, `Desktop` ise Windows PowerShell anlamına gelir) en önemli bilgilerdir.

## Çalışma Ortamları

PowerShell komutlarını yazmak ve çalıştırmak için birkaç seçeneğiniz vardır:

-   **PowerShell Konsolu (`pwsh.exe` veya `powershell.exe`):**
    Hızlı komutlar ve basit otomasyonlar için ideal, klasik komut satırı arayüzüdür.

-   **Windows PowerShell ISE (Integrated Scripting Environment):**
    Windows PowerShell ile birlikte gelen, basit bir grafik arayüze sahip script editörüdür. Kod renklendirme ve basit hata ayıklama özellikleri sunar ancak artık geliştirilmemektedir.

-   **Visual Studio Code (VS Code) - Önerilen Ortam:**
    Günümüzün endüstri standardıdır. Microsoft tarafından geliştirilen resmi [PowerShell eklentisi](https://marketplace.visualstudio.com/items?itemName=ms-vscode.PowerShell) ile birlikte profesyonel bir geliştirme ortamı sunar:
    -   **IntelliSense:** Gelişmiş kod tamamlama ve parametre ipuçları.
    -   **Hata Ayıklama (Debugging):** Script'lerinizi satır satır çalıştırıp hataları kolayca bulma.
    -   **Terminal Entegrasyonu:** Editör içinden doğrudan kod çalıştırma.
    -   **Git Entegrasyonu:** Kodlarınızı versiyon kontrol sistemleriyle yönetme.

## Execution Policy (Yürütme Politikası) Nedir?

PowerShell, güvenlik amacıyla script dosyalarının (`.ps1`) varsayılan olarak çalıştırılmasını engelleyen bir mekanizmaya sahiptir. Bu politikayı görmek için:
```powershell
Get-ExecutionPolicy
```
- **Restricted:** (Varsayılan) Hiçbir script çalıştırılamaz.
- **RemoteSigned:** Yerel olarak oluşturduğunuz script'ler çalışır, ancak internetten indirilen script'lerin güvenilir bir yayıncı tarafından imzalanmış olması gerekir.
- **Unrestricted:** Tüm script'ler çalışır. (Güvenli olmayan ortamlarda risklidir).

Öğrenme ve geliştirme ortamları için en yaygın ve makul ayar `RemoteSigned`'dır. Bu politikayı sadece mevcut kullanıcı için ayarlamak güvenli bir başlangıçtır:
```powershell
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
```
Bu komut, bir onay isteyecektir. `Y` (Yes) veya `A` (Yes to All) ile onaylayabilirsiniz.

---
[➡️ Sonraki Konu: Cmdlet, Syntax ve Yardım Sistemi](./02-Cmdlet-Syntax-ve-Yardim.md)
