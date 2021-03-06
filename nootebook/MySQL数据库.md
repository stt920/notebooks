## 数据库基础

#### 基本概念

##### 主键外键

外键为某个表中的一列，它包含另一个表的主键，定义了两个表的关系。

##### 数据类型

1. 整型

   TINYINT, SMALLINT, MEDIUMINT, INT, BIGINT 分别使用 8, 16, 24, 32, 64 位存储空间，一般情况下越小的列越好。

   INT(11) 中的数字只是规定了交互工具显示字符的个数，对于存储和计算来说是没有意义的。

2. 浮点型

   FLOAT 和 DOUBLE 为浮点类型，DECIMAL 为高精度小数类型。CPU 原生支持浮点运算，但是不支持 DECIMAl 类型的计算，因此 DECIMAL 的计算比浮点类型需要更高的代价。

   FLOAT、DOUBLE 和 DECIMAL 都可以指定列宽，例如 DECIMAL(18, 9) 表示总共 18 位，取 9 位存储小数部分，剩下 9 位存储整数部分。

3. 字符串

   主要有 CHAR 和 VARCHAR 两种类型，一种是定长的，一种是变长的。

   VARCHAR 这种变长类型能够节省空间，因为只需要存储必要的内容。但是在执行 UPDATE 时可能会使行变得比原来长，当超出一个页所能容纳的大小时，就要执行额外的操作。MyISAM 会将行拆成不同的片段存储，而 InnoDB 则需要分裂页来使行放进页内。

   VARCHAR 会保留字符串末尾的空格，而 CHAR 会删除。

4. 日期和时间

   MySQL 提供了两种相似的日期时间类型：DATATIME 和 TIMESTAMP。

   - **DATATIME**

   能够保存从 1001 年到 9999 年的日期和时间，精度为秒，使用 8 字节的存储空间。

   它与时区无关。

   默认情况下，MySQL 以一种可排序的、无歧义的格式显示 DATATIME 值，例如“2008-01-16 22:37:08”，这是 ANSI 标准定义的日期和时间表示方法。

   - **TIMESTAMP**

   和 UNIX 时间戳相同，保存从 1970 年 1 月 1 日午夜（格林威治时间）以来的秒数，使用 4 个字节，只能表示从 1970 年 到 2038 年。

   它和时区有关。

   MySQL 提供了 FROM_UNIXTIME() 函数把 UNIX 时间戳转换为日期，并提供了 UNIX_TIMESTAMP() 函数把日期转换为 UNIX 时间戳。

   默认情况下，如果插入时没有指定 TIMESTAMP 列的值，会将这个值设置为当前时间。

   应该尽量使用 TIMESTAMP，因为它比 DATETIME 空间效率更高。

#### SQL语句

SQL是结构化查询语句（Structured Query Language）的缩写，其功能包括数据查询（DQL）、数据定义（DDL）、数据操作（DML）和数据控制（DCL）4个部分。

- DQL（Data Query Language）：数据查询语言，用来查询记录（数据）。
- DDL（Data Definition Language）：数据定义语言，用来定义数据库对象：库、表、列等； CREATE、 ALTER、DROP
- DML（Data Manipulation Language）：数据操作语言，用来定义数据库记录（数据）； INSERT、 UPDATE、 DELETE
- DCL（Data Control Language）：数据控制语言，用来定义访问权限和安全级别；

##### MySQL使用

```mysql
SHOW DATABASES;#显示数据库
USE databases;#选择数据库
SHOW TABLES;#显示所选择数据库表
SHOW COLUMNS FROM table;#显示表设计信息
```

##### 检索数据

```mysql
SELECT pro_name FROM products;#查找单个列
SELECT pro_name,prod_id,rod_price FROM products;#查找多列
SELECT * FROM products；#查找所有列
SELECT DISTINCT vend_id FROM products；#检索数据不重复

SELECT prod_name FROM products LIMIT 5;#检索结果不多于5行（第一行开始）
SELECT prod_name FROM products LIMIT 5,5;#检索从第五行开始的五行
```

##### 排序检索数据

