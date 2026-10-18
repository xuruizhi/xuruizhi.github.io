---
layout: page
title: 分类
permalink: /categories/
description: 按栏目浏览全部文章。栏目是长期主线，标签是临时话题。
---

{%- assign date_format = site.minima.date_format | default: "%Y-%m-%d" -%}

<div class="cat-index">
  {%- for cat in site.categories_index -%}
    {%- assign cat_posts = site.categories[cat.slug] -%}
    <div class="cat-block" id="{{ cat.slug }}">
      <div class="cat-block-head">
        <h2 class="cat-block-title">{{ cat.name | escape }}</h2>
        <span class="cat-block-count">{{ cat_posts | size }} 篇</span>
      </div>
      <p class="cat-block-desc">{{ cat.description | escape }}</p>
      {%- if cat.keywords -%}
        <p class="cat-block-keywords">
          {%- for kw in cat.keywords -%}<span>#{{ kw }}</span>{%- endfor -%}
        </p>
      {%- endif -%}

      {%- if cat_posts -%}
        <ul class="post-list compact">
          {%- for post in cat_posts -%}
          <li class="post-item">
            <span class="post-meta">{{ post.date | date: date_format }}</span>
            <a class="post-link small" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
          </li>
          {%- endfor -%}
        </ul>
      {%- else -%}
        <p class="empty-hint">这个栏目正在筹备中。</p>
      {%- endif -%}
    </div>
  {%- endfor -%}
</div>

<h2 class="section-title">未归类的文章</h2>
{%- assign uncategorized = site.posts | where_exp: "p", "p.categories.size == 0" -%}
{%- if uncategorized.size > 0 -%}
  <ul class="post-list compact">
    {%- for post in uncategorized -%}
    <li class="post-item">
      <span class="post-meta">{{ post.date | date: date_format }}</span>
      <a class="post-link small" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
    </li>
    {%- endfor -%}
  </ul>
{%- else -%}
  <p class="empty-hint">暂无。每篇文章都归档在某个栏目下。</p>
{%- endif -%}
