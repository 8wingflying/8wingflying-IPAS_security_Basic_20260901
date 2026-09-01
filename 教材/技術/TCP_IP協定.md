# TCP/IP協定
- 應用層常用協定 (Application Layer Protocols)
- 傳輸層常用協定 (Transport Layer Protocols) 
- 網路層常用協定 (Network Layer Protocols) 


## 應用層常用協定 (Application Layer Protocols) 

以下是網路模型中常見應用層協定的統整列表，包含其預設埠號、功能說明以及與資訊安全相關的通訊特性：

| 協定名稱 | 預設埠號 | 功能說明與通訊特性 |
| :--- | :--- | :--- |
| **HTTP** (HyperText Transfer Protocol) | 80 | 用於網頁伺服器與客戶端之間的資料傳輸。採明文傳輸，易遭中間人攻擊（MitM）攔截或竄改。 |
| **HTTPS** (HTTP Secure) | 443 | 結合 TLS/SSL 的 HTTP，提供端到端加密通訊與伺服器身分認證，為現代網站傳輸標準。 |
| **DNS** (Domain Name System) | 53 | 將人類可讀的網域名稱解析為 IP 位址。經常成為 DNS Spoofing 或 DDoS 放大攻擊的目標。 |
| **SSH** (Secure Shell) | 22 | 提供加密的遠端系統管理、指令執行與隧道（Tunneling）功能，完全取代明文傳輸的 Telnet 以防止網路側錄。 |
| **FTP** (File Transfer Protocol) | 20, 21 | 用於伺服器與客戶端之間的檔案傳輸（21 埠控制，20 埠傳輸資料）。因憑證採明文傳送，實務上多以 SFTP 或 FTPS 替代。 |
| **SMTP** (Simple Mail Transfer Protocol)| 25, 465, 587| 負責發送電子郵件至郵件伺服器，或在伺服器之間進行信件路由。 |
| **POP3** (Post Office Protocol v3) | 110, 995| 用於接收電子郵件，主要機制為將信件下載至本地端設備後，從伺服器上刪除原始信件。 |
| **IMAP** (Internet Message Access Protocol)| 143, 993| 用於接收電子郵件，允許客戶端直接在伺服器上讀取、管理與雙向同步信件狀態。 |
| **DHCP** (Dynamic Host Configuration Protocol)| 67, 68| 自動分配 IP 位址、子網路遮罩及預設閘道等網路參數給區域網路內的新連線設備。 |

## 傳輸層常用協定 (Transport Layer Protocols) 

以下是網路模型中傳輸層（Transport Layer）常用協定的統整列表，包含其傳輸特性、功能說明以及與資訊安全相關的通訊特性：

| 協定名稱 | 傳輸特性 | 功能說明與資安/通訊特性 |
| :--- | :--- | :--- |
| **TCP** (Transmission Control Protocol) | 連線導向 (Connection-oriented)、可靠傳輸 | 透過三次握手 (Three-way Handshake) 建立連線，具備封包排序、錯誤檢查、流量控制與遺失重傳機制，確保資料完整抵達。<br>**資安特性**：其交握機制常被駭客利用於 SYN Flood 阻斷服務攻擊 (DDoS)，或透過 Nmap 等工具進行 TCP 隱匿掃描 (Stealth Scan) 來探測系統弱點。 |
| **UDP** (User Datagram Protocol) | 非連線導向 (Connectionless)、不可靠傳輸 | 不保證封包抵達順序或是否遺失，無交握程序，標頭極小且傳輸延遲低。適用於即時影音串流、VoIP 或 DNS 查詢。<br>**資安特性**：因無需驗證連線且容易偽造來源 IP，極常被利用於 UDP 反射與放大攻擊 (Reflection/Amplification Attack)。 |
| **QUIC** (Quick UDP Internet Connections) | 基於 UDP、內建安全機制的可靠傳輸 | 由 Google 開發並成為 HTTP/3 的底層標準。在 UDP 之上實作了可靠傳輸，並深度整合 TLS 1.3，達成 0-RTT 或 1-RTT 的極速連線，同時解決了 TCP 的隊頭阻塞 (Head-of-line blocking) 問題。 |
| **SCTP** (Stream Control Transmission Protocol) | 訊息導向 (Message-oriented)、支援多重主機 (Multi-homing) | 允許在兩個端點之間建立多條網路路徑，網路備援與容錯能力極高，廣泛應用於現代電信網路 (如 4G/5G 核心網的信令傳輸)。<br>**資安特性**：採用四次交握 (Four-way Handshake) 並引入 Cookie 機制，先天上對 SYN Flood 類型的資源耗盡攻擊具備較佳的防禦力。 |

## 網路層常用協定 (Network Layer Protocols) 

以下是網路模型中網路層（Network Layer）常用協定的統整列表，包含其核心功能以及與資訊安全相關的通訊特性：

| 協定名稱 | 核心功能說明 | 資安與通訊特性 |
| :--- | :--- | :--- |
| **IPv4** (Internet Protocol version 4) | 提供無連線 (Connectionless) 的資料傳遞與 32 位元的邏輯定址 (IP 位址)，是目前網際網路最核心的協定。 | 封包採明文傳輸且不保證傳送成功，缺乏內建的安全防護，極易遭受來源 IP 偽造 (IP Spoofing) 攻擊。 |
| **IPv6** (Internet Protocol version 6) | IPv4 的升級標準，提供 128 位元的龐大定址空間，徹底解決 IP 位址枯竭問題，並簡化標頭格式以提升路由處理效率。 | 原始設計中即深度整合 IPsec，原生支援網路層的端到端加密與身分驗證機制，整體安全性較 IPv4 大幅提升。 |
| **ICMP** (Internet Control Message Protocol) | 負責傳報網路錯誤訊息與診斷網路連線狀態（如主機不可達或路由逾時）。日常使用的 `ping` 與 `traceroute` 指令皆基於此協定。 | 常被攻擊者用來進行網路探測 (Ping Sweep) 盤點存活主機，或利用發送大量請求發動 ICMP Flood (如 Smurf Attack) 阻斷服務攻擊。 |
| **IPsec** (Internet Protocol Security) | 負責為 IP 封包提供加密、資料完整性校驗與來源驗證的協定套件（主要包含 AH 與 ESP），為建立 Site-to-Site VPN 的核心技術。 | 運作於網路層，能透明地保護上層（傳輸層與應用層）的所有通訊流量，應用程式無須任何修改即可享有加密保護。 |
| **ARP** (Address Resolution Protocol) | 負責將邏輯 IP 位址解析為實體的 MAC 位址，運作於網路層與資料鏈結層之間，是區域網路 (LAN) 內設備互連的基礎。 | 因設計上完全缺乏身分驗證機制，網域內的設備皆可盲目信任並接收 ARP 回應，極易遭受 ARP Spoofing (ARP 欺騙) 導致中間人攻擊 (MitM)。 |
| **BGP / OSPF** (Routing Protocols) | 路由協定負責在路由器之間動態交換網路拓樸資訊，計算並決定資料封包從來源至目的地的最佳傳遞路徑。 | OSPF 多用於內部網路；BGP 則是網際網路骨幹的核心。BGP 若遭惡意發布錯誤路由 (BGP Hijacking)，將導致大規模連線癱瘓或流量遭竊聽。 |