```mysql
SELECT prod_name FROM products ORDER BY prod_name;#检索并排序
SELECT prod_id，prod_price,prod_name FROM products ORDER BY prod_price,prod_naem;#按多个列排序
SELECT prod_name FROM products ORDER BY prod_name DESC;#指定排序方向，DESC降序
SELECT pro_price FROM products ORDER BY prod_price DESC LIMIT 1;#ORDER BY 与 LIMIT 组合使用
```

##### 过滤数据

```mysql
SELECT prod_name,prod_price FROM products WHERE prod_price=2.5;#操作符：=、<>(不等于)、!=、<、>、<=、>=、BETWTEEN a AND b、IS NULL
```

##### 数据过滤

```mysql
SELECT prod_id,prod_name,prod_price FROM products WHERE vend_id==1003 AND prod_price<=10;#组合WHERE; AND、OR

SELECT prod_name,prod_price FROM products WHERE vend_id IN (1002,1003) ORDER BY prod_name;#指定条件范围检索；NOT IN
```

##### 通配符过滤

```mysql
SELECT prod_name,prod_price FROM products WHERE prod_name LIKE 'jie%';
#'%':任何字符出现任意次数
#'_':匹配单个任意字符

REGEXP 正则表达式匹配
```

##### 创建计算字段

```mysql
SELECT Concat(vend_name,'(',vend_country,')') FROM vendors ORDER BY vend_name;#拼接字段
SELECT Concat(RTrim(vend_name）,'(',RTrim（vend_country）,')') AS vend_title FROM vendors ORDER BY vend_name;#拼接字段,使用别名
SELECT pro_id,quantity,item_price, quantity*item_price AS expended_price FROM orderitems WHERE order_num=2005;#执行算数计算
```

##### 组合查询

可用UNION操作符来组合数条SQL查询。利用UNION可给出多条SELECT语句，将它们的结果组合成单个结果集。

```mysql
SELECT vend_id,prod_id,prod_price FROM products WHERE prod_price<=5 UNION SELECT vend_id,prod_id,prod_price FROM products WHERE vend_id IN (1001,1002);
```

##### 插入数据

插入一行数据：

```mysql
INSERT INTO coustomers（cust_name,
	cust_address,
	cust_city,
	cust_state.
	cust_zip,
	cust_country,
	cust_contact)
  VALUES('Pep E.LaPew',
        '100 main street',
        'Los Angeles',
        'CA',
        '90046',
        'USA',
        NULL);
```

插入多行数据：

```mysql
INSERT INTO coustomers（cust_name,
	cust_address,
	cust_city,
	cust_state.
	cust_zip,
	cust_country,
	cust_contact)
  VALUES('Pep E.LaPew',
        '100 main street',
        'Los Angeles',
        'CA',
        '90046',
        'USA',
        NULL)，#逗号隔开
 		('Pep E.LaPew',
        '100 main street',
        'Los Angeles',
        'CA',
        '90046',
        'USA',
        NULL);
```

插入检索出的数据

```mysql
INSERT INTO coustomers（cust_name,
	cust_address,
	cust_city,
	cust_state.
	cust_zip,
	cust_country,
	cust_contact)
   SELECT cust_name,
	cust_address,
	cust_city,
	cust_state.
	cust_zip,
	cust_country,
	cust_contact
   FROM custnew；
```

##### 更新与删除

```mysql
#更新
UPDATE customers
SET cust_name='The Fudds',
	cust_email='12345@123.com'
WHERE cust_id=1005;

#删除
DELETE FROM customers WHERE cust_id=1006;#省略WHERE子句则删除整个表的内容，但不删除表

#DELETE不需要列名或通配符。DELETE删除整行而不是删除列。为了删除列，使用UPDATE语句
UPDATE customers SET cust_email=NULL WHERE cust_id=1005;#其中NULL用来删除cust_email列中所有值

#更快的删除，删除表中数据，不删除表
TRUNCATE TABLE#速度比delete更快

```

##### 创建和操纵表

```mysql
#创建表
CREATE TABLE customers
（
	cust_id		  int 	   NOT NULL	 AUTO_INCREMENT,
	cust_name	  char(50) NOT NULL,
	cust_address  char(50)  NULL,
	cust_city     char(50)  NULL,
	cust_state    char(5)   NULL,
	PRIMARY KEY (cust_id)
）ENGINE=InnoDB;
#AUTO_INCREMENT 值自动增加
```

