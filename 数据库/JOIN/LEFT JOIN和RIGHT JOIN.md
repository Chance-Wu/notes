#### 1. LEFT JOIN

---

左连接的含义就是求两个表的交集外加左表剩下的数据。从笛卡尔积的角度讲，就是**先从笛卡尔积中挑出ON子句条件成立的记录，然后加上左表中剩余的记录**：

```sql
SELECT
	t1.id,
	t2.id 
FROM
	t1
	LEFT JOIN t2 ON t1.person = t2.person
```

执行结果：

| id    | id(1) |
| ----- | ----- |
| ==1== | ==A== |
| ==2== | ==B== |
| 3     | NULL  |



#### 2. RIGHT JOIN

---

右连接就是求两个表的交集外加右表剩下的数据。再次从笛卡尔积的角度描述，右连接就是**从笛卡尔积中挑出ON子句条件成立的记录，然后加上右表中剩余的记录**：

```sql
SELECT
	t1.id,
	t2.id 
FROM
	t1
	RIGHT JOIN t2 ON t1.person = t2.person
```

执行结果：

| id    | id(1) |
| ----- | ----- |
| ==1== | ==A== |
| ==2== | ==B== |
| NULL  | C     |

