---
layout: post
title:  "ArangoDB 及 AQL 简明教程"
date:   2017-11-11 10:51:49
categories: ArangoDB
tags: ArangoDB
---

* content
{:toc}

## 安装

去 [ArangoDB 下载页面](https://www.arangodb.com/download/)下载相应的版本，Redhat 系列和 Debian 系列均有相应的 RPM 和 DEB 包，archlinux 用户可以通过 AUR 安装，不过 PGP 校验有点问题，需要使用 yaourt --m-arg "--skipchecksums --skippgpcheck" -Sb arangodb3 命令来安装。

安装完成后执行 arango-secure-installation ，进行基本的设置。


## 使用

web 版可以通过访问[管理页面](http://localhost:8529)进行管理，命令行可以通过执行 arangosh 来访问。


## 基本概念

Collections（集合） 相当于关系数据库中的表，而 Documents（文档） 相当于关系数据库中的记录，只不过 Documents 是 Schema-less 的。

每一个 Document 都有一个 _key 和 _id 字段，可以自动生成，也可以手动指定，_id 的名称通常是 “Collection名/_key” 的形式，例如 users/00001 。


## AQL 基础

AQL 可以用三种方式来执行：

  1. 在 web 界面的 Queries 中直接输入 AQL 语句
  2. 在 arangosh 或者 Foxx 服务中使用 db 对象
  3. 使用 raw HTTP API

AQL 是 ArangoDB 使用的查询语言，一个 AQL 语句要么需要 RETURN 一个结果，要么需要进行数据修改（使用 INSERT、UPDATE、REPLACE、REMOVE 和 UPSERT 之中的一个）。AQL 一次只能执行一个语句，因此不能像有些关系型数据库一样用分号来分隔多个语句。

### 注释

AQL 的注释类似 C 语言，可以用 // 的行注释，也可以用 /* */ 的块注释

### 关键字

AQL 常用的关键字有：

  - FOR: 循环迭代
  - RETURN: 返回结果
  - FILTER: 过滤结果，相当于 SQL 的 WHERE
  - SORT: 结果排序，降序的话加 DESC
  - LIMIT: 限制返回的记录条数
  - LET: 变量赋值
  - COLLECT: 结果分组，类似 SQL 的 GROUP BY
  - INSERT: 插入新文档
  - UPDATE: 更新已有文档
  - REPLACE: 替换已有文档
  - REMOVE: 删除已有文档
  - UPSERT: 插入新文档或更新已有文档
  
一个 AQL Query 可以包含若干个上面的关键字，关键字不区分大小写。

### 标识符

AQL 的标识符最长支持64字节，是 **大小写敏感** 的！！ 标识符用于集合(Collection)名，属性(Attribute)名，以及变量(Variable)名。

### 数据类型

AQL 的数据类型类似 JSON ，有数字和字符串两种原始类型，和数组以及对象两种组合类型，数组用方括号括起来，对象用花括号括起来。数组的下标可以是负数，表示从尾部往前数；对象的属性名需要用引号引起来。例如：

{% highlight AQL %}
[ 1, 2, 3 ]
[ -99, "yikes!", [ true, [ "no"], [ ] ], 1 ]
{ "name": "Fred", "age" : 36 }
{ "name" : "John", likes : [ "Swimming", "Skiing" ], "address" : { "street" : "Cucumber lane", "zip" : "94242" } }
{% endhighlight %}

对象的属性还可以是通过动态表达式计算出来的值，不过需要 **用中括号括起来** ，例如：

{% highlight AQL %}
{ [ CONCAT("test/", "bar") ] : "someValue" }
{% endhighlight %}

有一种简单的写法用来返回对象的属性，例如：

{% highlight AQL %}
LET name = "Peter"
LET age = 42
RETURN { name, age }
// 上面的 RETURN 语句与下面的等价
RETURN { name : name, age : age }
{% endhighlight %}

获取某个对象的属性可以用 **.属性名 或者 ["属性名"]** 的形式，相比于 .属性名 的方式，中括号的方式允许里面为表达式，例如： 

{% highlight AQL %}
u.friends[0].name.first
u["friends"][0]["name"]["first"]

LET attr1 = "friends"
LET attr2 = "name"
u[attr1][0][attr2][ CONCAT("fir", "st") ]
{% endhighlight %}

### 大小顺序

如果待比较的两个值的类型不同，则大小顺序是： **null < bool < number < string < array/list < object/document** 。

针对每一种数据类型，null 与 null 相等，false < true ，number按数值大小比较，string 按字母顺序比较。

如果是两个数组想比较，则按顺序比较它们的元素，一旦有一对存在大小关系，则大小关系就定下来了，例如：

{% highlight AQL %}
[ ] < [ 0 ]
[ 1 ] < [ 2 ]
[ 1, 2 ] < [ 2 ]
[ 99, 99 ] < [ 100 ]
[ false ] < [ true ]
[ false, 1 ] < [ false, '' ]
{% endhighlight %}

如果是两个对象进行比较，则系统会自动先把所有属性进行排序，先比较属性名，相同属性名再比较其值，一旦有一对存在大小关系，则大小关系就确定了，例如：

{% highlight AQL %}
{ } < { "a" : 1 }
{ } < { "a" : null }
{ "a" : 1 } < { "a" : 2 }
{ "b" : 1 } < { "a" : 0 }   // 由于第一个元素没有 "a" 这个属性，因此为 null ，比第二个元素小
{ "a" : { "c" : true } } < { "a" : { "c" : 0 } }
{ "a" : { "c" : true, "a" : 0 } } < { "a" : { "c" : false, "a" : 1 } }
{ "a" : 1, "b" : 2 } == { "b" : 2, "a" : 1 }
{% endhighlight %}

### 运算符

AQL 的运算符比较类似 Python、JavaScript 等语言，与 SQL 稍有不同，并且支持正则表达式，常用的运算符有：

  - == 相等
  - != 不等
  - <  小于
  - <= 小于等于
  - \>  大于
  - \>= 大于等于
  - IN 判断一个数组是否包含某个值
  - NOT IN 判断一个数组是否不包含某个值
  - LIKE 类似 SQL 的 LIKE
  - =~ 判断一个字符串是否匹配某个正则表达式
  - !~ 判断一个字符串是否不匹配某个正则表达式

需要指出的是 AQL 的 LIKE 可以使用通配符，其中 % 表示匹配零到任意个字符，而 _ 表示匹配一个任意字符，而如果字符中含有 % 或 _ ，则需要用两个反斜线来进行专义，例如：

{% highlight AQL %}
"abc" LIKE "a%"               // true
"abc" LIKE "_bc"              // true
"a_b_foo" LIKE "a\\_b\\_foo"  // true
{% endhighlight %}

#### 数组比较运算符

数组之间可以用 **ALL, ANY 或者 NONE** 来修饰运算符，实现更多样化的判断，例如：

{% highlight AQL %}
[ 1, 2, 3 ] ALL IN [ 2, 3, 4 ]   // false
[ 1, 2, 3 ] ALL IN [ 1, 2, 3 ]   // true
[ 1, 2, 3 ] NONE IN [ 3 ]        // false
[ 1, 2, 3 ] NONE IN [ 23, 42 ]   // true
[ 1, 2, 3 ] ANY IN [ 4, 5, 6 ]   // false
[ 1, 2, 3 ] ANY IN [ 1, 42 ]     // true
[ 1, 2, 3 ] ANY == 2             // true
[ 1, 2, 3 ] ANY == 4             // false
[ 1, 2, 3 ] ANY > 0              // true
[ 1, 2, 3 ] ANY <= 1             // true
[ 1, 2, 3 ] NONE < 99            // false
[ 1, 2, 3 ] NONE > 10            // true
[ 1, 2, 3 ] ALL > 2              // false
[ 1, 2, 3 ] ALL > 0              // true
[ 1, 2, 3 ] ALL >= 3             // false
["foo", "bar"] ALL != "moo"      // true
["foo", "bar"] NONE == "bar"     // false
["foo", "bar"] ANY == "foo"      // true
{% endhighlight %}

#### 逻辑运算符

AQL 的逻辑运算符有 && || 和 ! ，也可以用 AND OR 和 NOT 。

需要注意的是逻辑运算符是有返回值的，a && b 当 a 的值为 false 或者 false 等价的值时返回 a ，否则返回 b ； a || b 当 a 的值为 true 或者 true 等价的值时返回 a ，否则返回 b 。

AQL 会按照如下标准来判断逻辑值：

  - null、0和空字符串会被转换成 false，其他数字和字符串都会被转换成 true
  - 数组和对象不管内容是什么，都会被转化为 true

#### 算数运算符

算术运算符与常见的编程语言类似

#### 三目运算符

AQL 还支持类似 C 语言的那种三目运算符，例如：

{% highlight AQL %}
u.active == true ? u.userId : null
{% endhighlight %}

#### 范围运算符

AQL 支持类似 Python、Ruby 一样的范围运算符 .. ，可以获得一个双闭区间，例如：

{% highlight AQL %}
2010..2013   // 得到 [ 2010, 2011, 2012, 2013 ]
{% endhighlight %}


## 数据查询

### 简单查询

使用 RETURN 语句可以返回结果， RETURN 的参数可以是一个表达式，也可以是一个文档的 id 或者句柄，例如：

{% highlight AQL %}
RETURN "Hello ArangoDB!"
RETURN DOCUMENT("users/phil")
RETURN {"name": "Fred", "age": 36}
{% endhighlight %}

注意！！ 一个 AQL 语句只能有一个 RETURN 语句！ 一次查询只能有一个语句，不能像 SQL 一样多条语句之间用分号分隔。

使用 FOR...IN 结构可以很方便的遍历集合，使用 FILTER 语句可以类似 SQL 的 WHERE 一样设置条件，使用 SORT 可以进行排序，使用 LIMIT 可以限制返回的记录条数，例如：

{% highlight AQL %}
FOR doc IN users
    FILTER doc.status == "active"
    SORT doc.name DESC
    LIMIT 10
{% endhighlight %}

### 增删改

使用 INSERT 可以插入一条记录，可以手动指定一个 _key ，如果没有，系统会自动生成。由于 ArangoDB 是 schema-less ，因此插入的记录结构可以不一样，例如：

{% highlight AQL %}
INSERT {
    firstName: "Anna",
    name: "Pavlova",
    profession: "artist"
} IN users

INSERT {
    _key: "PhilCarpenter",
    firstName: "Phil",
    name: "Carpenter",
    middleName: "G.",
    status: "inactive"
} IN users
{% endhighlight %}

使用 UPDATE 可以更新一个文档，例如：

{% highlight AQL %}
UPDATE "PhilCarpenter" WITH {
    status: "active",
    location: "Beijing"
} IN users
{% endhighlight %}

使用 REPLACE 可以替换一个文档，即将一个文档的所有属性都 UPDATE 了，例如：

{% highlight AQL %}
REPLACE {
    _key: "PhilCarpenter",
    firstName: "Natacha",
    name: "Leclerc",
    status: "active",
    level: "premium"
} IN users
{% endhighlight %}

删除一个文档用 REMOVE 语句，例如：

{% highlight AQL %}
REMOVE "GilbertoGil" IN users
//或者
REMOVE { _key: "GilbertoGil" } IN users
{% endhighlight %}

### 批量增删改

配合 FOR...IN 可以实现批量增删改，例如：

{% highlight AQL %}
FOR u IN users
    FILTER u.status == "not active"
    UPDATE u WITH { status: "inactive" } IN users
	
FOR u IN users
    INSERT u IN backup
	
FOR u IN users
    FILTER u.status == "deleted"
    REMOVE u IN backup
{% endhighlight %}

### OLD 与 NEW

AQL 引入了两个伪变量，NEW 指向新插入的或者更新后的文档，OLD 指向被删除的或者更新前的文档，例如：

{% highlight AQL %}
FOR i IN 1..100
    INSERT { value: i } IN test 
    RETURN NEW
	
FOR u IN users
    FILTER u.status == "deleted"
    REMOVE u IN users 
    RETURN OLD
	
FOR u IN users
    FILTER u.status == "not active"
    UPDATE u WITH { status: "inactive" } IN users 
    RETURN { old: OLD, new: NEW }
{% endhighlight %}

### 投影（选取属性）

很多时候我们不需要返回整个文档，而只需要返回某几个属性，这时候就只需要用 .属性名 或者 ["属性名"] 的方式即可。中括号中的属性名还可以是变量或者表达式。

### 数据修改操作注意事项

要对数据进行修改的查询，在一个查询中只能对一个集合进行一个修改操作，即在一个查询中不能对一个集合放置多个 REMOVE 或 UPDATE 语句。

一个 AQL 语句中只能有一个修改数据的操作，且数据修改的操作不能放置在子查询中。

数据修改语句之后可以接 LET 或 RETURN ，但不能接其他的语句。


## 关键字详解

### FOR

FOR 有两种用法，一是用在数组中，另一个是用在图的遍历中，其形式分别是：

{% highlight AQL %}
FOR variableName IN expression

FOR vertexVariableName, edgeVariableName, pathVariableName IN traversalExpression
{% endhighlight %}

FOR 也可以多层嵌套使用

### RETURN

当 RETURN 被放在 FOR 循环里使用的时候，返回的结果会构成一个数组。

RETURN 中可以使用动态属性名，只需要用中括号括起来就可以了，还可以使用 **MERGE** 来合并结果，但需要注意的是，使用 MERGE 的时候，相同的 key 的键值对只有一个会被保留，例如：

{% highlight AQL %}
RETURN MERGE(
  FOR u IN users
    RETURN { [ u._id ]: u.age }
)
/* 会得到类似
[
  {
    "users/10074": 69,
    "users/9883": 32,
    "users/9915": 27
  }
]  */
{% endhighlight %}

在 RETURN 的结果前加上 **DISTINCT** 关键字可以像 SQL 一样去重复。需要注意的是， RETURN DISTINCT 只能用在 FOR 循环中！

### FILTER

FILTER 关键字可以用多个条件，也可以用多个 FILTER 来给定多个条件，条件会按顺序判断，例如：

{% highlight AQL %}
FOR u IN users
  FILTER u.active == true
  SORT u.age ASC
  LIMIT 5
  FILTER u.gender == "f"
  RETURN u
{% endhighlight %}

### SORT

SORT 类似 SQL 的 ORDER BY ，可以对几个字段同时进行排序，默认是升序，如果需要降序则用 DESC ，例如：

{% highlight AQL %}
FOR u IN users
  SORT u.lastName, u.firstName, u.id DESC
  RETURN u
{% endhighlight %}

### LIMIT

LIMIT 类似 MySQL ，用法是： *LIMIT [offset], count* ，其中 offset 从 0 开始，表示略过前面多少条记录， count 表示返回多少条记录。

### LET

LET 用于给变量赋值，这个变量的作用域会存在于 LET 出现的地方，LET 通常用来定义相对复杂的计算以避免重复计算，例如：

{% highlight AQL %}
FOR u IN users
  LET numRecommendations = LENGTH(u.recommendations)
  RETURN { 
    "user" : u, 
    "numRecommendations" : numRecommendations, 
    "isPowerUser" : numRecommendations >= 10 
  }
{% endhighlight %}

LET 的还有一种常见的用法是定义子查询，提高代码的可读性，例如：

{% highlight AQL %}
FOR u IN users
  LET friends = (
  FOR f IN friends 
    FILTER u.id == f.userId
    RETURN f
  )
  RETURN { 
    "user" : u, 
    "friends" : friends, 
    "numFriends" : LENGTH(friends), 
    "memberShips" : memberships 
  }
{% endhighlight %}

### COLLECT

COLLECT 有点类似 SQL 的 GROUP BY ，来进行分组和聚集，它会淘汰掉当前作用域中除 COLLECT 本身引入的之外的所有本地变量！ 常见的语法形式有：

{% highlight AQL %}
COLLECT variableName = expression options
COLLECT variableName = expression INTO groupsVariable options
COLLECT variableName = expression INTO groupsVariable = projectionExpression options
COLLECT variableName = expression INTO groupsVariable KEEP keepVariable options
COLLECT variableName = expression WITH COUNT INTO countVariable options
COLLECT variableName = expression AGGREGATE variableName = aggregateExpression options
COLLECT AGGREGATE variableName = aggregateExpression options
COLLECT WITH COUNT INTO countVariable options
{% endhighlight %}

#### 分组

用 COLLECT 可以实现分组，如果用了 GROUP...INTO ，则不仅可以看到分组，还可以看到每个分组中的记录，例如：

{% highlight AQL %}
// 下面的查询显示有哪些城市
FOR u IN users
  COLLECT city = u.city
  RETURN { 
    "city" : city 
  }

// 下面的查询不但会显示城市，还会有一个数组显示在某个城市中的所有 users 的信息
FOR u IN users
  COLLECT city = u.city INTO groups
  RETURN { 
    "city" : city, 
    "usersInCity" : groups 
  }
{% endhighlight %}

同 GROUP BY 一样， COLLECT 还可以对多个属性进行分组，形如： COLLECT xxx = xxx, yyy = yyy

#### 分组后仅选取部分属性

默认情况下 COLLECT...INTO 会把文档的所有属性都选进变量中，如果只需要部分属性，则可以在变量中进行指定，甚至还可以进行运算，例如：

{% highlight AQL %}
FOR u IN users
  COLLECT country = u.country, city = u.city INTO groups = { 
    "name" : u.name,    // 选择需要的属性
    "isActive" : u.status == "active"  // 属性的值可以由计算获得
  }
  RETURN { 
    "country" : country, 
    "city" : city, 
    "usersInCity" : groups 
  }
{% endhighlight %}

如果在语句中使用了 LET 进行变量赋值，那么可以使用 **KEEP** 关键字搭配 COLLECT 来设置哪些变量可以进入最终的结果，例如：

{% highlight AQL %}
FOR u IN users
  LET name = u.name
  COLLECT city = u.city INTO groups keep name
  RETURN { 
    "city" : city, 
    "userNames" : groups[*].name  //将所有 name 拼成一个字符串数组而非对象的数组
  }
  
// 下面的查询与上面结果等价
FOR u IN users
  COLLECT city = u.city INTO groups=u.name
  RETURN { 
    "city" : city, 
    "userNames" : groups
  }
{% endhighlight %}

#### 分组计数

如果需要统计每个分组的记录个数，可以用 **WITH COUNT** 搭配 COLLECT ，例如：

{% highlight AQL %}
FOR u IN users
  COLLECT city = u.city WITH COUNT INTO length
  RETURN { 
    "city" : city, 
    "count" : length 
  }
{% endhighlight %}

#### 聚集函数

使用 **AGGREGATE** 关键字搭配聚集函数，可以完成一些类似最大值、最小值、平均值之类的聚集操作，例如：

{% highlight AQL %}
FOR u IN users
  COLLECT ageGroup = FLOOR(u.age / 5) * 5 
  AGGREGATE minAge = MIN(u.age), maxAge = MAX(u.age)
  RETURN {
    ageGroup, 
    minAge, 
    maxAge 
  }
{% endhighlight %}

在底层，COLLECT 会使用 SORT 或者 HASH 来进行处理。


### REMOVE

REMOVE 的语法是： *REMOVE keyExpression IN collection options* 。其中 keyExpression 可以是表达式，例如：

{% highlight AQL %}
FOR i IN 1..1000
  REMOVE { _key: CONCAT('test', i) } IN users

FOR u IN users
  FILTER u.active == false
  REMOVE { _key: u._key } IN backup
{% endhighlight %}

使用 REMOVE 的时候可能会遇到诸如文档不存在等错误，这时候就需要用 options 来指定处理方式，例如：

{% highlight AQL %}
FOR i IN 1..1000
  REMOVE { _key: CONCAT('test', i) } IN users OPTIONS { ignoreErrors: true }
  
// 下面的 options 保证在查询返回的时候修改已经被写入
FOR i IN 1..1000
  REMOVE { _key: CONCAT('test', i) } IN users OPTIONS { waitForSync: true }
{% endhighlight %}

在删除了文档后，伪变量 OLD 会保存删除的文档信息，例如：

{% highlight AQL %}
REMOVE keyExpression IN collection options RETURN OLD

FOR u IN users
  REMOVE u IN users 
  LET removed = OLD
  RETURN removed._key
{% endhighlight %}

### UPDATE

UPDATE 有两种语法形式： *UPDATE document IN collection options* 以及 *UPDATE keyExpression WITH document IN collection options* 。

使用第一种形式的时候， **必须包含 _key 属性** ！！ 第二种形式则不需要。例如：

{% highlight AQL %}
FOR u IN users
  UPDATE { _key: u._key, name: CONCAT(u.firstName, " ", u.lastName) } IN users
  
FOR u IN users
  UPDATE u._key WITH { name: CONCAT(u.firstName, " ", u.lastName) } IN users

// 与上面的等价
FOR u IN users
  UPDATE u WITH { name: CONCAT(u.firstName, " ", u.lastName) } IN users
{% endhighlight %}

如果在 UPDATE 的时候需要用到当前记录的值，则需要用一个 FOR 循环或者用 LET 赋值变量来获取。

UPDATE 除了除了 REMOVE 常用的 ignoreErrors 、 waitForSync 选项之外，还有一些其他的选项，比如：

  - keepNull ： 默认为 true ，如果设置为 false ，则 UPDATE 会把更新成 null 的字段直接删除
  - mergeObject ： 默认为 true ，如果设置为 false ，则 UPDATE 里面没有更新的属性会被删除

UPDATE 后会使用两个伪变量： NEW 和 OLD ，前者指向当前更新后的文档，后者指向更新前的文档。

### REPLACE

REPLACE 用来替换文档，其语法形式是： *REPLACE document IN collection options* 或 *REPLACE keyExpression WITH document IN collection options* 。

与 UPDATE 一样，REPLACE 的第一种形式需要有 _key 属性，例如：

{% highlight AQL %}
FOR u IN users
  REPLACE { _key: u._key, name: CONCAT(u.firstName, u.lastName) } IN users

FOR u IN users
  REPLACE u WITH { name: CONCAT(u.firstName, u.lastName) } IN users
{% endhighlight %}

REPLACE 也有 ignoreErrors 、 waitForSync 两个选项，并使用 OLD 和 NEW 两个伪变量。

### INSERT

INSERT 的语法形式是： *INSERT document IN collection options* ，例如：

{% highlight AQL %}
FOR i IN 1..100
  INSERT { value: i } IN numbers
{% endhighlight %}

REPLACE 也有 ignoreErrors 、 waitForSync 两个选项，并使用 NEW 伪变量指向新插入的文档。

### UPSERT

UPSERT 是 UPDATE 和 INSERT 的结合体，表示当文档存在的时候就对它 UPDATE ，如果不存在则进行 INSERT 。

每个 UPSERT 语句只能针对一个集合，且集合名称不能是动态的！其语法形式是：

{% highlight AQL %}
UPSERT searchExpression INSERT insertExpression UPDATE updateExpression IN collection options
UPSERT searchExpression INSERT insertExpression REPLACE updateExpression IN collection options
{% endhighlight %}

searchExpression 用来搜索相应的文档，必须为一个 object literal ，不能有动态的属性名称。如果找不到，就会插入 insertExpression 指定的文档；如果找到了，就会执行 updateExpression 进行更新。例如：

{% highlight language %}
UPSERT { name: 'superuser' } 
INSERT { name: 'superuser', logins: 1, dateCreated: DATE_NOW() } 
UPDATE { logins: OLD.logins + 1 } IN users
{% endhighlight %}

UPSERT 的 options 类似 UPDATE ，也会使用 OLD 和 NEW 两个伪变量。

### WITH

AQL 语句可以用 WITH 来开头，后面跟集合名，这样的话这些集合在 AQL 执行期间会被锁定。从 ArangoDB 3.1 开始，集群中进行遍历就必须要使用 WITH 以防止死锁。

注意： 如果查询时知道要用到哪些集合，那么没必要用 WITH ，只有当查询需要动态访问集合（比如遍历、最短路径）的时候才有必要使用 WITH 。

WITH 用法举例：

{% highlight AQL %}
WITH managers, usersHaveManagers
FOR v, e, p IN OUTBOUND 'users/1' GRAPH 'userGraph'
  RETURN { v, e, p }
{% endhighlight %}

WITH 还被用在 UPDATE、UPSERT 之中。


## AQL 中的图遍历查询

AQL 的图查询主要有两类，一类是遍历（Traversal），另一类是最短路径（Shortest Path）。

AQL 的图查询主要针对两种图，一种叫做 named graph，是 ArangoDB 管理的集合组成的命名图形，另一种就是用户自己指定的顶点和边的集合，二者的语法分别是：

{% highlight AQL %}
FOR vertex[, edge[, path]]
  IN [min[..max]]
  OUTBOUND|INBOUND|ANY startVertex
  GRAPH graphName
  [OPTIONS options]
  
FOR vertex[, edge[, path]]
  IN [min[..max]]
  OUTBOUND|INBOUND|ANY startVertex
  edgeCollection1, ..., edgeCollectionN
  [OPTIONS options]
{% endhighlight %}

### vertex edge & path

语句中的 vertex 代表一个顶点（文档），edge 代表一条表（实际也是文档），而 path 包含两个数组成员，其中 vertices 包含了路径中的所有定点，edges 包含了路径中的所有边。

### min..max

min 和 max 表示搜索的最小和最大的深度。min 的默认值为 1 ，max 的默认值为 min 。

### OUTBOUND|INBOUND|ANY

指定边的方向，是从 startVertex 出发还是汇入 startVertex ，亦或者两者都可以（any）。

### startVertex

搜索的起点，类型只能是字符串或者 object ，字符串代表一个文档的 id ，也可以用 object 的形式指定文档的 _id 。

### GRAPH

图的名称，指定后该图所包含的定点和边将会被检索。如果用第二种语法形式，也可以指定

### OPTIONS

选项有如下三个：

  - uniqueVertices：默认为 "none" ，如果为 "path" 则确保每条路径中不会有重复节点，如果为 "global" 则会在两个顶点中的多条边中随意选一条，即每个节点仅被访问一次。
  - uniqueEdges：默认为 "path" ，确保每条路径中不会有重复边，如果为 "none" 则不做重复性检查，如果为 "global" 则会保证每条边仅被访问一次。
  - bfs：默认为 false ，采用深度优先搜索，如果设置为 true ，则会采用广度优先搜索。

### 使用混合的方向

我们可以在给定边集合的时候分别指定方向，不指定方向的将会使用前面的 IN 所指定的方向，例如：

{% highlight AQL %}
FOR vertex IN OUTBOUND
  startVertex
  edges1, ANY edges2, edges3
{% endhighlight %}

### 使用 FILTER 进行剪枝

使用 FILTER 可以对定点、边、路径进行剪枝，例如：

{% highlight AQL %}
// 剪枝边
FOR v, e, p IN 1..5 OUTBOUND 'circles/A' GRAPH 'traversalGraph'
  FILTER p.edges[0].theTruth == true
  RETURN p
  
// 剪枝顶点
FOR v, e, p IN 1..5 OUTBOUND 'circles/A' GRAPH 'traversalGraph'
  FILTER p.vertices[1]._key == "G"
  RETURN p
  
// 剪枝整个路径，这时候需要用到 ALL 、NONE 以及 ANY
FOR v, e, p IN 1..5 OUTBOUND 'circles/A' GRAPH 'traversalGraph'
  FILTER p.edges[*].theTruth ALL == true    // 也可以用 NONE 或 ANY
  RETURN p
{% endhighlight %}

## AQL 中的图最短路径查询

与图遍历一样，最短路径查询的语法也有两种，分别是：

{% highlight AQL %}
FOR vertex[, edge]
  IN OUTBOUND|INBOUND|ANY SHORTEST_PATH
  startVertex TO targetVertex
  GRAPH graphName
  [OPTIONS options]
  
FOR vertex[, edge]
  IN OUTBOUND|INBOUND|ANY SHORTEST_PATH
  startVertex TO targetVertex
  edgeCollection1, ..., edgeCollectionN
  [OPTIONS options]
{% endhighlight %}

其他参数与图遍历类似，OPTIONS 有以下选择：

  - weightAttribute ： 字符串类型，指定用边所在的文档的哪一个属性来作为边的权重，如果不指定则使用 defaultWeight
  - defaultWeight ： 数值类型，指定默认权重，默认值为 1 。

与图遍历一样，在指定边的集合的时候也可以针对每一个集合指定边的方向。
