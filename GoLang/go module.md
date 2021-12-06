# Go Modules

[TOC]



## Go Modules的历史

### GOPATH管理

在使用 GOPATH 模式下，我们需要将应用代码存放在固定的`$GOPATH/src`目录下，并且如果执行`go get`来拉取外部依赖会自动下载并安装到`$GOPATH`目录下。



### Go Dep包管理

### 





### Versioned Go (vgo)

vgo于2018年2月发布，vgo目前只能运行在go1.10之上。



官方觉得dep 的一些细节似乎不适合 Go，因此官方采取了另外的方案来推进Go的版本管理，即 vgo（Go modules的前身），最终演变为我们现在所见到的 Go modules，也在 Go1.11 正式进入了 Go 的工具链。



## Go Modules简介

Go modules 是 Go 语言的依赖解决方案，发布于 Go1.11，成长于 Go1.12，丰富于 Go1.13，正式于 Go1.14 推荐在生产上使用。

 





 Go1.11 是在 2018 年 08 月 25 日



Go Moudle 1.11版本开始支持，为了替代GOPATH



开关环境变量：GO111MODULE

辅助环境变量：GOPROXY、GONOPROXY、GOSUMDB、GONOSUMDB、GOPRIVATE

主要文件：go.mod 和 go.sum

命令行：go mod



### Go Modules的环境变量

#### GO111MODULE

GO111MODULE 这个命名代表着Go语言在 1.11 版本添加的，针对 Module 的变量。

Go语言提供了 GO111MODULE 这个环境变量来作为 Go modules 的开关，其允许设置以下参数：

- auto：只要项目包含了 go.mod 文件的话启用 Go modules，目前在 Go1.11 至 Go1.14 中仍然是默认值。
- on：启用 Go modules，将会是未来版本中的默认值。
- off：禁用 Go modules。



#### GOPROXY

主要是用于设置 Go 模块代理（Go module proxy）。其作用是用于能够脱离传统的GIT方式拉取代码，直接通过镜像站点来快速拉取。



**GOPROXY 配置**

默认值：`https://proxy.golang.org,direct`

GOPROXY的值是一个以英文逗号 “,” 分割的 Go 模块代理列表，允许设置多个模块代理。



> direct

​		"direct" 为特殊指示符，用于指示 Go 回源到模块版本的源地址去抓取(比如 GitHub)代码。

​		当值列表中上一个 Go module proxy 返回 404 或 410 错误时，Go 自动尝试列表中的下一个，遇见 “direct” 时回源，遇见 EOF 时终止并抛出类似 “invalid version: unknown revision...” 的错误。



**国内代理**

|                                            |          |
| ------------------------------------------ | -------- |
| GOPROXY=https://goproxy.io                 | 全球代理 |
| GOPROXY=https://goproxy.cn                 | 七牛云   |
| GOPROXY=https://mirrors.aliyun.com/goproxy | 阿里云   |
| GOPROXY=https://goproxy.baidu.com          | 百度     |



**GOSUMDB**

Go 1.13 新推出了一个 GOSUMDB（默认值是 sum.golang.org，国内无法访问）



### go.mod文件

### go.sum文件

**格式**

```golang
<module> <version>/go.mod <hash>
```

或者

```golang
<module> <version> <hash>
<module> <version>/go.mod <hash>
```

**说明**

- 其中module是依赖的路径，version是依赖的版本号。hash是以`h1:`开头的字符串，表示生成checksum的算法是第一版的hash算法（sha256）。
- 对于只有 `go.mod` 的 checksum，那么可能是因为对应的依赖没有单独下载。比如用 vendor 管理起来的依赖，便只有 `go.mod` 的 checksum。

- 有些项目实际上并没有 `go.mod` 这个文件，所以 Go 文档里提到这个 `/go.mod` 的 checksum，对于没有 `go.mod` 的项目，Go 会尝试生成一个可能的 `go.mod`，并取它的 checksum。

**场景**

1. 项目是否打tag

   - 未打tag

     会生成一个版本号，格式如下：`v0.0.0-commit日期-commitID`

     比如 `github.com/beorn7/perks v0.0.0-20180321164747-3a771d992973/go.mod h1:Dwedo/Wpr24TaqPxmxbtue+5NUziq4I4S80YR8gNf3Q=`。

   - 已打tag

     也会生成类似的版本号：v当前版本(tag)+1-commit日期-commitID

     当前分支的tag为v1.3.3。则为 `github.com/DATA-DOG/go-sqlmock v1.3.4-0.20191205000432-012d92843b00 h1:Cnt/xQ9MO4BiAjZrVpl0BiqqtTJjXUkWhIqwuOCVtWo=`。

