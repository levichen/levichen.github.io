---
layout: post
published: true
title: "Syslog-ng: Client configuration"
tags: 
  - Log
---

- centos in /etc/syslog.conf end insert
- ubuntu in /etc/rsyslog.conf end insert

## setting '/etc/syslong.conf

```
[root@client ~]$ vim /etc/syslog.conf
*.*   @server_ip
```

## use logger testing

```
[root@client ~]$ logger -i just one test
[root@client ~]$ tail -1 /var/log/messages
Jan 27 22:12:02 client rot[2861]: just one test
```

## check syslog-ng server

```
[root@server2 ~]$ cat /var/log/syslog-ng/20100128/ <server ip> /messages
Jan 28 04:24:32 <server ip> root[2861]: just one test
```
