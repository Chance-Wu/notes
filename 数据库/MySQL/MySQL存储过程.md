存储过程是为了**完成特定功能的SQL语句集**，经编译创建并保存在数据库中，用户可通过指定存储过程的名字并给定参数（需要时）来调用执行。

存储过程思想上很简单，就是数据库SQL语言层面的代码封装与重用。

优点：

- 存储过程可封装，并隐藏复杂的商业逻辑。
- 存储过程可以**回传值**，并可以**接受参数**。
- 存储过程无法使用 SELECT 指令来运行，因为它是子程序，与查看表，数据表或用户定义函数不同。
- 存储过程可以用在数据检验，强制实行商业逻辑等。

缺点：

- 存储过程，往往定制化于特定的数据库上，因为支持的编程语言不同。当切换到其他厂商的数据库系统时，需要重写原有的存储过程。
- 存储过程的性能调校与撰写，受限于各种数据库系统。



### 一、存储过程的创建和调用

---

- 存储过程就是具有名字的一段代码，用来完成一个特定的功能。
- 创建的**存储过程保存在数据库的数据字典中**。

#### 1.1 创建存储过程

```sql
CREATE
    [DEFINER = { user | CURRENT_USER }]
　PROCEDURE sp_name ([proc_parameter[,...]])
    [characteristic ...] routine_body
 
proc_parameter:
    [ IN | OUT | INOUT ] param_name type
 
characteristic:
    COMMENT 'string'
  | LANGUAGE SQL
  | [NOT] DETERMINISTIC
  | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
  | SQL SECURITY { DEFINER | INVOKER }
 
routine_body:
　　Valid SQL routine statement
 
[begin_label:] BEGIN
　　[statement_list]
　　　　……
END [end_label]
```

#### 1.2 声明语句结束符

可以自定义：

```sql
DELIMITER $$
或
DELIMITER //
```

#### 1.3 声明存储过程

```sql
CREATE PROCEDURE demo_in_parameter(IN p_in int)
```

#### 1.4 存储过程开始和结束符号

```sql
BEGIN ... END
```

#### 1.5 变量赋值

```sql
SET @p_in=1
```

#### 1.6 变量定义

```sql
DECLARE l_int int unsigned default 4000000;
```

#### 1.7 创建mysql存储过程、存储函数

```sql
create procedure 存储过程名(参数)
```

#### 1.8 存储过程体

```sql
create function 存储函数名(参数)
```

#### 1.9 实例

创建数据库，备份数据表用于示例操作：

```sql
mysql> create database db1;
mysql> use db1;
mysql> create table PLAYERS as select * from TENNIS.PLAYERS;
mysql> create table MATCHES  as select * from TENNIS.MATCHES;
```

删除给定球员参加的所有比赛：

```sql
mysql> delimiter $$　　#将语句的结束符号从分号;临时改为两个$$(可以是自定义)
mysql> CREATE PROCEDURE delete_matches(IN p_playerno INTEGER)
    -> BEGIN
    -> 　　DELETE FROM MATCHES
    ->    WHERE playerno = p_playerno;
    -> END$$
Query OK, 0 rows affected (0.01 sec)
 
mysql> delimiter;　　#将语句的结束符号恢复为分号
```

解析：默认情况下，存储过程和默认数据库相关联，如果想指定存储过程创建在某个特定的数据库下，那么在过程名前面加数据库名做前缀。 在定义过程时，使用 `DELIMITER $$` 命令将语句的结束符号从分号 **;** 临时改为两个 `$$`，**使得过程体中使用的分号被直接传递到服务器，而不会被客户端（如mysql）解释**。

调用存储过程：

```sql
call sp_name[(传参)]
```

#### 1.10 存储过程体

- 存储过程体包含了在过程调用时必须执行的语句，例如：dml、ddl语句，if-then-else和while-do语句、声明变量的declare语句等
- 过程体格式：以begin开始，以end结束(可嵌套)

```sql
BEGIN
　　BEGIN
　　　　BEGIN
　　　　　　statements; 
　　　　END
　　END
END
```

注意：每个嵌套及其中的每条语句，必须以分号结束，表示过程体结束的begin-end块（又叫做复合语句compound statement），则不需要分号。

为语句块贴标签：

```sql
label1: BEGIN
　　label2: BEGIN
　　　　label3: BEGIN
　　　　　　statements; 
　　　　END label3 ;
　　END label2;
END label1
```

- 增强代码的可读性
- 在某些语句（如：leave和itrate语句），需要用到标签。



