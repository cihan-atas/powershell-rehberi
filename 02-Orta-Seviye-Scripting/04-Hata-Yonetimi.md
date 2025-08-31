# 2.4. Hata Yönetimi

[↩️ Geri: İçindekiler](../README.md)

İyi yazılmış bir script, sadece doğru çalıştığında değil, aynı zamanda beklenmedik hatalarla karşılaştığında da öngörülebilir ve kontrollü bir şekilde davrandığında kendini belli eder. PowerShell, hataları yakalamak ve yönetmek için güçlü mekanizmalar sunar.

## Hata Türleri

PowerShell'de temel olarak iki tür hata vardır:

1.  **Terminating Errors (Sonlandırıcı Hatalar):** Bu hatalar o kadar ciddidir ki, komutun veya script'in çalışmasını tamamen durdurur. Örneğin, var olmayan bir cmdlet'i çağırmak. Bu hatalar, `Try/Catch` blokları tarafından varsayılan olarak yakalanır.

2.  **Non-Terminating Errors (Sonlandırıcı Olmayan Hatalar):** Bu hatalar bir sorun olduğunu bildirir ancak script'in çalışmasını durdurmaz. Örneğin, var olmayan bir dosyayı `Get-ChildItem` ile aramak. Komut bir hata mesajı verir ama pipeline'daki bir sonraki öğeyi işlemeye devam eder. `Try/Catch` blokları bu hataları varsayılan olarak **yakalamaz**.

## `-ErrorAction` Ortak Parametresi

Sonlandırıcı olmayan bir hatanın davranışını değiştirmek için `-ErrorAction` ortak parametresini kullanırız. Bu parametre, neredeyse tüm cmdlet'lerde mevcuttur.

-   **`SilentlyContinue`:** Hata mesajını gösterme ve çalışmaya devam et.
    ```powershell
    # Var olmayan dosyayı ararken hata mesajı gösterme
    Get-ChildItem -Path "C:\VarOlmayanDosya.txt" -ErrorAction SilentlyContinue
    ```
-   **`Continue`:** (Varsayılan) Hata mesajını göster ve çalışmaya devam et.
-   **`Stop`:** Sonlandırıcı olmayan bir hatayı, **sonlandırıcı bir hataya dönüştür**. Bu, hatanın `Try/Catch` bloğu tarafından yakalanabilmesi için **kritik öneme sahiptir**.
    ```powershell
    # Bu komut hata verirse, script'i durduracak ve catch bloğuna atlayacak
    Get-ChildItem -Path "C:\VarOlmayanDosya.txt" -ErrorAction Stop
    ```
-   **`Ignore`:** Hatanın kendisini ve hata akışına yazılmasını engeller. `SilentlyContinue`'den daha güçlüdür. Nadiren kullanılır.

## Hataları Yakalama: `Try / Catch / Finally`

`Try/Catch/Finally` bloğu, diğer programlama dillerinden aşina olabileceğiniz, yapılandırılmış hata yönetimi yöntemidir.

-   **`Try` Bloğu:** Hata oluşturma potansiyeli olan kod buraya yazılır.
-   **`Catch` Bloğu:** `Try` bloğunda **sonlandırıcı bir hata** meydana gelirse, kontrol bu bloğa geçer.
-   **`Finally` Bloğu:** Bu bloktaki kod, `Try` bloğunda hata olsun ya da olmasın **her zaman** çalıştırılır. Genellikle kaynakları temizlemek (örneğin, açık dosyaları kapatmak) için kullanılır.

**Sözdizimi:**
```powershell
try {
    # Hata verebilecek kodlar
}
catch {
    # Hata durumunda çalışacak kodlar
}
finally {
    # Her durumda çalışacak kodlar
}
```

**Pratik Örnek:**
Bir sunucuya bağlanmayı deneyen ve başarısız olursa anlamlı bir mesaj veren bir script yazalım.
```powershell
$computerName = "VarOlmayanSunucu"

try {
    Write-Host "Trying to connect to $computerName..."
    # Test-Connection normalde sonlandırıcı hata vermez, bu yüzden -ErrorAction Stop kullanıyoruz.
    Test-Connection -ComputerName $computerName -Count 1 -ErrorAction Stop
    Write-Host "Successfully connected to $computerName." -ForegroundColor Green
}
catch {
    # Hata yakalandığında bu blok çalışır.
    Write-Warning "Failed to connect to $computerName."

    # Hatanın kendisi hakkında bilgi almak için `$_` özel değişkeni kullanılır.
    Write-Host "Error Details: $($_.Exception.Message)" -ForegroundColor Red
}
finally {
    # Bu mesaj her zaman gösterilir.
    Write-Host "Connection attempt finished."
}
```

## `$Error` Değişkeni

PowerShell, oturum boyunca oluşan tüm hataları `$Error` adlı özel bir değişkende saklar. Bu değişken, bir dizi (Array) gibi davranır.
```powershell
# En son oluşan hatayı görmek için
$Error[0]

# Hata detaylarını görmek için
$Error[0].Exception.Message

# Hata geçmişini temizlemek için
$Error.Clear()
```
Bu değişken, `Try/Catch` kullanmadığınız durumlarda veya bir script'in neden başarısız olduğunu anlamak için interaktif olarak hata ayıklarken çok kullanışlıdır.

---
[➡️ Sonraki Konu: Modüller ve PowerShell Gallery](./05-Moduller-ve-PowerShell-Gallery.md)
