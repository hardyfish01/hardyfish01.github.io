---
title: ServerLess
categories: 
- 分布式
---

**Serverless要解决什么问题？**

Serverless是对运维体系的极端抽象，它给应用开发和部署提供了一个极简模型。

* 这种高度抽象的模型，可以让一个零运维经验的人，几分钟就部署一个Web应用上线，并对外提供服务。

也就是说，你根本不需要再学习怎么在Linux上装Web服务器，怎么配置负载均衡等等这些繁琐的偏运维方向的工作。

**为什么大公司都在关注Serverless？**

> Serverless可以有效降低企业中中长尾应用的运营成本。

中长尾应用就是那些每天大部分时间都没有流量或者有很少流量的应用，你可以想想你们公司是不是也有很多。

尤其是企业在落地微服务架构后，一些边缘的微服务被调用的概率其实很低。

而这个时候，我们往往又很难通过人工来控制中长尾应用，因为这里面不少应用还是被强依赖的，不可以直接下线处理。

Serverless之前，这些中长尾应用至少要独占1台虚拟机；现在有了Serverless的**极速冷启动**特性，企业就可以节省这部分开销。

> Serverless可以提高研发效能。

Serverless应用架构的设计，其中，SFF（Serverless For Frontend）可以让前端同学自行负责数据接口的编排，微服务BaaS化则让我们的后端同学更加关注领域设计。

* 可以说，这是一个颠覆性的变化，它能够进一步放大前端工程师的价值。

Serverless作为一门新兴技术，有创业公司用FaaS来做基础设施编排和云服务编排；也有外包公司利用Serverless应用架构的快速迭代能力，提升开发效率，降低出错率，还可以给自己沉淀领域的解决方案；

**广义的 Serverless 是指：**

> 构建和运行软件时不需要关心服务器的一种架构思想。 

虽然 Serverless 翻译过来是 **无服务器**，但这并不代表着应用运行不需要服务器，而是开发者不需要关心服务器。

> 基于 Serverless 思想实现的软件架构就是 Serverless 架构。

**Serverful 的架构**

在这种架构下，如果要保证网站持续稳定运行，就需要解决很多问题。

* 备份容灾： 要实现服务器、数据库的备份容灾机制，使一台服务器出故障不影响整个系统。

* 弹性伸缩： 系统能根据业务流量大小等指标，响应式地调整服务规模，实现自动弹性伸缩。

* 日志监控： 需要记录详细的日志，方便排查问题和观察系统运行情况，并且实现实时的系统监控和业务监控。

解决这些复杂的问题需要投入大量的人力、物力，小公司几乎无法自己去解决。

而对开发者来说，Serverful 的架构开发成本也非常高，原本几行代码就可以搞定一个简单的业务逻辑，但你却得添加庞大的框架，比如 RPC（Remote Procedure Call，远程调用）、缓存等。

> Serverless 就是为了解决这些问题诞生的。 它可以把底层的硬件、存储等基础资源隐藏起来，由平台统一调度、运维。
>
> 将常用的基础技术抽象、封装（比如数据库、消息队列等）以服务的方式提供给开发者。

开发者只专注于开发业务逻辑，所有业务无关的基础设施，都交给 Serverless 平台。

**Serverless 和 Serverful 的架构区别**

* 资源分配： 在 Serverless 架构中，你不用关心应用运行的资源（比如服务配置、磁盘大小）只提供一份代码就行。
* 计费方式： 在 Serverless 架构中，计费方式按实际使用量计费（比如函数调用次数、运行时长），不按传统的执行代码所需的资源计费（比如固定 CPU）。计费粒度也精确到了毫秒级，而不是传统的小时级别。
* 弹性伸缩： Serverless 架构的弹性伸缩更自动化、更精确，可以快速根据业务并发扩容更多的实例，甚至允许缩容到零实例状态来实现零费用，对用户来说是完全无感知的。

> 一个应用如果是 Serverless 架构的，必须要实现自动弹性伸缩和按量付费，这也是 Serverless 的核心特点。

Kubernetes 是介于 Serverful 和 Serverless 中间的产物。

> Serverless 是云原生的一种实现，云原生的另一种实现是 Kubernetes。

**云原生**指的是原生为云设计的架构模式，就是应用一开始设计开发就按照在云上运行的方式进行，充分利用云的优势。

Serverless 几乎封装了所有的底层资源调度和运维工作，让你更容易使用云计算基础设施，极大简化了基于云服务的编程。

