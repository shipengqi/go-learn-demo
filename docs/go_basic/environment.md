# Go 环境配置
## 安装
我是在 Windows 下安装，官网[下载安装包](https://golang.org/dl/)，直接安装。
默认情况下`.msi`文件会安装在`c:\Go`目录下。安装完成后默认会将`c:\Go\bin`目录添加到`PATH`环境变量中。
并添加环境变量`GOROOT`，值为 Go 安装根目录`C:\Go\`。重启命令窗口生效。

打开CMD 输入`go`命令，验证是否安装成功。否则检查环境变量`Path`和`GOROOT`。

## 工作区
### GOROOT
上面我们知道环境变量`GOROOT`用来指定 Go 的安装目录，Go 的标准库也在这个位置。目录结构与`GOPATH`类似。

### GOPATH
我们安装好 Go 之后，**必须配置一个环境变量`GOPATH`**，这个`GOPATH`路径是用来指定当前工作目录的。**不能和 Go 的安装目录（`GOROOT`）一样**。

工作区的目录结构：
```bash
GOPATH/
    src/ # 源码目录
    bin/ # 存放编译后的可执行程序
    pkg/ # 存放编译后的包的目标文件
```

`GOPATH`允许多个目录，当有多个目录时，请注意分隔符，多个目录的时候Windows是分号`;`，Linux系统是冒号`:`，当有多个`GOPATH`时，默认会将`go get`的内容放在第一个目录下。

#### 代码目录结构
`GOPATH`下的`src`目录就是接下来开发程序的主要目录，所有的源码都是放在这个目录下面，一般就是一个目录一个项目。
所以新建应用或者一个代码包时都是在`src`目录下新建一个文件夹，**文件夹名称一般是代码包名称**，允许多级目录，例如：
`$GOPATH/src/github.com/shipengqi/gotest`，这个包路径就是`github.com/shipengqi/gotest`。包名称是最后一个目录`gotest`。

创建一个包：
在`$GOPATH/src/`下创建一个`mymath`目录，新建`sqrt.go`文件：
```go
package mymath

func Sqrt(x float64) float64 {
    z := 0.0
    for i := 0; i < 1000; i++ {
        z -= (z*z - x) / (2 * x)
    }
    return z
}
```
建议`package`的名称和目录名保持一致，可以直接`import mymath`导入。

#### 编译
`go install`命令和`go build`命令相似，不同的是`go install`会保存每个包的编译成果，可执行程序会保存到`$GOPATH/bin`目录，
应用包放到`$GOPATH/pkg/${GOOS}_${GOARCH}`，`${GOOS}_${GOARCH}`对应的是操作系统和位数，例如我的64位 windows对应的就是`$GOPATH/pkg/windows_amd64`。

新建应用包`mathapp`，创建目录`mathapp`，并在这个目录下创建文件`main.go`:
```bash
package main

import (
  "mymath"
  "fmt"
)

func main() {
  fmt.Printf("Hello, world.  Sqrt(2) = %v\n", mymath.Sqrt(2))
}
```
注意`src`目录下的包的绝对路径就是`$GOPATH/src/<package path>`，如`GOPTAH/src/mymath`，**导入包时不需要加`$GOPATH/src/`，如`import "mymath"`**，
`import`里面可以引入多级目录，如果你有多个`GOPATH`，Go会自动在多个`$GOPATH/src`中寻找。

目录`mathapp`下执行`go build`编译后，可以运行可执行程序`mathapp`，也可以使用`go install`编译，可以在任意目录下运行`mathapp`。

命令`go build`，`go install`。构建和安装代码包的时候都会执行编译、打包等操作，并且，这些操作生成的任何文件都会先被保存到某个临时的目录中。

安装操作会先执行构建，然后还会进行链接操作，并且把结果文件搬运到指定目录。进一步说，如果安装的是库源码文件，那么结果文件会被搬运到它所在工作区的 pkg 目录下的某个子目录中。

如果安装的是命令源码文件，那么结果文件会被搬运到它所在工作区的 bin 目录中，或者环境变量`GOBIN`指向的目录中。

## 命令
## 工具
Go 命令：
```bash
$ go
Go is a tool for managing Go source code.

Usage:

        go command [arguments]

The commands are:

        build       compile packages and dependencies
        clean       remove object files and cached files
        doc         show documentation for package or symbol
        env         print Go environment information
        bug         start a bug report
        fix         update packages to use new APIs
        fmt         gofmt (reformat) package sources
        generate    generate Go files by processing source
        get         download and install packages and dependencies
        install     compile and install packages and dependencies
        list        list packages
        run         compile and run Go program
        test        test packages
        tool        run specified go tool
        version     print Go version
        vet         report likely mistakes in packages

Use "go help [command]" for more information about a command.

Additional help topics:

        c           calling between Go and C
        buildmode   build modes
        cache       build and test caching
        filetype    file types
        gopath      GOPATH environment variable
        environment environment variables
        importpath  import path syntax
        packages    package lists
        testflag    testing flags
        testfunc    testing functions

Use "go help [topic]" for more information about that topic.
```

### 下载包
使用`go get`命令下载一个包。如`go get github.com/golang/lint/golint`下载了`golint`包，`src`目录下会有`github.com/golang/lint/golint`包目录。
`bin`目录下可以看到`golint`可执行程序。

`go get`本质上可以理解为首先第一步是通过源码工具`clone`代码到`src`下面，然后执行`go install`。

**OPTIONS**
- `-u` 保证每个包是最新版本。

### 构建包
主要用于编译代码，使用`go build`命令编译，命令行参数指定的每个包。
有两种情况：
- `main`包，`go build`将调用链接器在当前目录创建一个可执行程序，以导入路径的最后一段作为可执行程序的名字。
- 如果包是一个库，则忽略输出结果；这可以用于检测包是可以正确编译的。

被编译的包会被保存到`$GOPATH/pkg`目录下，目录路径和`src`目录路径对应，可执行程序被保存到`$GOPATH/bin`目录。

**OPTIONS**
- `-o` 指定输出的文件名，可以带上路径，例如`go build -o a/b/c`
- `-i` 安装相应的包，编译并且`go install`
- `-a` 更新全部已经是最新的包的，但是对标准包不适用
- `-n` 把需要执行的编译命令打印出来，但是不执行，这样就可以很容易的知道底层是如何运行的
- `-p n` 指定可以并行可运行的编译数目，默认是CPU数目
- `-race` 开启编译的时候自动检测数据竞争的情况，目前只支持64位的机器
- `-v` 打印出来我们正在编译的包名
- `-work` 打印出来编译时候的临时文件夹名称，并且如果已经存在的话就不要删除
- `-x` 打印出来执行的命令，其实就是和-n的结果类似，只是这个会执行
- `-ccflags 'arg list'` 传递参数给5c, 6c, 8c 调用
- `-compiler name` 指定相应的编译器，gccgo还是gc
- `-gccgoflags 'arg list'` 传递参数给gccgo编译连接调用
- `-gcflags 'arg list'` 传递参数给5g, 6g, 8g 调用
- `-installsuffix suffix` 为了和默认的安装包区别开来，采用这个前缀来重新安装那些依赖的包，-race的时候默认已经是-installsuffix race,大家可以通过-n命令来验证
- `-ldflags 'flag list'` 传递参数给5l, 6l, 8l 调用
- `-tags 'tag list'` 设置在编译的时候可以适配的那些tag，详细的tag限制参考里面的 Build Constraints

### 运行
`go run`命令实际上是结合了构建和运行的两个步骤。

### install
`go install`命令和`go build`命令相似，不同的是`go install`会保存每个包的编译成果，并把`main`包生产的可执行程序放到`bin`目录，
这样就可以在任意目录执行编译好的命令。

### clean
`go clean` 用来移除当前源码包和关联源码包里面编译生成的文件。文件包括：
```
_obj/            旧的object目录，由Makefiles遗留
_test/           旧的test目录，由Makefiles遗留
_testmain.go     旧的gotest文件，由Makefiles遗留
test.out         旧的test记录，由Makefiles遗留
build.out        旧的test记录，由Makefiles遗留
*.[568ao]        object文件，由Makefiles遗留

DIR(.exe)        由go build产生
DIR.test(.exe)   由go test -c产生
MAINFILE(.exe)   由go build MAINFILE.go产生
*.so             由 SWIG 产生
```

一般都是利用这个命令清除编译文件。

**OPTIONS**
- `-i` 清除关联的安装的包和可运行文件，也就是通过`go install`安装的文件
- `-n` 把需要执行的清除命令打印出来，但是不执行，这样就可以很容易的知道底层是如何运行的
- `-r` 循环的清除在`import`中引入的包
- `-x` 打印出来执行的详细命令，其实就是`-n`打印的执行版本

### go fmt

`go fmt`命令 它可以帮你格式化你写好的代码文件，使你写代码的时候不需要关心格式，你只需要在写完之后执行`go fmt <文件名>.go`，
你的代码就被修改成了标准格式。

**OPTIONS**
- `-l` 显示那些需要格式化的文件
- `-w` 把改写后的内容直接写入到文件中，而不是作为结果打印到标准输出。
- `-r` 添加形如“a[b:len(a)] -> a[b:]”的重写规则，方便我们做批量替换
- `-s` 简化文件中的代码
- `-d` 显示格式化前后的diff而不是写入文件，默认是`false`
- `-e` 打印所有的语法错误到标准输出。如果不使用此标记，则只会打印不同行的前10个错误。
- `-cpuprofile` 支持调试模式，写入相应的cpufile到指定的文件

### 包文档
#### 注释
在代码中添加注释，用于生成文档。Go 中的文档注释一般是完整的句子，**第一行通常是摘要说明，以被注释者的名字开头。**
注释中函数的参数或其它的标识符并不需要额外的引号或其它标记注明。例如`fmt.Fprintf`的文档注释：
```go
// Fprintf formats according to a format specifier and writes to w.
// It returns the number of bytes written and any write error encountered.
func Fprintf(w io.Writer, format string, a ...interface{}) (int, error)
```

如果注释后仅跟着包声明语句，那注释对应整个包的文档。包文档注释只能有一个。可以在任意的源文件中。

但是如果包的注释较长，一般会放到一个叫做`doc.go`的源文件中。

#### go doc 命令
`go doc`打印文档。
```bash
# 指定包
go doc time

# 指定包成员
go doc time.Since

# 一个方法
go doc time.Duration.Seconds
```
#### godoc服务
`godoc`服务提供可以相互交叉引用的 HTML 页面，godoc的[在线服务](https://godoc.org)。包含了成千上万的开源包的检索工具。

也可以在启动本地的`godoc`服务：
```bash
# 在工作区目录下运行
godoc -http :8080
```

然后访问`http://localhost:8000/pkg`。

### 内部包
Go 的构建工具对包含`internal`名字的路径段的包导入路径做了特殊处理。这种包叫`internal`包。如`net/http/internal/chunked`。
一个`internal`包只能被和`internal`目录有同一个父目录的包所导入。如：`net/http/internal/chunked`只能被`net/http`包或者`net/http`下的包导入。

什么时候使用`internal`包？
当我们并不想将内部的子包结构暴露出去。同时，我们可能还希望在内部子包之间共享一些通用的处理包时。

### 查询包
使用`go list`命令查询可用包的信息。如`go list github.com/go-sql-driver/mysql`

```bash
# 列出工作区中的所有包
go list ...

# 列出指定目录下的所有包
go list gopl.io/ch3/...

# 某个主题相关的所有包
go list ...xml...

# 获取包完整的元信息 -json 参数表示用JSON格式打印每个包的元信息
go list -json hash
```
### 查看 Go 相关环境变量
使用`go env`命令查看 Go 所有相关的环境变量。

### 版本
`go version`查看go当前的版本

## 开发工具
常用IDE：
- LiteIDE
- Sublime
- IntelliJ IDEA
- VS Code
