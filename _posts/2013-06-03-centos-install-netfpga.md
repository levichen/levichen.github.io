---
layout: post
published: true
title: Centos install NetFPGA
tags: 
  - Network
  - NetFPGA
---

## Install NetFPGA

```
$ yum check-update
$ rpm -Uhv http://netfpga.org/yum/el5/RPMS/noarch/netfpga-repo-1-1_CentOS5.noarch.rpm 
$ yum install netfpga-base
$ /usr/local/netfpga/scripts/user_account_setup/user_account_setup.pl
$ reboot
$ cd ~/netfpga/
$ make
$ make install
$ reboot
$ lsmod | grep nf2
$ ifconfig -a | grep nf2
```

## Install PRMForge for required packages

```
$ wget http://apt.sw.be/redhat/el5/en/i386/rpmforge/RPMS/rpmforge-release-0.3.6-1.el5.rf.i386.rpm
$ sudo rpm --import http://apt.sw.be/RPM-GPG-KEY.dag.txt
$ sudo rpm -Uhv rpmforge-release-0.3.6-1.el5.rf.i386.rpm
```

## Install OpenFlow

```
$ yum -y install git automake pkgconfig libtool gcc
$ wget http://ftp.gnu.org/gnu/autoconf/autoconf-2.69.tar.gz
$ tar xvzf autoconf-2.69.tar.gz
$ cd autoconf-2.69
$ ./configure --prefix=/usr
$ make
$ make install
$ cd
$ git clone git://openflow.org/openflow.git
$ cd openflow
$ git checkout -b 1.0.0-netfpga origin/devel/tyabe/1.0.0-netfpga
$ ./boot.sh
$ cd (your openflow directory)
$ ./configure --enable-hw-lib=nf2
$ make
$ make install
```

## of_start_scdemo.sh

```
#!/bin/bash
dpid=90e6babbf72f #12 digit in Hex; Usually use OF SW's MAC address
controller_ip=140.116.164.180
ofdatapath punix:/var/run/dp0 -D -d $dpid -i nf2c0,nf2c1,nf2c2,nf2c3
sleep 5
ofprotocol unix:/var/run/dp0 tcp:$controller_ip --out-of-band -v  &
exit 1
```

## of_stop.sh

```
#!/bin/bash
killall ofprotocol
killall ofdatapath
```

## Start

```
$ chmod +x of_start_scdemo.sh
$ chmod +x of_stop.sh
$ ./of_start_scdemo.sh
```


## Ref.
1. [OpenFlow.org](http://archive.openflow.org/wk/index.php/CentOS_NetFPGA_Install)