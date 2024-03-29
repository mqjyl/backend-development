# 索引

目前大部分数据库系统及文件系统都采用B-Tree(B树)或其变种B+Tree(B+树)作为索引结构。红黑树等数据结构也可以用来实现索引，但B+Tree是数据库系统实现索引的首选数据结构。在MySQL中，索引属于存储引擎级别的概念，不同存储引擎对索引的实现方式是不同的。

## :pencil2: 1、分类

### :pen\_fountain: 1.1、实现方式

&#x20;按照实现方式可分为**B+Tree索引和hash索引。**

`MyISAM`和`InnoDB`存储引擎：只支持`BTREE`索引， 也就是说默认使用`BTREE`，不能够更换；`MEMORY/HEAP`存储引擎：支持`HASH`和`BTREE`索引。

### :pen\_fountain: 1.2、使用方式

#### :hamster: 1.2.1、单列索引

一个索引只包含单个列，但一个表中可以有多个单列索引。&#x20;

1. 普通索引：MySQL中基本索引类型，没有什么限制，允许在定义索引的列中插入重复值和空值，纯粹为了查询数据更快一点；
2. 唯一索引：索引列中的值必须是唯一的，但是允许为空值；
3. 主键索引：是一种特殊的唯一索引，不允许有空值。

#### :hamster: 1.2.2、组合索引

在表中的多个字段组合上创建的索引，只有在查询条件中使用了这些字段的左边字段时，索引才会被使用，使用组合索引时遵循最左前缀集合。

#### :hamster: 1.2.3、全文索引

`MySQL 5.6`中的`InnoDB`引擎已经开始支持全文索引了，在此之前只有在`MyISAM`引擎上才能使用，只能在`CHAR`,`VARCHAR`,`TEXT`类型字段上使用全文索引。全文索引，就是在一堆文字中，通过其中的某个关键字等，就能找到该字段所属的记录行。

#### :hamster: 1.2.4、空间索引

空间索引是对空间数据类型的字段建立的索引，`MySQL`中的空间数据类型有四种，`GEOMETRY`、`POINT`、`LINESTRING`、`POLYGON`。在创建空间索引时，使用`SPATIAL`关键字。要求，引擎为`MyISAM`，创建空间索引的列，必须将其声明为`NOT NULL`。

### :pen\_fountain: 1.3、存储方式

#### :hamster: 1.3.1、聚簇索引

叶节点包含了完整的数据记录。

#### :hamster: 1.3.2、非聚簇索引

叶节点的 data 域存放的是数据记录的地址。

\
