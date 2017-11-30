---
layout: post
title:  "psql 工具使用简介"
date:   2015-10-11 10:42:49
categories: PostgreSQL
tags: PostgreSQL
---

## psql 的简单使用

使用 psql 命令就可以进入 PostgreSQL 命令行模式，在其中输入 help 就可以查看帮助。psql 命令都是以斜杠 **\** 开头的。

系统自带的数据库有两个模板数据库 template0 和 template1 ，新建数据库的时候默认是从后者克隆出来的。


## psql 常用命令

### \d 命令

\d 命令用于显示表、索引等信息，可以跟通配符。如果什么参数都不带，就会显示当前数据库中的所有表；如果跟一个 **表名** ，就会显示表结构；\d 后面还可以跟通配符 * 或 ? ；\d+ 命令将会显示比 \d 命令更详细的信息。

\d 命令后面跟类型符号可以显示指定类型的对象，例如：

1. \dt  显示匹配的表
2. \di  显示匹配的索引
3. \ds  显示匹配的序列
4. \dv  显示匹配的视图
5. \df  显示匹配的函数
6. \dn  显示所有的schema
7. \db  显示所有的表空间
8. \du 或 \dg 显示所有的角色或用户
9. \dp 或 \z 显示表的权限分配情况

### 显示 SQL 已执行的时间

使用 \timing 命令可以显示 SQL 已执行的时间，例如：

{% highlight Bash %}
\timing on
{% endhighlight %}

### 执行存储在外部文件中的 SQL 命令

** \i <文件名> ** 可以执行存储在外部文件中的 SQL 命令。

### \x 命令

\x 命令可以将每行数据都拆分成单行显示。

### \echo 输出信息

\echo 命令用于输出一行信息，例如：\echo hello world 。

### 取消自动提交

执行 begin; 命令可以阻止自动提交，需要提交的时候执行 commit或rollback 即可。

还有一种方法是执行： \set AUTOCOMMIT off 。注意： AUTOCOMMIT 必须大写！！！

### 显示命令实际执行的 SQL 语句

启用 ECHO_HIDDEN 选项可以显示各种反斜杠开头的命令实际执行的 SQL 语句，比如 \d 实际执行的语句，语法是

{% highlight Bash %}
\set ECHO_HIDDEN on | off
{% endhighlight %}

在启动 psql 的时候加入“ -E ” 的参数也可以打开 ECHO_HIDDEN 参数。
