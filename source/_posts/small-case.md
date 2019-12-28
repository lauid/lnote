---
title: small-case
date: 2019-12-28 17:04:20
tags: note,tools
---

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

#### 用户相关

修改用户sh
usermod -s /bin/bash kaifa

删除用户
 rm -rf /var/spool/mail/user1
 rm -rf /home/user1
userdel -r user1    这样可以完全删除用户的家目录，邮箱目录等。

#### 随机密码生成

openssl rand -base64 10|tr A-Z a-z
