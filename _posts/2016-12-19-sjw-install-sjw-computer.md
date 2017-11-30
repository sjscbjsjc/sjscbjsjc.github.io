---
layout: post
title:  "数据分析网软件的安装"
date:   2016-12-19 10:46:49
categories: 数据分析网维护
tags: 数据分析网
---

* content
{:toc}

## 三合一软件的安装

### 三合一服务和管理端的安装

1. 安装管理端光盘中的“内网服务器集成安装”目录中的程序，完成MySQL的安装。
2. 执行管理端光盘根目录下的“usetup.exe”程序，选择涉密网络终端，管理端地址设置为 **本机IP** ，开始安装；等出现“单项导入设备相关程序即将安装”的对话框时，点击“确定”，安装WinPCap；等出现终端注册对话框的时候，输入相应的信息（其他客户端会通过网络得到这些信息），点击“确定”，注册成功后点击“退出”。最后重启完成安装。
3. 安装管理端光盘中的“管理端”目录中的程序，完成管理端的安装。

### 三合一客户端的安装

**在保证能ping通管理端主机的情况下** ，执行客户端光盘根目录下的“usetup.exe”程序，选择涉密网络终端，管理端地址设置为 **管理端机器IP** ，开始安装；等出现“单项导入设备相关程序即将安装”的对话框时，点击“确定”，安装WinPCap；等出现终端注册对话框的时候，输入相应的信息，点击“确定”，注册成功后点击“退出”。最后重启完成安装。

### 三合一的卸载

插入管理员锁，执行管理端光盘“uninstall”目录下的“卸载客户端.exe”程序，根据提示输入管理员锁的密码，完成卸载。


## 瑞星杀毒软件的安装

运行安装程序，选择安装“网络客户端”，中心地址设置为 **172.16.16.12** 即可，然后一路next，安装完成后重启即可。


## 虚拟云平台的安装

访问署数据分析网主页 http://172.16.27.11 ，在右下角有一个软件下载，下载对应版本的客户端以后，执行安装即可。


## 上传软件 FileZilla 的安装和使用

在数据上报的机器上需要安装FileZilla服务器端，将之作为服务器，署数据交换服务器作为FileZilla客户端，这样就可以将数据上传了。

访问署数据分析网主页 http://172.16.27.11 ，在右下角有一个软件下载，点击“数据上传软件下载”，下载FileZilla服务器端，执行安装即可。

安装完成后，选择菜单 Edit->Settings ，在 **General settings** 里面，将 **Number of threads** 设置为10；在 **Passive mode settings** 里面，将 **Use custom port range** 勾选上，范围设置为5000-5100；在 **FTP over TLS settings** 中，勾选 **Enable FTP over TLS support(FTPS)** ，然后点击 **Generate new certificate...** ，在弹出的对话框中，将2-Digit Country Code一项设置为21，后面四项全部设置为chengdu，**Common name** 一项设置为本机的IP地址，文件保存的位置可以自己随便设置，然后点击Generate certificate生成。最后点击OK返回。

在FileZilla Server的工具栏中点击 **Groups** 按钮，在General中点击Add按钮，加入一个Group，取名为chengdu；然后在Shared Folders中点击Add，把待上传数据的目录加进去，并赋予Files Read Write，Directories Create List和+Subdirs的权限，并点击 **Set as home dir** 按钮。最后返回主界面。

在FileZilla Server的工具栏中点击 **Users** 按钮，在General中点击Add按钮，加入一个用户chengdu，将其用户组也设置为chengdu，在password一栏中输入相应的密码；然后在Speed Limits中，勾选No Limit，点击OK返回。
