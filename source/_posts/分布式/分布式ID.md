---
title: 分布式ID
categories: 
- 分布式
---

在互联网的业务系统中，涉及到各种各样的ID，如在支付系统中就会有支付ID、退款ID等。

# UUID

算法的核心思想是结合机器的网卡、当地时间、一个随机数来生成UUID。

UUID的16个8位字节表示为32个十六进制的数字，显示在由连字符分割的五个组中，8-4-4-4-12总共36个字符，例如：

```java
123e4567-e89b-12d3-a456-426655440000
import java.util.UUID;
 
public class Uuid {
    public static void main(String[] args) {
        for (int i = 0; i < 5; i++) { 
            String uuid = UUID.randomUUID().toString();
    }
}
```

优点：

- 性能非常高：本地生成，没有网络消耗。

缺点：

- 不易于存储：UUID太长，16字节128位，通常以36长度的字符串表示，很多场景不适用。
- 信息不安全：基于MAC地址生成UUID的算法可能会造成MAC地址泄露。
- ID作为主键时在特定的环境会存在一些问题，比如做DB主键的场景下，UUID就非常不适用。

# 数据库自增ID

使用数据库的id自增策略，如 MySQL 的 `auto_increment`。

```sql
REPLACE INTO T_TEST (value) VALUES('b')
SELECT LAST_INSERT_ID();
```

`LAST_INSERT_ID`即为最后插入的ID值。

优点：

- 非常简单，利用现有数据库系统的功能实现，成本小，有DBA专业维护。
- ID号单调自增，可以实现一些对ID有特殊要求的业务。

缺点：

- 强依赖DB，当DB异常时整个系统不可用，属于致命问题。
- 配置主从复制可以尽可能的增加可用性，但是数据一致性在特殊情况下难以保证，主从切换时的不一致可能会导致重复发号。
- ID发号性能瓶颈限制在单台MySQL的读写性能。

# 雪花算法

SnowFlake：https://github.com/twitter-archive/snowflake

Twitter 开源的分布式 id 生成算法。

其核心思想就是：使用一个 64 bit 的 long 型的数字作为全局唯一 id。

在分布式系统中的应用十分广泛，且ID 引入了 时间戳，基本上保持自增的，结构如下：

![img](https://img-blog.csdnimg.cn/d3971690224b41a9b10523290a45d84d.png)

**号段解析**

> 第一部分：1bit

符号位，1表示负数，0表示正数，生成的id一般都是用整数，所以最高位固定为0。

> 第二部分：41bit

用来记录时间戳，毫秒级，表示时间戳位。

> 第三部分：10bit

用来表示工作机器id，可以分别表示1024台机器。

> 第三部分：12bit

序列号，用来记录同毫秒值内产生的不同ID。

**优点：**

- 毫秒数在高位，自增序列在低位，整个ID都是趋势递增的。
- 不依赖数据库等第三方系统，以服务的方式部署，稳定性更高，生成ID的性能也是非常高的。
- 可以根据自身业务特性分配bit位，非常灵活。

缺点：

- 强依赖机器时钟，如果机器上时钟回拨，会导致发号重复或者服务会处于不可用状态。

**算法实现**

```java
public class IdWorker {
    /**
     * 起始时间戳 2017-04-01
     */
    private final long epoch = 1491004800000L;
    /**
     * 机器ID所占的位数
     */
    private final long workerIdBits = 5L;
    /**
     * 数据标识ID所占的位数
     */
    private final long dataCenterIdBits = 5L;
    /**
     * 支持的最大机器ID,结果是31
     */
    private final long maxWorkerId = ~(-1L << workerIdBits);
    /**
     * 支持的最大数据标识ID,结果是31
     */
    private final long maxDataCenterId = ~(-1 << dataCenterIdBits);
    /**
     * 毫秒内序列在id中所占的位数
     */
    private final long sequenceBits = 12L;
    /**
     * 机器ID向左移12位
     */
    private final long workerIdShift = sequenceBits;
    /**
     * 数据标识ID向左移17(12+5)位
     */
    private final long dataCenterIdShift = sequenceBits + workerIdBits;
    /**
     * 时间戳向左移22(12+5+5)位
     */
    private final long timestampShift = sequenceBits + workerIdBits + dataCenterIdBits;
    /**
     * 生成序列的掩码，这里为4095 (0b111111111111=0xfff=4095)
     */
    private final long sequenceMask = ~(-1L << sequenceBits);
    /**
     * 数据标识ID（0～31）
     */
    private long dataCenterId;
    /**
     * 机器ID（0～31）
     */
    private long workerId;
    /**
     * 毫秒内序列（0～4095）
     */
    private long sequence;
    /**
     * 上次生成ID的时间戳
     */
    private long lastTimestamp = -1L;

    public IdWorker(long dataCenterId, long workerId) {
        if (dataCenterId > maxDataCenterId || dataCenterId < 0) {
            throw new IllegalArgumentException(String.format("dataCenterId can't be greater than %d or less than 0", maxDataCenterId));
        }
        if (workerId > maxWorkerId || workerId < 0) {
            throw new IllegalArgumentException(String.format("worker Id can't be greater than %d or less than 0", maxWorkerId));
        }
        this.dataCenterId = dataCenterId;
        this.workerId = workerId;
    }

    /**
     * 获得下一个ID (该方法是线程安全的)
     * @return snowflakeId
     */
    public synchronized long nextId() {
        long timestamp = timeGen();
        //如果当前时间小于上一次ID生成的时间戳,说明系统时钟回退过,这个时候应当抛出异常
        if (timestamp < lastTimestamp) {
            throw new RuntimeException(String.format("Clock moved backwards.  Refusing to generate id for %d milliseconds", lastTimestamp - timestamp));
        }
        //如果是同一时间生成的，则进行毫秒内序列
        if (timestamp == lastTimestamp) {
            sequence = (sequence + 1) & sequenceMask;
            //毫秒内序列溢出
            if (sequence == 0) {
                //阻塞到下一个毫秒,获得新的时间戳
                timestamp = nextMillis(lastTimestamp);
            }
        } else {//时间戳改变，毫秒内序列重置
            sequence = 0L;
        }
        lastTimestamp = timestamp;
        //移位并通过按位或运算拼到一起组成64位的ID
        return ((timestamp - epoch) << timestampShift) |
                (dataCenterId << dataCenterIdShift) |
                (workerId << workerIdShift) |
                sequence;
    }

    /**
     * 返回以毫秒为单位的当前时间
     * @return 当前时间(毫秒)
     */
    protected long timeGen() {
        return System.currentTimeMillis();
    }

    /**
     * 阻塞到下一个毫秒，直到获得新的时间戳
     * @param lastTimestamp 上次生成ID的时间截
     * @return 当前时间戳
     */
    protected long nextMillis(long lastTimestamp) {
        long timestamp = timeGen();
        while (timestamp <= lastTimestamp) {
            timestamp = lastTimestamp;
        }
        return timestamp;
    } 
}
```

# 其他

**百度UidGenerator**

UidGenerator是百度开源的分布式ID生成器，基于于snowflake算法的实现。

具体可以参考官网说明：https://github.com/baidu/uid-generator/blob/master/README.zh_cn.md

**美团Leaf**

美团开源的分布式ID生成器，能保证全局唯一性、趋势递增、单调递增、信息安全，需要依赖关系数据库、Zookeeper等中间件。

具体可以参考官网说明：https://tech.meituan.com/2017/04/21/mt-leaf.html