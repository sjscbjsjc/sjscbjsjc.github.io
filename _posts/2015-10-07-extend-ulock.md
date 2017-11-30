---
layout: post
title:  "专网迷彩U锁的延期"
date:   2015-10-07 20:45:49
categories: 专网维护
tags: 迷彩U锁
---

迷彩U锁如果没有过期，则可以在客户端上进行延期。对于 XP 系统，需要在专网下载中心下载“迷彩U锁证书更新工具”，安装重启后按照提示要求更新；如果是 Win7 系统，则不需要安装更新工具，直接访问网站即可。

迷彩U锁如果已经过期，则需要在 RA 上进行证书延期。首先插入管理员锁登录 RA ，查找到相应的用户后，点击“申请延期证书”，如图所示：

![申请延期]({{ site.url }}/assets/zw/extendlock01.png)

接下来需要在“证书审核管理”的“延期审核”中点击“延期审核通过”，如图所示：

![审核通过]({{ site.url }}/assets/zw/extendlock02.png)

然后需要在“证书签发管理”的“延期证书”中点击“延期用户证书”，如图所示：

![延期证书]({{ site.url }}/assets/zw/extendlock03.png)

在选择加密设备的时候，需要选择 “Microsoft Enhanced Cryptographic Provider v1.0” ，点击延期，如图所示：

![选择设备]({{ site.url }}/assets/zw/extendlock04.png)

延期完成后，需要用 ePassMgr 输入U锁密码后删除证书，然后在 RA 中进行 “恢复密钥” 的操作，此时的加密设备需要选择 FETIAN …… ，恢复完成即可。如果恢复中出现服务器错误，则需要致电服务热线，请北京那边处理。
