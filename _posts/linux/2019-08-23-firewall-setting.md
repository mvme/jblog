---
layout: post
title: centos7 防火墙设置
date: 2019-08-23 13:48:30 +0800
category: Linux相关
tags: centos7
keywords: centos7,linux,firewalld
comments: true
---

centos7 防火墙配置
-------------------------

1. 查看firewall服务状态
`systemctl status firewalld`
2. 查看firewall的状态
`firewall-cmd --state`
3. 开启、重启、关闭firewalld.service服务
```
# 开启
service firewalld start
# 重启
service firewalld restart
# 关闭
service firewalld stop
```
4. 查看防火墙规则
`firewall-cmd --list-all`
5. 查询、开放、关闭端口
```
# 查询端口是否开放
firewall-cmd --query-port=8080/tcp
# 开放80端口
firewall-cmd --permanent --add-port=80/tcp
# 移除端口
firewall-cmd --permanent --remove-port=8080/tcp
#重启防火墙(修改配置后要重启防火墙)
firewall-cmd --reload
# 参数解释
1、firwall-cmd：是Linux提供的操作firewall的一个工具；
2、--permanent：表示设置为持久；
3、--add-port：标识添加的端口；
```


Linux查询全部进程数
---------------------------

```
ps -ef | grep -v "grep" | wc -l
```

查询Java进程数
---------------------------

```
ps -ef | grep java | grep -v "grep" | wc -l
```

输出全部进程pid
-------------------------

```
ps -ef | grep -v "grep" | awk '{print $2}'
```

输出Java进程pid
--------------------------------

```
ps -ef | grep java | grep -v "grep" | awk '{print $2}'
```

shell脚本之java启动
---------------------------

```sh
#!/bin/bash
pid=`ps -ef | grep java | grep -v grep | awk '{print $2}'`

# echo "删除zfjd*.jar和nohup.out文件"
# rm -rf zfjd*.jar nohup.out

echo "kill -9 pid:" $pid
kill -9 $pid

echo "启动zfjd进程..."
nohup java -jar zfjd*.jar > /home/nohup.out 2>&1 &
```