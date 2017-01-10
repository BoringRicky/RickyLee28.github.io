---
layout: post
title: Android SQLite 介绍与简单使用(一)
tags:
- Android
categories: Android SQLite
description: Android SQLite 介绍与简单使用
---

SQLite 是一款小巧的嵌入式数据库，在Android和IOS中使用的数据库就是SQLite，它用的内存很少。并且它支持支持 SQL92（SQL2）标准的大多数查询语言的功能。而且它不需要配置，很容易使用。

### SQLite命令
#### DDL--数据定义语言
- CREATE:创建一个新表，或者其他对象
- ALERT: 修改数据库中的某个已有对象
- DROP：删除整个表或其他对象

#### DML--数据操作语言
- INSERT：插入一条记录
- UPDATE：修改记录
- DELETE：删除记录

#### DQL--数据查询语言
- SELECT：从一个或多表中检索记录

SQLite不区分大小写，但也有一些命令是大小写敏感的，比如GLOB和glob。

### SQLite 存储类和数据类型
####存储类
- **NULL**：值是一个 NULL 值。
- **INTEGER**：值是一个带符号的整数，根据值的大小存储在 1、2、3、4、6 或 8 字节中。
- **REAL**：值是一个浮点值，存储为 8 字节的 IEEE 浮点数字。
- **TEXT**：值是一个文本字符串，使用数据库编码（UTF-8、UTF-16BE 或 UTF-16LE）存储。
- **BLOB**：值是一个 blob 数据，完全根据它的输入存储。

存储类比数据类型更一般化。比如INTEGER存储类，包括6种不同长度的不同整形数据类型，这在磁盘上造成了差异。但是只要INTEGER值被从磁盘读出进入到内存进行处理，它们被转换成最一般的数据类型（8-字节有符号整形）。

#### 数据类型
SQLite支持‘类型近似’的观点，我们建表时给定的类型其实是推荐类型，这个不是必须的。SQLite的列可以存储任意类型数据，我可以认为它是无类型的。但是我们为了看起来更直观，更方便，我们还是应该给每个列设置类型。

每个sqlite3数据库中的列都被赋予下面类型近似中的一种：

- TEXT
- NUMERIC
- INTEGER
- REAL
- NONE

上面的这些类型其近似类型都比较多：

跟INTEGER近似的类型有：

- INT
- INTEGER
- TINYINT
- SMALLINT
- MEDIUMINT
- BIGINT
- UNSIGNED BIG INT
- INT2
- INT8

如果我们声明的泪习惯包含 ‘INT’字符，那么这个列就会被定义为 INTEGER近似，数据库存储时会按照INTEGER类型存储。

跟 TEXT 近似的类型有：

- CHARACTER(20)
- VARCHAR(255)
- VARYING CHARACTER(255)
- NCHAR(55)
- NATIVE CHARACTER(70)
- NVARCHAR(100)
- TEXT
- CLOB

如果这个列的声明类型包含”CHAR”，”CLOB”，或者”TEXT”中的任意一个，那么这个列就有了TEXT近似。数据库存储时会按照 TEXT 类型存储。

跟 NONE 近似的类型有：

- BLOB
- no datatype specified

如果列的声明类型中包含了字符串”BLOB”或者没有为其声明类型，这个列被赋予NONE近似。

跟 REAL 近似的类型有：

- REAL
- DOUBLE
- DOUBLE PRECISION
- FLOAT
 
跟NUMERIC近似的有：

- NUMERIC
- DECIMAL(10,5)
- BOOLEAN
- DATE
- DATETIME

在SQLite里没有Boolean类型，我们可以使用整数 0(false)或者1(true)表示。

### 创建数据库
在Android中使用的SQLite3,我们去Android SDK 目录下platform-tools文件夹就可以找到名为sqlite3.exe的文件。我们进入到这个目录下，然后使用：
	
	sqlite3 user.db

这句命令就可以创建一个名为 user 的数据库。

### 创建数据库表
创建数据库表使用 `create table` 语句。具体语法如下

	CREATE TABLE  表名(
	   column1 datatype  PRIMARY KEY(one or more columns),
	   column2 datatype,
	   column3 datatype,
	   .....
	   columnN datatype,
	);

数据库的表名必须是当前数据库中的唯一标识。例如：

	create table user ( 
		_id integer primary key autoincrement , 
		name text ,
		age integer , 
		address text
	);

上面的语句创建了一个数据库表，表名为user,这个表中有 _id,name,age,address 四个字段；其中_id 的是自增的。
### 插入数据
往数据库表里插入数据，使用 `insert into` 语句,语法如下：

	INSERT INTO TABLE_NAME (column1, column2, column3,...columnN)]
	VALUES (value1, value2, value3,...valueN);

示例：

	insert into user (name,age,address) values ('张三丰',20,'火星一号');

### 删除数据
在数据库表里删除数据，使用 `delete from` 语句,语法如下：

	DELETE FROM table_name WHERE [condition];

	delete from 表名 ;  删除表里的所有数据
	delete from 表名 where 条件； 根据条件的删除指定数据

示例：
	
	delete from user where _id = 7;

删除_id = 7 的列。

### 修改数据
修改数据库表中的某些数据使用 `update...set` 语句,语法如下：
	
	UPDATE table_name
	SET column1 = value1, column2 = value2...., columnN = valueN
	WHERE [condition];

示例：

	update user set name=张三',age=19,address='火星3号' where _id=3;

### 查询数据
查询数据库表中的数据使用 `select`语句，语法如下：

	SELECT column1, column2, columnN
	FROM table_name
	WHERE [CONTION | EXPRESSION];

示例：

	select * from user ； 查询user 表中的所有数据
	select name,age,address from user where _id = 1；查询user表中_id=1的列

查询语句稍微复杂一些，这里只做简单介绍。有兴趣可以自己研究一下。


