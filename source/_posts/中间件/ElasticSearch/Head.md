---
title: Head
categories: 
- 中间件
- Elasticsearch
---

**Elasticsearch Head** 是一个为 **ES** 开发的一个页面客户端工具，其源码托管于Github

> Github地址：https://github.com/mobz/elasticsearch-head

Head提供了以下安装方式

- 源码安装，通过`npm run start` 启动（不推荐）
- 通过docker安装（推荐）
- 通过chrome插件安装（推荐）
- 通过 ES 的plugin方式安装（不推荐）

**通过Docker方式安装**

```
#拉取镜像
docker pull mobz/elasticsearch-head:5
#创建容器
docker create --name elasticsearch-head -p 9100:9100 mobz/elasticsearch-head:5
#启动容器
docker start elasticsearch-head
```

通过浏览器进行访问：

<img src="https://img-blog.csdnimg.cn/6f24d46e863b402e8cfad8671331b3bf.png" alt="img" style="zoom:25%;" />

**通过Chrome插件安装**

打开 **Chrome** 的应用商店，即可安装

https://chrome.google.com/webstore/detail/elasticsearch-head/ffmkiejjmecolpfloofpjologoblkegm

<img src="https://img-blog.csdnimg.cn/a7d6c9a4e49b4c5892120b896ff902af.png" alt="img" style="zoom:50%;" />