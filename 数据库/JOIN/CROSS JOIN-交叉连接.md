![image-20220519005035198](CROSS%20JOIN.assets/image-20220519005035198.png)

![image-20220519005112806](CROSS%20JOIN.assets/image-20220519005112806-2892674.png)

笛卡尔积（交叉连接）就是将表1的每条记录与表2中的每一条记录拼成数据对，`CROSS JOIN`的SQL执行语句如下：

```sql
SELECT
	t1.id,
	t2.id 
FROM
	t1
	CROSS JOIN t2;
```

执行结果如下：

| id   | id(1) |
| ---- | ----- |
| 1    | A     |
| 2    | A     |
| 3    | A     |
| 1    | B     |
| 2    | B     |
| 3    | B     |
| 1    | C     |
| 2    | C     |
| 3    | C     |

