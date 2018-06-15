### 查看工作区和版本库里面最新版本的区别

`git diff HEAD -- readme.txt`

---

## 撤销修改的两种情况

### 场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令。

`git checkout -- file`

---

### 场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令，就回到了场景1，第二步按场景1操作。

`git reset HEAD file`

---

### 场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。

`git log`  
`git reflog`  
`git log --pretty=oneline //漂亮的格式`  
_首先，Git必须知道当前版本是哪个版本，在Git中，用HEAD表示当前版本，也就是最新的提交3628164...882e1e0（注意我的提交ID和你的肯定不一样），上一个版本就是HEAD^，上上一个版本就是HEAD^^，当然往上100个版本写100个^比较容易数不过来，所以写成HEAD~100_

---

### 本地仓库与github仓库关联并同步

```
git remote add origin git@github.com:danlxc/danny.git

git push -u origin master
```

# 创建并合并分支

查看分支：`git branch`

创建分支：`git branch <name>`

切换分支：`git checkout <name>`

创建+切换分支：`git checkout -b <name>`

合并某分支到当前分支：`git merge <name>`

删除分支：`git branch -d <name>`

## 细节详情

创建`dev`分支，然后切换到`dev`分支：

```
$ git checkout -b dev
```

`git checkout`命令加上`-b`参数表示创建并切换，相当于以下两条命令：

```
$ git branch dev

$ git checkout dev
```

用`git branch`命令查看当前分支：

```
$ git branch
* dev
  master
```

把`dev`分支的工作成果合并到`master`分支上：

```
$ git merge dev
```

删除`dev`分支了：

```
$ git branch -d dev
```

## 解决冲突

用带参数的`git log`也可以看到分支的合并情况：

```
$ git log --graph --pretty=oneline --abbrev-commit
```

## 撤销某次的提交,保留其后的提交

```
git revert HEAD                     //撤销最近一次提交
git revert HEAD~1                   //撤销上上次的提交，注意：数字从0开始
git revert 0ffaacc                  //撤销0ffaacc这次提交
```



