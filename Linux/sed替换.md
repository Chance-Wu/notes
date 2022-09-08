以下替换不使用-i参数真的替换文件内容，仅仅展示替换后的结果。



### 一、普通替换

---

原文件内容：hehe wiki jimo wiki wiki

```shell
sed 's/wiki/haha/g' jimo
hehe haha jimo haha haha
```



### 二、带有变量的替换

---

```shell
v1=lily
echo $v1
lily
```

这时候替换需要**使用双引号**，才能解析变量：

```shell
sed "s/wiki/$v1/g" jimo
hehe haha jimo lily lily
```

不过有个变量2：

```shell
v2=/tmp/txt
echo $v2
/tmp/txt
```

再次替换时发现报错：

```shell
sed "s/wiki/$v2/g" jimo
sed: -e expression #1, char 9: unknown option to `s'
```

带入变量，sed命令变成了 `sed "s/wiki/tmp/txt/g" jimo`, 这个命令是不合法的。

那么解决方案是什么？

自定义分隔符：

```shell
sed "s|wiki|$v2|g" jimo
hehe haha jimo /tmp/txt /tmp/txt
```