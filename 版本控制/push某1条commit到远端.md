默认情况下，git push会推送暂存区所有提交（即HEAD及其之前的提交），使用下面的命令

```bash
git push <remotename> <commit SHA>:<remotebranchname>
```

例如：git push origin 248ed23e2:branchname



### 一、第一种方式

---

即符合git操作的规则，从最初的commit开始一个一个提交，但是不能实现指定某一个commit,基本满足日常的开发异常情况了（只能按顺序提交）。

本地commit了3次提交但是并不想一下push到远程，根据功能或者时间的原因，想一个一个提交

 此时可以使用：

```bash
// 最下面的 一条为最老的一条，优先推送
git push origin 9267dd9:test  
// 接着第二条同样的命令，commit换掉即可
git push origin a3be5f8:test

// ... 依次按顺序一个一个提交...
```



### 二、第二种方式

---

采用cherry-pick用新分支去拉取当前分支的指定commit记录，之后推送到当前分支远程仓库实现推送指定历史提交的功能。

1. 创建临时分支

   ```bash
   // localbranch 为本地分支名  origin/feat 为远程目标分支
   git checkout -b  localbranch  --track origin/feat
   ```

2. 执行cherry-pick，将修改bug的记录同步过来

   ```bash
   git cherry-pick fcf254130f
   ```

3. 将临时分支记录推到目标分支