---
layout: post
published: false
title: Configure Open vSwitch on Proxmox Virtual Environment
tags: 
  - Proxmox VE
  - Visualization
---

因為研究的關係，會需要在 Proxmox Virtual Environment(PVE) 上安裝 Open vSwitch(OVS)
由於 PVE 把 Linux 原生的 bridge 內鍵在 kernel 中了，
所以無法用 OVS 來取代 Linux 的 bridge 造成 PVE 中的 Kernel Virtual Machine(KVM) 無法使用 OVS。
因此，要此重心 make PVE 的 kernel 後，把 bridge 的設為模組(module)，
好讓我們在安裝 OVS 的時候，可以取代它。

## 環境
* Proxmox VE 2.3
* Open vSwitch 1.4.0 


## 1.安裝必要的套件

```
[root@www ~]# apt-get install -y python-simplejson python-qt4 python-twisted-conch autoconf gcc uml-utilities libtool pkg-config automake build-essential openssl libssl-dev bridge-utils rpm2cpio lintian libncurses5-dev unzip 
```


## 2.利用 git clone PVE 的 Kernel
抓回來後放置在 /usr/src 資料夾內，這邊我們抓的是 pve-kernel-2.6.32 的版本。

```
[root@www ~]# git clone git://git.proxmox.com/git/pve-kernel-2.6.32.git pve-kernel
[root@www ~]# mv pve-kernel /usr/src
```

## 3.取得 PVE 的 source code
在剛剛的 pve-kernel 資料夾內下 make 它就會自動把 source code 抓回來囉
抓回來的 source code 會放置在 /usr/src/pve-kernel/linux-2.6-2.6.32

```
[root@www ~]# cd /usr/src/pve-kernel
[root@www pve-kernel]# make
```

## 4.清除暫存檔
先切換進去 linux-2.6-2.6.32 資料夾內，先清除暫存檔後，就可以開始 make kernel 了

```
# 把 source code 中的目標檔刪除

[root@www linux2.6.2-6.32]# make mrproper

# 清除暫存檔

[root@www linux2.6.2-6.32]# make clean
[root@www linux2.6.2-6.32]# make menuconfig
```

## 5.Linux Kernel Configuration
然後就會開始出現藍底白字畫面了，如下：

![pve1.png]({{site.baseurl}}/assets/images/blog/pve1.png)

先在 General setup ->> Local version – append to kernel release 設定你這次編譯的名稱

![pve2.png]({{site.baseurl}}/assets/images/blog/pve2.png)

Networking support -> Networking options -> 把 802.1d Ethernet Bridging * 改為 M

![pve3.png]({{site.baseurl}}/assets/images/blog/pve3.png)

![pve4.png]({{site.baseurl}}/assets/images/blog/pve4.png)

![pve5.png]({{site.baseurl}}/assets/images/blog/pve5.png)

最後，按下 Save an Alternate Configuration File
保留 config 檔，並離開 Linux Kernel Configuration

![pve6.png]({{site.baseurl}}/assets/images/blog/pve6.png)


## 6.Make Kernel

```
#先編譯核心
[root@www linux-2.6-2.6.32]# make bzImage

#再編譯模組
[root@www linux-2.6-2.6.32]# make modules

#實際安裝模組
[root@www linux-2.6-2.6.32]# make modules_install
```

## 7.設定 GRUB 多重核心選單
由於害怕剛剛編譯完的核心不能正常的開機，所以通常都是手動來設定成多重開機的選單，
好讓我們如果發生錯誤的設定，可以直接再次開機來進入 Linux 系統。

可以先看一下 /lib/modules/ 資料夾下確定我們載的是哪個 kernel 的版本

```
[root@www /boot]# ll /lib/modules/
2.6.32-26-pve/     2.6.32-19-pvebridge/
```

可以確定的 2.6.32-19-pvebridge 就是我們剛剛 make 好的 kernel modules
設複製剛剛 make 好的 bzImage 如 config 檔到 /boot 資料夾下

```
[root@www /boot]# cp /usr/src/pve-kernel/linux-2.6-2.6.32/arch/x86_64/boot/bzImage  /boot/vmlinuz-2.6.32-19-pvebridge
[root@www /boot]# cp /usr/src/pve-kernel/linux-2.6-2.6.32/.config /boot/config-2.6.32-19-pvebridge
```

## 8.建立相對應的 Initial Ram Disk(initrd)

```
[root@www /boot]# mkinitramfs -o /boot/initrd.img-2.6.32-19-pvebridge 2.6.32-19-pvebridge
```

## 9.編輯 GRUG 開機多重選單
在 GRUB 選單內加入我們這個版本的 Kernel，
可以參考一下列的寫法。

```
in /boot/grub/grub.cfg 

menuentry 'Proxmox Virtual Environment GNU/Linux, with Linux 2.6.32-19-pvebridge' --class proxmox --class gnu-linux --class gnu --class os {
    insmod part_msdos
    insmod ext2
    set root='(hd0,msdos1)'
    search --no-floppy --fs-uuid --set 9cc709ff-7275-461e-8e91-505d0b1c158f
    echo 'Loading Linux 2.6.32-19-pvebridge ...'
    linux /vmlinuz-2.6.32-19-pvebridge root=/dev/mapper/pve-root ro quiet
    echo 'Loading initial ramdisk ...'
    initrd /initrd.img-2.6.32-19-pvebridge
}
```

## 10.最後
再來你就可以重開機了，
重啟之後利 uname -r 查看你的 Kernel 版本是不是你剛剛編的

Open vSwitch 的部份，請參考 [Install and Configuring Open vSwitch on Ubuntu 12.04](http://levichen.github.io/2013/12/15/install-and-configuring-open-vswitch-on-ubuntu-12-04/)



