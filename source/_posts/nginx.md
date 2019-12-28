---
title: nginx常用配置
date: 2019-12-28 17:07:55
tags: nginx
---

语法
location的语法规则：location [=|~|~*|^~] /uri/ { … }
location匹配的变量是$uri
关于几种字符的说明

字符	描述
=	表示精准匹配
~	表示区分大小写的正则匹配
~*	表示不区分大小写的正则匹配
^~	表示uri以指定字符或字符串开头
/	通用匹配，任何请求都会匹配到规则优先级
= 高于 ^~ 高于 ~* 等于 ~ 高于 /

<!--more-->

示例1

location = "/12.jpg" { ... }
如：
www.syushin.com/12.jpg 匹配
www.syushin.com/abc/12.jpg 不匹配

location ^~ "/abc/" { ... }
如：
www.syushin.com/abc/123.html 匹配
www.syushin.com/a/abc/123.jpg 不匹配

location ~ "png" { ... }
如：
www.syushin.com/aaa/bbb/ccc/123.png 匹配
www.syushin.com/aaa/png/123.html 匹配

location ~* "png" { ... }
如：
www.syushin.com/aaa/bbb/ccc/123.PNG 匹配
www.syushin.com/aaa/png/123.html 匹配


location /admin/ { ... }
如：
www.syushin.com/admin/aaa/1.php 匹配
www.syushin.com/123/admin/1.php 不匹配
注意：
有些资料上介绍location支持不匹配 !~如： location !~ 'png'{ ... }
这是错误的，location不支持 !~
如果有这样的需求，可以通过if(location优先级小于if )来实现，如： if ($uri !~ 'png') { ... }

访问控制
web2.0时代，很多网站都是以用户为中心，网站允许用户发布内容到服务器。由于为用户开放了上传功能，因此有很大的安全风险，比如黑客上传木马程序等等。因此，访问控制就很有必要配置了。

deny与allow
字面上很容易理解就是拒绝和允许。
Nginx的deny和allow指令是由ngx_http_access_module模块提供，Nginx安装默认内置了该模块。

语法
语法：allow/deny address | CIDR | unix: | all

它表示，允许/拒绝某个ip或者一个ip段访问.如果指定unix:,那将允许socket的访问。
注意：unix在1.5.1中新加入的功能。
在nginx中，allow和deny的规则是按顺序执行的。

示例1：

location /
{
    allow 192.168.0.0/24;
    allow 127.0.0.1;
    deny all;
}
说明：这段配置值允许192.168.0.0/24网段和127.0.0.1的请求，其他来源IP全部拒绝。

示例2：

location ~ "admin"
{
    allow 192.168.30.7;
    deny all
}
说明：访问的uri中包含admin的请求，只允许192.168.30.7这个IP的请求。

基于location的访问控制
日常上，访问控制基本是配合location来做配置的，直接例子吧。
示例1：

location /blog/
{
    deny all;
}
说明：针对/blog/目录，全部禁止访问，这里的deny all;可以改为return 403;.
示例2

location ~ ".bak|\.ht"
{
    return 403;
}
说明：访问的uri中包含.bak字样的或者包含.ht的直接返回403状态码。

测试链接举例：

www.syushin.com/abc.bak

www.syushin.com/blog/123/.htalskdjf

如果用户输入的URL是上面其中之一都会返回403。
示例3

location ~ (data|cache|tmp|image|attachment).*\.php$
{
    deny all;
}
说明：请求的uri中包含data、cache、tmp、image、attachment并且以.php结尾的，全部禁止访问。

测试链接举例：

www.xxxxxx.com/aming/cache/1.php

www.xxxxxxx.com/image/123.phps

www.xxxxxx.com/aming/datas/1.php

基于$document_uri的访问控制
前面介绍了内置变量$document_uri含义是当前请求中不包含指令的URI。
如www.123.com/1.php?a=1&b=2的$document_uri就是1.php,不包含后面的参数。
我们可以针对这个变量做访问控制。
示例1

