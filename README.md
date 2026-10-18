# 安身、立命、静心、养性

一名全职爸爸的 AI 工程笔记，主要写 **AI 辅助教学**。

线上地址：<https://xuruizhi.github.io>

## 技术栈

- **主题**：[beautiful-jekyll](https://github.com/daattali/beautiful-jekyll) 6.0.1（MIT）
- **构建**：GitHub Actions + Jekyll 4（`.github/workflows/pages.yml`）
- **风格**：在主题之上叠加黑白极简样式（`assets/css/main-custom.scss`）

## 目录结构

```
.
├── _config.yml                # 站点配置：导航、社交、配色、栏目
├── _data/ui-text.yml          # 主题文案（可选）
├── _includes/                 # 主题组件（导航、页脚、评论等）
├── _layouts/                  # 主题布局（home / post / page / base）
├── _posts/                    # 文章
├── assets/
│   ├── css/beautifuljekyll.css      # 主题样式（勿改）
│   ├── css/main-custom.scss         # 自定义黑白极简样式（改这里）
│   └── img/avatar-icon.png          # 头像（替换成你自己的）
├── images/                    # 文章配图
├── index.markdown             # 首页
├── categories.md              # /categories/ 分类页
├── tags.html                  # /tags/ 标签页
├── search.md                  # /search/ 搜索页
├── search.json                # 搜索索引（自动生成，勿手改）
├── about.md                   # /about/
└── .github/workflows/pages.yml # 自动部署
```

## 写一篇新文章

在 `_posts/` 下新建 `YYYY-MM-DD-slug.md`：

```markdown
---
layout: post
title: "文章标题"
subtitle: "一句话副标题（可选，显示在标题下方）"
date: 2026-10-20 10:00:00 +0800
categories: ai-assisted-edu   # 取值见 _config.yml 的 categories_index
tags: [AI, 教学]
---

正文……
```

**栏目 slug**：

| slug | 栏目 | 说明 |
| --- | --- | --- |
| `ai-assisted-edu` | AI 辅助教学 | 把 AI 用在教学与陪学上，以及它的边界 |
| `engineering` | 工程底子 | Git 与协作流程等基本功 |
| `notes` | 杂记 | 读书笔记、工作方法、踩坑记录 |

新增栏目：在 `_config.yml` 的 `categories_index` 里加一项即可。

> 栏目定义是**唯一来源**：`/categories/` 页面、文章归类都读它。
> 改完栏目记得同步更新 `about.md` 的「这里写什么」和本 README 的表格。

## 本地预览

需要 Ruby 3.x + Bundler：

```bash
bundle install
bundle exec jekyll serve --livereload
# 打开 http://127.0.0.1:4000
```

## 发布

推送到 `main` 分支后，GitHub Actions 会自动构建并部署。

> **首次使用**：需要在仓库 Settings → Pages 里把 **Source** 改为 **GitHub Actions**（而不是 Deploy from a branch）。

## 页面宽度

宽度只有一个开关，在 `assets/css/main-custom.scss` 顶部：

```scss
:root {
  --content-max-width: 1100px;  // ← 改这一个值即可
}
```

- 想更宽：调大（如 `1280px`）；想收窄成阅读模式：调小（如 `800px`）
- `1100px` 是大屏下的最大宽度；屏幕比它窄时会自动收缩，不会出现横向滚动
- 正文两侧留白由 `.container` 的 `padding-left/right: 24px` 控制，窄屏下靠它避免文字贴边

> 为什么不能只改一处 Bootstrap 栅格？
> 主题原来的正文宽度是 `col-xl-8 offset-xl-2`（容器宽的 2/3）。
> 因此这里既放宽了容器，也去掉了栅格偏移，正文才能占满整个容器宽度。

## 常用自定义

- **改配色 / 间距**：编辑 `assets/css/main-custom.scss`
- **改页面宽度**：见上面的「页面宽度」
- **改导航**：`_config.yml` 的 `navbar-links`
- **改社交链接**：`_config.yml` 的 `social-network-links`
- **换头像**：替换 `assets/img/avatar-icon.png`
- **加评论**：主题内置 utterances / giscus / disqus，在 `_config.yml` 里配置即可