```mysql
#DEFAULT 指定默认值
CREATE TABLE customers
（
	cust_id		  int 	   NOT NULL	 AUTO_INCREMENT,
	cust_name	  char(50) NOT NULL,
	cust_address  char(50)  NULL	DEFAULT china,
	cust_city     char(50)  NULL,
	cust_state    char(5)   NULL,
	PRIMARY KEY (cust_id)
）ENGINE=InnoDB;
```

```mysql
#更新表
ALTER TABLE vendors ADD vend_phone CHAR(20);#添加列
ALTER TABLE vendors DROP COLUMN vend_phone;#删除列
```

```mysql
#删除表
DROP TABLE coustomers；
```

```mysql
#重命名表
RENAME TABLE customers2 TO customers;
```

##### 视图

```mysql
CREATE VIEW productcustomers AS SELECT cust_name,cust_contact,prod_id FROM customers,orders,orderitems WHERE customers.cust_id=orders.cust_id;
```

##### 内连接与外连接

1.内连接,显示两个表中有联系的所有数据;

2.左链接,以左表为参照,显示所有数据;

3.右链接,以右表为参照显示数据;

- **内连接**

内连接又称等值连接，使用 INNER JOIN 关键字。

```mysql
select a, b, c
from A inner join B
on A.key = B.key
```

可以不明确使用 INNER JOIN，而使用普通查询并在 WHERE 中将两个表中要连接的列用等值方法连接起来。

```mysql
select a, b, c
from A, B
where A.key = B.key
```

- **外连接**

外连接保留了没有关联的那些行。分为左外连接，右外连接以及全外连接，左外连接就是保留左表没有关联的行。

检索所有顾客的订单信息，包括还没有订单信息的顾客。

```sql
select Customers.cust_id, Orders.order_num
from Customers left outer join Orders
on Customers.cust_id = Orders.curt_id;
```

#### 数据库引擎

##### MyISAM与InnoDB

**1.区别**

MySQL默认是MyISAM;

**（1）事务处理：**

MyISAM不支持事务处理，InnoDB支持事务处理；

**（2）锁机制不同：**

MyISAM是表级锁，而InnoDB是行级锁；

**（3）select ,update ,insert ,delete 操作：**

MyISAM：如果执行大量的SELECT，MyISAM是更好的选择

InnoDB：如果你的数据执行大量的INSERT或UPDATE，出于性能方面的考虑，应该使用InnoDB表

**（4）查询表的行数不同：**

MyISAM：select count() from table,MyISAM只要简单的读出保存好的行数，注意的是，当count()语句包含   where条件时，两种表的操作是一样的

InnoDB ： InnoDB 中不保存表的具体行数，也就是说，执行select count(*) from table时，InnoDB要扫描一遍整个表来计算有多少行

**（5）外键支持：**

MyISAM表不支持外键，而InnoDB支持

**2. MyISAM比Innodb 查询速度快。**

INNODB在做SELECT的时候，要维护的东西比MYISAM引擎多很多；
1）数据块，INNODB要缓存，MYISAM只缓存索引块，  这中间还有换进换出的减少； 
2）innodb寻址要映射到块，再到行，MYISAM 记录的直接是文件的OFFSET，定位比INNODB要快
3）INNODB还需要维护MVCC一致；虽然你的场景没有，但他还是需要去检查和维护

MVCC ( Multi-Version Concurrency Control )多版本并发控制 

**3. 应用场景**

**MyISAM适合：**

(1)做很多count 的计算；(2)插入不频繁，查询非常频繁；(3)没有事务。

**InnoDB适合：**

(1)可靠性要求比较高，或者要求事务；(2)表更新和查询都相当的频繁，并且行锁定的机会比较大的情况。

#### 索引

​	索引是对数据库表中一列或多列的值进行排序的一种结构，使用索引可快速访问数据库表中的特定信息。合理的创建索引能够提升查询语句的执行效率，但降低了新增、删除操作的速度，同时也会消耗一定的数据库物理空间。

##### 索引分类

**普通索引、唯一索引、主键索引、组合索引、全文索引**

1.普通索引

是最基本的索引，它没有任何限制。它有以下几种创建方式：
（1）直接创建索引

```sql
CREATE INDEX index_name ON table(column(length))
```

