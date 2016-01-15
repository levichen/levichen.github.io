---
layout: post
published: true
title: Virtual eXtensible Local Area Network
tags: 
  - Tunneling
  - Network
---

Virtual eXtensible Local Area Network(VxLAN)
是由 Cisco、VMware、Citrix、Red Hat 等廠商在 2011 8 月所合作提出的 IETF 草案，
目前最新版本為第六版。
欲解決下列問題:

1. VLAN 數量不足在大型資料中心網路擴展性、隔離性與虛擬機器跨子網路遷移的問題：
傳統資料中心是利用 VLAN 來隔離不同租戶的網路，
但 VLAN 只提供了 12 bit 的 VLAN ID 的範圍用來區分
並希望透過 VxLAN 的機制將多個不同 layer3 的網路共同連結，就像是處在同一個 layer2 的網段。

2. GRE tunnel 的不足：
看了之前的 後就可以發現，
GRE 是一種 point to point 的 tunnel protocol，
使得在 broadcast 時需要針對每個 node 重複發送。

或許可以透過 GRE broadcast 的特性來擴充，
但是 VXLAN 的外層封裝的 UDP 特性使得在此方面的作用是無法被 GRE 取代的，
除非所有交換器都能察覺被封裝的封包內層資訊。

若要架設一個 VxLAN 需要下元件:
- Multicast support, IGMP and PIM
- VXLAN Network Identifer (VNI)
- VXLAN Gateway
- VXLAN Tunnel End Point(VTEP)
- VXLAN Segment/VXLAN Overlay Network

## VXLAN Header format

![2013_11_26_10.png]({{site.baseurl}}/assets/images/blog/2013_11_26_10.png)

## Outer Ethernet Header
### Destination Address
用來設定 destination VTEP 的 MAC address，如果 destination VTEP 與 Source VTEP 不屬於同一個 L3 的 domain 下的時候，這個欄位就會用來填入 next hop device(router) 的 MAC addrss。

### VLAN
是一個可填可不填的欄位，如果有填的話就會就會利用 ethertype of 0x8100 來實作可隔離網路的 VLAN ID Tag。

### Ethertype
在目前是設定 0x8000 格式的 IPv4 的封包，預計在未來的版本將會支援 Ipv6。

## IP Header
其實這邊的封包格式與一般 IPv4 的封包格式相同，
只是這邊的傳輸都是建立在 VTEP 與 VTEP 之間。

### Protocol
這邊會設定為 0x11 表示，封包裡面的內容是 UDP 封包

### Source IP
送出封包的來源 VTEP

### Destination IP
目標的 VTEP IP address，如果發送端的 VTEP 不知道的話，會透過下列的步驟來尋找:

發送端的 VM 會透過發送端的 VTEP 會送出 multicast 的封包給同在一個群組下的 VTEP(有著相同 VNI 的 VTEP)。
這些相同群組下的 VTEP 收到封包後，就會學到 soure virtual machine 的 MAC address，與發送端 VTEP IP address 的對應關係。
各 VTEP 會去 mapping 自己下面的 VM 有沒有這個 multicast 封包內的要找的 destination virtual machine， 如果有的話，就會回送一個 response 的封包給發送端的 VTEP，告訴他接收端的 VTEP IP address，
然後發送端的 VTEP 也會學起來 destination virtual machine MAC address 與 VTEP IP address 的對應關係。

## UDP Header
整個VXLAN 封包再由外層的 IP/UDP 封包透過第三層實體網路在兩個 VTEP 間傳遞。

### Source Port
用來作為 Layer2 trunk 或 L3 multipath 對封包進行 hashing 作為時轉送的依據，
而為了在多路徑的網路環境下等價多路徑路由（Equal Cost Multi-Path，ECMP）達到負載平衡，
建議 VTEP 在設定 UDP 的來源端埠號時參考內部被封裝的封包表頭的雜湊值，
可依此區別不同資料流（flow），確保屬於相同資料流的封包在第三層傳輸的路徑相同，
並讓不同資料流的封包分散至等價的路徑上。

而當封包被封裝完以後 VTEP 會判斷封包的來源與目的決定要透過點播(unicast)或者群播(multicast)來傳遞封包到另外一個 VTEP，
VTEP 的學習機制類似乙太網路交換器，不同的地方在於它是除了 MAC 外再額外加上目的 VTEP 的 IP 來學習，
就是先判斷透過 VNI 找到所有關聯的 VTEP 節點後就能學習到某個目標在哪個 VTEP 了。

