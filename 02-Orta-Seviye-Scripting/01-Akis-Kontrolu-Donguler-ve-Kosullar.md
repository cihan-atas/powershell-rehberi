# 2.1. Akış Kontrolü: Döngüler ve Koşullar

[↩️ Geri: İçindekiler](../README.md)

Şimdiye kadar tekil komutları ve basit pipeline'ları öğrendik. Script'lerimizi daha akıllı ve dinamik hale getirmek için, belirli koşullara göre farklı eylemler gerçekleştirmemiz ve işlemleri tekrarlamamız gerekir. Bu bölümde, PowerShell'in akış kontrolü mekanizmalarını—koşullu ifadeler ve döngüler—ele alacağız.

## Koşullu İfadeler

### `If` / `ElseIf` / `Else`

`If` bloğu, bir koşul doğru (`$true`) olduğunda bir kod bloğunu çalıştırmanızı sağlar. `ElseIf` ile ek koşullar ekleyebilir ve `Else` ile hiçbir koşul karşılanmadığında çalışacak varsayılan bir eylem belirleyebilirsiniz.

**Sözdizimi:**
```powershell
if (koşul) {
    # Koşul doğruysa bu kod çalışır
} elseif (başka bir koşul) {
    # Önceki koşul yanlış ama bu doğruysa bu kod çalışır
} else {
    # Hiçbir koşul doğru değilse bu kod çalışır
}
```

**Örnek:** Bir servisin durumunu kontrol edip ona göre işlem yapalım.
```powershell
$serviceName = "Spooler"
$service = Get-Service -Name $serviceName -ErrorAction SilentlyContinue

if ($service.Status -eq 'Running') {
    Write-Host "$serviceName servisi zaten çalışıyor." -ForegroundColor Green
} else {
    Write-Host "$serviceName servisi çalışmıyor. Başlatılıyor..." -ForegroundColor Yellow
    Start-Service -Name $serviceName
    Write-Host "$serviceName servisi başlatıldı." -ForegroundColor Green
}
```

#### Karşılaştırma Operatörleri
PowerShell, C# veya Java gibi dillerdeki `==`, `!=`, `>` operatörleri yerine, `-` ile başlayan özel karşılaştırma operatörleri kullanır.

| Operatör | Anlamı | Örnek |
| :--- | :--- | :--- |
| `-eq` | Eşittir (Equals) | `$a -eq $b` |
| `-ne` | Eşit Değildir (Not Equals) | `$a -ne $b` |
| `-gt` | Büyüktür (Greater Than) | `$a -gt 10` |
| `-ge` | Büyük veya Eşittir (Greater or Equal) | `$a -ge 10` |
| `-lt` | Küçüktür (Less Than) | `$a -lt 10` |
| `-le` | Küçük veya Eşittir (Less or Equal) | `$a -le 10` |
| `-like` | Benzer (Joker karakter `*` kullanılır) | `$isim -like 'ah*'` |
| `-notlike` | Benzer Değil | `$isim -notlike 'ah*'` |
| `-contains`| Bir koleksiyonun elemanı içerip içermediğini kontrol eder | `$dizi -contains 'değer'` |
| `-in`| Bir değerin koleksiyon içinde olup olmadığını kontrol eder | `'değer' -in $dizi` |

### `Switch`
`Switch` ifadesi, çok sayıda `If`/`ElseIf` bloğuna daha temiz bir alternatiftir. Bir değişkenin değerini birden çok olası değerle karşılaştırır.

**Örnek:** Günün adına göre bir mesaj yazdıralım.
```powershell
switch ((Get-Date).DayOfWeek) {
    "Monday"    { Write-Host "Haftanın ilk iş günü!" }
    "Friday"    { Write-Host "Hafta sonu yaklaşıyor!" }
    "Saturday"  { Write-Host "İyi tatiller!" }
    "Sunday"    { Write-Host "İyi tatiller!" }
    default     { Write-Host "Normal bir iş günü." }
}
```

## Döngüler

Döngüler, bir kod bloğunu tekrar tekrar çalıştırmak için kullanılır.

### `ForEach-Object` Döngüsü (Pipeline için)

Pipeline'dan gelen her bir nesne için bir işlem yapmak amacıyla kullanılır. Genellikle alias'ı olan `%` ile kısaltılır.
```powershell
# Durmuş olan tüm servisleri bul ve başlat
Get-Service | Where-Object { $_.Status -eq 'Stopped' } | ForEach-Object {
    Write-Host "$($_.Name) servisi başlatılıyor..."
    Start-Service -InputObject $_
}
```
`$_` (veya `$PSItem`), pipeline'daki o anki nesneyi temsil eder.

### `foreach` Döngüsü (Genel kullanım)

Bir dizideki veya koleksiyondaki her bir eleman için bir kod bloğunu çalıştırır.
```powershell
$sunucular = @("WEB01", "SQL01", "DC01")

foreach ($sunucu in $sunucular) {
    Write-Host "--- $sunucu sunucusunun durumu kontrol ediliyor ---"
    Test-Connection -ComputerName $sunucu -Count 1 -Quiet
}
```

### `For` Döngüsü

Bir kod bloğunu belirli bir sayıda çalıştırmak için kullanılır. Genellikle bir sayaç değişkeni ile kontrol edilir.
```powershell
# 1'den 5'e kadar olan sayıları ekrana yazdır
for ($i = 1; $i -le 5; $i++) {
    Write-Host "Sayı: $i"
}
```

### `While` ve `Do-While`/`Do-Until` Döngüleri

- **`While`:** Bir koşul doğru olduğu sürece çalışır. Koşul en başta kontrol edilir.
    ```powershell
    $sayac = 0
    while ($sayac -lt 5) {
        Write-Host "Sayaç değeri: $sayac"
        $sayac++ # Sayacı bir artır
    }
    ```
- **`Do-While` / `Do-Until`:** Kod bloğunu en az bir kez çalıştırır, sonra koşulu kontrol eder.
    - `Do-While` koşul doğru olduğu sürece devam eder.
    - `Do-Until` koşul doğru olana kadar devam eder.

---
[➡️ Sonraki Konu: Filtreleme, Seçme ve Sıralama](./02-Filtreleme-Secme-ve-Siralama.md)
