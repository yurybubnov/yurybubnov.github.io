---
title: Creating Local Copy Of Ubuntu Repo
layout: post
---

Sometime production servers are located behind firewall but some software needs to be installed. Usually, copy of Ubuntu's main repository already exists but you might find yourself in situation when you need packages from third-party sources.

In this case you need to create local copy of the repository. Following, I'll describe how to do that for [Docker](http://www.docker.com/) case for Ubuntu 14.04.

You'll need one Ubuntu box which has connection to the Internet to create local copy of the Docker repo.

```bash
$ sudo apt-get update
$ sudo apt-get install apt-mirror
```

Create ```mirror.list``` file.

```bash
$ sudo sh -c "echo 'deb https://get.docker.com/ubuntu docker main\n /
deb-i386 https://get.docker.com/ubuntu docker main\n /
deb-amd64 https://get.docker.com/ubuntu docker main'>/etc/apt/mirror.list"
```

Now run ```apt-mirror```.
```bash
$ sudo apt-mirror
Downloading 14 index files using 14 threads...
...# A lot of output here
Post Mirror script has completed. See above output for any possible errors.
```

Now youhave mirror under ```/var/spool/apt-mirror/mirror/get.docker.com/```.

Remove GPG key, unless you want to import it on target system:
```bash
$ sudo rm /var/spool/apt-mirror/mirror/get.docker.com/ubuntu/dists/docker/Release.gpg
```

Create a tar with mirror's content:
```bash
$ cd /var/spool/apt-mirror/mirror/get.docker.com/; tar zcf /tmp/docker_mirror.tgz ./ubuntu/
```

Now you have tar file with content of the mirror. Copy it to the target system.
You're done with this server.

___

### On the targer server:

Install ```apache2```:
```bash
$ sudo apt-get -y install apache2
```

Untar mirror to apache's www fodler
```bash
$ sudo tar zxvf /tmp/docker_mirror.tgz -C /var/www/html
```

make sure your apache is up and runnig and you're done with it.

---

### On server where Docker needs to be installed
Add your new mirror location to apt sources
```bash
$ sudo sh -c "echo deb http://{YOUR MIIRROR'S HOST NAME OR IP HERE}/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
```

Fire ```apt-get update``` and make sure your new server is in there
```bash
$ sudo apt-get update
Ign http://1.2.3.4 docker InRelease
Ign http://1.2.3.4 docker Release.gpg
Get:1 http://1.2.3.4 docker Release [1,525 B]
Get:2 http://1.2.3.4 docker/main amd64 Packages [5,453 B]
Get:3 http://1.2.3.4 docker/main i386 Packages [20 B] 
...
```

And install Docker
```bash
$ sudo apt-get -y install lxc-docker
```


You might get message that package is not authenticated, but ignore it.
Have fun with Docker!