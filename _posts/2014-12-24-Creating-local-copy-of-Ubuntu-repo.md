---
layout: post
title: Creating Local Copy Of Ubuntu Repo
---
Sometime production servers are located behind firewall but some software needs to be installed. Usually, copy of Ubuntu's main repository already exists but you might find yourself in situation when you need packages from third-party sources.

In this case you need to create local copy of the repository. Following, I'll describe how to do that for [Docker](http://www.docker.com/) case for Ubuntu 14.04.

First, install apt-mirror:

```bash
#sudo apt-get update
```
