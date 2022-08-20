#### 1. 一般的分页查询

使用简单的limit子句就可以实现。

```sql
SELECT * FROM table LIMIT [offset,] rows | rows OFFSET offset
```

- 第一个参数指定第一个返回记录行的偏移量。

- 第二个参数指定返回记录行的最大数目。（为-1时，表示检索从某一个偏移量到记录集的结束所有的记录行）

如果只给定一个参数，表示返回最大的记录行数目。

示例：

```sql
select * from orders_history where type=8 limit 1000,10;
-- 查询第1000条数据之后的10条数据，也就是第1001条到第10010条数据。
```

- 在查询记录量低于`100`时，查询时间基本没有差距，随着查询记录量越来越大，所花费的时间也会越来越多。
- 随着查询偏移的增大，尤其查询偏移大于`10万`以后，查询时间急剧增加。

>注意：这种方式会从数据库第一条记录开始扫描，所以越往后，查询速度越慢，而且查询的数据越多，也会拖慢查询查询速度。

#### 2. 使用子查询优化

这种方式先定位偏移位置的 id，然后往后查询，这种方式适用于 id 递增的情况。

```sql
select * from orders_history where type=8 limit 100000,1;
select id from orders_history where type=8 limit 100000,1; 快
select * from orders_history where type=8 and
id>=(select id from orders_history where type=8 limit 100000,1)
limit 100;快
```

#### 3. 使用id限定优化

这种方式假设数据表的id是连续递增的，则我们根据查询的页数和查询的记录数可以算出查询的id的范围，可以使用 id between and 来查询：

```sql
select * from orders_history where type=2 and id between 1000000 and 1000100 limit 100;
```

这种查询方式能够极大地优化查询速度，基本能够在几十毫秒之内完成。限制是只能使用于明确知道id的情况，不过一般建立表的时候，都会添加基本的id字段，这为分页查询带来很多便利。

#### 4. 关于数据表的id说明

一般情况下，在数据库中建立表的时候，强制为每一张表添加 id 递增字段，这样方便查询。

如果像是订单库等数据量非常庞大，一般会进行分库分表。这个时候不建议使用数据库的 id 作为唯一标识，而应该使用分布式的高并发唯一 id 生成器来生成，并在数据表中使用另外的字段来存储这个唯一标识。

使用先使用范围查询定位 id (或者索引)，然后再使用索引进行定位数据，能够提高好几倍查询速度。即先 select id，然后再 select *；