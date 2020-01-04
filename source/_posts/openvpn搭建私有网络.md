---
title: openvpn搭建私有网络
date: 2020-01-02 16:28:07
tags:
---


OpenVPN是一个用于创建虚拟专用网络加密通道的软件包，最早由James Yonan编写。OpenVPN允许创建的VPN使用公开密钥、电子证书、或者用户名／密码来进行身份验证。
它大量使用了OpenSSL加密库中的SSLv3/TLSv1协议函数库。
目前OpenVPN能在Solaris、Linux、OpenBSD、FreeBSD、NetBSD、Mac OS X与Microsoft Windows以及Android和iOS上运行，并包含了许多安全性的功能。它并不是一个基于Web的VPN软件，也不与IPsec及其他VPN软件包兼容。

<!--more-->

### 第一步、为vps安装openvpn及所有所需软件
安装EPEL仓库
wget http://dl.fedoraproject.org/pub/epel/6/i386/epel-release-6-8.noarch.rpm
rpm -Uvh epel-release-6-8.noarch.rpm
安装openvpn
yum install openvpn
安装openvpn最新的easy-rsa，该包用来制作ca证书，服务端证书，客户端证书。最新的为easy-rsa3
wget https://github.com/OpenVPN/easy-rsa/archive/master.zip
unzip master.zip
将解压得到的文件夹easy-rsa-master重命名为easy-rsa
mv easy-rsa-master/ easy-rsa/
然后将的到的easy-ras文件夹复制到/etc/openvpn/目录下
cp -R easy-rsa/ /etc/openvpn/
### 第二步、编辑vars文件，根据自己环境配置
A：先进入/etc/openvpn/easy-rsa/easyrsa3目录
cd /etc/openvpn/easy-rsa/easyrsa3/
B：复制vars.example 为vars
cp vars.example vars
C：修改下面字段，命令：vi vars，然后修改，最后wq保存
set_var EASYRSA_REQ_COUNTRY “CN” //根据自己情况更改
set_var EASYRSA_REQ_PROVINCE “Beijing”
set_var EASYRSA_REQ_CITY “Tong”
set_var EASYRSA_REQ_ORG “qingliu Certificate”
set_var EASYRSA_REQ_EMAIL “shuiqingliu14@gmail.com”
set_var EASYRSA_REQ_OU “My OpenVPN”
### 第三步、创建服务端证书及key
- ./easyrsa init-pki

创建根证书

- ./easyrsa build-ca

如下:
> Generating a 2048 bit RSA private key
> …………………………………….+++
> ……+++
> writing new private key to ‘/root/easy-rsa/easyrsa3/pki/private/ca.key’
> Enter PEM pass phrase:
> Verifying – Enter PEM pass phrase:
> —–
> You are about to be asked to enter information that will be incorporated
> into your certificate request.
> What you are about to enter is what is called a Distinguished Name or a DN.
> There are quite a few fields but you can leave some blank
> For some fields there will be a default value,
> If you enter ‘.’, the field will be left blank.
> —–
> Common Name (eg: your user, host, or server name) [Easy-RSA CA]:qingliu
> CA creation complete and you may now import and sign cert requests.
> Your new CA certificate file for publishing is at:
> /root/easy-rsa/easyrsa3/pki/ca.crt
注意：在上述部分需要输入PEM密码 PEM pass phrase，输入两次，此密码必须记住，不然以后不能为证书签名。还需要输入common name 通用名，这个你自己随便设置个独一无二的。
eg:Common Name (eg: your user, host, or server name) [Easy-RSA CA]:qingliu
###我输入qingliu