（2）修改表结构的方式添加索引

```sql
ALTER TABLE table_name ADD INDEX index_name ON (column(length))
```

（3）创建表的时候同时创建索引

```sql
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER NOT NULL ,
    `content` text CHARACTER NULL ,
    `time` int(10) NULL DEFAULT NULL ,
    PRIMARY KEY (`id`),
    INDEX index_name (title(length))
)
```

（4）删除索引

```sql
DROP INDEX index_name ON table
```

2.唯一索引

​	与前面的普通索引类似，不同的就是：索引列的值必须唯一，但允许有空值。如果是组合索引，则列值的组合必须唯一。它有以下几种创建方式：

（1）创建唯一索引

```sql
CREATE UNIQUE INDEX indexName ON table(column(length))
```

（2）修改表结构

```sql
ALTER TABLE table_name ADD UNIQUE indexName ON (column(length))
```

（3）创建表的时候直接指定

```sql
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER NOT NULL ,
    `content` text CHARACTER NULL ,
    `time` int(10) NULL DEFAULT NULL ,
    UNIQUE indexName (title(length))
);
```

3.主键索引

是一种特殊的唯一索引，一个表只能有一个主键，不允许有空值。一般是在建表的时候同时创建主键索引：

```sql
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) NOT NULL ,
    PRIMARY KEY (`id`)
);
```

4.组合索引

​	指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用组合索引时**遵循最左前缀集合**

​	复合索引还有一个优点，它通过被称为“最左前缀”（leftmost prefixing）的概念体现出来的。假设向一个表的多个字段（例如fristname、lastname、address）创建复合索引（索引名为fname_lname_address）.当where查询条件是以下各种字段的组合是，MySQL将使用fname_lname_address索引。其他情况将无法使用fname_lname_address索引。可以理解：一个复合索引（firstname、lastname、address）等效于（firstname，lastname，address）、（firstname，lastname）以及（firstname）三个索引。基于最做前缀原则，应尽量避免创建重复的索引，例如，创建了fname_lname_address索引后，就无需再first_name子段上单独创建一个索引

```sql
ALTER TABLE `table` ADD INDEX name_city_age (name,city,age); 
```

5.全文索引

​	主要用来查找文本中的关键字，而不是直接与索引中的值相比较。fulltext索引跟其它索引大不相同，它更像是一个搜索引擎，而不是简单的where语句的参数匹配。fulltext索引配合**match against**操作使用，而不是一般的where语句加like。它可以在create table，alter table ，create index使用，不过目前只有char、varchar，text 列上可以创建全文索引。值得一提的是，在数据量较大时候，现将数据放入一个没有全局索引的表中，然后再用CREATE index创建fulltext索引，要比先为一张表建立fulltext然后再将数据写入的速度快很多。

（1）创建表的适合添加全文索引

```sql
CREATE TABLE `table` (
    `id` int(11) NOT NULL AUTO_INCREMENT ,
    `title` char(255) CHARACTER NOT NULL ,
    `content` text CHARACTER NULL ,
    `time` int(10) NULL DEFAULT NULL ,
    PRIMARY KEY (`id`),
    FULLTEXT (content)
);
```

（2）修改表结构添加全文索引

```sql
ALTER TABLE article ADD FULLTEXT index_content(content)
```

（3）直接创建索引

```sql
CREATE FULLTEXT INDEX index_content ON article(content)
```

##### 聚集索引和非聚集索引

##### 使用B+树实现

##### 索引优缺点

优点：

1.  通过创建唯一性索引，可以保证数据库表中每一行数据的唯一性。
2.  可以大大**加快数据的检索速度**，这也是创建索引的最主要的原因。
3. 可以加速表和表之间的连接，特别是在实现数据的参考完整性方面特别有意义。
4. 在使用分组和排序子句进行数据检索时，同样可以显著减少查询中分组和排序的时间。
5. 通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。 

缺点：

1. **创建索引和维护索引要耗费时间**，这种时间随着数据量的增加而增加。
2. **索引需要占物理空间**，除了数据表占数据空间之外，每一个索引还要占一定的物理空间，如果要建立聚簇索引，那么需要的空间就会更大。
3.   当对表中的数据进行增加、删除和修改的时候，**索引也要动态的维护**，这样就降低了数据的维护速度。

