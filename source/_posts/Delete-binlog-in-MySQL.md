---
title: 删除/清理 MySQL 的 binlog
date: 2020-12-14 16:43:16
tags: [MySQL]
thumbnail: https://image.blog.chaosjohn.com/Delete-binlog-in-MySQL/banner.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Delete-binlog-in-MySQL.html)

## 背景
今年年中的时候，我司和腾讯达成了一项深度合作，涉及到的数据量还挺大的。果不其然，谈下合作后负责实施的还是我这个“什么都干司令”。

具体流程：

1. 从我司已有产品中，筛出一部分数据
2. 数据清理
3. 处理成通用格式（因为老板说后续不仅仅只跟腾讯合作）
4. 输出给腾讯（实时处理成腾讯方需要的格式）并且记录输出时间在本地

因为数据清理+处理比较费时间和空间，所以本地临时搞了一台高性能台式机作为服务器（是的，本“什么都干司令”还负责组装电脑），配置：

- CPU: AMD Ryzen 5 3600
- MEM: 32GB 
- SSD: 1TB

## 问题
还处在数据清理和处理的流程中，突然发现程序脚本异常退出了。

通过排查发现，1TB的磁盘，居然满了，定位到 `/var/lib/mysql` 路径下，发现了大量的 `binlog.xxxxxxx` 文件，通过 `ncdu` 分析出它们占据了大几百GB的磁盘空间。

（当时处理问题之前没有截图，所以只能截几张写本文时的图了，而写本文时，合作已经处于最后的输出流程了，所以数据库 `写操作` 只与 `记录输出时间` 相关，数据量小了很多）

![/var/lib/mysql 下大量的 binlog 文件](https://image.blog.chaosjohn.com/Delete-binlog-in-MySQL/binlog-files.png)

![ncdu 分析 binlog 文件占据磁盘空间](https://image.blog.chaosjohn.com/Delete-binlog-in-MySQL/ncdu.png)

## 原因
`binlog` 是记录所有数据库表结构变更（例如CREATE、ALTER TABLE…）以及表数据修改（INSERT、UPDATE、DELETE…）的二进制日志。

`binlog` 不会记录SELECT和SHOW这类操作，因为这类操作对数据本身并没有修改，但你可以通过查询通用日志来查看MySQL执行过的所有语句。

二进制日志包括两类文件：二进制日志索引文件（文件名后缀为 `.index`）用于记录所有的二进制文件，二进制日志文件（文件名后缀为 `.00000*`）记录数据库所有的DDL和DML(除了数据查询语句)语句事件。

所以简单点描述，数据库 `写操作` 越多越频繁，`binlog` 记录的越多，甚至出现笔者遇到的 `爆炸增长`。

## 解决
既然 `binlog` 是日志，只要确保（或者“假装确保”）过去的操作都正确，咱也没有 `从 binlog 恢复数据` 的需求，那就删了呗。

### 手动删除
直接在 `/var/lib/mysql` 路径下，将 `binlog.0*` 删除掉（注意不要删除 `binlog.index`）

该方法不推荐，因为手动删除并不会更新 `binlog.index`，而 `binlog.index` 的作用是加快查找 `binlog` 文件的速度。

### 删除指定编号之前的 binlog
```
mysql> PURGE MASTER LOGS TO 'binlog.000860';
Query OK, 0 rows affected (0.01 sec)
```
![指定编号之前的 binlog 已被删除](https://image.blog.chaosjohn.com/Delete-binlog-in-MySQL/binlog-files-after-purge-to-specified-index.png)

### 删除指定日期之前的 binlog
```
mysql> PURGE MASTER LOGS BEFORE '2020-11-11 11:11:11';
Query OK, 0 rows affected (0.19 sec)
```
![指定日期之前的 binlog 已被删除](https://image.blog.chaosjohn.com/Delete-binlog-in-MySQL/binlog-files-after-purge-before-datetime.png)

### 清空所有 binlog
```
mysql> RESET MASTER;
Query OK, 0 rows affected (0.09 sec)
```
![清空后 binlog 编号从 000001 开始](https://image.blog.chaosjohn.com/Delete-binlog-in-MySQL/binlog-files-after-reset.png)

### 配置自动清理
```
mysql> set global expire_logs_days=7;
```
则只保留7天内的 `binlog` 文件。

或者修改 `MySQL` 配置文件，设置 `expire_logs_days=7` 然后重启服务，即可永久生效。

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)