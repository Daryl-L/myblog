# 为什么Swoole可以加速php

## 前言

最近在研究`Swoole`，原来一直听别人在说`Swoole`可以加速，一直都是懵逼的。在研究了`Swoole`之后，我有了一些自己的理解。

## PHP-CGI 的黑历史

对于 `PHP` 处理网络请求，大家基本上也都是再用 `CGI` 的方式来做的。那么，什么是 `CGI` 呢。

### CGI

`CGI`，全称 `Common Gateway Interface`，中文称作公共网关接口。也许有很多人认为 `CGI` 是一个程序，没错，曾经的我也是这么认为的。直到我从《图解HTTP》开始细细地研究`HTTP`协议之后，我才知道，原来 `CGI` 是一种协议。任何编程语言，都可以实现 `CGI`，所以任何语言都可以作为网站的后台语言（扯远了）。

### PHP-CGI

上面说了，`CGI` 是一个协议，所以，`PHP` 有自己对 `CGI` 的实现，那就是 `PHP-CGI`。可是呢，随着技术的发展，人们开始意识到，`PHP-CGI` 的性能不是那么尽如人意。我们知道，`PHP` 在运行的时候，是依赖配置文件 `php.ini`的。所以，每当 `PHP-CGI` 开始工作的时候，它是完完全全的一个新进程，它需要重新加载配置文件并初始化，这就造成了很大的资源和时间的浪费。

### FastCGI

那么，怎么才能避免这种浪费呢，聪明的程序员们想出了另外一种方法：我们为什么不预先加载好配置，然后，每一个执行的任务只需要复制当前的进程，不就能避免上面的浪费了么。于是，` FastCGI` 便横空出世。

`FastCGI`，全称 `Fast Common Gateway Interface`，中文译作快速公共网管接口。没错，这又是个协议。当然，这个协议并不是因为 `PHP` 才有的。

### Apache (httpd)

几乎所有的 `Web` 容器都实现了 `FastCGI` 的功能。首先是 `httpd`。对于 `PHP` 来说，`httpd` 是通过自身来实现一个 `FastCGI` 的模块的。它会预先加载好 `php.ini` 文件中的配置。待到有请求进入需要 `PHP` 处理时，`PHP` 就不需要再对 `php.ini` 重新加载了。这也就是每改动过 `php.ini` 后都要重启 `httpd` 服务的原因。

### Nginx 与 php-fpm

`php-fpm` 也是 `FastCGI` 的一种实现。通常我们是将 `Nginx` 的 `PHP` 处理部分代理到 `php-fpm` 的端口上，交给 `php-fpm` 来处理。而 `php-fpm` 同样是通过预先加载配置，然后给到子进程的方式的，它会对进程做一些管理。

## Swoole

辣么问题来了，`php-fpm` 虽然实现了 `FastCGI`，但是，它在处理请求的时候，依然要重新运行一个脚本，像 `Laravel` 一样的框架，一开始就要加载辣么多依赖和文件，依然是一个不小的开销。我们看一下 `Laravel` 的 `public/index.php` 的源码。

```php
require __DIR__.'/../bootstrap/autoload.php';
$app = require_once __DIR__.'/../bootstrap/app.php';
$kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
$response = $kernel->handle(
    $request = Illuminate\Http\Request::capture()
);
$response->send();
$kernel->terminate($request, $response);
```

看看前面两条语句，这需要加载多少个依赖啊，这都是大把大把的时间和资源啊，每一次请求都需要加载一边，真是心疼啊。

那么，我们为什么不能像之前一样，能够不重新加载配置文件的 `FastCGI` ，来一个不用加载这么多的依赖的方式呢？

当然可以啦，这时候 `Swoole` 就派上用场了。既然是通过 `$app->make` 的方式来生成一个新的 `Kernel` 对象，那么 `Application` 的对象 `$app` 自然是不会有什么改变的了。所以，我们可以在收到请求之前，就把 `$app` 给生成好，这样就会快了，不是么？我们可以对它进行一个简单的改造。

```php
require __DIR__.'/../bootstrap/autoload.php';
$app = require_once __DIR__.'/../bootstrap/app.php';
$serv = new \Swoole\Server\Http('127.0.0.1', 9501);
$serv->on('request', function ($req, $res) use ($app) {
    $kernel = $app->make(Illuminate\Contracts\Http\Kernel::class);
    $response = $kernel->handle(
        $request = Illuminate\Http\Request::capture()
    );
    $res->end($response);
    $kernel->terminate($request, $response);
});
$serv->start();
```

好了，我们现在就可以通过执行这个脚本来监听9501端口了。然后就像 `Nginx` 配置 `php-fpm` 一样来配置它就可以了。这样我们可以看到，在收到请求之前，就已经把依赖加载干净了，剩下的就是处理请求了。

当然我的这个改动很简陋，根本无法用于生产环境的，只是提供一个例子。

## 后记

以上只是我自己的理解和对我自己的理解进行的总结。对于 `Swoole` 我还在探索当中，因为它需要的只是实在是太多了，需要一点一点积累。本文可能有不对的地方，欢迎各位大神来拍砖！
