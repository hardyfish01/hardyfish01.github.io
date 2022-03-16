---
title: 常用SQL
categories: 
- 高性能MySQL
---

appinfo表中有一个字段名字叫做`owner_id`，这个字段里面存的值是用逗号隔开的多个人的工号，想随便拿一个工号，按照`owner_id`这个字段去进行查询，就可以使用以下sql语句：

```sql
select*from appinfo where concat (',',owner_id,',') regexp ',123123,'
```

`t_feed_user_black_list`表中的user_id值批量存入`t_recommend_black`表中:

```sql
insert into stock_user.t_recommend_black(user_id) select user_id from stock_feed.t_feed_user_black_liston duplicate key update user_id =values(user_id);

insert into stock_tweet.t_exposure_edition(exposure_id, edition) select exposure_id,'full' from stock_tweet.t_exposure;
```

关联删除

```sql
delete a,b from stock_data.t_international_language as a inner join stock_data.t_international_language_key as b on a.id = b.language_id where b.business_type =7;
```

