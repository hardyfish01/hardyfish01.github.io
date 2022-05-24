---
title: Order By
categories: 
- 数据库
- MySQL
---

假设用一张员工表，表结构如下：

```sql
CREATE TABLE `staff` (
`id` BIGINT ( 11 ) AUTO_INCREMENT COMMENT '主键id',
`id_card` VARCHAR ( 20 ) NOT NULL COMMENT '身份证号码',
`name` VARCHAR ( 64 ) NOT NULL COMMENT '姓名',
`age` INT ( 4 ) NOT NULL COMMENT '年龄',
`city` VARCHAR ( 64 ) NOT NULL COMMENT '城市',
PRIMARY KEY ( `id`),
INDEX idx_city ( `city` )
) ENGINE = INNODB COMMENT '员工表'
```

有这么一个需求：**查询前10个，来自深圳员工的姓名、年龄、城市，并且按照年龄小到大排序**。对应的 SQL 语句就可以这么写：

```sql
select name,age,city from staff where city = '深圳' order by age limit 10;
```

# 工作原理

## 全字段排序

MySQL 会给每个查询线程分配一块小**内存**，用于**排序**的，称为 **sort_buffer**。

**整体的执行流程就是：**

1. MySQL 为对应的线程初始化**sort_buffer**，放入需要查询的name、age、city字段；
2. 从**索引树idx_city**， 找到第一个满足 city=’深圳’条件的主键 id；
3. 到**主键 id 索引树**拿到id=9的这一行数据， 取name、age、city三个字段的值，存到sort_buffer；
4. 从**索引树idx_city** 拿到下一个记录的主键 id；
5. 重复步骤 3、4 直到**city的值不等于深圳**为止；
6. 前面5步已经查找到了所有**city为深圳**的数据，在 sort_buffer中，将所有数据根据age进行排序；
7. 按照排序结果取前10行返回给客户端。

<img src="https://img-blog.csdnimg.cn/9ae713c6b33648169cf5387cb14f9b28.png" alt="img" style="zoom:25%;" />

将查询所需的字段全部读取到sort_buffer中，就是**全字段排序**。

**磁盘临时文件辅助排序**

实际上，sort_buffer的大小是由一个参数控制的：**sort_buffer_size**。

如果要排序的数据小于sort_buffer_size，排序在**sort_buffer** 内存中完成，如果要排序的数据大于sort_buffer_size，则**借助磁盘文件来进行排序**

## Rowid排序

rowid 排序就是，只把查询SQL**需要用于排序的字段和主键id**，放到sort_buffer中。

> 怎么确定走的是全字段排序还是rowid 排序排序呢？

有个参数控制：**max_length_for_sort_data**，它表示MySQL用于排序行数据的长度的一个参数，如果单行的长度超过这个值，MySQL 就认为单行太大，就换rowid 排序。

**使用rowid 排序的话，整个SQL执行流程：**

1. MySQL 为对应的线程初始化**sort_buffer**，放入需要排序的age字段，以及主键id；
2. 从**索引树idx_city**， 找到第一个满足 city=’深圳’条件的主键 id；
3. 到**主键 id 索引树**拿到id=9的这一行数据， 取age和主键id的值，存到sort_buffer；
4. 从**索引树idx_city** 拿到下一个记录的主键 id；
5. 重复步骤 3、4 直到**city的值不等于深圳**为止；
6. 前面5步已经查找到了所有city为深圳的数据，在 **sort_buffer**中，将所有数据根据age进行排序；
7. 遍历排序结果，取前10行，并按照 id 的值**回到原表**中，取出city、name 和 age 三个字段返回给客户端。

<img src="https://img-blog.csdnimg.cn/48e67cb0852242ff9b060c7ebd2e09e7.png" alt="img" style="zoom:25%;" />

对比一下**全字段排序**的流程，rowid 排序多了一次**回表**。

> 什么是回表？拿到主键再回到主键索引查询的过程，就叫做回表

**全字段排序与Rowid排序对比**

- 全字段排序： sort_buffer内存不够的话，就需要用到磁盘临时文件，造成**磁盘访问**。
- rowid排序： sort_buffer可以放更多数据，但是需要再回到原表去取数据，比全字段排序多一次**回表**。

一般情况下，对于InnoDB存储引擎，会优先使**用全字段**排序。

可以发现 **max_length_for_sort_data** 参数设置为1024，这个数比较大的。

一般情况下，排序字段不会超过这个值，也就是都会走**全字段**排序。