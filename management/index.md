---
layout: page
title: 管理
description: 团队管理与组织设计
---

这里记录管理相关的思考：团队建设、授权框架、组织设计、跨团队协作。

## 文章列表

{% assign management_posts = site.posts | where: "category", "management" %}
{% if management_posts.size > 0 %}
<ul class="post-list">
  {% for post in management_posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span class="meta">{{ post.date | date: "%Y-%m-%d" }}{% if post.tags %} · {{ post.tags | join: ", " }}{% endif %}</span>
  </li>
  {% endfor %}
</ul>
{% else %}
<p>暂无文章。</p>
{% endif %}
