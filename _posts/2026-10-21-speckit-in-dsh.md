---
layout: post
title: "在 DSH 里用 Spec Kit：从空目录到 46 个测试全绿的实测记录"
subtitle: "constitution → specify → plan → tasks → implement 五步走完，附 1.2.2 的安装、清单与三个真实缺陷"
date: 2026-10-21 13:00:00 +0800
categories: notes
tags: [AI编程, 规格驱动开发, 工程实践]
description: "Spec Kit 1.2.2 原生支持 DSH，集成标识是 dsh。本文记录实测跑通的一遍：装好之后生成什么结构、五步各产出什么，以及 45 个任务、46 个测试里暴露出的三个真缺陷。"
---

Spec Kit 做的事很朴素：**把「想清楚再写」变成一套有步骤、有产物、可以回头对照的流程。** 这次实测的版本是 1.2.2，它已经带 CLI、带技能包、带扩展体系，并且**内置了 DeepSeek Harness 的官方集成，标识就叫 `dsh`**。

所以接入不用绕路：`specify init --integration dsh` 一条命令，`.dsh/skills/` 下直接生成十个 `speckit-*` 技能，DSH 打开这个目录就能用。

光看结构说明不了什么。我用一个「待办清单 CLI 小工具」把五步全走了一遍：从一个空目录，到 45 个任务全部实现、46 个测试全部通过。**这篇记的就是那一遍的实测过程**——生成什么、每一步产出什么、以及哪些地方是真的会咬人的。

## 一、动手前的三条

正式开工之前，先把这三件事确认掉，比装完再排查省事得多。

| 检查项 | 怎么做 | 为什么 |
| --- | --- | --- |
| 集成是否可用 | `specify check`，列表里找 `DeepSeek Harness` | 有这一项就不用自己搓适配层 |
| CLI 版本 | `specify --version` | 1.2.2 已实测通过 |
| 依赖工具 | `uv`、`git`、Python 3.11+ | 装 CLI 用 `uv`；指定技能要先被 DSH 看到 |

`specify check` 在某些受限环境里会显示 `(not found)`，那通常只是探测可执行文件路径失败，不影响 `--integration dsh` 正常工作。

这次实测用的版本：

| 组件 | 版本 |
| --- | --- |
| `uv` | 0.12.5 |
| `specify` | 1.2.2 |
| Python | 3.11+（示例项目的要求） |
| `pytest` | 9.1.1 |

## 二、安装与初始化

```powershell
# 1 安装（已发布到 PyPI）
uv tool install specify-cli      # → specify 1.2.2

# 2 在项目目录初始化，关键是 --integration dsh
cd 你的项目目录
specify init . --integration dsh --ignore-agent-tools --non-interactive
```

初始化输出三行，可以直接判断有没有装对：

```text
Selected coding agent integration: dsh
├── ● Install integration (DeepSeek Harness)
├── ● Install shared infrastructure (scripts (ps) + templates)
└── ● Finalize (project ready)
```

生成的结构：

```text
你的项目/
├── .dsh/skills/              10 个技能，DSH 从这里加载
│   ├── speckit-constitution/
│   ├── speckit-specify/
│   ├── speckit-plan/
│   ├── speckit-tasks/
│   ├── speckit-implement/
│   ├── speckit-converge/
│   ├── speckit-clarify/
│   ├── speckit-analyze/
│   ├── speckit-checklist/
│   └── speckit-taskstoissues/
├── .specify/
│   ├── memory/constitution.md      项目章程
│   ├── templates/                  spec / plan / tasks / checklist / constitution 五套模板
│   ├── scripts/powershell/         本机自动选中的 PowerShell 版本
│   └── workflows/speckit/          捆绑的工作流
├── specs/                          之后每一步的产物落在这里
└── AGENTS.md
```

十个技能分工如下，前面六个是主链路和它的收尾，后面四个按需插入：

