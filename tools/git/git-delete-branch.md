## Git删除本地分支

### 1.删除除了master之外的所有分支

```
git branch | grep -v "master" | xargs git branch -D
```

**注意点**

* 执行前需要切换到master分支执行
* 当前分支未做修改

### 2.删除指定分支

#### 删除本地分支：

```
git branch -d 分支名（remotes/origin/分支名）
```

#### 强制删除本地分支：

```
git branch -D 分支名
```

#### 删除远程分支：

```
git push origin --delete 分支名（remotes/origin/分支名）
```