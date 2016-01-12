---
layout: post
published: true
title: Install and Configuring Open vSwitch on Ubuntu 12.04
tags: 
  - "Visualization - Open vSwitch"
---

## 1. 更新 apt-get 套件，並且安裝 Open vSwitch 需求的套件
```
[root@ubuntu ~] # apt-get update
[root@ubuntu ~] # apt-get install -y git python-simplejson python-qt4 python-twisted-conch autoconf gcc uml-utilities libtool pkg-config automake build-essential openssl libssl-dev bridge-utils
```

## 2. 先把 network-manager 關閉
請先確保你是在本地端登錄，然後把 network-manager 關閉，
不然在之後的節驟可能會出現錯誤
```
[root@ubuntu ~]# /etc/init.d/network-manager stop
```

## 3. 下載 OVS tarball
```
[root@ubuntu ~] # wget http://openvswitch.org/releases/openvswitch-1.4.0.tar.gz
[root@ubuntu ~] # tar zxvf openvswitch-1.4.0.tar.gz
[root@ubuntu ~] # mv openvswitch-1.4.0 openvsiwtch
[root@ubuntu ~] # cd openvsiwtch
```

## 4. 註解重覆的程式碼
因為這個版本的某些 code 跟原生的 linux code 有衝突到
所以在這邊，請接以下這些標頭檔的 function 注解掉

修改 datapath/linux/compat/include/linux/skbuff.h 的函式 skb_frag_page 跟 skb_reset_mac_len
修改 datapath/linux/compat/include/linux/if.h 的變數 IFF_OVS_DATAPATH
修改 datapath/linux/compat/include/linux/if_vlan.h 的函式 vlan_set_encap_proto

## 5. 編譯 Open vSwitch
```
[root@ubuntu openvswitch] # ./boot.sh
[root@ubuntu openvsiwtch] # ./configure --with-linux=/lib/modules/`uname -r`/build
[root@ubuntu openvsiwtch] # make && make install
```

## 6. 載入 OVS module
```
# bridge 不先關的話可能會造成 ovs insmod failed
[root@ubuntu openvsiwtch] # lsmod | grep bridge
[root@ubuntu openvsiwtch] # rmmod bridge

#load kernel module
[root@ubuntu openvsiwtch] # insmod datapath/linux/openvswitch.ko
[root@ubuntu openvsiwtch] # insmod datapath/linux/brcompat_mod.ko  #可以讓 ovs 取代 linux bridge 的模組
```

## 7. 設定相關參數
```
[root@ubuntu openvsiwtch] # mkdir -p /usr/local/etc/openvswitch

[root@ubuntu openvsiwtch] # ovsdb-tool create /usr/local/etc/openvswitch/conf.db vswitchd/vswitch.ovsschema

[root@ubuntu openvsiwtch] # ovs-vsctl --no-wait init

[root@ubuntu openvsiwtch] # ovsdb-server /usr/local/etc/openvswitch/conf.db \
--remote=punix:/usr/local/var/run/openvswitch/db.sock \
--remote=db:Open_vSwitch,manager_options \
--private-key=db:SSL,private_key \
--certificate=db:SSL,certificate \
--bootstrap-ca-cert=db:SSL,ca_cert --pidfile --detach

[root@ubuntu openvsiwtch] # ovs-vswitchd unix:/usr/local/var/run/openvswitch/db.sock --detach

[root@ubuntu openvsiwtch] # ovs-brcompatd --pidfile --detach
```

## 8. 利用 ovs-vsctl 查看是否安裝成功
```
[root@ubuntu ~] # ovs-vsctl show
85bbe26c-f2fe-49f6-aa74-e34b026a197f
```

## 9. 開機時，自動載入相關設定
可以先將下列寫成一支 shell，然後在 /etc/rc.local 載入
這樣一來，每次重開機時，就都會載入相同的設定

```
#!/bin/bash

 
PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/bin:/usr/local/sbin:~/bin
export PATH
 
#移除原有模組並新增open vSwitch

rmmod bridge
insmod /root/openvswitch/datapath/linux/openvswitch_mod.ko
insmod /root/openvswitch/datapath/linux/brcompat_mod.ko
 
#啟動

ovsdb-server --remote=punix:/usr/local/var/run/openvswitch/db.sock \

   --remote=db:Open_vSwitch,manager_options \

   --private-key=db:SSL,private_key \

   --certificate=db:SSL,certificate \

   --bootstrap-ca-cert=db:SSL,ca_cert \

   --detach
 
#檢查是否開過

SERVICE='ovs-vswitchd'
 
if ps ax | grep -v grep | grep $SERVICE > /dev/null
then
    echo "$SERVICE service running"
else
    ovs-vswitchd unix:/usr/local/var/run/openvswitch/db.sock --detach
fi
 
ovs-brcompatd --pidfile --detach
exit 0
```