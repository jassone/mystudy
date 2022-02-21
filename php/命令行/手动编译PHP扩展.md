## 手动编译PHP扩展

以pdo为例
```ini
# 进入扩展目录 
cd pdo

# 生成.configure文件
phpize

# 编译三部曲
./configure --with-php-config = prefix/bin/php-config

make ; make install
```

说明：
* --with-php-config 选项指定php安装目录下的bin文件下的php-config
* prefix 即php的安装目录 。详细参考 php源码安装 , 链接在上方。

php.ini中的配置
```ini
# 开启扩展
extensions=pdo
```