| 技能 | 作用 | 什么时候用 |
| --- | --- | --- |
| `speckit-constitution` | 定项目原则 | 只做一次 |
| `speckit-specify` | 写功能规格（What / Why） | 每个功能一次 |
| `speckit-clarify` | 消除规格里的歧义 | plan 之前，可选 |
| `speckit-plan` | 技术选型与设计 | 紧跟 specify |
| `speckit-checklist` | 生成质量检查清单 | plan 之后，可选 |
| `speckit-tasks` | 拆成原子任务 | 紧跟 plan |
| `speckit-analyze` | 查 spec / plan / tasks 三者一致性 | tasks 之后，可选 |
| `speckit-implement` | 按任务清单实现 | 按 tasks 推进 |
| `speckit-converge` | 找没落实的部分，追加任务 | 收尾，反复跑到 `Converged` |
| `speckit-taskstoissues` | 任务转 GitHub Issue | 要和多人在线协作时 |

### 2.1 怎么在 DSH 里用

**最容易踩的一点：技能只从「当前工作区」的 `.dsh/skills` 加载。** 如果 DSH 开的是别的目录，初始化生成的那十个技能不会出现在补全列表里——不是坏了，是没看到。

| 步骤 | 做法 |
| --- | --- |
| 1 | 用 DSH 打开 `specify init` 的那个目录作为工作目录 |
| 2 | 输入框里敲 `/`，列表里应出现 `speckit-*` 十项 |
| 3 | 按顺序执行，每一步看完产物再往下 |

主链路：

```text
/speckit-constitution     只做一次，定项目原则
/speckit-specify  <功能描述>
/speckit-plan     <技术选型>
/speckit-tasks
/speckit-implement
/speckit-converge         反复 implement + converge 直到报 Converged
```

### 2.2 两条动手前就该知道的注意

**① `.dsh/` 不要提交进 git。** `specify init` 结束时会专门提示这里可能包含凭证。`.gitignore` 里加一行即可；`AGENTS.md` 和 `.specify/` 可以正常提交。

**② PowerShell 执行策略会挡脚本。** `.specify/scripts/powershell/` 下的脚本默认跑不起来。用 `-ExecutionPolicy Bypass` 只对这一次调用放行，不改系统设置：

```powershell
powershell.exe -NoProfile -ExecutionPolicy Bypass -File .\.specify\scripts\powershell\setup-plan.ps1
```

这一步是每个步骤技能内部都要先做的（技能定义里写死了要先跑模板解析脚本），不处理就会卡在第一步。

## 三、五步走一遍：一个待办清单 CLI

用一个「待办清单 CLI 小工具」贯穿，这个规模刚好够看清每一步的产出形态。

```
/speckit-constitution
  └─ .specify/memory/constitution.md      5 条不可协商原则
/speckit-specify
  └─ specs/001-todo-cli/spec.md           4 用户故事 / 13 FR / 6 SC / 7 边界
/speckit-plan
  └─ specs/001-todo-cli/plan.md           技术背景 / 宪法门禁 / 项目结构
     research.md / data-model.md / contracts/cli.md / quickstart.md
/speckit-tasks
  └─ specs/001-todo-cli/tasks.md          45 个可执行任务
/speckit-implement
  └─ src/ + tests/                        46 个测试全绿
```

### 3.1 constitution：先定规矩

第一步产出的 `constitution.md` v1.0.0 有五条原则，全部是「不可协商」级别的：

| 原则 | 核心约束 |
| --- | --- |
| I. Simplicity First | YAGNI，抽象被用到两次才引入，不为不存在的场景加配置 |
| II. Text I/O Contract | stdout 出结果、stderr 出诊断，每个命令必须有 JSON 模式，退出码有意义 |
| III. Test-First | 先写测试 → 确认失败 → 再实现 |
| IV. Local-First Persistence | 单一本地文件，任何命令不许联网，格式可人工恢复 |
| V. Deterministic Output | 相同输入产生字节级相同的输出，排序必须有文档 |

