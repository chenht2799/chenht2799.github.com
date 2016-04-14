# php 5.2 下使用 curl 扩展请求 https 报502错误

最近用php的curl扩展去请求对方的 https 接口时会报错 Segmentation fault (core dumped)

php版本是 5.2


示例代码如下

```php
<?php

$ch = curl_init();
// 设置curl允许执行的最长秒数
curl_setopt($ch, CURLOPT_TIMEOUT, 15);
curl_setopt($ch,CURLOPT_SSL_VERIFYPEER,false);
curl_setopt($ch,CURLOPT_SSL_VERIFYHOST,false);
// 获取的信息以文件流的形式返回，而不是直接输出。
curl_setopt($ch, CURLOPT_RETURNTRANSFER,1);
curl_setopt($ch, CURLOPT_URL, 'https://www.baidu.com');

$res = curl_exec($ch);
var_dump($res);

```

对于PHP开发来说，很少会碰到 Segmentation fault (core dumped) 这样的错误，调试也不好调试，服务器上对于这个502的错误，

php error log 什么都没有，php-fpm logs 也没看出什么明显的东西，于是乎，用 gbd 工具来查bug

```shell
(gdb) run /www/test_curl.php 
```


(gdb) bt

4504276

貌似问题出在了 NSS 的库编译上，于是又google了一番 ，找到了这个答案 https://bugs.centos.org/view.php?id=7399

7378273

PHP 的作者给出答案，NSS的版本问题，因此，对服务器上的 curl 的 NSS 做降级处理，把版本弄成 3.16.2.3

1848770

继续执行上面的脚本，成功，bingo ~~~

总结：

遇到 php 5.2 下使用 curl 扩展请求 https 报502错误 这么坑爹的问题，解决问题如下：

+ ** 像我上文所描述的解决办法处理； **

+ ** 直接把 php 版本升级了，比较暴力； **
