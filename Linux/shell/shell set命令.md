set命令用来修改Shell环境的运行参数，也就是可以定制环境。一共有十几个参数可以定制，官方手册有完整清单。

官方：https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html

### 一、常用

```shell
set -o errexit
set +o nounset
set -o pipefail
```



### 二、脚本中报错即刻退出

---

- `set -e` 使得脚本只要发生错误，就终止执行；
- set +e 表示关闭-e选项。（-e的另一种写法-o errexit）

```shell
set -o errexit
```



### 三、遇到不存在的变量报错，并停止执行

---

```shell
#!/usr/bin/env bash

echo $a
echo bar
```

执行结果：

```shell

bar
```

可以看到，`echo $a`输出了一个空行，Bash 忽略了不存在的$a，然后继续执行`echo bar`。

`set -u` ：脚本在头部加上它，**遇到不存在的变量会报错，并停止执行**。-u还有另一种写法-o nounset。

```shell
set -o nounset
```



### 四、只要一个子命令失败，整个管道命令就失败，脚本终止执行

---

管道命令，就是多个子命令通过管道运算符（|）组合成为一个大的命令。**Bash 会把最后一个子命令的返回值，作为整个命令的返回值**。也就是说，只要最后一个子命令不失败，管道命令总是会执行成功，因此它后面命令依然会执行，set -e就失效了。（这是set -e 失效的一个特殊情况）

`set -o pipefail`：只要一个子命令失败，整个管道命令就失败，脚本就会终止执行。

```shell
set -o pipefail
```



###  五、打印执行的命到屏幕

---

在liunx脚本中可用set -x就可有详细的日志输出。

shell脚本中：

```shell
set -x　　　 #启动"-x"选项 
set +x　　　　 #关闭"-x"选项
```

>也可以在启动脚本的时候添加该参数：
>
>```shell
>/bin/bash -x test.sh
>```
>
>**-x选项可以⽤来跟踪脚本的执⾏**，使shell在执⾏脚本的过程中把它实际执⾏的每⼀个命令⾏显⽰出来，并且在⾏⾸显⽰一个"+"，"+"后⾯显⽰的是经过了变量替换后的命令⾏内容，有助于分析实际执⾏的命令。