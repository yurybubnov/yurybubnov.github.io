---
title: Installing Java from DEB file and no access to internet
layout: post
---

Sometome you need to install Oracle Java on Ubuntu box which has no access to internet. Here are the steps whick will help to install it with help of [webupd8team](https://launchpad.net/~webupd8team/+archive/ubuntu/java). 

## Obtain nessesary files
You'll need two files on Ubuntu box.

 - DEB installer file could be downloaded from [PPA](http://ppa.launchpad.net/webupd8team/java/ubuntu/pool/main/o/)
 - Download appropriate TAR.GZ file from [JDK downlaod page](http://www.oracle.com/technetwork/java/javase/downloads/index.html). If you downloaded `oracle-java8-installer_8u60+8u60arm-1~webupd8~1_all.deb`, you'll need `jdk-8u60-linux-x64.tar.gz`

## Installation steps
On target Ubuntu box, execute following:

 - Create folder `/var/cache/oracle-jdk8-installer` and place TAR.GZ file there (change folder name if nessesary)
 - Execute `install` on DEB file
 
```bash
$ mkdir /var/cache/oracle-jdk8-installer
$ mv ./jdk-8u60-linux-x64.tar.gz /var/cache/oracle-jdk8-installer
$ dpkg -i ./oracle-java8-installer_8u60+8u60arm-1~webupd8~1_all.deb
Selecting previously unselected package oracle-java8-installer.
...
Oracle JDK 8 installed
update-alternatives: using /usr/lib/jvm/java-8-oracle/jre/lib/amd64/libnpjp2.so to provide /usr/lib/mozilla/plugins/libjavaplugin.so (mozilla-javaplugin.so) in auto mode
Oracle JRE 8 browser plugin installed
Processing triggers for shared-mime-info (1.2-0ubuntu3) ...
Processing triggers for mime-support (3.54ubuntu1) ...
root@phxdbx1021:/tmp# java -version
java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
```

Now yout have Java installed:

```bash
> java -version
'java version "1.8.0_60"
Java(TM) SE Runtime Environment (build 1.8.0_60-b27)
Java HotSpot(TM) 64-Bit Server VM (build 25.60-b23, mixed mode)
```