**基于 Serverless，电商网站部署架构如下：**

<img src="https://img-blog.csdnimg.cn/c5bbaf1cf9d74ced9aaef633be0b0975.png" style="zoom:15%;" />

* 通过网关承接用户流量，并将流量转发到在 FaaS 平台运行的函数中。每个函数都是一个特定的接口，实现单一业务逻辑。

* 基于 BaaS 实现复杂业务功能。而函数本身，也还可以调用其他微服务。

基于这样的架构，我们就完全不用关心运维，并且 FaaS 平台的弹性伸缩能力，就能实现业务的秒级弹性。

> 总的来说，基于 Serverless开发者就只需要关心业务逻辑的开发。

进行应用部署时也不再需要关心服务器，不需要关心后续的运维，应用也天然具备了弹性伸缩的能力，并且实现了按需使用，按量付费，也更能进一步节省成本。

**总结来说Serverless的含义有这样两种：**

* 第一种：狭义Serverless（最常见）= Serverless computing架构 = FaaS架构 = Trigger（事件驱动）+ FaaS（函数即服务）+ BaaS（后端即服务，持久化或第三方服务）= FaaS + BaaS

* 第二种：广义Serverless = 服务端免运维 = 具备Serverless特性的云服务

<img src="https://img-blog.csdnimg.cn/f2785c910eb84d00aa87dc89e5e76f4a.png" style="zoom:25%;" />

广义Serverless包含的东西更多，适用范围更广，但我们经常在工作中提到的Serverless一般都是指狭义的Serverless。

* FaaS(Function as a Service)就是函数即服务。

* BaaS(Backend as a Service)就是后端即服务。

* XaaS(X as a Service)就是X即服务，这是云服务商喜欢使用的一种命名方式，比如我们熟悉的SaaS、PaaS、IaaS都是这样。

FaaS，函数即服务，它还有个名字叫作Serverless Computing，它可以让我们随时随地创建、使用、销毁一个函数。

**通常函数的使用过程：**

* 它需要先从代码加载到内存，也就是实例化，然后被其它函数调用时执行。

* 在FaaS中也是一样的，函数需要实例化，然后被触发器Trigger或者被其他的函数调用。

* 二者最大的区别就是在Runtime，也就是函数的上下文，函数执行时的语境。

* FaaS的Runtime是预先设置好的，Runtime里面加载的函数和资源都是云服务商提供的，我们可以使用却无法控制。
* 你可以理解为FaaS的Runtime是临时的，函数调用完后，这个临时Runtime和函数一起销毁。

* FaaS的函数调用完后，云服务商会销毁实例，回收资源，所以FaaS推荐无状态的函数。

MVC架构里面，一个HTTP的数据请求，就会对应一个Control函数，我们完全可以用FaaS函数来代替Control函数。

* 在HTTP的数据请求量大的时候，FaaS函数会自动扩容多实例同时运行；

* 在HTTP的数据请求量小时，又会自动缩容；当没有HTTP数据请求时，还会缩容到0实例，节省开支。

**BaaS：**

BaaS其实是一个集合，是指具备高可用性和弹性，而且免运维的后端服务。

说简单点，就是专门支撑FaaS的服务。

* FaaS就像高铁的车头，如果我们的后端服务还是老旧的绿皮火车车厢，那肯定是要散架的，而BaaS就是专门为FaaS准备的高铁车厢。

* MVC架构中的Model层，就需要我们用BaaS来解决。

Model层我们以MySQL为例，后端服务最好是将FaaS操作的数据库的命令，封装成HTTP的OpenAPI，提供给FaaS调用，自己控制这个API的请求频率以及限流降级。

* 这个后端服务本身则可以通过连接池、MySQL集群等方式去优化。

* 各大云服务商自身也在改造自己的后端服务，BaaS这个集合也在日渐壮大。

<img src="https://img-blog.csdnimg.cn/fd6ea7b8f3c546b397dcab93a5aa740a.png" style="zoom:25%;" />

基于Serverless架构，我们完全可以把传统的MVC架构转换为BaaS+View+FaaS的组合，重构或实现。

我们最常见的Serverless都是指Serverless Computing架构，也就是由Trigger、FaaS和BaaS架构组成的应用。

**什么是广义Serverless呢？**

广义Serverless，其实就是指服务端免运维，也是未来的主要趋势。

总结来说的话就是，我们日常谈Serverless的时候，基本都是指狭义的Serverless，但当我们提到某个服务Serverless化的时候，往往都是指广义的Serverless。