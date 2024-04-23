# Hello world

## Install

Linux/Mac/Windows官网安装包[下载地址](https://golang.google.cn/doc/install)

## Start Go

这里通过创建hello world程序的例子初识Go；
参考：[官网教程](https://golang.google.cn/doc/tutorial/getting-started)

在 <https://github.com/liang-junming/golang> 仓库helloWorld目录中的示例中

1. 创建helloWorld目录

    创建helloWorld目录并进入目录

    ``` shell
    mkdir helloWorld
    cd helloWorld
    ```

2. 创建依赖跟踪文件`go.mod`

    当你的代码依赖其他`module`的`packages`时，通过`go.mod`文件管理依赖

    `go.mod`文件通过执行`go mod init`命令并给其自己代码的模块路径作为参数创建；
    通常这个模块路径是你保存代码的仓库路径，如：`github.com/mymodule`；
    如果你希望别人能使用你的模块则模块路径必须是`Go tools`可以下载的路径

    在这个例子里暂时使用`examplu.com/hello`

    ``` shell
    $ go mod init example.com/hello
    go: creating new go.mod: module example.com/hello
    ```

3. 创建并编辑`hello.go`文件

    ``` go
    package main

    import "fmt"

    func main() {
        fmt.Println("Hello, World!")
    }
    ```

    在上面代码中：
    - 声明一个主包（包是对功能进行分组的一种方式，它由同一目录中的所有文件组成）
    - import`fmt package`，其中包含格式文本以及打印到控制台的功能，
    这个包是安装Go时自带的标准库包之一
    - 实现`main`函数来向控制台打印一条消息，`main`会在执行这个`package`时默认执行

4. 执行代码

    ``` shell
    $ go run .
    Hello, World!
    ```

    `go run`命令是众多go命令中的一个，使用`go help`命令查看更多其它命令
