---
layout: page
title: 元认知
description: 思维工具与认知迭代
---

这里记录认知相关的探索：复盘方法论、思维工具、认知偏误的觉察与修正。

## 文章列表

{% assign meta_posts = site.posts | where: "category", "meta" %}
{% if meta_posts.size > 0 %}
<ul class="post-list">
  {% for post in meta_posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span class="meta">{{ post.date | date: "%Y-%m-%d" }}{% if post.tags %} · {{ post.tags | join: ", " }}{% endif %}</span>
  </li>
  {% endfor %}
</ul>
{% else %}
<p>暂无文章。</p>
{% endif %}
