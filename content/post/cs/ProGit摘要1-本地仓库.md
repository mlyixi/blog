---
title: "ProGit摘要1-本地仓库"
date: 2014-10-24T10:08:52+08:00
tags: ["ProGit"]
categories: ["CS"]
toc: true
---

看了几遍git方便的使用指南,但是由于不经常用,总是忘了,只记得最基本的clone,submodule之类的,完全不够用啊. 现在再看一篇经典著作《Pro Git》,并摘要如下.

书籍地址：<https://iissnan.com/progit/>

## 基本原理
不像其它CVS那样保存差异数据,Git直接记录快照.解释可以看下面两张图.

![CVS](/cs/CVS.png "CVS")
![GIT](/cs/GIT.png "GIT")

这样的好处是Git中大部分操作都只在本地而不用连网. 

## 本地仓库
### 文件状态
* 未跟踪

* 已跟踪
  * 已提交(committed):已保存到本地git
  * 已修改(modified):已修改了本地git中的某个文件
  * 已暂存(staged):把已修改的文件放到下次提交清单中

对应的,我们有三个工作区域:工作目录->暂存区域->本地仓库(.git)
对应的,git基本流程有
1. 在工作目录中修改某些文件
2. 对修改后的文件进行快照,然后保存到暂存区域
3. 提交更新,将保存在暂存区域的文件快照永久转储到Git目录中

### 配置
#### 用户信息

```zsh
git config [--global] user.name "mlyixi"
git config [--global] user.email mlyixi@example.com
```
其中global开关指定是全局还是单个项目的用户信息.

#### 编辑器

```zsh
git config --global core.editor vim
```

#### 差异分析

```zsh
git config --global merge.tool vimdiff
```
#### 查看配置

```zsh
git config --list
```


### 新建项目
#### 初始化

```zsh
git init
```

#### 克隆

```zsh
git clone git://uri [new name]
```

#### 查看状态

```zsh
git status
```

#### 跟踪/暂存文件
如未跟踪,则跟踪文件;如已修改,则暂存文件

```zsh
git add file
```
一般我们直接用`git add .`添加文件夹内所有文件.

#### 提交
```zsh
git commit -m 'message'
```
我们可以允许跳过暂存直接提交

```zsh
git commit -a -m 'message'
```
#### 忽略文件
在`.gitignore`文件中加入文件模式,如
```zsh
*.[oa]
*~
```
为什么要有忽略文件,因为有些文件是自动生成在当前目录的,但是我们一般都是*.*跟踪全部文件,然后跳过暂存直接提交,所以要指定忽略文件

#### 比较暂存和未暂存的信息/ 暂存和已提交的信息

```zsh
git diff [file name] [--staged]
```

#### 移除文件

```zsh
git rm [--cached] fileName/filePattern
```
cached指明删除跟踪但不删除文件.否则文件必须删除

#### 移动文件

```zsh
git mv file_from file_to
```
主要用于重全名

#### 查看提交日志

```zsh
git log
```
参数比较复杂,一般用`gitk`图形界面查看.

#### 撤消暂存

```zsh
git reset HEAD fileName
```

#### 撤消提交

```zsh
git commit -amend
```

#### 撤消修改

```zsh
git checkout -- fileName
```

## 分支
git的分支,其本质上仅仅是指向commit对象的可变指针. git会使用`master`作为分支的默认名字,而每次提交该指针就会自动向前移动. 同时它保存了一个名为`HEAD`的指针,指向我们工作目录中的当前分支.

### 分支基础

#### 创建分支

```zsh
git branch branceName
```

#### 切换分支

```zsh
git checkout branceName
```CVS

上面两个合起来可以为:

```zsh
git checkout -b branceName
```
#### 查看分支

```zsh
git branch [-v] [--merged] [--no-merged]
```

#### 删除分布

```zsh
git checkout -d branceName
```

#### 合并分支

```zsh
git merge branceName
```

### 分支应用

现在我们假设你已经熟悉了上面的命令,来看一个应用

#### 快速合并

我们经过3次提交并确认了一个问题iss53,如图：

![Alt](/cs/18333fig0311-tn.png '开始')
接着你决定解决这个问题,切换到这个分支,并进行一次提交
![Alt](/cs/18333fig0312-tn.png 'iss53-1')
结果也上你就接到电话有个bug要修复,所以你只能切换回master并新建一个分支hotfix进行修复(进行一次提交)
![Alt](/cs/18333fig0313-tn.png 'hotfix-1')
好了,bug修复完成,我们将hotfix合并到master里,并删除它.
![Alt](/cs/18333fig0314-tn.png 'hotfix-2')

#### 递归合并

现在我们假设iss53和hotfix完全无关,即修改的是不同文件,下面继续

紧急工作完成了我们还得接着干苦力--iss53.接着切换到这个分支进行修改并提交
![Alt](/cs/18333fig0315-tn.png 'iss53-2')

这时合并的时候出现问题:它有三个头了(之前只有两个头),即两个分支末端和它们最近的共同祖先.
![Alt](/cs/18333fig0316-tn.png 'iss53-3')

这时,git会进行一次三方合并,然后将结果重新做一个提交,并将分支指向这个提交
![Alt](/cs/18333fig0317-tn.png 'iss53-4')

嗯,这还好,git还没让我们伤脑筋

#### 冲突合并
不幸的来了,现在我们假设issue53和hotfix相关,即我们修改了同一个文件,那怎么办呢?
逻辑上来说,这种问题只能由人来解决--伤脑筋吧

这时我们合并iss53,它会提示:Automatic merge failed; fix conflicts and then commit the result.

这时你打开文件发现多了差异内容
```zsh
<<<<<<< HEAD
<div id="footer">contact : email.support@github.com</div>
=======
<div id="footer">
  please contact us at support@github.com
</div>
>>>>>>> iss53
```
其中等号上面的是master分支内容,下面是iss53内容,我们只有手动地修改这些内容并删除多余的行.当然,我们可以指定mergetool为我们准备图形化工具解决冲突.

然后就是提交了