后面每一步都要回头对着这五条做一次 **Constitution Check**，不过就填复杂度追踪表说明理由。**这是 SDD 和「直接让 AI 写代码」最大的区别**——先有一份可以拿来说「这违反第三条」的纸面依据。

### 3.2 specify：只写 What 和 Why

`spec.md` 的核心是**按价值排序、且各自可独立测试**的四个用户故事：

| 优先级 | 故事 | 独立测试方式 |
| --- | --- | --- |
| P1 | 记住一件要做的事 | 添加 → 列出 → 确认存在 |
| P2 | 看清还剩什么 | 混合状态下列表，验证顺序和状态 |
| P3 | 标记完成 / 撤销 | 标记完成 → 再标回未完成 |
| P4 | 删除不再需要的 | 加两条删一条，验证另一条仍在 |

为什么这么排，理由只有一句：**P1 单独实现就是一个能用的 MVP**——「捕获」是唯一没有替代品的动作，而删除属于便利功能，可以先缺。**把删除排到 P4，第一版就不必面对「删除后标识符复用」的坑。**

再往下是 13 条功能需求（FR-001~013）和 6 条成功标准（SC-001~006），全部量化、且技术无关：

| 编号 | 内容 |
| --- | --- |
| FR-002 | 每条任务有稳定唯一标识符 |
| FR-010 | 必须有机器可读输出 + 非零退出码 |
| FR-011 | 输出确定性 |
| FR-013 | 存储故障必须上报，不能静默 |
| SC-001 | 5 秒内完成一次捕获 |
| SC-002 | 200 条任务 1 秒内列出 |
| SC-003 | 相同数据两次列出，输出**字节级一致** |
| SC-006 | 并发调用不丢、不重复 |

此外还有 7 类边界情况写进了规格：含空格 / 引号 / 标点的描述、超长描述不破坏排版、同名任务视为不同任务、首次使用没有存储文件、存储损坏时明确报错不静默丢数据、并发调用不损坏数据。

**值得单独说一句：整份 spec 里没有一个技术名词。** 没有 Python、没有 SQLite、没有 argparse——这些全被推到 `plan`。这份文档拿给不写代码的人看也是通的，这正是 SDD 想要的：**先定 What 和 Why，再定 How。**

`specify` 技能还会跑一轮模板质量校验，实测 16 项检查一次通过，没有 `[NEEDS CLARIFICATION]` 残留。

### 3.3 plan：这里才选技术

最终选型：Python 3.11+、`argparse`、`sqlite3`、`pytest`（仅开发依赖）——**运行时零第三方依赖**。

`research.md` 里记了 8 项技术决策，其中三条直接关系到后面有没有 bug：

| 决策 | 理由 |
| --- | --- |
| 用 SQLite 而不是 JSON 文件 | SC-006 要求「并发调用不丢不重复」。JSON 方案得自己实现文件锁，而 Windows 的 `msvcrt.locking` 和 POSIX 的 `fcntl` 语义不同，直接违反「不得有平台相关分支」 |
| 显式用 `AUTOINCREMENT` | SQLite 默认的 `INTEGER PRIMARY KEY` 在删除最大行后会**复用** ID，用户引用旧编号时会命中一条无关的新任务，破坏 FR-002 的「稳定唯一」 |
| 排序带 `id` 兜底 | 只按 `created_at` 排序时，同一秒创建的多条任务顺序未定义，违反原则 V |

计划阶段除 `plan.md` 之外还产出三份：

| 文件 | 内容 |
| --- | --- |
| `research.md` | 8 项技术决策，每项含理由与备选方案 |
| `data-model.md` | 实体、状态机、校验规则、SQL schema |
| `contracts/cli.md` | 5 个命令的完整接口契约 |
| `quickstart.md` | 11 个端到端验证场景 |

