# 3.5. Veri İşleme: CSV, JSON ve XML

[↩️ Geri: İçindekiler](../README.md)

PowerShell'in gücü sadece sistemleri yönetmekle kalmaz, aynı zamanda bu sistemlerden topladığı veriyi yapılandırılmış formatlarda okuma, işleme ve yazma yeteneğinde de yatar. Raporlar oluşturmak, konfigürasyonları yönetmek veya uygulamalar arasında veri alışverişi yapmak için CSV, JSON ve XML gibi standart formatlarla çalışmak kaçınılmazdır.

## CSV (Comma-Separated Values)

CSV, verileri tablo formatında saklamak için kullanılan en yaygın ve en basit metin tabanlı formattır. Her satır bir kaydı, her virgül ise sütunları temsil eder. Excel gibi tablo uygulamaları tarafından kolayca açılabilir.

### Veriyi CSV Olarak Dışa Aktarma: `Export-Csv`

Herhangi bir PowerShell komutunun nesne çıktısını doğrudan bir `.csv` dosyasına yazabilirsiniz.
```powershell
# Sistemdeki tüm servislerin adını, durumunu ve başlangıç türünü bir CSV dosyasına aktaralım
Get-Service | Select-Object Name, Status, StartType | Export-Csv -Path "C:\Temp\ServisRaporu.csv"
```
Bu komut, `ServisRaporu.csv` adında bir dosya oluşturur.

**Önemli Parametreler:**
-   **`-NoTypeInformation`:** Bu parametre, CSV dosyasının başına `#TYPE System.Management.Automation.PSCustomObject` gibi bir bilgi satırı eklenmesini **engeller**. Diğer programlarla uyumluluk için bu parametreyi **her zaman** kullanmak en iyi pratiktir.
    ```powershell
    Get-Process | Export-Csv -Path "C:\Temp\ProcessList.csv" -NoTypeInformation
    ```
-   **`-Encoding`:** Türkçe karakterler (`ç`, `ğ`, `ı`, `ş` vb.) içeren verilerle çalışırken Excel'in dosyayı doğru açması için encoding belirtmek önemlidir.
    ```powershell
    Export-Csv -Path "C:\Temp\Rapor.csv" -Encoding UTF8
    ```
-   **`-Append`:** Var olan bir CSV dosyasına yeni veriler ekler.

### CSV Dosyasından Veri Okuma: `Import-Csv`

`Import-Csv`, bir `.csv` dosyasını okur ve her satırı, sütun başlıklarını özellik olarak kullanan bir **PSCustomObject**'e dönüştürür. Bu, verilerle PowerShell içinde çalışmayı inanılmaz kolaylaştırır.
```powershell
# Daha önce oluşturduğumuz servis raporunu içeri aktaralım
$servisler = Import-Csv -Path "C:\Temp\ServisRaporu.csv"

# Artık bu bir nesne koleksiyonu. Durmuş olan servisleri filtreleyebiliriz.
$servisler | Where-Object { $_.Status -eq 'Stopped' }
```

## JSON (JavaScript Object Notation)

JSON, özellikle web API'leri ve modern konfigürasyon dosyaları arasında veri alışverişi için kullanılan, insana okunabilir, hafif bir veri formatıdır. Nested (iç içe) yapıları ve farklı veri tiplerini (diziler, sayılar, string'ler) destekler.

### Nesneyi JSON'a Dönüştürme: `ConvertTo-Json`
Bir PowerShell nesnesini (örneğin bir hashtable veya custom object) JSON formatında bir metin dizesine dönüştürür.
```powershell
# Bir kullanıcı bilgisi nesnesi oluşturalım
$userObject = [pscustomobject]@{
    UserID = 1024
    UserName = "zeus"
    Roles = @("Admin", "Auditor")
    LastLogin = Get-Date
}

# Bu nesneyi JSON formatına çevirelim
$jsonString = $userObject | ConvertTo-Json

# JSON çıktısını ekrana yazdır
$jsonString
```

### JSON'dan Nesneye Dönüştürme: `ConvertFrom-Json`
Bir JSON dizesini, özelliklerine kolayca erişebileceğiniz bir PowerShell nesnesine dönüştürür.
```powershell
# Az önce oluşturduğumuz JSON dizesini geri dönüştürelim
$newUserObject = $jsonString | ConvertFrom-Json

# Artık nesnenin özelliklerine erişebiliriz
Write-Host "Kullanıcı Adı: $($newUserObject.UserName)"
Write-Host "İlk Rol: $($newUserObject.Roles[0])"
```

## XML (eXtensible Markup Language)

XML, verileri hiyerarşik bir yapıda tanımlamak için kullanılan, etiket tabanlı bir formattır. JSON kadar popüler olmasa da, hala birçok kurumsal uygulama, konfigürasyon dosyası ve eski sistemlerde yaygın olarak kullanılır.

PowerShell'in XML ile çalışma şekli olağanüstüdür. Bir XML dosyasını veya dizesini `[xml]` türüne dönüştürdüğünüzde, onu noktalarla gezebileceğiniz bir nesneye çevirir.

```powershell
# Örnek bir XML verisi
$xmlString = @"
<books>
  <book id="bk101">
    <author>Gambardella, Matthew</author>
    <title>XML Developer's Guide</title>
    <price>44.95</price>
  </book>
  <book id="bk102">
    <author>Ralls, Kim</author>
    <title>Midnight Rain</title>
    <price>5.95</price>
  </book>
</books>
"@

# String'i bir XML nesnesine dönüştür (casting)
[xml]$xmlData = $xmlString

# Artık verilere nokta notasyonu ile erişebiliriz
# İlk kitabın yazarını alalım
$xmlData.books.book[0].author

# İkinci kitabın fiyatını alalım
$xmlData.books.book[1].price
```
Bir dosyadan XML okumak için önce `Get-Content` ile içeriği alıp sonra `[xml]` türüne dönüştürebilirsiniz:
```powershell
[xml]$configFile = Get-Content -Path "C:\App\config.xml"
$connectionString = $configFile.configuration.connectionStrings.add.connectionString
```

---
[➡️ Sonraki Bölüm: Mavi Takım Uygulamaları](../04-Mavi-Takim-Uygulamalari/01-Guvenlik-Loglama-ve-Auditing.md)
