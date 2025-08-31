# 2.3. Fonksiyonlar ve Parametreler

[↩️ Geri: İçindekiler](../README.md)

Script yazarken, belirli bir görevi yapan kod bloklarını sık sık tekrarladığınızı fark edeceksiniz. Kod tekrarı, hatalara ve yönetimi zor script'lere yol açar. **Fonksiyonlar**, bu tekrarlanan kod bloklarını isimlendirilmiş, yeniden kullanılabilir paketler haline getirmenizi sağlar.

## Basit Fonksiyonlar

En temel haliyle bir fonksiyon, `function` anahtar kelimesi, bir isim ve `{}` süslü parantezler içine yazılmış kod bloğundan oluşur.

**Sözdizimi:**
```powershell
function FonksiyonAdı {
    # Çalıştırılacak kodlar
}
```

**Örnek:** Mevcut dizindeki dosya ve klasör sayısını gösteren basit bir fonksiyon yazalım.
```powershell
function Get-ItemCount {
    $count = (Get-ChildItem).Count
    Write-Host "Bu dizinde toplam $count öğe var."
}
```
Bu fonksiyonu çalıştırmak için sadece adını yazmanız yeterlidir:
```powershell
Get-ItemCount
```
> **💡 İpucu:** Kendi fonksiyonlarınızı isimlendirirken, PowerShell'in `Fiil-İsim` standardına uymak iyi bir pratiktir. Bu, fonksiyonlarınızın diğer cmdlet'ler gibi hissettirmesini sağlar.

## Parametreler: Fonksiyonları Dinamik Hale Getirme

Fonksiyonların gerçek gücü, dışarıdan bilgi alabildiklerinde ortaya çıkar. Bu bilgi, **parametreler** aracılığıyla sağlanır. Parametreler, fonksiyonu daha esnek ve genel amaçlı hale getirir.

Parametreler, fonksiyonun başında bir `param()` bloğu içinde tanımlanır.

**Örnek:** Belirli bir yoldaki öğe sayısını sayan bir fonksiyon yazalım.
```powershell
function Get-ItemCount {
    param(
        $Path
    )

    $count = (Get-ChildItem -Path $Path).Count
    Write-Host "$Path dizininde toplam $count öğe var."
}
```
Artık fonksiyonu bir parametre ile çağırabiliriz:
```powershell
Get-ItemCount -Path "C:\Windows"
```

## Gelişmiş Fonksiyonlar (Advanced Functions)

PowerShell'de fonksiyonları, yerleşik cmdlet'ler gibi davranacak şekilde geliştirebilirsiniz. Bu, `[CmdletBinding()]` ve `[Parameter()]` öznitelikleri (attributes) kullanılarak yapılır. Gelişmiş fonksiyonlar size şu yetenekleri kazandırır:
- `-Verbose`, `-Debug` gibi ortak parametreleri otomatik olarak destekleme.
- Parametreleri zorunlu kılma.
- Parametreler için doğrulama (validation) ekleme.
- Pipeline'dan girdi kabul etme.

**Örnek:** `Get-ItemCount` fonksiyonumuzu gelişmiş bir fonksiyona dönüştürelim.
```powershell
function Get-ItemCount {
    [CmdletBinding()]
    param(
        # -Path parametresini zorunlu hale getiriyoruz.
        [Parameter(Mandatory=$true)]
        [string]$Path
    )

    # -Verbose parametresi kullanıldığında bu mesaj gösterilecek.
    Write-Verbose "Öğe sayısı hesaplanıyor: $Path"

    try {
        $items = Get-ChildItem -Path $Path -ErrorAction Stop
        $count = $items.Count
        Write-Host "$Path dizininde toplam $count öğe var."
    }
    catch {
        Write-Error "Hata: Yol bulunamadı veya erişilemedi. Detaylar: $($_.Exception.Message)"
    }
}
```

**Gelişmiş Fonksiyonun Kullanımı:**
```powershell
# Zorunlu parametre girilmediğinde PowerShell sorar
Get-ItemCount

# Verbose çıktısı ile çalıştırma
Get-ItemCount -Path "C:\Windows" -Verbose

# Hatalı bir yol verildiğinde hata yönetimi çalışır
Get-ItemCount -Path "C:\VarOlmayanKlasor"
```

### Parametre Doğrulama (Validation)

Kullanıcının girdiği parametrelerin geçerli olduğundan emin olmak için doğrulama öznitelikleri ekleyebilirsiniz.
```powershell
function Set-ComputerState {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$true)]
        [ValidateSet("Running", "Stopped", "Restarting")]
        [string]$State
    )
    
    Write-Host "Bilgisayar durumu '$State' olarak ayarlanıyor..."
}

# Kullanımı:
# Set-ComputerState -State Running # Geçerli
# Set-ComputerState -State Rebooting # Geçersiz, PowerShell hata verir ve seçenekleri sunar.
```

### Pipeline Girdisi Alma

Bir fonksiyonun pipeline'dan gelen veriyi işlemesini sağlamak için `ValueFromPipeline=$true` özniteliği kullanılır. Bu, fonksiyonunuzu `... | FonksiyonAdı` şeklinde kullanılabilir hale getirir.
```powershell
function Get-FileSizeInMB {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$true, ValueFromPipeline=$true)]
        $FileInfo
    )

    process {
        foreach ($file in $FileInfo) {
            $sizeInMB = ($file.Length / 1MB).ToString("F2") # Virgülden sonra 2 basamak
            Write-Host "$($file.Name) dosyasının boyutu: $sizeInMB MB"
        }
    }
}

# Kullanımı:
Get-ChildItem -Path "C:\Windows\System32" -File | Get-FileSizeInMB
```
`process` bloğu, pipeline'dan gelen her bir nesne için bir kez çalışır.

---
[➡️ Sonraki Konu: Hata Yönetimi](./04-Hata-Yonetimi.md)
