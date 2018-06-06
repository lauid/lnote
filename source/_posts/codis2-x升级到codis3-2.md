---
title: codis2.x升级到codis3.2
date: 2018-03-21 09:58:39
tags: codis
categories: 运维
---

## step 1.配置迁移

step a.导出配置文件
```
/usr/local/codis3.2/bin/codis-admin --config-dump --product=apitest --zookeeper=10.0.0.40:2181 -1 | tee codis_v2.0.json
```

该命令会从 zookeeper 上拉取 /zk/codis/db_codis_v2.0 下全部的文件，并组织成 json 格式并输出。
选项 -1 表示配置文件是 Codis 1.x 版本，缺省是 Codis 3.x 版本。

step b. 转换配置文件
```
/usr/local/codis3.2/bin/codis-admin --config-convert codis_v2.0.json | tee codis_v3.0.json
```
该命令会将 Codis 1.x 版本的配置文件中有效信息提取出来，并转成 Codis 3.x 版本的配置文件并输出。

step c. 更新配置文件
注意：更新配置文件时，请确保 Codis 3.x 中该集群不存在，否则可能导致更新失败或者集群状态异常。
```
NEW_PRODUCT_NAME="apitet_codis3.2"
ZOOKEEPER_ADDR="10.0.0.40:2181"
#/usr/local/codis3.2/bin/codis-admin --config-restore=codis_v3.0.json --product=$NEW_PRODUCT_NAME --zookeeper=$ZOOKEEPER_ADDR --confirm
/usr/local/codis3.2/bin/codis-admin --config-restore=codis_v3.0.json --product=apitest_codis3.2 --zookeeper=10.0.0.40:2181 --confirm
```

<!--more-->
## step 2.check新集群配置文件
```
/usr/local/zookeeper/bin/zkCli.sh -server 10.0.0.40:2181
mkdir -p /home/config/codis3.2
mkdir -p /home/logs/codis3.2
mkdir -p /home/data/codis3.2
chown -R codis:codis /home/config/codis3.2/
chown -R codis:codis /home/data/codis3.2/
```


## step 3.codis-dashboard 生成默认的配置文件
```
/usr/local/codis3.2/bin/codis-dashboard --default-config | tee /home/config/codis3.2/dashboard.toml
```
PS. 修改其中相关配置:[coordinator_name="zookeeper", coordinator_addr, product_name, admin_addr];



## step 4.配置supervisor，并启动dashboard
```
nohup /usr/local/codis3.2/bin/codis-dashboard --ncpu=2 --config=/home/config/codis3.2/dashboard.toml --log=/home/logs/codis3.2/dashboard.log --log-level=WARN &
```

## step 5.生成codis-fe配置文件,check编辑
配置文件codis.json可以手动编辑，也可以通过codis-admin从外部存储（这里是zookeeper）中拉取，如下操作：
```
/usr/local/codis3.2/bin/codis-admin --dashboard-list --zookeeper=10.0.0.40:2181 | tee /home/config/codis3.2/codis.json
```

## step 6.配置supervisor，启动codis-fe
```
nohup /usr/local/codis3.2/bin/codis-fe --ncpu=2 --log=/home/logs/codis3.2/fe.log --log-level=WARN --dashboard-list=/home/config/codis3.2/codis.json --listen=0.0.0.0:8080 &
```


## step 7.编辑codis-server配置文件
## step 8.配置supervisor，启动codis-server, 添加到新集群做slave
```
/usr/local/codis3.2/bin/codis-server /home/config/codis3.2/codis-server-14369.conf
```
PS. mkdir [dir] ;
    运行codis-server的当前用户，需要写权限「dir, /home/config/codis3.2/」,否则会导致主从同步失败;
    测试redis-cli -h 10.0.0.40 -p 14369 ping, 否则无法加入到server-group;
    #Opening the temp file needed for MASTER <-> SLAVE synchronization: Permission denied
    #CONFIG REWRITE failed: Permission denied

## step 9.codis-procxy 生成默认的配置文件:
```
/usr/local/codis3.2/bin/codis-proxy --default-config | tee /home/config/codis3.2/proxy.toml
```

## step 10. 配置supervisor，启动codis-proxy
```
nohup /usr/local/codis3.2/bin/codis-proxy --ncpu=2 --config=/home/config/codis3.2/proxy-19400.toml --log=/home/logs/codis3.2/proxy-19400.log --log-level=WARN &
```

