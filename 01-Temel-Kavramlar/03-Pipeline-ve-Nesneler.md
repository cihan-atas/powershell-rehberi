# 1.3. Pipeline ve Nesneler: PowerShell'in Kalbi

[↩️ Geri: İçindekiler](../README.md)

PowerShell'i diğer komut satırı arayüzlerinden (CMD, Bash) ayıran en temel ve en güçlü özellik, metin tabanlı değil, **nesne (object)** tabanlı çalışmasıdır. Bu nesneler, komutlar arasında **Pipeline (`|`)** adı verilen bir boru hattı ile taşınır.

## Pipeline (`|`) Nedir?

Pipeline operatörü (`|`), bir komutun çıktısını bir sonraki komuta girdi olarak gönderir.

-   **Geleneksel Shell'lerde (Bash):** Komutlar arasında metin (string) akar. Bir komutun çıktısını işlemek için `awk`, `sed`, `grep` gibi araçlarla metin üzerinde karmaşık işlemler yapmanız gerekir.
-   **PowerShell'de:** Komutlar arasında zengin, yapılandırılmış **nesneler** akar. Her nesne, verileri (Properties) ve eylemleri (Methods) içinde barındırır.

Bu sayede veriyi ayrıştırmakla (parsing) uğraşmak yerine, doğrudan nesnenin özelliklerine erişebilirsiniz.

### Pratik Bir Örnek

Sistemdeki en çok bellek (WorkingSet) kullanan 5 işlemi bulalım.

**Geleneksel Yöntem (Düşünce):** `tasklist` komutunu çalıştır, çıktıyı metin olarak al, başlık satırlarını atla, bellek sütununa göre sırala, ilk 5 satırı al. Bu çok zahmetli olurdu.

**PowerShell Yöntemi:**
```powershell
Get-Process | Sort-Object -Property WorkingSet -Descending | Select-Object -First 5
```

Bu komutun adımları:
1.  **`Get-Process`**: Sistemdeki tüm işlemleri birer **Process Nesnesi** olarak üretir. Her nesne `Name`, `Id`, `CPU`, `WorkingSet` gibi onlarca özelliğe sahiptir.
2.  **`|`**: Bu nesne koleksiyonunu `Sort-Object` komutuna gönderir.
3.  **`Sort-Object -Property WorkingSet -Descending`**: Gelen her bir nesnenin `WorkingSet` özelliğine bakar ve koleksiyonu büyükten küçüğe doğru sıralar.
4.  **`|`**: Sıralanmış nesne koleksiyonunu `Select-Object` komutuna gönderir.
5.  **`Select-Object -First 5`**: Gelen koleksiyonundaki ilk 5 nesneyi seçer ve ekrana yazdırır.

Sonuç, filtrelenmiş ve sıralanmış, tam teşekküllü nesnelerdir.

### Çıktıyı Formatlama

PowerShell, nesneleri ekrana farklı şekillerde yazdırmak için formatlama cmdlet'leri sunar. Bu cmdlet'ler genellikle pipeline'ın **en sonunda** kullanılır.

-   **`Format-Table` (`ft`):** Çıktıyı sütunlar halinde bir tablo olarak gösterir. (Varsayılan format budur).
    ```powershell
    Get-Process | Sort-Object WS -Descending | Select-Object -First 5 | Format-Table Name, Id, WS
    ```
-   **`Format-List` (`fl`):** Her nesnenin özelliklerini alt alta bir liste olarak gösterir. Çok fazla özelliği olan nesneleri incelemek için idealdir.
    ```powershell
    # Güvenlik olay günlüğündeki son olayın tüm detaylarını gör
    Get-EventLog -LogName Security -Newest 1 | Format-List *
    ```
    Buradaki `*` (yıldız), nesnenin **tüm** özelliklerini göstermesini sağlar.

## Nesneleri Keşfetme: `Get-Member`

Bir komutun ne tür bir nesne ürettiğini ve bu nesnenin hangi özelliklere (Properties) ve metotlara (Methods) sahip olduğunu görmek için `Get-Member` (`gm`) cmdlet'i kullanılır. Bu, PowerShell'i öğrenirken en çok kullanacağınız araçlardan biridir.

```powershell
# Get-Service komutunun ürettiği nesnenin yapısını inceleyelim
Get-Service -Name "Spooler" | Get-Member
```

**Çıktıdan Bazı Örnekler:**
```
   TypeName: System.ServiceProcess.ServiceController

Name                      MemberType     Definition
----                      ----------     ----------
Name                      AliasProperty  Name = ServiceName
Stop                      Method         void Stop()
Start                     Method         void Start(string[] args)
Status                    Property       System.ServiceProcess.ServiceControllerStatus Status {get;}
DisplayName               Property       string DisplayName {get;set;}
ServiceName               Property       string ServiceName {get;set;}
```

Bu çıktı bize şunu söyler:
-   **TypeName:** Bu nesnenin .NET'teki tam adı `System.ServiceProcess.ServiceController`'dır.
-   **Properties (Özellikler):** Nesne hakkındaki verilerdir (`Status`, `DisplayName`, `ServiceName`). Bunlar, `Where-Object` ile filtreleme veya `Sort-Object` ile sıralama yaparken kullandığımız alanlardır.
-   **Methods (Metotlar):** Nesne ile yapabileceğiniz eylemlerdir (`Start()`, `Stop()`).

Bu bilgiyi kullanarak bir servisi doğrudan durdurabiliriz:
```powershell
# Önce nesneyi bir değişkene atayalım
$spooler = Get-Service -Name "Spooler"

# Şimdi nesnenin Stop() metodunu çağıralım
$spooler.Stop()
```
Bu, `Stop-Service -Name "Spooler"` komutuyla aynı işi yapar ancak PowerShell'in nesne yönelimli doğasını gösteren harika bir örnektir.

---
[➡️ Sonraki Konu: Değişkenler ve Veri Tipleri](./04-Degiskenler-ve-Veri-Tipleri.md)
