---
title: rpm打包
date: 2018-05-11 11:19:03
tags: rpm
---

装rpm包制作工具
sudo yum install -y rpm-build rpmdevtools


生成一个 rpm 包的骨架目录
rpmdev-setuptree

生成一个 spec 骨架文件
rpmdev-newspec  helloworld.spec

打包
rpmbuild -ba SPECS/helloworld.spec  


解压
rpm2cpio oracle-instantclient11.2-basic-11.2.0.2.0.i386.rpm | cpio -div

