---
layout: post
title:  "专网和内网U锁的制作"
date:   2015-10-06 20:45:49
categories: 专网维护 内网维护
tags: 迷彩U锁 红U锁
---

迷彩U锁的制作需要使用U锁管理中心和RA。第一步，我们需要在OA中通过webmaster用户登录，建立相应的用户。然后插入待制作的U锁，使用 ePassMgr 对其进行初始化，打开 ePassMgr ，点击“初始化令牌”，如图所示：

![初始化令牌]({{ site.url }}/assets/zw/makeseclock01.png)

第二步，插入迷彩管理中心U锁，登录迷彩U锁管理中心，选择“注册机关用户”，在迷彩U锁列表中选中相应的U锁，并在相应部门中选择新建的人员，点击“注册”，如图所示：

![注册机关用户]({{ site.url }}/assets/zw/makeseclock02.png)

第三步，接下来需要注册网口封堵，在管理中心中点击“其他应用”中的“注册网口封堵”，选择相应的U锁，输入密码后点击“注册”，如图所示：

![注册网口封堵]({{ site.url }}/assets/zw/makeseclock03.png)

第四步，接下来换成RA的管理U锁，点击桌面上的RA图标，输入密码后登录到RA，如图所示：

![登录RA]({{ site.url }}/assets/zw/makeseclock04.png)

第五步，接下来在“用户信息管理”中选择“注册用户”，并输入相应的信息，如图所示：

![注册用户]({{ site.url }}/assets/zw/makeseclock05.png)

第六步，然后在“证书审核管理”中选择“签发审核”，点击“签发审核通过”，如图所示：

![审核通过]({{ site.url }}/assets/zw/makeseclock06.png)

第七步，最后选择“证书签发管理”中的“签发证书”，在“加密设备”一栏中选择 FETIAN …… ，点击制作，输入密码后制作完成，如图所示：

![签发证书]({{ site.url }}/assets/zw/makeseclock07.png)

红U锁的制作与迷彩U锁类似，不过不需要使用管理中心，只需要插入管理员红锁，在RA中根据第4-7步的操作制发证书即可。
