---
layout: post
published: true
title: Syslog 記錄 history
tags: 
  - log
---

現實生活中，有可能伺服器會遭到駭客的攻擊
而大部分的駭客都會把存在系統上的 history 記錄清掉
讓你搞不清楚他做了些什麼

這篇文章主要要記錄，如何透過 syslog 可以備份本地端的 history 到 遠端的 syslog server

## 1.下載 bash

```
$ wget http://ftp.gnu.org/gnu/bash/bash-4.2.tar.gz
$ tar zxvf bash-4.2.tar.gz -C /usr/local/bash-4.2
$ cd /usr/local/bash-4.2
```

## 2.修改 bashhist.c 內的 bash_syslog_history function 約在 708 行

去掉 bash-4.2/config-top.h 中的 define SYSLOG_HISTORY 的注釋

```
#if defined (SYSLOG_HISTORY)
#define SYSLOG_MAXLEN 600

void
bash_syslog_history (line)
     const char *line;
{
  char trunc[SYSLOG_MAXLEN];
  char vmip[20];

  get_local_ip(&vmip);

  if (strlen(line) < SYSLOG_MAXLEN)
    syslog (SYSLOG_FACILITY|SYSLOG_LEVEL, "HISTORY: VMIP=%s PID=%d PPID=%d SID=%d User=%s CMD=%s", vmip, getpid(), getppid(), getsid(getpid()), current_user.user_name, line);
  else
    {
      strncpy (trunc, line, SYSLOG_MAXLEN);
      trunc[SYSLOG_MAXLEN - 1] = '\0';
      syslog (SYSLOG_FACILITY|SYSLOG_LEVEL, "HISTORY (TRUNCATED): VMIP=%s PID=%d PPID=%d SID=%d User=%s CMD=%s", vmip, getpid(), getppid(), getsid(getpid()), current_user.user_name, trunc);
    }
}
#endif
```

## 3.編譯安裝

```
$ ./configure
$ make
$ make install
```

## 4.把 User 的 bash 換成現在安裝的 bash4.1

這樣 log 就會記錄在 /var/log/syslog

```
$ vim /etc/passwd

    dongwm:x:501:501::/home/dongwm:/usr/local/bash_4.1/bin/bash
```

## 5.syslog server 配置(加入 syslog server 的位址)

```
$ vim /etc/rsyslog.conf

    .  @your.syslog.server.ip
```


## Ref.
- http://www.zdh1909.com/html/linux/17097.html
- http://ftp.gnu.org/gnu/bash/


