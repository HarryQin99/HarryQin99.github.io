---
layout: post
title: 📕【Database】MySQL的索引
date: 2021-9-20 00:00
---



[Hash索引](#Hash索引)
[B+树索引](#B+树索引)
[Hash索引和B+树索引有什么区别？](#Hash索引和B+树索引有什么区别？)
[为什么不使用B树而使用B+树？](#为什么不使用B树而使用B+树？)
[聚集索引(Cluster index)](#聚集索引(Cluster index))
[非聚集索引(Uncluster index)](#非聚集索引(Uncluster index))
[联合索引](#联合索引)
[覆盖索引](#覆盖索引)

### Hash索引

1. Hash索引主要运用了哈希表的数据结构，使用键值对的方式存储数据。通过Key的哈希值能快速找到相应的Value，所以时间复杂度最优情况下会是O(1)
2. 使用哈希表就无法避免一个问题 -- 哈希冲突。在极端下，哈希冲突无法很好解决的话，会导致查询的时间复杂度趋于O(n)

### B+树索引

1. B+树是一种特殊的平衡多叉树，每一个非叶子结点储存的仅仅是对下一层的索引，所有的数据都只存于最后一层叶子结点，并且最后一层叶子结点是排好序且每个结点直接用双向链表连接。每一次查询都稳定在O(m, m=树高)

### Hash索引和B+树索引有什么区别？

1. 查询的时间复杂度不同。Hash索引最优情况下是O(1)，B+树是稳定O(m)
2. Hash索引会有哈希冲突的情况，对查询的时间复杂度有较大影响
3. Hash索引无法解决范围查询，相反B+树十分擅长解决范围查询

### 为什么不使用B树而使用B+树？

1. 什么是B树 ---- B树可以理解为是一个平衡多叉树，与B+树最大的不同在于他的每个结点不仅仅储存对下一层的索引，还同时储存数据
2. **查询IO层面**。因为B+树的非叶子结点储存的仅仅是索引，相比于B树(既储存索引，又储存数据)，同一个16K的结点，B+树结点中的索引数会更加多。因此B+树非叶子结点的子分支也会多很多。储存相同量的数据，从树的形状看，B+树在横轴上会更加宽，层数也会相较B树小，所以每次索引查询数据时，所需要的IO也更加小。
3. **范围查询的支持层面**。因为B+树特殊的数据结构，对范围查询特别友好，找到符合条件数据在叶子结点的起始位置，利用双向链表即可。但使用B树进行范围查询有需要跨层再次查询的机会，会产生不必要的IO
4. **查询稳定性层面**。对于B树，查询同一数据不同的索引条件下会有不同的IO次数，依赖于该数据在B树的第几层。但是对于B+树来水，查询效率是稳定的，每一次都稳定O(m, m=树高)

## 聚集索引和非聚集索引

### 聚集索引(Cluster index)

聚集索引是B+树索引的某一种具体例子，每张表格都会有一个基于主键的聚集索引，其叶子结点储存的是该表的所有数据，对于利用主键的范围查询速度非常快，每一次查询的复杂度就是B+树的层高

### 非聚集索引(Uncluster index)

非聚集索引和聚集索引最大的区别是，在非聚集索引的叶子结点中，储存的仅仅是索引的键值和拥有该键值数据的主键，数据库会通过主键和使用主键建立的聚集索引找到相应的数据。简单来说，非聚集索引只能找到符合条件的数据的主键，为了找到符合条件的数据需要再一次使用基于主键的聚集索引查询一次。

举个🌰 有一个表格User_table有三个数据，一是id，二是名字，三是年龄。因为id是主键，所以当我们创建表格时，MySQL就已经自动创建id作为键值的聚集索引。但这时候，小明有需要通过名字查找数据的需求，为了加快查找的速度，那么他对User_table中的名字创建了一个名字作为键值的索引，该索引也就是非聚集索引。
小明输入 SELECT * FROM User_table WHERE name = Harry时，MySQL需要：通过基于名字的非聚集索引找到键值为Harry的数据，记录下该数据中的主键Id，记为id_Harry，再去到基于主键的聚集索引中查找id为id_Harry的数据，这时候就完成了一次通过名字查找表格中的所有信息，这个过程也就做回表。

## 其他索引

### 联合索引

联合索引是一种基于多个索引列创建的非聚集索引。

最大的好处就是使用联合索引时，其会按由左到右的健值对数据进行排序。在特定情况下，使用联合索引可以避免多一次排序操作。

以下引用《MySQL技术内幕》的例子

```SQL
CREATE TABLE record (
	userid INT UNSIGNED NOT NULL,
  record_date DATE
)ENGINE=InnoDB

INSERT INTO record VALUE (1, '2020-01-01')
INSERT INTO record VALUE (2, '2020-01-01')
INSERT INTO record VALUE (3, '2020-01-01')
INSERT INTO record VALUE (1, '2020-02-01')
INSERT INTO record VALUE (3, '2020-02-01')
INSERT INTO record VALUE (1, '2020-04-01')

ALTER TABLE record ADD KEY ( userid )
ALTER TABLE record ADD KEY ( userid, record_data)
```

我们创建了两个索引，一个以userid为索引健，一个以( userid, record_date) 为索引健，也就是说我们当前有两个B+树，一棵是以userid排序的；一棵是先以userid排序，在同userid下又以record_date排序的。

```sql
SELECT * FROM record WHERE id=1;
```

对于这个sql语句，优化器选择以userid排序的B+树进行索引

```sql
SELECT * FROM record where id=1 ORDER BY record_date;
```

这个sql语句用两个索引当中的哪一个都可以，但是因为语句要求对recored_date进行排序，而同时因联合索引( userid, record_date)的缘故，其索引树针对同id的数据进行了基于record_date排序，所以优化器会选择以( userid, record_date) 为索引健的B+树执行sql语句，避免了一次排序操作

### 覆盖索引(Covering Indexes)

正常的一次不基于主键的非聚集索引查询，都需要在查到相关符合条件数据的主键后，再对主键进行一次使用聚集索引的回表操作，覆盖索引的意思是，仅仅通过一个对非聚集索引B+树的查询就可以得到需要的查询记录，避免回表。

```sql
CREATE TABLE record (
	userid INT UNSIGNED NOT NULL,
  record_date DATE
)ENGINE=InnoDB

INSERT INTO record VALUE (1, '2020-01-01')
INSERT INTO record VALUE (2, '2020-01-01')
INSERT INTO record VALUE (3, '2020-01-01')
INSERT INTO record VALUE (1, '2020-02-01')
INSERT INTO record VALUE (3, '2020-02-01')
INSERT INTO record VALUE (1, '2020-04-01')

ALTER TABLE record ADD KEY ( userid )
```

例如以上例子：

当我们运行  ```select * from record where userid=1```时，因为使用了机遇 userid的非聚集索引，且非聚集索引的每个节点只有(主键， userid)，而缺失了record_date的数据，所以这时候不得不做一次回表操作，拿出所有符合条件的主键id在基于主键的聚集索引下找到相关的数据行，并包括record_data的值。

而当我们运行 ```select count(*) from record where userid=1```时就不需要回表操作了，因为仅仅通过一次对非聚集B+树的查询就可以获得userid为1数据的数量，这就是**覆盖查询**，避免了一次回表，减少了IO操作。

同理可得，当我们 ```ALTER TABLE record ADD KEY ( userid, record_data)```后，再运行 ```select (userid, record_date) from record where userid=1```就不需要回表了，因为基于```( userid, record_data)```的非聚集B+树的叶子节点数据中有userid和record_data，不必再一次回表操作了。

