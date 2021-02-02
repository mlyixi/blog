---
title: "ProGit摘要4-衍合"
date: 2014-10-28T13:18:52+08:00
tags: ["ProGit"]
categories: ["CS"]
toc: true
---

之前我们讨论了合并(merge),它会进行一次三头合并.
![Alt](/cs/18333fig0328-tn.png "merge to C5")

## 衍合
其实,我们还可以把C3里产生的补丁在C4中再打一遍,这种操作就叫衍合

```zsh
git checkout experiment
git rebase master
```

或

```zsh
git rebase [主分支] [特性分支]
```
![Alt](/cs/18333fig0329-tn.png "rebase")

确认后,你就可以将之快速合并了

![Alt](/cs/18333fig0330-tn.png "rebase")

那么你会问,有区别吗?衍合时也是需要你手动个性补丁的.但是比较两都的提交图,我们可以看到,衍合可以提供一个更为整洁的提交历史(线性).这也是使用衍合的目的,如果你不是维护者,那么最好用衍合.

## onto
其实,我们还可以将衍合放到其它分支
![Alt](/cs/18333fig0331-tn.png "提交图")
我们想着把c8和c9的补丁用于master,而跳过server支(c3,c4.c10)

```zsh
git rebase --onto master server client
```
![Alt](/cs/18333fig0332-tn.png "提交图")

那么后面如何衍合server呢?

## 特别注意
什么时候不能用衍合?

**一旦分支中的提交对象发布到远程仓库，就千万不要对该分支进行衍合操作**。

某人合并分支后并推送到远程仓库,此时我已经进行修改(c2,c3),然后我同步成c7
![Alt](/cs/18333fig0337-tn.png "注意事项")

此时某人决定不用合并而采用衍合(c4')并推送,而我又需要同步.此时我就打了两次c4补丁!!!
![Alt](/cs/18333fig0339-tn.png "注意事项")