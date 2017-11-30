---
layout: post
title:  "PostgreSQL起步"
date:   2015-10-06 21:08:49
categories: PostgreSQL
tags: PostgreSQL
---

## 安装和配置

### PostgreSQL的安装

PostgreSQL 的安装推荐用 Linux 自带的包管理器或者 [Windows Installer](http://www.enterprisedb.com/products-services-training/pgdownload#windows) 来安装。

笔者在 Archlinux 上安装完成后需要执行下列的命令来初始化数据库：

{% highlight Bash %}
initdb --locale zh_CN.UTF-8 -E UTF8 -D '/var/lib/postgres/data' 
{% endhighlight %}

### 基本配置

首先需要修改 PostgreSQL 的超级用户 postgres 的密码，首先使用 sudo -i -u postgres 命令切换到 postgres 用户，执行 psql 命令进入 PostgreSQL 命令行，执行如下的命令：

{% highlight Bash %}
\password postgres
{% endhighlight %}


## 创建用户与数据库

### 创建用户

创建用户使用如下命令：

{% highlight SQL %}
CREATE USER dbuser WITH PASSWORD 'password';
{% endhighlight %}

### 创建用户数据库

创建用户数据库使用如下命令：

{% highlight SQL %}
CREATE DATABASE exampledb OWNER dbuser;
{% endhighlight %}

### 给用户授权

建立了数据库之后，还需要给用户授权，否则无法操作，简单的授权命令如下：

{% highlight SQL %}
GRANT ALL PRIVILEGES ON DATABASE exampledb to dbuser;
GRANT ALL PRIVILEGES ON ALL TABLES IN SCHEMA public TO sjscb;
{% endhighlight %}


## 基本操作

### 连接数据库

在命令行切换到 postgres 用户后可以采用如下的命令可以连接数据库：

{% highlight Bash %}
psql -U 用户名 -d 数据库名 -h 主机 -p 端口
{% endhighlight %}

### 常用的控制台命令

常用的控制台命令如下：

{% highlight Bash %}
\h： 查看SQL命令的解释，比如\h select。
\?： 查看psql命令列表。
\l： 列出所有数据库。
\c： [database_name]：连接其他数据库。
\d： 列出当前数据库的所有表格。
\l： 列出所有数据库
\d： [table_name]：列出某一张表格的结构。
\du：列出所有用户。
\e： 打开文本编辑器。
\conninfo：列出当前数据库和连接的信息。
{% endhighlight %}


## 备份和恢复

### 备份

常见的备份命令是：

{% highlight Bash %}
pg_dump -U 用户名 -W -v -t 表名 -f 备份文件名 待备份的数据库名
{% endhighlight %}

### 恢复
{% highlight Bash %}
psql -U 用户名 -W -d 数据库名 -f 备份文件名
{% endhighlight %}
