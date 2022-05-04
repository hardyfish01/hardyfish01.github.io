---
title: MAT
categories: 
- 开发工具
---

**安装**

1.安装地址：

https://www.eclipse.org/mat/downloads.php

下载太慢可以更改镜像地址

<img src="https://img-blog.csdnimg.cn/50fa18bd01524e9fb237b6183107738d.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5pyI5Ly06aOe6bG8,size_20,color_FFFFFF,t_70,g_se,x_16" style="zoom:25%;" />

> mat需要高版本java环境。

2.一定要将app移入Applications后再打开，不然会报错

3.加载大dump文件，报错java heap space

编辑文件MemoryAnalyzer.ini

```
-Xmx2048m
```

9个G的dump，我用了16384m。