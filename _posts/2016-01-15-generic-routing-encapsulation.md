---
layout: post
published: true
title: Generic Routing Encapsulation
tags: 
  - Tunneling
  - Network
---

## 簡介
Generic Routing Encapsulation(GRE)
是一個由 Cisco 開發的點對點無狀態通道協定(point to point stateless tunneling protocol)，
實際運作會將內容封裝到 Layer 3 的 IP 封包中傳送到遠端路由器後解開還原。

## GRE header format

![2013_11_26_1.png]({{site.baseurl}}/assets/images/blog/2013_11_26_1.png)

如上圖所示經過 GRE 封包的封包會多上一層 GRE 標頭(header)，當封包傳送到目的路由器後則進行解封裝，此協定本身沒有加密的功能，但是能搭配 IPSec 來進行安全傳輸，且 GRE 支援多點群播(multicast)和 IPv6。

![2013_11_26_2.png]({{site.baseurl}}/assets/images/blog/2013_11_26_2.png)

GRE 封包結構如上圖，在 GRE Header 裡面最重要的欄位是 Key，如果 GRE 通道的兩端分別是路由器，則辨別兩端 GRE 通道是否能夠順利通訊的關鍵則是此欄位(Key)要設為相同。

C、K、S:分別代表 checksum、key、 sequence number 的 Bit flages
Ver: GRE 的版本，目前是第 0 版
Protocol: Ethertype of the encapsulated protocol
Checumsum: Packet checksum(optional)
Key:Tunnel Key(optional)
Sequence Number: GRE sequence number(optional)

## GRE Tunnel architectural
下圖就是兩台 switch 間建起 gre tunnel 連線後，
不同 domain 的 client 就可以互相溝通了。

與 IP in IP 不同的是，
Gre Tunnel encapsulation、decapsulation Header 是由中間的 middlebox(swtich 或是 route)來做的。

![2013_11_26_3.png]({{site.baseurl}}/assets/images/blog/2013_11_26_3.png)

如果用 Wireshark 打開封包來看的話
如下圖:

![2013_11_26_4.png]({{site.baseurl}}/assets/images/blog/2013_11_26_4.png)

最下層就是 client 的 srouce、destination IP
中間包了 GRE 的 Header 後
上層就是兩台 switch 的 srouce、destination IP 了

## GRE tunnel 的缺點

如果要兩個 host 建立 GRE tunnel 連線的話，會需要一條 GRE tunnel
如果要將三個點連線的話呢?
會需要三條 GRE tunnel 才能連將三個 host 串連，如下圖:

![2013_11_26_5.png]({{site.baseurl}}/assets/images/blog/2013_11_26_5.png)

這樣就很容易看出來，
GRE tunnel 的 scalability 太差，
從根本上來說，他只是一個 Point to Point 的 tunnel，
所以在之後的 VXLAN 等等新的 Tunneling 也同時解決了這個問題。


## Ref.
- [Packet Life.net - GRE vs IPIP Tunneling](http://packetlife.net/blog/2012/feb/27/gre-vs-ipip-tunneling/)


