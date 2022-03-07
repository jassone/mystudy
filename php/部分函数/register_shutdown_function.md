## register_shutdown_function
注册一个会在php中止时执行的函数。

```php
function shutdown()
{
    // This is our shutdown function, in 
    // here we can do any last operations
    // before the script is complete.

    echo 'Script executed with success', PHP_EOL;
}

register_shutdown_function('shutdown');
```