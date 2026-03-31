# Ctrl + Alt + Reflect

个人知识库站点，基于 Jekyll + GitHub Pages。

## 本地开发

```bash
# 安装依赖
bundle install

# 本地预览
bundle exec jekyll serve

# 访问 http://localhost:4000
```

## 目录结构

```
ctrl-alt-reflect/
├── _config.yml          # 站点配置
├── _layouts/            # 页面模板
├── _includes/           # 公共组件
├── _posts/              # 文章（按日期命名）
├── _drafts/             # 草稿
├── assets/              # 静态资源
├── engineering/         # 工程板块
├── management/          # 管理板块
├── meta/                # 元认知板块
├── weeknotes/           # 周记板块
└── index.md             # 首页
```

## 写文章

### 标准文章

在 `_posts/` 下创建文件，命名格式：`YYYY-MM-DD-title.md`

```yaml
---
layout: post
title: "文章标题"
date: 2025-01-15
category: engineering
tags: [ai, architecture]
---

正文内容...
```

### 周记

```yaml
---
layout: post
title: "Week 3 - 2025"
date: 2025-01-15
category: weeknotes
---

本周的思考...
```

### 页面（非时间线内容）

在 `engineering/`、`management/` 等目录下创建 `.md` 文件：

```yaml
---
layout: page
title: "页面标题"
---

内容...
```

## 部署到 GitHub Pages

1. 推送到 GitHub 仓库
2. 仓库设置 → Pages → Source 选择分支
3. 访问 `https://arborlin-dev.github.io/ctrl-alt-reflect`

## 自定义域名（可选）

1. 在仓库根目录创建 `CNAME` 文件，内容为域名
2. DNS 配置 CNAME 记录指向 `arborlin-dev.github.io`
