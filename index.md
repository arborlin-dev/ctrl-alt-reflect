---
layout: default
title: 首页
---

<section class="hero">
  <h1>Ctrl + Alt + Reflect</h1>
  <p class="subtitle">技术建构 · 管理思考 · 认知迭代</p>
</section>

<section class="categories">
  <h2 class="section-title">探索</h2>
  
  <div class="category-grid">
    <a href="{{ '/engineering/' | relative_url }}" class="category-card">
      <h3>🛠 工程</h3>
      <p>技术架构、AI工程化、移动开发、系统设计的沉淀</p>
    </a>
    
    <a href="{{ '/management/' | relative_url }}" class="category-card">
      <h3>🎯 管理</h3>
      <p>团队建设、授权框架、组织设计的实践笔记</p>
    </a>
    
    <a href="{{ '/meta/' | relative_url }}" class="category-card">
      <h3>🧠 元认知</h3>
      <p>思维工具、复盘方法论、认知偏误的觉察</p>
    </a>
    
    <a href="{{ '/weeknotes/' | relative_url }}" class="category-card">
      <h3>📝 周记</h3>
      <p>每周的碎片化思考与随笔</p>
    </a>
  </div>
</section>

<section class="recent-posts">
  <h2 class="section-title">最近更新</h2>
  
  <ul class="post-list">
    {% for post in site.posts limit:5 %}
    <li>
      <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
      <span class="meta">{{ post.date | date: "%Y-%m-%d" }} · {{ post.category }}</span>
    </li>
    {% endfor %}
  </ul>
  
  {% if site.posts.size == 0 %}
  <p style="color: #888;">还没有文章，开始写第一篇吧 →</p>
  {% endif %}
</section>
