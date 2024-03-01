---
title: 文件
tags: [mysql, file]
sidebar_label: File
sidebar_position: 3
---

# 文件

## 参数文件

参数文件用于存放MySQL的配置参数和运行时参数。

## 日志文件

日志文件记录了影响MySQL数据库的各种类型活动。

### 错误日志

错误日志文件对MySQL的启动、运行、关闭过程进行了记录。用户可以用过命令 `SHOW VARIALBES LIKE 'log_error'` 来定位该文件。

### 慢查询日志

慢查询日志可以帮助DBA定位可能存在问题的SQL语句，从而优化SQL语句。可以用过下面的命令查看具体的参数：

* `log_slow_queries`：是否开启慢查询日志记录。
* `long_query_time`：慢查询日志记录的时间阈值，默认为10。
* `log_queries_not_using_indexes`：是否记录没有使用索引的查询。
* `log_throttle_queries_not_using_indexes`：每分钟允许记录慢查询的且未使用索引的SQL语句次数，默认值为0。
* `long_query_io`：将超过指定逻辑IO次数的SQL语句记录到日志文件中，默认值100。
* `slow_query_type`：0-不记录，1-按时间记录，2-按逻辑次数记录，3-按运行时间&逻辑IO次数记录。

DBA 可以使用 `mysqldumpslow file.log` 命令来查看慢查询的日志内容。

### 查询日志

查询日志记录了所有对MySQL数据库查询的信息。

### 二进制日志

二进制文件（binary log）记录了对MySQL数据库执行更改的所有操作，所以也不包括SELECT和SHOW这类操作。该文件有以下作用：

1. 恢复：可以根据日志对数据进行恢复。用户可以使用日志来进行point-in-time恢复。
2. 复制：类似于恢复，可使用日志进行主从复制。
3. 审计：记录可用于审计。

二进制日志文件默认并没有开启，需要手动指定，开启这个选项会使性能下降1%，十分有限，但可以使用复制和point-in-time恢复。MySQL提供了mysqlbinlog工具来查看日志内容。

二进制日志格式（binlog_format参数）:

1. STATEMENT：记录逻辑SQL语句。在某些情况下会存在不一致问题，例如随机不确定函数。
2. ROW：记录表的行更改。恢复和复制更加可靠，但是可能会导致日志文件过大。
3. MIXED：大部分情况使用STATEMENT格式，某些情况下使用ROW格式，例如使用了UUID()、USER()等不确定函数，INSERT DELAY语句，使用了UDF，使用了临时表等。

## 套接字文件

在UNIX系统下本地连接MySQL可以采用UNIX域套接字方式，套接字文件由参数 `socket` 控制，一般在 `/tmp/mysql.sock` 位置。

## pid文件

MySQL实例启动时，会有一个pid文件，记录自己的进程ID，文件位置一般由参数 `pid_file` 控制，一般位于数据库目录下。

## 表结构定义文件

每张表都有一个以frm为后缀名的文件，用于定义表的结构。frm文件还可以用于存放视图的定义。

## InnoDB存储引擎文件

### 表空间文件

InnoDB采用将存储的数据按表空间进行存放的设计。

默认配置下会有一个初始大小为10MB名为ibdata1的文件，路径可通过 `innodb_data_file_path` 参数进行配置。所有的表都会记录到该共享表空间。

当然，也可以通过设置 `innodb_file_per_table` 参数来为每张表创建一个独立的表空间，文件一般为：表名.ibd。值得注意的是，表空间文件仅存储该表的数据、索引和插入缓存BITMAP等信息，其余信息还是存放在默认的表空间中。

### 重做日志文件

重做日志文件用来实现事务的某些特性。

一般，在数据目录下默认有两个名为ib_logfile0和iblogfile1的重做日志文件，重做日志文件会在这两个文件之间来回写，当然也可以通过修改 `innodb_log_file_in_group` 参数来控制文件个数。`innodb_log_file_size` 参数可以控制文件的大小。

# 参考资料

* 《MySQL技术内幕 - InnoDB存储引擎》