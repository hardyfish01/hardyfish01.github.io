---
title: TCP IP基础知识
categories: 
- 计算机基础
- 网络相关
---

# TCP和IP模型

OSI模型注重通信协议必要的功能；TCP/IP更强调在计算机上实现协议应该开发哪种程序。

**TCP/IP划分了四层网络模型**

- 第一层：应用层，主要有负责web浏览器的HTTP协议， 文件传输的FTP协议，负责电子邮件的SMTP协议，负责域名系统的DNS等
- 第二层：传输层，主要是有**可靠传输**的TCP协议，特别**高效**的UDP协议。主要负责传输应用层的数据包。
- 第三层：网络层，主要是IP协议。主要负责寻址（找到目标设备的位置）
- 第四层：数据链路层，主要是负责转换数字信号和物理二进制信号。

<img src="https://img-blog.csdnimg.cn/2c306962843a4ea59c890f3dd727202a.png" style="zoom:25%;" />

**四层网络协议的作用**

- 发送端是由上至下，把上层来的数据在头部加上各层协议的数据（部首）再下发给下层。
- 接受端则由下而上，把从下层接受到的数据进行解密和去掉头部的部首后再发送给上层。
- 层层加密和解密后，应用层最终拿到了需要的数据。

**举个例子：**

我们需要发送一个**index.html**。

* 两台电脑在应用层都使用HTTP协议（即都使用浏览器）。

* 在传输层，TCP协议会将HTTP协议发送的数据看作一个数据包，并在这个数据包前面加上TCP包的一部分信息（部首）

* 在网络层，IP协议会将TCP协议要发送的数据看作一个数据包，同样的在这个数据包前端加上IP协议的部首

* 在数据链路层，对应的协议也会在IP数据包前端加上以太网的部首。

<img src="https://img-blog.csdnimg.cn/270ea156a87d470c938049611cfae9ad.png" style="zoom:50%;" />

* 源设备和目标设备通过网线连接，就可以通过物理层的二进制传输数据。

* 数据链路层，会使用对应的协议找到物理层的二进制数据，解码得到以太网的部首信息和对应的IP数据包，再将IP数据包传给上层的网络层。

* 数据链路层>网络层>传输层>应用层，一层层的解码，最后就可以在浏览器中得到目标设备传送过来的**index.html**。

<img src="https://img-blog.csdnimg.cn/b3a1a77662fa4d2a9052033792e2bc64.png" style="zoom:50%;" />

**TCP/IP协议族**

从字面意义上来讲，TCP/IP是指**传输层**的TCP协议和**网络层**的IP协议。

实际上，TCP/IP只是利用 IP 进行通信时所必须用到的协议群的统称。

具体来说，在网络层是IP/ICMP协议、在传输层是TCP/UDP协议、在应用层是SMTP、FTP、以及 HTTP 等。他们都属于 TCP/IP 协议。

# 网络层

**MAC地址**

MAC称为物理地址，也叫硬件地址，用来定义网络设备的位置，MAC地址是网卡出厂时设定的，是固定的（但可以通过在设备管理器中或注册表等方式修改，同一网段内的MAC地址必须唯一）。

MAC地址采用十六进制数表示，长度是6个字节（48位），分为前24位和后24位。

> MAC地址对应于OSI参考模型的第二层数据链路层，工作在数据链路层的交换机维护着计算机MAC地址和自身端口的数据库，交换机根据收到的数据帧中的目的MAC地址字段来转发数据帧。

MAC地址更像是身份证，是一个唯一的标识。它的唯一性设计是为了组网的时候，不同的网卡放在一个网络里面的时候，可以不用担心冲突。从硬件角度，保证不同的网卡有不同的标识。

## IP地址

常见的IP地址分为IPv4与IPv6两大类，当前广泛应用的是IPv4，目前IPv4几乎耗尽，下一阶段必然会进行版本升级到IPv6；

IP地址是以网络号和主机号来标示网络上的主机的，我们把网络号相同的主机称之为本地网络，网络号不相同的主机称之为远程网络主机

* 本地网络中的主机可以直接相互通信；远程网络中的主机要相互通信必须通过本地网关（Gateway）来传递转发数据。

IP地址对应于OSI参考模型的第三层网络层，工作在网络层的路由器根据目标IP和源IP来判断是否属于同一网段，如果是不同网段，则转发数据包。

**IP地址格式和表示**

