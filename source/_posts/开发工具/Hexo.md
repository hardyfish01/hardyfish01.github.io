---
title: Hexo
categories: 
- 开发工具
---

# 安装

```
npm install -g hexo-cli

hexo init blog
```

输入`hexo g`生成静态网页，然后输入`hexo s`打开本地服务器

**连接GitHub与本地**

首先右键打开Git Bash，然后输入下面命令：

```
git config --global user.name "godweiyang"

git config --global user.email "792321264@qq.com"

生成密钥SSH key：
ssh-keygen -t rsa -C "792321264@qq.com"
```

打开GitHub，在头像下面点击Settings，再点击SSH and GPG keys，新建一个SSH，名字随便。

```
cat ~/.ssh/id_rsa.pub
```

将输出的内容复制到框中，点击确定保存。

输入`ssh -T git@github.com`，出现你的用户名，那就成功了。

# 配置

**主页文章按更新时间排序**

```yaml
index_generator:
  path: ''
  per_page: 20
  order_by: -updated
```

**域名绑定**

cd 你的hexo本地目录， `cd source`，`touch CNAME`创建CNAME文件

```
xiaoflyfish.cn
```

给解析添加A类型：

-  185.199.108.153

-  185.199.109.153

-  185.199.110.153

-  185.199.111.153

**备份**

```
切换并创建一个叫hexo的分支
git checkout -b hexo

# 切换分支
git checkout hexo

将复制过来的文件推送到github
git add .

git commit -m "新建分支"

git remote add origin https://github.com/zine-fj/zine-fj.github.io.git

git push -u origin hexo
```

> 注意：themes下的tree文件夹无法上传，因为有`.git`文件

```
1. 删除子文件夹下.git后，依然无法提交子文件夹下的文件
2. git rm --cached themes/hueman
3. git add .
4. git commit -m "xxx"
5. git push origin master
```

**换电脑配置：**

进入 原来电脑 hexo 博客目录，只拷如下几个目录：

```
scaffolds            文章模版                           必须备份
source               博客文章                           必须备份
themes               主题                              必须备份
.gitignore           限定在push时那些文件可以忽略         必须备份
_config.yml          站点配置文件                       必须备份
package.json         安装包的名称                       必须备份
.ssh                 密钥文件                          必须备份
```

换电脑，同时要把`.git文件`夹复制过去，然后安装相关插件

# 部署

**部署到GitHub**

```xml
deploy:
  type: git
  repository: https://github.com/xiaoflyfish/xiaoflyfish.github.io
  branch: master
```

安装一个扩展`npm i hexo-deployer-git`

最后输入`hexo d`上传到GitHub上