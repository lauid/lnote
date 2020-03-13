---
title: small-case
date: 2019-12-28 17:04:20
tags: note,tools
---
### 查看服务端口

ss -taln
ss -ltp
-l = listen, -t = tcp, -p = show program name

### ssh相关文件权限配置

chmod 700 -R .ssh #设置.ssh目录权限

chmod 600. ssh/authorized_keys

chmod 400 私钥

使用 ssh-keygen -m PEM 将 OpenSSH 密钥转换为 PEM 格式

<!--more-->

#### 根据私钥读取公钥
sh-keygen -y -f <path/to/private_key> > <path/to/public_key>
-y Read private key file and print public key
-y 的意思是读取私钥并将公钥打印出来

#### 生成pem格式的create RSA private key
ssh-keygen -m PEM -t rsa -b 4096 -C "dep1@example.com" -f dep1.pem

#### 用户相关

修改用户sh
usermod -s /bin/bash kaifa
- /bin/false什么也不做只是返回一个错误状态,然后立即退出。将用户的shell设置为/bin/false,用户会无法登录,并且不会有任何提示。
- /usr/sbin/nologin
nologin会礼貌的向用户显示一条信息,并拒绝用户登录:
This account is currently not available.
有一些软件,比如一些ftp服务器软件,对于本地非虚拟账户,只有用户有有效的shell才能使用ftp服务。这时候就可以使用nologin使用户即不能登录系统,还能使用一些系统服务,比如ftp服务。/bin/false则不行,这是二者的重要区别之一。
- /etc/nologin
如果存在/etc/nologin文件,则系统只允许root用户登录,其他用户全部被拒绝登录,并向他们显示/etc/nologin文件的内容。


删除用户
 rm -rf /var/spool/mail/user1
 rm -rf /home/user1
userdel -r user1    这样可以完全删除用户的家目录，邮箱目录等。

#### 随机密码生成

openssl rand -base64 10|tr A-Z a-z

#### visudo修改编辑器vim
update-alternatives --config editor

#### nologin用户

在linux中建立网站时，我们一般分配一个www之类的用户给网站应用程序。
如果我们使用root或者具有管理员权限的账号在网站目录下去创建文件时，会遇到各种权限问题。
这时我们可以切换到www用户，这类用户一般是nologin，不允许登录。
如果我们su www或者sudo www,切换到www用户时，会出错。
网上解决办法时修改/etc/passwd文件 nologin改为bin/bash，这样www用户可以登录服务器，
比较危险。可以通过以下办法使用www用户执行命令
方法1. 为了安全，使用nologin账号来运行程序，
su -s /bin/bash -c "ls" www
这条命令到底做了什么呢？su -s 是指定shell，这里www用户是nologin用户，是没有默认的shell的，这里指定使用/bin/bash, -c 后面接需要运行的命令， 后面www是用www用户来运行
方法2：
sudo -u www command 这样也可以使用www用户来执行命令

#### 回收站工具

现在我们来安装 trash-cli加入我们使用 CentOS，Fedora，Ubuntu等主流操作系统，我们可以直接使用软件包管理命令安装如

安装trash-cli工具，其实就是回收站的命令行模式：

sudo apt-get install trash-cli

如果是centos系统
sudo yum install -y trash-cli


#### yum之后查看安装包安装了那些东西


rpm -ql openvpn

#### deb安装到指定目录
有时候，我们没有root用户的时候，我们进行安装deb包就不能之间安装到系统之中了；

为了方便，我们可以直接解压 dpkg -x same.deb .;

直接解压到当前目录，然后在配置环境变量，即可启动运行程序；

当然，还有另外一个命令：

dpkg -i --instdir=/dest/dir/path some.deb

#### 查找linux下进程占用CPU过高的原因
- 找出占用CPU最高的10个进程
ps aux | sort -k3nr | head -n 10
- 或查看占用内存最高的10个进程
ps aux | sort -k4nr | head -n 10　　