设计完还要**再跑一次 Constitution Check**，对照五条原则逐条给结论。实测五条全过：四个命令共用一个薄存储层（I）、stdout/stderr 分离且 0/1/2 三级退出码（II）、任务按 TDD 排序（III）、SQLite 单文件零网络（IV）、`created_at` + `id` 双键排序（V）。

### 3.4 tasks：拆到「另一个执行者不用问也能做」

`tasks.md` 45 个任务，按阶段分布：

| 阶段 | 任务数 | 说明 |
| --- | --- | --- |
| 环境搭建 | 4（T001–T004） | 目录、`pyproject.toml`、`.gitignore`、测试 fixture |
| 基础层 | 11（T005–T015） | 模型、存储、输出、CLI 骨架，**阻塞所有故事** |
| US1（P1）🎯 | 5（T016–T020） | MVP |
| US2（P2） | 6（T021–T026） | 列表与筛选 |
| US3（P3） | 6（T027–T032） | 标记完成与撤销 |
| US4（P4） | 5（T033–T037） | 删除 |
| 收尾 | 8（T038–T045） | JSON 契约、边界、并发、故障、README、性能 |

统计：`[P]` 可并行标记 21 个（46%），含明确文件路径的 40 个（89%），剩下 5 个是「执行验证」「宪法复查」这类本来就没有文件的任务。

任务格式是固定的：`- [ ] T0xx [P?] [USx] 描述 + 路径`。而且任务描述里是**原文引用约束**，不是让实现者自己判断：

```text
T008 建表……必须使用 AUTOINCREMENT（防止标识符复用，影响 FR-002）
T009 WAL + BEGIN IMMEDIATE + busy_timeout=5000（见 research.md 决策 4）
```

因为宪法的第三条要求测试先行，任务清单也严格按 TDD 排序——每个故事都先有「写测试」任务（明确要求**先确认失败**），才有实现任务：

```text
T027 [P] [US3] 编写集成测试：标记完成后再列出显示为已完成...   ← 先写测试
T028 [P] [US3] 编写集成测试：引用不存在的标识符时退出码 1...    ← 先写测试
T029     [US3] 编写幂等性测试...                              ← 先写测试
T030     [US3] 实现 set_state(task_id, new_state)             ← 后写实现
T031     [US3] 实现 done 子命令
T032     [US3] 实现 undo 子命令
```

**仅完成阶段 1+2+3 共 20 个任务，就是可交付的 MVP**——能添加、能列出、能持久保存。后面三个故事在此基础上叠加，每个都不破坏前面的。

### 3.5 implement：46 个测试全绿，以及三个真 bug

| 项目 | 数值 |
| --- | --- |
| 源码 | 472 行（7 个文件） |
| 测试 | 634 行（8 个文件，46 用例） |
| 测试结果 | **46 passed** |
| 任务完成 | **45 / 45** |
| 性能 | 列出 200 条任务平均 **89.6 ms**（SC-002 要求 < 1000 ms） |

比「全绿」更值得记的是**三个在真实运行里暴露出来的缺陷**，测试没抓到，跑起来才抓到：

**① `--json` 的位置契约矛盾。** 测试报 2 个失败。`--json` 是全局选项，argparse 要求它写在子命令**之前**（`todo --json list`），但 `contracts/cli.md` 写的是 `todo list --json`。按契约「`--json` 可与其他任意命令组合」，正确解法是**让子命令也接受它**，两种写法都能用。

**② 目录创建失败被报成数据库损坏。** `storage.py` 一开始把 `mkdir` 和 `sqlite3.connect` 放在同一个 `try` 块里，权限错误和「目录还不存在」混成一条「存储错误」。拆开以后，两种失败给两种提示。这个问题是在跑 `quickstart.md` 的验证场景时暴露的——单元测试覆盖不到。

**③ GBK 控制台崩溃——这个最严重。**

```text
UnicodeEncodeError: 'gbk' codec can't encode character '\u2713' in position 1
```

