---
layout: post
published: true
title: System Log
tags: 
  - Log
---

## syslog 簡介
在 linux 的執行的背景中，有相當多的 daemon 在同時執行著

一些重要的服務萬一爛掉?或者想要知道 who? where? when? do what? 在使用這個 service

就是將這些重要的資訊記錄下來，讓系統的管理者可以更容易、清楚的控管自己的系統

而這些 syslog 產生的方式分有兩種

由軟體開發商自行定義寫入的 syslog 與相關格式，例如 apache 等等..

由 linux 提供的 syslog 服務統一管理，只要將這個訊息丟給這個服務後

它會自行幫你寫入到你指定的地方，而 CentOS 提供 syslogd 這個服務來統一管理

而除了 syslog 外，核心也利用 klogd 這個服務來記錄，也就是說 登錄檔所需要服務主要就是 syslogd 與 klogd

也因為服務很多，所以 log 的數量、容量是很驚人的，

Linux提供 logrotate (登錄檔輪替) 來自動化處理 log 的容量更新的問題

## syslog 的設定檔 /etc/syslog.conf

syslogd 是負責產生各個 service 的 log，而這些 log 是有「嚴重等級」之分的

but…這些 log 最終要送去哪，就是靠 syslogd 這個 syslog.conf 來設定

像是 sendmail 這個 daemon 就是委託 syslogd 來記錄它程式的 log

![2011_11_25_1.gif]({{site.baseurl}}/assets/images/blog/2011_11_25_1.gif)

## 安全性設置

可以利用 chattr、lsattr[2] 來設定 syslog 只能寫，不能改

以防止一些惡意的修改

這個需要注意的是因為設定成只能寫，不能改、刪

所以系統在做 logrotate 的時候也不行= =

三思而後行…

```
$ chattr +a /var/log/messages
$ lsattr /var/log/messages
  —–a——- /var/log/messages
```

利用下列的指令來取消

```
$ chattr -a /var/log/messages
```

## syslog server client set

重頭戲就是這個啦!!!

一開始就是想要利用這種方式，把所有的 log 集中在一台 server

然後可以利用一些軟體 如 graylog2 ，或是自己寫的分析系統來分析多台 server 的 log

在 server 的部分需要將 udp 的 port 514 打開監聽(設定檔、iptables)

![2011_11_25_2.gif]({{site.baseurl}}/assets/images/blog/2011_11_25_2.gif)

```
# /etc/sysconfig/iptables

# 加入以下這行，對 port 514 放行  
  -A RH-Firewall-1-INPUT -m state –state NEW -m udp -p udp –dport 514 -j ACCEPT
```

```
# /etc/sysconfig/syslog

# 修改這行~  
  SYSLOGD_OPTIONS="-m 0 -r"  
```

然後再將 syslog 重啟， server 的部分就差不多囉~

再來是 client，設定剛剛說的 /etc/syslog.conf 檔案

```
# /etc/syslog.conf

# 加入以下這行，將所有的 service log 傳送到 你的 server IP  
*.* @your server ip
```

## 設定 apache log 導入 syslog server
就如同剛剛說的

apache 需要自己設定 log 的存法[3]

```
# /etc/httpd/conf/httpd.conf

# error log 的部分就是這樣修改，將 apache 的 log 委託給 syslogd 處理  
# 然後 syslogd 又會將這個 log 導入到剛剛的 syslog server~~  
ErrorLog syslog  
  
# access 的 log 就是這樣設定，觀念跟剛剛是一樣的  
CustomLog |/usr/local/apache/bin/apache_syslog combined
```

## Ref.
1.[鳥哥的私房菜 – syslog](http://linux.vbird.org/linux_basic/0570syslog.php)
2.[鳥哥的私房菜 – chattr、lsattr](http://linux.vbird.org/linux_basic/0220filemanager.php#chattr)
3.[Apache log 設定方法](http://linux.vbird.org/linux_basic/0220filemanager.php#chattr)
