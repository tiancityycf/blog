<!--
author: chiefyang
head: http://pingodata.qiniudn.com/jockchou-avatar.jpg
date: 2015-07-31
title: LINUX PHP C++扩展开发
tags: c++ php扩展 php
category: php
status: publish
summary: LINUX PHP C++扩展开发
-->
LINUX PHP C++扩展开发

1.下载相同版本的php源码包

    wget http://cn2.php.net/distributions/php-5.6.9.tar.gz

    注意，有一些历史版本没有源码包，需要升级php到提供源码包的版本

2.解压

    tar -xzvf php-5.6.9.tar.gz 

    cd php-5.6.9/ext

3.生产扩展骨架

./ext_skel --extname=test
提示如下：

Creating directory test
Creating basic files: config.m4 config.w32 .gitignore test.c php_test.h CREDITS EXPERIMENTAL tests/001.phpt test.php [done].


To use your new extension, you will have to execute the following steps:


1.  $ cd ..
2.  $ vi ext/test/config.m4
3.  $ ./buildconf
4.  $ ./configure --[with|enable]-test
5.  $ make
6.  $ ./sapi/cli/php -f ext/test/test.php
7.  $ vi ext/test/test.c
8.  $ make


Repeat steps 3-6 until you are satisfied with ext/test/config.m4 and
step 6 confirms that your module is compiled into PHP. Then, start writing
code and repeat the last two steps as often as necessary.

4.修改配置文件config.m4
先去掉PHP_ARG_ENABLE的三行注释，再在最后面 if结束前加上下面的代码
PHP_REQUIRE_CXX()
PHP_ADD_LIBRARY(stdc++, "", EXTRA_LDFLAGS)
CPPFILE="test.cpp"
PHP_NEW_EXTENSION(test,$CPPFILE, $ext_shared)  
5. 将源文件（test.c文件）后缀改.cpp，再对头文件和源文件加 extern "C"{}
头文件 php_test.h :

```c++
extern "C" {
#ifdef ZTS
#include "TSRM.h"
#endif
}
```

源文件 test.cpp ：

```c++
extern "C" {
#ifdef HAVE_CONFIG_H
#include "config.h"
#endif

#include "php.h"
#include "php_ini.h"
#include "ext/standard/info.h"
}
```

6. 重新编译扩展
         运行：phpize -> ./configure -> make -> make install
         /home/work/php/bin/phpize --clean
         在扩展目录执行如下命令
         ./configure --with-php-config=/home/work/php/bin/php-config --enable-test
         make && make install
         提示：
         Installing shared extensions:     /home/work/php/lib/php/extensions/no-debug-non-zts-20131226/

7.编辑php.ini 添加生成的so
         extension = "test.so"

8.重启 php-fpm   查看 phpinfo  
         pkill  php-fpm
         找到 php-fpm 目录  进入目录  ./php-fpm  
         查看 phpinfo    搜索test  看扩展是否安装成功  