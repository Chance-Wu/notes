### 项目场景

---

在dev分支基础上新建一个分支tmp。也就是在dev和tmp同时进行开发。



### 问题描述

---

开发一段时间后，需要将dev合并到tmp。登陆gitlab在线将dev合并到tmp，在合并时出现了conflicts，在线解决冲突之后，再点击确定合并，结果反向将tmp合并到dev分支了！！



#### 原因

---

官方解释是，解决冲突后会将目标分支tmp合并到源功能分支上。参考文章[【精选】巨坑的GitLab在线解决冲突(解决后做了反向合并代码的操作？）_gitlab 冲突-CSDN博客](https://blog.csdn.net/u013487071/article/details/123485341)



### 避坑

---

不要gitlab在线[合并分支](https://so.csdn.net/so/search?q=合并分支&spm=1001.2101.3001.7020)，最好在本地用工具合并。



### 解决问题

---

如果已经出现反向合并了（tmp多次提交的日志已经穿插在了dev提交日志中），现在需要在dev分支上移除掉tmp分支提交的代码。我选择的方式是查看dev、tmp的提交记录，查找dev最早没有被tmp污染的commitID,然后基于此commitID新建一个新的分支dev_copy。然后通过git cherry-pick commonID 命令将dev上在此commitID之后的提交记录一一合并到dev_copy。最后废弃掉被污染的dev分支，留下dev_copy分支作为开发分支。 一次次的cherry-pick比较繁琐，但还是解决了问题。

#### 方法一

临时分支替代法：分支feature要合并到dev分支，且出现了冲突，可以先从feature分支拉一个临时分支feature_temp，用临时分支feature_temp合并到dev分支。

```
git checkout feature //先切换到feature 分支
git checkout -b feature_temp //拉出来新的分支
git push origin feature_temp //推送到远端
git branch --set-upstream-to=origin/feature_temp 
```

然后再gitlab界面上面选择feature_temp分支去做合并操作。

#### 方法二

回滚补救法：设feature分支要合并到dev分支，且出现了冲突，合并完成后，对feature分支做回滚操作。

```
git log //找到上一个版本的commitID
git reset --hard HEAD/commitID  //强制回退本地分支到上一个版本
git push origin HEAD --force 或者 git push -f origin feature //强制回退远程
```



















