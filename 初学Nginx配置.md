在日常的学习中，逐渐地想把环境从Apache转移到Nginx上。据说Nginx是个老牛逼的东西了，但是一直没有接触它。由于各种网上的资料和教程已经很旧了，于是我总结自己在配置lnmp时的若干经验一共日后参考。

# Nginx

### 准备工作

我的服务器系统是Ubuntu 14.04.1 LTS，其他的Linux发行版请参考各自的包管理器。

Nginx在上个月似乎进入了1.10这个版本，而且第一次在介绍中新加入了HTTP/2。同时为了使用Certificate Transparency，还需要使用nginx-ct这个模块，这个模块是用来配置SSL的。

在正式开始安装前，需要编译环境。在Ubuntu下，需要下面的环境。

```sh
apt-get install build-essential
apt-get install libtool
```

其他Linux发行版类似，参照包管理器。

在装好了编译环境之后，就可以开始Nginx的安装了。首先需要安装的是PCRE和zlib。PCRE是用来进行rewrite的，zlib是用来进行gzip压缩的。

##### 安装PCRE

因为等一下安装Nginx的时候依然需要PCRE的源码，所以源码在编译安装之后保留一下。PCRE的下载地址在[ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/](ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/)。

```sh
wget ftp://ftp.csx.cam.ac.uk/pub/software/programming/pcre/pcre-8.37.tar.gz
tar -zxvf pcre-8.37.tar.gz
cd pcre-8.37
./configure --prefix=/usr/local/pcre
make && make install
```

##### 安装zlib

同样的，由于等一下安装Nginx的时候需要zlib的源码，故编译安装之后也需要保留zlib的源码。zlib的下载地址在[http://zlib.net/zlib-1.2.8.tar.gz](http://zlib.net/zlib-1.2.8.tar.gz)。

```sh
wget http://zlib.net/zlib-1.2.8.tar.gz
tar -zxvf zlib-1.2.8.tar.gz
cd zlib-1.2.8
./configure --prefix=/usr/local/zlib
make && make install
ln -s /usr/local/nginx/nginx/usr/bin
```

##### OpenSSL以及nginx-ct

OpenSSL可以在这里下载[https://www.openssl.org/source/](https://www.openssl.org/source/)。这里不需要安装，只是之后哦安装Nginx的时候需要用到。

```sh
wget https://www.openssl.org/source/openssl-1.0.2h.tar.gz
tar -zxvf openssl-1.0.2h.tar.gz
```

nginx-ct可以在这里下载[https://github.com/grahamedgecombe/nginx-ct](https://github.com/grahamedgecombe/nginx-ct)。

```sh
wget -O nginx-ct.zip -c https://github.com/grahamedgecombe/nginx-ct/archive/master.zip
uzip nginx-ct.zip
```

##### 安装Nginx

Nginx的源码下载地址在这里[http://nginx.org/en/download.html](http://nginx.org/en/download.html)。安装的时候，需要用到上面的PCRE、zlib、OpenSSL的源码。注意，这里一定是要指定源码的路径，而不是安装路径！！

```sh
wget http://nginx.org/download/nginx-1.10.0.tar.gz
tar -zxvf nginx-1.10.0.tar.gz
cd ginx-1.10.0
./configure --sbin-path=/usr/local/nginx/nginx --conf-path=/usr/local/nginx/nginx.conf --pid-path=/usr/local/nginx/nginx.pid --with-http_ssl_module --with-pcre=~/pcre-8.37 --with-zlib=~/zlib-1.2.8 --with-openssl=~/openssl-1.0.2h --add-module=~/nginx-ct --with-http_v2_module --with-http_ssl_module
make && make install
```

- **不知道为什么，我在编译的时候带上nginx-ct会报错，试了好几次依旧没有解决= =找到原因了在更新。先暂时不编译nginx-ct了╮(╯_╰)╭**

- **还有OpenSSL，也没有成功编译。于是我用apt-get安装了libssl-devel，编译的事情找到原因再更新╮(╯_╰)╭**

# PHP

PHP是世界上最好的编程语言，这是当然的啦。PHP 7已经更新到PHP 7.0.6了，当然我们应该下载最新版来体验啦。可以在这里找到下载地址[http://php.net/get/php-7.0.6.tar.gz/from/a/mirror](http://php.net/get/php-7.0.6.tar.gz/from/a/mirror)。

```sh
wget wget http://jp2.php.net/distributions/php-7.0.6.tar.gz
tar -zxvf php-7.0.6.tar.gz
```

和Apache不同，在Nginx里，PHP是借助php-fpm来运行的，所以要安装php-fpm才可以。php-fpm是PHP FastCGI管理器，是只用于PHP的，可以在[http://php-fpm.org/download](http://php-fpm.org/download)下载得到。新版的PHP已经集成了php-fpm，不需要再单独安装了，只需要在./configure时加上参数--enable-fpm参数即可开启php-fpm。

```sh
apt-get install libxml2 libxml2-dev libjpeg-dev pkg-config libpng++-dev libfreetype6 libfreetype6-dev libmcrypt-dev
```

```sh
./configure --prefix=/usr/local/php --with-config-file-path=/usr/local/php/etc --enable-fpm --with-fpm-user=www --with-fpm-group=www --with-mysqli --with-pdo-mysql --with-iconv-dir --with-freetype-dir --with-jpeg-dir --with-png-dir --with-zlib --with-libxml-dir=/usr --enable-xml --disable-rpath --enable-bcmath --enable-shmop --enable-sysvsem --enable-inline-optimization --with-curl --enable-mbregex --enable-mbstring --with-mcrypt --enable-ftp --with-gd --enable-gd-native-ttf --with-openssl --with-mhash --enable-pcntl --enable-sockets --with-xmlrpc --enable-zip --enable-soap --without-pear --with-gettext --disable-fileinfo --enable-maintainer-zts
make && make install
ln -s /usr/local/php/bin/php /usr/bin
```

完成之后，运行`php -v`，来确定PHP是否安装成功。

在安装PHP的过程中可能会遇到下面的错误。附上参考的解决方案。

**configure: error: Cannot find OpenSSL's libraries**

```sh
apt-get install pkg-config
```

**configure: error: Please reinstall the libcurl distribution - easy.h should be in <curl-dir>/include/curl/**

```sh
apt-get install libcurl4-gnutls-dev
```

##### 配置php-fpm

上面安装好之后，需要对php-fpm进行配置。

```sh
cd /usr/local/php/etc
cp php-fpm.conf.default php-fpm.conf
cp php-fpm.d/www.conf.default php-fpm.d/www.conf
ln -s /usr/local/php/bin/php-fpm /usr/bin
```

然后修改php-fpm.conf文件，添加

```conf
user=www-data
gourp=www-data
```

如果www-data用户不存在，那么需要新建一个账户

```sh
groupadd www-data
useradd -g www-data www-data
```

全部修改完之后，执行php-fpm即可。

#####配置Nginx

修改nginx.conf，将这一段注释去掉。

```conf
# pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
#
#    location ~ \.php$ {
#        root           html;
#        fastcgi_pass   127.0.0.1:9000;
#        fastcgi_index  index.php;
#        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
#        include        fastcgi_params;
#    }
```

启动Nginx服务，即可。

