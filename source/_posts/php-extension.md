---
title: php extension
date: 2017-01-08 00:23:28
tags: php
---

PHP扩展（PECL）跟PHP引擎一样都是使用C语言开发。PHP核心开发组成员鸟哥Laruence使用的是VIM进行PHP开发。
http://www.laruence.com/2011/09/13/2139.html
书籍: http://www.walu.cc/phpbook/
案例: php-src/ext
PECL开发邮件组: http://news.php.net/php.pecl.dev
尽量编写一些phpt测试用例,php-src/tests下有很多参考.
测试时用--enable-debug编译PHP,要做到执行你的扩展逻辑,不输出任何错误信息.
用valgrind检测内存泄露.
<!--more-->

PHP异步网络扩展Swoole作者博客： http://rango.swoole.com/
PHP中文分词扩展SCWS和XunSearch全文搜索引擎作者博客： http://hightman.cn
PHP代码加密扩展php-beast作者博客：http://my.oschina.net/liexusong/blog

如果你在Linux上要使用IDE开发，可以看看Eclipse CDT或者[Qt Creator](http://my.oschina.net/eechen/blog/166969)。

构建PHP扩展:
http://wiki.swoole.com/wiki/page/238.html (视频教程)
http://php.net/manual/zh/internals2.buildsys.php
php-src/ext/ext_skel脚本用于生成PECL扩展源码骨架.
