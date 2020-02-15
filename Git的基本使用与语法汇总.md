- [常用操作：](#%e5%b8%b8%e7%94%a8%e6%93%8d%e4%bd%9c)
  - [例：拉取远程分支代码与本地分支代码合并：](#%e4%be%8b%e6%8b%89%e5%8f%96%e8%bf%9c%e7%a8%8b%e5%88%86%e6%94%af%e4%bb%a3%e7%a0%81%e4%b8%8e%e6%9c%ac%e5%9c%b0%e5%88%86%e6%94%af%e4%bb%a3%e7%a0%81%e5%90%88%e5%b9%b6)
  - [例：切换至master分支报以下错误](#%e4%be%8b%e5%88%87%e6%8d%a2%e8%87%b3master%e5%88%86%e6%94%af%e6%8a%a5%e4%bb%a5%e4%b8%8b%e9%94%99%e8%af%af)
  - [例：merge代码时报如下错误：](#%e4%be%8bmerge%e4%bb%a3%e7%a0%81%e6%97%b6%e6%8a%a5%e5%a6%82%e4%b8%8b%e9%94%99%e8%af%af)
  - [例：误删本地分支和远程分支导致代码丢失，如何恢复？](#%e4%be%8b%e8%af%af%e5%88%a0%e6%9c%ac%e5%9c%b0%e5%88%86%e6%94%af%e5%92%8c%e8%bf%9c%e7%a8%8b%e5%88%86%e6%94%af%e5%af%bc%e8%87%b4%e4%bb%a3%e7%a0%81%e4%b8%a2%e5%a4%b1%e5%a6%82%e4%bd%95%e6%81%a2%e5%a4%8d)
  - [例：退出git log 或 git show ？](#%e4%be%8b%e9%80%80%e5%87%bagit-log-%e6%88%96-git-show)
  - [例：git stash 的应用](#%e4%be%8bgit-stash-%e7%9a%84%e5%ba%94%e7%94%a8)
- [报错信息：](#%e6%8a%a5%e9%94%99%e4%bf%a1%e6%81%af)
- [常用命令：](#%e5%b8%b8%e7%94%a8%e5%91%bd%e4%bb%a4)
    - [将文件放入Git仓库(两步)：](#%e5%b0%86%e6%96%87%e4%bb%b6%e6%94%be%e5%85%a5git%e4%bb%93%e5%ba%93%e4%b8%a4%e6%ad%a5)
    - [版本回退(不区分大小写)：](#%e7%89%88%e6%9c%ac%e5%9b%9e%e9%80%80%e4%b8%8d%e5%8c%ba%e5%88%86%e5%a4%a7%e5%b0%8f%e5%86%99)
    - [版本恢复：](#%e7%89%88%e6%9c%ac%e6%81%a2%e5%a4%8d)
    - [撤消工作区修改：](#%e6%92%a4%e6%b6%88%e5%b7%a5%e4%bd%9c%e5%8c%ba%e4%bf%ae%e6%94%b9)
    - [撤销暂存区修改：](#%e6%92%a4%e9%94%80%e6%9a%82%e5%ad%98%e5%8c%ba%e4%bf%ae%e6%94%b9)
    - [删除版本库中的文件](#%e5%88%a0%e9%99%a4%e7%89%88%e6%9c%ac%e5%ba%93%e4%b8%ad%e7%9a%84%e6%96%87%e4%bb%b6)
- [附：](#%e9%99%84)
    - [1.区别：](#1%e5%8c%ba%e5%88%ab)
    - [2.连接远程：](#2%e8%bf%9e%e6%8e%a5%e8%bf%9c%e7%a8%8b)
    - [3.分支操作：](#3%e5%88%86%e6%94%af%e6%93%8d%e4%bd%9c)
    - [4.解决冲突](#4%e8%a7%a3%e5%86%b3%e5%86%b2%e7%aa%81)
    - [5.Fast forward模式](#5fast-forward%e6%a8%a1%e5%bc%8f)
    - [6.修复Bug分支](#6%e4%bf%ae%e5%a4%8dbug%e5%88%86%e6%94%af)
    - [7.删除feature分支](#7%e5%88%a0%e9%99%a4feature%e5%88%86%e6%94%af)
    - [8.多人协作](#8%e5%a4%9a%e4%ba%ba%e5%8d%8f%e4%bd%9c)
    - [9.创建标签](#9%e5%88%9b%e5%bb%ba%e6%a0%87%e7%ad%be)
    - [10.本地仓库与远程的gitLab和gitHub连接](#10%e6%9c%ac%e5%9c%b0%e4%bb%93%e5%ba%93%e4%b8%8e%e8%bf%9c%e7%a8%8b%e7%9a%84gitlab%e5%92%8cgithub%e8%bf%9e%e6%8e%a5)

# 常用操作：


## 例：拉取远程分支代码与本地分支代码合并：

```
`远程分支：feature/v1
本地分支：feature/abc/v1

1. git checkout feature/v1 
//  若本地没有feature/v1分支，则创建一个：git checkout -b feature/v1 origin/feature/v1(同时建立与远程分支的连接)

2. git pull 
// 拉取远程的feature/v1分支上的代码（本地的feature/v1与远程的feature/v1已建立连接）
// 若拉取失败，说明未建立连接，执行git branch --set-upstream-to feature/v1 origin/feature/v1
 
3. git checkout feature/abc/v1	 //切换分支

4. git merge feature/v1  //合并分支
// 若有冲突，git status查看发生冲突的文件，手动解决冲突即可。` 

```

## 例：切换至master分支报以下错误

```
`error: pathspec 'master' did not match any file(s) known to git

// 解决办法

git branch -a // 查看分支

git fetch // 抓取远程分支至本地

git checkout master` 
```

## 例：merge代码时报如下错误：

```
`Please enter a commit message to explain why this merge is necessary.` 
```

此时界面无法操作，输入 :wq 即可。

详见：[https://www.cnblogs.com/wei325/p/5278922.html](https://www.cnblogs.com/wei325/p/5278922.html)

## 例：误删本地分支和远程分支导致代码丢失，如何恢复？

```
`git reflog // 查看分支操作历史

git branch <branch_name> HEAD@{4} // 恢复指定的分支` 
```

## 例：退出git log 或 git show ？

按下英文状态的 “q” 键即可。

## 例：git stash 的应用

需求：从远程分支上拉到本地的master分支，直接修改master分支上内容发现应该在新分支上应用修改并且撤销对master分支修改。

```
`//情形一：只有工作区修改，未进行git commit操作

git stash // 存储工作区修改
git checkout -b feature/new/v1 // 从master分支上切出新的分支
git stash list // 查看现有的储藏
git stash pop // 应用最新的储藏并清除当前的储藏

// 情形二：进行了git commit操作并且又有新的修改未提交

git reflog // 查看历史
git reset --soft [commit_id] // 回退到commit之前的那个版本并且保留所有修改
git stash // 存储所有修改
git checkout -b feature/new/v1
git stash pop // 应用先前在master分支上的所有修改` 
```

# 报错信息：

1.git bash无法输入内容：

此时终端实际上是可以输入内容的，只是看不到，解决办法：

```
`//输入
reset 回车 //具有清屏效果` 
```

2.git pull提示no tracking information

说明本地分支和远程分支的连接关系没有创建，用命令

```
`git branch --set-upstream-to dev origin/dev

//附：创建远程origin的dev分支到本地
git checkout -b dev origin/dev
//这样本地的dev分支就和远程的dev分支连接` 
```

# 常用命令：

```
`pwd //查看当前目录

ls -ah //查看当前目录下的文件，包含.git文件

git init // 初始化一个git仓库

git status //查看当前哪些文件被修改

git diff //即查看difference，可以查看哪些内容被修改

git log //查看提交的历史记录

git log --pretty=oneline //每条历史只显示一行，反馈结果包含commit id（版本号）
该版本号用一串16进制数表示，目的是为了解决多人开发造成版本号冲突。

git diff HEAD -- 文件名 //查看某个版本下某个文件修改的情况

git diff HEAD^ -- 文件名
//查看上个版本修改的内容

git diff 版本号 -- 文件名 //根据版本号查看修改内容

git checkout -- 文件名 //丢弃工作区的修改

git checkout . // 撤销工作区所有的修改

git rebase //把本地未push的分叉提交历史整理成直线

git add -f dist //强制提交被.gitignore忽略的文件

git branch -m dev develop //分支改名

cat README.md //查看文件内容

rm -rf .git //删除本地仓库

git push origin master -f //远程推送失败(强制推送)

git remote add origin git地址 //远程连接

git clone git地址 //克隆项目

git checkout 分支名 //切换分支

git branch 分支名 //创建新分支（必须置于commit之后）

git branch -d dev //删除本地分支

git push origin -d dev //删除远程分支

git config user.name //查看用户名

git config user.email //查看邮箱` 
```

### 将文件放入Git仓库(两步)：

```
`git add 文件名  /*将文件添加到暂存区*/

git commit -m "描述" /*将暂存区的所有内容提交至当前分支上*/` 
```

### 版本回退(不区分大小写)：

**前提：未推送到远程**

```
`当前版本：HEAD。

上一个版本：HEAD^。

上上个版本：HEAD^^。

往上100个版本： HEAD~100

//回退到上个版本
git reset --hard HEAD^` 
```

### 版本恢复：

恢复到回退前的版本就必须知道之前版本的版本号。通过git reflog查看你使用过的每一个命令：

```
`git reflog //可以获取之前版本的commit id

git reset --hard commit_id //恢复到之前的版本` 
```

### 撤消工作区修改：

1.  readme.txt自修改后还没有被放到暂存区，现在，撤销修改就回到和版本库一模一样的状态；
    
2.  readme.txt已经添加到暂存区后，又作了修改，现在，撤销修改就回到添加到暂存区后的状态
    

```
`git checkout -- readme.txt` 
```

### 撤销暂存区修改：

```
`git reset HEAD readme.txt // 此时回到工作区

git checkout -- readme.txt // 撤销工作区修改` 
```

### 删除版本库中的文件

```
`git rm 文件名

git commit -m "delete"

//误删文件恢复
git checkout -- 文件名` 
```

git rm相当于在工作区删除文件，因此使用git checkout可以撤销对工作区的操作，但文件只能恢复到最后一次提交前的状态。

# 附：

### 1.区别：

git commit -am 和git commit -m：

git commit -am 可以看成是git add 和git commit -m

git fetch和 git pull

git pull 等同于 git fetch和git merge

### 2.连接远程：

```
`git remote add origin git地址 //关联远程仓库

git push -u origin master //第一次推送master分支的所有内容

git push origin master //此后推送最新修改` 
```

### 3.分支操作：

```
`//创建分支并切换
git checkout -b dev

//等价于

git branch dev
git checkout dev

//查看当前分支
git branch

//合并分支
git merge dev

//删除分支（无法在当前分支下删除当前分支）
git branch -d dev` 
```

### 4.解决冲突

```
`//master分支修改readme内容
git checkout master
git add readme.txt
git commit -m 'modify'

//dev分支修改readme内容
git checkout dev
git add reaadme.txt
git commit -m 'modify'

//merge
git checkout master
git merge dev //出现冲突

1.打开冲突文件，会自动标出冲突位置，如：
<<<<<<< HEAD
Creating a new branch is quick & simple.
=======
Creating a new branch is quick AND simple.
>>>>>>> dev
手动修改此处的内容
2.git add readme.txt
3.git commit -m "merge"

//此时master分支已解决冲突，dev分支直接通过：
git merge master即可` 
```

### 5.Fast forward模式

合并分支时，如果可能，Git会用Fast forward模式，但这种模式下，删除分支后，会丢掉分支信息。

要强制禁用Fast forward模式，Git就会在merge时生成一个新的commit，这样，从分支历史上就可以看出分支信息。

使用–no-ff禁用Fast forward模式

```
`git merge --no-ff -m "merge with no-ff" dev
//此次合并生成一个新的commit，因此需要添加commit的描述` 
```

### 6.修复Bug分支

现在在dev分支上开发，但代码未提交，需要切换到master分支上去修复Bug。

修复bug时，我们会通过创建新的bug分支进行修复，然后合并，最后删除。

```
`git stash   //储存当前工作状态

git status  //working tree clean

git checkout master //创建临时分支解决冲突

git checkout -b temp //修改冲突

git add readme.txt

git commit -m 'fix bug'

git checkout master

git merge --no-ff -m 'bugfixed' temp

git branch -d temp //删除临时分支

git checkout dev //切换回原分支

git stash list //查看之前存储的工作状态

//状态恢复的两种方式：
1. git stash apply, 但stash内容并不删除，需要用git stash drop删除

2. git stash pop，恢复的同时把stash内容也删了

//可以多次stash，恢复的时候，先用git stash list查看，然后恢复指定的stash
git stash apply stash@{0}` 
```

### 7.删除feature分支

```
`git checkout -b feature/v1 //创建一个新分支

git add readme.txt

git commit -m "new" //添加新功能

git checkout master

//feature/v1分支多余，需删除

git branch -d feature/v1 //报错

git branch -D feature/v1 //强行删除即可` 
```

### 8.多人协作

远程仓库名称： remote

```
`git remote // 查看远程库信息

git remote -v //查看详细信息
origin  git@github.com:michaelliao/learngit.git (fetch)
origin  git@github.com:michaelliao/learngit.git (push)
//显示了可以抓取和推送的origin的地址。如果没有推送权限，就看不到push的地址。` 
```

### 9.创建标签

```
`git tag v1.0 //v1.0标签会打在最新的commit上

git tag //查看所有标签

git tag v0.9 f52c633 //针对具体的commit id打tag

git show v0.9 //查看具体的标签信息

git tag -a v0.1 -m "version 0.1 released" 1094adb //创建带有说明的标签，用-a指定标签名，-m指定说明文字

git tag -d v0.1 //删除标签

git push origin v1.0 //推送某个标签至远程

git push origin --tags //一次性推送全部尚未推送至远程的本地标签

//删除远程的tag
git tag -d v0.9 //先删除本地的tag
git push origin :refs/tags/v0.9` 
```

### 10.本地仓库与远程的gitLab和gitHub连接

```
`配置sshKey

git remote add gitlab 远程仓库名

git remote add github 远程仓库名

//非git remote add origin 远程仓库名

git remote -v //查看连接的远程仓库

//推送代码
git push gitlab master
git push github master` 
```
