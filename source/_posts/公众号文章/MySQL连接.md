---
title: MySQL连接
categories: 
- 公众号文章
---

文章首发在公众号（月伴飞鱼），之后同步到个人网站：[xiaoflyfish.cn/](https://xiaoflyfish.cn/)

**觉得有收获，希望帮忙点赞，转发下哈，谢谢，谢谢**

# 前言

最近遇到在将本地的项目部署到服务器上之后遇到的一个奇怪问题

在部署完成后，网站当时可以正常工作，但是第二天访问网站的时候却会遇到一个500 Server Error。

**从日志中可以看出是MySQL数据库出现了异常**

![](https://img-blog.csdnimg.cn/img_convert/836ada99cd591cec4548548126c14fa1.png)

翻译如下:

> 最后一个数据包在 83827560 ms 之前被成功接收，最后一个数据包在83827560 ms 之前被成功发送。比服务的配置参数`wait_timeout`的值要长。

**日志中给出的建议如下**

![](https://img-blog.csdnimg.cn/img_convert/c19f19ccdcd4e0ed5a4605c695293cb1.png)

翻译如下：

> 你应考虑在程序中进行数据库操作之前检验数据库连接的有效性或者将数据库的autoReconnect属性设置为true来避免这个问题

关于`wait_timeout`和autoReconnect下面我们会依次分析介绍！

# 原因分析
我们进入mysql的命令行查询超时时间

![](https://img-blog.csdnimg.cn/img_convert/057fbd11acad2427f03cffc5251942fd.png)

28800单位是秒转化成小时就是8小时

看出MySQL的默认设置，当一个连接的空闲时间超过8小时后，MySQL就会断开该连接

所以发现问题出在如果超过这个`wait_timeout`时间(默认是8小时)对数据库没有任何操作，那么MySQL会自动关闭数据库连接以节省资源

数据库连接自动断开的问题确实是在第二天发生了，也就是在一个晚上没有对数据库进行操作(显然超过了8小时)的情况下发生的这个问题

![](https://img-blog.csdnimg.cn/20201206132908283.png)
**大家用命令show processlist; 可以查看Sleep状态的进程Sleep，同时可以看到每个进程Sleep多久了：**

![](https://img-blog.csdnimg.cn/img_convert/fd4fa6b582615a934d86d9997964cd69.png)

下面介绍下解决和优化办法！

# 解决方法
## 1.autoReconnect

这个参数表示在mysql超时断开连接后会自动重新连接

配置的话，只需要在连接mysql的语句写上autoReconnect=true 

```
jdbc:mysql://127.0.0.1:3306/stock_tweet?autoReconnect=true 
```

**下面是MySQL官网对autoReconnect的解释:**

![](https://img-blog.csdnimg.cn/img_convert/7de30449aadc072622d8f2f7b89121b2.png)
同时可以看到官网不推荐使用这个参数，因为它有一些副作用

具体介绍下：

* 原有连接上的事务将会被回滚，事务的提交模式将会丢失
* 原有连接持有的表的锁将会全部释放
* 原有连接关联的会话Session将会丢失，重新恢复的连接关联的将会是一个新的会话Session
* 原有连接定义的用户变量将会丢失
* 原有连接定义的预编译SQL将会丢失
* 原有连接失效，新的连接恢复后，MySQL将会使用新的记录行来存储连接中的性能数据

## 2.修改配置
涉及到两个配置参数`interactive_timeout和wait_timeout`

`wait_timeout `指的是mysql在关闭一个非交互的连接之前所要等待的秒数

`interactive_time` 指的是mysql在关闭一个交互的连接之前所要等待的秒数

对于交互和非交互连接，说得直白一点就是，通过mysql客户端连接数据库是交互式连接，通过jdbc连接数据库是非交互式连接。 

配置方法：

> 1.会话方式

```
msyql> set global wait_timeout=2880000;
msyql> set global interactive_timeout=2880000;
```

这种方式只对当前会话生效

> 2.修改配置文件方式

修改/etc/my.cnf文件，在 [mysqld] 节中设置： 

![](https://img-blog.csdnimg.cn/img_convert/01fb7af6950ec916e7c0a0324bb3b629.png)

之后再重启下服务器就好了

**注意：**

将`wait_timeout`这个值设置得大了，可能会导致空闲连接过多。

如果你的MySQL Server有大量的闲置连接，他们不仅会白白消耗内存，而且如果连接一直在累加而不断开，最终肯定会达到MySQL Server的连接上限数，这会报'too many connections'的错误。
![](https://img-blog.csdnimg.cn/20201206133121973.png)
## 连接池配置
因为连接池的配置也会影响项目和MySQL的连接，所以也需要对数据库连接池的一些配置做一定修改

我们以Spring Boot 2.0默认的数据库连接池HikariCP为例

主要是下面这几个配置

**maximum-pool-size：**

最大连接数，超过这个数，新的数据库访问线程会被阻，缺省值：10。

常见的错误是设置一个太大的值，连接数多反而性能下降。

参考计算公式是：

```
#core_count：CPU个数，effective_spindle_count:硬盘个数
connections = ((core_count * 2) + effective_spindle_count)
```

例如：一个4核，1块硬盘的服务器，连接数 = （4 * 2） + 1 = 9，凑个整数，10就可以了。

**minimum-idle：**

最小的连接数目

**max-lifetime:**

最大的连接时间，用来设置一个connection在连接池中的存活时间

缺省：30分钟。强烈建议设置比数据库超时时长少一点（MySQL的`wait_timeout`参数一般为8小时）。

**idle-timeout:**

一个连接idle状态的最长时间，超时则被释放

其他参数详见：https://github.com/brettwooldridge/HikariCP/wiki/About-Pool-Sizing

# 最后
微信搜索：月伴飞鱼，交个朋友

* 日常分享一篇实用的技术文章，对面试，工作都有帮助

* 后台回复666，获得免费电子书籍，会持续更新

参考：

https://dev.mysql.com/doc/refman/5.7/en/