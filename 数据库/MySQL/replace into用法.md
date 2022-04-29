在向表中插入数据的时候，经常遇到这样的情况：

1. 首先判断数据是否存在；
2. 如果不存在，则插入；
3. 如果存在，则更新。

> SQL Server写法：

```sql
if not exists (select 1 from table where id = 1) insert into table(id, update_time) values(1, getdate()) else update table set update_time = getdate() where id = 1
```

> 在MySQL 中也可以先select，判断是否存在，存在则 update 否则 insert。但在MySQL 中有更简单的方法，使用 **replace into** 关键字

```sql
replace into table(id, update_time) values(1, now());
```

或

```sql
replace into table(id, update_time) select 1, now();
```

或

```	sql
replace into table set id=1, update_time=now();
```

replace into 跟 insert 功能类似，不同点在于：replace into 首先尝试插入数据到表中。

1. 如果发现表中已经有此行数据（根据主键或者唯一索引判断）则先删除此行数据，然后插入新的数据。
2. 否则，直接插入新数据。



#### replace into使用注意事项

---

1. 插入数据的==表必须有主键或者是唯一索引==！否则的话，replace into 会直接插入数据，这将导致表中出现重复的数据。
2. 如果数据库里边有这条记录，则直接修改这条记录；如果没有则，则直接插入，在有外键的情况下，对主表进行这样操作时，因为如果主表存在一条记录，被从表所用时，直接使用replace into是会报错的，这和replace into的内部原理是相关（先删除然后再插入）。



#### 表中有一个自增主键带来的问题

---

replace操作在自增主键的情况下，遇到唯一键冲突时执行的是delete+insert,但是在记录binlog时,却记录成了update操作,update操作不会涉及到auto_increment的修改。备库应用了binlog之后，备库的表的auto_increment属性不变。如果主备库发生主从切换，备库变为原来的主库，写新的主库则有风险发生主键冲突

频繁的REPLACE INTO 会造成新纪录的主键的值迅速增大。总有一天。达到最大值后就会因为数据太大溢出了。就没法再插入新纪录了。数据表满了，不是因为空间不够了，而是因为主键的值没法再增加了。