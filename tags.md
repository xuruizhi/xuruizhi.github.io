---
layout: page
title: 标签
permalink: /tags/
description: 标签由每篇文章的 front matter 自动汇总，点击跳转到对应标签下的所有文章。
---

{%- assign date_format = site.minima.date_format | default: "%Y-%m-%d" -%}
{%- assign sorted_tags = site.tags | sort -%}

{%- if sorted_tags.size > 0 -%}
  <div class="tag-cloud">
    {%- for tag in sorted_tags -%}
      {%- assign tag_name = tag[0] -%}
      {%- assign tag_count = tag[1].size -%}
      <a class="tag-chip" href="#{{ tag_name | slugify }}">#{{ tag_name }} <span class="count">{{ tag_count }}</span></a>
    {%- endfor -%}
  </div>

  <div class="tag-index">
    {%- for tag in sorted_tags -%}
      {%- assign tag_name = tag[0] -%}
      {%- assign tag_posts = tag[1] -%}
      <section class="tag-block" id="{{ tag_name | slugify }}">
        <h2 class="tag-block-title">#{{ tag_name }} <span class="count">{{ tag_posts.size }}</span></h2>
        <ul class="post-list compact">
          {%- for post in tag_posts -%}
          <li class="post-item">
            <span class="post-meta">{{ post.date | date: date_format }}</span>
            <a class="post-link small" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
          </li>
          {%- endfor -%}
        </ul>
      </section>
    {%- endfor -%}
  </div>
{%- else -%}
  <p class="empty-hint">还没有文章携带标签。</p>
{%- endif -%}
