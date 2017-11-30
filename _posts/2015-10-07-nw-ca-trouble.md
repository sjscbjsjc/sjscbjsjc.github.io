---
layout: post
title:  "内网无法连接到网关问题的处理"
date:   2015-10-07 20:46:50
categories: 内网维护
---

有时候内网在锁和CA都正常且右下角锁的图标为正常连接的情况下出现无法连接到网关或者是连接网关超时的故障，此时需要建立一个.bat文件，其内容是：

{% highlight bat %}
"C:\Program Files\SecretAuth\CliCtrl.exe" exit
"C:\Program Files\SecretAuth\SSLVPNRedirectorInstaller.exe" -u
"C:\Program Files\SecretAuth\SSLVPNRedirectorInstaller.exe" -a
"C:\Program Files\SecretAuth\CliCtrl.exe" run
exit;
{% endhighlight %}
