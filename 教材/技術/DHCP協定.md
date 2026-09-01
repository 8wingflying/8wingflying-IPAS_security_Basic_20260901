# DHCP（Dynamic Host Configuration Protocol）

## 一、DHCP 是什麼？

DHCP（Dynamic Host Configuration Protocol，動態主機設定協定）是一種用來**自動分配與管理網路參數**的協定。

當電腦、手機、印表機或其他設備連上網路時，不需要管理者逐台手動設定 IP 位址，DHCP Server 即可自動提供：

- IP Address
- Subnet Mask
- Default Gateway
- DNS Server
- Lease Time
- Domain Name
- NTP Server 等資訊

例如 DHCP 可能提供：

```text
IP Address      : 192.168.1.105
Subnet Mask     : 255.255.255.0
Default Gateway : 192.168.1.1
DNS Server      : 192.168.1.1
Lease Time      : 86400 seconds
```

基本概念：

```text
設備加入網路
    ↓
向 DHCP Server 要求網路設定
    ↓
DHCP Server 分配 IP
    ↓
設備開始正常進行網路通訊
```

---

## 二、DHCP 使用的通訊協定與 Port

DHCP 使用 UDP：

- DHCP Server：UDP Port 67
- DHCP Client：UDP Port 68

```text
DHCP Client
UDP 68
   │
   │ DHCP Request
   ▼
DHCP Server
UDP 67
```

如果防火牆完全阻擋 UDP 67/68，DHCP Client 通常無法正常取得 IP 位址。

---

## 三、DHCP 的核心流程：DORA

DHCP 最重要的觀念是 DORA：

- D：Discover
- O：Offer
- R：Request
- A：Acknowledge

```text
┌───────────────┐                     ┌───────────────┐
│ DHCP Client   │                     │ DHCP Server   │
│ 0.0.0.0       │                     │ 192.168.1.1   │
└───────┬───────┘                     └───────┬───────┘
        │                                     │
        │ ① DHCPDISCOVER                     │
        │ ----------------------------------> │
        │                                     │
        │ ② DHCPOFFER                         │
        │ <---------------------------------- │
        │                                     │
        │ ③ DHCPREQUEST                       │
        │ ----------------------------------> │
        │                                     │
        │ ④ DHCPACK                           │
        │ <---------------------------------- │
        │                                     │
        ▼                                     ▼
取得 IP：192.168.1.100
```

### 1. DHCPDISCOVER

當 Client 剛加入網路時，通常還沒有有效 IP，也不知道 DHCP Server 的位置，因此會送出廣播封包。

```text
Source IP      : 0.0.0.0
Destination IP : 255.255.255.255
Source Port    : UDP 68
Destination    : UDP 67
```

可理解為：

> 網路上有 DHCP Server 嗎？請提供我網路設定。

### 2. DHCPOFFER

DHCP Server 收到 Discover 後，會提出可使用的網路設定，例如：

```text
IP Address      : 192.168.1.100
Subnet Mask     : 255.255.255.0
Gateway         : 192.168.1.1
DNS             : 8.8.8.8
Lease Time      : 86400 seconds
DHCP Server     : 192.168.1.1
```

### 3. DHCPREQUEST

Client 可能同時收到多台 DHCP Server 的 Offer，例如：

```text
DHCP Server A
Offer → 192.168.1.100

DHCP Server B
Offer → 192.168.1.200
```

Client 會選擇其中一個 Offer，並送出 DHCPREQUEST。

### 4. DHCPACK

DHCP Server 最後送出 DHCPACK，代表正式確認租約。

```text
Client                         DHCP Server
  │                                │
  │ ------ DHCP Discover --------> │
  │                                │
  │ <------ DHCP Offer ----------- │
  │                                │
  │ ------ DHCP Request ---------> │
  │                                │
  │ <------ DHCP ACK ------------- │
```

---

## 四、DHCP Lease：IP 租約

DHCP 配發的 IP 通常不是永久的，而是採用 Lease（租約）機制。

例如：

```text
IP Address : 192.168.1.100
Lease Time : 8 hours
```

典型租約週期：

```text
取得 Lease
     │
     ├── 50%：T1 Renewal
     │
     ├── 87.5%：T2 Rebinding
     │
     └── 100%：Lease Expired
```

### T1：Renewal

通常在租約時間到約 50% 時，Client 會向原 DHCP Server 要求續租。

