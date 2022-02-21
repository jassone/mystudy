## xhprof


### 1、安装
```
wget http://pecl.php.net/get/xhprof-0.9.2.tgz

tar zxf xhprof-0.9.2.tgz

cd xhprof-0.9.2/extension/

sudo phpize
./configure --with-php-config=/usr/local/php/bin/php-config
sudo make
sudo make install
```

### 2、代码示例
```php
//xhprof
if(isset($_REQUEST['xhprof']) && $_REQUEST['xhprof'] == 'on'){
    xhprof_enable(XHPROF_FLAGS_NO_BUILTINS + XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);
    function xhprof_test(){
        $data = xhprof_disable(); //返回运行数
// xhprof_lib在下载的包里存在这个目录,记得将目录包含到运行的php代码中
        include_once "/usr/local/webdata/xhprof/xhprof_lib/utils/xhprof_lib.php";
        include_once "/usr/local/webdata/xhprof/xhprof_lib/utils/xhprof_runs.php";
        $objXhprofRun = new XHProfRuns_Default();
// 第一个参数j是xhprof_disable()函数返回的运行信息
// 第二个参数是自定义的命名空间字符串(任意字符串),
// 返回运行ID,用这个ID查看相关的运行结果
        $run_id = $objXhprofRun->save_run($data, "xhprof");

        echo("<script> if(window.console) { console.log('http://xhprof.fanli.com/index.php?run=$run_id&source=xhprof'); }</script>");

    }
    register_shutdown_function('xhprof_test');
}
```
### 相关wiki
https://www.jb51.net/article/70997.htm