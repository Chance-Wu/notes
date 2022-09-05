> **记录用户对mysql的操作过程**
>
> ```shell
> mysql -u root -p --tee=C:\test.log
> ```
>
> //注意这里路径不需要加上引号
>
> 不想记录log时，使用 `notee` 命令，在这个命令后的操作将不会再被记录。
>
> ```mysql
> mysql> notee;
> ```



>**将查询结果导出到文件**
>
>```shell
># 直接使用重定向功能
>mysql -u root -p -e "use mysql;show tables;" > C:\log.txt
>```
>
>```mysql
># 使用tee命令
>mysql> tee C:\log.txt;
>mysql> use mysql;
>mysql> show tables;
>mysql> notee;
>```
>
>```mysql
>select * from tableName into outfile 'fineNane';
>```



>**执行外部文件中的sql语句**
>
>```shell
># 如果ss.sql中使用了use数据库，则-D数据库选项可以忽略
>mysql –uroot –p123456 -Dtest < d:\test\ss.sql
>```
>
>```mysql
># 进入mysql控制台后，使用source命令执行
>mysql> source d:\test\ss.sql 或 \. d:\test\ss.sql
>```