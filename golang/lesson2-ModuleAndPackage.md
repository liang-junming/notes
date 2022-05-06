# Module 和 Package

`module`和`package`是Go组织代码的方式；`module`是一起发布的相关`package`的集合；
简单的说，一个`module`包含一个或多个`package`，一个`package`包含一个或多个`function`；

## Module

`module`可以理解为一个可完整发布的单元，它可以是一个Go可执行程序，也可以是一个Go库；
一个`module`中包含一个或多个`package`；
每个`module`的源码目录的根目录必须有一个`go.mod`文件，这个文件用于管理代码的依赖，一般由`go mod init`命令创建

在 <https://github.com/liang-junming/golang> 仓库localModule目录中的示例中，创建一个名为greetings的`module`，其中包含两个`package`：morning和night

1. 创建目录和`go.mod`文件

    ``` shell
    mkdir greetings
    cd greetings
    go mod init example.com/greetings
    ```

2. 创建morning目录和morning`package`

    ``` shell
    mkdir morning
    cd morning
    vim morning.go
    ```

    编辑morning.go文件如下

    ``` go
    package morning

    import "fmt"

    func GetMorningGreeting(name string) string {
        message := fmt.Sprintf("Good morning, %v", name)
        return message
    }
    ```

    在Go中，以大写字母开头的命名将会被导出

3. 创建night目录和night`package`

    ``` shell
    cd ../
    mkdir night
    cd night
    vim night.go
    ```

    编辑night.go文件如下

    ``` go
    package night

    import "fmt"

    func GetNightGreeting(name string) string {
        message := fmt.Sprintf("Good night, %v", name)
        return message
    }
    ```

4. 在greetings目录同级目录创建`module`hello调用greetings中的代码

    ``` shell
    cd ../../
    mkdir hello
    go mod init example.com/hello
    vim hello.go
    ```

    编辑hello.go文件如下

    ``` go
    package main

    import (
        "fmt"

        "example.com/greetings/morning"
        "example.com/greetings/night"
    )

    func main() {
        message := morning.GetMorningGreeting("Allen")
        fmt.Println(message)

        message = night.GetNightGreeting("Allen")
        fmt.Println(message)
    }
    ```

    这里因为是导入的本地的`module`，`go tool`无法在网络上访问到<https://example.com/greetings>，因为压根不存在，
    通过执行以下命令解决

    ``` shell
    go mod edit -replace example.com/greetings=../greetings
    ```

    执行完之后，`go.mod`文件会自动增加下面一行

    ``` shell
    replace example.com/greetings => ../greetings
    ```

    执行`go mod tidy`命令同步依赖

    ``` shell
    go mod tidy
    ```

5. 运行hello.go

    ``` shell
    $ go run .
    Good morning, Allen
    Good night, Allen
    ```

## package

`package`在Go文件中的第一条语句中申明，如上文中的

``` go
package main
```

``` go
package morning
```

`package`通过`import`语句导入，如上文中的

``` go
import (
    "fmt"

    "example.com/greetings/morning"
    "example.com/greetings/night"
)
```

`package`的`import`路径是其所在`module`的路径加上其在模块内的子目录；
这里example.com/greetings是`module`greetings的路径，morning是`package`morning在greeting中的子目录位置；
标准库的`package`的导入不需要加`module`路径，如：fmt；
在一个Go文件中可以导入本`module`中的`package`也可以导入外部`module`中的`package`

`package`名称分为两种，一种是命名为`main`的`package`，这通常存在于需要编译为Go可执行程序的`module`中；
名为`main`的`package`通常包含一个命名为`main`的`function`作为程序的入口

一个`package`可以包含多个文件，做法是在同一目录下的不同文件中申明相同的包名

在 <https://github.com/liang-junming/golang> 仓库localModule目录中的示例中，继续修改代码查看关于`package`的特性

1. 给`package`morning增加一个文件

    在greetings/morning目录中增加morning2.go并按如下内容编辑

    ``` go
    package morning

    import "fmt"

    func GetMorningGreeting2(name string) string {
        message := fmt.Sprintf("Hi, Good morning, %v", name)
        return message
    }
    ```

    修改greetings/hello/hello.go文件，在`main`函数中增加

    ``` go
    message = morning.GetMorningGreeting2("Allen")
    fmt.Println(message)
    ```

    执行hello.go

    ``` shell
    $ go run .
    Good morning, Allen
    Good night, Allen
    Hi, Good morning, Allen
    ```

2. 导入本`module`中的`package`

    在greetings/hello目录中创建localPackage目录并在其目录中创建hi.go文件

    ``` shell
    mkdir localPackage
    cd localPackage
    vim hi.go
    ```

    编辑hi.go文件

    ``` go
    package hi

    import "fmt"

    func GetHi(name string) string {
        message := fmt.Sprintf("Hi, %v", name)
        return message
    }
    ```

    修改hello.go文件

    ``` go
    package main

    import (
        "fmt"

        "example.com/greetings/morning"
        "example.com/greetings/night"

        "example.com/hello/localPackage" //new
    )

    func main() {
        message := morning.GetMorningGreeting("Allen")
        fmt.Println(message)

        message = night.GetNightGreeting("Allen")
        fmt.Println(message)

        message = morning.GetMorningGreeting2("Allen")
        fmt.Println(message)

        message = hi.GetHi("Allen") //new
        fmt.Println(message) //new
    }
    ```

    执行hello.go

    ``` shell
    $ go run .
    Good morning, Allen
    Good night, Allen
    Hi, Good morning, Allen
    Hi, Allen
   ```

    这里新建了名为hi的`package`但是放在名为localPackage的目录中，说明目录名和`package`名可以不同；
    但是通常情况下建议写成相同的，因为前面有提到`package`的`import`路径是按照文件目录来索引的（如：localPackage），而调用时是使用`package`名作为前缀（如：hi.GetHi），这容易造成歧义

## Developing modules

在上面的例子中，我们创建了一个本地`module` `example.com/greetings`；但从生产的角度看这不是很好的做法，
假如另外有一个Go程序也需要导入这个`module`的`package`时，则需要在它的代码库中另外再存一份此`module`的代码，这显然不合理

Go会根据被导入`module`的路径从网络上下载代码到本地(~/go/pkg/mod)，所以一个`module`可以也应当对应一个单独的代码库

在 <https://github.com/liang-junming/golang> 仓库remoteModule目录中的示例中，创建一个名为say的`module`，在say的同级目录创建hello程序通过远程导入的方式导入say中的`package`
（注意：这里没有将say作为一个单独的仓库是为了把所以的示例代码放在同一个仓库中，但效果一样）

1. 创建`moudle`取名为say

    ``` shell
    mkdir say
    cd say
    go mod init github.com/liang-junming/golang/remoteModule/say
    # 注意：module名必须和路径一致，不然会发生错误
    vim say.go
    ```

    编辑say.go

    ``` go
    package say

    import "fmt"

    func Say(word string) string {
        message := fmt.Sprintf("say %v", word)
        return message
    }
    ```

    上传say目录代码

    ``` shell
    git push origin HEAD
    ```

2. 创建Go程序hello导入say

    ``` shell
    mkdir hello
    cd hello
    go mod init hello
    vim hello.go
    ```

    编辑hello.go

    ``` go
    package main

    import (
        "fmt"

        "github.com/liang-junming/golang/remoteModule/say"
    )

    func main() {
        message := say.Say("hello")
        fmt.Println(message)
    }
    ```

    同步依赖

    ``` shell
    go mod tidy
    ```

    执行hello.go

    ``` shell
    $ go run .
    say hello
    ```