## step 11. 集群添加codis-proxy
codis-proxy启动后，处于waiting状态，监听proxy_addr地址，但是不会accept连接，添加到集群并完成集群状态的同步，才能改变状态为online。添加的方法有以下两种：
第一种：通过codis-fe添加，通过Add Proxy按钮，将admin_addr加入到集群中，如下图（具体操作要等到后面codis-fe启动后才可以）：
第二种：通过codis-admin命令行工具添加，方法如下（添加3个proxy）：
```
/usr/local/codis3.2/bin/codis-admin --dashboard=127.0.0.1:18080 --create-proxy --addr 127.0.0.1:11080
/usr/local/codis3.2/bin/codis-admin --dashboard=127.0.0.1:18080 --create-proxy -x 127.0.0.1:11081
```
其中–dashboard需要指定codis-dashboard的管理地址，–create-proxy指定为和codis-proxy的admin_addr地址,。添加过程中，codis-dashboard会完成如下一系列动作：
1）获取proxy信息，对集群name以及auth进行验证，并将其信息写入到外部存储中；
2）同步slots状态；
3）标记proxy状态为online，此后proxy开始accept连接并开始提供服务；
PS. 在添加codis-proxy的时候需要特别注意，这个地方容易出现各种问题，最好能一次性添加成功，不然就需要删除zookeeper数据了，然后重新开始。


## step 12. 业务全部切换到新的codis-proxy,
## step 13. 保证业务不再使用老集群proxy，可以check老集群后台的ops为0，之后，再promote新的codis-server为master, 完成升级
PS. slave为了数据一致性，默认为只读。

切换之后，就无法切换回来。codis2的server无法兼容codis3数据, 就是codis-server2为slave，codis3-server为master 无法兼容
一般是低版本无法兼容高版本的 rdb 导致的。
[21658] 21 Mar 18:53:32.735 # Can't handle RDB format version 7
[21658] 21 Mar 18:53:32.735 # Fatal error loading the DB: Invalid argument. Exiting.


## xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx其他常用操作
##添加Redis Server Group（在集群中一台服务器上操作即可，也可以在面板图形界面操作)

首先使用codis-admin工具创建组，按照上面的规划的，我们创建2个组即可：
/usr/local/codis3.2/bin/codis-admin --dashboard=127.0.0.1:18080 --create-group --gid=1
/usr/local/codis3.2/bin/codis-admin --dashboard=127.0.0.1:18080 --create-group --gid=2

然后使用codis-admin工具给组添加Redis主机（2个主机为一个组）：
/usr/local/codis3.2/bin/codis-admin --dashboard=127.0.0.1:18080 --group-add --gid=1 --addr=127.0.0.1:6379
/usr/local/codis3.2/bin/codis-admin --dashboard=127.0.0.1:18081 --group-add --gid=1 --addr=127.0.0.1:6380

/usr/local/codis3.2/bin/codis-admin --dashboard=127.0.0.1:18082 --group-add --gid=2 --addr=127.0.0.1:6381
/usr/local/codis3.2/bin/codis-admin --dashboard=127.0.0.1:18083 --group-add --gid=2 --addr=127.0.0.1:6382

把从库跟主库同步:
/usr/local/codis3.2/bin/codis-admin --dashboard=127.0.0.1:18080 --sync-action --create --addr=127.0.0.1:6379
/usr/local/codis3.2/bin/codis-admin --dashboard=127.0.0.1:18080 --sync-action --create --addr=127.0.0.1:6380
需要注意的是如果当从库运行一段时间后挂掉了，那么重新启动后需要人为手动地将主从进行同步，执行上面的命令即可，或者图形界面操作（反之，如果主库挂了，马上又好了，这时候从库会自动重新连接上的，不需要人为干预）。

若slave需要提升为master，操作如下:
/usr/local/codis3.2/bin/codis-admin --dashboard=10.0.60.152:18080 --promote-server --gid=1 --addr=10.0.60.154:6379
注意的是，当从库提升为主库后，那么原来的主库就会掉线，如果想把它作为现在主库的从库，还是需要人为干预的，因为这个涉及到数据安全性的问题。

##初始化slots并设置server group服务的slot 范围（在集群中一台服务器上操作即可，也可以在面板图形界面操作）

Codis采用Pre-sharding的技术来实现数据的分片, 默认分成1024个slots (0-1023)，对于每个key来说，通过以下公式确定所属的Slot Id : SlotId = crc32(key) %1024。每一个slot都会有一个且必须有一个特定的server group id来表示这个slot的数据由哪个server group来提供。

/usr/local/codis3.2/bin/codis-admin --dashboard=10.0.60.152:18080 --slot-action --create-range --beg=0 --end=511 --gid=1
/usr/local/codis3.2/bin/codis-admin --dashboard=10.0.60.152:18080 --slot-action --create-range --beg=512 --end=1023 --gid=2
