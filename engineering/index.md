---
layout: page
title: 工程
description: 技术架构与工程实践
---

这里记录技术相关的深度文章：架构设计、AI工程化、移动开发、系统设计。

## 文章列表

{% assign engineering_posts = site.posts | where: "category", "engineering" %}
{% if engineering_posts.size > 0 %}
<ul class="post-list">
  {% for post in engineering_posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span class="meta">{{ post.date | date: "%Y-%m-%d" }}{% if post.tags %} · {{ post.tags | join: ", " }}{% endif %}</span>
  </li>
  {% endfor %}
</ul>
{% else %}
<p>暂无文章。</p>
{% endif %}

## 页面

{% if site.engineering.size > 0 %}
<ul class="post-list">
  {% for page in site.engineering %}
  <li>
    <a href="{{ page.url | relative_url }}">{{ page.title }}</a>
    {% if page.description %}
    <span class="meta">{{ page.description }}</span>
    {% endif %}
  </li>
  {% endfor %}
</ul>
{% endif %}