`[✓]` 在中文 Windows 的 GBK 控制台下直接抛异常。**而 46 个测试一个都没抓到它**，因为 `conftest.py` 里显式设了 `PYTHONIOENCODING=utf-8`，把默认编码问题捂住了。修法分两层：stdout / stderr 设 `errors="replace"` 兜底，编码不支持时降级成 `[v]`。

**这三条是这次流程里最有价值的部分：测试全绿不等于真的能用，脱离测试环境的那次真实运行才暴露出来。**

## 四、已知限制：这套流程不保证什么

写到这里容易滑向「SDD 真好」，所以这一节把不能指望的东西列清楚。

| 限制 | 说明 |
| --- | --- |
| 文档质量靠执行者 | `specify` CLI 只负责生成模板和技能文件，`spec.md` 写得好不好、FR 有没有漏，全是执行者的事，CLI 不校验 |
| 只有模板内置于 CLI | `specify init` 无需联网，但 `specify extension add`、`self upgrade` 要联网，网络不通就用不了扩展 |
| 技能不跨目录加载 | `.dsh/skills` 只从当前工作区读，换工作目录等于这些命令不存在 |
| 测试先行靠自觉 | 这次第一组测试就没先失败：`models.py` 先写完才跑测试，直接 `5 passed`，违反原则 III。后面 storage / output / cli 才是严格先红后绿 |
| `converge` 才是完成判定 | 按宪法，只有 `converge` 报 `Converged` 才算功能完成。这次走到 `implement` 就停了，严格说还没到「完成」 |
| 示例项目不等于真项目 | 这份 constitution 是按「待办 CLI」的合理默认值写的，不是谁的偏好。做真项目时应该整体重写 |
| 规模是刻意的 | 待办 CLI 是示范用的，看清流程靠它，评价 SDD 在真实项目里的收益不能靠它 |

还有一点要如实交代：**pytest 是用国内镜像装的**，直连下载卡住了。依赖的获取方式和你构建脚本里写的不一样，值得知道。

## 五、一些数字

| 维度 | 数字 |
| --- | --- |
| 工具版本 | uv 0.12.5 · specify 1.2.2 · pytest 9.1.1 |
| Spec Kit 技能 | 10 个 |
| spec 文件 | 4 用户故事 / 13 条 FR / 6 条 SC / 7 类边界 |
| 计划产物 | plan / research / data-model / contracts / quickstart 五份 |
| 任务 | 45 个，`[P]` 并行 21 个（46%），带路径 40 个（89%） |
| 代码 | 源码 472 行 + 测试 634 行 |
| 测试 | 46 个，全通过 |
| 性能 | 列出 200 条 89.6 ms |
| 修掉的真缺陷 | 3 个，其中 1 个是被测试环境掩盖掉的 |

## 六、给下一次的自己

三条。

**第一，Spec Kit 的门槛已经很低了，接入只有两条命令。** 官方原生支持 DSH，`--integration dsh` 一条命令，`.dsh/skills/` 里十个技能开箱可用。真正需要动手的只有两件小事：**用 DSH 打开项目目录**（不然技能加载不到），**`.dsh/` 加进 `.gitignore`**（init 自己也会提示）。

**第二，SDD 的价值不在「AI 写出更多代码」，在于把判断提前。** 这次 45 个任务里，最省时间的不是实现阶段，是 `research.md` 里那条「用 SQLite 不用 JSON」和 `data-model.md` 里那条「必须 `AUTOINCREMENT`」——它们把一个隐蔽的 bug（删了任务再加，旧编号指向新任务）挡在了写代码之前。**先定 What 和 Why，再定 How；规矩写在纸上，才有「你违反了第三条」这句话可说。**

**第三，全绿的测试是不可信的，尤其是环境被设得太干净的时候。** 46 个测试全过，真实运行第一下就崩在 GBK 编码上。以后写测试，conftest 里**不要**替程序把环境变量抹平——那等于把「换台机器就崩」的位置提前藏起来。
