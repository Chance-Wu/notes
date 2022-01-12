用于字符串输出。

#### 1. 显示普通字符串

---

```shell
echo "It is a test"
```

#### 2. 显示转义字符

---

```shell
echo "\"It is a test\""
```

#### 3. 显示变量

---

read命令从标准输入中读取一行，并把输入行的每个字段的值指定给shell变量。

```shell
read name
chance
echo "$name It is a test"
```

#### 4. 显示换行

---

-e 开启转义。

```shell
echo -e "OK\n"
```

#### 5. 不显示换行

---

-e 开启转义 \c不换行

```shell
echo -e “OK! \c”
```

#### 6. 显示结果定向至文件

---

```shell
echo "It is a test" > myfile.txt
```

#### 7. 原样输出字符串，不进行转义或取变量（用单引号）

---

```shell
echo '$name\"'
```

#### 8. 显示命令执行结果

---

使用反引号。结果显示当前日期。

```shell
echo `date`
```