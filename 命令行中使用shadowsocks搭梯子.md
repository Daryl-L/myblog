作为一名开发者，每天都呆在天朝局域网中是不可能的。所以，我大天朝的程序员就多了一项基本支出——梯子。

最方便的梯子当属VPS+ss了。我买的Linode的VPS，还算快，虽然有时候抽风。每个月70，相对于能访问自由的互联网来说，这种程度的支出就不值得一提了。

ss在桌面客户端的配置很容易，但是将其应用于命令行中就需要做一些配置了。

##安装shadowsocks

首先当然是下载ss客户端了。下面是在Ubuntu上的实例，其他Linux发行版也基本差不多。

```sh
$ apt-get install python-pip

$ pip install shadowsocks
```

如果上面执行好了之后，试着输入这条命令`sslocal`，如果有，那就表示安装成功了。

安装好了之后就要配置shadowsocks，具体的配置根据你vps的配置来配了（这你一定要有个国外的VPS而且配了shadowsocks才可以啊！！！）。具体呢，有两种方法。

###第一种：

首先先新建文件，编辑下面的内容

```json
{
    "server":"my_server_ip", //你的配置好shadowsocks的vps
    "server_port":8388, //vps配置的shadowsocks的端口号
    "local_port":1080, //本地代理socks5的端口
    "password":"barfoo!", //shadowsocks服务器的密码
    "timeout":600,
    "method":"table" //shadowsocks服务器的加密方式
}
```

然后运行使用`sslocal`来进行配置

```sh
$ sslocal -c 你的文件
```

###第二种：

不需要新建配置文件，使用下面的命令

```sh
$ sslocal -s 你的ip -p 你的端口号 -k 你的密码
```

这样就直接占用了一个shell窗口，可以用nohup来后台运行

```sh
$ nohup sslocal -c 你的文件 > ss.log 2>&1 &
```

这样标准输出和错误输出就全都记录在`ss.log`文件中了。而且可以在后台运行。

###安装privoxy

只有shadowsocks是并没有什么软用的，要将socks5转换到http才可以。这就需要代理工具。这里推荐privoxy。运行下面的命令：

```sh
$ apt-get install privoxy
```

装完之后对配置文件进行配置。打开`/etc/privoxy/config`文件，找到`listen-address 127.0.0.1:8118`，如果是注释着的就接触注释，如果没有这一行就自己加上这一行。后面的端口号是http代理的端口号。如果已经冲突了，就换其它不冲突的。然后再加上这一行`forward-socks5t / 127.0.0.1:1080 .`。这个端口号是按照你的`sslocal`配置来的，注意，一定不要忘记后面那个`.`！！！！！！

在全部配置完之后，运行命令启动privoxy服务：

```sh
$ service privoxy start
```

运行之后看一下provixy有没有启动，运行

```sh
$ service privoxy status
```

如果回显的是`* privoxy is not running`的话，那就是运行失败了，如果你觉得你的配置都没有问题，那么你需要检查一下端口有没有冲突，换个端口试试。如果回显的是`* privoxy is running`就说明privoxy成功起来了。

###配置命令行

光有这些还是不行的，还要配置命令行的环境变量来实现代理。运行下面的命令：

```sh
$ export http_proxy=http://127.0.0.1:8118
$ export ftp_proxy=ftp://127.0.0.1:8118
```

运行这两个命令之后只是暂时性的，重启之后就没有了。这样就方便了取消代理，毕竟有些时候代理也会碍事的。取消代理只要这样就可以了。

```sh
$ export -n http_proxy
$ export -n ftp_proxy
```

当然如果想永久生效，就修改`bash_profile`或者`/etc/profile`这两个文件，在其中的一个文件中加上上面那两行命令就OK了。试一下`www.google.com`能不能访问。

```sh
$ curl www.google.com
```

如果可以访问，那么ok，接下去就可以享受自由的真·互联网了。

###注意

在配置的时候可能会碰上下面的错误：

* `Error: Wrong number of parameters for forward-socks4a directive in configuration file.`

	遇到这个错误的原因就是忘记之前的那个点了！！！记住`forward-socks5t / 127.0.0.1:1080 .`后面的那个点一定要有！！！！！


