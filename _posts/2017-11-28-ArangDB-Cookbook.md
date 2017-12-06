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

去 [ArangoDB 下载页面](https://www.arangodb.com/download/)下载相应的版本，Redhat 系列和 Debian 系列均有相应的 RPM 和 DEB 包，archlinux 用户可以通过 AUR 安装，不过 PGP 校验有点问题，需要使用 *yaourt \-\-m-arg \"\-\-skipchecksums \-\-skippgpcheck\" -Sb arangodb3* 命令来安装。

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

{% highlight shell %}
arangoimp --file ~/arango/companies.csv --server.database ThreeKingdoms --collection companies --create-collection true --type csv
arangoimp --file "data.json" --collection "myedges" --create-collection true --create-collection-type edge
{% endhighlight %}


## 文档与边的结构

文档默认会有一个 _key 字段和 _id 字段，且 _id 名一般为 **集合名/_key** 。

边除了有 _key 和 _id 字段外，还有 _from 和 _to 字段，用来表示边的指向。


## 索引

_id、_key、_from 和 _to 字段会被 ArangoDB 自动生成索引。

### 索引类型

#### Primary Index

Primay Index 主要建立在文档的 _id 和 _key 字段上，是一种 Unsorted Hash Index ，不能被删除或者改变，也不能创建用户自定义的 Primary Index 。有两个方法可以利用索引快速定位文档：

{% highlight javascript %}
db.collection.document("<document-key>");
db._document("<document-id>");
{% endhighlight %}

#### Edge Index

Edge Index 建立在边集合上，针对 _from 和 _to 两个属性，有一些方法可以利用索引快速定位边所在的文档：

{% highlight javascript %}
db.collection.edges("<from-value>");
db.collection.edges("<to-value>");
db.collection.outEdges("<from-value>");
db.collection.outEdges("<to-value>");
db.collection.inEdges("<from-value>");
db.collection.inEdges("<to-value>");
{% endhighlight %}

#### Hash Index

Hash Index 是没有排序的，因此支持相等的比较，而不支持范围查询和排序。

Hash Index 可以创建在文档的一个或多个属性上，当且仅当所有的被索引属性都放在查询里并使用 == 来进行查询时，才会使用到 Hash Index 。

Hash Index 可以是 unique 的，也可以是 non-unique 的，可以是 sparse 的，也可以是 non-sparse 的。

#### Skiplist Index

Skiplist Index 是一种排序的索引，支持快速定位文档、范围查询和排序，它可以创建在文档的一个或多个属性上。

在 FILTER 或者 SORT 中会使用 Skiplist Index 当且仅当它们按照 Skiplist Index 建立时指定的属性的顺序使用（可以按顺序使用部分属性），且 SORT 多个属性时使用同样的排序方法，例如：

{% highlight AQL %}
// 假设 Skiplist Index 建立在 value1 和 value2 这两个属性上
FILTER doc.value1 > ...    // 能用到索引
FILTER doc.value1 == ... && doc.value2 > ...  //能用到索引
SORT value1 DESC, value2 DESC  // 能用到索引
SORT value2  // 不能用到索引，因为索引的顺序首先是 value1
SORT value1 DESC, value2 ASC  // 不能用到索引，因为排序方法不同，一个逆序，一个顺序
{% endhighlight %}

Skiplist Index 也有 unique 和 sparse 的情况。

#### Persistent Index

Persistent Index 是一种排序的索引，与 Primary Index 加载在内存里不一样，Persistent Index 会直接写入磁盘，这样当系统重启的时候，不需要重建索引。

Persistent Index 的使用要求与 Skiplist Index 一样。

#### Fulltext Index

全文检索索引可以用在单个属性上，使用 libicu 来进行处理，将停用词以外的词进行索引。

全文检索索引会在全文检索查询和简单查询中被使用。

### 创建索引

使用 collection 的 **ensureIndex** 方法可以创建索引，该方法需要 type 和 fields 两个参数，后者是个数组，对应创建索引的一个或多个属性，如果要对某个属性的子属性创建索引，则使用 dot notation ，例如：

{% highlight javascript %}

// 假设数据为 { name: "Smith", age: 35 }
db.posts.ensureIndex({ type: "hash", fields: [ "name" ] })
db.posts.ensureIndex({ type: "hash", fields: [ "name", "age" ] })

// 假设数据为 { name: {last: "Smith", first: "John" } }
db.posts.ensureIndex({ type: "hash", fields: [ "name.last" ] })
db.posts.ensureIndex({ type: "hash", fields: [ "name.last", "name.first" ] })
{% endhighlight %}

#### 对数组创建索引

如果被属性的属性本身是个数组，那默认情况下 ArangoDB 会将数组整个作为索引而不是对其中的一个一个元素进行索引，如果要做到数组元素级的索引，需要使用 array expansion operator ，即 [*] ，例如：

{% highlight javascript %}
db.posts.ensureIndex({ type: "hash", fields: [ "tags[*]" ] });
db.posts.insert({ tags: [ "foobar", "baz", "quux" ] });
{% endhighlight %}

需要注意的是，建立索引的时候，每个属性只能用一次 [*] ，不能写成诸如 "tags[*].name[*].value" 的形式。

对于建立了元素级索引的数组属性来说，插入的时候会自动去除重复，比如执行 db.posts.insert({ tags: [ "foobar", "bar", "bar" ] }) 会仅仅插入一个 "bar" 。

如果不要系统在数组中自动去重复，则需要在建立索引的时候将 deduplicate 参数由默认的 true 改成 false ，例如：

{% highlight javascript %}
db.posts.ensureIndex({ type: "hash", fields: [ "tags[*]" ], deduplicate: false });
{% endhighlight %}

如果元素级索引的数组属性在插入的时候并没有传入一个数组值，则该文档不会被索引。

### 索引的选择

索引的类别选择主要针对查询的情况，如果需要范围查询或者排序，则选用 skiplist ，如果需要写入磁盘而不是放在内存中则选用 persistent ，如果需要全文检索则选用 fulltext 等等。

#### 是否 sparse ，是否 unique

sparse 索引表示当建立索引的属性不存在或者为 null 的时候，就不对该文档进行索引。 unique 则表示索引的元素必须唯一。 建立这两种索引只需要将相应的 sparse 或 unique 参数设置为 true 即可，例如：

{% highlight javascript %}
db.collection.ensureIndex({ type: "hash", fields: [ "attributeName" ], unique: true, sparse: true }); 
db.collection.ensureIndex({ type: "skiplist", fields: [ "attributeName1", "attributeName2" ], sparse: true });
{% endhighlight %}


## 常用操作

### 数据库(Database)级别的操作

在 arangosh 中输入如下命令：

  - db._createDatabase(<name>)    创建数据库
  - db._dropDatabase(<name>)      删除数据库
  - db._databases()               列出所有的数据库
  - db._useDatabase(<name>)       切换数据库
  - db._name()                    获取当前数据库名
  - db._id()                      获取当前数据库ID
  - db._path()                    获取当前数据库路径
  
### 集合(Collection)级别的操作

在 arangosh 中输入如下命令：

  - db._collections()                  列出所有数据库中的所有集合
  - db._collection(<name>)             获取指定的集合
  - db._create(<name>, <properties>)   创建集合
  - db._createEdgeCollection(<name>)   创建边的集合
  - db._drop(<name>)                   删除集合
  
### 用户(User)级别的操作

在 arangosh 中输入如下命令进行用户管理：

{% highlight javascript %}
// 创建用户
var users = require('@arangodb/users');
users.save('用户名', '密码');
// 授权
users.grantDatabase('用户名', '数据库名', 'rw|ro|none');
// 删除用户
users.remove('用户名')
{% endhighlight %}


## 从 SQL 到 AQL

概念的对应关系：

| SQL | AQL |
| :-: | :-: |
| database | database |
| table | collection |
|row | document |
|column | attribute |
|table joins | collection joins |
|primary key | primary key |
|index | index |

### 插入

#### 插入单个记录/文档

SQL 如下：

{% highlight SQL %}
INSERT INTO users (name, gender) 
  VALUES ("John Doe", "m");
{% endhighlight %}

AQL 如下：

{% highlight AQL %}
INSERT { name: "John Doe", gender: "m" } 
  INTO users
{% endhighlight %}

#### 插入多个记录/文档

SQL 如下：

{% highlight SQL %}
INSERT INTO users (name, gender) 
  VALUES ("John Doe", "m"), 
         ("Jane Smith", "f");
{% endhighlight %}

AQL 如下：

{% highlight AQL %}
FOR user IN [ 
  { name: "John Doe", gender: "m" }, 
  { name: "Jane Smith", gender: "f" } 
]
  INSERT user INTO users
{% endhighlight %}

### 更新

#### 更新单个记录/文档

SQL 如下：

{% highlight SQL %}
UPDATE users 
  SET name = "John Smith"
  WHERE id = 1;
{% endhighlight %}

AQL 如下：

{% highlight AQL %}
UPDATE { _key: "1" }
  WITH { name: "John Smith" }
  IN users
{% endhighlight %}

#### 增加一个字段/属性

SQL 如下：

{% highlight SQL %}
ALTER TABLE users 
  ADD COLUMN numberOfLogins 
  INTEGER NOT NULL default 0;
{% endhighlight %}

AQL 如下：

{% highlight AQL %}
FOR user IN users
  UPDATE user 
    WITH { numberOfLogins: 0 } IN users
{% endhighlight %}

#### 删除一个字段/属性

SQL 如下：

{% highlight SQL %}
ALTER TABLE users
  DROP COLUMN numberOfLogins;
{% endhighlight %}

AQL 如下：

{% highlight AQL %}
FOR user IN users
  UPDATE user WITH { numberOfLogins: null } 
    IN users 
  OPTIONS { keepNull: false }
{% endhighlight %}

### 删除

#### 删除单个记录/文档

SQL 如下：

{% highlight SQL %}
DELETE FROM users
  WHERE id = 1;
{% endhighlight %}

AQL 如下：

{% highlight AQL %}
REMOVE { _key:"1" } 
  IN users
{% endhighlight %}

#### 删除多个记录/文档

SQL 如下：

{% highlight SQL %}
DELETE FROM users
  WHERE active = 1;
{% endhighlight %}

AQL 如下：

{% highlight AQL %}
FOR user IN users
  FILTER user.active == 1
  REMOVE user IN users
{% endhighlight %}

### 查询

#### 基本查询

SQL 如下：

{% highlight SQL %}
SELECT * FROM users
  WHERE active = 1
  ORDER BY name, gender;
{% endhighlight %}

AQL 如下：

{% highlight AQL %}
FOR user IN users
  FILTER user.active == 1
  SORT user.name, user.gender
  RETURN user
{% endhighlight %}

#### 计数

SQL 如下：

{% highlight SQL %}
SELECT gender, COUNT(*) AS number FROM users
  WHERE active = 1
  GROUP BY gender
  HAVING number > 20;
{% endhighlight %}

AQL 如下：

{% highlight AQL %}
FOR user IN users
  FILTER user.active == 1
  COLLECT gender = user.gender 
    WITH COUNT INTO number
	FILTER number > 20
  RETURN { 
    gender: gender, 
    number: number 
  }
{% endhighlight %}

### JOIN

#### 内连接

SQL 如下：

{% highlight SQL %}
SELECT * FROM users
  INNER JOIN friends 
  ON (friends.user = users.id);
{% endhighlight %}

AQL 如下：

{% highlight AQL %}
FOR user IN users 
  FOR friend IN friends
    FILTER friend.user == user._key
    RETURN MERGE(user, friend)
{% endhighlight %}

或者这样以避免属性名冲突：

{% highlight AQL %}
FOR user IN users 
  FOR friend IN friends
    FILTER friend.user == user._key
    RETURN { user: user, friend: friend }
{% endhighlight %}

#### 外连接

SQL 如下：

{% highlight SQL %}
SELECT * FROM users
  LEFT JOIN friends 
    ON (friends.user = users.id);
{% endhighlight %}

AQL 不直接支持外连接，只有用子查询来实现，如下：

{% highlight AQL %}
FOR user IN users 
  LET friends = (
    FOR friend IN friends
      FILTER friend.user == user._key
      RETURN friend
  )
  FOR friendToJoin IN (
    LENGTH(friends) > 0 ? friends :
      [ { /* no match exists */ } ]
    )
    RETURN { 
      user: user,
      friend: friend
    }
{% endhighlight %}
