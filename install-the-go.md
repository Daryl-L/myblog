# 从零开始学习 Go —— 安装

## 0x01 设置 Go 环境

要安装并顺利使用 Go，第一步就是要设置 Go 的环境。

需要设置的 Go 的环境变量，一共有三个。

- `GOROOT` Go 语言的源码以及安装目录。
- `GOPATH` Go 语言的开发目录，目录可以有多个，但是，当我们执行 `go get` 命令的时候，如未指定目录，会默认保存在第一个目录下。
- `GOROOT_BOOTSTRAP` 这个目录在安装 Go 1.5 版本及之后的版本时需要设置。由于在 1.4 版本后，Go 编译器实现了自举，即通过 1.4 版本来编译安装之后版本的编译器。如果不设置该环境变量的话，会产生这样一个错误 `Set $GOROOT_BOOTSTRAP to a working Go tree >= Go 1.4.`。

除此之外，还需要配置 `PATH` 环境变量到 Go 的二进制程序目录。

我们需要在 `~/.bash_profile` 中添加下面的代码（我把所有的 Go 语言相关的东西都放在了 `~/.golang` 下面了）：

```shell
export GOROOT=$HOME/.golang/go
export GOPATH=$HOME/.golang/path
export PATH=$PATH:$HOME/.golang/go/bin
export GOROOT_BOOTSTRAP=$HOME/.golang/go1.4
```

## 0x02 安装 Go

我们有两种方式下载 Go，一个是直接[下载源码](https://storage.googleapis.com/golang/go1.8.3.src.tar.gz)，另一个是通过 GitHub 克隆项目，个人推荐选择第二种，地址：[GayHub](https://github.com/golang/go)。

首先将项目克隆到本地。

```shell
$ git clone https://github.com/golang/go.git ~/.golang/go
```

然后再复制一份作为 1.4 版本的目录。

```shell
$ cp -r go go1.4
```

进入 1.4 的文件夹后，将切换分支开始安装。

```shell
$ git checkout -b release-branch.go1.4 origin/release-branch.go1.4
$ cd go1.4/src
$ ./make.bash
```

编译安装好之后，进入之前的 go 文件夹，真正开始编译安装 Go。

```shell
$ cd go/src
$ ./make.bash
```

最后，我们试试 `go version` 来查看版本，可能会发现很奇怪的东西。

```shell
$ go version
go version devel +d64c49098c Sun May 28 10:23:38 2017 +0000 darwin/amd64
```

这是我们编译了 HEAD 的版本，也就是最新提交的版本，这个版本并不稳定。我们可以将分之切换到稳定版本来进行安装。截止到这篇文章，Go 的最新稳定版本时 1.8.3。所以我们要讲分支切换到 `release-branch.go1.8`。

## 0x03 完整命令

```shell
$ echo "export GOROOT=$HOME/.golang/go" >> ~/.bash_profile
$ echo "export GOPATH=$HOME/.golang/path" >> ~/.bash_profile
$ echo "export PATH=$PATH:$HOME/.golang/go/bin" >> ~/.bash_profile
$ echo "export GOROOT_BOOTSTRAP=$HOME/.golang/go1.4" >> ~/.bash_profile
$ source ~/.bash_profile
$ cd ~
$ mkdir .golang
$ git clone https://github.com/golang/go.git go
$ cp -r go go1.4
$ cd go1.4
$ git checkout -b release-branch.go1.4 origin/release-branch.go1.4
$ cd src
$ ./make.bash
$ cd ../../go
$ git checkout -b release-branch.go1.8 origin/release-branch.go1.8
$ cd src
$ ./make.bash
$ go version
```


