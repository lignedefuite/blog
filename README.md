# 拍動 | Blog

个人博客，基于 Hugo + PaperMod 主题构建。

🔗 **在线访问**: [lignedefuite.github.io/blog](https://lignedefuite.github.io/blog/)

## 技术栈

- **静态站点生成器**: [Hugo](https://gohugo.io/)
- **主题**: [PaperMod](https://github.com/adityatelange/hugo-PaperMod)
- **部署**: GitHub Pages + GitHub Actions

## 工作流

### 写文章

```bash
# 在 content/posts/ 目录下创建新文章
cd content/posts
vim my-new-post.md
```

文章格式：

```yaml
---
title: 文章标题
date: 2025-02-05
draft: false
tags:
  - 标签1
  - 标签2
---

正文内容...
```

### 发布

```bash
git add .
git commit -m "add: 新文章标题"
git push
```

推送后，GitHub Actions 会自动构建并部署到 GitHub Pages（约 1 分钟）。

## 本地预览

```bash
hugo server -D
# 访问 http://localhost:1313
```

## 项目结构

```
.
├── content/posts/       # 博客文章
├── themes/PaperMod/     # 主题（submodule）
├── config.toml          # 站点配置
└── .github/workflows/   # 自动部署配置
```

---

✍️ 用心记录，持续创作