### 二、存储过程的参数

---

MySQL存储过程的参数用在存储过程的定义，共有三种参数类型，IN,OUT,INOUT，形式如下：

```sql
CREATE PROCEDURE 存储过程名([IN |OUT |INOUT ] 参数名 数据类型...)
```

- **IN 输入参数**：表示调用者向过程传入值（传入值可以是字面量或变量）
- **OUT 输出参数**：表示过程向调用者传出值(可以返回多个值)（传出值只能是变量）
- **INOUT 输入输出参数**：既表示调用者向过程传入值，又表示过程向调用者传出值（值只能是变量）

#### 2.1 in输入参数

```sql
mysql> delimiter $$
mysql> create procedure in_param(in p_in int)
		-> begin
		-> select p_in;
		-> select p_in=2;
		-> select P_in;
		-> end$$
mysql> delimiter ;

mysql> set @p_in=1;

mysql> call in_param(@p_in);
+------+
| p_in |
+------+
|    1 |
+------+
 
+------+
| P_in |
+------+
|    2 |
+------+

mysql> select @p_in;
+-------+
| @p_in |
+-------+
|     1 |
+-------+
```

以上可以看出，p_in在存储过程中被修改，但并不影响@p_in的值，因为前者为局部变量、后者为全局变量。

#### 2.2 out输出参数

```sql
mysql> delimiter //
mysql> create procedure out_param(out p_out int)
    ->   begin
    ->     select p_out;
    ->     set p_out=2;
    ->     select p_out;
    ->   end
    -> //
mysql> delimiter ;
 
mysql> set @p_out=1;
 
mysql> call out_param(@p_out);
+-------+
| p_out |
+-------+
|  NULL |
+-------+
　　#因为out是向调用者输出参数，不接收输入的参数，所以存储过程里的p_out为null
+-------+
| p_out |
+-------+
|     2 |
+-------+
 
mysql> select @p_out;
+--------+
| @p_out |
+--------+
|      2 |
+--------+
　　#调用了out_param存储过程，输出参数，改变了p_out变量的值
```

#### 2.3 inout输入参数

```sql
mysql> delimiter $$
mysql> create procedure inout_param(inout p_inout int)
    ->   begin
    ->     select p_inout;
    ->     set p_inout=2;
    ->     select p_inout;
    ->   end
    -> $$
mysql> delimiter ;
 
mysql> set @p_inout=1;
 
mysql> call inout_param(@p_inout);
+---------+
| p_inout |
+---------+
|       1 |
+---------+
 
+---------+
| p_inout |
+---------+
|       2 |
+---------+
 
mysql> select @p_inout;
+----------+
| @p_inout |
+----------+
|        2 |
+----------+
#调用了inout_param存储过程，接受了输入的参数，也输出参数，改变了变量
```

注意：

1. 如果过程没有参数，也必须在过程名后面写上小括号。
2. 确保参数的名字不等于列的名字，否则在过程体中，参数名被当做列名来处理。

建议：

- 输入值使用in参数。
- 返回值使用out参数。
- inout参数就尽量的少用。



### 三、变量

---

#### 3.1 变量定义

局部变量声明一定要放在存储过程体的开始：

```sql
DECLARE variable_name [,variable_name...] datatype [DEFAULT value];
```

其中，datatype为MySQL的数据类型，如：int、float、date、varchar(length)。

例：

```sql
DECLARE l_int int unsigned default 4000000;  
DECLARE l_numeric number(8,2) DEFAULT 9.95;  
DECLARE l_date date DEFAULT '1999-12-31';  
DECLARE l_datetime datetime DEFAULT '1999-12-31 23:59:59';  
DECLARE l_varchar varchar(255) DEFAULT 'This will not be padded';
```

#### 3.2 变量赋值

```sql
SET 变量名 = 表达式值 [,variable_name = expression ...]
```

#### 3.3 用户变量

> 在MySQL客户端使用用户变量：

```sql
mysql > SELECT 'Hello World' into @x;  
mysql > SELECT @x;  
+-------------+  
|   @x        |  
+-------------+  
| Hello World |  
+-------------+  
mysql > SET @y='Goodbye Cruel World';  
mysql > SELECT @y;  
+---------------------+  
|     @y              |  
+---------------------+  
| Goodbye Cruel World |  
+---------------------+  
 
mysql > SET @z=1+2+3;  
mysql > SELECT @z;  
+------+  
| @z   |  
+------+  
|  6   |  
+------+
```

