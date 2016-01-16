---
layout: post
published: true
title: "Syslog-ng: Store syslog to MySQL"
tags: 
  - Log
---

## Install Package

```
[root@server2 ~]$ yum install php php-mysql mysql mysql-server mysql-devel
```

## setting iptables

```
[root@server2 ~]$ vim /etc/sysconfig/iptables
-A RH-Firewall-1-INPUT -p udp -m udp --dport 514 -j ACCEPT
-A RH-Firewall-1-INPUT -p tcp -m tcp --dport 514 -j ACCEPT
```

```
[root@server2 ~]$ ln -s /usr/lib64/mysql/ /usr/local/lib/mysql
[root@server2 ~]$ ln -s /usr/include/mysql/ /usr/local/include/
[root@server2 ~]$ cd /usr/local/src/software/sqlsyslogd
```

## download syslogd packge

```
[root@server2 software]$ wget -d -r -np http://www.frasunek.com/sources/security/sqlsyslogd/
[root@server2 software]$ cd www.frasunek.com/sources/security/sqlsyslogd/
[root@server2 sqlsyslogd]$ rm -rf index.html*
[root@server2 sqlsyslogd]$ cd contrib/
[root@server2 contrib]$ rm -rf index.html*
[root@server2 contrib]$ cd
[root@server2 ~]$ mv/usr/local/src/software/www.frasunek.com/sources/security/sqlsyslogd/ /usr/
local/src/software/
```

## make syslog package

```
[root@server2 ~]$ cd /usr/local/src/software/sqlsyslogd/
[root@server2 sqlsyslogd]$ make
cc -O6 -Wall -pipe -I/usr/local/include-DCONF=\"/usr/local/etc/sqlsyslogd.conf\" -L/usr/local/lib/mysql-lmysqlclient sqlsyslogd.c   -o sqlsyslogd
[root@server2 sqlsyslogd]$ cp sqlsyslogd /usr/local/sbin/
```

## exec syslog

```
[root@server2 sqlsyslogd]$ sqlsyslogd 
usage: sqlsyslogd [-h hostname] <-u username> [-p] <-t table>[database]
```

修改 /etc/ld.so.conf，並使其生效，這個文件維護著編譯動態連接函式庫的位置

```
[root@server2sqlsyslogd]$ vim /etc/ld.so.conf
include ld.so.conf.d/*.conf
/usr/local/lib/mysql
[root@server2 sqlsyslogd]$ ldconfig
```

在 mysql 中，創建相對應的資料表

```
mysql> create database syslog;
Query OK, 1 row affected (0.00 sec)

mysql> use syslog
Database changed
mysql> create table logs (Id int(10) NOT NULL auto_increment,Timestampvarchar(16),Host varchar(50),Prog varchar(50),Mesg text,PRIMARY KEY (id));
Query OK, 0 rows affected (0.01 sec)

mysql> exit
Bye
```

該文件定義，mysql 的密碼

```
[root@server2sqlsyslogd]$ cat /usr/local/etc/sqlsyslogd.conf 
123456
```

在 syslog-ng.conf 增加下列這幾行

```
[root@server2sqlsyslogd]$ vim /usr/local/syslog-ng/etc/syslog-ng.conf

destination sqlsyslogd{
 program("/usr/local/sbin/sqlsyslogd -u root -t logs syslog -p");
};

log {
        source(s_remote);
        destination(sqlsyslogd);
};
```

```
[root@server2 sqlsyslogd]$ /etc/init.d/syslog-ng restart
Stopping KernelLogger:                                   [ OK ]
Starting KernelLogger:                                   [ OK ]
```