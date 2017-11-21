# 源码安装 NSQ

因为业务需要，要用到 NSQ。所以学习了下 NSQ。首先是安装，我在自己电脑上，倾向于源码安装。一是源码安装可以安装最新的代码，二是整个安装过程可以自己掌控。

但是，安装过程中遇到了一些坑。主要还是我对 Go 以及一些衍生工具用的不是特别熟悉，并且在网上搜索到的文章，都是抄来抄去的很多并不能解决我的问题。所以我把整个安装过程记录下来，给自己一个备忘，给别人一个方便。

## 安装 Go

NSQ 是用 Go 写的，所以安装 NSQ 之前，要先安装 Go。

我这里给出安装具体过程的命令。具体可以参考我写的另外一篇文章 [从零开始学习 Go ——安装](https://segmentfault.com/a/1190000009594143)。

```shell
echo "export GOROOT=$HOME/.golang/go" >> ~/.bash_profile
echo "export GOPATH=$HOME/.golang/path" >> ~/.bash_profile
echo "export PATH=$PATH:$HOME/.golang/go/bin" >> ~/.bash_profile
echo "export GOROOT_BOOTSTRAP=$HOME/.golang/go1.4" >> ~/.bash_profile
source ~/.bash_profile
cd ~
mkdir .golang
git clone https://github.com/golang/go.git go
cp -r go go1.4
cd go1.4
git checkout -b release-branch.go1.4 origin/release-branch.go1.4
cd src
./make.bash
cd ../../go
git checkout -b release-branch.go1.8 origin/release-branch.go1.8 #可以根据需要选择合适的版本
cd src
./make.bash
go version
```

## 安装 dep

由于 NSQ 采用了 dep 作为包管理工具，所以我们还要安装这个工具。

```shell
go get -u github.com/golang/dep/cmd/dep
```

`go get` 是 Go 官方的包管理工具。执行完这个命令之后，我们会发现在 `$HOME/.golang/go/bin` 中能够找到可执行文件 `dep`。

## 安装 NSQ

准备工作都做完了，我们正式开始安装 NSQ 了。这里有官方文档，可以看一下。[http://nsq.io/deployment/installing.html](http://nsq.io/deployment/installing.html)。

```shell
git clone https://github.com/nsqio/nsq $GOPATH/src/github.com/nsqio/nsq
cd $GOPATH/src/github.com/nsqio/nsq
dep ensure
```

官网写到了这一步。然后，我发现，我到处都找不到 NSQ 的执行文件。然后我各种姿势上网搜，结果搜出来的都没什么软用。于是我仔仔细细地研究了下目录里面的文件，发现了一个 `Makefile` 文件。打开一看，发现，需要 make 一下才可以。所以我们需要多做下面这一步。

```shell
make
```

好了，到此 NSQ 就安装完了。 


