# 5.1. Keşif ve Bilgi Toplama (Discovery)

[↩️ Geri: İçindekiler](../../README.md)

Bir saldırgan bir sisteme ilk erişimi (initial foothold) elde ettikten sonraki ilk adımı, "nerede olduğunu" ve "etrafında ne olduğunu" anlamaktır. Bu aşamaya Keşif (Discovery) denir. PowerShell, "Living Off the Land" (LOTL) felsefesiyle, yani sisteme dışarıdan şüpheli bir araç getirmeden, mevcut Windows araçlarını kullanarak bu keşfi yapmak için mükemmeldir.

Bu bölümde kullanılan komutlar, Mavi Takım'ın izlediği birçok temel aktiviteyi tetikler, ancak bir saldırganın bakış açısıyla bu komutların neden değerli olduğunu anlamak önemlidir.

### Yerel Sistem Keşfi

Saldırgan, ilk olarak sızdığı makine hakkında bilgi toplar.

-   **Mevcut Kullanıcıyı ve Yetkilerini Öğrenme (T1033: System Owner/User Discovery)**
    ```powershell
    whoami /all
    ```

-   **Sistem Bilgilerini Toplama (T1082: System Information Discovery)**
    ```powershell
    Get-ComputerInfo | Select-Object OsName, OsVersion, CsDomain, CsProcessors
    systeminfo
    ```

-   **Ağ Yapılandırmasını Keşfetme (T1016: System Network Configuration Discovery)**
    ```powershell
    ipconfig /all
    Get-NetIPConfiguration
    Get-NetRoute
    ```

-   **Çalışan İşlemleri Listeleme (T1057: Process Discovery)**
    Antivirüs, EDR gibi güvenlik yazılımlarını veya değerli hedefleri (örn: parola yöneticileri) aramak için kullanılır.
    ```powershell
    Get-Process | Select-Object Name, Path
    tasklist /svc
    ```

-   **Mevcut Ağ Bağlantılarını Görme (T1049: System Network Connections Discovery)**
    Makinenin hangi diğer sistemlerle konuştuğunu, potansiyel C2 (komuta-kontrol) kanallarını veya yanal hareket hedeflerini bulmak için kullanılır.
    ```powershell
    netstat -anob
    Get-NetTCPConnection
    ```

### Ağ ve Active Directory Keşfi

Yerel makineyi anladıktan sonra saldırgan, ağdaki diğer hedefleri ve Active Directory (AD) ortamını haritalamaya başlar.

-   **Ağdaki Diğer Sistemleri Keşfetme (T1018: Remote System Discovery)**
    Basit bir `for` döngüsü ve `Test-Connection` ile ağdaki canlı ana makineler (host) bulunabilir.
    ```powershell
    # 192.168.1.0/24 ağını tarayalım
    for ($i=1; $i -le 254; $i++) {
        Test-Connection -ComputerName "192.168.1.$i" -Count 1 -ErrorAction SilentlyContinue | Select-Object Address, StatusCode
    }
    ```

-   **Domain Kullanıcılarını Listeleme (T1087.002: Account Discovery: Domain Account)**
    AD modülü veya PowerView gibi araçlar kullanılır.
    ```powershell
    # Active Directory Modülü ile
    Get-ADUser -Filter * | Select-Object SamAccountName, Enabled, LastLogonDate
    ```

-   **Domain Gruplarını ve Üyelerini Keşfetme (T1069.002: Permission Groups Discovery: Domain Groups)**
    Özellikle "Domain Admins", "Enterprise Admins" gibi yüksek yetkili gruplar hedeftir.
    ```powershell
    Get-ADGroupMember -Identity "Domain Admins"
    ```

-   **Domain Güven İlişkilerini (Trusts) Haritalama (T1482: Domain Trust Discovery)**
    Bir domain'den diğerine atlamak için güven ilişkileri kullanılır.
    ```powershell
    Get-ADTrust -Filter *
    ```

---
[➡️ Sonraki Konu: Erişim ve Kod Yürütme](./02-Erisim-ve-Kod-Yurutme.md)
