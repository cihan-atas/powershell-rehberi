# 3.4. Arka Plan Görevleri (Jobs)

[↩️ Geri: İçindekiler](../README.md)

Bazı PowerShell komutları veya script'leri (örneğin, yüzlerce sunucuda envanter toplama, büyük dosyaları kopyalama, kapsamlı log analizi) dakikalarca, hatta saatlerce sürebilir. Normalde, bir komut çalışırken PowerShell konsolunuz kilitlenir ve komut bitene kadar başka bir işlem yapamazsınız.

**Arka Plan Görevleri (Background Jobs)**, bu uzun süren görevleri mevcut PowerShell oturumunuzdan bağımsız, ayrı bir süreçte arka planda çalıştırmanıza olanak tanır. Görev arka planda çalışırken, siz konsolu interaktif olarak kullanmaya devam edebilirsiniz.

## Bir Görev Nasıl Başlatılır: `Start-Job`

Bir görevi arka planda başlatmak için `Start-Job` cmdlet'i kullanılır. Çalıştırılacak komut veya komutlar bir `-ScriptBlock` içine yazılır.

**Örnek:** Ağdaki 100 sunucuya ping atmanın uzun süreceğini varsayalım. Bu işlemi bir "job" olarak başlatalım.
```powershell
# Sunucu listemizi oluşturalım (örnek)
$servers = 1..100 | ForEach-Object { "SERVER$_" }

# Bu sunucuları tarayan script bloğunu bir job olarak başlat
$job = Start-Job -ScriptBlock {
    # Script bloğuna dışarıdan değişken göndermek için $using: kullanılır
    # Veya -ArgumentList parametresi ile gönderilir
    param($serverList)
    
    $results = @()
    foreach ($server in $serverList) {
        if (Test-Connection -ComputerName $server -Count 1 -Quiet) {
            $results += [pscustomobject]@{
                ComputerName = $server
                Status = 'Online'
            }
        } else {
            $results += [pscustomobject]@{
                ComputerName = $server
                Status = 'Offline'
            }
        }
    }
    return $results
} -ArgumentList (,$servers) # ArgumentList bir dizi bekler, tek bir diziyi dizi olarak göndermek için başına virgül konur.

Write-Host "Sunucu tarama görevi arka planda başlatıldı. İş ID: $($job.Id)"
```
`Start-Job` komutu hemen bir "job" nesnesi döndürür ve konsol size geri verilir. Artık başka komutlar çalıştırabilirsiniz.

## Görevleri Yönetme

Bir veya daha fazla görevi başlattıktan sonra, durumlarını kontrol etmek ve sonuçlarını almak için ilgili cmdlet'leri kullanırız.

### 1. Görevleri Listeleme: `Get-Job`
Oturumda başlatılmış olan tüm görevleri ve durumlarını (`Running`, `Completed`, `Failed`) listeler.
```powershell
Get-Job
```
**Örnek Çıktı:**
```
Id     Name            PSJobTypeName   State         HasMoreData     Location             Command
--     ----            -------------   -----         -----------     --------             -------
1      Job1            BackgroundJob   Running       True            localhost            ...
```

### 2. Görev Sonuçlarını Alma: `Receive-Job`
Tamamlanmış bir görevin ürettiği çıktıyı (return ettiği değerleri) almak için kullanılır.
```powershell
# ID'si 1 olan görevin sonuçlarını al
Receive-Job -Id 1
```
> **⚠️ Önemli:** `Receive-Job` komutunu çalıştırdığınızda, görevin sonuçları bellekten okunur ve bir daha aynı komutla çağrıldığında boş döner. Sonuçları kalıcı olarak saklamak isterseniz, `-Keep` parametresini kullanın veya sonuçları bir değişkene atayın.
> ```powershell
> $jobResults = Receive-Job -Id 1 -Keep
> ```

### 3. Bir Görevi Durdurma: `Stop-Job`
Henüz tamamlanmamış bir görevi zorla durdurmak için kullanılır.
```powershell
Stop-Job -Id 1
```

### 4. Görevleri Temizleme: `Remove-Job`
Tamamlanmış veya durdurulmuş bir görevi ve sonuçlarını bellekten tamamen temizler.
```powershell
# Tamamlanmış tüm görevleri temizle
Get-Job -State Completed | Remove-Job

# Tüm görevleri temizle
Get-Job | Remove-Job
```

## Diğer Görev Türleri

`Start-Job` yerel makinede bir arka plan süreci oluşturur. Ancak farklı görev türleri de vardır:

-   **`Invoke-Command -AsJob`:** Bir komutu birden çok uzak makinede **arka plan görevi olarak** çalıştırır. Her bir uzak makine için ayrı bir alt görev (child job) oluşturur. Bu, yüzlerce sunucuda aynı anda uzun süren bir script çalıştırmak için en verimli yöntemdir.
    ```powershell
    $servers = "WEB01", "SQL01"
    $remoteJob = Invoke-Command -ComputerName $servers -ScriptBlock { Start-Sleep -Seconds 20 } -AsJob
    ```
-   **Zamanlanmış Görevler (Scheduled Jobs):** `PSSCheduler` modülü ile, belirli bir zamanda veya periyodik olarak çalışacak PowerShell görevleri oluşturulabilir. Bu, Windows Görev Zamanlayıcısı'nın (Task Scheduler) PowerShell versiyonudur.

## Ne Zaman "Job" Kullanmalıyım?

-   Script'inizin çalışması 30-60 saniyeden uzun sürüyorsa.
-   Paralel olarak birden çok görevi aynı anda başlatmak istediğinizde (örneğin, 10 farklı sunucudan log dosyası indirmek).
-   İnteraktif bir oturumda, bir komutun bitmesini beklerken başka işler yapmak istediğinizde.

---
[➡️ Sonraki Konu: Veri İşleme: CSV, JSON ve XML](./05-Veri-Isleme-CSV-JSON-XML.md)
