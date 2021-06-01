# RedoLog 和 BinLog

## 1.RedoLog 重做日志

为了减少数据更新多次访问数据库，MySQL 中对于数据更新，采用 Write-Ahead Logging 策略，即先写日志，再写磁盘。

具体的来说，当一条数据需要更新的时候，InnoDB 会先把记录写入到 RedoLog 里，并更新内存，然后在适当的时候讲操作记录更新到磁盘里。这个更新一般发生在比较空闲的时候。

InnoDB 的 RedoLog 是一组固定大小的文件，且数据的写入是循环写入，即当文件写满后，又会回到文件开头擦除数据并继续写，就像一个闭环。

擦除的数据指针是要先于写入的数据指针的。

使用 RedoLog 可以有效避免数据库发生异常重启后，之前提交的记录都不会丢失。

`RedoLog 是 InnoDB 特有的日志`

## 2.BinLog 归档日志

BinLog 日志是 MySQL 服务层提供日志

和 RedoLog 相比BinLog 有以下几个区别

1. RedoLog 是物理日志，记录的是在某个数据上做了什么修改；BinLog 是逻辑日志，记录的是这个语句的原始含义
2. RedoLog 是循环写；BinLog 是可追加写入，当 BinLog 文件写入到一定大小后会切换到下一个



**RedoLog 写在前，BinLog 写在后**

两段式提交

当 在写 binlog 前down 掉，重启后 redolog 会继续事务然后写 BinLgo 提交

redolog 只是完成了 prepare

但是写 binlog 后才会提交