> 在存储过程中使用用户变量：

```sql
mysql > CREATE PROCEDURE GreetWorld( ) SELECT CONCAT(@greeting,' World');
mysql > SET @greeting='Hello';
mysql > CALL GreetWorld( );
+----------------------------+  
| CONCAT(@greeting,' World') |  
+----------------------------+  
|  Hello World               |  
+----------------------------+
```

> 在存储过程间传递全局范围的用户变量：

```sql
mysql> CREATE PROCEDURE p1()   SET @last_procedure='p1';  
mysql> CREATE PROCEDURE p2() SELECT CONCAT('Last procedure was ',@last_procedure);  
mysql> CALL p1( );  
mysql> CALL p2( );  
+-----------------------------------------------+  
| CONCAT('Last procedure was ',@last_proc       |  
+-----------------------------------------------+  
| Last procedure was p1                         |  
 +-----------------------------------------------+
```

注意：

1. 用户变量名一般以@开头
2. 滥用用户变量会导致程序难以理解及管理



### 四、存储过程的查询

---

查看某数据库下的存储过程：

```sql
selectname from mysql.proc where db='数据库名';

或者
selectroutine_name from information_schema.routines where routine_schema='数据库名';

或者
showprocedure status where db='数据库名';
```

查看存储过程的详细：

```sql
SHOW CREATE PROCEDURE 数据库.存储过程名;
```



### 五、存储过程的控制语句

---

#### 5.1 变量作用域

内部变量在其作用域范围内享有更高的优先权，当执行到end时，内部变量消失，此时已经在其作用域外，变量不再可见，因为在存储过程外再也不能找到这个申明的变量，但是可以通过out参数或者将其值指派给会话变量来保存其值。

#### 5.2 条件语句

> **if-then-else 语句**
>
> ```sql
> if 条件 then
> 	...sql
> else
> 	...sql
> end if;
> ```

>**case 语句**
>
>```sql
>case var
>	when 0 then
>		...sql
>	when 1 then
>		...sql
>	else
>		...sql
>end case;
>
>或
>case
>    when var=0 then
>        insert into t values(30);
>    when var>0 then
>    when var<0 then
>    else
>end case
>```

#### 5.3 循环语句

>**while...end while**
>
>```sql
>while 条件 do
>	--循环体
>end while;
>```

>**repeat...end repeat**
>
>在执行操作后检查结果，而 while 则是执行前进行检查。
>
>```sql
>repeat
>	--循环体
>until 循环条件
>end repeat;
>```

>**loop...end lop**
>
>loop 循环不需要初始条件，这点和 while 循环相似，同时和 repeat 循环一样不需要结束条件，leave 语句的意义是离开循环。
>
>```sql
>LOOP_LABLE:loop
>	insert into t values(v);  
>	set v=v+1;  
>	if v >=5 then 
>		leave LOOP_LABLE;  
>	end if;
>end loop;  
>```

>LABLED 标号：标号可以用在 begin repeat while 或者 loop 语句前，语句标号只能在合法的语句前面使用。可以**跳出循环**，使运行指令达到复合语句的最后一步。

#### 5.4 迭代

ITERATE 通过引用符合语句的标号，来重新开始复合语句。

```sql
declare v int;  
set v=0;  
LOOP_LABLE:loop  
	if v=3 then   
		set v=v+1;  
		ITERATE LOOP_LABLE;  
	end if;  
	insert into t values(v);  
	set v=v+1;  
	if v>=5 then 
		leave LOOP_LABLE;  
	end if;  
end loop;
```



### 六、常用

---

#### 6.1 表名为变量

如果一个存储过程的变化的部分只有表名的部分，可以给存储过程传入这个表名。这就需要我们承接一下传入的参数，然后使用PREPARE了，关于PREPARE，需要参考官方文档来解释一下，这里先贴上解决问题的代码：

```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `updateTest`(IN table_name VARCHAR(50))
BEGIN
    SET @tbl_name=CONCAT("",table_name);
    SET @statement = CONCAT("UPDATE ",@tbl_name," AS c,`x1` AS b SET c.`hello` = b.`world` WHERE c.`id` = b.`uuid`; ");
    PREPARE stmt FROM @statement;
    EXECUTE stmt;
END
```

>PREPARE语句准备一个SQL的statement，并且，它会给这个statement一个名字以便我们之后引用。我们可以使用EXECUTE语句执行这个准备好的statement，也可以使用DEALLOCATE PREPARE来释放掉它。

#### 6.2 show procedure status