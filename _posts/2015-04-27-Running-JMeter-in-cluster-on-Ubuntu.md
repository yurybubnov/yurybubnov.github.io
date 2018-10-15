---
title: Running JMeter in cluster on Ubuntu
layout: post
tags: [Java, JMeter]
--- 

There are several catches when you want to run Jmeter in cluster set up. 

___

## Ubuntu Specifics
If you're getting exception like  
```bash
jmeter.threads.JMeterThread: Test failed! java.lang.NoClassDefFoundError: Could not initialize class org.apache.jmeter.testbeans.gui.GenericTestBeanCustomizer
```
while running Jmeter Server on Ubuntu, install following packages 
```bash
sudo apt-get -y install libxrender1 libxtst6 libxi6
```
___

## JMeter Cluster

### Definitions
JMeter has somewhat confusing definitions for cluster configuration.  
**JMeter Client** is a command center, it sends commands to JMeter Servers. Tere only one instance of it.  
**JMeter Server** is an application that actually runs test. There are many instances of JMeter Server.

### JMeter ports
If there is no firewall between Client and Server, just use JMeter documentaion to set up cluster.  

Following, I'll describe how to set up Cluster in presence of firewall between Client and Server.  

JMeter requires Client and Server be avalable for direct RMI calls. Client will sends commands to Server for executon and Server will send results back. This means, that both client and server are listening on some ports. Client opens just one port to receive staistics, Server opens two ports, one for RMI and another for receiving commands. Second port, usually, is dynamically allocated and advertised via look up command on RMI port. If you have firewall between Client and Server, you want the port be static, this have pros and cons. Pros, that you know the port and can statically configure it elsewhere; cons, that you have to make sure this port is available at Server start time.

### Running JMeter in Cluster
Sicne we need listerning ports available on both sides, easiest way to run JMeter, will be to establish ssh tunnels between Client and Servers hosts.
If you don't know SSH tunneling, read [this](http://linuxcommand.org/man_pages/ssh1.html).  

#### Server Configuration
Add/modify following in `user.properties`. Change ports to differnt values for every Server.
```bash
server_port=24001 
server.rmi.port=24001 #Save value as above
server.rmi.localport=26001
```
In `jmeter-server` add following to start parameters. First one is to forces JMeter work on IPv4 stack and the second one forces to open port on `localhost` instead of "real" IP address.
```bash
-Djava.net.preferIPv4Stack=true -Djava.rmi.server.hostname=127.0.0.1
```

#### Client Configuration
Add/modify following in `user.properties`. In addition to those properties, you might want to modify 'mode' to fit your requirement. 
``` bash
remote_hosts=127.0.0.1:24001
client.rmi.localport=25000
```

#### Running SSH
Now, time to establish tunnels between hosts. You can do that with following command. Modify `-L` ports for each Server to values, you specified in `jmeter.properties`
```bash
ssh  -L 24001:0.0.0.0:24001 -L 26001:0.0.0.0:26001 -R 25000:0.0.0.0:25000 jmeter-server-1
```