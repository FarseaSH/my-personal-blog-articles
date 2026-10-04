---
date: 2021-11-07 10:36:24
title: 简明图示解释Git各类合并的区别
category: 想法与笔记
tags:
 - Git
updated:
copyright: true  # 暂不支持
toc: true
toc_fold: false
---

本文尝试用一系列图示解释各种Git合并的区别。

为了叙述方便，我们将被合并的分支称为**dev开发分支**，合并到的目标分支成为**master主分支**。

![master分支与dev分支合并前状态](https://z3.ax1x.com/2021/11/07/I1xKQf.png)

<!--more-->

## 常规Commit

常规的Merge方法，合并时候在master主分支产生新的合并commit。

![常规commit后的状态](https://z3.ax1x.com/2021/11/07/I1zuN9.png)

如果将开发分支删除，开发分支上的commit历史会保留。

![常规commit后删除dev分支后的状态](https://z3.ax1x.com/2021/11/07/I1zJBD.png)

## Fast-forward Commit

和Normal Commit一致，只是将指针移动到dev开发分支上，master主分支不会产生新的合并commit。

![Fastforward Commit合并后的状态](https://z3.ax1x.com/2021/11/07/I1xHfI.png)

如果将开发分支删除，开发分支上的commit历史会保留，但是在主分支呈现的，看起来就像在主分支进行提交commit一样。

![Fastforward合并后删除dev分支后的状态](https://z3.ax1x.com/2021/11/07/I1xqpt.png)

Fastforward Commit方式的执行是有条件的，如果master主分支与dev开发分支有无法解决的冲突，则不会进行 fast forward commit。

![上面情况无法执行fast-forward commit](https://z3.ax1x.com/2021/11/07/I1z9ts.png)

## Squash Commit

与常规 commit类似，不过合并时候，会将dev开发分支看作一次整体的commit合并到master主分支上，主分支会产生新的合并commit。

![Squash commit合并后的分支状态](https://z3.ax1x.com/2021/11/07/I1zfCn.png)

如果将dev开发分支删除，开发分支上的commit历史不会保留。

![Squash合并后删除dev分支后的分支状态](https://z3.ax1x.com/2021/11/07/I1zTDU.png)

## Rebase

前面三种都是Merge形式的合并，Rebase与Merge有本质的不同，Merge只会产生新的commit，不会对commit历史进行修改，但Rebase会对commit历史的部分链条进行修改。

通过Rebase合并到主分支时候，dev开发分支不会以Merge形式合并到主分支，而是在主分支上将dev开发分支的修改重新以若干个commit提交到主分支上。为了方便理解，给一个不恰当的比喻：将你同桌的考试解答，抄一遍到自己的试卷，然后以自己的名义上交。

![Rebase后的分支状态](https://z3.ax1x.com/2021/11/07/I3SuqS.png)

如果将dev开发分支删除，开发分支上的commit历史不会保留。

![Rebase后删除dev分支后的分支状态](https://z3.ax1x.com/2021/11/07/I3SG2q.png)

另外顺便一提，Rebase合并时候的commit数量是可以自定义的，你可以以与dev开发分支相同数量的commit提交到master主分支上，也可以以类似Squash提交的方面将dev开发分支的多个commit合在一起作为一个commit在master提交。甚至，你可以选择跳过dev开发分支中的某些commit。这些都在Rebase interactive模型中实现。

![在interactive rebase模式下可以自由设定合并方式](https://z3.ax1x.com/2021/11/07/I3SgsK.png)