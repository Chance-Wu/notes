> 常见的方式为字段设置**主键**或**唯一索引**，当插入重复数据时，抛出错误，程序终止，但这会给后续处理带来麻烦，因此需要对插入语句做特殊处理，尽量避开或忽略异常。



#### insert ignore into

---

插入数据时，**如果数据存在，则忽略此次插入**，前提是插入的数据字段设置了主键或唯一索引。

使用`insert ignore into`语句时，如果主键冲突，只是提示"warnings"。

>复制表的时候可以避免重复记录
>
>`insert ignore into t1 select name from t2;`



#### on duplicate key update

---

插入数据时，**如果数据存在，则执行更新操作**，前提是插入的数据字段设置了主键或唯一索引，测试SQL语句如下，当插入本条记录时，MySQL数据库会首先检索已有数据，如果存在，则执行update更新操作，如果不存在，则直接插入。



#### replace into

---

插入数据时，如果数据存在，则**删除再插入**，前提条件是插入的数据字段需要设置主键或唯一索引，当插入本条记录时，MySQL数据库会首先检索已有数据，如果存在，则先删除旧数据，然后再插入，如果不存在，则直接插入。

```sql
replace into table_name(col_name, ...) values(...) 
replace into table_name(col_name, ...) select ... 
replace into table_name set col_name=value, ...
```

REPLACE的运行与INSERT很相像,但是如果旧记录与新记录有相同的值，则在新记录被插入之前，旧记录被删除。旧记录与新记录有相同的值的判断标准就是：表有一个PRIMARY KEY或UNIQUE索引，否则，使用一个REPLACE语句没有意义。



#### insert if not exists

---

即`insert into … select … where not exist …` ，这种方式适合于插入的数据字段没有设置主键或唯一索引，当插入一条数据时，首先判断MySQL数据库中是否存在这条数据，如果不存在，则正常插入，如果存在，则忽略。

