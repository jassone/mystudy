
> PHP7即使不启用Opcache速度也比PHP-5.6启用了Opcache快

### 那么什么是Opcache呢？

OPcache 通过将 PHP 脚本预编译的字节码存储到共享内存中来提升 PHP 的性能， 存储预编译字节码的好处就是 省去了每次加载和解析 PHP 脚本的开销。

 PHP的正常执行流程如下
 ![1128628-20180504142714761-711951956.png](https://pic.imgdb.cn/item/611a87e14907e2d39c400faf.png)

 request请求（nginx,apache,cli等）-->Zend引擎读取.php文件-->扫描其词典和表达式 -->解析文件-->创建要执行的计算机代码(称为Opcode)-->最后执行Opcode--> response 返回

每一次请求PHP脚本都会执行一遍以上步骤，如果PHP源代码没有变化，那么Opcode也不会变化，显然没有必要每次都重新生成Opcode，结合在Web中无所不在的缓存机制，我们可以把Opcode缓存下来，以后直接访问缓存的Opcode岂不是更快，启用Opcode缓存之后的流程图如下所示：
![1128628-20180504142702126-1584014725.png](https://pic.imgdb.cn/item/611a88134907e2d39c4124b6.png)

  Opcode cache 的目地是避免重复编译，减少 CPU 和内存开销。

### 查看数据

- phpinfo()中查看 Cache hits

### 相关函数
* 清空缓存

```php
opcache_reset();
```

* 预先为某个文件生成缓存

```php
opcache_compile_file('test2.php');
```

* 检查某个文件是否已经生成了缓存

```php
var_dump(opcache_is_script_cached('test2.php'));
// 打印true/false
```

* 只想清空某个文件的缓存

```php
var_dump(opcache_invalidate('test2.php', true));
// 该函数的作用是使得指定脚本的字节码缓存失效。 如果 第二个参数 force 没有设置或者传入的是 false，那么只有当脚本的修改时间 比对应字节码的时间更新，脚本的缓存才会失效。
```

* 获取opcache相关信息

```php
//获取缓存的状态信息
var_dump(opcache_get_status());

//获取缓存的配置信息
var_dump(opcache_get_configuration());
```
### 安装 opcache

- PHP7.0之后自带opcache.so无需安装
- 安装完后可以php -m 查看或者phpinfo() 查看

### 配置

php.ini加入

```sh
zend_extension="/usr/local/opt/php@7.4/lib/php/20190902/opcache.so";
```

关于opcache的详细参数配置比较核心的参数如下：

```
启用opcache
opcache.enable=1

CLI环境下，PHP启用OPcache
opcache.enable_cli=1

使用共享内存大小,单位MB
opcache.memory_consumption=200

字符串缓存大小。PHP使用了一种叫做字符串驻留（string interning）的技术来改善性能。例如，如果你在代码中使用了1000次字符串“foobar”，在PHP内部只会在第一使用这个字符串的时候分配一个不可变的内存区域来存储这个字符串，其他的999次使用都会直接指向这个内存区域。这个选项则会把这个特性提升一个层次——默认情况下这个不可变的内存区域只会存在于单个php-fpm的进程中，如果设置了这个选项，那么它将会在所有的php-fpm进程中共享。在比较大的应用中，这可以非常有效地节约内存，提高应用的性能。
这个选项的值是以兆字节（megabytes）作为单位，如果把它设置为16，则表示16MB，默认是4MB
opcache.interned_strings_buffer=8

最大缓存文件数量。这个选项用于控制内存中最多可以缓存多少个PHP文件。这个选项必须得设置得足够大，大于你的项目中的所有PHP文件的总和。
设置值取值范围最小值是 200，最大值在 PHP 5.5.6 之前是 100000，PHP 5.5.6 及之后是 1000000。也就是说在200到1000000之间。
opcache.max_accelerated_files=8000

出现异常，立即释放全部内存。从字面上理解就是“允许更快速关闭”。它的作用是在单个请求结束时提供一种更快速的机制来调用代码中的析构器，从而加快PHP的响应速度和PHP进程资源的回收速度，这样应用程序可以更快速地响应下一个请求。把它设置为1就可以使用这个机制了。
opcache.fast_shutdown=1

启用文件缓存时间戳。如果启用（设置为1），OPcache会在opcache.revalidate_freq设置的秒数去检测文件的时间戳（timestamp）检查脚本是否更新。
如果这个选项被禁用（设置为0），opcache.revalidate_freq会被忽略，PHP文件永远不会被检查。这意味着如果你修改了你的代码，然后你把它更新到服务器上，再在浏览器上请求更新的代码对应的功能，你会看不到更新的效果

**强烈建议你在生产环境中设置为0，更新代码后，再平滑重启PHP和web服务器。或者使用 opcache_reset() 或者 opcache_invalidate() 函数来手动重置 OPcache。**
opcache.validate_timestamps=0 

文件检测周期，检查脚本时间戳是否有更新的周期，以秒为单位。 设置为 0 会导致针对每个请求， OPcache 都会检查脚本更新。如果 opcache.validate_timestamps 配置指令设置为禁用，那么此设置项将会被忽略。
revalidate_freq=3600

开启Opcache File Cache(实验性), 通过开启这个, 我们可以让Opcache把opcode缓存缓存到外部文件中, 对于一些脚本, 会有很明显的性能提升.
这样PHP就会在/tmp目录下Cache一些Opcode的二进制导出文件, 可以跨PHP生命周期存在.
opcache.file_cache=/tmp

最大允许占用内存百分比，超过此限制会重启进程
opcache.max_wasted_percentage=20

如果启用，OPcache 将在哈希表的脚本键之后附加改脚本的工作目录， 以避免同名脚本冲突的问题。 禁用此选项可以提高性能，但是可能会导致应用崩溃。
比如,两个脚本名称一样..不使用此项,可能导致 两个脚本错乱!!!
opcache.use_cwd=1
```

##### 更多配置
https://www.php.net/manual/zh/opcache.configuration.php

### 更新缓存
1 设置Opcache脚本验证时间
opcache.revalidate_freq
opcache.validate_timestamps

在实际生产环境中,为了尽可能达到最优性能,尽量不开启文件更新验证,因为每次验证都会重新预编译PHP代码到共享内存中。

2 重启 | 重载 php-fpm 进程


3 手动清理缓存
opcache_reset()
opcache_invalidate() 

注意:当PHP以PHP-FPM的方式运行的时候，opcache的缓存是无法通过php命令进行清除的，只能通过http或cgi到php-fpm进程的方式来清除缓存,我们可以编写一个对外接口,来达到清理缓存的目的。

相关实现如下(框架:laravel):

```php
Route::any('cache-reset', function () {
  //重置整个Opcode缓存
  dd(opcache_reset());
});

Route::any('cache-update', function () {
  //清除掉最近一次更新文件的缓存
  exec('git diff --name-only HEAD~ HEAD', $output);
  foreach ($output as $file) {
    $path = base_path($file);
    opcache_invalidate($path, true);
  }
  dd('刷新完成');
});
```

##### 注意
1 如果代码发布是覆盖更新旧目录，则可以重启php-fpm及在脚本中或代码文件中使用opcache_reset函数来清理所有缓存。
2 如果代码发布是全量发布，切换软链接的方式，可以设置opcache.validate_timestamps=1和opcache.revalidate_freq=X来定时自动更新缓存。









