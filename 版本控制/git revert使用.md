### 一、用法

---

git revert 撤销某次操作，此次操作之前和之后的commit和history都会保留，并且**把这次撤销，作为一次最新的提交**。

1.  `git revert HEAD` 撤销前一次 commit
2.  `git revert HEAD^` 撤销前前一次 commit
3.  `git revert commit_id`（比如:fa042ce57ebbe5bb9c8db709f719cec2c58ee7ff）

git revert是提交一个新的版本，将需要revert的版本的内容再反向修改回去，版本会递增，不影响之前提交的内容。

> Tip：通常情况下，上面这条revert命令会让程序员修改注释，这时候程序员应该标注revert的原因，假设程序员就想使用默认的注释，可以在命令中加上`-n`或者`--no-commit`，应用这个参数会让revert 改动只限于程序员的本地仓库，而不自动进行commit，如果程序员想在revert之前进行更多的改动，或者想要revert多个commit。



### 二、进阶用法

---

当有多个commit需要撤销，有可能是连续的，或是不连续的，那该怎么操作？

#### 2.1 连续

```bash
git revert -n commit_id_start..commit_id_end
```

使用该命令可以将提交撤回到commit_id_start的位置。

#### 2.2 不连续

1. git revert -n commit_id_1
2. git revert -n commit_id_3

使用该命令可以撤回到commit_id_1和commit_id_3的提交。