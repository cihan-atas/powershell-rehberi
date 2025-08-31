# 1.4. Değişkenler ve Veri Tipleri

[↩️ Geri: İçindekiler](../README.md)

Script yazmanın temel taşı, verileri geçici olarak saklayıp daha sonra kullanmaktır. PowerShell'de bu işlem **değişkenler** aracılığıyla yapılır. Bu bölümde, değişkenlerin nasıl oluşturulduğunu ve PowerShell'in temel veri tiplerini öğreneceksiniz.

## Değişkenler Nedir ve Nasıl Kullanılır?

Değişkenler, değerleri (metin, sayı, tarih, nesne vb.) saklamak için kullanılan isimlendirilmiş bellek alanlarıdır. PowerShell'de tüm değişkenler dolar işareti (`$`) ile başlar.

Bir değişkene değer atamak için eşittir (`=`) operatörü kullanılır.

```powershell
# Bir metin (string) değişkeni
$kullaniciAdi = "zeus"

# Bir sayı (integer) değişkeni
$yas = 30

# Bir komutun çıktısını değişkene atama
$calisanServisler = Get-Service | Where-Object { $_.Status -eq 'Running' }
```
Değişkenin değerini görmek için sadece adını yazmanız yeterlidir:
```powershell
$kullaniciAdi
# Çıktı: zeus

# Bir metin içinde değişken kullanma (çift tırnak ile)
Write-Host "Kullanıcının adı: $kullaniciAdi"
# Çıktı: Kullanıcının adı: zeus
```

### Tek Tırnak (`'`) vs. Çift Tırnak (`"`)

Bu, PowerShell'de sıkça karıştırılan önemli bir konudur:
-   **Çift Tırnak (`"`):** İçindeki değişkenleri ve özel ifadeleri **yorumlar** (genişletir).
-   **Tek Tırnak (`'`):** İçindeki her şeyi **olduğu gibi** (literals) kabul eder, hiçbir şeyi yorumlamaz.

```powershell
$kullanici = "whoami"

Write-Host "Çift tırnak: Merhaba, $kullanici"
# Çıktı: Çift tırnak: Merhaba, whoami

Write-Host 'Tek tırnak: Merhaba, $kullanici'
# Çıktı: Tek tırnak: Merhaba, $kullanici
```

## Temel Veri Tipleri

PowerShell, kullandığınız verinin türünü genellikle otomatik olarak anlar. Ancak temel veri tiplerini bilmek önemlidir.

-   **String:** Metin verileri. Tırnak işaretleri içinde belirtilir.
    ```powershell
    $mesaj = "Bu bir metindir."
    ```
-   **Integer (`[int]`):** Tam sayılar.
    ```powershell
    $sayi = 100
    ```
-   **Boolean (`[bool]`):** Sadece iki değeri olabilen mantıksal veri tipi: `$true` veya `$false`.
    ```powershell
    $aktifMi = $true
    ```
-   **DateTime:** Tarih ve saat bilgisi tutar. `Get-Date` komutu bu türde bir nesne döndürür.
    ```powershell
    $simdi = Get-Date
    ```

Bir değişkenin veri tipini öğrenmek için `.GetType()` metodunu kullanabilirsiniz:
```powershell
$sayi = 123
$sayi.GetType().Name
# Çıktı: Int32
```

## Koleksiyonlar: Diziler ve Hashtable'lar

Bazen tek bir değişkende birden çok değeri bir arada tutmak gerekir. Bunun için koleksiyonlar kullanılır.

### Dizi (Array)

Virgülle ayrılmış bir değerler listesidir. Bir dizi oluşturmak için değerleri virgülle ayırmanız yeterlidir.
```powershell
$sunucular = "WEB01", "SQL01", "DC01"
```
veya `@()` operatörünü kullanabilirsiniz:
```powershell
$kullanicilar = @("ahmet", "mehmet", "ayse")
```

Dizinin elemanlarına indeks numarasıyla (0'dan başlar) erişilir:
```powershell
# İlk sunucuyu getir
$sunucular[0]
# Çıktı: WEB01

# Son sunucuyu getir
$sunucular[-1]
# Çıktı: DC01
```

### Hashtable (Sözlük)

Anahtar-değer (key-value) çiftlerinden oluşan bir koleksiyondur. Özellikle yapılandırılmış verileri saklamak için çok kullanışlıdır. `@`{}` ile oluşturulur.
```powershell
$kullaniciBilgisi = @{
    KullaniciAdi = "sdemir"
    UserID       = 1024
    Departman    = "IT"
    AktifMi      = $true
}
```
Hashtable'daki bir değere anahtar adıyla erişilir:
```powershell
# Departman bilgisini al
$kullaniciBilgisi.Departman
# Çıktı: IT

# Veya
$kullaniciBilgisi["KullaniciAdi"]
# Çıktı: sdemir
```

## Özel Değişkenler

PowerShell'in kendi içinde önceden tanımlanmış, özel anlamları olan bazı değişkenler bulunur.
-   `$_` veya `$PSItem`: Pipeline'daki mevcut nesneyi temsil eder.
-   `$null`: Hiçbir değeri temsil etmez (boşluk veya sıfır değildir).
-   `$true` / `$false`: Mantıksal doğru ve yanlış değerleri.
-   `$Error`: En son oluşan hataları içeren bir dizi.

---
[➡️ Sonraki Bölüm: Orta Seviye Scripting](./../02-Orta-Seviye-Scripting/01-Akis-Kontrolu-Donguler-ve-Kosullar.md)