### T2：Rebinding

如果原 DHCP Server 沒有回應，到租約大約 87.5% 時，Client 會嘗試透過廣播尋找其他 DHCP Server。

---

## 五、DHCP 位址池 DHCP Pool

DHCP Server 通常會建立可以分配的 IP 範圍。

```text
Network:
192.168.1.0/24

Gateway:
192.168.1.1

DHCP Pool:
192.168.1.100
       ↓
192.168.1.200
```

例如：

- 192.168.1.1：保留給 Router
- 192.168.1.2 ～ 192.168.1.99：保留給 Server、Printer、NAS
- 192.168.1.100 ～ 192.168.1.200：供 DHCP 自動分配

---

## 六、DHCP Reservation

DHCP Reservation 可讓特定設備每次透過 DHCP 都取得相同 IP。

例如：

```text
Printer MAC:
00:11:22:33:44:55

固定 DHCP IP:
192.168.1.50
```

```text
00:11:22:33:44:55
        ↓
DHCP Server
        ↓
192.168.1.50
```

這與完全手動設定 Static IP 不完全相同。

---

## 七、Static IP 與 DHCP 比較

| 項目 | Static IP | DHCP |
|---|---|---|
| IP 配置 | 手動 | 自動 |
| 管理成本 | 高 | 低 |
| 適合大量設備 | 較差 | 很適合 |
| IP 衝突風險 | 較高 | 較低 |
| Server 使用 | 常見 | 可搭配 Reservation |
| 一般 PC | 較少使用 | 非常常見 |

企業網路常見方式：

```text
Server
Router
Firewall
Domain Controller
DNS
    ↓
Static IP

User PC
Notebook
Mobile Phone
    ↓
DHCP
```

---

## 八、DHCP Relay

DHCPDISCOVER 通常是 Layer 2 Broadcast，而 Router 一般不會直接轉送廣播。

如果 DHCP Server 位於不同 VLAN，通常需要 DHCP Relay。

```text
Client
192.168.10.x
    │
    │ DHCP Broadcast
    ▼
Router / L3 Switch
DHCP Relay
    │
    │ DHCP Relay
    ▼
DHCP Server
192.168.100.10
```

Cisco 常見設定：

```text
interface vlan 10
 ip helper-address 192.168.100.10
```

---

## 九、DHCP Option

DHCP 除了配發 IP，也可以透過 DHCP Options 傳送其他網路資訊。

| DHCP Option | 功能 |
|---|---|
| Option 1 | Subnet Mask |
| Option 3 | Router / Default Gateway |
| Option 6 | DNS Server |
| Option 15 | Domain Name |
| Option 42 | NTP Server |
| Option 51 | IP Address Lease Time |
| Option 53 | DHCP Message Type |
| Option 54 | DHCP Server Identifier |
| Option 66 | TFTP Server Name |
| Option 67 | Boot File Name |

Option 53 常用來表示 DHCP 訊息類型：

```text
1 = DHCP Discover
2 = DHCP Offer
3 = DHCP Request
5 = DHCP ACK
```

---

## 十、DHCP 與 APIPA

Windows Client 如果無法找到 DHCP Server，可能會自行產生：

```text
169.254.x.x
```

這稱為 APIPA（Automatic Private IP Addressing）。

位址範圍：

```text
169.254.0.0/16
```

例如：

```text
IPv4 Address:
169.254.35.17
```

通常代表 Client 無法成功取得 DHCP 租約。

常見檢查項目：

- DHCP Server 是否正常
- 網路線或 Wi-Fi
- VLAN 設定
- DHCP Relay
- Switch Port
- Firewall
- DHCP Scope 是否耗盡

---

## 十一、Windows 查看 DHCP

查看完整網路設定：

```powershell
ipconfig /all
```

例如：

```text
DHCP Enabled. . . . . . . . : Yes
DHCP Server . . . . . . . . : 192.168.1.1
IPv4 Address. . . . . . . . : 192.168.1.100
Default Gateway . . . . . . : 192.168.1.1
```

釋放與重新取得 IP：

```powershell
ipconfig /release
ipconfig /renew
```

---

## 十二、Linux 查看 DHCP

查看 IP：

```bash
ip addr
```

使用 NetworkManager：

```bash
nmcli device show
```

查看路由與 Default Gateway：

```bash
ip route
```

---

## 十三、Wireshark 分析 DHCP

Wireshark 顯示過濾器：

