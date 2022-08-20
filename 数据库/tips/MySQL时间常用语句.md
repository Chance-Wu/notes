### 一、常用时间函数

---

#### to_days(date) 

计算日期 d 距离 0000 年 1 月 1 日的天数。

```sql
SELECT TO_DAYS('0001-01-01 01:01:01');
-> 366
```



#### now()

返回当前日期和时间。

```sql
select now();
-> 2022-07-19 00:19:28
```



#### date_sub(date, interval expr type)

从日期减去指定的时间间隔。

```sql
select date_sub(now() , interval 2 day) as date_time;
-> 2022-07-17 00:28:25（当前时间-2天）
```



#### curdate()

返回当前日期。

```sql
select curdate();
-> 2022-07-19
```



#### date_format(d ,f)

按表达式 f 的要求显示日期 d，

```sql
select date_format('2022-07-19 00:01:01', '%y%y-%m-%d')
-> 2222-07-19
```



#### period_diff(period1, period2)

返回两个时段之间的月份差值。

```sql
select period_diff( date_format( now( ) , '%y%m' ) , date_format( '2022-01-01 00:00:01', '%y%m' ) );
-> 6
```



#### quarter(d)

返回日期d是第几季节，返回1到4。

```sql
select quarter('2022-07-19 00:00:01');
-> 3
```



#### year(d)

返回年份。

```sql
select year('2022-07-19');
-> 2022
```



#### yearweek(date, mode)

返回年份及第几周（0到53），mode中0表示周天，1表示周一，以此类推。

```sql
select yearweek("2022-07-19");
-> 202229
```

















### 二、常用时间语句

---

#### 今天

```sql
select
	*
from
	表名
where
	to_days( now()) = to_days(时间字段名);
```



#### 昨天

---

```sql
select * from 表名 where to_days( now( ) ) - to_days( 时间字段名 ) <= 1;
```



#### 近7天

---

```sql
select
	*
from
	表名
where
	date_sub(curdate(), interval 7 day) <= date(时间字段名);
```



#### 近30天

---

```sql
select
	*
from
	表名
where
	date_sub(curdate(), interval 30 day) <= date(时间字段名);
```



#### 本月

---

```sql
select
	*
from
	表名
where
	date_format( 时间字段名, '%y%m' ) = date_format( curdate( ) , '%y%m' );
```



#### 上一月

---

```sql
select
	*
from
	表名
where
	period_diff( date_format( now( ) , '%y%m' ) , date_format( 时间字段名, '%y%m' ) ) = 1;
```



#### 查询本季度数据

```sql
select
	*
from
	表名
where
	quarter(create_date)= quarter(now());
```



#### 查询本年数据

```sql
select
	*
from
	表名
where
	year(create_date)= year(now());
```



#### 查询上年数据

```sql
select
	*
from
	表名
where
	year(时间字段名)= year(date_sub(now(), interval 1 year));
```



#### 查询当前这周的数据

```sql
select
	*
from
	表名
where
	yearweek(date_format(时间字段, '%y-%m-%d')) = yearweek(now());
```



#### 查询上周的数据

```sql
select
	*
from
	表名
where
	yearweek(date_format(时间字段名, '%y-%m-%d')) = yearweek(now())-1;
```



#### 查询距离当前现在6个月的数据

```sql
select
	*
from
	表名
where
	时间字段名 between date_sub(now(), interval 6 month) and now();
```