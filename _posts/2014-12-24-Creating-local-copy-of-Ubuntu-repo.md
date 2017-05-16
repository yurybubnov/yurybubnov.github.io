---
layout: post
title: Creating Local Copy Of Ubuntu Repo
---
Sometime production servers are located behind firewall but some software needs to be installed. Usually, copy of Ubuntu's main repository already exists but you might find yourself in situation when you need packages from third-party sources.

In this case you need to create local copy of the repository. Following, I'll describe how to do that for [Docker](http://www.docker.com/) case for Ubuntu 14.04.

You'll need one Ubuntu box which has connection to the Internet to create local copy of the Docker repo.

```bash
>sudo apt-get update
>sudo apt-get install apt-mirror
```

Create ```mirror.list``` file.

```bash
>sudo sh -c "echo 'deb https://get.docker.com/ubuntu docker main
deb-i386 https://get.docker.com/ubuntu docker main
deb-amd64 https://get.docker.com/ubuntu docker main'>/etc/apt/mirror.list"
```

Now run ```apt-mirror```.
{%highlight bash %}
>sudo apt-mirror
Downloading 14 index files using 14 threads...
Begin time: Wed Dec 24 14:09:58 2014
[14]... [13]... [12]... [11]... [10]... [9]... [8]... [7]... [6]... [5]... [4]... [3]... [2]... [1]... [0]...
End time: Wed Dec 24 14:09:59 2014

Processing tranlation indexes: [T]

Downloading 0 translation files using 0 threads...
Begin time: Wed Dec 24 14:09:59 2014
[0]...
End time: Wed Dec 24 14:09:59 2014

Processing indexes: [P]

117.1 MiB will be downloaded into archive.
Downloading 37 archive files using 20 threads...
Begin time: Wed Dec 24 14:09:59 2014
[20]... [19]... [18]... [17]... [16]... [15]... [14]... [13]... [12]... [11]... [10]... [9]... [8]... [7]... [6]... [5]... [4]... [3]... [2]... [1]... [0]...
End time: Wed Dec 24 14:10:17 2014

0 bytes in 0 files and 0 directories can be freed.
Run /var/spool/apt-mirror/var/clean.sh for this purpose.

Running the Post Mirror script ...
(/var/spool/apt-mirror/var/postmirror.sh)


Post Mirror script has completed. See above output for any possible errors.
{%endhighlight%}

Now you have mirror under ```/var/spool/apt-mirror/mirror/get.docker.com/```.

Remove GPG key, unless you want to import it on target system:
{%highlight bash%}
>sudo rm /var/spool/apt-mirror/mirror/get.docker.com/ubuntu/dists/docker/Release.gpg
{%endhighlight%}

Create a tar with mirror's content:
{%highlight bash%}
>cd /var/spool/apt-mirror/mirror/get.docker.com/; tar zcf /tmp/docker_mirror.tgz ./ubuntu/
{%endhighlight%}

Now you have tar file with content of the mirror. Copy it to the target system.
You're done with this server.

___

###On the targer server:

Install ```apache2```:
{%highlight bash%}
>sudo apt-get -y install apache2
{%endhighlight%}

Untar mirror to apache's www fodler
{%highlight bash%}
>sudo tar zxvf /tmp/docker_mirror.tgz -C /var/www/html
{%endhighlight%}

make sure your apache is up and runnig and you're done with it.

---

###On server where Docker needs to be installed
Add your new mirror location to apt sources
{%highlight bash%}
sudo sh -c "echo deb http://{YOUR MIIRROR'S HOST NAME OR IP HERE}/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
{%endhighlight%}

Fire ```apt-get update``` and make sure your new server is in there
{%highlight bash%}
>sudo apt-get update
Ign http://1.2.3.4 docker InRelease
Ign http://1.2.3.4 docker Release.gpg
Get:1 http://1.2.3.4 docker Release [1,525 B]
Get:2 http://1.2.3.4 docker/main amd64 Packages [5,453 B]
Get:3 http://1.2.3.4 docker/main i386 Packages [20 B] 
...
{%endhighlight%}

And install Docker
{%highlight bash%}
>sudo apt-get -y install lxc-docker
{%endhighlight%}


You might get message that package is not authenticated, but ignore it.
Have fun with Docker!