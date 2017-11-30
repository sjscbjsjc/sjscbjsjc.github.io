---
layout: post
title:  "Oracle 常用命令"
date:   2015-12-17 10:51:49
categories: Oracle
tags: Oracle
---

## 表空间和用户

Oracle的用户可以理解为SQL Server中的库，每个用户下可以有若干张表；用户的数据存放在表空间里，一个用户的数据可以存放在多个表空间。

### 建立表空间和用户

建立用户和表空间的语句如下：

{% highlight SQL %}
create tablespace 表空间名
  datafile '文件路径1' size xxM autoextend on next xxM maxsize 3000M,
  '文件路径2' size xxM autoextend on next xxM maxsize 3000M
  extent management local segment space management auto;

create user 用户名 identified by 密码 default tablespace XXX account unlock;

grant connect to 用户名 with admin options;
{% endhighlight %}

当然，还可以给用户grant更高的权限，比如dba、resource等。

### 表空间扩容

当我们的表空间满了的时候，可以增加数据文件进行扩容，命令如下：

{% highlight SQL %}
alter tablespace 表空间名
add datafile '文件路径' size xxM autoextend on next xxM maxsize 3000M;
{% endhighlight %}


## 数据的导入与导出

Oracle数据的导入导出有两套对应的工具，分别是imp和exp，以及impdp和expdp。后者功能更强大，支持更多的选项，但缺点是不能跨版本使用。因此我们在实际工作中前者用得更多。

导入数据的时候如果使用imp，则必须搞清楚需要建立的表空间和用户，否则有可能会报错。如果是impdp，则可以使用 **remap_tablespace和remap_schema** 两个参数来改变用户和表空间。

### 用exp导出数据

基本的exp命令如下：

{% highlight Bash %}
exp 用户名/密码@地址:端口号/实例名 file=导出文件路径.dmp
{% endhighlight %}

在上面的命令中，如果是导出本机的数据，则地址端口号这些可以省略。exp可以设置的参数有：

1. owner：设置导出哪些用户下的表，多个用户名之间用逗号分隔，例如： owner=scott 以及 owner=(scott, sjscb)。
2. log：指定导出的日志文件的路径。
3. tables：指定要导出的表，多个表名之间用逗号分隔，例如：tables=(table1,table2,table3)。
4. buffer：指定缓存大小，单位是byte，例如要设置为1M，则需要设置为buffer=1048576，提高一点buffer会提升速度，但太高了反而会影响效率。 
5. grants、indexes、constraints和triggers：如果在把这些选项都等于y，例如grants=y，就会相应地导出授权、索引、约束和触发器。
6. filesize：指定导出文件的大小，比如filesize=2G，如果导出的内容大于filesize，则会分成若干个文件。

### 用imp导入数据

基本的imp命令如下：

{% highlight Bash %}
imp 用户名/密码@地址:端口号/实例名 file=dmp文件路径 full=y
imp 用户名/密码@地址:端口号/实例名 file=dmp文件路径 fromuser=xxx touser=xxx
{% endhighlight %}

第一个命令是全部导入，导入的时候需要建立与导出的文件一致的用户。第二个命令可以将dmp文件中的指定用户下面的数据导入到touser指定的用户中去。

如果是用exp导出且指定了filesize，则可能形成多个dmp文件，这时就需要在file中将多个文件括起来，文件名之间用逗号分隔，例如：file=(a_1.dmp, a_2.dmp, a_3.dmp)。

同理，fromuser和touser也可以指定多个用户。

imp的常见参数如下：

1. log：指定导入的日志文件的路径。
2. tables：指定要导入的表，多个表名之间用逗号分隔，例如：tables=(table1,table2,table3)。
3. buffer：设置缓冲区大小。
4. ignore：如果设置为y，则会忽略创建错误
5. grants、indexes：如果设置为y，则会相应地导入授权或索引。

### 用expdp导出数据

用expdp导出的数据会存放在Directory对象指定的路径下，因此我们需要首先创建Directory对象，命令如下：

{% highlight SQL %}
create directory 目录名 as 路径;
grant read,write on directory 目录名 to 用户名;
{% endhighlight %}

目录创建好之后就可以进行导出了，常见的导出命令如下：

{% highlight Bash %}
expdp 用户名/密码@地址:端口号/实例名 directory=Directory对象名 dumpfile=文件名.dmp
{% endhighlight %}

如果不指定文件名，则默认是expdat.dmp，其他参数还有log、tables、tablespaces等。

### 用impdp导入数据

用impdp导入数据的时候，需要先将dmp文件放在Directory对象指定的目录中，然后执行：

{% highlight Bash %}
impdp 用户名/密码@地址:端口号/实例名 directory=Directory对象名 dumpfile=文件名.dmp
{% endhighlight %}

如果要更换表空间，可以使用 **remap_tablespace** 参数。用法是：remap_tablespaces=源表空间:目标表空间。
