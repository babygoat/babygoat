---
title: journalctl使用筆記
catalog: true
date: 2019-04-25 00:54:18
subtitle:
header-img:
categories:
- Linux
tags:
- Linux
- Systemd
---

# 前言
最近因為工作上的關係，需要寫一些init scripts在GCE上，在debug時就需要看一些log

因為coreOS(GCE Host OS)是使用Systemd(PID=1)來管理service和daemon，而其中systemd-journald.service就是systemd的logging機制，會紀錄所有來自kernel及user space的logs，做成集中管理的journal，透過journalctl來調控，本篇記錄一些常用的options

# systemd-journald service

## configuration

有關於journal的設定檔，可以在/etc/systemd/journald.conf來調整，目前有使用到比較重要的是storage這個項目

```
~>sudo vim /etc/systemd/journald.conf

...

[Journal]
Storage=persistent #會將journal寫進filesystem (/var/log/jouranl/)
```

預設是volatile，也就是in-memory的journal，只能在/run/log/journal/，重開機後就不見了
如果後面提到的查詢前幾次開機的選項無法正常運作的話，可以檢查一下這個設定

## options

底下筆記幾個常用的options

### journactl -r

從最新的journal檢視，預設是從最早的開始檢視

```
-- Logs begin at Fri 2019-01-25 03:35:16 UTC, end at Wed 2019-04-24 17:25:00 UTC. --
Apr 24 17:25:00 test sudo[175502]: pam_tty_audit(sudo:session): changed status from 1 to 1
Apr 24 17:25:00 test sudo[175502]: pam_unix(sudo:session): session opened for user root by babygoat(uid=0)
Apr 24 17:25:00 test sudo[175502]: babygoat : TTY=pts/0 ; PWD=/etc/systemd ; USER=root ; COMMAND=/usr/bin/jour
Apr 24 17:24:58 test sudo[175498]: pam_unix(sudo:session): session closed for user root
Apr 24 17:24:57 test sudo[175498]: pam_tty_audit(sudo:session): changed status from 1 to 1
Apr 24 17:24:57 test sudo[175498]: pam_unix(sudo:session): session opened for user root by babygoat(uid=0)
Apr 24 17:24:57 test sudo[175498]: babygoat : TTY=pts/0 ; PWD=/etc/systemd ; USER=root ; COMMAND=/
```

### journalctl --list--boots

列出每次開機的journal的清單
其中0代表本次開機，-1代表上次開機，-2代表上上次的開機，以此類推
```
 -2 8a2032168d334de39ffed4390b40459f Thu 2019-04-18 09:24:29 UTC—Thu 2019-04-18
 -1 7d713cc3a68d431bb04effc604d0cefc Thu 2019-04-18 09:33:56 UTC—Thu 2019-04-18
  0 dd92c7ac6d74429fabc483dcde373917 Thu 2019-04-18 09:51:41 UTC—Wed 2019-04-24
```

### journalctl -b [#boot]
列出第幾次開機的journal

e.g., journalctl -b 0 # 本次開機的journal 

### journalctl -u [service]
因為journal的log涵蓋蠻多不同服務，可以利用-u這個選項，指定我們要查詢的服務的log，省去自己grep的麻煩，例如我想查詢cloud-init這個service的服務log

```
~> sudo journalctl -u cloud-init
...
Apr 18 09:51:56 test systemd[1]: Started Initial cloud-init
Apr 18 09:51:55 test cloud-init[481]: [CLOUDINIT] __init__.p
Apr 18 09:51:55 test cloud-init[481]: [CLOUDINIT] __init__.p
Apr 18 09:51:55 test cloud-init[481]: [CLOUDINIT] __init__.p
Apr 18 09:51:55 test cloud-init[481]: [CLOUDINIT] stages.py
```

### journalctl -p [log-level]

journald沿用了syslog中的message level

0. emerg
1. alert
2. crit
3. err
4. warning
5. notice
6. info
7. debug

可根據需求列出對應的log level
```
~>sudo journalctl -p err # 或sudo journalctl -p 3
...
Apr 24 17:16:33 test crash_sender[175337]: Error calling D-B
Apr 24 17:16:33 test crash_sender[175337]: CallMethodAndBlo
```

### journalctl -k 
列出kernel message，使用上與dmesg指令相同

### journalctl --since [time window] --until [time window]

另外也可以根據時間範圍來查詢

time window的格式如下

```
YYYY-MM-DD HH:MM:SS
```

此外常用的一些時間名詞也可辨認，如
"yesterday", "today", "1 hour ago", "now"

# Reference
1. [how-to-use-journalctl-to-view-and-manipulate-systemd-logs](https://www.digitalocean.com/community/tutorials/how-to-use-journalctl-to-view-and-manipulate-systemd-logs)
2. [鳥哥的Linux私房菜:認識與分析登錄檔](http://linux.vbird.org/linux_basic/0570syslog.php)
3. [use-journalctl-read-linux-system-logs](https://www.maketecheasier.com/use-journalctl-read-linux-system-logs)

