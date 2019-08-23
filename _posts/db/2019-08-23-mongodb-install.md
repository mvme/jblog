---
layout: post
title: MongoDB安装
date: 2019-08-23 13:35:30 +0800
category: 数据库相关
tags: MongoDB
keywords: MongoDB
comments: true
---


安装步骤
--------------------------------

1. 从官网下载.tgz的安装包，Linux可以通过crul 或者 wget 下载都可以
```
wget https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.9.tgz
curl -# -O https://fastdl.mongodb.org/linux/mongodb-linux-x86_64-4.0.9.tgz
```
2. 解压
`tar -xzvf mongodb-linux-x86_64-4.0.9.tgz`
3. 把解压好的文件夹移动到指定目录
`mv mongodb-linux-x86_64-4.0.9 /usr/local/mongodb`
4. 创建MongoDB数据存放目录及日志存放目录
`mkdir -p /home/mongodb/data  /home/mongodb/log`
5. 启动MongoDB服务
    1. 直接启动
    ```
    /usr/local/mongodb/bin/mongod --dbpath /home/mongodb/data/ --logpath /home/mongodb/log/mongodb.log --fork --port 27017 --logappend --bind_ip 0.0.0.0
    ```
    2. 新增MongoDB的配置文件的方式启动
    ```
    touch /etc/mongodb.cnf
    # mongodb的参数说明：
    # --dbpath 数据库路径（数据文件）
    # --logpath 日志文件路径
    # --master 指定为主机器
    # --slave 指定为从机器
    # --source 指定主机器的IP地址
    # --pologSize 指定日志文件大小不超过64M，因为resync是非常操作量打且耗时，最好通过设置一个足够大的oplogSize来避免resync(默认的oplog大小是空闲磁盘大小的5%)。
    # --logappend 日志文件末尾添加
    # --port 启动端口号
    # --fork 在后台运行
    # --only 指定只复制哪一个数据库
    # --slavedelay 指从复制检测的时间间隔
    # --auth 是否需要验证权限登录（用户名和密码）
    # 注：MongoDB配置文件里面的参数很多，定制特定的需求，请参考官方文档
    dbpath=/home/mongodb/data
    logpath=/home/mongodb/log/mongodb.log
    logappend=true
    port=27017
    fork=true
    bing_ip=0.0.0.0
    ```
执行此命令启动：
`/usr/local/mongodb/bin/mongod -f /etc/mongodb.cnf`




mongodb服务自启动设置
-------------------------------
```
echo "/usr/local/mongodb/bin/mongod --dbpath /home/mongodb/data/ --logpath /home/mongodb/log/mongodb.log --fork --port 27017 --logappend --bind_ip 0.0.0.0" >> /etc/rc.local
```




MongoDB复制集shell脚本
-----------------------------

```sh
#!/bin/bash
#IP=192.168.1.165
# ip addr | grep ens33 | grep -o -E "inet [0-9.]+" | grep -o -E "[0-9.]+"
IP=`ip addr | grep ens33 | grep -o -E "inet [0-9.]+" | awk '{print $2}'`
NA=rs2

if [ $1 == 'reset' ]; then
    pkill -9 mongo
    rm -rf /home/mongodb
    exit;
fi

if [ $1 == 'repl' ]; then
mkdir -p /home/mongodb/data/m17 /home/mongodb/data/m18 /home/mongodb/data/m19 /home/mongodb/log

/usr/local/mongodb/bin/mongod --dbpath /home/mongodb/data/m17/ --logpath /home/mongodb/log/m17.log --port 27017 --logappend --fork --bind_ip 0.0.0.0 --replSet ${NA}

/usr/local/mongodb/bin/mongod --dbpath /home/mongodb/data/m18/ --logpath /home/mongodb/log/m18.log --port 27018 --logappend --fork --bind_ip 0.0.0.0 --replSet ${NA}

/usr/local/mongodb/bin/mongod --dbpath /home/mongodb/data/m19/ --logpath /home/mongodb/log/m19.log --port 27019 --logappend --fork --bind_ip 0.0.0.0 --replSet ${NA}

/usr/local/mongodb/bin/mongo <<EOF
use admin
var rsconf = {
      _id: "${NA}",
      version: 1,
      members: [
         { _id: 0, host: "${IP}:27017" },
         { _id: 1, host: "${IP}:27018" },
         { _id: 2, host: "${IP}:27019" }
      ]
   }

rs.initiate(rsconf);
EOF

exit;
fi
```



ip变动错误解决
-----------------------

```
db.isMaster(): Does not have a valid replica set config

rs0:OTHER> db.isMaster()
{
    "hosts" : [
        "mongodb-rs0-0.mongodb-rs0.default.svc.cluster.local:27017",
        "mongodb-rs0-1.mongodb-rs0.default.svc.cluster.local:27017",
        "mongodb-rs0-2.mongodb-rs0.default.svc.cluster.local27017"
    ],
    "setName" : "rs0",
    "ismaster" : false,
    "secondary" : false,
    "info" : "Does not have a valid replica set config",
    "isreplicaset" : true,
    "maxBsonObjectSize" : 16777216,
    "maxMessageSizeBytes" : 48000000,
    "maxWriteBatchSize" : 100000,
    "localTime" : ISODate("2018-07-10T14:34:48.640Z"),
    "logicalSessionTimeoutMinutes" : 30,
    "minWireVersion" : 0,
    "maxWireVersion" : 6,
    "readOnly" : false,
    "ok" : 1,
    "operationTime" : Timestamp(1531232610, 1),
    "$clusterTime" : {
        "clusterTime" : Timestamp(1531232610, 1),
        "signature" : {
            "hash" : BinData(0,"AAAAAAAAAAAAAAAAAAAAAAAAAAA="),
            "keyId" : NumberLong(0)
        }
    }
}

You could just re-configure the peplica set and only keep reachable members.

rs0:OTHER> re_conf = rs.conf()
rs0:OTHER> re_conf.members = [re_conf.members[0]]
rs0:OTHER> rs.reconfig(re_conf, {force : true})
rs0:PRIMARY> rs.add("mongodb-rs0-1.mongodb-rs0.default.svc.cluster.local:27017")
rs0:PRIMARY> rs.add("mongodb-rs0-2.mongodb-rs0.default.svc.cluster.local:27017")
```