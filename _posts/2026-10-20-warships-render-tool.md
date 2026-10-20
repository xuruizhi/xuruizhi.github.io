---
layout: post
title: "用 AI 造一个 World of Warships（战舰世界）渲染工坊：功能详解与部署指南"
subtitle: "从游戏客户端直出 4K/8K/16K 舰船图，一个双击就能运行的本地网页"
date: 2026-10-20 23:00:00 +0800
categories: ai-assisted-game
tags: [AI编程, 游戏工具, 工程实践]
description: "先按功能拆解一个本地舰船渲染工具：795 艘船筛选、6 视角、4K 到 16K 出图、命令行批量；再讲本地部署与已知限制，最后简述 AI 构建过程。"
---

儿子最近迷上了《战舰世界》。他会趴在屏幕前，指着某艘船说「这个是俾斯麦，它的炮塔能转」，然后转过头问我：爸爸，这张图能不能放大一点？

游戏里当然能看，但看不久——镜头被限制，画面里挤着 HUD、队友和一片晃动的海。想给他一张干干净净的大图，当科普卡片也好，当壁纸也好，都不太容易。

于是有了这个小工具：从游戏安装目录里读出舰船 3D 模型，用 Blender Cycles 渲染成高清 PNG。**选一艘船，点一下，图就出来了。** 本机运行，没有账号、没有云服务、不上传任何文件。双击一个 `bat`，浏览器打开 `http://127.0.0.1:8770`，几分钟到十几分钟后，图到手边。

![渲染工具界面](/images/2026.10.20/ui.png)

*左栏筛选与参数，右栏实时日志与出图。页面估算「约 1.3 小时」是因为默认勾了全部 6 个视角 + 4K，改小就快了。*

