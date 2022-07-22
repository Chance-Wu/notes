清空表数据，建议直接使用 truncate，效率上truncate远高于delete，**truncate不走事务，不会锁表，也不会产生大量日志写入日志文件**，我们访问log执行日志可以发现每次delete都有记录。truncate table table_name 会**立刻释放磁盘空间**，并**重置auto_increment的值**，delete 删除不释放磁盘空间，insert会覆盖之前的数据上，因为我们创建表的时候有一个创建版本号。



首先看delete命令的参数信息。

```sql
delete [low_priority] [quick] [ignore] from tbl_name
[where ...]
[order by ...]
[limit row_count]
```

delete后面是可以跟limit关键词的，但仅支持单个参数，用于告诉服务器在控制命令被返回到客户端前被删除的行的最大值。

如果要用order by 必须要和 limit 联用，否则被优化掉。



对于delete limit 的使用，MySQL大佬丁奇有一道题：

如果你要删除一个表里面的前 10000 行数据，有以下三种方法可以做到：
第一种，直接执行 delete from T limit 10000;
第二种，在一个连接中循环执行 20 次 delete from T limit 500;
第三种，在 20 个连接中同时执行 delete from T limit 500。

肉山：

第一个方案，一次占用锁的时间比较长，可能导致其他客户端一致等待资源。
第二个方案，分多次占用锁，串行化执行，不占有锁的间隙，其他客户端可以工作，每次执行不同片段的数据，我理解为分段锁concurrentHashmap
第三个方案。自己制造锁竞争，加剧并发。可能锁住同一记录导致死锁的可能性增大。