##### 索引创建原则

适合条件：

1. 在**经常需要搜索**的列上，可以加快搜索的速度；
2. 在作为**主键**的列上，强制该列的唯一性和组织表中数据的排列结构；
3. 在经常用在连接的列上，这些列主要是一些**外键**，可以加快连接的速度；
4.  在经常需要根据范围进行搜索的列上创建索引，因为索引已经排序，其指定的范围是连续的；
5. 在经常**需要排序**的列上创建索引，因为索引已经排序，这样查询可以利用索引的排序，加快排序查询时间；
6. 在经常使用在**WHERE子句**中的列上面创建索引，加快条件的判断速度。

不适条件：

1. 在查询中很少使用或者参考的列不应该创建索引。既然这些列很少使用到，因此有索引或者无索引，并不能提高查询速度。相反，由于增加了索引，反而降低了系统的维护速度和增大了空间需求。
2. 只有很少数据值的列也不应该增加索引。由于这些列的取值很少，例如人事表的性别列，在查询的结果中，结果集的数据行占了表中数据行的很大比例，即需要在表中搜索的数据行的比例很大。增加索引，并不能明显加快检索速度。
3. 对于那些定义为text, image和bit数据类型的列不应该增加索引。这些列的数据量要么相当大，要么取值很少。
4. 当修改性能远远大于检索性能时，不应该创建索引。修改性能和检索性能是互相矛盾的。当增加索引时，会提高检索性能，但是会降低修改性能。当减少 索引时，会提高修改性能，降低检索性能。因此，当修改性能远远大于检索性能时，不应该创建索引。

#### [MySQL优化](https://www.nowcoder.com/discuss/20591)

1. 选择合适的引擎

- MyISAM 索引顺序访问方法，支持全文索引，非事务安全，不支持外键，会加表级锁

```
三个文件：
    FRM 存放表结构
    MYD 存放数据
    MYI 存放索引
```

- InnoDB 事务型存储引擎，加行锁，支持回滚，崩溃恢复，ACID事务控制，表和索引放在一个表空间里头，表空间多个文件。

```
例：
    update tableset age=3 where name like "%jeff%";
    //会锁表
```

2. 正确使用索引

- 给合适的列表建立索引，给where子句，连接子句建立索引，而不是select选择列表  
- 索引值应该不相同，唯一值时效果最好，大量重复效果很差  
- 使用短索引，指定前缀长度`char(50)`的前20，30值唯一`例：文件名`；索引缓存一定（小）时，存的索引多，消耗IO更小，能提高查找速度  
- 最左前缀n列索引，最左列的值匹配，更快。  
- like查询，索引会失效，尽量少用like。百万、千万数据时，用like Sphinx开源方案结合MySQL  
- 不能滥用索引

```
1.索引占用空间
2.更新数据，索引必须更新，时间长，尽量不要用在长期不用的字段上建立索引
3.SQL执行一个查询语句，增加查询优化的时间
```

3. 避免使用`SELECT *`

- 返回结果过多，降低查询的速度  
- 过多的返回结果，会增大服务器返回给APP端的数据传输量。例：`网络传输速度面，弱网络环境下，容易造成请求失效`

4. 字段尽量设置为NOT NULL

```
"" 和 NULL

{"name":"myf"} {"name":""} {"hobby":空array}

NULL占空间
    例：安卓需要判断""还是NULL
    Java和OC都是强类型，会造成APP闪退
```

#### [范式](https://blog.csdn.net/zymx14/article/details/69789326)

- 1NF： 字段是最小的的单元不可再分 
- 2NF：满足1NF,表中的字段必须完全依赖于全部主键而非部分主键 
- 3NF：满足2NF,非主键外的所有字段必须互不依赖
- 巴斯-科德范式（BCNF）:在3NF基础上，任何非主属性不能对主键子集依赖（在3NF基础上消除对主码子集的依赖）
- ​4NF

