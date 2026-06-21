# Personal Site Content

个人网站的内容仓库，存储博客文章、作品集等 Markdown 文件。

## 目录结构

```
personal-site-content/
├── config/              # 配置文件
├── blogs/               # 博客文章
│   ├── YYYY/MM/         # 按年月归档
│   └── drafts/          # 草稿（不会同步到网站）
├── works/               # 作品集
├── snippets/            # 代码片段
└── assets/              # 资源文件
```

## 博客文章格式

文件命名格式：`YYYY-MM-DD-slug.md`

Frontmatter 模板：
```yaml
---
title: 文章标题
excerpt: 文章摘要
category: 分类
tags: 标签1, 标签2
cover: assets/covers/image.jpg
---
# 正文开始

文章内容...
```

## 作品集格式

文件命名格式：`project-slug.md`

```yaml
---
title: 项目名称
description: 项目描述
cover: assets/covers/project-cover.jpg
tech: React, TypeScript, Node.js
demo: https://demo.example.com
repo: https://github.com/user/repo
---
# 项目详情

项目介绍...
```

## 同步方式

内容更新后推送到 GitHub，网站会自动同步。
