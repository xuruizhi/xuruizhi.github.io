---
layout: post
title:  "Step by step: 参与 GitHub 上的开源项目"
date:   2021-04-10 15:00:00 +0800
categories: programming
---

[Step-by-step guide to contributing on GitHub](https://www.dataschool.io/how-to-contribute-on-github/)


参与开源项目

## fork 项目

## 在项目 fork 好之后，将其拉取到本地
$ git clone ssh://git@git.huawei.com:2222/x00505716/HTS.git


## 添加上游仓库，并使本地仓库与上游仓库保持同步

### 添加上游仓库
$ git remote add upstream ssh://git@git.huawei.com:2222/HMS_Core_SPDT/HMSTools/HTS.git

### 上游仓库添加好之后，之后都可以通过以下命令来使本地仓库与上游仓库保持同步

# 切换分支
$ git checkout master
# 更新上游代码
$ git fetch upstream
# 合并代码到 master 分支
$ git merge upstream/master
# 上传代码
$ git push

## 开发

### 新建开发分支
$ git checkout -b xxx

在新分支上进行代码修改

$ git push

### 提交 PR


### Review

### 删除开发分支
$ git branch -d xxx
