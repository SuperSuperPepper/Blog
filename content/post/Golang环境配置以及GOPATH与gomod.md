---
title: "Go 环境配置+为什么要设置GOPATH和GOMod"
date: 2020-07-07T15:09:46+08:00
draft: true
categories:
    - Golang
---
# 概述

本文主要涉及：Go 环境配置+为什么要设置GOPATH和GOMod

如果只对上述感兴趣，建议直接看正文。

## 起因

在看一个Go的框架时，发现同一个命令 ``` go install server```在公司没问题，在家里就只能在项目目录下跑，在GOPATH目录下，总报一个```can't load package: package hello: cannot find package```错误。虽然已经可以在项目下install出来，但是这丁点差别实在是太让人难受。

## 排查

搜了无数的博客，把各种环境变量GOPATH，GOROOT，GOBIN 设置了一遍，最后还是在两篇官方文档里面找到了答案——自己混用了GOPATH和Go Mod。公司用的是GOPATH，家里用的Go mod

[GOPATH设置教程](https://golang.org/doc/gopath_code.html)  
[Go Mod设置教程，需要go版本在1.13以后](https://golang.org/doc/gopath_code.html)  （推荐）后文讲为什么

# 正文

有时间还是强烈建议看一下官方教程。下文只是我个人的总结。不一定对。

## golang环境配置

[官网](https://golang.org/) 下载安装完事。

几个常用的环境变量

```
GOPATH: //go 的工作目录
GOROOT：//go 的安装目录
GOBIN：//go 的bin文件目录
GO111MODULE： //go mod 开关 on为开启go mod

GOPROXY=https://goproxy.cn,direct // https://goproxy.cn,direct 国内代理
```

因为国内墙的缘故，需要设置代理 ``` go env -w GOPROXY https://goproxy.cn,direct``` 这样在Vscode安装package时就不会失败。

根据个人情况使用GOPATH配置方式 或Go mod模式

### GOPATH

GOPATH是早期的设置方式，将工作目录设置GOPATH到全局环境变量。[设置教程](https://github.com/golang/go/wiki/SettingGOPATH) 目录结构如下图：

```
bin/
    hello                          # command executable
    outyet                         # command executable
src/
    github.com/golang/example/
        .git/                      # Git repository metadata
	hello/
	    hello.go               # command source
	outyet/
	    main.go                # command source
	    main_test.go           # test source
	stringutil/
	    reverse.go             # package source
	    reverse_test.go        # test source
    golang.org/x/image/
        .git/                      # Git repository metadata
	bmp/
	    reader.go              # package source
	    writer.go              # package source
    ... (many more repositories and packages omitted) ...
```

也就是说所有的项目都在src下面。不同的项目都在GOPATH/src/下。很显然这种设置方法是不太方便的，因为不同项目引用的package到放到了一起，这用git管理起来很麻烦，比如Ａ项目引用了a,b两个package，B项目引用了c,d两个package，那么如果我在A中修改了package的内容，我提交Ａ项目时想要带着package时就很麻烦。

其次是GOPATH需要设置全局环境变量，很多新手在对这些不熟悉的时候，很容易出错。项目名也必须不同。否则无法区分。

这也是为什么我在公司里时可以在GOPATH目录下进行```go install server``` 正如官方文档里面写的

​    Note that you can run this command from anywhere on your system. The `go` tool finds the source code by looking for the `github.com/user/hello` package inside the workspace specified by `GOPATH`.

为什么可以在anywhere呢，因为你的项目目录其实是被定死了，所以只要你有server这个项目go tool肯定可以找到。

### Go mod

go mod 正是为了解决上述问题（并不单单是上述问题，还有依赖引用问题）。在1.13以后开始推行。因为没多长时间，所以现在网络上的教程两种版本的都有，很容易混淆。

go mod可以完全替代GOPATH设置。只需要``` go env -w GO111MODULE=on```开启go mod

```
bin/
    hello                          # command executable
    outyet                         # command executable
src/
    myproject/server.go
```

进入到myproject 执行 ```go mod init myproject ```

就会创建出一个go.mod文件。在该目录下执行```go mod install myproject```即可生成bin文件。这样不同的项目就不要放到一起了，只需要一个gomod文件就可以管理包依赖。项目管理也会更加方便。

这也引出了为什么我在GOPATH目录下无法install，因为我在家里使用的是gomod模式。项目的路径并不是唯一的，所以他也不能在anywhere来执行```go mod install myproject``` 正如他教程原文写的：

Commands like `go install` apply within the context of the module containing the current working directory. If the working directory is not within the `example.com/user/hello` module, `go install` may fail.

这时你会发现如果你需要经常测试的话，需要经常在Bin和project目录来切换。官方为了解决这个问题。官方的建议是把Bin目录加到环境变量中，这样便可以在一个目录下，install && run。原文：

For convenience, `go` commands accept paths relative to the working directory, and default to the package in the current working directory if no other path is given. So in our working directory, the following commands are all equivalent:

```
$ go install example.com/user/hello
$ go install .
$ go install
```

Next, let's run the program to ensure it works. For added convenience, we'll add the install directory to our `PATH` to make running binaries easy:

```
# Windows users should consult https://github.com/golang/go/wiki/SettingGOPATH
# for setting %PATH%.
$ export PATH=$PATH:$(dirname $(go list -f '{{.Target}}' .))
$ hello
Hello, world.
$
```



## 吐槽

先吐槽自己，自己真的是懒，命名官方文档写的那么清楚，按照官方的教程操作一遍，也就半个小时，就能清楚理解所有。结果自己就懒的把这块研究清楚，就想图一个“快”。

为了图“快”第一次就用的goland，环境这块goland全帮你做了。以至于压根没搞清这些概念也没耽误自己把第一个server撸出来。要不是因为goland的破解过段时间就过期，每次在网上找破解烦的要死，我也不会想着在vscode上重新搭建一下环境。

为了图“快”草草的在国内搜索“Golang VSCode环境配置”，配置完后只是知其然不知其所以然。压根没弄明白go mod，gopath 的区别。这也导致了上文的错误。

欲速则不达，古人诚不我欺。

再吐槽国内的一些博客主，博客要么就不写，要么就稍微认真一点，不说写的多么好，对别人有多大帮助，起码要认真把。发现很多博客就起个名字，内容就一行“转载于XXXXXX”。我明白他本人可能就是想存个出签方便自己下次来找，但这无疑是给网络中丢垃圾，每次进到这种博客，内心都是十万个草泥马在奔腾 。如果只是给自己看，那网易云笔记他不香嘛。既然是分享就是让别人看的啊。又不能靠这点流量和点击换钱。网络环保也是环保啊。

---

辣椒酱 2020.7.7