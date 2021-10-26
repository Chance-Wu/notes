#### 空间清理

---

linux空间清理

```shell
# 发现大量刚刚删除文件的进程存在，kill掉进程（或者重启进程）
lsof | grep deleted

# 循环检测发现大目录及其内的文件
du -h --max-depth=1 | sort -gr
```

获取大文件：到jenkins的家目录

```shell
find . -type f -size +100M
du -ah --max-depth=1
find . -type f -size +100M | wc -l
find . -type f   -size +100M  -exec  rm -rf {} \;

du -sh .[!.]* * | sort -hr  #查看隐藏文件

find . -type f -size +800M  -print0 | xargs -0 du -h | sort -nr
du -hm --max-depth=2 | sort -nr | head -12  
du -h --max-depth=1


#######
###删除操作
在jenkins工作路径下/var/lib/jenkins
find . -type f   -size +200M   -name *.jar    -exec  rm -rf {} \;
```



#### 构建工作区（workspace）

---

> 每一个构建（build）都需要一个workspace 目录作为构建的工作区，执行job配置中指定的任务。默认情况下，构建工作区workspace是不会自动清理的，也就是说每一个job的build结束后，workspace被遗留在master或者slave对应的工作区目录下，Jenkins的本意是为下次构建复用工作区目录，这样一些代码下载，编译等可以加速。
> ==可用Workspace Cleanup Plugin 清理==

- job 中保存的是项目是在 jenkins 上的配置、日志、构建结果等；
- workspace 就是工作目录，一般就是 Down 下来的源代码目录(网页打开的显示工作目录。

