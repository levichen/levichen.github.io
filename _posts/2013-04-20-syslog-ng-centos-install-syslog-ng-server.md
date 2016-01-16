---
layout: post
published: true
title: "Syslog-ng: Centos install syslog-ng server"
tags: 
  - Log
---


- Environment: centos 5.9
- server IP:140.133.76.222
- client IP:140.133.76.212

## Create dir and install pkgconfig

```
[root@server2 ~]$ mkdir -p /usr/local/src/tarbag/
[root@server2 ~]$ mkdir -p /usr/local/src/software/
[root@server2 ~]$ yum install pkgconfig 
```

Check you system already install glib, if not install it.

## Install glib

```
[root@server2 ~]$ cd /usr/local/src/tarbag 
[root@server2 tarbag]$ wget ftp://ftp.gtk.org/pub/glib/2.10/glib-2.10.1.tar.gz
[root@server2 tarbag]$ tar -zxvf glib-2.10.1.tar.gz -C ../software/ 
[root@server2 tarbag]$ cd ../software/glib-2.10.1/ 
[root@server2 glib-2.10.1]$ ./configure --prefix=/usr/local/glib && make && make install
```

## Install eventlog

```
[root@server2 ~]$ cd /usr/local/src/tarbag/
[root@server2 tarbag]$ wget http://www.balabit.com/downloads/files/eventlog/0.2/eventlog_0.2.9.tar.gz
[root@server2 tarbag]$ tar -zxvf eventlog_0.2.9.tar.gz -C ../software/
[root@server2 tarbag]$ cd ../software/eventlog-0.2.9/
[root@server2 eventlog-0.2.9]$ ./configure  --prefix=/usr/local/eventlog && make && make install
[root@server2 eventlog-0.2.9]$ ls /usr/local/eventlog/
include   lib
```

## Install libol

```
[root@server2 syslog-ng-3.0.5]$ cd /usr/local/src/tarbag
[root@server2 tarbag]$ wget http://www.balabit.com/downloads/files/libol/0.3/libol-0.3.9.tar.gz
[root@server2 tarbag]$ tar -zxvf libol-0.3.9.tar.gz -C ../software/
[root@server2 tarbag]$ cd ../software/libol-0.3.9/
[root@server2 libol-0.3.9]$ ./configure --prefix=/usr/local/libol &&make && make install
[root@server2 libol-0.3.9]$ ls /usr/local/libol/
bin   include   lib
```

## Install syslog

```
[root@server2 tarbag]$ wget http://www.balabit.com/downloads/files/syslog-ng/sources/3.0.5/source/syslog-ng_3.0.5.tar.gz
[root@server2 tarbag]$ tar -zxvf syslog-ng_3.0.5.tar.gz -C ../software/
[root@server2 tarbag]$ cd ../software/syslog-ng-3.0.5/
[root@server2 syslog-ng-3.0.5]$ export LD_LIBRARY_PATH=/usr//local/glib/lib/
[root@server2 syslog-ng-3.0.5]$ export PKG_CONFIG_PATH=/usr/local/enventlog/lib/pkgconfig/:/usr/local/glib/lib/pkgconfig/
[root@server2 syslog-ng-3.0.5]$ ./configure  --prefix=/usr/local/syslog-ng --with-libol=/usr/local/libol &&  make && make install
[root@server2 syslog-ng-3.0.5]$ ls /usr/local/syslog-ng/
bin   libexec   sbin   share
```

```
[root@server2 syslog-ng-3.0.5]$ mkdir /usr/local/syslog-ng/etc
[root@server2 syslog-ng-3.0.5]$ mkdir /usr/local/syslog-ng/var
[root@server2 syslog-ng-3.0.5]$ cp contrib/syslog-ng.conf.RedHat  /usr/local/syslog-ng/etc/
[root@server2 syslog-ng-3.0.5]$ cp contrib/init.d.RedHat /etc/init.d/syslog-ng
[root@server2 syslog-ng-3.0.5]$ cd /usr/local/syslog-ng/etc/
[root@server2 etc]$ mv syslog-ng.conf.RedHat syslog-ng.conf
```

## setting '/etc/init.d/syslog-ng'

```
[root@server2 etc]$ chmod +x /etc/init.d/syslog-ng 
[root@server2 etc]$ vim /etc/init.d/syslog-ng 
 #!/bin/bash
 #chkconfig: --add syslog-ng
 #chkconfig: 2345 12 88
 #Description: syslog-ng

 # Full path to daemon
 INIT_PROG="/usr/local/syslog-ng/sbin/syslog-ng"    
 # options passed to daemon
 INIT_OPTS="-f /usr/local/syslog-ng/etc/syslog-ng.conf"                      

 PATH=/bin:/sbin:/usr/bin:/usr/sbin:/usr/local/syslog-ng/bin:/usr/local/syslog-ng/sbin

 INIT_NAME=`basename "$INIT_PROG"`

 # Source Redhat function library.
 #
 . /etc/rc.d/init.d/functions

 # Uncomment this if you are on Redhat and think this is useful
 #
 #. /etc/sysconfig/network
 #
 #if [ ${NETWORKING} = "no" ]
 #then
 #       exit 0
 #fi

 RETVAL=0

 umask 077
 ulimit -c 0

 # See how we were called.
 case "$1" in
   start)
 echo -n "Starting $INIT_NAME: "
 daemon --check $INIT_PROG "$INIT_PROG $INIT_OPTS"
 RETVAL=$?
 echo -n "Starting Kernel Logger: "
 [ -x "/sbin/klogd" ] && daemon klogd
 echo
 [ $RETVAL -eq 0 ] && touch "/var/lock/subsys/${INIT_NAME}"
 ;;
   stop)
 echo -n "Stopping $INIT_NAME: "
 killproc $INIT_PROG
 RETVAL=$?
 echo -n "Stopping Kernel Logger: "
 [ -x "/sbin/klogd" ] && killproc klogd
 echo
 [ $RETVAL -eq 0 ] && rm -f "/var/lock/subsys/${INIT_NAME}"
 ;;
   status)
 status $INIT_PROG
 RETVAL=$?
 ;;
   restart|reload)
 $0 stop
 $0 start
 RETVAL=$?
 ;;
   *)
 echo "Usage: $0 {start|stop|status|restart|reload}"
 exit 1
 esac
 exit $RETVAL

 
-- set eventlog lib ln -- 
[root@server2 etc]$ ln -s /usr/local/eventlog/lib/* /lib/  
[root@server2 etc]$ ln -s /usr/local/eventlog/lib/* /lib64/
```

