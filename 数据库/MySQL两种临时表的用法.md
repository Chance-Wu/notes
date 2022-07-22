#### 外部临时表

---

通过 `create temporary table` 创建的临时表，这种临时表称为外部临时表。这种临时表只对当前用户可见，当前会话结束的时候，该临时表会自动关闭。这种临时表的命名与非临时表可以同名（同名后非临时表将对当前会话不可见，直到临时表被删除）。



#### 内部临时表

---

内部临时表是一种特殊轻量级的临时表，用来进行性能优化。这种临时表会**被MySQL自动创建并用来存储某些操作的中间结果**。这些操作可能包括在**优化阶段**或者**执行阶段**。这种内部表对用户来说是不可见的，但是通过EXPLAIN或者SHOW STATUS可以查看MYSQL是否使用了内部临时表用来帮助完成某个操作。内部临时表在SQL语句的优化过程中扮演着非常重要的角色， MySQL中的很多操作都要依赖于内部临时表来进行优化。但是使用内部临时表需要创建表以及中间数据的存取代价，所以用户在**写SQL语句的时候应该尽量的去避免使用临时表**。

内部临时表有两种类型：一种是**HEAP临时表**，这种临时表的所有数据都会存在内存中，对于这种表的操作不需要IO操作。另一种是**OnDisk临时表**，这种临时表会将数据存储在磁盘上。OnDisk临时表用来处理中间结果比较大的操作。如果HEAP临时表存储的数据大于MAX_HEAP_TABLE_SIZE，HEAP临时表将会被自动转换成OnDisk临时表。OnDisk临时表在5.7中可以通过INTERNAL_TMP_DISK_STORAGE_ENGINE系统变量选择使用MyISAM引擎或者InnoDB引擎。

首先定义一个表：

```sql
CREATE TABLE t1( a int, b int);
INSERT INTO t1 VALUES(1,2),(3,4);
```

在SQL中使用SQL_BUFFER_RESULT hint

SQL_BUFFER_RESULT 主要用来让MySQL尽早的释放表上的锁。因为如果数据量很大的话，需要较长时间将数据发送到客户端，通过将数据缓冲到临时表中可以有效减少读锁对表的占用时间。

例如：

```sql
explain format=json select SQL_BUFFER_RESULT * from t1;
EXPLAIN
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "1.40"
    },
    "buffer_result": {
      "using_temporary_table": true,
      "table": {
        "table_name": "t1",
        "access_type": "ALL",
        "rows_examined_per_scan": 2,
        "rows_produced_per_join": 2,
        "filtered": "100.00",
        "cost_info": {
          "read_cost": "1.00",
          "eval_cost": "0.40",
          "prefix_cost": "1.40",
          "data_read_per_join": "32"
        },
        "used_columns": [
          "a",
          "b"
        ]
      }
    }
  }
}
```

如果SQL语句中包含了DERIVED_TABLE。

在5.7中，由于采用了新的优化方式，我们需要使用 set optimizer_switch='derived_merge=off'来禁止derived table合并到外层的Query中。

例如：

```sql
explain format=json select * from (select * from t1) as tt;
EXPLAIN
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "1.40"
    },
    "table": {
      "table_name": "t1",
      "access_type": "ALL",
      "rows_examined_per_scan": 2,
      "rows_produced_per_join": 2,
      "filtered": "100.00",
      "cost_info": {
        "read_cost": "1.00",
        "eval_cost": "0.40",
        "prefix_cost": "1.40",
        "data_read_per_join": "32"
      },
      "used_columns": [
        "a",
        "b"
      ]
    }
  }
}
```

如果我们查询系统表的话，系统表的数据将被存储到内部临时表中。

当前不能使用EXPLAIN来查看是否读取系统表数据需要利用到内部临时表，但是可以通过SHOW STATUS来查看是否利用到了内部临时表。

```sql
select * from information_schema.character_sets;
show status like 'CREATE%';
```

如果DISTINCT语句没有被优化掉，即DISTINCT语句被优化转换为GROUP BY操作或者利用UNIQUE INDEX消除DISTINCT，内部临时表将会被使用。

```sql
explain format=json select distinct a from t1;
EXPLAIN
{
  "query_block": {
    "select_id": 1,
    "cost_info": {
      "query_cost": "1.40"
    },
    "duplicates_removal": {
      "using_temporary_table": true,
      "using_filesort": false,
      "table": {
        "table_name": "t1",
        "access_type": "ALL",
        "rows_examined_per_scan": 2,
        "rows_produced_per_join": 2,
        "filtered": "100.00",
        "cost_info": {
          "read_cost": "1.00",
          "eval_cost": "0.40",
          "prefix_cost": "1.40",
          "data_read_per_join": "32"
        },
        "used_columns": [
          "a"
        ]
      }
    }
  }
}
```

如果查询带有 ORDER BY 语句，并且不能被优化掉。下面几种情况会利用到内部临时表缓存中间数据，然后对中间数据进行排序。

































