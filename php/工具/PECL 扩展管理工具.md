## PECL 扩展管理工具
PECL 的全称是 The PHP Extension Community Library ，是一个开放的并通过 PEAR(PHP Extension and Application Repository，PHP 扩展和应用仓库)打包格式来打包安装的 PHP扩展库仓库。通过 PEAR 的 Package Manager 的安装管理方式，可以对 PECL 模块进行下载和安装。

```sh
curl -o go-pear.php http://pear.php.net/go-pear.phar
chmod +x go-pear.php
/usr/local/php-7.1.13/bin/php go-pear.php
```

如安装swoole：
```sh
pecl install swoole
```

升级扩展
```sh
pecl upgrade swoole
```