### VXLAN Port
已向 IANA 申請了4789 作為辨識 VXLAN 的依據。

### Checksum
這個欄位應該要被設定為 Ethertype 0x0000，
如果 Source VTEP 不是設定 0x0000 格式的話，
Receiving VTEP 就會去認證 checksum，如果不符合的話，就會丟棄(dropped)這個封包。

## VXLAN Header
### VXLAN Flags(8bit)
如果 I flag 被設為 1，就一定要去認證 VXLAN Network ID(VNI)，
其它 7 個欄位都是保留欄位，目前都被設定為 0。

VXLAN Segment ID/VXLAN Network Identifier (VNI):
24 bit 的欄位，用來代表不同 VXLAN 群組的編號，不同編號的 VNI 群組，不能互相溝通。
可支援超過一千六百萬個不同虛擬網路，遠大於 VLAN 4094 個的限制。

下列會舉一個例子來說明，當兩個屬於同一 VNI 群組下的 VTEP 的 VM 如何互相溝通。

## VXLAN Header
### VXLAN Flags(8bit)
如果 I flag 被設為 1，就一定要去認證 VXLAN Network ID(VNI)，
其它 7 個欄位都是保留欄位，目前都被設定為 0。

### VXLAN Segment ID/VXLAN Network Identifier (VNI)
24 bit 的欄位，用來代表不同 VXLAN 群組的編號，不同編號的 VNI 群組，不能互相溝通。
可支援超過一千六百萬個不同虛擬網路，遠大於 VLAN 4094 個的限制。

下列會舉一個例子來說明，當兩個屬於同一 VNI 群組下的 VTEP 的 VM 如何互相溝通。

![2013_11_26_12.png]({{site.baseurl}}/assets/images/blog/2013_11_26_12.png)


當 VM1 要送一個封包給 VM2 的時候，會透過下列的步驟來知道 VM2 的 MAC address：

- VM1 送出一個 ARP 的封包，去問說 192.168.0.101(VM2) 的 MAC address 是多少。
- 這個 ARP 的封包經由 VTEP1 封裝後，把它塞進 multicast 的封包內，並且送給屬於同一個 multicast group 下的 VTEP( VNI 為 864 的 VTEP)
- 同群組下的 VTEP 收到 multicast 的封包後，會把 VTEP1 跟 VM1 的相關性，(00:0C:29:2F:32:A0是由 IP address 165.193.123.45 這個 VTEP 1 送出來的)，記錄在 VXLAN table 內。
- VTEP2 接到這個 multicast 封包後，會解開來，然後送出一個 broadcast 的封包給屬於 VNI 864 portgroups 內的所有 VM。
- VM2 看到這個 ARP 的封包，並且回應他自己的 MAC address(00:0C:29:2F:32:0A)。
- VTEP2 把這個 ARP response 封包後，透過 unicast IP 的封包，送給 VTEP1。
- VTEP1 接到後，解開來，並且把封包 pass 給 VM1。

透過上面的步驟 VM1 已經知道 VM2 的 MAC address 了，並且可以直接送封包給 VM2 了:

- VM1 送一個 IP 的封包給 VM2(Srouce IP address:192.168.0.100, Destation IP address:192.168.0.101)
- VTEP1 把這個封包封裝起來，並且把下列這些資訊加 Header 內:
  * VXLAN Header VNI = 864
  * 標準的 UDP header 並且把 UDP checkcum 設為 0x0000，然後把 destination port 設為 INAN 認證的 VXLAN port(4789)
  * 標準的 IP header 並且設定 destination IP address 為 VTEP2 的 IP address，然後 protocol 設定為 0x11，要使用 UDP 去傳送
  * 標準的 MAC header，destination MAC address 為 next hot address，在這個例子看來，就是中間 router 的 MAC address
- VTEP2 接到了從 UDP destination port(4789) 傳進來的 VXLAN 封包後，把封包解開來，然後會去 mapping 屬於 VNI 864 這個 portgroup 的 VM，知道是送給 VM2 後，就把 VXALN 的封包再解開，然後把封包 pass 給 VM2。
- VM2 接到從 VM1 送來的封包，並且不知道中間發生了什麼事(VXLAN 封裝、解封裝的過程)




