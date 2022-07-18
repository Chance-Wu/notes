### 使用 ALTER TABLE 语句重命名表

---

```sql
ALTER TABLE tbl_name RENAME TO new_tbl_name tbl_name
```

>RENAME TABLE 语句不能用于重命名临时表，这时就可以使用`ALTER TABLE`语句来**重命名一个临时表**。



### RENAME TABLE

---

要更改一个或多个表，我们使用`RENAME TABLE`语句如下：

```sql
RENAME TABLE old_table_name TO new_table_name;
```

旧表(`old_table_name`)必须存在，新表(`new_table_name`)必须不存在。 如果新表`new_table_name`存在，则该语句将失败。

> 在执行`RENAME TABLE`语句之前，必须确保没有活动事务。
>
> 注意`RENAME TABLE`语句不是原子的。**如果在任何时候发生错误，MySQL会将所有重新命名的表都回滚到旧名称**。
>
> 在安全性方面，**任何权限必须手动迁移到新表**。
>
> 在重命名表之前，应该彻底地评估影响。 例如，应该调查哪些应用程序正在使用该表。 如果表的名称更改，那么引用表名的应用程序代码也需要更改。 此外，必须手动调整引用该表的其他数据库对象。

#### 重命名视图引用的表





#### 重命名由存储过程引用的表





#### 重命名引用外键的表





#### 重命名多个表

