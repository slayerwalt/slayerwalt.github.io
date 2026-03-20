---
title: 使用 Hugo + FixIt + GitHub Pages + GitHub Actions 搭建博客
date: 2026-03-19
tags:
- github
categories:
- blog
featuredimage: img/cover_688471.png
---

> 以下内容均由 AI 生成，仅供参考。

本文简单记录使用 Hugo 静态站点生成器搭建个人博客，并通过 GitHub Actions 自动部署到 GitHub Pages 的过程。主题使用 [FixIt](https://github.com/hugo-fixit/FixIt)。

## 环境准备

- 安装 [Hugo Extended](https://gohugo.io/installation/)（本站使用 v0.158.0）——FixIt 主题使用 SCSS，必须是 Extended 版本
- 安装 [Git](https://git-scm.com/)
- 一个 GitHub 账号

## 创建 Hugo 站点

```bash
hugo new site my-blog
cd my-blog
git init
```

## 安装主题

使用 [FixIt](https://github.com/hugo-fixit/FixIt) 主题，通过 Git Submodule 方式引入：

```bash
git submodule add https://github.com/hugo-fixit/FixIt.git themes/FixIt
```

## 配置站点

在 `hugo.toml` 中进行基础配置：

```toml
baseURL = "https://<username>.github.io/"
defaultContentLanguage = "zh-cn"
hasCJKLanguage = true
title = "My Blog"
theme = "FixIt"

[params]
  version = "0.3.X"
  description = "我的个人博客"
  defaultTheme = "auto"

  [params.home.profile]
    enable = true
    avatarURL = "/img/avatar.jpg"
    title = "用户名"
    subtitle = "博客简介"
    social = true

  [params.social]
    GitHub = "<username>"

[menu]
  [[menu.main]]
    identifier = "posts"
    name = "博客"
    url = "/posts/"
    weight = 1
  [[menu.main]]
    identifier = "tags"
    name = "标签"
    url = "/tags/"
    weight = 2
```

本地预览：

```bash
hugo server --buildDrafts
```

浏览器访问 `http://localhost:1313/` 即可看到效果。

## 编写文章

在 `content/posts/` 目录下创建 Markdown 文件：

```bash
hugo new content/posts/my-first-post.md
```

文件开头的 front matter 控制标题、日期、标签等元信息：

```markdown
---
title: '文章标题'
date: '2026-02-28'
draft: false
tags: ['tag1', 'tag2']
---

正文内容...
```

> 注意：`draft: true` 的文章在正式构建时不会发布，写好后记得改为 `false`。

## 部署到 GitHub Pages

### 创建 GitHub 仓库

在 GitHub 上创建一个名为 `<username>.github.io` 的仓库（替换 `<username>` 为你的 GitHub 用户名）。

### 配置 GitHub Actions

在项目中创建 `.github/workflows/deploy.yml`：

```yaml
name: Deploy Hugo site to Pages

on:
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    env:
      HUGO_VERSION: 0.158.0
    steps:
      - name: Install Hugo CLI
        run: |
          wget -O ${{ runner.temp }}/hugo.deb https://github.com/gohugoio/hugo/releases/download/v${HUGO_VERSION}/hugo_extended_${HUGO_VERSION}_linux-amd64.deb \
          && sudo dpkg -i ${{ runner.temp }}/hugo.deb
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: recursive
      - name: Setup Pages
        id: pages
        uses: actions/configure-pages@v5
      - name: Build with Hugo
        env:
          HUGO_CACHEDIR: ${{ runner.temp }}/hugo_cache
          HUGO_ENVIRONMENT: production
          TZ: Asia/Shanghai
        run: |
          hugo --minify --baseURL "${{ steps.pages.outputs.base_url }}/"
      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

关键点：

- `hugo_extended_${HUGO_VERSION}` 必须使用 extended 版本，否则 FixIt 的 SCSS 无法编译
- `submodules: recursive` 确保 CI 环境能拉取主题子模块
- `HUGO_VERSION` 建议与本地版本保持一致

### 配置仓库 Pages 设置

进入仓库 **Settings → Pages**，将 **Source** 改为 **GitHub Actions**。

### 推送并部署

```bash
git remote add origin git@github.com:<username>/<username>.github.io.git
git add .
git commit -m "Initial commit"
git branch -M main
git push -u origin main
```

推送后 GitHub Actions 会自动触发构建和部署，稍等片刻即可通过 `https://<username>.github.io/` 访问博客。

之后每次 push 到 `main` 分支，站点都会自动更新。

## 日常写作流程

```
写文章 → git add & commit → git push → 自动部署 → 上线
```

就这么简单。