1. **第一范式**（1NF）：确保每一列的原子性

   如果每一列都是不可再分的最小数据单元，则满足第一范式。

   | id   | 地址     |
   | ---- | -------- |
   | 1    | 中国广东 |
   | 2    | 中国云南 |

   上面的表地址字段其实可以继续分：

   | id   | 国家 | 省份 |
   | ---- | ---- | ---- |
   | 1    | 中国 | 广东 |
   | 2    | 中国 | 云南 |

   但是具体地址到底要不要拆分 还要看具体情形，比如看看将来会不会按国家或者省市进行分类汇总或者排序，如果需要，最好就拆，如果不需要而仅仅起字符串的作用，可以不拆，操作起来更方便。

2. **第二范式**：非键字段必须依赖于键字段

   如果一个关系满足1NF，并且除了主键以外的其它列，都依赖与该主键，则满足二范式(2NF)，第二范式要求每个表只描述一件事。

   例如：

   | 字段     | 例子     |
   | -------- | -------- |
   | 订单编号 | 001      |
   | 产品编号 | a011     |
   | 订购日期 | 2017-4-8 |
   | 价格     | ￥30     |

   而实际上，产品编号与订单编号并没有明确的关系，订购日期与订单编号有关系，因为一旦订单编号确定下来了，订购日期也确定了，价格与订单编号也没有直接关系，而与产品有关，所以上面的表实际上可以拆分：

   订单表：

   | 订单编号 | 001      |
   | -------- | -------- |
   | 日期     | 2017-4-8 |

   产品表：

   | 产品编号 | a011 |
   | -------- | ---- |
   | 价格     | ￥30 |

3. **第三范式**：在1NF基础上，除了主键以外的其它列都不传递依赖于主键列，或者说： 任何非主属性不依赖于其它非主属性

   （在2NF基础上消除传递依赖）

   例如：	

   | 字段     | 例子     |
   | -------- | -------- |
   | 订单编号 | 001      |
   | 订购日期 | 2017-4-8 |
   | 顾客编号 | a01      |
   | 顾客姓名 | howard   |

   上面的满足第一和第二范式，但是不满足第三范式，原因如下：

   通过顾客编号可以确定顾客姓名，通过顾客姓名可以确定顾客编号，即在这个订单表里，这两个字段存在传递依赖，只需要一个就够了。

   又如：	

   | 主键 | 学号 | 姓名   | 成绩 |
   | ---- | ---- | ------ | ---- |
   | 1    | 111  | howard | 90   |
   | 2    | 222  | tom    | 90   |

   上面的表，学号和姓名存在传递依赖，因为(学号，姓名)->成绩，学号->成绩，姓名->成绩。所以学号和姓名有一个冗余了，只需要保留一个。

#### [事务](https://www.cnblogs.com/fjdingsd/p/5273008.html)

 	所谓事务，是指用户的一个数据库操作序列，这些操作要么全做要么全不做，是一个不可分割的工作。事物具有四个特性：原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）和持续性（Durability）。这四种特性简称未ACID特性。

##### 事务ACID特性

- **原子性**：整个事务中的所有操作，要么全部完成，要么全部不完成，不可能停滞在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
- **一致性**：在事务开始之前和事务结束以后，数据库的完整性约束没有被破坏。
- **隔离性**：隔离状态执行事务，使它们好像是系统在给定时间内执行的唯一操作。如果有两个事务，运行在相同的时间内，执行相同的功能，事务的隔离性将确保每一事务在系统中认为只有该事务在使用系统。这种属性有时称为串行化，为了防止事务操作间的混淆，必须串行化或序列化请求，使得在同一时间仅有一个请求用于同一数据。
- **持久性**：在事务完成以后，该事务所对数据库所作的更改便持久的保存在数据库之中，并不会被回滚。

##### 事务并发性问题

​	当多个线程都开启事务操作数据库中的数据时，数据库系统要能进行隔离操作，以保证各个线程获取数据的准确性，在介绍数据库提供的各种隔离级别之前，如果不考虑事务的隔离性，会发生的几种问题：

1. **脏读**

　　脏读是指在一个事务处理过程里读取了另一个未提交的事务中的数据。

　　当一个事务正在多次修改某个数据，而在这个事务中这多次的修改都还未提交，这时一个并发的事务来访问该数据，就会造成两个事务得到的数据不一致。例如：用户A向用户B转账100元，对应SQL命令如下

```sql
    update account set money=money+100 where name=’B’;  (此时A通知B)

    update account set money=money - 100 where name=’A’;
```