> 说明：本文是**工具说明书**，重点在功能与部署。关于 AI 在其中做了什么，见文末第五节的简述。
>
> 代码已开源：[github.com/xuruizhi/wows-render](https://github.com/xuruizhi/wows-render)，部署见第三节。

## 一、它解决什么问题

游戏里的战舰模型是 `.geometry` 网格 + `.dds` 贴图，存在 200 多个打包文件里，肉眼看不到、也截不出「干净」的图——进游戏截图会有 HUD、队友、海况和镜头限制。

这个工具做的事，是把这条链路自动化：

```
游戏安装目录
   │  wows-gltf-exporter      读出网格 + 贴图，拼成 .glb
   ▼
舰船模型 (glb)
   │  sanitize_glb.py         剔除空 mesh，让 Blender 能导入
   ▼
可渲染模型
   │  Blender + Cycles        自动取景、物理天空、程序化海面、大气雾
   ▼
WoWs_Renders/<国家>/<类型>/Tier <级别>/<战舰Key>/<视角>.png
```

命令行当然也能做，但每次都要敲参数、算路径、看日志。工具把这条链路包进了一个网页，**参数是选的，路径是自动的，进度是实时的。**

## 二、功能

### 2.1 选船：795 艘，全中文

左栏三组下拉框依次是**国家 → 舰种 → 级别**，下面第四个是战舰列表，显示成：

```
俾斯麦 Bismarck  ·  VIII级 战列舰  ·  D系 Germany
```

当前清单覆盖 **795 艘**：

| 国家（14） | Japan 130 · USA 121 · United_Kingdom 91 · Germany 85 · Russia 69 · France 59 · Pan_Asia 46 · Europe 43 · Events 41 · Italy 39 · Commonwealth 22 · Netherlands 18 · Pan_America 18 · Spain 13 |
| --- | --- |
| 舰种（6） | 巡洋舰 287 · 战列舰 197 · 驱逐舰 196 · 航空母舰 60 · 潜艇 37 · 辅助舰 18 |
| 级别 | I ~ X 级，外加 ★ 级（超级战舰）25 艘 |

中文名不是手写映射表，而是从游戏自带的 gettext 翻译文件里提取的（见 2.5）。游戏版本更新后，重跑一次 `build_catalog.py` 就能刷新清单。

### 2.2 视角：6 个机位 + 一键全选

| 视角 | 中文名 | 机位说明 |
| --- | --- | --- |
| `hero` | 主视角 | 三分之四艏侧，最常用的展示角 |
| `side` | 侧舷 | 正侧面试，看舷侧细节与涂装 |
| `stern` | 艉侧 | 三分之四艉侧 |
| `bow` | 艏部 | 正艏低角，有冲击力 |
| `top` | 俯视 | 高角看甲板布置 |
| `low` | 低角 | 极低角，贴近水线 |

「全部视角」是一个总开关，点一下六项全勾或全不勾；单独点某一项会自动同步总开关的状态。

每个视角都是**独立渲染**、独立存成 `<视角>.png`，不是从一个图里裁出来的。

### 2.3 分辨率、采样与光照

| 参数 | 可选项 | 说明 |
| --- | --- | --- |
| 分辨率 | 1080p / 2K / 4K / 8K / 16K | 默认 4K；16K = 15360×8640，约 1.33 亿像素，是实测上限 |
| 采样 | 32 / 64 / 128 / 256 | Cycles 光线采样数，默认 128（含降噪） |
| 光照 | 白昼 / 日落 / 蓝调 / 阴天 | 四套预设，分别调整太阳高度角、天空强度、补光与曝光 |

界面上还有一个**实时耗时估算**：改分辨率、改采样数、增减视角，底部那行「预计约 X」立刻变。它来自实测基准（1080p / 64 采样 / 2 视角 ≈ 180 秒）换算，量级正确，用来判断「要不要趁现在跑」足够。

服务端还会二次校验：超过 16K 的请求直接拒绝，而不是跑到一半因显存不足崩掉。

### 2.4 任务与出图

点「开始生成」后，任务在后台线程里跑，页面每 1.5 秒轮询一次状态。状态机是：

```
排队中 → 导出模型 → 清理网格 → 渲染中 → 完成 / 出错
```

右栏三块内容：

- **状态行**：当前阶段 + 已用秒数 + 舰名
- **实时日志**：带时间戳，显示导出参数、清理结果、每个视角的渲染进度
- **出图区**：跑完一张显示一张，点图看大图，右侧标着文件大小和「下载」链接

出图路径自动生成：

```
WoWs_Renders\<国家>\<舰种>\Tier <级别>\<战舰Key>\<视角>.png

例：WoWs_Renders\Germany\Battleships\Tier VIII\PGSB108_Bismarck\hero.png
```

国家与舰种做了统一映射，避免同一概念出现两种拼法：`United_Kingdom → UK`、`Russia → USSR`、`AirCarrier → AircraftCarriers`；超级战舰单独归到 `Tier Star`。

任务结束后，中间产物 `.glb` 会被自动删除，不占硬盘。

### 2.5 中文名从哪里来

游戏自带翻译文件：

```
<游戏目录>\bin\<build>\res\texts\zh\LC_MESSAGES\global.mo
```

这是个 gettext 二进制文件，共 55,288 条 msgid。舰名键形如 `IDS_PGSB108`；工具把模型键 `PGSB108_Bismarck` 拆出基础 ID `PGSB108` 去查表，得到「俾斯麦」。

提取结果已存成 `_render\shipnames_zh.json`（**3172 条** `IDS_* → 中文`），所以正常情况下不用重跑提取。游戏更新后要刷新清单，重跑一次 `python build_catalog.py` 即可。

一个容易误会的点：R 系舰名在亚服的中文文本里是**鱼类单字**（Kongō = 鲤、Amagi = 鲑），这是游戏内的正式译名，不是工具出错。

### 2.6 命令行批量

网页适合「挑着来」，要批量跑就用同一套管线的 PowerShell 脚本。在项目根目录下进入 `_render\` 执行：

```powershell
cd _render

# 单艘
.\render_all.ps1 -Ships PGSB108_Bismarck

# 船名支持模糊匹配，不必写全 Key
.\render_all.ps1 -Ships Bismarck

# 某个国家某个舰种的全 8 级
.\render_all.ps1 -Nation Germany -Type Battleship -Tier 8

# 多艘指定
.\render_all.ps1 -Ships PGSB108_Bismarck,PJSB007_Kongo_1942

# 全量（很久），跳过已有出图
.\render_all.ps1 -Everything -SkipExisting

# 4K、128 采样、双视角、日落光
.\render_all.ps1 -Nation USA -Tier 10 -Res 3840x2160 -Samples 128 -Views hero,side -Preset sunset
```

常用参数一览：

| 参数 | 默认值 | 说明 |
| --- | --- | --- |
| `-Ships` | — | 船 Key，支持模糊匹配 |
| `-Nation` / `-Type` / `-Tier` | 全部 | 按国家 / 舰种 / 级别筛选，`-Tier` 取 `1`–`11` |
| `-Everything` | 关 | 忽略上面筛选，跑全部 |
| `-Res` | `3840x2160` | 宽x高 |
| `-Samples` | `128` | Cycles 采样数 |
| `-Views` | `hero,side,bow,top` | 逗号分隔，可选 6 种 |
| `-Preset` | `day` | `day` / `sunset` / `blue` / `storm` |
| `-TexSize` | `4096` | 贴图分辨率上限 |
| `-SkipExisting` | 关 | 主视角图已存在则跳过整艘船 |
| `-KeepGlb` | 关 | 保留中间 `.glb`（调试用） |
| `-GameDir` / `-Python` | 环境变量或内置 | 覆盖游戏目录 / 解释器 |

脚本逐船执行「导出 → 清理 → 渲染」，成功/失败连耗时一起写到 `render_log.tsv`，单船失败不会中断整批。

> 想加新视角？`_render\render_ship.py` 顶部有 `VIEW_PRESETS` 和 `PRESETS` 两张表，改完即可。

## 三、本地部署

### 3.1 环境要求

| 项目 | 要求 |
| --- | --- |
| 系统 | Windows 10/11 x64（渲染链目前只在 Windows 验证过） |
| 游戏 | 《战舰世界》客户端，需含 zh 翻译文件；**模型与贴图都来自它** |
| Python | 3.11 及以上（仅用标准库，无需 `pip install`） |
| 显卡 | NVIDIA 显卡 + 较新驱动（启用 OptiX 光追加速）；纯 CPU 也能跑，但慢一个数量级 |
| 磁盘 | Blender 与工具约 1 GB；出图视分辨率而定，4K PNG 单张约 1–10 MB |

### 3.2 目录结构

```
warships\
├─ _render\                       ← 工具本体，双击这里启动
│   ├─ 启动渲染服务.bat            一键启动
│   ├─ server.py                  网页服务：目录 API / 任务队列 / 进度 / 图片
│   ├─ build_catalog.py           生成 ships.json（船表 + 中文名 + 标签）
│   ├─ extract_names.py           从 global.mo 提取中文舰名
│   ├─ render_ship.py             Blender 渲染脚本（取景/光照/海面/雾）
│   ├─ sanitize_glb.py            修复导出器的空 mesh 问题
│   ├─ render_all.ps1             命令行批量渲染
│   ├─ ships.json                 795 艘船的中文目录（已生成）
│   ├─ shipnames_zh.json          3172 条舰名映射（已生成）
│   └─ glb\                       中间文件缓存（用完即删）
├─ _tools\                        ← 依赖（自行下载，不入库）
│   ├─ blender-4.5.10-windows-x64\
│   └─ model-exporter\            wows-gltf-exporter.exe / wows-list-ships.exe
└─ WoWs_Renders\                  ← 渲染输出
```

> **仓库只含脚本**：不含任何游戏素材、模型、图片或第三方二进制，渲染输出与依赖目录都已 `.gitignore`。`.bat` 被 `.gitattributes` 标为 `-text`，防止 Git 改动换行符——这也是它 GBK 编码能一直正常工作的原因。

### 3.3 部署步骤

**第零步：拿到代码**

工具已开源在 GitHub：[**xuruizhi/wows-render**](https://github.com/xuruizhi/wows-render)

```powershell
git clone https://github.com/xuruizhi/wows-render.git
```

没装 Git 的话，在仓库页面点 `Code → Download ZIP` 解压即可。下面所有命令都以克隆或解压出来的目录为项目根目录。

> 仓库里只有脚本与两份已生成的 JSON 目录，**不含**模型、图片和第三方二进制——`_tools\` 下的依赖要按第一步自己下载。

**第一步：确认依赖就位**

在项目根目录下执行：

```powershell
Test-Path _tools\blender-4.5.10-windows-x64\blender.exe
Test-Path _tools\model-exporter\wows-gltf-exporter.exe
Test-Path _tools\model-exporter\wows-list-ships.exe
```

三行都应是 `True`。若缺少，按下面的**目录名**解压，名字不一致脚本会报 `missing dependency: ...`：

- **Blender 4.5.x** → 从 [blender.org](https://www.blender.org/download/) 下载 Windows 便携版，解压并重命名为 `_tools\blender-4.5.10-windows-x64\`
- **model-exporter** → 从 [wows-tools/wows-model-exporter](https://github.com/wows-tools/wows-model-exporter/releases) 下载最新 Release，解压到 `_tools\model-exporter\`
- **wows-depack** → 从 [wows-tools/wows-depack](https://github.com/wows-tools/wows-depack/releases) 下载最新 Release，解压到 `_tools\wows-depack\`

**第二步：配置游戏路径（不用改代码）**

游戏安装目录支持**环境变量**或**命令行参数**，脚本里的内置默认值只在两者都没提供时才生效。优先级：**`-GameDir` 参数 > `WOWS_GAME_DIR` 环境变量 > 内置默认值**。

```powershell
# 推荐：设置环境变量，对全部脚本一次生效
$env:WOWS_GAME_DIR = "<游戏安装目录>\World of Warships"

# 想永久生效（用户级环境变量）
[Environment]::SetEnvironmentVariable("WOWS_GAME_DIR", "<游戏安装目录>\World of Warships", "User")
```

`server.py`、`build_catalog.py`、`extract_names.py` 都读 `WOWS_GAME_DIR`；`server.py` 另支持 `--game-dir`，另外两个支持 `--game`。批处理时也可以只覆盖本次：

```powershell
.\render_all.ps1 -Nation Germany -GameDir "<游戏安装目录>\World of Warships"
```

> 这里的路径指向**你自己的游戏客户端**，是项目目录之外的绝对路径，所以只能由你提供，工具无法用相对路径推算。`_tools\` 下的依赖路径则是从脚本位置相对推算的，只要保持目录结构就无需配置。

**第三步：确认 Python**

`启动渲染服务.bat` 按 3.13 → 3.12 → 3.11 → `where python` 的顺序探测；`render_all.ps1` 则在 `-Python` 参数 > `WOWS_PYTHON` 环境变量 > 常见安装路径 > `PATH` 中依次查找。都找不到时会给出明确提示。

**第四步：双击启动**

```
双击  _render\启动渲染服务.bat
```

启动时脚本会把实际用到的 Python 和游戏目录打印出来，方便核对：

```
python   : <python.exe 路径>
game dir : <游戏安装目录>
```

若游戏目录不存在，不会给你一堆看不懂的报错，而是直接提示：

```
game dir not found: ...
  请设置环境变量 WOWS_GAME_DIR，或用 -GameDir 指定游戏安装目录。
```

换端口就带个参数：

```powershell
启动渲染服务.bat 8780
```

**关闭那个黑色命令行窗口 = 停止服务。**

> 唯一需要留意的编码问题：`启动渲染服务.bat` 内部会执行 `chcp 936` 并按 **GBK** 保存。改动用记事本保存时务必选「ANSI」；存成 UTF-8 会让中文变成乱码并报错。

### 3.4 第一次验证

建议先用**最小成本**跑一次，确认链路通了再上大图：

1. 选一艘你熟悉的船（例如德国战列舰 Bismarck）
2. 分辨率改 **1920×1080**
3. 只勾**主视角**，采样 **32**
4. 点「开始生成」

日志里应依次出现「导出模型 → 清理空 mesh → 渲染」，约 1–2 分钟后右侧出现图片。成功后 `WoWs_Renders\Germany\Battleships\Tier VIII\PGSB108_Bismarck\hero.png` 就有文件了。

### 3.5 耗时参考

实测环境：RTX 4060 Laptop / i7-14700HX / 8 GB 显存。

| 分辨率 | 采样 | 视角数 | 耗时 |
| --- | --- | --- | --- |
| 1280×720 | 16 | 3 | ~290 秒 |
| 1920×1080 | 32 | 1 | ~110 秒 |
| 3840×2160 | 128 | 4 | ~500–600 秒 |
| 7680×4320 | 32 | 1 | ~546 秒 |
| 15360×8640 | 16 | 1 | ~700 秒 |

按 4K×4 视角计算，单船约 10 分钟，795 艘全量要 **8–9 天**。所以：**值钱的图认真跑，普通展示用 1080p 就够了。**

## 四、已知限制

| 限制 | 说明 |
| --- | --- |
| 平台 | 只在 Windows 验证过；路径写法、`.bat` 编码、`.exe` 都依赖 Windows |
| 模型缺陷 | 船体镜像、螺旋桨缺失/错位、鱼雷缺失、老模型不支持——来自上游导出器，不是渲染问题 |
| 涂装 | 一律默认涂装，不支持迷彩替换 |
| 水面 | 程序化生成的材质，不是游戏内的真实海面 |
| 任务执行 | 串行，没有并行/分布式，一个跑完才开始下一个 |
| 分辨率上限 | 16K 是硬上限，更高会显存不足崩溃 |
| 贴图上限 | 源贴图最大 4096×4096；8K 以上多出的分辨率来自几何细节而非贴图 |
| 依赖游戏客户端 | 脚本不附带任何游戏资源，没装客户端就跑不起来 |

## 五、它是怎么被写出来的（简）

整个工具是在一次对话里由 AI 逐步构建并调试完成的，人负责定目标和验收，AI 负责查证、写代码、改错。

三件事值得一提：

**没有手写任何映射表。** 中文舰名不是人工整理的，而是 AI 发现游戏自带 gettext 翻译文件，解析出一个自己写的 `.mo` 读取器，795 艘零漏配。

**四个关键的渲染问题是被实测逼出来的。** 导出器产出的 `.glb` 里存在 `"primitives": []` 的空 mesh，Blender 会拒收整个文件，于是有了 `sanitize_glb.py`；用包围盒自动取景会怼到船身上（`bound_box` 是局部坐标、不认父子变换），于是改成用真实顶点投影后二分搜索距离；太阳方位角固定会导致某些视角死黑，于是让太阳跟着相机偏转 52°；海面平面不够大导致地平线露边，于是把海面放大到船长 4000 倍并加 Mist 大气雾。

**过程并非一次成功。** 最早的结论「包体加密、拿不出来」是错的——数据从没加密，只是索引在另外的 `.idx` 文件里；还踩过 `United Kingdom` 中间是单空格导致切分丢行、下拉框默认选中第一项导致筛出 0 艘船、`.bat` 存成 UTF-8 被 `cmd.exe` 按 GBK 解析直接崩掉这些问题。

一句话总结：**AI 负责查证与试错，人负责定目标和验收；结论不是一次说对的，是被实测纠正出来的。**

## 六、开源依赖

| 工具 | 用途 | 许可 |
| --- | --- | --- |
| [wows-model-exporter](https://github.com/wows-tools/wows-model-exporter) | 游戏模型 → `.glb` | 见仓库 |
| [wows-depack](https://github.com/wows-tools/wows-depack) | 解析游戏打包格式 | 见仓库 |
| [Blender](https://www.blender.org/) 4.5 | Cycles 渲染 | GPL |

本工具自身的代码同样开源：[**xuruizhi/wows-render**](https://github.com/xuruizhi/wows-render)（MIT）。

工具本体的 `server.py` / `render_ship.py` / `build_catalog.py` / `extract_names.py` / `sanitize_glb.py` 只用了 Python 标准库和 Blender 内置 API，没有第三方 Python 包。

---

## 七、版权边界

这一点值得单独说清楚：

- 仓库里**只有脚本**，外加两张从游戏文本派生的目录（`shipnames_zh.json`、`ships.json`）
- **不含**任何 Wargaming 素材：没有游戏文件、没有模型、没有图片
- **不含**从 Wiki 或游戏提取的图库数据

渲染出来的图片是游戏模型的**衍生作品**，模型版权归 Wargaming.net 所有。本仓库不为渲染结果授予任何再分发权利——是否公开、如何使用，请自行评估并遵守当地法律与游戏最终用户协议。

脚本本身：MIT。

---

*本项目已开源：[github.com/xuruizhi/wows-render](https://github.com/xuruizhi/wows-render)。欢迎提 issue 或 PR。*
