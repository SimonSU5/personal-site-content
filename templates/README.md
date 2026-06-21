# 模板使用指南

## 配置完成 ✅

现在你可以使用以下两个模板：

### 1. 博客文章模板
- **位置**: `templates/博客文章模板.md`
- **适用**: `blogs/` 目录下的新文件
- **格式**: 按照博客文章的标准 frontmatter 格式

### 2. 作品集模板  
- **位置**: `templates/作品集模板.md`
- **适用**: `works/` 目录下的新文件
- **格式**: 按照作品集的标准 frontmatter 格式

## 使用方法

### 方法一：使用 Templater 插件（推荐）

1. 在 Obsidian 中按 `Cmd + P` 打开命令面板
2. 输入 "Templater: Create new note from template"
3. 选择对应的模板
4. 新文件会自动填充模板内容

### 方法二：使用模板快捷键

1. 在目标目录（blogs/ 或 works/）创建新文件
2. 在 Obsidian 中按 `Cmd + P` 打开命令面板  
3. 输入 "Insert template"
4. 选择对应的模板插入到当前文件

### 方法三：手动复制

直接复制 `templates/` 目录下的模板文件内容，然后根据需要修改。

## 博客文章命名规范

博客文章应该按以下格式命名：
- 带日期: `YYYY-MM-DD-slug.md` (如: `2026-06-22-my-post.md`)
- 带分类: `category-slug.md` (如: `tech-如何使用-obsidian.md`)

## 作品集命名规范

作品集文件应该按以下格式命名：
- `project-slug.md` (如: `my-awesome-project.md`)

## 注意事项

- 模板中的 `<% tp.date.now("YYYY-MM-DD") %>` 会在使用时自动替换为当前日期
- `published: false` 表示文章还未发布，发布时改为 `true`
- 记得为文章设置合适的封面图片路径