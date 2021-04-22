>EXPLAIN SQL的输出如下：
>
>| id   | select type | table              | partitions | type  | possible_keys | key     | key_len | ref  | rows | filtered | Extra        |
>| ---- | ----------- | ------------------ | ---------- | ----- | ------------- | ------- | ------- | ---- | ---- | -------- | ------------ |
>| 1    | UPDATE      | member_import_file | Null       | index | Null          | PRIMARY | 8       | Null | 187  | 100      | Using  where |

#### 1. id（JSON名：select_id）

>select标识符，SQL执行的顺序的标识，SQL从大到小的执行：
>
>- id相同时，执行顺序由上至下
>- 如果是子查询，id的序号会递增，id值越大优先级越高，越先被执行
>- 如果id相同，则认为是一组，从上往下顺序执行；在索引组中，id值越大，优先级越高，越先被执行。

#### 2. select_type

>查询的类型，主要用来区别普通查询，联合查询，子查询等复杂查询。
>
>| select_type  | 描述                                                         |
>| ------------ | ------------------------------------------------------------ |
>| SIMPLE       | 简单的select查询                                             |
>| PRIMARY      | 查询中若包含任何复杂的子部分，最外层的查询则被标记为PRIMARY  |
>| SUBQUERY     | 在select或者where列表中包含了子查询                          |
>| DERIVED      | 在from列表中包含的子查询会被标记为DERICED(衍生)，Mysql会递归地执行这些子查询，然后把结果放到临时表 |
>| UNION        | 若第二个select语句出现在UNION之后，则被标记为UNION。若UNION包含在from子句的子查询中，外层select则被标记为DERIVED |
>| UNION RESULT | 从union表获取结果的select                                    |

#### 3. partitions（JSON名：partitions）

>记录与查询匹配的分区。值为NULL表示为费分区表。

#### 4. type（JSON名：access_type）

> 访问类型排序，显示查询使用了何种类型，从最好到最差依次是：
>
> `system>const>eq_ref>ref>range>index>ALL`
>
> | type      | 描述                                                         |
> | --------- | ------------------------------------------------------------ |
> | system    | 表只有一行记录，这是const类型的特例，平时很少出现            |
> | ==const== | 只读取一次就能获得数据（如：主键）                           |
> | qe_ref    | 驱动表和关联表中的每行进行组合并且仅有一行记录。这是除了system 和 const 类型之外, 这是最好的联接类型。当连接使用索引的所有部分时, 索引是主键或唯一非 NULL 索引时, 将使用该值 |
> | ref       | 非唯一性索引扫描，返回符合某个索引值的所有记录，可能会有多条记录匹配 |
> | ==range== | 范围扫描，基于索引做扫描（如：BETWEEN、IN、>=、LIKE等操作）  |
> | ==index== | 全索引扫描，index与all的区别是index类型只遍历索引树          |
> | ==all==   | 全表扫描，遍历全表直至找到匹配的行                           |

#### 5. possible_keys（JSON名： possible_keys）

>如果该列是NULL，则代表没有相关的索引。在这种情况下，可以通过==检查WHERE子句看它是否引用了某些列或适合索引的列来提高查询性能==。如果是这样，那么就需要创造一个适当的索引，并再次用`EXPLAIN`检查。

#### 6. key（JSON名：key）

>如果没有选择索引，键是NULL。要想强制MySQL使用或忽视possible_keys列中的索引，在查询中使用==FORCE INDEX==、==USE INDEX==或者==IGNORE INDEX==。
>
>对于`InnoDB`而言，即便是查询也选择主键索引，辅助索引（`secondary index`）可能会覆盖所选列，因为InnoDB将主键值存储在每个辅助索引中。如果key为NULL，则代表MySQL未发现可用于提高效率的索引。
>
>对于`MyISAM`的表，运行 ANALYZE TABLE 有助于优化器选择更好的索引。`myisamchk --analyze` 也是如此。

#### 7. ref（JSON名：ref）

>被用来标识那些用来进行索引比较的列或者常量。

#### 8. rows（JSON名：rows）

>表示mysql根据表统计信息及索引选用情况，估算的找到所需的记录所需要读取的行数。

#### 9. Extra

>mysql附加信息，提供了与操作有关联的信息。
>
>- `const row not found`：对空表做类似SELECT ... FROM tabel_name的查询操作
>- `Deleting all rows`：使用DELETE时，某些存储引擎(MyISAM)支持一些简单快速的处理方法。
>- `Distinct`：去重搜索是会显示出该内容
>- `FirstMatch(tbl_name)`：表示 table_name 使用的半连接的FirstMatch连接策略。
>- `Full scan on NULL key`：当查询优化器不能使用索引查询时，那么查询优化后执行回退策略。
>- `Impossible HAVING`：HAVING条件过滤没有效果，或者是始终选不出任何列(理解为返回已有查询的结果集)。
>- `Impossible WHERE`：WHERE条件过滤没有效果，或者是始终选不出任何列(理解为最终是全表扫描)。
>- `No tables used`：查询没有FROM子句，或者有一个 FROM DUAL子句。
>- `Not exists`：MySQL能够对LEFT JOIN查询进行优化，并且在查找到符合LEFT JOIN条件的行后，则不再查找更多的行。
>- ==Using index==：==只需通过索引树就可以从表中获取列的信息==，无需额外去读取真实的行数据。如果查询使用的列值仅仅是一个简单索引的部分值，则会使用这种策略来优化查询。对于innoDB数据库中的表有一个自定义的聚簇索引，该索引能够起作用，即使是Using index并没有出现在Extra列中。这种情况下的type字段为index并且key字段的值为PRIMARY。
>- ==Using where==：WHERE条件用于筛选出与下一个表匹配的数据然后返回给客户端。除非故意做的全表扫描，否则连接类型是`ALL`或者是`index`，且在`Extra`列的值中没有`Using Where`，则该查询可能是有问题的。