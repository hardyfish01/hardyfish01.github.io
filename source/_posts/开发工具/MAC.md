---
title: MAC
categories: 
- 开发工具
---

MAC精品应用下载：

* https://xclient.info/

* https://www.macwk.com/

活动监视器：看到后台运行程序

# 快捷键

 移到废纸篓：

* Command+Backspace

清空废纸篓：

* Command+Shift+Backspace

显示隐藏文件：

* Command+Shift+.

前往文件目录：

* Command+Shift+G

新建页面：

* Command+N

终端操作：

* Command+T：打开多个窗口
* Command+W：关闭当前窗口
* Command+M：当前窗口最小化
* Command+加号键：终端放大字体

打开Spotlight：

* Command+空格

应用程序的切换：

* Command+Tab

关闭应用程序：

* Command+Q

上/下一级文件切换：

* Command+上下键

# HomeBrew

国内安装可用：

```
/bin/zsh -c "$(curl -fsSL https://gitee.com/cunkai/HomebrewCN/raw/master/Homebrew.sh)"
```

# JDK安装

> 1.到Oracle官网下载JDK1.8安装包：

https://www.oracle.com/java/technologies/downloads/#java8

> 2.配置系统的环境变量：

JDK的真实主目录如下：

```
/Library/Java/JavaVirtualMachines/jdk1.8.0_211.jdk/Contents/Home
```

打开`.bash_profile`并编辑：

在文件的末尾加入这几行语句：

```
open ~/.bash_profile

export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk1.8.0_281.jdk/Contents/Home
export PATH=$JAVA_HOME/bin:$PATH:.
export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:.

source ~/.bash_profile
```

> 3.验证JDK1.8是否安装成功

```
# 确认配置生效
java -version
```

**多JDK版本切换**

Oracle JDK12： https://www.oracle.com/technetwork/java/javase/downloads/jdk12-downloads-5295953.html

系统中默认安装位置：`/Library/Java/JavaVirtualMachines/`

```
sudo tar -zxf  openjdk-12_osx-x64_bin.tar.gz -C /Library/Java/JavaVirtualMachines/
```

验证是否安装成功：

```
 java -version
```

查看所有JDK的在系统中默认的安装位置：

```
/usr/libexec/java_home  -V
```

查看指定版本JDK在系统中默认安装位置：

```
/usr/libexec/java_home -v 9
```

手动切换JDK版本

通过修改` ~/.bash_profile`文件修改JAVA_HOME

```
export JAVA_8_HOME=$(/usr/libexec/java_home -v1.8)
export JAVA_9_HOME=$(/usr/libexec/java_home -v9)
export JAVA_10_HOME=$(/usr/libexec/java_home -v10)
export JAVA_11_HOME=$(/usr/libexec/java_home -v11)

alias java8='export JAVA_HOME=$JAVA_8_HOME'
alias java9='export JAVA_HOME=$JAVA_9_HOME'
alias java10='export JAVA_HOME=$JAVA_10_HOME'
alias java11='export JAVA_HOME=$JAVA_11_HOME'
#默认是Java 11
```

执行下面命令，让文件生效

```
source ~/.bash_profile
```

切换JDK版本就执行对应命令别名，比如，我要切换成JDK9，就执行一下java9就可以了。

# Maven安装

> 1.下载：

下载地址：http://maven.apache.org/download.cgi

修改`settting.xml`文件，在mirrors标签下添加子节点。

```xml
 <mirror>
 	<id>nexus-aliyun</id>
 	<mirrorOf>central</mirrorOf>
 	<name>Nexus aliyun</name>
	 <url>http://maven.aliyun.com/nexus/content/groups/public</url>
 </mirror>
```

> 2.配置系统的环境变量：

```
open ~/.bash_profile

export MAVEN_HOME=/Users/kun/Desktop/midongtools/apache-maven-3.5.0
export PATH=$PATH:$MAVEN_HOME/bin

source ~/.bash_profile
```

> 3.验证是否安装成功

```
mvn -v
```

# Iterm2

iTerm2下载地址：https://iterm2.com/downloads.html

> 主题配置：

iTerm2 最常用的主题是 Solarized Dark theme，下载地址：[http://ethanschoonover.com/solarized](https://links.jianshu.com/go?to=http%3A%2F%2Fethanschoonover.com%2Fsolarized)

打开 Preferences 配置界面，然后`Profiles -> Colors -> Color Presets`，在下拉列表中选择 Import，选择刚才解压的`solarized->iterm2-colors-solarized->Solarized Dark.itermcolors`文件，导入成功后，在 Color Presets下选择 Solarized Dark 主题。

# 安装Zsh

```
# 使用brew安装zsh
brew install zsh 
# 将zsh加入shells备选列表
echo /usr/local/bin/bash | sudo tee -a /etc/shells
# 切换到zsh（之后需要重新打开命令窗口）
chsh -s /usr/local/bin/zsh
```

**安装ohmyzsh**

```
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

配置zsh

```
vi ~/.zshrc
```

根据主题列表 https://github.com/ohmyzsh/ohmyzsh/wiki/themes ，选择你想要的主题

```
ZSH_THEME="af-magic

#改为
ZSH_THEME="ys"
```

