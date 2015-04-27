---
layout: post
title: Running JMeter in cluster on Ubuntu
---
There are several catches when you want to run Jmeter in cluster set up. 

____
###Ubuntu specifics
If you're getting exception like ```jmeter.threads.JMeterThread: Test failed! java.lang.NoClassDefFoundError: Could not initialize class org.apache.jmeter.testbeans.gui.GenericTestBeanCustomizer``` while running Jmeter Server on Ubuntu, install following packages 
{%highlight bash%}
>sudo apt-get -y install libxrender1 libxtst6 libxi6
{%endhighlight%}