如果需要将两个select语句的结果作为一个整体显示出来，我们就需要用到union或者union all关键字。union(或称为联合)的作用是**将多个结果合并在一起显示出来**。

UNION 操作符用于合并两个或多个 SELECT 语句的结果集。

**UNION 内部的每个 SELECT 语句必须拥有相同数量的列**。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。

区别：

- Union：对两个结果集进行并集操作，不包括重复行，同时进行默认规则的排序；
- Union All：对两个结果集进行并集操作，包括重复行，不进行排序；
- Intersect：对两个结果集进行交集操作，不包括重复行，同时进行默认规则的排序；
- Minus：对两个结果集进行差操作，不包括重复行，同时进行默认规则的排序。



### 一、语法

---

UNION 语法：

```sql
SELECT column_name(s) FROM table1
UNION
```

注释：默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL。

UNION ALL 语法：

```sql
SELECT column_name(s) FROM table1
UNION ALL
SELECT column_name(s) FROM table2;
```



### 二、示例

---

下面是选自 “Websites” 表的数据：

mysql> SELECT * FROM Websites;

+----+--------------+---------------------------+-------+---------+

| id | name         | url                       | alexa | country |

+----+--------------+---------------------------+-------+---------+

| 1  | Google       | https://www.google.cm/    | 1     | USA     |

| 2  | 淘宝          | https://www.taobao.com/   | 13    | CN      |

| 3  | 菜鸟教程      | http://www.runoob.com/    | 4689  | CN      |

| 4  | 微博          | http://weibo.com/         | 20    | CN      |

| 5  | Facebook     | https://www.facebook.com/ | 3     | USA     |

| 7  | stackoverflow | http://stackoverflow.com/ |   0 | IND     |

+----+---------------+---------------------------+-------+---------+



下面是 “APPs” APP 的数据：



mysql> SELECT * FROM APPs;

+----+------------+-------------------------+---------+

| id | APP_name   | url                     | country |

+----+------------+-------------------------+---------+

|  1 | QQ APP     | http://im.qq.com/       | CN      |

|  2 | 微博 APP | http://weibo.com/       | CN      |

|  3 | 淘宝 APP | https://www.taobao.com/ | CN      |

+----+------------+-------------------------+---------+

3 rows in set (0.00 sec)



SQL UNION 实例

下面的 SQL 语句从 “Websites” 和 “APPs” 表中选取所有不同的country（只有不同的值）：



实例



SELECT country FROM Websites

UNION

SELECT country FROM APPs

ORDER BY country;



执行以上 SQL 输出结果如下：



注释：UNION 不能用于列出两个表中所有的country。如果一些网站和APP来自同一个国家，每个国家只会列出一次。UNION 只会选取不同的值。请使用 UNION ALL 来选取重复的值！



SQL UNION ALL 实例

下面的 SQL 语句使用 UNION ALL 从 “Websites” 和 “APPs” 表中选取所有的country（也有重复的值）：

实例



SELECT country FROM Websites

UNION ALL

SELECT country FROM APPs

ORDER BY country;



执行以上 SQL 输出结果如下：

带有 WHERE 的 SQL UNION ALL

下面的 SQL 语句使用 UNION ALL 从 “Websites” 和 “APPs” 表中选取所有的中国(CN)的数据（也有重复的值）：



实例

SELECT country, name FROM Websites

WHERE country='CN'

UNION ALL

SELECT country, APP_name FROM APPs

WHERE country='CN'

ORDER BY country;

执行以上 SQL 输出结果如下：

注释：UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。



### 三、注意要点

---

union和union all关键字需要注意：

1. union 和 union all都可以将多个结果集合并，而不仅仅是两个，你可以将多个结果集串起来。
2. 使用union和union all必须保证各个select 集合的结果有相同个数的列，并且每个列的类型是一样的。但列名则不一定需要相同，oracle会将第一个结果的列名作为结果集的列名。