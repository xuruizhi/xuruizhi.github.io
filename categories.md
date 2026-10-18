---
layout: page
title: 分类
subtitle: 按栏目浏览全部文章
permalink: /categories/
---

{%- assign date_format = "%Y-%m-%d" -%}

<div class="categories-page">
{%- for cat in site.categories_index -%}
  {%- assign cat_posts = site.categories[cat.slug] -%}
  <section class="category-block" id="{{ cat.slug }}">
    <h2 class="category-block-title">
      {{ cat.name | escape }}
      <span class="category-block-count">{{ cat_posts | size }} 篇</span>
    </h2>
    <p class="category-block-desc">{{ cat.description | escape }}</p>

    {%- if cat_posts -%}
      <ul class="posts-list list-unstyled" role="list">
        {%- for post in cat_posts -%}
        <li class="post-preview">
          <a href="{{ post.url | absolute_url }}">
            <h3 class="post-title">{{ post.title | strip_html }}</h3>
          </a>
          <p class="post-meta">{{ post.date | date: date_format }}</p>
        </li>
        {%- endfor -%}
      </ul>
    {%- else -%}
      <p class="empty-hint">这个栏目正在筹备中。</p>
    {%- endif -%}
  </section>
{%- endfor -%}
</div>

{%- assign uncategorized = site.posts | where_exp: "p", "p.categories.size == 0" -%}
{%- if uncategorized.size > 0 -%}
<h2 class="category-block-title">未归类</h2>
<ul class="posts-list list-unstyled" role="list">
  {%- for post in uncategorized -%}
  <li class="post-preview">
    <a href="{{ post.url | absolute_url }}"><h3 class="post-title">{{ post.title | strip_html }}</h3></a>
    <p class="post-meta">{{ post.date | date: date_format }}</p>
  </li>
  {%- endfor -%}
</ul>
{%- endif -%}
