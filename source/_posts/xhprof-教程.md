title: xhprof 教程
date: 2016-05-20 16:51:29
tags:
---

### 简介 ###
XHProf 是一个轻量级的分层性能测量分析器。 __在数据收集阶段，它跟踪调用次数与测量数据，展示程序动态调用的弧线图。 它在报告、后期处理阶段计算了独占的性能度量，例如运行经过的时间、CPU 计算时间和内存开销。__ 函数性能报告可以由调用者和被调用者终止。 在数据搜集阶段 XHProf 通过调用图的循环来检测递归函数，通过赋予唯一的深度名称来避免递归调用的循环。

XHProf 包含了一个基于 HTML 的简单用户界面(由 PHP 写成)。 基于浏览器的用户界面使得浏览、分享性能数据结果更加简单方便。 同时也支持查看调用图。

XHProf 的报告对理解代码执行结构常常很有帮助。 比如此分层报告可用于确定在哪个调用链里调用了某个函数。

XHProf 对两次运行进行比较（又名 "diff" 报告），或者多次运行数据的合计。 对比、合并报告，很像针对单次运行的“平式视图”性能报告，就像“分层式视图”的性能报告。

### 安装扩展 ###
* 下载合适的版本 [xhprof](https://pecl.php.net/package/xhprof)
* 通过 phpize 安装扩展
```shell
	cd ~/Downloads/xhprof-0.9.4/extension/
	phpize
	./configure
	make
	make install
```
_这个时候 /usr/local/Cellar/php55/5.5.30/lib/php/extensions/no-debug-non-zts-20121212/ 就会产生 xhprof.so 的扩展_

* 修改 php.ini 配置文件
```shell
	vim /usr/local/etc/php/5.5/php.ini
```
```
[xhprof]
extension=xhprof.so;
; directory used by default implementation of the iXHProfRuns
; interface (namely, the XHProfRuns_Default class) for storing
; XHProf runs.
;
;xhprof.output_dir=<directory_for_storing_xhprof_runs>
xhprof.output_dir=/tmp/xhprof
```

* 重启 php-fpm


### 快速部署脚本 ###
```shell
	#!/bin/bash
	# fexiang
	# Used to install xhprof
	# 2014/11/27

	cd /tmp
	wget http://pecl.php.net/get/xhprof-0.9.2.tgz
	tar xvf xhprof-0.9.2.tgz
	cd xhprof-0.9.2/extension/
	/usr/local/webserver/php/bin/phpize
	./configure --with-php-config=/usr/local/webserver/php/bin/php-config
	make && make install

	mkdir /tmp/xhprof
	chown www:www /tmp/xhprof -R

	cat >> /usr/local/webserver/php/etc/php.ini << EOF
	[xhprof]
	extension=xhprof.so;
	; directory used by default implementation of the iXHProfRuns
	; interface (namely, the XHProfRuns_Default class) for storing
	; XHProf runs.

	xhprof.output_dir=<directory_for_storing_xhprof_runs>
	xhprof.output_dir=/tmp/xhprof
	EOF

	/etc/init.d/php-fpm restart

	yum install graphviz
	yum install graphviz-gd

```


### 代码调优 ###
```php
	<?php
	// cpu:XHPROF_FLAGS_CPU 内存:XHPROF_FLAGS_MEMORY
	// 如果两个一起：XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY
	xhprof_enable(XHPROF_FLAGS_CPU + XHPROF_FLAGS_MEMORY);
	// 要测试的php代码
	 
	$data = xhprof_disable(); //返回运行数据
	 
	// xhprof_lib在下载的包里存在这个目录,记得将目录包含到运行的php代码中
	include_once "xhprof_lib/utils/xhprof_lib.php";
	include_once "xhprof_lib/utils/xhprof_runs.php";
	 
	$objXhprofRun = new XHProfRuns_Default();
	 
	// 第一个参数j是xhprof_disable()函数返回的运行信息
	// 第二个参数是自定义的命名空间字符串(任意字符串),
	// 返回运行ID,用这个ID查看相关的运行结果
	$run_id = $objXhprofRun->save_run($data, "bluefrog");
	var_dump($run_id);
```


### 查看结果 ###
访问 xxx/xhprof_html/index.php?run=$run_id&source=bluefrog 就可经看到你的php代码运行的相关情况  
下面是一些参数说明

- Function Name：方法名称。
- Calls：方法被调用的次数。
- Calls%：方法调用次数在同级方法总数调用次数中所占的百分比。
- Incl.Wall Time(microsec)：方法执行花费的时间，包括子方法的执行时间。（单位：微秒）
- IWall%：方法执行花费的时间百分比。
- Excl. Wall Time(microsec)：方法本身执行花费的时间，不包括子方法的执行时间。（单位：微秒）
- EWall%：方法本身执行花费的时间百分比。
- ** Incl. CPU(microsecs)：方法执行花费的CPU时间，包括子方法的执行时间。（单位：微秒）**
- ICpu%：方法执行花费的CPU时间百分比。
- Excl. CPU(microsec)：方法本身执行花费的CPU时间，不包括子方法的执行时间。（单位：微秒）
- ECPU%：方法本身执行花费的CPU时间百分比。
- ** Incl.MemUse(bytes)：方法执行占用的内存，包括子方法执行占用的内存。（单位：字节）**
- IMemUse%：方法执行占用的内存百分比。
- Excl.MemUse(bytes)：方法本身执行占用的内存，不包括子方法执行占用的内存。（单位：字节）
- EMemUse%：方法本身执行占用的内存百分比。
- Incl.PeakMemUse(bytes)：Incl.MemUse峰值。（单位：字节）
- IPeakMemUse%：Incl.MemUse峰值百分比。
- Excl.PeakMemUse(bytes)：Excl.MemUse峰值。单位：（字节）
- EPeakMemUse%：Excl.MemUse峰值百分比。


### 遇到的问题 ###
* 安装扩展，运行测试的时候的日志写入文件夹权限问题
* 使用图表方式展示结果报错 failed to execute cmd: " dot -Tpng". stderr: `sh: dot: command not found '
```shell
	#红帽系列
	yum install graphviz
	yum install graphviz-gd
	#Ununtu debian
	apt-get install graphviz
	#mac系统
	brew install graphviz
```
