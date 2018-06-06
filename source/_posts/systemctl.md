---
title: systemctl
date: 2018-05-21 09:42:13
tags: systemctl
categories: 运维
---

1、systemctl是RHEL 7 的服务管理工具中主要的工具，它融合之前service和chkconfig的功能于一体。可以使用它永久性或只在当前会话中启用/禁用服务。

systemctl可以列出正在运行的服务状态

systemd-cgls以树形列出正在运行的进程，它可以递归显示控制组内容。如图：


2、如何启动/关闭、启用/禁用服务？

启动一个服务：systemctl start postfix.service
关闭一个服务：systemctl stop postfix.service
重启一个服务：systemctl restart postfix.service
显示一个服务的状态：systemctl status postfix.service
在开机时启用一个服务：systemctl enable postfix.service
在开机时禁用一个服务：systemctl disable postfix.service
查看服务是否开机启动：systemctl is-enabled postfix.service
查看已启动的服务列表：systemctl list-unit-files|grep enabled
查看启动失败的服务列表：systemctl --failed

PS：使用命令 systemctl is-enabled postfix.service 得到的值可以是enable、disable或static，这里的 static 它是指对应的 Unit 文件中没有定义[Install]区域，因此无法配置为开机启动服务。 

说明：启用服务就是在当前“runlevel”的配置文件目录/etc/systemd/system/multi-user.target.wants/里，建立/usr/lib/systemd/system里面对应服务配置文件的软链接；禁用服务就是删除此软链接，添加服务就是添加软连接。





