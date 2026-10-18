# 安身、立命、静心、养性

一名测试工程师的 AI 工程笔记 —— AI 辅助研发、AI Agent 工程与 AI 质量保障。

线上地址：<https://xuruizhi.github.io>

## 目录结构

```
.
├── _config.yml              # 站点配置：标题、定位、导航、栏目 / 标签定义
├── _includes/
│   ├── head.html            # <head> 覆写（主题色等）
│   ├── header.html          # 顶部导航覆写（当前页高亮）
│   ├── post-meta.html       # 文章的分类 / 标签 chip
│   └── ...                  # 其余沿用 minima 默认
├── _layouts/
│   ├── home.html            # 首页：hero + 栏目入口 + 最新文章
│   ├── post.html            # 文章页：元信息 + 上一篇/下一篇 + 可选评论
│   └── page.html            # 普通页面
├── _posts/                  # 文章（文件名格式 YYYY-MM-DD-title.md）
├── assets/main.scss         # 站点样式（覆写 minima 变量 + 自定义外观）
├── categories.md            # /categories/ 分类页
├── tags.md                  # /tags/ 标签页
├── archive.md               # /archive/ 归档页
├── search.md                # /search/ 前端搜索页
├── search.json              # 搜索索引（由文章自动生成，勿手改）
├── about.md                 # /about/
└── index.markdown           # 首页正文
```

## 写一篇新文章

在 `_posts/` 下新建 `YYYY-MM-DD-slug.md`：

```markdown
---
layout: post
title: "文章标题"
date: 2026-10-20 10:00:00 +0800
categories: ai-quality          # 必填一项，取值见 _config.yml 的 categories_index
tags: [LLM, Evaluation]         # 可选，自由填写
description: "一句话摘要，会用于首页摘要和搜索结果。"
---

正文……
```

**栏目 slug**（`categories: <slug>`）：

| slug | 栏目 |
| --- | --- |
| `ai-assisted-dev` | AI 辅助研发 |
| `agent-engineering` | Agent 工程 |
| `ai-quality` | AI 质量保障 |
| `engineering` | 工程底子 |
| `notes` | 杂记 |

> 新增栏目只需在 `_config.yml` 的 `categories_index` 里加一项，首页、分类页会自动出现。

## 本地预览

需要 Ruby 3.x + Bundler。

```bash
bundle install
bundle exec jekyll serve --livereload --drafts
# 打开 http://127.0.0.1:4000
```

## 发布

推送到 `main` 分支即可，GitHub Pages 自动构建（Settings → Pages → Source: `Deploy from a branch` / `main` / `root`）。

## 可选的增强

- **评论**：在 `_config.yml` 加下面一段，文章页底部会自动出现评论区（基于 GitHub Issues，无需后端）：

  ```yaml
  comments:
    provider: utterances
    utterances:
      repo: xuruizhi/xuruizhi.github.io
      issue_term: pathname
      theme: github-light
  ```

  需要在 GitHub 上安装 [utterances app](https://github.com/apps/utterances)。

- **分类/标签与搜索**：已内置，无需额外插件（GitHub Pages 的 safe 模式不支持自定义插件，所以这里全部用 Liquid + 原生 JS 实现）。
