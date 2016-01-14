---
layout: post
published: true
title: "Kernel-based Virtual Machine"
tags: 
  - Visualization
---

## Introduction
KVM( 全名為 Kernel-based Virtual Machine )，
自 2006 年 12 月起，就屬於 Linux 核心架構下的一部分，
簡單來說，就是以後 Linux 系統核心，就具備支援虛擬化的功能，
並且是以核心模組化的方式載入執行，
只要硬體搭配得宜（此指 CPU 支援虛擬化技術，也就是 lntel VT 或 AMD-V ），
再安裝 KVM 這一套虛擬系統，就可以直接在現有的 Linux 系統架構之下，
建構裸機式（Bare-Metal ）虛擬系統平台。

## Install KVM on Ubuntu-12.04
1.先檢查你的 CPU 是否支援虛擬化

```
[root@ubuntu]$  egrep -c '(vmx|svm)' /proc/cpuinfo
#大於 0 的話代表支援
```

2.檢查主機是否支援 KVM

```
[root@ubuntu]$ apt-get install cpu-checker
[root@ubuntu]$ kvm-ok 
INFO: /dev/kvm does not exist
HINT:   sudo modprobe kvm_intel
INFO: Your CPU supports KVM extensions # 有看到這行表示OK

KVM acceleration can be used
```

3.開始安裝

```
[root@ubuntu]$ apt-get install virtinst
[root@ubuntu]$ apt-get install ubuntu-vm-builder
[root@ubuntu]$ apt-get install virt-viewer
[root@ubuntu]$ apt-get install qemu-kvm libvrit-bin
[root@ubuntu]$ apt-get install bridge-utils  #bridge model

[root@ubuntu]$ apt-get install libcap2-bin
[root@ubuntu]$ apt-get install virt-manager  #圖型化 vm 管理介面

[root@ubuntu]$ setcap cap_net_admin=ei /usr/bin/qemu-system-x86_64
```

## KVM 目前有三套好用的管理工具
- virt-manager :一個圖形介面的管理工具，可以安裝在有 X window 的 linux 機器上。
- virt-install: 一個用 python 撰寫的文字介面管理工具，Red Hat 開發。
- ubuntu-vm-builder:文字介面管理工具，Canonical 開發。

## Virt-manager
全名為 Virtual Machine Manager，可以在 x window 的環境中，操控 VM
一開始，要把你所登入的使用者，加入 libvirtd 的 group 裡面
這樣，你所登入的使用者，才有權限操作 Virtual Machine Manager
例如我想要讓 mrblack 有操作 Virtual Machine Manager 的權限

```
[root@ubuntu]$ adduser mrblack libvirtd
[root@ubuntu]$ /etc/init.d/libvirt-bin reload   #重新載入設定
```

就可以利用 virt-manager 來管理 VM

![kvm_1.png]({{site.baseurl}}/assets/images/blog/kvm_1.png)

## Virsh 文字管理工具
virsh 是一套用於虛擬化管理的工具，本身是由 libvirt 實作而來。
是一種跨平台(支援 XEN、KVM 等虛擬化系統)的虛擬化管理 API。
關於它的操作，可以看 libvirt 官網 有詳細的介紹。

## Ref.
- [WebModelling](http://webmodelling.com/webbits/ubuntu/ubuntu-virtualization.aspx)
- [ubuntu documentation](https://help.ubuntu.com/community/KVM)
- [指令參考 Gavin's HOME](http://chengavin.blogspot.tw/2011/04/kvm.html)

