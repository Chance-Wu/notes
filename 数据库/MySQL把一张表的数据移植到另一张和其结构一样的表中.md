#### 所有字段都移到新表的写法

---

```sql
insert into new_table_name select * from table_name where job_id=1;
```



#### 移动部分字段到新表的写法

---

```sql
insert into new_table_name  (column1,column2)  select column1,column2 from table_name where job_id=1;
```