if ($document_uri ~ "/admin/")
{
    return 403;
}
说明：当请求的uri中包含/admin/时，直接返回403.

注意：if结构中不支持使用allow和deny。

测试链接：

1. www.xxxxx.com/123/admin/1.html 匹配
2. www.xxxxx.com/admin123/1.html  不匹配
3. www.xxxxx.com/admin.php  不匹配
示例2

if ($document_uri = /admin.php)
{
    return 403;
}
说明：请求的uri为/admin.php时返回403状态码。

测试链接：

1. www.xxxxx.com/admin.php # 匹配
2. www.xxxxx.com/123/admin.php # 不匹配
示例3

if ($document_uri ~ '/data/|/cache/.*\.php$')
{
    return 403;
}
说明：请求的uri包含data或者cache目录，并且是php时，返回403状态码。

测试链接：

1. www.xxxxx.com/data/123.php  # 匹配
2. www.xxxxx.com/cache1/123.php # 不匹配
基于$request_uri访问控制
$request_uri比$docuemnt_uri多了请求的参数。主要是针对请求的uri中的参数进行控制。
示例

if ($request_uri ~ "gid=\d{9,12}")
{
    return 403;
}
说明：\d{9,12}是正则表达式，表示9到12个数字，例如gid=1234567890就符号要求。

测试链接：

1. www.xxxxx.com/index.php?gid=1234567890&pid=111  匹配
2. www.xxxxx.com/gid=123  不匹配
背景知识：
曾经有一个客户的网站cc攻击，对方发起太多类似这样的请求：/read-123405150-1-1.html
实际上，这样的请求并不是正常的请求，网站会抛出一个页面，提示帖子不存在。
所以，可以直接针对这样的请求，return 403状态码。

基于$http_user_agent的访问控制(反爬虫)
user_agent可以简单理解成浏览器标识，包括一些蜘蛛爬虫都可以通过user_agent来辨识。假如观察访问日志，发现一些搜索引擎的蜘蛛对网站访问特别频繁，它们并不友好。为了减少服务器的压力，其实可以把除主流搜索引擎蜘蛛外的其他蜘蛛爬虫全部封掉。
示例

if ($user_agent ~ 'YisouSpider|MJ12bot/v1.4.2|YoudaoBot|Tomato')
{
    return 403;
}
说明：user_agent包含以上关键词的请求，全部返回403状态码。

测试：

1. curl -A "123YisouSpider1.0"
2. curl -A "MJ12bot/v1.4.1"
基于$http_referer的访问控制
$http_referer除了可以实现防盗链的功能外，还可以做一些特殊的需求。
比如：

网站被黑挂马，搜索引擎收录的网页是有问题的，当通过搜索引擎点击到网站时，却显示一个博彩网站。
由于查找木马需要时间，不能马上解决，为了不影响用户体验，可以针对此类请求做一个特殊操作。
比如，可以把从百度访问的链接直接返回404状态码，或者返回一段html代码。
示例

if ($http_referer ~ 'baidu.com')
{
    return 404;
}
或者

if ($http_referer ~ 'baidu.com')
{
    return 200 "<html><script>window.location.href='//$host$request_uri';</script></html>";
}
Nginx参数优化
Nginx作为高性能web服务器，即使不特意调整配置参数也可以处理大量的并发请求。当然，配置调优会使Nginx性能更加强悍，配置参数需要结合服务器硬件性能等做参考。

worker进程优化
worker_processes num;

该参数表示启动几个工作进程，建议和本机CPU核数保持一致，每一核CPU处理一个进程，num表示数字。
worker_rlimit_nofile

它表示Nginx最大可用的文件描述符个数，需要配合系统的最大描述符，建议设置为102400。
还需要在系统里执行ulimit -n 102400才可以。
也可以直接修改配置文件/etc/security/limits.conf修改
增加：
#* soft nofile 655350 (去掉前面的#)
#* hard nofile 655350 (去掉前面的#)
worker_connections

