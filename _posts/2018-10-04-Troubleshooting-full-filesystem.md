---
title: Troubleshooting full filesystem
layout: post
---
There are several cases when disk might become 100% full and they have different solutions.

## Case #1: `/var`
In this case we have `/var` full.
```bash
root@host:/# df -h
Filesystem                           Size  Used Avail Use% Mounted on
/dev/mapper/isw_jghacajdf_Volume0p2  9.3G  3.9G  5.0G  44% /
none                                 4.0K     0  4.0K   0% /sys/fs/cgroup
udev                                  63G   12K   63G   1% /dev
tmpfs                                 13G  1.1M   13G   1% /run
none                                 5.0M     0  5.0M   0% /run/lock
none                                  63G   40K   63G   1% /run/shm
none                                 100M     0  100M   0% /run/user
/dev/mapper/isw_jghacajdf_Volume0p5  845G  269G  534G  34% /home
/dev/mapper/isw_jghacajdf_Volume0p3  9.3G  9.3G     0 100% /var
``` 
By issuing `du` command it is possible to find out which file(s) occupy space. Note that `du` __does not show__ hidden files. `ncdu` does but it needs to be installed which is impossible with 100% full `/var` system.
```bash
root@host:/var# du -sh *
924K	backups
103M	cache
103M	lib
4.0K	local
0	lock
9.0G	log
16K	lost+found
4.0K	mail
4.0K	opt
0	run
28K	spool
60M	tmp
root@host:/var# cd log
root@host:/var/log# du -sh *
48K	alternatives.log
68K	apt
845M	auth.log
8.0K	boot.log
24K	btmp
4.0K	dist-upgrade
112K	dmesg
112K	dmesg.0
624K	dpkg.log
8.0K	faillog
4.0K	fontconfig.log
12K	fsck
17M	installer
1.2M	kern.log
40K	lastlog
108K	mail.err
108K	mail.log
4.0K	ntpstats
30M	puppet
4.0K	sssd
8.1G	syslog
4.0K	sysstat
1.1M	udev
60K	upstart
108K	wtmp
``` 
`/var/log/syslog` occupies 8GB, which is quite a lot. 

By looking into it we can find a spamming application.
```bash
root@host:/var/log# tail ./syslog
Oct  2 15:18:52 host /usr/sbin/irqbalance: irq 134 affinity_hint subset empty
Oct  2 15:18:52 host /usr/sbin/irqbalance: irq 135 affinity_hint subset empty
Oct  2 15:18:52 host /usr/sbin/irqbalance: irq 147 affinity_hint subset empty
Oct  2 15:18:52 host /usr/sbin/irqbalance: irq 148 affinity_hint subset empty
Oct  2 15:18:52 host /usr/sbin/irqbalance: irq 149 affinity_hint subset empty
Oct  2 15:18:52 host /usr/sbin/irqbalance: irq 150 affinity_hint subset empty
Oct  2 15:18:52 host /usr/sbin/irqbalance: irq 151 affinity_hint subset empty
Oct  2 15:18:52 host /usr/sbin/irqbalance: irq 152 affinity_hint subset empty
Oct  2 15:18:52 host /usr/sbin/irqbalance: irq 153 affinity_hint subset empty
``` 

Googling point to a known [bug](https://bugs.launchpad.net/ubuntu/+source/irqbalance/+bug/1321425) which was fixed in Ubuntu 14.04.3. Lets check our system. 
```bash
root@host:/var/log# lsb_release -a
No LSB modules are available.
Distributor ID:	Ubuntu
Description:	Ubuntu 14.04.2 LTS
Release:	14.04
Codename:	trusty
``` 
Installed system is behind and needs to be updated. 

## Case #2: Deleted files
Sometime file was deleted but process which is using it was not stopped. `lsof` command can help find those files.
```bash
root@host:/var/log# lsof +L1
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NLINK   NODE NAME
cron    2453 root    6r   REG  252,3  6806312     0 150177 /var/lib/sss/mc/passwd (deleted)
``` 
In this case either process should be stopped or system needs to be rebooted.

## Case #3: Some really crazy stuff
Sometimes you need to go deeper to get to the root cause of the problem. Here are some points about tools that were used previosly. 

**`df`**

`df` shows available space on the actual device. For example, in Case #1, `/dev/mapper/isw_jghacajdf_Volume0p2` is mounted under `/` and `/dev/mapper/isw_jghacajdf_Volume0p3` is mounted under `/var`. But original file system also has `/var` folder and `/dev/mapper/isw_jghacajdf_Volume0p3` just overrides it for navigation purposes. Content from original `/var` is not gone. So, it is possible that `df` reports no space available but `du`, which is using navigation, cannot find it. 

**`du`**

As mentioned before, `du` ignores hidden files. To see them, either use `ncdu` or `zsh`. In `zsh` run following command
```bash
du -sch * .*
```

**`statfs`**
You can use `statfs` to get more information about file system. 
```bash
root@host:~# strace -e statfs df /dev/sdy3
statfs("/", {f_type="EXT2_SUPER_MAGIC", f_bsize=4096, f_blocks=2428040, f_bfree=129341, f_bavail=241, f_files=625856, f_ffree=518195, f_fsid={-1174424332, 396490694}, f_namelen=255, f_frsize=4096}) = 0
Filesystem     1K-blocks    Used Available Use% Mounted on
/dev/sdy3        9712160 9194796       964 100% /
+++ exited with 0 +++
```
Two values are interesting: `f_bfree` and `f_bavail`. First one shows number of blocks free, including ones, reserved for root and the second one shows blocks available for regular usage.

**`debugfs`**

As mentioned in the beginning of the section, `mount` could hide original folders. `debugfs` allow to browse original file system without unmounting, which is impossible in case of `/var`. 