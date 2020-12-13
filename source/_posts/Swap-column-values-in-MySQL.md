---
title: MySQL 交换两列值
date: 2020-12-13 20:16:41
tags: [MySQL]
thumbnail: https://image.blog.chaosjohn.com/Swap-column-values-in-MySQL/banner.png
---

[欢迎转载，但请在开头或结尾注明原文出处【blog.chaosjohn.com】](https://blog.chaosjohn.com/Swap-column-values-in-MySQL.html)

## 前言
今年五月下旬的时候，公司某线上数据库遭遇表被删事件，对，没错，就是那种 `删库` 事件。

不过不是恶意删库事件，是某开发童鞋的不小心，而且他也没有跑路。

在发现表被删的第一时间，我就插手处理（假设表名为 `sample`）：

1. 先把被删的表结构重建起来，先争取线上相关业务接口不再报 `502` 错误
2. 再从阿里云那边下载当日早些时候的完整数据库备份
3. 创建本地 MySQL 环境，将被删表从完整备份中恢复到本地
4. 等到业务高峰过去后，短暂下线 `sample` 表相关的服务，即 `服务降级`
5. 将线上 `sample` 表的数据导出到 `CSV` 文件（包含了从删库后重建表开始到服务降级之间的所有数据）
6. 从 `CSV` 中合并增量数据到本地 `sample` 表（追加在尾部）
7. 将线上 `sample` 表备份后删除，将本地 `sample` 表复制到线上
8. 恢复服务

完美收工！！！

只不过第二天做投放的小伙伴告诉我，后台数据有错乱。我一检查，发现上述的 `步骤6` 出了纰漏。合并增量数据的时候，`CSV` 的列与 `sample` 表的列没对齐，即有两列交换了位置。

## 解决
所以，解决目标就是，对于数据表里出问题的行，要将该两列交换回来。

### 方案一
利用临时变量 `temp`，适用场景：`x` 与 `y` 必须都不为 `NULL`
```
UPDATE sample SET x=y, y=@temp WHERE (@temp:=x) IS NOT NULL;
```

### 方案二（笔者选用的方案）
同样利用临时变量 `temp`，但是 `x` 与 `y` 没有不为 `NULL` 的限制
```
UPDATE sample SET x=(@temp:=x), x=y, y=@temp;
```

### 方案三
`s1` 得到更新，而 `s2` 则用来拉取老数据（注：该方案要求表必须有主键`id`）
```
UPDATE sample s1, sample s2 SET s1.x=s1.y, s1.y=s2.x WHERE s1.id=s2.id;
```

### 方案四（如果 `x` 与 `y` 都是数值型）
```
UPDATE sample SET x=x+y,y=x-y,x=x-y;
```

---

最后，如果该文对读者有些许帮助，考虑下给点捐助鼓励一下呗😊
![](https://image.blog.chaosjohn.com/donate-me.png)