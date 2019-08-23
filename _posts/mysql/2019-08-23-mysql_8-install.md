---
layout: post
title: mysql 8安装
date: 2019-08-23 11:12:30 +0800
category: mysql相关
tags: mysql
keywords: mysql
comments: true
---

安装步骤
------------------------

1. 从官网下载安装包
`mysql-8.0.16-el7-x86_64.tar.gz`
2. 解压
`tar -xzvf mysql-8.0.16-el7-x86_64.tar.gz`
3. 移动到/usr/local/mysql目录
`mv mysql-8.0.16-el7-x86_64 /usr/local/mysql`
4. 创建mysql数据存放目录和日志目录
`mkdir -p /home/mysql/data /home/mysql/log`
5. 创建mysql用户组
`groupadd mysql`
6. 创建mysql用户并添加到mysql用户组
`useradd -r -g mysql -s /bin/false mysql`
7. 修改mysql文件夹访问权限
`chown -R mysql:mysql /home/mysql /usr/local/mysql`
8. 查询mariadb数据库并卸载
```
rpm -qa | grep mariadb
rpm -e --nodeps mariadb-libs-5.5.56-2.el7.x86_64
```
9. 编辑mysql配置文件
```
[mysql]
max_allowed_packet = 500M
default-character-set = UTF8MB4
[mysqld]
# 密码认证插件
default_authentication_plugin=mysql_native_password
port = 3306
basedir = /usr/local/mysql
datadir = /home/mysql/data
max_connections = 200
character-set-server = UTF8MB4
collation-server = utf8mb4_unicode_ci
init_connect = 'SET NAMES utf8mb4'
character-set-client-handshake = FALSE
default-storage-engine = INNODB
# skip-grant-tables
server-id = 1
log-bin = /home/mysql/data/mysqlbin
log-error = /home/mysql/log/mysqlerr
read-only = 0
binlog-ignore-db = mysql
binlog-ignore-db = information_schema
binlog-ignore-db = performance_schema
performance_schema_max_table_instances = 600
table_definition_cache = 400
table_open_cache = 256
# sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES
# 权限密码认证...
default_authentication_plugin=mysql_native_password
[client]
default-character-set = UTF8MB4
```
10. 初始化数据库
`/usr/local/mysql/bin/mysqld --initialize --user=mysql --console`
11. 启动mysql服务
```
/usr/local/mysql/support-files/mysql.server start
启动ssl
  /usr/local/mysql/bin/mysql_ssl_rsa_setup
  启动mysql服务方式二：
  /usr/local/mysql/bin/mysqld_safe --user=mysql &
```
12. 把mysql服务加入到系统服务进程
`cp /usr/local/mysql/support-files/mysql.server /etc/init.d/mysql.server`
13. 重启mysql服务
`service mysqld restart`
14. 查看mysql生产的随机密码
`grep 'temporary password' /home/mysql/log/mysqlerr.err`
15. 登陆数据库
`/usr/local/mysql/bin/mysql -u root -p`
16. 修改密码
`ALTER `user` 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'root';`
17. 设置允许远程登陆
```
USE mysql;
UPDATE `user` SET user.host='%' WHERE user.user='root';
FLUSH PRIVILEGES;
```
18. 创建用户
```
CREATE USER 'test'@'%' IDENTIFIED WITH mysql_native_password BY '29dIg;2^';
GRANT ALL ON *.* TO 'test'@'%'; -- 授权
REVOKE ALL ON *.* FROM 'test'@'%'; -- 撤销授权
```

数据库命令查询
--------------------------

```
# 查看dba连接的方式
\s

# 查询sql语句后面加\G，格式化显示结果
SHOW GLOBAL VARIABLES LIKE '%ssl%'; -- 查看ssl是否开启
SHOW VARIABLES WHERE Variable_name LIKE 'character_set_%' OR Variable_name LIKE 'collation%'; -- 查看字符集
SHOW VARIABLES LIKE '%password%'; -- 查看密码策略信息
# 查询mysql最大连接数设置
SHOW GLOBAL VARIABLES LIKE 'max_conn%';
SELECT @@MAX_CONNECTIONS AS 'Max Connections';
# 查看最大链接数
SHOW GLOBAL STATUS LIKE 'Max_used_connections';
# 查看慢查询日志是否开启以及日志位置
SHOW VARIABLES LIKE 'slow_query%';
# 查看慢查询日志超时记录时间
SHOW VARIABLES LIKE 'long_query_time';
# 查看链接创建以及现在正在链接数
SHOW STATUS LIKE 'Threads%';
# 查看数据库当前链接
SHOW PROCESSLIST;
# 查看数据库配置
SHOW VARIABLES LIKE '%quer%';
# 查看二进制日志是否已开启
SHOW VARIABLES LIKE 'log_%';
# 查看所有binlog日志列表
SHOW MASTER LOGS;

数据库连接参数中:
characterEncoding=utf8会被自动识别为utf8mb4，也可以不加这个参数，会自动检测。
而autoReconnect=true是必须加上的。
java连接8.0及以上MySQL数据库使用新驱动 mysql-connector-java-5.1.46s
```

windows之mysql进程
-----------------------

```
tasklist | findstr "mysql"
windows杀掉mysql进程
taskkill /f /t /im mysqld.exe
```