创建服务器端证书
./easyrsa gen-req server nopass
如下：
[root@localhost easyrsa3]# ./easyrsa gen-req server nopass
Generating a 2048 bit RSA private key
……………………………………………………………………..+++
……………………+++
writing new private key to ‘/root/easy-rsa/easyrsa3/pki/private/server.key’
—–
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter ‘.’, the field will be left blank.
—–
Common Name (eg: your user, host, or server name) [server]:shuiqingliu
Keypair and certificate request completed. Your files are:
req: /root/easy-rsa/easyrsa3/pki/reqs/server.req
key: /root/easy-rsa/easyrsa3/pki/private/server.key
该过程中需要输入common name，随意但是不要跟之前的根证书的一样
签约服务端证书：
./easyrsa sign server server
如下:
You are about to sign the following certificate.
Please check over the details shown below for accuracy. Note that this request
has not been cryptographically verified. Please be sure it came from a trusted
source or that you have verified the request checksum with the sender.
Request subject, to be signed as a server certificate for 3650 days:
subject=
commonName = shuiqingliu
Type the word ‘yes’ to continue, or any other input to abort.
Confirm request details: yes
Using configuration from /root/easy-rsa/easyrsa3/openssl-1.0.cnf
Enter pass phrase for /root/easy-rsa/easyrsa3/pki/private/ca.key:
Check that the request matches the signature
Signature ok
The Subject’s Distinguished Name is as follows
commonName
RINTABLE:’shuiqingliu’
Certificate is to be certified until Apr 20 06:02:10 2024 GMT (3650 days)
Write out database with 1 new entries
Data Base Updated
Certificate created at: /root/easy-rsa/easyrsa3/pki/issued/server.crt
该命令中.需要你确认生成，要输入yes，还需要你提供我们当时创建CA时候的密码。如果你忘记了密码，那你就重头开始再来一次吧。
创建Diffie-Hellman，确保key穿越不安全网络的命令：
./easyrsa gen-dh
如下：
Note: using Easy-RSA configuration from: ./vars
Generating DH parameters, 2048 bit long safe prime, generator 2
This is going to take a long time
……..+……………………………….+..+…………………………………………………………………………………………………………………………….
DH parameters of size 2048 created at /etc/openvpn/easy-rsa/easyrsa3/pki/dh.pem
第四步、创建客户端证书
进入root目录新建client文件夹，文件夹可随意命名，然后拷贝前面解压得到的easy-ras文件夹到client文件夹,进入下列目录
cd /root/
mkdir client && cd client
cp -R easy-rsa/ client/
cd client/easy-rsa/easyrsa3/
初始化
./easyrsa init-pki
创建客户端key及生成证书（记住生成是自己输入的密码）
./easyrsa gen-req qingliu //名字自己定义
将的到的qingliu.req导入然后签约证书
进入到/etc/openvpn/easy-rsa/easyrsa3/
cd /etc/openvpn/easy-rsa/easyrsa3/
导入req
./easyrsa import-req /root/client/easy-rsa/easyrsa3/pki/reqs/qingliu.req qingliu
签约证书
./easyrsa sign client qingliu
//这里生成client所以必须为client，qingliu要与之前导入名字一致
上面签约证书跟server类似，就不截图了，但是期间还是要输入CA的密码
这步很重要，现在说一下我们上面都生成了什么东西
/etc/openvpn/easy-rsa/easyrsa3/pki/ca.crt
/etc/openvpn/easy-rsa/easyrsa3/pki/reqs/server.req
/etc/openvpn/easy-rsa/easyrsa3/pki/reqs/qingliu.req
/etc/openvpn/easy-rsa/easyrsa3/pki/private/ca.key
/etc/openvpn/easy-rsa/easyrsa3/pki/private/server.key
/etc/openvpn/easy-rsa/easyrsa3/pki/issued/server.crt
/etc/openvpn/easy-rsa/easyrsa3/pki/issued/qingliu.crt
/etc/openvpn/easy-rsa/easyrsa3/pki/dh.pem
客户端：(root/client/easy-rsa文件夹)
/root/client/easy-rsa/easyrsa3/pki/private/qingliu.key
/root/client/easy-rsa/easyrsa3/pki/reqs/qingliu.req //这个文件被我们导入到了服务端文件所以那里也有
这一步就是拷贝这些文件放入到相应位置。将下列文件放到/etc/openvpn/ 目录执行命令：
cp /etc/openvpn/easy-rsa/easyrsa3/pki/ca.crt /etc/openvpn
cp /etc/openvpn/easy-rsa/easyrsa3/pki/private/server.key /etc/openvpn
cp /etc/openvpn/easy-rsa/easyrsa3/pki/issued/server.crt /etc/openvpn
cp /etc/openvpn/easy-rsa/easyrsa3/pki/dh.pem /etc/openvpn
这样就将上述四个文件放入到了/etc/openvpn目录下
b.这一步将下列文件放到/root/client 目录下执行命令：
cp /etc/openvpn/easy-rsa/easyrsa3/pki/ca.crt /root/client
cp /etc/openvpn/easy-rsa/easyrsa3/pki/issued/qingliu.crt /root/client
cp /root/client/easy-rsa/easyrsa3/pki/private/qingliu.key /root/client
这样就将上述三个文件复制到了/root/client目录，包括：ca.crt、qingliu.crt、qingliu.key
第五步、为服务端编写配置文件
当你安装好了openvpn时候，他会提供一个server配置的文件例子，在/usr/share/doc/openvpn-2.3.2/sample/sample-config-files下会有一个server.conf文件，我们将这个文件复制到/etc/openvpn
cp /usr/share/doc/openvpn-2.3.2/sample/sample-config-files/server.conf /etc/openvpn
然后修改配置vi server.conf如下：
local 192.227.161.xx（跟自己vps IP）
port 1194
proto udp
dev tun
ca /etc/openvpn/ca.crt
cert /etc/openvpn/server.crt
key /etc/openvpn/server.key # This file should be kept secret
dh /etc/openvpn/dh.pem
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist ipp.txt
push “redirect-gateway def1 bypass-dhcp”
push “dhcp-option DNS 8.8.8.8″
keepalive 10 120
comp-lzo
max-clients 100
persist-key
persist-tun
status openvpn-status.log
verb 3
每个项目都会由一大堆介绍,上述修改，openvpn提供的server.conf已经全部提供，我们只需要去掉前面的注释#，然后修改我们自己的有关配置。
第六步、下载openvpn客户端，并进行配置
用sftp将我们在vps生成的客户端证书和key下载到客户端电脑
ca.crt qingliu.crt qingliu.key //这三个文件
去官网下载openvpn客户端进行安装，然后安装目录找到simple-config
D:\Program Files\OpenVPN\sample-config\client.ovpn
将client.ovpn 复制到D:\Program Files\OpenVPN\config下，当然我把客户端装在了D盘你根据自己情况选择.
将下载到的三个文件放入D:\Program Files\OpenVPN\config
编辑配置文件：
client
dev tun
proto udp
remote 192.227.161.xx 1194 //主要这里修改成自己vps ip
resolv-retry infinite
nobind
persist-key
persist-tun
ca ca.crt //这里需要证书
cert qingliu.crt
key qingliu.key
comp-lzo
verb 3
我们只需要以上项目每行一个。
第七步、测试排错
启动vps上的openvpn服务
Service openvpn start
Oh，不幸的是出现service start failed！！！
但是当你运行：
Openvpn /etc/openvpn/server.conf
又可以运行，解决办法：
删除/etc/openvpn/下的ipp.txt openvpn-status.log
然后就可以启动服务了。如果你还不能解决，那就去var/log中找message慢慢分析原因