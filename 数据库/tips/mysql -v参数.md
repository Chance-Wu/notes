可以用shell脚本操作mysql数据库，使用mysql的-e参数可以执行各种sql的(创建，删除，增，删，改、查)等各种操作 。

**-e选项**	execute command and quit

用 **mysql -e** 生成结果导入指定文件时：

- `-v`：显示语句本身
- `-vv`：增加显示查询结果行数
- `-vvv`：增加显示执行时间

```shell
mysql -vvv -e "source file" > output.log
```