　　当只执行第一条SQL时，A通知B查看账户，B发现确实钱已到账（此时即发生了脏读），而之后无论第二条SQL是否执行，只要该事务不提交，则所有操作都将回滚，那么当B以后再次查看账户时就会发现钱其实并没有转。

2. **不可重复读**

　　不可重复读是指在对于数据库中的某个数据，一个事务范围内多次查询却返回了不同的数据值，这是由于在查询间隔，被另一个事务修改并提交了。

　　例如事务T1在读取某一数据，而事务T2立马修改了这个数据并且提交事务给数据库，事务T1再次读取该数据就得到了不同的结果，发送了不可重复读。

　　不可重复读和脏读的区别是，脏读是某一事务读取了另一个事务未提交的脏数据，而不可重复读则是读取了前一事务提交的数据。

　　在某些情况下，不可重复读并不是问题，比如我们多次查询某个数据当然以最后查询得到的结果为主。但在另一些情况下就有可能发生问题，例如对于同一个数据A和B依次查询就可能不同，A和B就可能打起来了……

3. **虚读(幻读)**

　　幻读是事务非独立执行时发生的一种现象。例如事务T1对一个表中所有的行的某个数据项做了从“1”修改为“2”的操作，这时事务T2又对这个表中插入了一行数据项，而这个数据项的数值还是为“1”并且提交给数据库。而操作事务T1的用户如果再查看刚刚修改的数据，会发现还有一行没有修改，其实这行是从事务T2中添加的，就好像产生幻觉一样，这就是发生了幻读。

　　幻读和不可重复读都是读取了另一条已经提交的事务（这点就脏读不同），所不同的是不可重复读查询的都是同一个数据项，而幻读针对的是一批数据整体（比如数据的个数）。

**小结：不可重复读的和幻读很容易混淆，不可重复读侧重于修改，幻读侧重于新增或删除。解决不可重复读的问题只需锁住满足条件的行，解决幻读需要锁表**

##### 事务四种隔离级别

- 未提交读

  事务中的修改，即使没有提交，对其他事务也是可见的。

- 提交读

  一个事务读取已经提交的事务所做的修改。即，一个事务所做的修改在提交前对其他事务是不可见的。

- 可重复读

  保证在同一个事务中多次读取多样的数据的结果是一样的。

- 可串行化

  强制事务串行执行。

| 事务隔离级别                 | 脏读 | 不可重复读 | 幻读 |
| ---------------------------- | ---- | ---------- | ---- |
| 未提交读（read-uncommitted） | 是   | 是         | 是   |
| 提交读（read-committed）     | 否   | 是         | 是   |
| 可重复读（repeatable-read）  | 否   | 否         | 是   |
| 序列化（serializable）       | 否   | 否         | 否   |

*mysql默认的事务隔离级别为repeatable-read

#### 存储过程

​	SQL语句执行的时候要先编译，然后被执行。在大型数据库中，为了提高效率，将为了完成特定功能的SQL语句集进行编译优化后，存储在数据库服务器中，用户通过特定存储过程的名字来调用执行。

```MYSQL
	#创建存储过程常用语法如下：
		creat procedure sp_name @[参数名][类型]
								as
								begin
								……
								end
    #调用存储过程语法：CALL sp_name[参数名]
   # 删除存储过程语法：drop procedure sp_name
```

​	使用存储过程可以增强SQL语言的功能和灵活性，由于可以用流程控制语句编写存储过程，有很强的灵活性，所以可以完成复杂的判断和运算，并且可以保证数据的安全性和完整性，同时存储过程可以使没有权限的用户在控制之下间接地存取数据库，保证了数据的安全。

​	但存储过程不等于函数，两者虽然本质上没有区别，但具体而言有以下几个方面的区别：

```
1. 存储过程一般是作为一个独立的部分来执行的，而函数可以作为查询语句的一个部分来调用。由于函数可以返回一个对象，因此它可以在查询语句中位于From关键字后面。
2. 一般而言，存储过程的功能较复杂，而函数实现的功能针对性比较强。
3. 函数需要用括号包住输入的参数，且返回一个值或表对象，存储过程可以返回多个参数。
4. 函数可以嵌入在SQL中使用，可以在select中调用，存储过程不可以。
5. 函数不能直接操作实体表，只能操作内建表。
6. 存储过程在创建时即在服务器上进行编译，执行速度更快。
```

