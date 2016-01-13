---
layout: post
published: true
title: "Unicast & Broadcast & Multicast"
tags: 
  - Network
---

一直都搞不太清楚這三者通訊方式的差別，
所以在此記錄一下。

以下會利用下列這個網路拓璞來說明各種通訊方式:

![gIpjcZkmTaq0LIb4JVBH_network.png]({{site.baseurl}}/assets/images/blog/gIpjcZkmTaq0LIb4JVBH_network.png)

## 1.Unicast(單傳傳播)
通常指的是特定的目的地位址，一般是主機之間互相傳遞封包的方式，也被稱為 One-to-One 的通訊方式。
連結頻寬 = 單點資料量 * 瀏覽者數量

Server 要向每個 Client 分別傳送文件
同樣的文件要傳輸多次，也就是說網路上的流量就會有

4.5 M = 1.5 M * 3

![Lp6f8BVeRIGpLmYi6vHE_unicast.png]({{site.baseurl}}/assets/images/blog/Lp6f8BVeRIGpLmYi6vHE_unicast.png)

## 2.Broadcast(多點廣播)
通常發生在 MutiAccess 的網路媒介中。
在 Layer2 中的 Header destination Mac 通常是 FF:FF:FF:FF:FF:FF
在 Layer3 中的 Header destination IP 通常是 255.255.255.255
連接至通一 LAN 網路媒介上的所有主機級網路設備都會接收到這個封包進行行處
比較常件的就是 Arp 的 Broadcast 的封包了
因此被稱為 One-to-All 的通訊方式。

只需發送一個文件，只要是同一個 LAN 中的 Client 都能收到這個訊息
網路間的流量為 1.5 M

![t6qqOZ1PSvA4eEMqgZfp_brodcast.png]({{site.baseurl}}/assets/images/blog/t6qqOZ1PSvA4eEMqgZfp_brodcast.png)

## 3.Multicast(多播/群播)
一般應用於相同資料來源同時要傳送給一群特定的 Multicast Group Client，
但是 Server 只會發送一份資料，
因此頻寬的使用量不會因為 Client 增加而增加，
因此也被稱為 One-to-Many or Many-to-Many 的通訊方式

傳輸的方式跟 Broadcast 有點相似，只是不是發給網路中的每一個客戶，
而是誰需要就發給誰，如果 3 個 Client 中只有 2 個需要這個文件，
那麼 Multicast 可以滿足要求，只發給其中的 2 個 Client。

以我們的例子來說:
我們的 LAN 中有 3 台 client，但只有 client2、client3 需要文件
那首先 client2、client3 必須對 server 發出 request，
Server 會根據 Request 回應 Client

![HiLF4s3QEqw23F83Bx9w_muticast.png]({{site.baseurl}}/assets/images/blog/HiLF4s3QEqw23F83Bx9w_muticast.png)

## Ref.
1. [漫談Unicast/Broadcast/Multicast](http://ccie11440.blogspot.tw/2008/09/unicastbroadcastmulticast.html)
2. [unicast、broadcast、multicast](http://isen-blog.blogspot.tw/2008/07/unicastbroadcastmulticast.html)