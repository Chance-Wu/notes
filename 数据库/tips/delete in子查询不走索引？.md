>delete in 子查询，是否会走索引？

#### 问题复现

---

假设当前有两张表`account`和`old_account`,表结构如下：

```sql
CREATE TABLE `old_account` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键Id',
  `name` varchar(255) DEFAULT NULL COMMENT '账户名',
  `balance` int(11) DEFAULT NULL COMMENT '余额',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_name` (`name`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1570068 DEFAULT CHARSET=utf8 ROW_FORMAT=REDUNDANT COMMENT='老的账户表';
 
CREATE TABLE `account` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键Id',
  `name` varchar(255) DEFAULT NULL COMMENT '账户名',
  `balance` int(11) DEFAULT NULL COMMENT '余额',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  `update_time` datetime NOT NULL ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_name` (`name`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=1570068 DEFAULT CHARSET=utf8 ROW_FORMAT=REDUNDANT COMMENT='账户表';
```

执行如下SQL：

```sql
delete from account where name in (select name from old_account);
```

explain执行计划：

```sql
explain delete from account where name in (select name from old_account);
```

| id   | select_type        | table       | partitions | type               | possible_keys | key      | key_len | ref  | rows | filtered  | Extra           |
| ---- | ------------------ | ----------- | ---------- | ------------------ | ------------- | -------- | ------- | ---- | ---- | --------- | --------------- |
| 1    | DELETE             | account     |            | **ALL**            |               |          |         |      | 3    | **100.0** | **Using where** |
| 2    | DEPENDENT SUBQUERY | old_account |            | **index_subquery** | idx_name      | idx_name | 768     | func | 1    | **100.0** | **Using index** |

可以发现Type为ALL，**全表s扫描account**，然后逐行执行子查询判断条件是否满足；显然，这个执行计划和我们预期不符，因为并没有走索引。



#### 优化写法——join

---

实际上，对于update或者delete子查询的语句，MySQL官网是推荐join的方式优化。

```sql
explain delete a from account a inner join old_account b on a.name=b.name;
```

| id   | select_type | table | partitions | type  | possible_keys | key      | key_len | ref         | rows | filtered | Extra                    |
| ---- | ----------- | ----- | ---------- | ----- | ------------- | -------- | ------- | ----------- | ---- | -------- | ------------------------ |
| 1    | SIMPLE      | b     |            | index | idx_name      | idx_name | 768     |             | 2    | 100.0    | Using where; Using index |
| 1    | DELETE      | a     |            | ref   | idx_name      | idx_name | 768     | test.b.name | 1    | 100.0    |                          |



#### 优化写法——起别名

---

```sql
explain delete a from account a where a.name in (select name from old_account);
```

| id   | select_type | table       | partitions | type  | possible_keys | key      | key_len | ref                   | rows | filtered | Extra                               |
| ---- | ----------- | ----------- | ---------- | ----- | ------------- | -------- | ------- | --------------------- | ---- | -------- | ----------------------------------- |
| 1    | SIMPLE      | old_account |            | index | idx_name      | idx_name | 768     |                       | 2    | 100.0    | Using where; Using index; LooseScan |
| 1    | DELETE      | a           |            | ref   | idx_name      | idx_name | 768     | test.old_account.name | 1    | 100.0    |                                     |

>为什么加个别名，delete in子查询又走索引了呢？
>
>查看explain的执行计划，可以发现Extra那一栏，有个**LooseScan**。
>
>加别名呢，会走**LooseScan策略**，而LooseScan策略，本质上就是**semi join子查询**的一种执行策略。



>show WARNINGS 可以查看优化后，最终执行的sql。