```text
dhcp
```

某些版本也可能使用：

```text
bootp
```

正常 DHCP 封包序列：

```text
DHCP Discover
DHCP Offer
DHCP Request
DHCP ACK
```

如果只看到：

```text
Discover
Discover
Discover
```

則可能表示 Client 找不到 DHCP Server 或 DHCP 回覆無法到達 Client。

---

## 十四、DHCP 的資安風險

### 1. Rogue DHCP Server

攻擊者在區域網路架設假的 DHCP Server：

```text
             ┌── Legitimate DHCP Server
Client ──────┤
             └── Rogue DHCP Server
                     │
                     ▼
               惡意 Gateway
               惡意 DNS
```

攻擊者可能提供惡意的 Gateway 或 DNS，進一步造成：

```text
Rogue DHCP
    ↓
惡意 Gateway / DNS
    ↓
MITM
    ↓
流量竊聽或導向
```

### 2. DHCP Starvation Attack

攻擊者偽造大量 MAC Address，不斷向 DHCP Server 申請 IP：

```text
MAC 1 → 要 IP
MAC 2 → 要 IP
MAC 3 → 要 IP
MAC 4 → 要 IP
...
```

可能導致：

```text
DHCP Pool
    ↓
大量惡意申請
    ↓
Pool Exhausted
    ↓
正常使用者無法取得 IP
```

這是一種具有 DoS 特性的攻擊。

---

## 十五、DHCP Snooping

DHCP Snooping 是交換器常見的 Layer 2 安全機制，可降低 Rogue DHCP Server 等風險。

交換器 Port 可分成：

- Trusted Port
- Untrusted Port

```text
DHCP Server
     │
     │ Trusted Port
     ▼
┌───────────────┐
│    Switch     │
└───────┬───────┘
        │
        │ Untrusted Port
        ▼
      Client
```

一般情況下，只有 Trusted Port 才允許合法的 DHCP Server 回應，例如：

```text
DHCPOFFER
DHCPACK
```

若攻擊者從一般 Client Port 啟動 Rogue DHCP：

```text
Attacker
   │
   │ DHCPOFFER
   ▼
Switch
   │
   X Block
```

即可降低 Rogue DHCP Server 攻擊成功的機率。

---

## 十六、DHCP Snooping Binding Table

DHCP Snooping 可建立 Binding Table，例如：

```text
MAC Address         IP Address       VLAN   Interface
-----------------------------------------------------
00:11:22:33:44:55   192.168.1.100    10     Gi0/1
00:11:22:33:44:66   192.168.1.101    10     Gi0/2
```

此資料還可以搭配：

- Dynamic ARP Inspection（DAI）
- IP Source Guard

形成常見的 Layer 2 安全防禦架構。

```text
DHCP Snooping
      │
      ├── Dynamic ARP Inspection
      │
      └── IP Source Guard
```

---

## 十七、DHCP 整體架構

```text
                   DHCP Server
                 192.168.100.10
                       ▲
                       │
                    UDP 67
                       │
                 DHCP Relay
                       ▲
                       │
             ┌─────────┴─────────┐
             │                   │
           VLAN 10             VLAN 20
        192.168.10.0/24     192.168.20.0/24
             │                   │
          Client A            Client B
           UDP 68              UDP 68
```

DORA 流程：

```text
Discover
   ↓
Offer
   ↓
Request
   ↓
ACK
```

---

## 十八、重點速記

| 重點 | 說明 |
|---|---|
| DHCP | Dynamic Host Configuration Protocol |
| 功能 | 自動分配 IP 與網路參數 |
| 傳輸層 | UDP |
| Server Port | UDP 67 |
| Client Port | UDP 68 |
| DORA | Discover → Offer → Request → ACK |
| DHCP Relay | 讓 DHCP 跨 VLAN 運作 |
| DHCP Snooping | 防禦 Rogue DHCP 等風險 |
| APIPA | DHCP 失敗時 Windows 可能使用 169.254.0.0/16 |

---

## 十九、教學延伸方向

DHCP 適合與以下主題一起教學：

- ARP
- VLAN
- DHCP Relay
- DHCP Snooping
- Rogue DHCP
- DHCP Starvation
- Dynamic ARP Inspection
- IP Source Guard
- MITM
- Layer 2 Security

這些機制彼此關聯密切，可作為企業交換式網路與資安防禦的重要基礎。
