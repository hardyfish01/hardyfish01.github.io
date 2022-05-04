---
title: IDEA插件
categories: 
- 公众号文章
---

# 前言

"工欲善其事 必先利其器"，一些好的工具确实可以提高工作效率，之前分享过几篇工具文章

- [分享一些工作中常用的工具软件，值得收藏！](https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247485866&idx=1&sn=d753c520434e4f6ef04c2e29635e8b14&scene=21#wechat_redirect)
- [分享一些常用的网站和工具，值得收藏！](https://mp.weixin.qq.com/s?__biz=MzUyOTg1OTkyMA==&mid=2247485782&idx=1&sn=da568c665d8aa5ec44e302ac09d84fcd&scene=21#wechat_redirect)

本篇文章主要分享一些工作中常用的IDEA插件，希望对大家有帮助！

文章首发在公众号（月伴飞鱼），之后同步到个人网站：xiaoflyfish.cn/

![](https://img-blog.csdnimg.cn/7e2573f7da834e6fa6650992593020f7.png)

喜欢的话，之后会分享更多系列文章！

**觉得有收获，希望帮忙点赞，转发下哈，谢谢，谢谢**

微信搜索：月伴飞鱼，交个朋友，进面试交流群

# Java Stream Debugger

JDK1.8新增的Stream流操作，极大地提升了编程快感，也精简了代码。

同时，存在一个问题，debugger下不易调试，不能一行一行地看到执行结果。

Java Stream Debugger 这个插件解决了此问题。

如下代码：

```java
public class Main {
    public static void main(String[] args) {
        List list = new ArrayList();
        list.add("月");
        list.add("伴");
        list.add("飞");
        list.add("鱼");
        list.stream().distinct().findFirst().get();
    }
}
```

使用插件调试：

![](https://img-blog.csdnimg.cn/4c2f9f3fb25b4e8591154b69d236ce08.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Jclasslib bytecode viewer

jclasslib bytecode viewer是一个可以可视化已编译Java类文件和所包含的字节码的工具。

使用时直接选择 View --> Show Bytecode With jclasslib

> 注意：如果是自己项目的源码需要先编译

![](https://img-blog.csdnimg.cn/e139e47a62d7488b93e45d36686be3cd.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# LeetCode editor

LeetCode刷算法神器，可以拉取到LeetCode题，提交代码到LeetCode帮忙执行，非常赞。

![](https://img-blog.csdnimg.cn/fb7a149a0e7b43448e0ad5121628640b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Maven Helper

此插件可用来方便显示Maven的依赖树，在没有此插件时，如果想Maven的依赖树需要输入命令行：`mvn dependency:tree` 才可查看依赖。

如果想看是否有依赖包冲突的话也需要输入命令行等等的操作。

而如果安装Maven Helper插件就可免去命令行困扰，通过界面即可操作完成。

![](https://img-blog.csdnimg.cn/a7576e35be1a42afbd3a29e4c74b5919.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

相应操作：

1. Conflicts（查看冲突）
2. All Dependencies as List（列表形式查看所有依赖）
3. All Dependencies as Tree（树形式查看所有依赖）

# Jrebel

日常开发中，当你项目在debug 的时候，修改任意一个 java 文件时，tomcat并不能将此文件的修改实时编译并反映到运行的项目中去，所以只能重启项目，这个过程是相当痛苦的，尤其是项目稍微大点的话，开发期间可能你就是一直在重启项目。

JRebel是一款JVM插件，它使得Java代码修改后不用重启系统，立即生效。

# GenerateAllSetter

该插件作用是可以快速针对已有的model实体对象的属性生产set代码，免去开发者在开发过程中set属性值时还需要去实体对象中翻查的时间，生成的同时会附带类型默认值

![](https://img-blog.csdnimg.cn/987ce1ad7ab54173abb514b3e82d55f0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# VisualVM Launcher

这个插件一般可用于在本地开发进行压力测试，性能测试之类的监控器，其他场景一般不推荐使用此模式启动。

会启动另外一个Visual VM窗口，这个窗口是JDK bin目录下的JvisualVM 。

配置地址：

![](https://img-blog.csdnimg.cn/e68821cb4bd04978a7c7e8468be0b859.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/b9cd4e4fa8dc4cc0b98f30d757de0136.png)

# SequenceDiagram

在接手老项目时，一上手很难窥到全貌，这时候要是能够把接口的调用关系，整个时序图展示出来，对深入了解项目帮助很大。

有这么一款插件SequenceDiagram能够根据方法的调用关系，自动生成执行时序图。

安装完成后，在某个类的某个函数中，右键 --> Sequence Diagaram即可调出。

![](https://img-blog.csdnimg.cn/366e5c01ba2045c0858b56f40441252b.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Auto filling Java call arguments

代码生成插件，通过快捷键自动补全函数的调用参数，针对包含大量参数的构造函数和方法非常有用！

![](https://img-blog.csdnimg.cn/8cc3c59c711d4b688de7aaf8a4a3cc3f.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# GitToolBox

配合版本控制工具Git使用，非常直观显示当前项目分支，及代码未更新，未提交数目，省去查询分支和最新代码等不必要的麻烦。

# IntelliJad

IntelliJad是一个Java class文件的反编译工具，需要在 setting 中设置本地`Java jad.exe`工具的地址。

找到一个Jar包选择class文件右键Decompile，会出现反编译的结果。

# Key Promoter X

这个Key Promoter X插件就会用对话框的形式提醒你使用快捷键操作。

非常适合不熟悉jetbrains全家桶IDE的同学，学习使用快捷键。

![](https://img-blog.csdnimg.cn/65a164859fb54e6494fad3b6b68d0bcf.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Code Screenshots

代码截图工具，有了它可以快速截出漂亮的代码。

配置地址：

![](https://img-blog.csdnimg.cn/d36276f5ae284558939559da4c3713a1.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Codota AI Autocomplete for Java

这款插件基于AI技术，通过对大量开源的项目学习，帮助开发者提供更可靠的智能推荐，让编程变得更方便！

官网地址：https://www.codota.com/signin/get

下载安装：

![](https://img-blog.csdnimg.cn/916f6302b3834410b8156efac65dde38.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

**代码提示：**

当你对写代码的时候的时候，Codota便会根据它学习到代码知识，给出相应的建议，使用的比例。

![](https://img-blog.csdnimg.cn/fc11c7d15be340de90ddd0d09baa7625.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

**示例代码**

如果你要找某个类的示例代码，只需要选择某个类名，点击右键选择Get relevant examples。

便可以获取到在github、stackoverflow等上排名最高的片段，并在IDE中显示出来，更快、更方便。

![](https://img-blog.csdnimg.cn/854b141a818f480a8d1df5e91ce8e947.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

也可以通过搜索方式，支持JDK和知名第三方库的函数的使用方法搜索，可以看到其他知名开源项目对该函数的用法 。

![](https://img-blog.csdnimg.cn/81b5da7a6b214287a866a8e10fee1e3c.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_19,color_FFFFFF,t_70,g_se,x_16)

![](https://img-blog.csdnimg.cn/0f0b9dee001d41f2848234c112e2f1dc.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Alibaba Java Coding Guidelines

为了让开发者更加方便、并且达到快速规范代码格式的目的并实行起来，阿里巴巴基于《阿里巴巴Java开发规约》手册内容，研发了一套自动化的IDE检测插件，它就是Alibaba Java Coding Guidelines 插件。

该插件由阿里巴巴 P3C 项目组研发。

- 代码已经开源，GitHub：https://github.com/alibaba/p3c
- 阿里介绍文章：https://mp.weixin.qq.com/s/IbibsXlWHlM59kfXJqRvZA#rd

**如何使用：**

在你的项目上或者选中某一个类点击右键就可以看到

![](https://img-blog.csdnimg.cn/24f69e9aa49e4ce0b48a7eb2461883d9.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

**扫描出坏代码**

![](https://img-blog.csdnimg.cn/58b1e20411074a84933c3f60b6fef494.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Material Theme UI

工具的颜值也很重要，好的主题让人赏心悦目，有码代码的欲望，这是一个IDEA颜值类插件：Material Theme UI

**设置**

在这里对Material Theme UI插件进行设置

> File > Settings > Appearance & Behavior > Material Theme

![](https://img-blog.csdnimg.cn/64808fe9b8af4c9eab4fb657436b29a2.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Translation

Translation是一款非常好用的翻译插件，可以随时随地翻译单词、甚至一段话，不再需要额外打开浏览器搜索翻译网站了！

![](https://img-blog.csdnimg.cn/b574521781884a50bef437b1c5d555a0.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Properties to YAML Converter

这个插件可以将Properties 配置文件一键转换成YAML 文件，很实用的一个插件。

![](https://img-blog.csdnimg.cn/e2bf12d5f89c44a3b93092156f8ac2f6.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Hiberbee Theme

一款受到了 Monokai Pro 和 MacOS Mojave 启发的主题，是一款色彩层次分明的深色主题。

这个主题的效果图如下，看着也是非常赞！适合编码！

![](https://img-blog.csdnimg.cn/b6c230a3ba7c42168a8aacbb27ef2072.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

**使用主题包**

推荐一个网站color-themes

> http://color-themes.com/

# GsonFormatPlus

一个非常实用的插件，它可以将`JSON`字符串自动转换成`Java`实体类。

在和其他系统对接时，往往以`JSON`格式传输数据，而我们需要用`Java`实体接收数据入库或者包装转发，如果字段太多一个一个编写那就太麻烦了。

![](https://img-blog.csdnimg.cn/61a0a370fe0c4d3abb3bf706ad89f0ea.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# Grep Console

默认情况下，IDEA控制台窗口在打印日志时都是一种颜色到底，比如各种info，warn，和error等类型的日志信息交织在一起，不好分辨。

Grep Console插件则可以帮助我们自定义设置不用日志用不同的颜色进行标识，非常方便观看！

自定义配置：

![](https://img-blog.csdnimg.cn/f14b0de3aa984da5b52d56b6df24119a.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)

# JOL Java Object Layout

查看对象布局和大小的插件！

![](https://img-blog.csdnimg.cn/101644056a264bbc848abadb46ca3476.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16)