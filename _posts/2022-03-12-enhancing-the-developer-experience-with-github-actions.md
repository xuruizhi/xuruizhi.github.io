---
layout: post
title:  "使用GitHub Actions提升开发者体验"
date:   2022-03-12 15:40:00 +0800
categories: "software_testing"
---

## 背景

身处一个致力于构建全球顶级应用生态的组织，作为其中的一名SDK测试人员，肩负着验证N多示例Demo的重任。这些Demo同时发布在 GitHub/Gitee上，是开发者了解产品的门户，其功能是否可用在很大程度决定了开发者对产品的第一印象。

### 业务痛点

通过分析历史VOD(Voice Of Developer)问题发现：

1. 有不少的Demo用户下载后居然**编译报错**；
2. 提供的指导文档README.md里面，存在**URL失效**；
3. GitHub和Gitee的**代码不同步**，有很多代码问题，在GitHub上已修复，Gitee上还仍然存在；反之亦然。

前期这部分的测试看护，纯由人工完成，人工下载代码、人工验证，难免疏漏。在2022年还用这种人拉肩扛的方式，是可忍孰不可忍，故有此文。

## 示例

[github-action-test](https://github.com/xuruizhi/github-action-test)

## GitHub Action

GitHub Actions 是 GitHub 的**持续集成**服务，于2018年10月推出。

### 基本概念

![github-actions](/images/2022.03.12/github-actions.png)

1. workflow （工作流程）：持续集成一次运行的过程，就是一个 workflow。
2. job （任务）：一个 workflow 由一个或多个 jobs 构成，含义是一次持续集成的运行，可以完成多个任务。
3. step（步骤）：每个 job 由多个 step 构成，一步步完成。
4. action （动作）：每个 step 可以依次执行一个或多个命令（action）。

## 解决方案

利用GitHub Action，构建Demo的持续集成能力，解决[上述3个痛点问题](#业务痛点)。

### 添加 Android CI，在 push 和 pull_request 到 master 分支时，触发编译检查+URL有效性检查

#### 添加 build job

```YAML
name: Android CI

on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ master ]
  pull_request:
    branches: [ master ]
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: set up JDK 1.8
      uses: actions/setup-java@v2
      with:
        java-version: '8'
        distribution: 'temurin'
        cache: gradle
    - name: Grant execute permission for gradlew
      run: chmod +x ./gradlew
    - name: Build with Gradle
      run: ./gradlew app:assemble
    - name: show apk file
      run: cd ./app/build/outputs/apk/release; ls -l
```

#### 添加 markdown-link-check job

```YAML
jobs:
  markdown-link-check:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - uses: gaurav-nelson/github-action-markdown-link-check@v1
      with:
          use-verbose-mode: 'yes'
```

### 添加 mirror-to-gitee workflow，在 push 和 pull_request 到 master 分支时，触发代码同步到 Gitee

```YAML
name: mirror-to-gitee

on:
  # Triggers the workflow on push or pull request events but only for the main branch
    push:
    pull_request:
        branches: [master]
  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

jobs:
  run:
    name: Run
    runs-on: ubuntu-latest
    steps:
    - name: Checkout source codes
      uses: actions/checkout@v2
    - name: Mirror the Github organization repos to Gitee.
      uses: Yikun/hub-mirror-action@v1.2
      with:
        src: github/xuruizhi
        dst: gitee/xuruizhi
        dst_key: ${{ secrets.GITEE_PRIVATE_KEY }}
        dst_token: ${{ secrets.GITEE_TOKEN }}
        white_list: github-action-test
```

## 参考

[GitHub Actions 入门教程](https://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html)

[Getting started with GitHub Actions](https://itnext.io/getting-started-with-github-actions-fe94167dbc6d#8640)
