---
title: 第一个Go程序
categories: 
- 编程语言
- Go基础
---

创建目录one，在goland编辑器中打开。

在目录one下输入命令：`go mod init one` 用于创建和初始化项目one的 `.mod` 文件。

在目录下创建一个`main.go`文件，这是Go程序的入口，编写如下代码：

```go
package main

import "fmt"

func main() {
  fmt.Println("Hello World!")
}
```

在目录one下运行`go build` 命令，会生成一个可执行文件，运行可执行文件，就看到终端输出了`Hello World!`。

如果在目录one下运行`go run main.go` 命令，也会在终端输出`Hello World!`，只是这样不会生成可执行文件