该参数用来配置每个Nginx worker进程最大处理的连接数,
这个参数也决定了该Nginx服务器最多能处理多少客户端请求（worker_processes * worker_connections)
建议把该参数设置为10240，不建议太大。
http/tcp连接数优化
use epoll

使用epoll模式的事件驱动模型，该模型为Linux系统下最优方式。
multi_accept on

使每个worker进程可以同时处理多个客户端请求。
sendfile on

使用内核的FD文件传输功能，可以减少user mode和kernel mode的切换，从而提升服务器性能。
tcp_nopush on

当tcp_nopush设置为on时，会调用tcp_cork方法进行数据传输。
使用该方法会产生这样的效果：当应用程序产生数据时，
内核不会立马封装包，而是当数据量积累到一定量时才会封装，然后传输。
tcp_nodelay on

不缓存data-sends（关闭 Nagle 算法），这个能够提高高频发送小数据报文的实时性。
(关于Nagle算法)

【假如需要频繁的发送一些小包数据，比如说1个字节，以IPv4为例的话，则每个包都要附带40字节的头，
也就是说，总计41个字节的数据里，其中只有1个字节是我们需要的数据。
为了解决这个问题，出现了Nagle算法。
它规定：如果包的大小满足MSS，那么可以立即发送，否则数据会被放到缓冲区，等到已经发送的包被确认了之后才能继续发送。
通过这样的规定，可以降低网络里小包的数量，从而提升网络性能。
keepalive_timeout

定义长连接的超时时间，建议30s，太短或者太长都不一定合适，当然，最好是根据业务自身的情况来动态地调整该参数。
keepalive_requests

定义当客户端和服务端处于长连接的情况下，每个客户端最多可以请求多少次，可以设置很大，比如50000.
reset_timeout_connection on

设置为on的话，当客户端不再向服务端发送请求时，允许服务端关闭该连接。
client_body_timeout

客户端如果在该指定时间内没有加载完body数据，则断开连接，单位是秒，默认60，可以设置为10。
send_timeout

这个超时时间是发送响应的超时时间，即Nginx服务器向客户端发送了数据包，但客户端一直没有去接收这个数据包。
如果某个连接超过send_timeout定义的超时时间，那么Nginx将会关闭这个连接。单位是秒，可以设置为3。
压缩
对于纯文本的内容，Nginx是可以使用gzip压缩的。使用压缩技术可以减少对带宽的消耗。
由ngx_http_gzip_module模块支持

配置如下：

gzip on; //开启gzip功能
gzip_min_length 1024; //设置请求资源超过该数值才进行压缩，单位字节
gzip_buffers 16 8k; //设置压缩使用的buffer大小，第一个数字为数量，第二个为每个buffer的大小
gzip_comp_level 6; //设置压缩级别，范围1-9,9压缩级别最高，也最耗费CPU资源
gzip_types text/plain application/x-javascript text/css application/xml image/jpeg image/gif image/png; //指定哪些类型的文件需要压缩
gzip_disable "MSIE 6\."; //IE6浏览器不启用压缩
测试：

curl -I -H "Accept-Encoding: gzip, deflate" http://www.xxxxx.com/1.css
日志
错误日志级别调高，比如crit级别，尽量少记录无关紧要的日志。

对于访问日志，如果不要求记录日志，可以关闭，

静态资源的访问日志关闭

静态文件过期
对于静态文件，需要设置一个过期时间，这样可以让这些资源缓存到客户端浏览器，
在缓存未失效前，客户端不再向服务期请求相同的资源，从而节省带宽和资源消耗。

配置示例如下：

location ~* ^.+\.(gif|jpg|png|css|js)$                                      
{
    expires 1d; //1d表示1天，也可以用24h表示一天。
}