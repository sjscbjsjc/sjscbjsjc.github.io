---
layout: post
title:  "ArangoDB Cookbook"
date:   2017-11-29 13:41:49
categories: ArangoDB
tags: ArangoDB
---

* content
{:toc}

## 安装

去 [ArangoDB 下载页面](https://www.arangodb.com/download/)下载相应的版本，Redhat 系列和 Debian 系列均有相应的 RPM 和 DEB 包，archlinux 用户可以通过 AUR 安装，不过 PGP 校验有点问题，需要使用 *yaourt --m-arg "--skipchecksums --skippgpcheck" -Sb arangodb3* 命令来安装。

安装完成后执行 *arango-secure-installation* ，进行基本的设置。


## 使用

web 版可以通过访问[管理页面(http://localhost:8529)](http://localhost:8529)进行管理，命令行可以通过执行 arangosh 来访问。


## 导入 csv 数据

使用 arangoimp 命令可以导入数据，常用的参数有：

  - --file [文件路径] ： 指定要导入的文件路径
  - --type [文件类型] ： 文件类型，有 json、csv、tsv 等，默认是 json ，对于 csv 文件来说，填入 csv 即可
  - --server.user [用户名] ： 连接服务器的用户名，默认是 root
  - --server.endpoint [地址] ： 服务器地址，默认是 http://127.0.0.1:8529
  - --server.password [密码] ： 用户的密码
  - --server.database [数据库] ： 指定 csv 要导入的数据库名
  - --create-collection [bool] ： 如果集合不存在是否创建之，默认是 false
  - --create-collection-type [类型] ： 创建的集合类型，可以取的值有 document 和 edge
  - --collection [集合名称] ： 指定集合的名称，默认是 ""
  - --quote [符号] ： csv 文件专用，指定引用的符号，默认是双引号
  - --separator [符号] ： 指定字段之间的分隔符，如果指定了 type 为 csv 且为逗号分隔，则此参数可省

举例：

{% highlight bash %}
arangoimp --file ~/arango/companies.csv --server.database ThreeKingdoms --collection companies --create-collection true --type csv
arangoimp --file "data.json" --collection "myedges" --create-collection true --create-collection-type edge
{% endhighlight %}


## 文档与边的结构

文档默认会有一个 _key 字段和 _id 字段，且 _id 名一般为 **集合名/_key** 。

边除了有 _key 和 _id 字段外，还有 _from 和 _to 字段，用来表示边的指向。


## 索引

_id、_key、_from 和 _to 字段会被 ArangoDB 自动生成索引。

