---
layout: post
title:  "看懂<代码覆盖率报告>"
date:   2023-02-28 20:00:00 +0800
categories: "software_testing"
---

## 定义

[代码覆盖](https://en.wikipedia.org/wiki/Code_coverage)（英语：Code coverage）是软件测试中的一种度量，描述程序中源代码被测试的比例和程度。通常，它有4个测量维度：

- 语句覆盖率（statement coverage）：程序中的每条语句都执行了吗？
- 分支覆盖率（branch coverage）：每个控制结构（例如if语句）的每个分支都执行了吗？
- 函数覆盖率（function coverage）：程序中的每个函数都被调用了吗？
- 行覆盖率（line coverage）：是否每一行都执行了？

> Istanbul 是 JavaScript 程序的代码覆盖率工具，本文以它输出的代码覆盖率报告为例，介绍这4种覆盖度量方式的具体算法。

---

## 代码覆盖率算法

### 示例代码

```JavaScript
function add(a, b) {
    if ((a + b) > 2) { console.log('a plus b is greater than 2'); }
}

function sub(a, b) {
    if ((a - b) > 2) {
        console.log('a minus b is greater than 2');
    }
}

add(0, 1);
```

### 使用 Istanbul 输出覆盖率报告

```Shell
$ istanbul cover cov.js
```

![覆盖率](/images/2023.02.28/cov.png)

### 说明

- 语句(statement): 指编程语句，如赋值语句、控制语句、返回语句等
- 分支(branch): 指控制结构的某个执行路径，如if语句中条件为true的路径即为1个分支
- 函数(function): 代码中定义的函数
- 行(line): 指可执行代码行(Lines of Executable Code)，而不是源文件中所有的代码行

#### 语句覆盖率

语句覆盖率 = 已覆盖的语句个数/语句总数 = 4/7 = 57.14%

> 特别注意：代码第2行，虽然书写上来说是1行，但是是2条语句（1条控制语句，1条函数调用语句）

#### 分支覆盖率

分支覆盖率 = 已覆盖的分支个数/分支总数 = 1/4 = 25%

> 2个if语句，则共有4个分支。

#### 函数覆盖率

函数覆盖率 = 已覆盖的函数个数/函数总数 = 1/4 = 25%

#### 行覆盖率

行覆盖率 = 已覆盖的可执行代码行数/可执行代码行总数 = 4/6

---

## 应用

- 对于质量要求不高的程序，使用语句覆盖率进行评估即可。
- 对于质量要求非常严格的程序，则需要使用分支覆盖率，甚至条件覆盖率、修改条件/判断覆盖([MCDC](https://en.wikipedia.org/wiki/Modified_condition/decision_coverage))进行评估，这里就不一一赘述了。

## 参考

[代码覆盖率工具 Istanbul 入门教程](https://www.ruanyifeng.com/blog/2015/06/istanbul.html)

[看懂「测试覆盖率报告」](https://github.com/JChehe/blog/issues/49)

[Code coverage](https://en.wikipedia.org/wiki/Code_coverage)