2. 项目是否使用 go module

   - 项目使用了 go module

     使用正常地用 tag 来作为版本号

     比如 `github.com/DATA-DOG/go-sqlmock v1.3.3 h1:CWUqKXe0s8A2z6qCgkP4Kru7wC11YoAnoupUKFDnH08=`。

   - 项目打了 tag，但是没有用到 go module

     为了跟用了 go module 的项目相区别，需要加个 `+incompatible` 的标志。

     比如 `github.com/google/martian v2.1.0+incompatible/go.mod h1:9I4somxYTbIHy5NJKHRl3wXiIaQGbYVAs8BPL6v8lEs=`

3. 项目用的 go module 版本是不是 v2+

   通过让依赖路径带版本号后缀来区分同一个项目里不同版本的依赖，类似于 `gopkg.in/xxx.v2` 的效果。

   - 使用了 v2+ go module 的项目

     项目路径会有个版本号的后缀。

     比如 `github.com/googleapis/gax-go/v2 v2.0.5 h1:sjZBwGj9Jlw33ImPtvFviGYvseOtDM7hkSKB7+Tv3SM=`

**功能**

1. 提供分布式环境下的包管理依赖内容校验

   Go 并没有一个中央仓库。发布者在 GitHub 上给自己的项目打上 0.1 的 tag 之后，依旧可以删掉这个 tag ，提交不同的内容后再重新打个 0.1 的 tag。

   所以只能在每个项目里存储自己依赖到的所有组件的 checksum，才能保证每个依赖不会被篡改。

2. 作为 transparent log 来加强安全性

   go.sum 还有一个很特别的地方，就是它不仅仅记录了当前依赖的checksum，还保留了历史上每次依赖的 checksum。这种做法效仿了 transparent log 的[概念](https://research.swtch.com/tlog)。

   go.sum 之所以要用 transparent log 的形式记录历史上的每个checksum，是为了便于 sum db 的工作。

**缺点**

1. 容易产生合并冲突

   由于许多项目都没有通过打tag的方式来管理发布，每个commit都相当于新发布一个版本，这导致拉取它们的代码时会偶尔往 go.sum 文件里插入一条新记录。

   

   多人合作开发时，同时依赖同一个库的不同commit，插入2条不同的记录。在合并代码的时候 go.sum可能会冲突。

2. 





#### Go mod 命令行

| 命令            | 作用                             |
| :-------------- | :------------------------------- |
| go mod init     | 生成 go.mod 文件                 |
| go mod download | 下载 go.mod 文件中指明的所有依赖 |
| go mod tidy     | 整理现有的依赖                   |
| go mod graph    | 查看现有的依赖结构               |
| go mod edit     | 编辑 go.mod 文件                 |
| go mod vendor   | 导出项目所有的依赖到vendor目录   |
| go mod verify   | 校验一个模块是否被篡改过         |
| go mod why      | 查看为什么需要依赖某模块         |



go  get github.com/Azure/go-autorest@bca49d5	#指定commit

go get -v github.com/Azure/go-autorest@v10.15.5





// 这里以github.com为例，当然其他的仓库也是可以的

go get github.com/xxx/xxx@commit-id



go mod edit -require github.com/kubernetes-client/go@9dac5e4c5400

go mod tidy



go mod edit -require github.com/Azure/go-autorest@v10.15.5

go mod edit -require github.com/Azure/go-autorest@v10.15.5 -go 1.15

go mod edit -exclude github.com/Azure/go-autorest/autorest/adal@v0.8.3 -go 1.15

go mod edit -exclude github.com/Azure/go-autorest/autorest/adal@v0.9.17 -go 1.15



### 初始化项目



## Go Modules的说明

### indirect[间接]是什么意思



### 如何使用非语义化版本

具体来说, vgo 理解特殊的伪版本 `v0.0.0-yyyymmddhhmmss-commit` 引用给定的提交标识符, 它通常是一个缩短的 Git 哈希, 并且必须具有与(UTC)时间戳匹配的提交时间. 这种形式是 `v0.0.0` 预发布的有效语义版本字符串. 例如, 节选自 Gopkg.toml 的一段:

```golang
require (
    "google.golang.org/appengine" v1.0.0
    "github.com/google/go-github" v0.0.0-20180116225909-922ceac0585d
)
```



### 如何查看依赖



### 如何使用可视化工具

1. 源码地址

   https://github.com/poloxue/modv

2. 安装绘图软件

   ```shell
   yum install graphviz
   ```

3. 安装工具

   ```shell
   go install github.com/poloxue/modv
   ```

4. 使用

   ```shell
   go mod graph | modv | dot -T svg -o modv.svg
   ```





### 参考

- [如何实现 Go Module 依赖关系的可视化](https://zhuanlan.zhihu.com/p/88097101)

- [Go Modules 终极入门](https://studygolang.com/articles/26836)

- [尝鲜vgo](https://studygolang.com/articles/12429)

- [谈谈go.sum](http://www.zyiz.net/tech/detail-97723.html)
- [go get 指定commit 版本](https://blog.csdn.net/weixin_44676081/article/details/118070746)

