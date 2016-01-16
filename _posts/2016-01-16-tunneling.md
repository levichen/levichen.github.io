---
layout: post
published: true
title: Tunneling
tags: 
  - Network
  - Tunneling
---

![2013_11_25_1.jpg]({{site.baseurl}}/assets/images/blog/2013_11_25_1.jpg)

## 前言
隨著雲端網路的時代到來，
隨用隨租的運作模式已徹底改變大眾對於數位服務的觀念，
以一個現代的資料中心來說;
會有上萬台的實體機器(Physical Machine)、虛擬主機(Virtual Machine 以下簡稱 VM)，
並必須支援不同網路頻寬的需求，在現在網路架構中如何動態的網路配置變得相當重要。

如同一使用者所建 VM 若跨不同的 Physical Machine，
則需要在 Physical Machine 間配置 Virtual Network；
以及不同使用者的 Physical Machine 之間需要區隔不同的 subnet 等，
為此 Virtual Network 技術漸獲重視。

傳統的作法，基本上還是以 802.1Q Virtual Local Area Network(VLAN) 來區分不同的使用者，
但是會產生以下 3 點問題:

1. VLAN 的識別碼僅 12 位元僅能區隔最多 4094 個用戶，無法因應大量需求。
2. Layer 2 switch 所使用的 Spanning Tree Protocol(STP) 也讓可用頻寬受限，無法發揮網路最大的使用率。
3. 同時數量龐大的 VM 也將會造成 Top of Rack Switch (ToR）過量的負荷。

因此既有的 VLAN 方案無法滿足雲端環境所需要的 virtual network 需求，
有鑑於此，VM 透過 virtual switch 間的 tunnel 建立連線成為快速解決的途徑，
相關設定均在 server 端即可完成，底層網路僅為傳遞封包通透使用，無需額外設定。

## Tunneling Protocol
Wiki[1] 上的訂義為:
> 是一個網路協議的載體。使用隧道的原因是在不兼容的網路上傳輸數據，或在不安全網路上提供一個安全路徑。
穿隧協議可能使用數據加密來傳送不安全的負載協議。

也就是說 Tunnel 會把下層(例如 layer3)的 packet 包裝到上一層(例如 layer4)，
以 SSH Tunneling 來說，就是將 IPv4 的封包，包裝到上一層 SSH protocol 中進行傳輸，
從而實現際體網路間的穿透。

很明顯的，
實現的前提就是 sender、receiver 都要有一種能夠解析這種 pakcet 的 kernel module 或是設備才行。


常見的 Tunneling 技術有:

### 1. IP in IP [2] [3] [4]
其實說穿了就是把 IP layer(layer 3) 的 packet 再包一層 IP 的 packet，
看似好像沒什麼學問，也浪費。
但是其實不然，
它的作用其實等於實現一個 layer2 的 bridge，
我們都知道 MAC 是在 layer2 的，根本不需要 IP，
但是 IP in IP 是透過兩端的 router(switch) 串成的一個 Tunnel，
可以把遠端的兩個 virtaul private network 串連。

也可用來當作 IPv6 over IPv4 的 tuneling。

### 2. Generic Routing Encapsulation(GRE) [5] [6]
和 IP in IP 類似，只是功能更強了一些，還支援 broadcast。

## 新一代的 Tunneling 技術有以下的優點
1. Ability to manage overlapping addresses between multiple tenants
2. Decoupling of the virtual topology provided by the tunnels from the physical topology of the network
3. Support for virtual machine mobility independent of the physical network
4. Support for essentially unlimited numbers of virtual networks (in contrast to VLANs, for example)
5. Decoupling of the network service provided to servers from the technology used in the physical network (e.g. providing an L2 service over an L3 fabric)
6. Isolating the physical network from the addressing of the virtual networks, thus avoiding issues such as MAC table size in physical switches. 

Switch-Switch Tunnel 架構 下來我們要介紹的 tunnel 技術 VXLAN、NVGRE、STT 都屬於 Switch-Switch Tunnel 的架構， 好處就是 VM 與 VM 溝通時，是不會知道傳輸的路徑中間如何連接的， 使系統管理者只需專注在中間這層的 Tunnel End Point 如何設定就行了。

![2013_11_25_2.png]({{site.baseurl}}/assets/images/blog/2013_11_25_2.png)

接下來會一一依介紹幾個的 tunneling 技術
[Generic Routing Encapsulation(GRE)](http://levichen.github.io/2013/11/26/generic-routing-encapsulation/)
[Virtual eXtensible Local Area Network(VXLAN)](http://levichen.github.io/2013/11/26/virtual-extensible-local-area-network/)
[Network Virtualization using Generic Routing Encapsulation(NVGRE)](http://levichen.github.io/2013/12/08/network-virtualization-using-generic-routing-encapsulation/)
[Stateless Transport Tunneling Protocol[STT]](http://levichen.github.io/2013/12/09/stateless-transport-tunneling-protocol/)


## Ref.
1. [Wiki - Tunneling Protocol](https://zh.wikipedia.org/wiki/%E9%9A%A7%E9%81%93%E5%8D%8F%E8%AE%AE)
2. [Wiki - IP in IP](https://en.wikipedia.org/wiki/IP_in_IP)
3. [Wiki - IP Tunnel](https://en.wikipedia.org/wiki/IP_tunnel)
4. [Wiki - Generic Routing Encapsulation](https://en.wikipedia.org/wiki/GRE_(disambiguation))