IP地址(IPv4)由32位二进制数组成，分为4段（4个字节），每一段为8位二进制数（1个字节）

* 每一段8位二进制，中间使用英文的标点符号`.`隔开

由于二进制数太长，为了便于记忆和识别，把每一段8位二进制数转成十进制，大小为0至255。

IP地址的这种表示法叫做**点分十进制表示法**。

* 举个栗子：`210.21.196.6`就是一个IP地址的表示。

计算机的IP地址由两部分组成，一部分为网络标识，一部分为主机标识，同一网段内的计算机网络部分相同，主机部分不能同时重复出现。

* **路由器**连接不同网段，负责不同网段之间的数据转发，**交换机**连接的是同一网段的计算机。

通过设置网络地址和主机地址，在互相连接的整个网络中保证每台主机的IP地址不会互相重叠，即IP地址具有了唯一性。

**IP地址分类**

IP地址分A、B、C、D、E五类，其中A、B、C这三类是比较常用的IP地址，D、E类为特殊地址。

<img src="https://img-blog.csdnimg.cn/2c725f4c914a4d46ac9e625d3532a036.png" style="zoom:30%;" />

**公有IP地址和私有IP地址**

![](https://img-blog.csdnimg.cn/06628445eb4444b3a31ec25e89098934.png)

平时我们看到的数据中心里，办公室、家里或学校的IP地址，一般都是私有IP地址段。因为这些地址允许组织内部的IT人员自己管理、自己分配，而且可以重复。因此，你学校的某个私有IP地址段和我学校的可以是一样的。

* 这就像每个小区有自己的楼编号和门牌号，你们小区可以叫6栋，我们小区也叫6栋，没有任何问题。

* 但是一旦出了小区，就需要使用公有IP地址。就像人民路888号，是国家统一分配的，不能两个小区都叫人民路888号。

公有IP地址有个组织统一分配，你需要去买。如果你搭建一个网站，给你学校的人使用，让你们学校的IT人员给你一个IP地址就行。

但是假如你要做一个类似网易163这样的网站，就需要有公有IP地址，这样全世界的人才能访问。

* 表格中的`192.168.0.x`是最常用的私有IP地址。

* 你家里有Wi-Fi，对应就会有一个IP地址。一般你家里地上网设备不会超过256个。

不需要将十进制转换为二进制32位，就能明显看出192.168.0是网络号，后面是主机号。

而整个网络里面的第一个地址192.168.0.1，往往就是你这个私有网络的出口地址。

例如，你家里的电脑连接Wi-Fi，Wi-Fi路由器的地址就是192.168.0.1，而192.168.0.255就是广播地址。

* 一旦发送这个地址，整个192.168.0网络里面的所有机器都能收到。

**动态主机配置协议（DHCP）**

配置了IP之后一般不能变的，配置一个服务端的机器还可以，但是如果是客户端的机器呢？我抱着一台笔记本电脑在公司里走来走去，或者白天来晚上走，每次使用都要配置IP地址，那可怎么办？

* 还有人事、行政等非技术人员，如果公司所有的电脑都需要IT人员配置，肯定忙不过来啊。

因此，我们需要有一个自动配置的协议，也就是**动态主机配置协议（Dynamic Host Configuration Protocol）**，简称**DHCP**。

* 有了这个协议，网络管理员就轻松多了。他只需要配置一段共享的IP地址。

* 每一台新接入的机器都通过DHCP协议，来这个共享的IP地址里申请，然后自动配置好就可以了。

* 等人走了，或者用完了，还回去，这样其他的机器也能用。

所以说，**如果是数据中心里面的服务器，IP一旦配置好，基本不会变，这就相当于买房自己装修。DHCP的方式就相当于租房。你不用装修，都是帮你配置好的。你暂时用一下，用完退租就可以了。**

一个广播的网络里面接入了N台机器，我怎么知道每个MAC地址是谁呢？这就是**ARP协议**，也就是已知IP地址，求MAC地址的协议。

<img src="https://img-blog.csdnimg.cn/36995f9f91204a4580d24cd8cdfb2cf1.png" style="zoom:25%;" />

为了避免每次都用ARP请求，机器本地也会进行ARP缓存。当然机器会不断地上线下线，IP也可能会变，所以ARP的MAC地址缓存过一段时间就会过期。

**ICMP协议**

ping的操作是基于ICMP协议工作的。

* **ICMP**全称**Internet Control Message Protocol**，就是**互联网控制报文协议**。

网络包在异常复杂的网络环境中传输时，常常会遇到各种各样的问题。

* 当遇到问题的时候，总不能死个不明不白，要传出消息来，报告情况，这样才可以调整传输策略。

这就相当于我们经常看到的电视剧里，古代行军的时候，为将为帅者需要通过侦察兵、哨探或传令兵等人肉的方式来掌握情况，控制整个战局。

ICMP报文是封装在IP包里面的。因为传输指令的时候，肯定需要源地址和目标地址。它本身非常简单。

<img src="https://img-blog.csdnimg.cn/a104b3fd98b04fb18b0657c297e817c0.png" style="zoom:25%;" />

**ARP协议**

ARP即地址解析协议， 用于实现从 IP 地址到 MAC 地址的映射，即询问目标IP对应的MAC地址。

**ARP协议的工作过程**

* 首先，每个主机都会有自己的ARP缓存区中建立一个ARP列表，以表示IP地址和MAC地址之间的对应关系

当源主机要发送数据时，首先检测ARP列表中是否对应IP地址的目的主机的MAC地址，如果有，则直接发送数据，如果没有，就向本网段的所有主机发送ARP数据包

当本网络的所有主机收到该ARP数据包时，首先检查数据包中的IP地址是否是自己的IP地址，如果不是，则忽略该数据包，如果是，则首先从数据包中取出源主机的IP和MAC地址写入到ARP列表中，如果存在，则覆盖然后将自己的MAC地址写入ARP响应包中，告诉源主机自己是它想要找的MAC地址

源主机收到ARP响应包后，将目的主机的IP和MAC地址写入ARP列表，并利用此信息发送数据，如果源主机一直没有收到ARP响应数据包，表示ARP查询失败。

## 子网掩码

**子网掩码的概念及作用**

通过子网掩码，才能表明一台主机所在的子网与其他子网的关系，使网络正常工作。

* 子网掩码和IP地址做与运算，分离出IP地址中的网络地址和主机地址，用于判断该IP地址是在本地网络上，还是在远程网络网上。

子网掩码还用于将网络进一步划分为若干子网，以避免主机过多而拥堵或过少而IP浪费。

**子网掩码的组成**

同IP地址一样，子网掩码是由长度为32位二进制数组成的一个地址。

子网掩码32位与IP地址32位相对应，IP地址如果某位是网络地址，则子网掩码为1，否则为0。

* 举个栗子：如：`11111111.11111111.11111111.00000000`

> 左边连续的1的个数代表网络号的长度，（使用时必须是连续的，理论上也可以不连续），右边连续的0的个数代表主机号的长度。

**为什么要使用子网掩码**

两台主机要通信，首先要判断是否处于同一网段，即网络地址是否相同。

* 如果相同，那么可以把数据包直接发送到目标主机，否则就需要路由网关将数据包转发送到目的地。

> 可以这么简单的理解：A主机要与B主机通信，A和B各自的IP地址与A主机的子网掩码进行And与运算，看得出的结果：
>
> 1、结果如果相同，则说明这两台主机是处于同一个网段，这样A可以通过ARP广播发现B的MAC地址，B也可以发现A的MAC地址来实现正常通信。
>
> 2、如果结果不同，ARP广播会在本地网关终结，这时候A会把发给B的数据包先发给本地网关，网关再根据B主机的IP地址来查询路由表，再将数据包继续传递转发，最终送达到目的地B。

------

> 计算机的网关（Gateway）就是到其他网段的出口，也就是路由器接口IP地址。
>
> 路由器接口使用的IP地址可以是本网段中任何一个地址，不过通常使用该网段的第一个可用的地址或最后一个可用的地址，这是为了尽可能避免和本网段中的主机地址冲突。

在如下拓扑图示例中，A与B，C与D，都可以直接相互通信（都是属于各自同一网段，不用经过路由器）

但是A与C，A与D，B与C，B与D它们之间不属于同一网段，所以它们通信是要经过本地网关，然后路由器根据对方IP地址，在路由表中查找恰好有匹配到对方IP地址的直连路由，于是从另一边网关接口转发出去实现互连

<img src="https://img-blog.csdnimg.cn/a3a6f3542c5e4bca810d056157362b5c.png" style="zoom:50%;" />

**子网掩码和IP地址的关系**

* 子网掩码是用来判断任意两台主机的IP地址是否属于同一网络的依据

* 拿双方主机的IP地址和自己主机的子网掩码做与运算，如结果为同一网络，就可以直接通信

**如何根据IP地址和子网掩码，计算网络地址：**

* 将IP地址与子网掩码转换成二进制数。

* 将二进制形式的 IP 地址与子网掩码做与运算。

* 将得出的结果转化为十进制，便得到网络地址。

<img src="https://img-blog.csdnimg.cn/f243e18a4e134ce388aa8f85ecdf2a2d.png" style="zoom:50%;" />

## 网关

**怎么在宿舍上网？**

还记得咱们在宿舍的时候买了台交换机，几台机器组了一个局域网打游戏吗？

* 可惜啊，只能打局域网的游戏，不能上网啊！盼啊盼啊，终于盼到大二，允许宿舍开通网络了。

学校给每个宿舍的网口分配了一个IP地址。

* 这个IP是校园网的IP，完全由网管部门控制。宿舍网的IP地址多为192.168.1.x。校园网的IP地址，假设是10.10.x.x。

这个时候，你要在宿舍上网，有两个办法：

* 第一个办法，让你们宿舍长再买一个网卡。这个时候，你们宿舍长的电脑里就有两张网卡。一张网卡的线插到你们宿舍的交换机上，另一张网卡的线插到校园网的网口。而且，这张新的网卡的IP地址要按照学校网管部门分配的配置，不然上不了网。**这种情况下，如果你们宿舍的人要上网，就需要一直开着宿舍长的电脑。**

* 第二个办法，你们共同出钱买个家庭路由器。家庭路由器会有内网网口和外网网口。把外网网口的线插到校园网的网口上，将这个外网网口配置成和网管部的一样。内网网口连上你们宿舍的所有的电脑。**这种情况下，如果你们宿舍的人要上网，就需要一直开着路由器。**

> 这两种方法其实是一样的。只不过第一种方式，让你的宿舍长的电脑，变成一个有多个口的路由器而已。

而你买的家庭路由器，里面也跑着程序，和你宿舍长电脑里的功能一样，只不过是一个嵌入式的系统。

* 当你的宿舍长能够上网之后，接下来，就是其他人的电脑怎么上网的问题。这就需要配置你们的**网卡。**

* 当然DHCP是可以默认配置的。在进行网卡配置的时候，除了IP地址，还需要配置一个Gateway**的东西，这个就是**网关。

网关实质上是一个网络通向其他网络的IP地址。

* 比如有网络A和网络B，网络A的IP地址范围为`192.168.1.1~192. 168.1.254`，子网掩码为`255.255.255.0`；

* 网络B的IP地址范围为`192.168.2.1~192.168.2.254`，子网掩码为`255.255.255.0`。

在没有路由器的情况下，两个网络之间是不能进行TCP/IP通信的，即使是两个网络连接在同一台交换机(或集线器)上，TCP/IP协议也会根据子网掩码(`255.255.255.0`)判定两个网络中的主机处在不同的网络里。

* 而要实现这两个网络之间的通信，则必须通过网关。

如果网络A中的主机发现数据包的目的主机不在本地网络中，就把数据包转发给它自己的网关，再由网关转发给网络B的网关，网络B的网关再转发给网络B的某个主机。网络B向网络A转发数据包的过程。

**所以说，只有设置好网关的IP地址，TCP/IP协议才能实现不同网络之间的相互通信。**

> 那么这个IP地址是哪台机器的IP地址呢？

网关的IP地址是具有路由功能的设备的IP地址，具有路由功能的设备有路由器、启用了路由协议的服务器(实质上相当于一台路由器)、代理服务器(也相当于一台路由器)。

* **如果是同一个网段**，例如，你访问你旁边的兄弟的电脑，那就没网关什么事情，直接将源地址和目标地址放入IP头中，然后通过ARP获得MAC地址，将源MAC和目的MAC放入MAC头中，发出去就可以了。

* **如果不是同一网段**，例如，你要访问你们校园网里面的BBS，该怎么办？

这就需要发往默认网关Gateway。Gateway的地址一定是和源IP地址是一个网段的。

**如何发往默认网关呢？**

网关不是和源IP地址是一个网段的么？

* 这个过程就和发往同一个网段的其他机器是一样的：将源地址和目标IP地址放入IP头中，通过ARP获得网关的MAC地址，将源MAC和网关的MAC放入MAC头中，发送出去。

* 网关所在的端口，例如`192.168.1.1/24`将网络包收进来，然后接下来怎么做，就完全看网关的了。

**网关往往是一个路由器，是一个三层转发的设备。**

* 三层设备？就是把MAC头和IP头都取下来，然后根据里面的内容，看看接下来把包往哪里转发的设备。

在你的宿舍里面，网关就是你宿舍长的电脑。一个路由器往往有多个网口，如果是一台服务器做这个事情，则就有多个网卡，其中一个网卡是和源IP同网段的。

很多情况下，人们把网关就叫做路由器。其实不完全准确，而另一种比喻更加恰当：**路由器是一台设备，它有五个网口或者网卡，相当于有五只手，分别连着五个局域网。每只手的IP地址都和局域网的IP地址相同的网段，每只手都是它握住的那个局域网的网关。**

任何一个想发往其他局域网的包，都会到达其中一只手，被拿进来，拿下MAC头和IP头，看看，根据自己的路由算法，选择另一只手，加上IP头和MAC头，然后扔出去。

**静态路由是什么？**

大致可以分为两类，一个是**静态路由**，一个是**动态路由**。

* **静态路由，其实就是在路由器上，配置一条一条规则。**

这些规则包括：想访问BBS站（它肯定有个网段），从2号口出去，下一跳是IP2；

想访问教学视频站（它也有个自己的网段），从3号口出去，下一跳是IP3，然后保存在路由器里。

* 每当要选择从哪只手抛出去的时候，就一条一条的匹配规则，找到符合的规则，就按规则中设置的那样，从某个口抛出去，找下一跳IPX。

MAC地址是一个局域网内才有效的地址。因而，MAC地址只要过网关，就必定会改变，因为已经换了局域网。

* 两者主要的区别在于IP地址是否改变。不改变IP地址的网关，我们称为**转发网关；改变IP地址的网关，我们称为NAT网关**。

现在大家每家都有家用路由器，家里的网段都是`192.168.1.x`，所以你肯定访问不了你邻居家的这个私网的IP地址的。

* 所以，当我们家里的包发出去的时候，都被家用路由器NAT成为了运营商的地址了。

很多办公室访问外网的时候，也是被NAT过的，因为不可能办公室里面的IP也是公网可见的，公网地址实在是太贵了，所以一般就是整个办公室共用一个到两个出口IP地址。

**路由协议**

家里的网段是私有网段，出去的包需要NAT成公网的IP地址，因而路由器是一个NAT路由器。

两个运营商都要为这个网关配置一个公网的IP地址。如果你去查看你们家路由器里的网段，基本就是我图中画的样子。

<img src="https://img-blog.csdnimg.cn/b1175b1f0f4f45c7b4a51a70aa817557.png" style="zoom:25%;" />

## DNS

DNS通过主机名，最终得到该主机名对应的IP地址的过程叫做域名解析（或主机名解析）。

* **通俗的讲**，我们更习惯于记住一个网站的名字，www.baidu.com，而不是记住它的ip地址，比如：167.23.10.2

**工作原理**

将主机域名转换为ip地址，属于应用层协议，使用UDP传输。

<img src="https://img-blog.csdnimg.cn/76f522fa8e764b3c84be4bd6d18638bc.png" style="zoom:50%;" />

<img src="https://img-blog.csdnimg.cn/63167cd00b9046fd85c5ccb580cd7c31.png" style="zoom:50%;" />

* 第一步，客户端向本地DNS服务器发送解析请求

* 第二步，本地DNS如有相应记录会直接返回结果给客户端，如没有就向DNS根服务器发送请求

* 第三步,DSN根服务器接收到请求，返回给本地服务器一个所查询域的主域名服务器的地址

* 第四步，本地dns服务器再向返回的主域名服务器地址发送查询请求

* 第五步，主域名服务器如有记录就返回结果，没有的话返回相关的下级域名服务器地址

* 第六步，本地DNS服务器继续向接收到的地址进行查询请求

* 第七步，下级域名服务器有相应记录，返回结果

* 第八步，本地dns服务器将收到的返回地址发给客户端，同时写入自己的缓存，以便下次查询

DNS域名查询实际上就是个不断递归查询的过程，直到查找到相应结果，需要注意的时，当找不到相应记录，会返回空结果，而不是超时信息
