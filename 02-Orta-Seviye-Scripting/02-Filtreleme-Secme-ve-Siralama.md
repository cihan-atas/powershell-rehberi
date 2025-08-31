# 2.2. Filtreleme, Seçme ve Sıralama

[↩️ Geri: İçindekiler](../README.md)

PowerShell'in gerçek gücü, komutların ürettiği yüzlerce veya binlerce nesne arasından tam olarak ihtiyacınız olan bilgiyi hassas bir şekilde ayıklayabilmekten gelir. Bu işlem için pipeline üzerinde çalışan üç temel cmdlet bulunur: `Where-Object`, `Select-Object` ve `Sort-Object`. Bu üçlü, PowerShell'deki en sık kullanılan komutlar arasındadır.

## Nesneleri Filtreleme: `Where-Object`

`Where-Object` (alias: `where` veya `?`), pipeline'dan gelen nesneleri belirli bir koşula göre filtreler. Sadece koşulu sağlayan (`$true` döndüren) nesneler pipeline'da yollarına devam eder.

**Sözdizimi:**
```powershell
Komut | Where-Object { $_.Özellik -Operatör Değer }
```
`$_` (veya `$PSItem`), pipeline'daki o anki nesneyi temsil eder.

**Örnekler:**

1.  **Durmuş olan tüm servisleri bulma:**
    ```powershell
    Get-Service | Where-Object { $_.Status -eq 'Stopped' }
    ```

2.  **500 MB'dan fazla bellek kullanan işlemleri bulma:**
    PowerShell, `KB`, `MB`, `GB` gibi sonekleri anlar. `WorkingSet` özelliği byte cinsinden olduğu için karşılaştırmayı kolaylaştırır.
    ```powershell
    Get-Process | Where-Object { $_.WorkingSet -gt 500MB }
    ```

3.  **Birden çok koşulla filtreleme (Mantıksal Operatörler):**
    `-and` ve `-or` operatörleri ile karmaşık filtreler oluşturabilirsiniz.
    ```powershell
    # 'svchost' ile başlayan VE 100 MB'dan fazla bellek kullanan işlemleri bul
    Get-Process | Where-Object { ($_.Name -like 'svchost*') -and ($_.WS -gt 100MB) }
    ```

## Nesne Özelliklerini Seçme: `Select-Object`

`Select-Object` (alias: `select`), bir nesnenin tüm özellikleri yerine sadece istediğiniz belirli özellikleri seçmenizi sağlar. Ayrıca, nesne koleksiyonunun başından veya sonundan belirli sayıda nesne almak için de kullanılır.

**Örnekler:**

1.  **İşlemlerin sadece adını ve ID'sini seçme:**
    ```powershell
    Get-Process | Select-Object -Property Name, Id
    ```

2.  **İlk 5 veya son 5 öğeyi seçme:**
    ```powershell
    # En yeni 10 olay günlüğü kaydını al
    Get-EventLog -LogName System | Select-Object -First 10

    # Alfabeye göre son 5 servisi al
    Get-Service | Sort-Object Name | Select-Object -Last 5
    ```

3.  **Hesaplanmış Özellikler (Calculated Properties):**
    `Select-Object`'in en güçlü özelliklerinden biri, var olan özelliklerden yeni özellikler türetmenize olanak tanımasıdır. Bu, bir hashtable kullanılarak yapılır.
    ```powershell
    # İşlemlerin bellek kullanımını MB cinsinden gösteren yeni bir özellik ekle
    Get-Process | Select-Object Name, Id, @{Name="Memory(MB)"; Expression={($_.WorkingSet / 1MB) -as [int]}}
    ```
    - **Name (`N`):** Yeni oluşturulan sütunun başlığı.
    - **Expression (`E`):** Bu sütunun değerini hesaplayan kod bloğu.

4.  **Yinelenenleri Eleme (Unique):**
    Bir listedeki benzersiz değerleri bulmak için kullanılır.
    ```powershell
    # Sistemdeki işlemlerin kullandığı benzersiz işlem adlarını listele
    Get-Process | Select-Object -Property Name -Unique
    ```

## Nesneleri Sıralama: `Sort-Object`

`Sort-Object` (alias: `sort`), nesne koleksiyonunu bir veya daha fazla özelliğe göre sıralar.

**Örnekler:**

1.  **İşlemleri alfabetik olarak ada göre sıralama:**
    Varsayılan sıralama A'dan Z'ye (artan) şekildedir.
    ```powershell
    Get-Process | Sort-Object -Property Name
    ```

2.  **Servisleri görüntü adına göre tersten (Z'den A'ya) sıralama:**
    `-Descending` switch parametresi sıralamayı tersine çevirir.
    ```powershell
    Get-Service | Sort-Object -Property DisplayName -Descending
    ```

3.  **Birden çok özelliğe göre sıralama:**
    Önce ilk özelliğe göre sıralar, eğer ilk özellik aynıysa ikinci özelliğe göre sıralar.
    ```powershell
    # İşlemleri önce isme, sonra ID'ye göre sırala
    Get-Process | Sort-Object -Property Name, Id
    ```

## Hepsini Bir Araya Getirme

Bu üç cmdlet'in gücü, birlikte kullanıldıklarında ortaya çıkar.

**Senaryo:** "svchost" ile başlayan işlemler arasından, CPU kullanımı en yüksek olan ilk 3 tanesini bulup, adını, işlem ID'sini ve CPU kullanımını gösterelim.

```powershell
Get-Process |                                        # 1. Tüm işlemleri al
Where-Object { $_.Name -like 'svchost*' } |          # 2. Sadece 'svchost' ile başlayanları filtrele
Sort-Object -Property CPU -Descending |              # 3. CPU kullanımına göre tersten sırala
Select-Object -First 3 |                             # 4. Sadece ilk 3'ünü al
Format-Table -Property Name, Id, CPU -AutoSize       # 5. İstenen özellikleri tablo olarak formatla
```

---
[➡️ Sonraki Konu: Fonksiyonlar ve Parametreler](./03-Fonksiyonlar-ve-Parametreler.md)
