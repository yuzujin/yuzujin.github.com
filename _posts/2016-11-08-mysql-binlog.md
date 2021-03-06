---
layout: post
title: "mysql binlog入门"
description: ""
category: [MySQL]
tags: [binlog]
---
{% include JB/setup %}

# Binlog

二进制日志（BINLOG），记录了所有的DDL（数据定义语言）语句和DML（数据操纵语言）语句，但是不包括数据查询语句。语句以“事件”的形式保存，它描述了数据的更改过程。此日志对于灾难时的数据恢复起着极其重要的作用。

## 日志的位置和格式

当用--log-bin[=file_name]选项启动时，mysqld开始将数据变更情况写入日志文件。如果没有给出filename值，默认名为主机名后面跟“-bin”。如果给出了文件名，但没有包含路径，则文件默认被写入参数DATADIR（数据目录）指定的目录（重启mysql一次将会自动生成一个新的binlog）。

开启状态的查看:

```
mysql> show variables like '%log_bin%';
+---------------------------------+-------+
| Variable_name                   | Value |
+---------------------------------+-------+
| log_bin                         | ON    |
| log_bin_trust_function_creators | OFF   |
| log_bin_trust_routine_creators  | OFF   |
| sql_log_bin                     | ON    |
+---------------------------------+-------+
4 rows in set (0.00 sec)
```

MySQL 5.5中，二进制日志的格式分为3种：STATEMENT、ROW、MIXED，可以在启动时通过参数--binlog_format进行设置，这三种日志格式区别如下：

`Statement`：每一条会修改数据的sql都会记录在binlog中。

优点：不需要记录每一行的变化，减少了binlog日志量，节约了IO，提高性能。(相比row能节约多少性能与日志量，这个取决于应用的SQL情况，正常同一条记录修改或者插入，row格式所产生的日志量还小于Statement产生的日志量，但是考虑到如果带条件的update操作，以及整表删除，alter表等操作，ROW格式会产生大量日志，因此在考虑是否使用ROW格式日志时应该跟据应用的实际情况，其所产生的日志量会增加多少，以及带来的IO性能问题。)

缺点：由于记录的只是执行语句，为了这些语句能在slave上正确运行，因此还必须记录每条语句在执行的时候的一些相关信息，以保证所有语句能在slave得到和在master端执行时候相同 的结果。另外mysql 的复制,像一些特定函数功能，slave可与master上要保持一致会有很多相关问题(如sleep()函数， last_insert_id()，以及user-defined functions(udf)会出现问题).

使用以下函数的语句也无法被复制：

* LOAD_FILE()

* UUID()

* USER()

* FOUND_ROWS()

* SYSDATE() (除非启动时启用了 --sysdate-is-now 选项)

同时在INSERT ...SELECT 会产生比 RBR 更多的行级锁

`Row`：不记录sql语句上下文相关信息，仅保存哪条记录被修改。

优点： binlog中可以不记录执行的sql语句的上下文相关的信息，仅需要记录那一条记录被修改成什么了。所以rowlevel的日志内容会非常清楚的记录下每一行数据修改的细节。而且不会出现某些特定情况下的存储过程，或function，以及trigger的调用和触发无法被正确复制的问题

缺点:所有的执行的语句当记录到日志中的时候，都将以每行记录的修改来记录，这样可能会产生大量的日志内容,比如一条update语句，修改多条记录，则binlog中每一条修改都会有记录，这样造成binlog日志量会很大，特别是当执行alter table之类的语句的时候，由于表结构修改，每条记录都发生改变，那么该表每一条记录都会记录到日志中。

`Mixedlevel`： 是以上两种level的混合使用，一般的语句修改使用statment格式保存binlog，如一些函数，statement无法完成主从复制的操作，则采用row格式保存binlog,MySQL会根据执行的每一条具体的sql语句来区分对待记录的日志形式，也就是在Statement和Row之间选择一种.新版本的MySQL中队row level模式也被做了优化，并不是所有的修改都会以row level来记录，像遇到表结构变更的时候就会以statement模式来记录。至于update或者delete等修改数据的语句，还是会记录所有行的变更。



## 二进制日志的管理

**开启二进制日志配置**

方法一、修改my.cnf参数文件，该方法需要重启

log-bin = mysql-bin #打开日志(主机需要打开)，这个mysql-bin也可以自定义，这里也可以加上路径，如：/home/www/mysql_bin_log/mysql-bin

关闭二进制日志的方法：log-bin = mysql-bin注释掉即可
 

方法二、不重启修改二进制日志配置，该方法mysql的版本需要5.6以上

SET @@global.log_bin=1|0  （1为开启，0为关闭）

SET @@global.binlog_size=37268(单位bytes)

**暂停二进制日志**

SET sql_log_bin={0|1}

**修改二进制日志的大小**

修改my.cnf参数文件中的max_binlog_size的值；

说明：如果你的二进制文件的大小超过了max_binlog_size，它就是自动创建新的二进制文件。当然如果恰好在日志文件到达它的最大尺寸时写入了大的事务，那么日志文件还是会超过max_binlog_size的大小

**进行二进制日志的切换**

默认情况下当二进制日志写满了或者数据库重启了才会进行切换，但是也可以手工的进行切换的动作

mysql> flush logs;

**其他参数**

binlog-cache-size=100m 设置二进制日志缓存大小

sync-binlog=N（每个N秒将缓存中的二进制日志记录写回硬盘，默认值为0。不过，你经常会陷入group commit函数与I/O之间二选一的矛盾。如果在replication环境中，由于考虑到耐久性与一致性，则需要设置1。同时，还需要设置innodb_flush_log_at_trx_commit=1以及innodb-support-xa=1默认开启；）

**二进制日志的读取**

由于日志以二进制方式存储，不能直接读取，需要用mysqlbinlog工具来查看，语法如下：

```
shell> mysqlbinlog log-file
```

## 二进制日志的删除

对于比较繁忙的OLTP（在线事务处理）系统，由于每天生成日志量大，这些日志如果长时间不清除，将对磁盘空间带来很大的浪费。因此，定期删除日志是DBA维护MYSQL数据库的一个重要工作内容。下面将介绍几种删除日志的常见方法。

**方法一**

执行RESET MASTER命令，该命令将删除所有BINLOG日志，新日志编号从“000001”开始，hostname-bin.index是日志的索引文件，记录了最大的日志序号。

**方法二**

执行PURGE MASTER LOGS TO "mysql-bin.******"命令，该命令将删除“******”编号之前的所有日志。

**方法三**

执行PURGE MASTER LOGS BEFORE 'yyyy-mm-dd hh24:mi:ss'命令，该命令将删除日期为“yyyy-mm-dd hh24:mi:ss”之前产生的所有日志。

**方法四**

在my.cnf的[mysqld]中设置参数--expire_logs_days=#，此参数的含义是设置日志的过期天数，过了指定的天数后日志将会被自动删除，这样将有利于减少DBA管理日志的工作量。
