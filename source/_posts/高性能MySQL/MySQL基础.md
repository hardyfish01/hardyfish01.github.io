---
title: MySQL基础
categories: 
- 高性能MySQL
---

**utf8和utf8mb4字符集**

在MySQL中字符集表示一个字符所用最大字节长度在某些方面会影响系统的存储和性能，所以设计MySQL定义了两个概念:

* utf8mb3：只使用1~3个字节表示字符。在MySQL中utf8是utf8mb3的别名。

* utf8mb4：正宗的utf8字符集，使用1~4个字节表示字符。 

> 存储一些emoji表情啥的，那请使用utf8mb4。

**SQL查询语句执行顺序**

SELECT语句的完整语法：

```sql
1. SELECT 
2. DISTINCT <select_list>
3. FROM <left_table>
4. <join_type> JOIN <right_table>
5. ON <join_condition>
6. WHERE <where_condition>
7. GROUP BY <group_by_list>
8. HAVING <having_condition>
9. ORDER BY <order_by_condition>
10.LIMIT <limit_number>
```

执行顺序是下面这样的：

```sql
FROM
<表名> # 笛卡尔积
ON
<筛选条件> # 对笛卡尔积的虚表进行筛选
JOIN <join, left join, right join...> 
<join表> # 指定join，用于添加数据到on之后的虚表中，例如left join会将左表的剩余数据添加到虚表中
WHERE
<where条件> # 对上述虚表进行筛选
GROUP BY
<分组条件> # 分组
<SUM()等聚合函数> # 用于having子句进行判断，在书写上这类聚合函数是写在having判断里面的
HAVING
<分组筛选> # 对分组后的结果进行聚合筛选
SELECT
<返回数据列表> # 返回的单列必须在group by子句中，聚合函数除外
DISTINCT
# 数据除重
ORDER BY
<排序条件> # 排序
LIMIT
<行数限制>
```
