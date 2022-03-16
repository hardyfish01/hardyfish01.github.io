---
title: Order By
categories: 
- 高性能MySQL
---

```sql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `city` varchar(16) NOT NULL,
  `name` varchar(16) NOT NULL,
  `age` int(11) NOT NULL,
  `addr` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `city` (`city`)
) ENGINE=InnoDB;
```

```sql
select city,name,age from t where city='杭州' order by name limit 1000  ;
```

**全字段排序**

Extra这个字段中的**Using filesort**表示的就是需要排序，MySQL会给每个线程分配一块内存用于排序，称为`sort_buffer`。

通常情况下，这个语句执行流程如下所示 ：

1. 初始化`sort_buffer`，确定放入name、city、age这三个字段；
2. 从索引city找到第一个满足city='杭州’条件的主键id；
3. 到主键id索引取出整行，取name、city、age三个字段的值，存入`sort_buffer`中；
4. 从索引city取下一个记录的主键id；
5. 重复步骤3、4直到city的值不满足查询条件为止；
6. 对`sort_buffer`中的数据按照字段name做快速排序；
7. 按照排序结果取前1000行返回给客户端。

这个排序过程，称为全字段排序

**按name排序**这个动作，可能在内存中完成，也可能需要使用外部排序，这取决于排序所需的内存和参数`sort_buffer_size`。

`sort_buffer_size`，就是MySQL为排序开辟的内存（`sort_buffer`）的大小。

如果要排序的数据量小于`sort_buffer_size`，排序就在内存中完成。

> 但如果排序数据量太大，内存放不下，则不得不利用磁盘临时文件辅助排序。

**rowid排序**

上面这个算法有一个问题，就是如果查询要返回的字段很多的话，那么`sort_buffer`里面要放的字段数太多，这样内存里能够同时放下的行数很少，要分成很多个临时文件，排序的性能会很差。

> 所以如果单行很大，这个方法效率不够好。

修改一个参数，让MySQL采用另外一种算法。

```sql
SET max_length_for_sort_data = 16;
```

`max_length_for_sort_data`，是MySQL中专门控制用于排序的行数据的长度的一个参数。

> 它的意思是，如果单行的长度超过这个值，MySQL就认为单行太大，要换一个算法。

city、name、age 这三个字段的定义总长度是36，我把`max_length_for_sort_data`设置为16，我们再来看看计算过程有什么改变。

新的算法放入`sort_buffer`的字段，只有要排序的列（即name字段）和主键id。

但这时，排序的结果就因为少了city和age字段的值，不能直接返回了，整个执行流程就变成如下所示的样子：

1. 初始化sort_buffer，确定放入两个字段，即name和id；
2. 从索引city找到第一个满足city='杭州’条件的主键id；
3. 到主键id索引取出整行，取name、id这两个字段，存入`sort_buffer`中；
4. 从索引city取下一个记录的主键id；
5. 重复步骤3、4直到不满足city='杭州’条件为止；
6. 对sort_buffer中的数据按照字段name进行排序；
7. 遍历排序结果，取前1000行，并按照id的值回到原表中取出city、name和age三个字段返回给客户端。

**全字段排序和rowid排序**

如果MySQL认为内存足够大，会优先选择全字段排序，把需要的字段都放到`sort_buffer`中，这样排序后就会直接从内存里面返回查询结果了，不用再回到原表去取数据。

这也就体现了MySQL的一个设计思想：**如果内存够，就要多利用内存，尽量减少磁盘访问。**

对于InnoDB表来说，rowid排序会要求回表多造成磁盘读，因此不会被优先选择。