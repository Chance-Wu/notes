全连接就是左连接和右连接的并集，但是MySQL中并不支持全连接的写法。

不过可以**用UNION联合左连接和右连接的结果**来代替。

```sql
SELECT
	t1.id,
	t2.id 
FROM
	t1
	LEFT JOIN t2 ON t1.person = t2.person UNION
SELECT
	t1.id,
	t2.id 
FROM
	t1
	RIGHT JOIN t2 ON t1.person = t2.person
```

| id   | id(1) |
| ---- | ----- |
| 1    | A     |
| 2    | B     |
| 3    | NULL  |
| NULL | C     |