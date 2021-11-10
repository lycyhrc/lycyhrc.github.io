# 从一个Hello_World说起(Go)




> 写这篇博客的一个契机是，在Udemy上参加了名师xx的Go语言的课程，Section 2用26分钟讲解了如何如宏观理解一个简单的Go语言程序,这种讲解的方法思路清晰，简洁明了，仅以此记录一下。

程序如下:

```Go
package main

import "fmt"

func main() {
	fmt.Println("Hello there!")
}
```



## 1.运行.go文件时发生了什么?

在命令行运行go文件主要有以下指令 :

- **go build**: 编译go代码源文件
- **go run:** 编译和执行go代码源文件
- **go fmt:** 格式化当前目录每个文件的所有代码
- **go install:** 编译和 "installs" 一个包 
- **go get:** 下载来自于其他packge的源代码
- **go test:** 运行与当前项目相关的测试文件

运行方式：**go 指令 文件名.go**



## 2.main.go文件怎么组织?

| Code                                        | Explanation                |
| ------------------------------------------- | -------------------------- |
| packge main                                 | 定义包                     |
| import "fmt"                                | 导入其他需要的包           |
| func main() { fmt.Println("Hello there!") } | 定义函数，Go需要执行的动作 |

- packge main
  - package = Project = Workspace
  - 一个Package Main里面会有多个go文件，启动时以main.go为准
  - package main→go build→main.exe（如果运行该文件,main函数将会自动执行）
  - package xx→go build→nothing（如果运行该文件,main函数将不会自动执行，xx包会执行）

- package分为两类

  - Executable package
    - **package main**：定义一个可以编译和执行的包（必须要有main函数才能运行）
  - Reusable package：
    - package calculator：定义一个可以用来当做依赖的包

- import "fmt"

  ![](/02/package_and_fmt.jpg)

- func
  - 定义一个函数
  - 函数名
  - 参数列表
  - 函数体
    - fmt.Println()


