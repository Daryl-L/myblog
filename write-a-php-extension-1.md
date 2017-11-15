# 从零开始写 PHP 扩展（一）

PHP 是用 C 语言写的。对于每个 PHPer 来说，都有着内心的一种希望写扩展的冲动了吧。然而，缺乏一个很好的切入点。Google 上搜 PHP 扩展开发，大部分都是复制品文章，甚至有些人连操作都没有操作过就搬运在了自己的博客。不过也有几篇好教程，但是都是 PHP 5 时代的产物，隐藏着非常多的坑。我会将我自己慢慢踩坑的过程记录下来，也许这就成了其它人的“教程”了吧。

## 生成一个扩展

想必很多人已经看到很多网上的教程了。大多都是教我们执行这个命令：`$ ./ext_skel --extname=extname`。但是，当你 clone 了 PHP 源码后会发现，master 分支下并没有 `ext/ext_skel` 这个文件。所以，我总结了一下：

如果你是直接下载 PHP 的源码，或者在已经 release 的版本分之下，你可以执行这个命令

```bash
$ cd ext
$ ./ext_skel --extname=extname
```

如果你是直接在 master 分支下，只有 `ext_skel.php` 文件，这个时候你就直接可以执行这个 PHP 文件

```bash
$ cd ext
$ php ext_skel.php --ext extname
```

由于我是直接在 master 分支下开发的，所以后面的都是默认在 master 分之下的操作。

生成了扩展之后，我们会看到四个文件和一个文件夹。现在这个阶段，我们只需要用到两个文件，`.c` 文件和 `.h` 文件。

## 一个小坑

在我们生成好扩展之后，我们可以试着编译一下

```bash
$ phpize
$ ./configure
$ make && make test
```

我们会惊讶地发现，编译的时候会有一个 warning。

```
warning: implicit declaration of function
      'ZEND_PARSE_PARAMETERS_NONE' is invalid in C99 [-Wimplicit-function-declaration]
        ZEND_PARSE_PARAMETERS_NONE();
        ^
1 warning generated.
```

然后你再执行 `make test` 发现有一个测试没有通过。没错，脚本为我们生成好的文件，居然通不过自己的测试。有没有觉得很诡异。我们看看 warning 的具体信息。找不到函数 `ZEND_PARSE_PARAMETERS_NONE`。看了一下文件，发现在第 15 行。看看这个函数名大概也能猜出来是什么意思了。于是我去 PHP 源码里搜了一下。可是我们发现了这样一个宏定义。

```c
#ifndef zend_parse_parameters_none
#define zend_parse_parameters_none()	\
        zend_parse_parameters(ZEND_NUM_ARGS(), "")
#endif
```

替换掉原来的大写之后，就没有 warning 了。这也算是官方给我们挖了一个小坑吧。虽然大写的有宏定义，但是为什么会报错，我也不太清楚了。

## 定义一个函数

我想，大多数人写扩展，肯定至少希望实现一个函数，不会是要几个全局变量就去写个扩展的吧（雾

这里 PHP 给我们提供了一个有用的宏 PHP_FUNCTION。生成好的代码里也有定义好的两个函数，可以参照它的用法。这个宏最终会被翻译成一个函数。例如 `PHP_FUNCTION(name)` 最终会被翻译成 `void zif_name(zend_execute_data *execute_data, zval *return_value)`

同时我们看到有定义了这么一个数组

```c
const zend_function_entry cesium_functions[] = {
	PHP_FE(cesium_test1,		arginfo_cesium_test1)
	PHP_FE(cesium_test2,		arginfo_cesium_test2)
	PHP_FE_END
};
```

我们需要将新添加的函数添加到这个数组里。像这样

```c
const zend_function_entry cesium_functions[] = {
	PHP_FE(cesium_test1,		arginfo_cesium_test1)
	PHP_FE(cesium_test2,		arginfo_cesium_test2)
	PHP_FE(name,           NULL)
	PHP_FE_END
};
```

记住，结尾不要加分号或者逗号。最后，我们可以个这个函数一个输出

```c
PHP_FUNCTION(name)
{
    php_printf("Hello\n");
}
```

编译安装完了之后我们就可以使用这个函数了

## 总结

本文仅仅是展示了从创建扩展开始到运行的全过程，本着能运行的心态来走完这些流程。

> 由于作者水平有限，如有错误，敬请不吝赐教。


