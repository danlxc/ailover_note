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

## 回退的几个易混淆内容

```
命令 作用域    常用情景
git reset    提交层面    在私有分支上舍弃一些没有提交的更改
git reset    文件层面    将文件从缓存区中移除
git checkout    提交层面    切换分支或查看旧版本
git checkout    文件层面    舍弃工作目录中的更改
git revert    提交层面    在公共分支上回滚更改
git revert    文件层面    （然而并没有）
```

| 命令 | 作用域 | 常用情景 |
| :--- | :--- | :--- |
| git reset | 提交层面 | 在私有分支上舍弃一些没有提交的更改 |
|  | git reset HEAD~2 | 让某个分支向后回退了两个提交 |
|  | git reset --mixed HEAD | 将你当前的改动从缓存区中移除，但是这些改动还留在工作目录中 |
|  | git reset --hard HEAD | 完全舍弃你没有提交的改动 |
| git reset | 文件层面 | 将文件从缓存区中移除 |
|  | git reset HEAD~2 foo.py | 将倒数第二个提交中的foo.py加入到缓存区中，供下一个提交使用 |
|  | git reset HEAD foo.py | 将当前的 foo.py 从缓存区中移除出去，而不会影响工作目录中对 foo.py 的更改 |
| git checkout | 提交层面 | 切换分支或查看旧版本 |
|  | git checkout hotfix | 将HEAD移到一个新的分支，然后更新工作目录 |
|  | git checkout HEAD~2 | 将checkout 到当前提交的祖父提交 |
| git checkout | 文件层面 | 舍弃工作目录中的更改 |
|  | git checkout HEAD~2 foo.py | 将工作目录中的 foo.py 同步到了倒数第二个提交中的 foo.py |
| git revert | 提交层面 | 在公共分支上回滚更改 |
|  | git revert HEAD~2 | 找出倒数第二个提交，然后创建一个新的提交来撤销这些更改，然后把这个提交加入项目中。 |
| git revert | 文件层面 | （然而并没有） |

## 分支管理策略 --no-ff

通常，合并分支时，如果可能，Git会用`Fast forward`模式，但这种模式下，删除分支后，会丢掉分支信息。

如果要强制禁用`Fast forward`模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

下面我们实战一下`--no-ff`方式的`git merge`：

```
$ git merge --no-ff -m "merge with no-ff" dev
Merge made by the 'recursive' strategy.
 readme.txt | 1 +
 1 file changed, 1 insertion(+)
```

## Bug分支  git stash

工作区是干净的，刚才的工作现场存到哪去了？用`git stash list`命令看看：

```
$ git stash list
stash@{0}: WIP on dev: f52c633 add merge
```

工作现场还在，Git把stash内容存在某个地方了，但是需要恢复一下，有两个办法：

一是用`git stash apply`恢复，但是恢复后，stash内容并不删除，你需要用`git stash drop`来删除；

另一种方式是用`git stash pop`，恢复的同时把stash内容也删了：

```
$ git stash pop
On branch dev
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   hello.py

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   readme.txt

Dropped refs/stash@{0} (5d677e2ee266f39ea296182fb2354265b91b3b2a)
```