#### 视图

- 视图：是从一个或多个表导出的虚拟的表，其内容由查询定义。具有普通表的结构，但是不实现数据存储。
- 对视图的修改：单表视图一般用于查询和修改，会改变基本表的数据，多表视图一般用于查询，不会改变基本表的数据。

```mysql
--创建视图--  
create or replace view v_student as select * from student;  
--从视图中检索数据--  
select * from v_student;  
--删除视图--  
drop view v_student;  
```

**作用：**

1. 简化了操作，把经常使用的数据定义为视图。

   ​	我们在使用查询时，在很多时候我们要使用聚合函数，同时还要 显示其它字段的信息，可能还会需要关联到其它表，这时写的语句可能会很长，如果这个动作频繁发生的话，我们可以创建视图，这以后，我们只需要select * from view就可以了，这样很方便。 

2. 安全性，用户只能查询和修改能看到的数据。

   ​	因为视图是虚拟的，物理上是不存在的，只是存储了数据的集合，我们可以将基表中重要的字段信息，可以不通过视图给用户，视图是动态的数据的集合，数据是随着基表的更新而更新。同时，用户对视图不可以随意的更改和删除，可以保证数据的安全性。 

3. 逻辑上的独立性，屏蔽了真实表的结构带来的影响。

   ​	视图可以使应用程序和数据库表在一定程度上独立。如果没有视图，应用一定是建立在表上的。有了视图之后，程序可以建立在视图之上，从而程序与数据库表被视图分割开来。

**缺点: **

1. 性能差

   ​	数据库必须把视图查询转化成对基本表的查询，如果这个视图是由一个复杂的多表查询所定义，那么，即使是视图的一个简单查询，数据库也要把它变成一个复杂的结合体，需要花费一定的时间。 

2. 修改限制

   ​	当用户试图修改视图的某些信息时，数据库必须把它转化为对基本表的某些信息的修改，对于简单的视图来说，这是很方便的，但是，对于比较复杂的试图，可能是不可修改的。



#### 触发器

​	触发器是一种特殊的存储过程，它由事件触发，而不是程序调用或手工启动。当数据库有特殊的操作时，对这些操作由数据库中的事件来触发，自动完成这些SQL语句。使用触发器可以用来保证数据的有效性和完整性，完成比约束更复杂的数据约束。

​	根据SQL语句不同，触发器可分为两类：DML触发器和DLL触发器。

​	DML触发器是当数据库服务器发生数据操作语言事件执行的存储过程，有After和Instead of两种触发器。After触发器被激活触发是在记录改变之后进行的一种触发器。Instead Of触发器是记录变更之前，去执行触发器本身定位的操作，而不是执行原来SQL语句里的操作。DLL触发器是在响应数据定义语言事件执行的存储过程。

#### [数据库锁](https://www.cnblogs.com/luyucheng/p/6297752.html)

​	数据库锁定机制简单来说，就是数据库为了保证数据的一致性，而使各种共享资源在被并发访问变得有序所设计的一种规则。对于任何一种数据库来说都需要有相应的锁定机制，所以MySQL自然也不能例外。MySQL数据库由于其自身架构的特点，存在多种数据存储引擎，每种存储引擎所针对的应用场景特点都不太一样，为了满足各自特定应用场景的需求，每种存储引擎的锁定机制都是为各自所面对的特定场景而优化设计，所以各存储引擎的锁定机制也有较大区别。MySQL各存储引擎使用了三种类型（级别）的锁定机制：**表级锁定，行级锁定和页级锁**定。

#### 数据库日志

日志文件记录所有对数据库数据的修改，主要是保护数据库防止故障，以及恢复数据时使用。特点如下：

      1. 每一个数据库至少包含两个日志文件组。每个日志文件组至少包含两个日志文件成员。
      2. 日志文件组以循环方式进行写操作。
3. 每一个日志文件对应一个物理文件。

通过日志文件来记录数据库事物可以最大限度保证数据库的一致性与安全性，但一旦数据库中日志满了，就只能执行查询等操作，不能执行更改、备份等操作。其原因是任何写操作都要记录日志，也就是说基本上处于不能使用的状态。
