---
layout: post
published: true
title: Network Virtualization using Generic Routing Encapsulation
tags: 
  - Tunneling
  - Network
---

Network Virtualization using Generic Routing Encapsulation(NVGRE)
是由 Micrsoft 為首與 Intel、Dell、Broadcom、Arista 在 2011 年 9 月共同提出的 IETF 草案，
僅略晚 VXLAN 不到一個月，目前已修訂到第三版，
不同於 VXLAN 使用 UDP 封包格式
NVGRE 採用現有一般路由封裝(Generic Routing Encapsulation，GRE) 之格式，
隧道兩端點稱為網路虛擬化端點(Network Virtualization Edge，NVE)。

與 VXLAN 有相同程度上的雷同，因為兩個草案都是要解決類似的問題，
在大型資料心中心中多客戶的網路隔離問題。


## Header Format
![2013_12_08_1.png]({{site.baseurl}}/assets/images/blog/2013_12_08_1.png)

## Outer Ethernet Header

### Destination Address
用來設定 destination NVGRE endpoint 的 MAC address，如果 destination NVE 與 Source NVE 不屬於同一個 L3 的 domain 下的時候，這個欄位就會用來填入 next hop device(router) 的 MAC addrss。

### VLAN
是一個可填可不填的欄位，如果有填的話就會就會利用 ethertype of 0x8100 來實作可隔離網路的 VLAN ID Tag。

### IP Header
目前可以支援 IPv4 與 IPv6，
在這裡我們僅說明 IPv4 的封包格式，
其實這邊的封包格式與一般 IPv4 的封包格式相同，
只是這邊的傳輸都是建立在 NVE 與 NVE 之間。

### Protocol
這邊會設定為 0x2F 表示，封包裡面的內容是 GRE 封包

### Source IP
送出封包的來源 NVE

### GRE header
NVGRE 是由 GRE header 來包覆被封裝的 inner ethernet 的封包，

### Virtual Subnet ID(VSID)
24 bit 的欄位，用來代表不同 NVGRE 群組的編號，不同編號的 VSID 群組，不能互相溝通。
可支援超過一千六百萬個不同虛擬網路，遠大於 VLAN 4094 個的限制。

### Flow ID
8 bit 的欄位，是用來識別不同的資料流，
然而現有的等價多路徑路由機制(Equal-Cost Multi path Routing, ECMP)僅根據 TCP/IP header 來辦識資料流，
並不支援 GRE header，因此在底層設備並未支援 FlowID 的檢視之前，
NVGRE 建議使用多個 NVE 實體 IP 位置來對應不同的資料流，
整體上來說，在負載平衡上的處理仍指不夠細緻。

例如當 NVE 收到封包是一個會檢查其目的乙太網路位置來查詢要送到哪個通道，
並且查詢來源乙太網路位址屬於哪位客戶，透過這兩個欄位(VSID、FlowID)來決定封包的傳送方式。

### Inner ethernet header
與一般的 ethernet header 相同，
與 VXLAN 不同的是 inner ethernet header 並不支援 802.1Q VLAN。

### Brocast、Muticast
在廣播和群播的處理上，NVGRE 可使用 IP 群播的機制，
若底層網路不支援 IP 群播，亦可用多重單點傳送來達成相同功能；
同時考量隧道封包機制會將增加整體封包長度，
若長度超過底層網路的最大傳輸單元（Maximum Transmission Unit，MTU）時封包會被強制分段（fragmentation）至零碎的封包，
這些封包於隧道的另一端接收後再進行重組工作，
將會增加隧道端點的負荷並降低網路傳輸效率，
NVGRE 在內部封裝為 IP 封包時會辨識網路路徑 MTU（Path MTU），
若大於封裝後的長度則會強迫封包維持原狀以避免此分段的情境，
以上兩點均讓 NVGRE 適於在現有網際網路上傳遞。
