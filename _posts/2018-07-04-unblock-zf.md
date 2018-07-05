---
layout: post
title:  "设置保险箱监控程序允许中孚检查的方法"
date:   2018-07-04 15:41:49
categories: 专网维护
tags: 保险箱 中孚 一键自查
---

为了能让中孚检查工具能检查保险箱中的文件，必须要确保保险箱监控程序未将中孚软件相关可执行文件列入黑名单，操作步骤如下：

第一步： 从桌面右下角菜单找到【保险箱监控程序】，双击打开【保险箱监控程序】，如下图所示：

![打开保险箱监控程序]({{ site.url }}/assets/zw/unblock_zf01.png)

第二步： 点击【黑名单】，查看是否有 C:\Program Files\ZFChkProof 路径下的程序，包括 ZfchkPro.exe, zhp.exe, ChkProofTray.exe, ChkProof.exe, ChkProofService.exe等。如果有，请选中该条目，并点击右下方“回收站”图标删除。如下图所示：

![删除黑名单]({{ site.url }}/assets/zw/unblock_zf02.png)

第三步： 下次微点弹出是否允许这些程序访问保险箱时，请点击“允许”，为了方便起见，最好在上图的界面中切换到【黑名单】选项卡，将上述的几个 exe 文件加入白名单中，如下图所示：

![加入白名单]({{ site.url }}/assets/zw/unblock_zf03.png)

PS： 如果中孚软件出现问题，可以用如下方式解决，首先在文件管理器里面进入 C:\Program Files 目录，按下 Shift 键的同时鼠标右键点击 ZFChkProof 目录，选择“在此处打开命令窗口”，如下图所示：

![打开命令窗口]({{ site.url }}/assets/zw/unblock_zf04.png)

在命令行窗口中输入： **uninst.exe -autouninstall** ，就可以卸载中孚，如下图所示：

![卸载中孚]({{ site.url }}/assets/zw/unblock_zf05.png)

接下来接入专网，用 IE 浏览器访问 10.16.16.30 下载最新版中孚软件，并完成联网安装注册。
