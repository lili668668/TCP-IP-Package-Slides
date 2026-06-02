# IP 協定的主要兩個功能

----

## 功能一：裝箱

- 裝箱就需要看 MTU （Maximum Transmission Unit 最大傳輸單位）
- 每個 Data layer 可處理的最大傳輸單位是不同的

| 網路型態 | MTU |
|---------|-----|
| Ethernet | 1,500 Bytes |
| X.25 | 1,600 Bytes |
| FDDI | 576 Bytes |
| ATM | 9,180 Bytes |
| Token Ring (16 Mbps) | 17,914 Bytes |
| Token Ring (4 Mbps) | 4,464 Bytes |

----

## 功能二：找路

![5d7c9c7a-9b77-49ba-91bf-89bd81138392](https://hackmd.io/_uploads/rJzClCLyfe.png)


---

# IP 封包（IP Package）

----

![截圖 2026-05-17 12.07.48](https://hackmd.io/_uploads/rJJXyaLyzg.png)

- IP 表頭 -> 投遞貨物的資料
- IP Payload -> 貨物

----

## IPv4 表頭

----
- Version
- Header Length
- Differentiated Services Field -> 封包怎麼處理

----

### Differentiated Services Field
- DSCP -> 優先級

|         DSCP 名稱         | 數值 |                用途               |
|:-------------------------:|:----:|:---------------------------------:|
| CS0 (Default)             | 0    | 一般流量，無優先級 |
| CS1                       | 8    | 低優先級（背景流量）              |
| AF11~AF43                 | 各種 | 一般商務應用，分等級保證          |
| EF (Expedited Forwarding) | 46   | 即時通訊（VoIP 電話、視訊會議）   |
| CS6                       | 48   | 網路控制協定                      |
| CS7                       | 56   | 最高優先級（保留給網路自己用）    |

- ECN -> 路上有沒有塞車

| CN 值 |             名稱            |                意思               |
|:-----:|:---------------------------:|:---------------------------------:|
| 00    | Not-ECT                     | 這個封包不支援 ECN |
| 10    | ECT(0)                      | 支援 ECN（傳輸層標記法 0）        |
| 01    | ECT(1)                      | 支援 ECN（傳輸層標記法 1）        |
| 11    | CE (Congestion Experienced) | 路由器標的：我這裡塞車了！        |

----

- Total Length -> IPv4 封包總長度
- Identification -> 封包的識別 ID
    - 如果超過 MTU ，需要把封包切分，需要這個識別 ID 得知哪幾個封包是同一個封包
- Flags -> 旗標
    - Reserved bit -> 保留 bit
    - Don't fragment -> 沒有分片
    - More fragments -> 後面還有
- Fragment Offset -> 偏移量
- Time to Live -> TTL
    - 每經過一個路由，就會減一，如果小於 0 封包就會被丟棄（Timeout）
- Protocol -> 協議（UDP/TCP）
- Header Checksum -> 校驗碼，避免傳輸過程中的電磁干擾，導致資料變異損毀，沒有被發現
    - 由於 TTL 的機制，每經過一個路由 checksum 就要重算一次，所以這個 checksum ，非常沒有效率，在 IPv6 的協議裡被移除了
    - 現在的網卡會有晶片做硬體加速計算 checksum，但是 Wireshark 會在封包到網卡前攔截，所以 checksum 常常是驗證失敗的
- Source Address -> 來源
- Destination Address -> 目的地

---

# IPv6 表頭


----

- Version
- Traffic class -> 跟 IPv4 的Differentiated Services Field 一樣
- Payload Length -> 資料長度（不包含表頭）
- Next Header -> 下一個協議
- Hop Limit -> 跟 IPv4 的 TTL 是一樣的
(checksum 消失了owo)
- Source Address
- Destination Address

---

# ARP 協議（Address Resolution Protocol）

將 IP -> MAC 地址的協議

----

## ARP 封包

- Hardware type -> 網路技術
- Protocol type -> 協議
- Hardware size -> MAC address 的長度
- Protocol size -> IP address 的長度
- Opcode -> 執行代碼
- Sender MAC address -> 傳送者的 MAC address
- Sender IP address -> 傳送者的 IP address
- Target MAC address -> 接收者的 MAC address
- Target IP address -> 接收者的 IP address

---

# DNS 封包

----

- Transaction ID
- Flags
- questions -> 問題數
- Answer RRs -> 有幾筆回答
- Authority RRs -> 有幾個 NS 有這個記錄
- Additional RRs -> 額外的答案

----

有發現一件事嗎？

----

ARP 跟 DNS 封包沒有加密喔(° ∀ °)

----

沒加密 = 可攻擊


----

# DNS 汙染

[A] google.com IP 是什麼？-> [中間人] google.com IP 是什麼？-> [DNS Server]

[A] <- google.com IP 是 8.87.87.2 [中間人] <- google.com IP 是 3.12.45.7 [DNS Server]

----

# DNS over TLS

- 讓 DNS 經過 TCP 連線，且加密

----

# ARP 攻擊防範

----

- Dynamic ARP inspection -> 監聽 DHCP（動態 IP 分配協議） 的交握流程，與每個 ARP 封包確保每個封包資料正確
- Private VLAN -> 一個裝置一個虛擬區網，讓大家都是獨立區網就沒有人可以攻擊你
- 802.1X 連線認證 -> 認證連入區網的使用者是誰
- 換掉 ARP
    - IPv6 使用 NDP 協議（ARP + ICMP）可搭配 SEND （有加密的 NDP）使用（但實務 SEND 好像很少人用的樣子），大家都用 RA Guard （跟 DAI 是一樣的概念）

---

# ICMP 協議

- 用來做網路診斷的 debug 協議
- 封包
    - Type
    - Code -> Type 的結果代碼，不同的 Type 、同一個 Code 會有不同的意義
    - Identifier -> ID，因為你可能同時在執行多個 ping （之類的）
    - Sequence Number -> ping 會送出很多封包嘛，這個是封包的 ID
    - data -> 前面是時間戳，後面是無意義的資料填充

---

# TCP 協議

- 三方交握
    - [SYN]：來，握手 
    - [SYN, ACK]：好，握手
    - [ACK]：好，我們握手了

![ChatGPT Image 2026年6月2日 下午08_04_25](https://hackmd.io/_uploads/BkLW-InxGe.jpg)


- 其他
    - [ECE]：路上有壅塞了，請降速
    - [CWR]：收到，我已經降速了
    - [PSH]：現在立刻給我資料

----

# TCP 封包的其他部分

- Window size -> 用來告訴對方我現在腹地有多大，你可以送多大的封包過來
- Options -> 一堆參數

---

# UDP 協議

- 一個只負責發訊息，沒有要管訊息有沒有到的協議
- 因為沒什麼好講的，所以我們來講講他的其他應用

----

# DNS 協議

- 又回來了owo
- 因為 DNS 協議是基於 UDP 去傳輸的

----

# QUIC

- 全名叫「Quick UDP Internet Connection」（我很快 UDP）
- 我大 Google 提出的協議
- 加密協議，資料穿好衣服

----

# QUIC 三方交握

![ChatGPT Image 2026年6月2日 下午08_28_47](https://hackmd.io/_uploads/ByOZaH2eGe.jpg)

---

# 補充：MACsec

- 網路層級的第二層的連線加密防護
- 因為做在第二層，所以屬於硬體層級的加密
- 一般家用電腦沒有辦法使用，通常用在資料中心或企業網路

---

# 補充：HTTP 封包

- 其實我好像應該只有 TCP/IP 層的封包解說而已，應用層應該不在我的範圍
- 但是都拆了 DNS 封包了，我覺得好像拆一下 HTTP 封包也沒有問題
- 看吶，這裡有個 https://httpforever.com/ 我們來看看他裸奔的資料吧

