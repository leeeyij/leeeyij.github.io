---
title: "Hugo + PaperMod 搭建博客完整指南"
date: 2026-03-09T10:00:00+08:00
draft: false
tags: ["hugo", "papermod", "github-pages", "教程"]
categories: ["技术"]
description: "从零开始使用 Hugo 和 PaperMod 主题搭建个人博客，并部署到 GitHub Pages"
cover:
    image: ""
    alt: "Hugo PaperMod"
    caption: ""
    relative: false
---

## 为什么选择 Hugo + PaperMod？

在众多静态网站生成器中，**Hugo** 以其极速的构建速度著称（每秒可生成数千页面）。配合 **PaperMod** 主题，可以快速搭建一个美观、功能完善的博客。

### 优势

- ⚡ **极速构建** - 用 Go 语言编写，构建速度极快
- 🎨 **PaperMod 主题** - 简洁现代，支持深色模式
- 🔍 **内置搜索** - 无需第三方服务
- 📱 **响应式设计** - 完美适配移动端
- 🆓 **免费托管** - 可部署到 GitHub Pages

## 安装 Hugo

### macOS

```bash
brew install hugo
```

### Windows

```powershell
winget install Hugo.Hugo.Extended
```

### Linux

```bash
sudo apt install hugo
```

验证安装：

```bash
hugo version
```

## 创建站点

```bash
hugo new site myblog --format yaml
cd myblog
git init
```

## 安装 PaperMod 主题

```bash
git submodule add --depth=1 https://github.com/adityatelange/hugo-PaperMod.git themes/PaperMod
```

在 `hugo.yaml` 中设置主题：

```yaml
theme: PaperMod
```

## 配置站点

创建 `hugo.yaml`：

```yaml
baseURL: 'https://yourusername.github.io/'
languageCode: zh-cn
title: 我的博客
theme: PaperMod

params:
  env: production
  description: '一个简洁的个人博客'
  author: 作者名
  defaultTheme: auto
  ShowReadingTime: true
  ShowShareButtons: true
  # ... 更多配置

menu:
  main:
    - identifier: home
      name: 首页
      url: /
      weight: 10
    - identifier: archives
      name: 归档
      url: /archives/
      weight: 20
```

## 创建内容

### 文章

```bash
hugo new content posts/my-first-post.md
```

Front Matter 示例：

```yaml
---
title: "文章标题"
date: 2026-03-10T08:00:00+08:00
draft: false
tags: ["标签1", "标签2"]
categories: ["分类"]
description: "文章摘要"
---
```

### 搜索页面

```bash
hugo new content search.md
```

内容：

```yaml
---
title: "搜索"
layout: "search"
summary: "search"
placeholder: "搜索文章..."
---
```

### 归档页面

```bash
hugo new content archives.md
```

```yaml
---
title: "归档"
layout: "archives"
url: "/archives/"
summary: "archives"
---
```

## 本地预览

```bash
hugo server -D
```

访问 http://localhost:1313 查看效果。

## 构建

```bash
hugo --minify
```

静态文件生成在 `public/` 目录。

## 部署到 GitHub Pages

### 1. 创建仓库

在 GitHub 创建名为 `yourusername.github.io` 的仓库。

### 2. 推送代码

```bash
git remote add origin https://github.com/yourusername/yourusername.github.io.git
git add .
git commit -m "Initial commit"
git push -u origin main
```

### 3. 启用 GitHub Pages

进入仓库 Settings > Pages：
- Source: Deploy from a branch
- Branch: main / (root)

或使用 GitHub Actions 自动部署（推荐）：

创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true
      
      - name: Setup Hugo
        uses: peaceiris/actions-hugo@v2
        with:
          hugo-version: 'latest'
          extended: true
      
      - name: Build
        run: hugo --minify
      
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

## 总结

Hugo + PaperMod 是一个强大的博客组合，构建速度快、主题美观、功能完善。通过 GitHub Pages 可以免费托管，非常适合个人博客。

有问题欢迎留言交流！
