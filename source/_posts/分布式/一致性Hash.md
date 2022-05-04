---
title: 一致性Hash
categories: 
- 分布式
---

我们 经常会用 Redis 做缓存，把一些数据放在上面，以减少数据的压力。

当数据量变大，并发量也增加的时候，通常我们会搭建集群环境，让数据尽量平均的放到每一台 Redis 中，比如集群中有 4 台Redis。

> 那么如何把数据尽量平均地放到这 4 台Redis中呢？最简单的就是取模算法：

```
hash(key)%N，N 为 Redis 的数量，在这里 N = 4
```

如果4 台 `Redis` 不够了，需要再增加 4 台 Redis，那么这个求余算法就会变成：`hash(key)%8，`这样大部分缓存的位置都会是错误的，极端情况下，就会造成缓存雪崩。

<img src="https://img-blog.csdnimg.cn/18575de986c94148b05c42764c4e144e.png" style="zoom:50%;" />

**一致性Hash算法可以很好地解决这个问题，大概过程：**

* 把 0 作为起点，`2^32-1` 作为终点，画一条直线，再把起点和终点重合，直线变成一个圆，方向是顺时针从小到大，0 的右侧第一个点是 1 ，然后是 2 ，以此类推。

* 对三台服务器的 IP 或其他关键字进行 hash 后对 `2^32` 取模，这样能落在这个圈上的某个位置，记为 Node1、Node2、Node3。

<img src="https://img-blog.csdnimg.cn/95a9aec86e01443db817201f74f4304f.png" style="zoom:25%;" />

对数据 key 进行相同的hash操作，它也会落在圈上的某个位置；

然后顺时针行走，可以找到某一个 `Node`，这就是这个 key 要储存的服务器。

<img src="https://img-blog.csdnimg.cn/a7161c14e6444e3cb7603073f02341e7.png" style="zoom:25%;" />

如果增加一台服务器或者删除一台服务器，只会影响部分数据。

<img src="https://img-blog.csdnimg.cn/f5479d2c27604613931e0e57c2cdcd66.png" style="zoom:25%;" />

**数据倾斜**

但如果节点太少或分布不均匀的时候，容易造成**数据倾斜**，也就是大部分数据会集中在某一台服务器上。

<img src="https://img-blog.csdnimg.cn/784ffeacecc543fdae07da433b0c485c.png" style="zoom:25%;" />

为了解决数据倾斜问题，一致性 `Hash` 算法提出了虚拟节点，会对每一个服务节点计算多个哈希，然后放到圈上的不同位置。

<img src="https://img-blog.csdnimg.cn/aab68f440cb84aceb73864da5a39e5b2.png" style="zoom:25%;" />

**总结：**

一致性哈希通过一个哈希环实现，Hash 环的基本思路是获取所有的服务器节点 hash 值，然后获取 key 的 hash，与节点的 hash 进行对比，找出顺时针最近的节点进行存储和读取。