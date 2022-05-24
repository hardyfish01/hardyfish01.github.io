---
title: Rebase和Merge
categories: 
- 开发工具
- GIT
---

**git rebase 让你的提交记录更加清晰可读**

rebase 翻译为变基，他的作用和 merge 很相似，用于把一个分支的修改合并到当前分支上。

如下图所示，下图介绍了经过 rebase 前后提交历史的变化情况。

<img src="https://img-blog.csdnimg.cn/76d69482cb88477b8563ed995f389f38.png" alt="img" style="zoom:25%;" />

假设我们现在有2条分支，一个为 master ，一个为 feature/1，他们都基于初始的一个提交add readme进行检出分支，之后，master分支增加了`3.js`和`4.js`的文件，分别进行了2次提交，feature/1也增加了`1.js`和`2.js`的文件，分别对应以下2条提交记录。

之后，切换到 **feature/1** 分支下，执行 `git rebase master` ，成功之后，通过 log 查看记录。

下图所示：可以看到先是逐个应用了 mater 分支的更改，然后以 master 分支最后的提交作为基点，再逐个应用 feature/1的每个更改。

<img src="https://img-blog.csdnimg.cn/4e024ef9e9d04941b5e455253249cd56.png" alt="img" style="zoom:25%;" />

所以，我们的提交记录就会非常清晰，没有分叉。

上面演示的是比较顺利的情况。

但是大部分情况下，rebase 的过程中会产生冲突的，此时，就需要手动解决冲突，然后使用`git add 、git rebase --continue`的方式来处理冲突，完成 rebase，如果不想要某次 rebase 的结果，那么需要使用 `git rebase --skip`来跳过这次 rebase。

**git merge 和 git rebase 的区别**

不同于 `git rebase`的是，`git merge` 在不是 `fast-forward`（快速合并）的情况下，会产生一条额外的合并记录，类似`Merge branch 'xxx' into 'xxx'`的一条提交信息。

![img](https://img-blog.csdnimg.cn/ab3d812ff1904b63a2fce621876dca28.png)

另外，在解决冲突的时候，用 merge 只需要解决一次冲突即可，简单粗暴，而用 rebase 的时候 ，需要一次又一次的解决冲突。

**git rebase 交互模式**

在开发中，常会遇到在一个分支上产生了很多的无效的提交，这种情况下使用 rebase 的交互式模式可以把已经发生的多次提交压缩成一次提交，得到了一个干净的提交历史，例如某个分支的提交历史情况如下：

![img](https://img-blog.csdnimg.cn/666ec45c1a2e4104a3e17a074b38cdca.png)

进入交互式模式的方式是执行：

```
git rebase -i <base-commit>
```

参数base-commit就是指明操作的基点提交对象，基于这个基点进行 rebase 的操作，对于上述提交历史的例子，我们要把最后的一个提交对象（ac18084）之前的提交压缩成一次提交，我们需要执行的命令格式是：

```
git rebase -i ac18084
```

此时会进入一个 vim 的交互式页面，编辑器列出的信息像下列这样。

<img src="https://img-blog.csdnimg.cn/b5e247493d87465a8de4e72040175d35.png" alt="img" style="zoom:25%;" />

想要合并这一堆更改，我们要使用 squash 策略进行合并，即把当前的 commit 和它的上一个 commit 内容进行合并， 大概可以表示为下面这样。

```
pick  ... ...
s     ... ... 
s     ... ... 
s     ... ... 
```

修改文件后 按下:然后wq保存退出，此时又会弹出一个编辑页面，这个页面是用来编辑提交的信息，修改为`feat:` 更正，最后保存一下，接着使用`git branch`查看提交的 commit 信息。

rebase 后的提交记录如下图所示，rebase 操作可以让我们的提交历史变得更加清晰。

![img](https://img-blog.csdnimg.cn/d46b6fc2acfb455c89aea9718ae00f67.png)