## setting '/usr/local/syslog-ng/etc/syslog-ng.conf'

```
@version:3.0
 options {
 long_hostnames(off);
 log_msg_size(8192);
 flush_lines(1);
 log_fifo_size(20480);
 time_reopen(10);
 use_dns(yes);
 dns_cache(yes);
 use_fqdn(yes);
 keep_hostname(yes);
 chain_hostnames(no);
 perm(0644);
 stats_freq(43200);
 };
 source s_internal { internal(); };
 destination d_syslognglog { file("/var/log/syslog-ng.log"); };
 log { source(s_internal); destination(d_syslognglog); };

 source s_local {
 unix-dgram("/dev/log");
 file("/proc/kmsg" program_override("kernel:"));
 };

 filter f_messages { level(info..emerg); };
 filter f_secure   { facility(authpriv); };
 filter f_mail     { facility(mail); };
 filter f_cron     { facility(cron); };
 filter f_emerg    { level(emerg); };
 filter f_spooler  { level(crit..emerg) and facility(uucp, news); };
 filter f_local7   { facility(local7); };

 destination d_messages { file("/var/log/messages"); };
 destination d_secure   { file("/var/log/secure"); };
 destination d_maillog  { file("/var/log/maillog"); };
 destination d_cron     { file("/var/log/cron"); };
 destination d_console  { usertty("root"); };
 destination d_spooler  { file("/var/log/spooler"); };
 destination d_bootlog  { file("/var/log/demsg"); };

 log {source(s_local); filter(f_emerg);  destination(d_console); };
 log {source(s_local); filter(f_secure); destination(d_secure); flags(final); };
 log {source(s_local); filter(f_mail);   destination(d_maillog); flags(final); };
 log {source(s_local); filter(f_cron);   destination(d_cron); flags(final); };
 log {source(s_local); filter(f_spooler); destination(d_spooler); };
 log {source(s_local); filter(f_local7); destination(d_bootlog); };
 log {source(s_local); filter(f_messages); destination(d_messages); };

 source s_remote {
 tcp(ip(0.0.0.0) port(514));
 udp(ip(0.0.0.0) port(514));
 };

 destination r_console {file("/var/log/syslog-ng/$YEAR$MONTH$DAY/$HOST/console" owner("root") group("root") perm(0640) dir_perm(0750) create_dirs(yes)); };

 destination r_secure {file("/var/log/syslog-ng/$YEAR$MONTH$DAY/$HOST/secure" owner("root") group("root") perm(0640) dir_perm(0750) create_dirs(yes)); };

 destination r_cron {file("/var/log/syslog-ng/$YEAR$MONTH$DAY/$HOST/cron" owner("root") group("root") perm(0640) dir_perm(0750) create_dirs(yes)); };

 destination r_spooler {file("/var/log/syslog-ng/$YEAR$MONTH$DAY/$HOST/spooler" owner("root") group("root") perm(0640) dir_perm(0750) create_dirs(yes)); };

 destination r_bootlog {file("/var/log/syslog-ng/$YEAR$MONTH$DAY/$HOST/bootlog" owner("root") group("root") perm(0640) dir_perm(0750) create_dirs(yes)); };

 destination r_messages {file("/var/log/syslog-ng/$YEAR$MONTH$DAY/$HOST/messages" owner("root") group("root") perm(0640) dir_perm(0750) create_dirs(yes)); };

 log { source(s_remote); filter(f_emerg); destination(r_console); };
 log { source(s_remote); filter(f_secure); destination(r_secure); flags(final); };
 log { source(s_remote); filter(f_cron); destination(r_cron); flags(final); };
 log { source(s_remote); filter(f_spooler); destination(r_spooler); };
 log { source(s_remote); filter(f_local7); destination(r_bootlog); };
 log { source(s_remote); filter(f_messages); destination(r_messages); };
```

## start syslong-ng

```
[root@server2 ~]$ /etc/init.d/syslog-ng start
Starting KernelLogger:                                    [   OK   ]

[root@server2 etc]$ cat /var/log/syslog-ng.log 
Oct 15 11:29:47 localhost.localdomain syslog-ng[28022]: syslog-ng starting up; version='3.0.5'
```

## Ref.
1. http://hi.baidu.com/qingchunranzhi/item/a7d1ab98371a31b5cc80e57b
2. http://wenku.baidu.com/view/c3bb49c58bd63186bcebbc7a.html
3. http://blog.csdn.net/mandyliu301/article/details/8349239
4. http://blog.csdn.net/xiangliangyu2008/article/details/8072392
5. http://blog.sina.com.cn/s/blog_4a071ed80100cssu.html