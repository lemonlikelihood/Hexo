---
title: git简单攻略
date: 2019-05-24 10:04:10
tags: git
categories: git
top: 100
---

### git全局用户申明
```
git config --global user.name "Your Name"
git config --global user.email "email@example.com" 
```

### 创建管理库
```
git init
```

### 添加文件
```
git add reamde.md
```

### 提交文件到仓库
```
git commit -m "message"
```

为什么git提交文件需要add和commit两步呢，因为commit可以一次提交很多次add不同的文件，比如
```
git add file1.txt
git add file2.txt file3.txt
git commit -m "add 3files"
```

### 查看仓库状态
```
git status
```

### 查看文件修改内容
```
git diff readme.txt
```

### 查看提交历史
```
git log
```

以便确定回退到哪个版本。

### 查看命令历史
```
git reflog
```
以便确定回到未来的哪个版本。

### 版本指针
HEAD指向的版本就是当前的版本，HEAD^指向前一个版本，HEAD^^指向前前版本，HEAD~100指向第前100个版本。因此，git允许我们在历史之间穿梭。

### 版本穿梭
```
git reset --hard commit_id
```

### 丢弃工作区的修改
```
git checkout -- file
```

命令`git checkout -- readme.txt`意思就是，把readme.txt文件在工作区的修改全部撤销，这里有两种情况：

一种是readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；

一种是readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态。

总之，就是让这个文件回到最近一次git commit或git add时的状态。

### 从暂存区回到工作区
```
git reset HEAD readme.txt
```

如果你把文件git add到暂存区，但是还没有git commit到仓库，可以使用git reset HEAD file 将暂存区的修改撤销掉，重新放回到工作区。
git reset命令既可以回退版本，也可以把暂存区的修改回退到工作区。当我们用HEAD时，表示最新的版本。

### 小结
场景1：当你改乱了工作区某个文件的内容，想直接丢弃工作区的修改时，用命令git checkout -- file。

场景2：当你不但改乱了工作区某个文件的内容，还添加到了暂存区时，想丢弃修改，分两步，第一步用命令git reset HEAD file，就回到了场景1，第二步按场景1操作。

场景3：已经提交了不合适的修改到版本库时，想要撤销本次提交，参考版本回退一节，不过前提是没有推送到远程库。

### 删除文件
```
git rm test.txt
git commit -m "remove test.txt"
```
一般情况下，你通常直接在文件管理器中把没用的文件删了，或者用rm命令删了：rm test.txt。这个时候，Git知道你删除了文件，因此，工作区和版本库就不一致了，git status命令会立刻告诉你哪些文件被删除了。

现在你有两个选择，一是确实要从版本库中删除该文件，那就用命令git rm删掉，并且git commit，现在，文件就从版本库中被删除了。

另一种情况是删错了，因为版本库里还有呢，所以可以很轻松地把误删的文件恢复到最新版本：
```
git checkout -- test.txt
```

小提示：先手动删除文件，然后使用git rm <file>和git add<file>效果是一样的。
注意：从来没有被添加到版本库就被删除的文件，是无法恢复的！ 

### 添加远程仓库
```
git remote add origin git@github.com:hahaha/hahaha.git
```

### 把本地库的所有内容推送到远程库上
```
git push -u origin master
```

把本地库的内容推送到远程，用git push命令，实际上是把当前分支master推送到远程。由于远程库是空的，我们第一次推送master分支时，加上了-u参数，Git不但会把本地的master分支内容推送的远程新的master分支，还会把本地的master分支和远程的master分支关联起来，在以后的推送或者拉取时就可以简化命令。

### 提交仓库到到远程
```
git push origin master
```

### 从远程库克隆
```
git clone git@github.com:hahaha/gitskills.git
```
现在，假设我们从零开发，那么最好的方式是先创建远程库，然后，从远程库克隆。
首先，登陆GitHub，创建一个新的仓库，名字叫gitskills：
我们勾选Initialize this repository with a README，这样GitHub会自动为我们创建一个README.md文件。创建完毕后，可以看到README.md文件：
现在，远程库已经准备好了，下一步是用命令git clone克隆一个本地库：

注意把Git库的地址换成你自己的，然后进入gitskills目录看看，已经有README.md文件了：

### 转载申明
本文转载自廖雪峰的博客：[https://www.liaoxuefeng.com]


