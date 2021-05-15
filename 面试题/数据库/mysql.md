#### 1. MySQL如何解决需要区分英文大小写的场景

---

例如，登录用户名为admin，此时填写ADMIN也能登录。

>解决方案一：
>
>mysql默认字符检索策略：`utf8_general_ci`，表示不区分大小写。
>
>可以使用`utf8_bin`，表示二进制比较，同样区分大小写。
>
>```sql
>-- 修改表结构的Collation属性
>ALTER TABLE TABLENAME MODIFY COLUMN COLUMNNAME VARCHAR(50) BINARY CHARACTER SET utf8 COLLATE utf8_bin DEFAULT NULL;
>```

>解决方案二：
>
>修改sql语句，在要查询的字段前面加上`binary关键字`。
>
>```sql
>-- 在每一个条件前加上binary关键字
>select * from user where binary username = 'admin' and binary password = 'admin';
>
>-- 将参数以binary('')包围
>select * from user where username like binary('admin') and password like binary('admin');
>```



#### 2. innodb的事务与日志的实现

---

>日志：
>
>- 错误日志
>- 查询日志
>- **慢查询日志**：设置一个阈值，将运行时间超过该值的所有SQL语句都记录到慢查询的日志文件中（s`how variables like '%slow_query_log%'`）
>- 二进制日志：记录对数据库执行更改的所有操作
>- 中继日志：也是二进制日志，用来给slave库恢复
>- **事务日志**：==重做日志redo==和==回滚日志undo==

>**事务是如何通过日志来实现的？**
>
>事务日志是通过redo和innodb的存储引擎日志缓冲来实现的。
>
>- 当开始一个事务的时候，会记录该事务的lsn（日志序列号）；
>- 事务执行时，会往innodb存储引擎的日志缓存里面插入事务日志；
>- 事务提交时，必须将存储引擎的日志缓冲写入磁盘（通过`innodb_flush_log_at_trx_commit`来控制），也就是写数据前，需要先写日志。这种方式称为“预写日志方式”。



#### 3. MySQL binlog的几种日志录入格式以及区别

---

>`Statement`：每一条会修改数据的sql都会记录在binlog中。

>`Row`：不记录sql语句上下文相关信息，仅保存哪条记录被修改。

>`Mixedlevel`：以上两种level的混合使用。

