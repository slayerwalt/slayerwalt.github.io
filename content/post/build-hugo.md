+++
date = '2026-02-28'
draft = false
title = '使用 Hugo + GitHub Pages + GitHub Actions 搭建博客'
tags = ['Hugo', 'GitHub Pages', 'Blog']
+++

本文简单记录使用 Hugo 静态站点生成器搭建个人博客，并通过 GitHub Actions 自动部署到 GitHub Pages 的过程。

## 环境准备

- 安装 [Hugo](https://gohugo.io/installation/)（本站使用 v0.157.0）
- 安装 [Git](https://git-scm.com/)
- 一个 GitHub 账号

## 创建 Hugo 站点

```bash
hugo new site my-blog
cd my-blog
git init
```

## 安装主题

这里使用 [Beautiful Hugo](https://github.com/halogenica/beautifulhugo) 主题，通过 Git Submodule 方式引入：

```bash
git submodule add https://github.com/halogenica/beautifulhugo.git themes/beautifulhugo
```

主题自带了一个 `exampleSite` 目录，可以将其内容复制到项目根目录作为起步模板：

```bash
cp -r themes/beautifulhugo/exampleSite/* .
```

然后在 `hugo.toml` 中确认主题配置：

```toml
theme = "beautifulhugo"
```

其他需要自定义的配置项包括 `title`、`subtitle`、`[Params.author]` 中的个人信息、`[[menu.main]]` 导航菜单等，按需修改即可。

本地预览：

```bash
hugo serve
```

浏览器访问 `http://localhost:1313/` 即可看到效果。

## 编写文章

在 `content/post/` 目录下创建 Markdown 文件：

```bash
hugo new content/post/my-first-post.md
```

文件开头的 front matter 控制标题、日期、标签等元信息：

```markdown
+++
date = '2026-02-28'
draft = false
title = 'My First Post'
tags = ['tag1', 'tag2']
+++

Your content here...
```

> 注意：`draft = true` 的文章在正式构建时不会发布，写好后记得改为 `false`。

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
      HUGO_VERSION: 0.157.0
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
