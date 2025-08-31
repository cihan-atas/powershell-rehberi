# 1.2. Cmdlet, Syntax ve Yardım Sistemi

[↩️ Geri: İçindekiler](../README.md)

PowerShell'in gücünü ve kullanım kolaylığını anlamak için temel yapı taşlarını bilmek gerekir. Bu bölümde, komutların (Cmdlet) nasıl yapılandırıldığını, nasıl kullanılacağını ve en önemlisi, bilmediğiniz bir komutu nasıl öğrenebileceğinizi keşfedeceksiniz.

## Cmdlet Nedir? Fiil-İsim (Verb-Noun) Yapısı

PowerShell'deki komutlara **Cmdlet** (command-let) denir. Cmdlet'lerin en güzel özelliği, isimlerinin tahmin edilebilir bir standartta olmasıdır: **`Fiil-İsim` (Verb-Noun)**.

-   **Fiil (Verb):** Komutun yapacağı eylemi belirtir. `Get`, `Set`, `New`, `Add`, `Remove`, `Start`, `Stop` gibi.
-   **İsim (Noun):** Eylemin hangi nesne üzerinde yapılacağını belirtir. `Service`, `Process`, `Item` (Dosya/Klasör) gibi.

Bu yapı sayesinde, "Bir servisi durdurmak için hangi komutu kullanmalıyım?" diye düşündüğünüzde aklınıza hemen `Stop-Service` gelmelidir.

| Görev | Komut | Açıklama |
| :--- | :--- | :--- |
| Çalışan işlemleri listele | `Get-Process` | İşlemleri **al/getir**. |
| Yeni bir klasör oluştur | `New-Item` | Yeni bir **öğe** oluştur. |
| Bir servisi başlat | `Start-Service` | Bir servisi **başlat**. |

Sistemdeki onaylanmış fiillerin tam listesini görmek için `Get-Verb` komutunu kullanabilirsiniz.

## Komut Sözdizimi (Syntax) ve Parametreler

Bir cmdlet'in davranışını değiştirmek veya ona bilgi sağlamak için **parametreler** kullanılır. Parametreler daima tire (`-`) işareti ile başlar.

```powershell
# Genel yapı: Fiil-İsim -ParametreAdı Değer
Get-Service -Name "Spooler"
```
Bu örnekte:
- `Get-Service` cmdlet'tir.
- `-Name` bir parametredir.
- `"Spooler"` ise `-Name` parametresine atanan değerdir (argüman).

### Parametre Türleri

-   **Switch Parametresi:** Sadece varlığıyla bir durumu `true` yapar. Değer almaz.
    ```powershell
    # -Recurse bir switch parametresidir.
    Get-ChildItem -Path "C:\Windows" -Recurse
    ```
-   **Zorunlu Parametre:** Cmdlet'in çalışması için mutlaka girilmesi gereken parametre. Eğer girmezseniz, PowerShell size soracaktır.
    ```powershell
    # -Path parametresi zorunludur.
    Get-Item -Path "C:\Windows"
    ```

## En Önemli Yetenek: Yardım Sistemini Kullanmak

PowerShell'i öğrenmenin en hızlı yolu, kendi içindeki yardım sistemini etkin bir şekilde kullanmaktır.

### Komut Bulma: `Get-Command`

Sisteminizde hangi komutların olduğunu bulmak için kullanılır. Özellikle joker karakter (`*`) ile çok güçlü hale gelir.
```powershell
# İsminde "EventLog" geçen tüm komutları bul
Get-Command *EventLog*

# "Get" fiiline sahip tüm komutları listele
Get-Command -Verb Get
```

### Komutları Öğrenme: `Get-Help`

Bir komutun ne işe yaradığını, parametrelerini ve nasıl kullanılacağını öğrenmek için en iyi dostunuzdur. Linux'taki `man` komutunun karşılığıdır.
```powershell
# Bir komut hakkında temel yardım al
Get-Help Get-Process

# Pratik kullanım örneklerini göster (EN FAYDALISI!)
Get-Help Get-Process -Examples

# Tüm detayları (parametre açıklamaları dahil) göster
Get-Help Get-Process -Full

# Komutun en güncel yardım sayfasını web tarayıcısında aç
Get-Help Get-Process -Online
```

> **`Get-Command` vs. `Get-Help` Farkı Nedir?**
> -   **`Get-Command`:** "Hangi komutlar var?" sorusuna cevap verir. (Komutları **bulur**).
> -   **`Get-Help`:** "Bu komut nasıl kullanılır?" sorusuna cevap verir. (Komutu **öğretir**).

### Yardım Sistemini Güncel Tutma

Yardım dosyaları zamanla eskiyebilir. İnternetten en güncel yardım dosyalarını indirmek için PowerShell'i **yönetici olarak** çalıştırıp şu komutu girmeniz önemlidir:
```powershell
Update-Help
```
Bu işlemi ayda bir yapmak iyi bir alışkanlıktır.

### Kavramsal Yardım: `about_*` Dosyaları

PowerShell, sadece komutlar için değil, temel kavramlar (değişkenler, döngüler, operatörler vb.) için de yardım dosyaları içerir. Bunlara `about_*` denir.
```powershell
# Karşılaştırma operatörleri hakkında bilgi al
Get-Help about_Comparison_Operators

# Döngüler hakkında bilgi al
Get-Help about_Loops
```
Bu dosyalar, PowerShell'in teorisini anlamak için paha biçilmezdir.

---
[➡️ Sonraki Konu: Pipeline ve Nesneler: PowerShell'in Kalbi](./03-Pipeline-ve-Nesneler.md)
