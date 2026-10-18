---
layout: page
title: 归档
permalink: /archive/
description: 全部文章按年份倒序排列。
---

{%- assign date_format = site.minima.date_format | default: "%Y-%m-%d" -%}
{%- assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" -%}

{%- if site.posts.size > 0 -%}
  <p class="archive-summary">共 {{ site.posts.size }} 篇文章，跨 {{ posts_by_year.size }} 个年份。</p>

  {%- for year in posts_by_year -%}
    <section class="archive-year">
      <h2 class="archive-year-title">{{ year.name }}<span class="count">{{ year.items.size }}</span></h2>
      <ul class="post-list compact">
        {%- for post in year.items -%}
        <li class="post-item">
          <span class="post-meta">{{ post.date | date: "%m-%d" }}</span>
          <a class="post-link small" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
          {%- if post.categories.size > 0 -%}
            {%- assign cat = site.categories_index | where: "slug", post.categories.first | first -%}
            <span class="cat-inline">{{ cat.name | default: post.categories.first }}</span>
          {%- endif -%}
        </li>
        {%- endfor -%}
      </ul>
    </section>
  {%- endfor -%}
{%- else -%}
  <p class="empty-hint">还没有文章。</p>
{%- endif -%}
