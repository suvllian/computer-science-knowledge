## Git回退操作

### 删除远程仓库分支上的某次提交

使用git进行项目管理的过程中，可能会遇到这样的问题。在对本地文件修改后，提交commit并push到了远程仓库，之后发现自己忘记切换分支了。

很多人的解决方案是先将本地回滚，然后强制删除远程分支，实际上这是一种高风险的操作，任何用户不应该擅自做远程仓库的回退工作，这样可能对其他用户产生不可预知的影响。

#### 删除最后一次提交

``` shell
git revert HEAD
git push origin master
```

`revert`是放弃指定提交的修改，但是会生成一次新的提交，需要填写相应commit的message，以前的历史记录都在。

``` shell
git reset --hard HEAD
git push origin master -f
```

但是`reset`是将HEAD指针指到指定的提交，历史记录中不会出现放弃的提交记录。

`-f`是参数强制提交，因为reset之后，本地仓库版本落后于远程仓库，因此需要强制提交。

**reset参数解释**

* `git reset –-mixed`：回退到某个版本，只保留源码，回退commit和index file；
* `git reset --soft`：回退到某个版本，只回退commit信息，保留index file，如果还需要提交，可以直接commit；
* `git reset --hard`：彻底回退到某个版本，源码的本地改动也会被删除。