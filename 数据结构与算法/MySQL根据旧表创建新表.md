`create table table_new like table_old;` 

`create table table_new as select * from table_old;` 



区别：

- create table like 复制表结构和索引等约束，没有数据，不支持oracle。
- create table as select复制表结构和数据，没有索引等约束。

两种方式在复制表的时候均不会复制权限对表的设置。比如说原本对表B做了权限设置，复制后，表A不具备类似于表B的权限。