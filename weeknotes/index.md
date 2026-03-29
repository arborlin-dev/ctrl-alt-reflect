---
layout: page
title: 周记
description: 每周的碎片化思考
---

低门槛的持续写作。不用完美，只要记录。

## 文章列表

{% assign weeknote_posts = site.posts | where: "category", "weeknotes" %}
{% if weeknote_posts.size > 0 %}
<ul class="post-list">
  {% for post in weeknote_posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span class="meta">{{ post.date | date: "%Y-%m-%d" }}</span>
  </li>
  {% endfor %}
</ul>
{% else %}
<p>暂无周记。</p>
{% endif %}
