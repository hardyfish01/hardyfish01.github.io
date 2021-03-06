---
title: Sentinel机制
categories: 
- 读书笔记
- Redis设计与实现
---

Sentinel（哨兵）是Redis的**高可用性**的解决方案，由一个或多个Sentinel实例组成的Sentinel系统可以监视任意多个主服务器以及属下的所有从服务器。

* 当主服务器下线时，自动将下线的某个主服务器属下的某个从服务器**升级**为新的主服务器。

从而实现**故障转移**，当原来的主服务器重新上线时，会被降级为从服务器。

**创建连向主服务器的网络连接**

Sentinel会为监视的主服务器创建两个异步网络连接：

- **命令连接**：专用于向主服务器发送命令，接收命令回复。
- **订阅连接**：专用于订阅主服务器`__sentinel__:hello`频道。
- 由于Redis的发布订阅消息不会保存，客户端断线就会丢失，为了不丢失，必须使用专门的频道连接。

**获取主从服务器信息**

Sentinel默认1**0秒一次**通过命令连接被监视的主服务器并发送**INFO**命令，获取主服务器信息。

* 主要获取主服务器本身信息（如服务器运行ID），下属从服务器信息（如ip，port，offset）。

对应属性进行更新，如果没有某个从服务器新信息就会创建一个实例结构，放到主服务器的slaves字典中，键为ip+端口，值为sentinelRedisInstance。

除了创建新实例，还会创建连接到从服务器的**命令连接**和**订阅连接**。

**向主服务器和从服务器发送消息**

sentinel默认以两秒一次，向服务器的`__sentinel__:hello`频道发送消息；

**接收来自主服务器和从服务器的频道信息**

Sentinel与一个主服务器或从服务器建立订阅连接后，会发送`SUBSCRIBE _sentinel_:hello`命令。

* 也就是Sentinel通过命令连接发送信息到频道，又通过订阅连接接收频道中的信息。

一个Sentinel发的信息也会被其他Sentinel接收，根据信息记录的Sentinel运行id和接收信息的Sentinel**运行id是否相同**，来决定**是否处理**这条消息。通过这种透明的沟通机制，Sentinel可以对各自监听的服务器信息进行更新。

**更新Sentinels字典**

根据接收而来的消息，Sentinel会更新实例结构中sentinels字典保存的所有Sentinel实例的信息。

* 键为Sentinel的ip+端口，值为某个Sentinel的实例。

消息接收者会检查发送消息的Sentinel（源sentinel）结构是否在sentinels字典存在则更新，没有则创建实例，和自己相同的sentinel不会被放入。

通过这种发布订阅的方式，Sentinel不需要各自发信息告诉对方，而是监视同一个主服务器的多个Sentinel自动发现对方。

**创建连向其他Sentinel的命令连接**

sentinel也会为对方互相创建命令连接，最终监视同一主服务器的多个sentinel会形成一个**网络**。

但他们互相之间**不会创建订阅连接**，因为他们通过主或从服务器发来的频道来发现未知的sentinel。

**检测主观下线状态**

Sentinel默认每秒与创建命令连接的实例（主服务器，从服务器，其他sentinel）发送PING命令，通过回复判断是否在线。

* 如果实例返回除了`+PONG`，`-LOADING`，`-MASTERRDOWN`之外的回复或未及时回复，就认为是**无效回复**。

* 根据配置文件的`down-after-milliseconds`指定的**主观下线所需时长内**是否一直无效回复，来判断实例是否已经主观下线。

下线了就将实例的的flags标识属性打开`SRI_S_DOWN`标识。

由于每个Sentinel中的主观下线时间配置都可以不同，所有有可能**某个Sentinel判断主观下线时，另一个Sentinel认为在线状态**。

**检查客观下线状态**

当Sentinel判断主服务器为主观下线时，还会向其他Sentinel询问，得到足量数据已下线判断后，就会判定服务器为客观下线，并执行故障转移。

**发送sentinel is-master-down-by-addr命令**

Sentinel使用：`SENTINEL is-master-down-by-addr <ip> <port> <current. epoch> <runid>`命令询问其他Sentinel是否同意主服务器下线。这些参数分别是Sentinel的ip，端口，配置纪元和运行id。

**接收sentinel is-master-down-by-addr命令的回复**

统计其他Sentinel同意主服务器已下线数量，当数量超过配置值（quorum参数）时，sentinel会将主服务器实例的flags属性的`SRI_O_DOWN`属性打开，表示已进入客观下线状态。

**选举领头Sentinel**

当主服务器被判断为客观下线时，sentinel会协商选举领头sentinel，并由领头sentinel对下线主服务器执行故障转移操作。

**故障转移**

故障转移包括3步：

1. 在已下线的主服务器属下从服务器里选出一个将其转为主服务器。
2. 让其他从服务器都复制新主服务器。
3. 当原来的主服务器再次上线时，让他成为新主服务器的从服务器。

**选出新主服务器**

如何选新的主服务器？Sentinel会将所有从服务器放入列表，**一项一项**过滤：

- 删除处于下线或断线状态的从服务器。
- 删除最近5秒没有回复过领头`sentinel INFO`命令的从服务器。
- 删除与已下线服务器段开时间超过`down-after-milliseconds*10`毫秒的从服务器。

然后根据**优先级排序**，相同则选**偏移量最大**的，相同则选运行ID最小的。

选出来之后，对这个从服务器发送`SLAVEOF no one`命令，然后以**每秒一次**的频率向它发送`INFO`命令，观察返回的role属性如果变成master，就表示顺利升级为主服务器了。

**修改从服务器的复制目标**

向所有其他从服务器发送`SLAVEOF`命令，让他们都去复制新的主服务器。

**将旧的主服务器变为从服务器**

当原来的主服务器上线时，Sentinel就会向它发送`SLAVEOF`命令，让他成为新主服务器的从服务器。