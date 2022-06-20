---
title: GitHub
categories: 
- 开发工具
- GIT
---

# 搜索

**用文件查找器快速、轻松地搜索仓库中的文件**

在运行时的仓库页面，按键盘上的 t 键，那么GitHub就会激活文件查找器。

然后你只需要输入目标文件名，比如`ServiceProvider.cs` 文件，文件查找器就会显示你想要的文件。

**使用搜索限定词搜索你想要的目标**

现在，假设你不知道目标文件位于哪个仓库中，或者你想在组织中查找某个用户。

然后，你可以使用搜索限定词在GitHub的任何页面上搜索所需的目标。

我们在Marketplace页面上，并希望在dotnet组织中搜索`ConfigurationBuilder.cs`文件。

然后，你只需要输入搜索限定词即可表明此目的。

```
org:dotnet filename:ConfigurationBuilder.cs
```

**根据仓库创建时间**

使用 created 限定符搜索。

| 限定符               | 举例                                                         |
| -------------------- | ------------------------------------------------------------ |
| `created:YYYY-MM-DD` | `python created:<2020-06-08`   意思：搜索2020年6月8日之前创建、具有“python”字样的仓库。 |

**根据仓库上传更新时间**

使用 pushed 限定符搜索。

| 限定符              | 举例                        |
| ------------------- | --------------------------- |
| `pushed:YYYY-MM-DD` | `python pushed:>2020-06-08` |

**根据文件内容或路径**

使用 in  限定符搜索。

使用`in`限定符，根据文件内容、文件路径来搜索，或满足这两个条件其中之一的条件搜索。

如果不使用`in`限定符，则只是搜索文件内容。

| 限定符         | 举例                                                         |
| -------------- | ------------------------------------------------------------ |
| `in:file`      | `demo in:file`   意思：搜索文件内容中出现“demo”的代码。      |
| `in:path`      | `demo in:path`   意思：搜索文件路径中出现“demo”的代码。      |
| `in:file,path` | `demo in:file,path`  意思：搜索文件内容或文件路径中出现“demo”的代码。 |

**仓库星标数量**

使用 stars  限定符搜索。

根据仓库星标数量进行大于、小于或范围限定搜索。

| 限定符                           | 举例                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| `stars:n`                        | `stars:100`   意思：搜索刚好100星标的仓库。                  |
| stars:n..m`                      | `stars:10..20`   意思：搜索星标数是10到20的仓库。            |
| `stars:>=n fork:true language:m` | `stars:>=100 fork:true language:python`   意思：搜索大于或等于100星标（包括分叉的星标），以python编写的仓库。 |

**仓库名称或自述文件（README）**

使用 in 限定符搜索。

通过`in`限定符，将搜索限制为根据仓库创建时间、仓库名称、仓库星标数等条件搜索（或这些条件的任意组合）。

如果不用`in`限定符，则只是搜索仓库名称和仓库说明。

| 限定符                         | 举例                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| `in:name`                      | `python in:name`   意思：搜索名称中有“python”字样的仓库。    |
| `in:description`  或`in:about` | `in:description demo`   意思：搜索简介有“demo”字样的仓库。   |
| `in:name,description`          | `python in:name,description`  意思：搜索名称或说明中有“python”字样的仓库。 |
| `in:readme`                    | `python in:readme`    意思：搜索自述文件中提及“python”的仓库。 |
| `repo:owner/name`              | `repo:TheAlgorithms/python`   意思：搜索TheAlgorithms的python仓库（搜索特定仓库名称）。 |

**根据文件名**

使用 filename 限定符搜索。

使用`filename`限定符根据文件名搜索代码文件。

| 限定符                                               | 举例                                                         |
| ---------------------------------------------------- | ------------------------------------------------------------ |
| `filename:FILENAME`                                  | `filename:demo`   意思：搜索名为“demo”的文件。               |
| `filename:FORMAT`                                    | `filename:.txt demo`   意思：搜索具有“demo”字样的.txt文件。  |
| `filename:FILENAME path:DIRECTORY language:LANGUAGE` | `filename:test path:demo language:python`   意思：搜索demo目录中名为test的python文件。 |

# 基本概念

**Issue**

发现代码 Bug，但是目前没有成型代码，需要讨论，即提出建议和意见的反馈区。

<img src="https://img-blog.csdnimg.cn/23b203314c2f4caaa4515c8f28f4cc60.png" style="zoom:25%;" />

**Stars**

打开对应的项目主页，点击右上角star按钮即可收藏。

**关注（Watch）**

当你选择Watching，表示你以后会关注这个项目的所有动态，以后只要这个项目发生变动，如被别人提交了pull request、被别人发起了issue等等情况，你都会在自己的个人通知中心，收到一条通知消息，如果你设置了个人邮箱，那么你的邮箱也可能收到相应的邮件。

**发起请求（Pull Request）**

张三修改了fork的项目中的文件，希望更新到原来的仓库，这时候他要新建一个pull request。

之后原用户查看后可以确定合并，点击conform merge即可。

**复制克隆项目（Fork）**

张三fork了李四的项目，相当于复制了李四的项目，所以自己也独立有了一个一样名称的仓库（注：该仓库会声明来自于李四，但是独立存在的）。

# 其他

**Trending排行榜**

https://github.com/trending/java

**Github Topics**

Github Topics展示了最新、最热门的讨论主题，宣传语是浏览GitHub上的热门话题。

在这里不仅可以看到开源项目，还可以看到一些非开发技术的讨论主题。

地址：https://github.com/topics

**Github Explore**

Github Explore这里是根据你平时的兴趣，推荐一些项目。

地址：https://github.com/explore

# 插件

**源码查看插件推荐**

> 下载地址：[https://github.com/buunguyen/octotree](https://link.zhihu.com/?target=https%3A//github.com/buunguyen/octotree)

安装完之后，访问Github 会在左边出现一个树形。

**Refined GitHub**

1. GitHub只能下载单个文件，不能下载某个目录，该插件可以解决这个问题
2. GitHub文件浏览器优化
3. 能收缩得文件目录
4. 显示共同关注

**Github1s**

这本质上一个 Web 版本的 VSCode，我觉得这比 Octotree 更好用。

**Sourcegraph**

**翻译插件**

https://github.com/k1995/github-i18n-plugin

**镜像加速下载**

https://greasyfork.org/zh-CN/scripts/412245-github-增强-高速下载

**Enhanced GitHub**

- 显示仓库大小
- 显示每个活动分支中的每个文件大小（不适用于文件夹 / 符号链接）
- 显示每个文件的下载链接（不适用于文件夹 / 符号链接）
- 将文件内容直接复制到剪贴板（对于 markdown 文件不起作用）
- 在查看文件内容时下载文件

**GitHub File Icons**

**GitHub Hovercard**

如果你想在GitHub中快速了解到一个作者或者一个仓库甚至一个issues的信息的时候，你可以使用这款插件。