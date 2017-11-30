---
layout: post
title:  "办内专网与外网地址切换"
date:   2017-08-19 9:15:00
categories: 专网维护
tags: IP地址
---

* content
{:toc}


办内的因特网采用的是 DHCP 的形式；专网采用的是固定 IP 的形式，且每个人都分配有自己的 IP 地址，大家可以在专网公告里看到。

我们可以建两个 .bat 的批处理文件，一个叫做 “外网.bat”，内容如下（请把“本地连接”换成您机器的连接名称）：

{% highlight shell %}
netsh interface ip set address name="本地连接" source=dhcp
netsh interface ip set dns name="本地连接" source=dhcp
{% endhighlight %}

另一个叫“专网.bat”，内容如下（请把 XXX 换成您的 IP 地址）：

{% highlight shell %}
netsh interface ip set address name="本地连接" source=static addr=10.31.10.XXX mask=255.255.255.0 gateway=10.31.10.1
netsh interface ip set dns name="本地连接" source=static addr=10.31.4.2
{% endhighlight %}
