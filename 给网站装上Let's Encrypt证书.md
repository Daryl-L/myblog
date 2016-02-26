在全民https的浪潮下，咱也不能落后呀。但是要想让浏览器的地址栏出现那一抹美丽的绿色，就必须使用认证机构的证书。但是有很多证书并不是免费的，所幸的是有一款开源的证书到来了。

![Let's Encrypt](https://www.daryl.red/wp-content/uploads/2016/02/lets-encrypt.png)

当当当当，这就是**Let's Encypt**。什么是Let's Encrypt呢，Let's Encrypt是由ISRG根据现在人们日益增长的https需求而开始的开源项目。什么什么的都挺好的，就是有效期只有3个月，每3个月就要重新进行一次认证。具体的呢可以前往Let's Encrypt的官方网站[https://letsencrypt.org](https://letsencrypt.org)一探究竟。

首先呢，将Let's Encrypt官方开源的程序克隆下来，然后看帮助

```sh
$ git clone https://github.com/letsencrypt/letsencrypt
$ cd letsencrypt
$ ./letsencrypt-auto --help
```

看完帮助呢，哈哈该会的差不多也都会了，看不懂的就往下看好了= =

```sh
letsencrypt certonly --webroot -w /var/www/example -d example.com -d www.example.com
```

这里的`-w`是web目录，`-d`是域名，这两个均可以指定多个，每个前面都需要加上`-w`或`-d`才可以。然后根据提示填写相应的信息就阔以了= =

在上面的步骤都完成之后，会在目录`/etc/letsencrypt/live/example.com/`中生成密钥文件，一共是四个，`cert.pem`、`chain.pem`、`fullchain.pem`和`privkey.pem`。

接下来在apache中配置ssl密钥。
首先是在`httpd.conf`文件中开启ssl的支持。找到下面的一行，然后取消注释：

```conf
Include conf/extra/httpd-ssl.conf
```

接下来找到`extra/httpd-ssl.conf`进行编辑。添加下面的内容：

```conf
<VirtualHost *:443>
     ServerAdmin webmaster@example.com
     DocumentRoot "/var/www/example.com"
     <Directory "/var/www/example.com">
         Options FollowSymLinks
         AllowOverride None
         Order allow,deny
         Allow from all
         Require all granted
         DirectoryIndex index.php server.php
     </Directory>
     ServerName www.example.com
     ServerAlias example.com
     ErrorLog "/var/log/httpd/www.example.com-error_log"
     TransferLog "/var/log/httpd/www.example.com-access_log"
     SSLEngine on

     SSLCertificateFile "/etc/letsencrypt/live/example.com/cert.pem"
     SSLCertificateKeyFile "/etc/letsencrypt/live/example.com/privkey.pem"
     SSLCertificateChainFile "/etc/letsencrypt/live/example.com/chain.pem"
</VirtualHost>
```

基本的配置和配置apache的虚拟主机差不多，只是下面加上了整数的有关信息。

这时候会发现当在浏览器中键入链接时，还是http。这里就在httpd-vhost里配置一下跳转就OK了。在对应的虚拟主机中加入下面的几行就阔以了。

```conf
RewriteEngine on
RewriteCond %{SERVER_PORT} !^443$
RewriteRule ^(.*)?$ https://%{SERVER_NAME}/$1 [L,R]
```


