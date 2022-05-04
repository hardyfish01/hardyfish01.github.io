---
title: SQL语句
categories: 
- 读书笔记
- 阿里Java开发手册
- MySQL数据库
---

1.不要使用count(列名)或count(常量)来替代`count(*)`，`count(*)`是SQL92定义的标准统计行数的语法，跟数据库无关，跟NULL和非NULL无关。

> 说明：`count(*)`会统计值为NULL的行，而count(列名)不会统计此列为NULL值的行。

2.`count(distinct col)`计算该列除NULL之外的不重复行数，注意 `count(distinct col1, col2)` 如果其中一列全为NULL，那么即使另一列有不同的值，也返回为0。

3.当某一列的值全是NULL时，`count(col)`的返回结果为0，但`sum(col)`的返回结果为NULL，因此使用sum()时需注意NPE问题。

> 正例：可以使用如下方式来避免sum的NPE问题：`SELECT IF(ISNULL(SUM(g)),0,SUM(g)) FROM table;`

4.使用ISNULL()来判断是否为NULL值。

说明：NULL与任何值的直接比较都为NULL。

> NULL<>NULL的返回结果是NULL，而不是false。
>
> NULL=NULL的返回结果是NULL，而不是true。
>
> NULL<>1的返回结果是NULL，而不是true。

5.不得使用外键与级联，一切外键概念必须在应用层解决。

6.禁止使用存储过程，存储过程难以调试和扩展，更没有移植性。

7.数据订正（特别是删除、修改记录操作）时，要先select，避免出现误删除，确认无误才能执行更新语句。

8.in操作能避免则避免，若实在避免不了，需要仔细评估in后边的集合元素数量，控制在1000个之内。

9.如果有全球化需要，所有的字符存储与表示，均以utf-8编码，注意字符统计函数的区别。

> 如果需要存储表情，那么选择utf8mb4来进行存储，注意它与utf-8编码的区别。

10.`TRUNCATE TABLE` 比 DELETE 速度快，且使用的系统和事务日志资源少，但TRUNCATE无事务且不触发trigger，有可能造成事故，故不建议在开发代码中使用此语句。

> 说明：TRUNCATE TABLE 在功能上与不带WHERE子句的DELETE语句相同。