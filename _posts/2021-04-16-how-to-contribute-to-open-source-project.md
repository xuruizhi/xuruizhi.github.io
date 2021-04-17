---
layout: post
title:  "Step by step: 参与开源项目开发"
date:   2021-04-16 15:00:00 +0800
categories: skill
---

本文档以在 GitHub 上参与 [NullAway项目](https://github.com/uber/NullAway) 为例，介绍如何参与开源项目开发。其余代码托管网站的操作都类似。

## 1. Fork the project repository - Fork 需要参与的 项目仓
Fork: 分叉、派生。为什么需要 Fork? 因为对于非本人的仓库，没有权限做任何修改，需要基于`项目仓`分叉出自己的`Fork仓`，在Fork仓上修改后，提交 PR(Pull Request)，经项目维护者Accept PR后，代码才会合入`项目仓`。

在 项目仓 https://github.com/uber/NullAway 页面
点击右上角的 `Fork` 按钮，即可得到本人的 Fork 仓: https://github.com/xuruizhi/NullAway 。
![fork-on-github](/images/2021.04.16/fork-on-github.png)

## 2. Clone your fork - 克隆 Fork 仓 到本地
`$ git clone git@github.com:xuruizhi/NullAway.git`

查看远程仓库，可得：
```
$ git remote -v
origin  https://github.com/xuruizhi/NullAway.git (fetch)
origin  https://github.com/xuruizhi/NullAway.git (push)
```

可以看到，`Fetch 仓`被设置为当前本地仓库关联的远程仓，命名为默认值 origin。

## 3. Add the project repository as the "upstream" remote - 添加`项目仓`关联，并命名为 upstream

`$ git remote add upstream git@github.com:uber/NullAway.git`

查看远程仓库，可得：
```
$ git remote -v
origin  https://github.com/xuruizhi/NullAway.git (fetch)
origin  https://github.com/xuruizhi/NullAway.git (push)
upstream        https://github.com/uber/NullAway.git (fetch)
upstream        https://github.com/uber/NullAway.git (push)
```

上述 1-3 流程如下图所示：

![fork_on_github](/images/2021.04.16/diagram-01.png)

- 项目 仓（远程）：Project repository on GitHub
- Fork 仓（远程）：Your fork on GitHub
- 本地 仓：Your local repository

## 4. Pull the latest changes from upstream into your local repository - 拉取 项目仓 最新代码到 本地仓

在本地开发之前，需要保证本地代码与远程代码的一致性，以减少代码冲突。
拉取`项目仓`master分支最新代码到`本地仓`。
`$ git pull upstream master`

## 5. Create a new branch - 新建开发分支并切换到新分支
`$ git checkout -b fixbugs`

## 6. 在新分支上进行开发
代码修改后：

```
$ git add -A                # 添加所有修改文件到暂存区
$ git commit                # 提交修改，并设置提交信息
$ git push origin fixbugs   # 推送到远程 Fork 仓的 fixbugs 分支
```

## 7. Create the pull request - 提交 PR

![fork_on_github](/images/2021.04.16/pull-request-on-github.png)

上述 4-7 流程如下图所示：

![fork_on_github](/images/2021.04.16/diagram-02.png)

## 8. Review the pull request - 审查 PR

项目维护者会在线审查提交的PR，如果符合要求，则直接合入。
如果不符合，需要根据反馈意见进行修改，重复 步骤 6 直到符合。

> 在 PR close 前都可以添加变更。

## 9. 删除开发分支

Delete your branch from your fork - 删除远程分支

```
$ git push origin --delete fixbugs
To https://github.com/xuruizhi/NullAway.git
 - [deleted]         fixbugs
```

Delete your branch from your local repository - 删除本地分支

```
$ git checkout master
$ git branch -d fixbugs
```

## 10. Synchronize your fork with the project repository - 更新 Fork 仓

提交的代码变更只合入了`项目仓`，这就造成了`本地仓`、`Fork 仓`和`项目仓`的不同步。

### 切换到 master 分支
switch to the `master` branch:
`$ git checkout master`

### 更新 upstream 的 master 分支最新代码到本地仓库
pull the latest changes from `upstream` (the project repository) into your `local repository`
`$ git pull upstream master`

### 上传本地仓代码到 origin 的 master 分支
push those changes from your local repository to the "origin" (your fork)
`$ git push origin master`

## References
[Step-by-step guide to contributing on GitHub](https://www.dataschool.io/how-to-contribute-on-github/)