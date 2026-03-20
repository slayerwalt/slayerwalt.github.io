# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 常用命令

```bash
# 本地开发
hugo server

# 本地开发（草稿也显示）
hugo server -D

# 构建生产版本
hugo --minify

# 新建博文
hugo new posts/文章名.md

# 初始化/更新主题 submodule
git submodule update --init --recursive
```

## 架构概览

这是一个 Hugo 静态博客，部署到 GitHub Pages。

- **主题**: FixIt（位于 `themes/FixIt/`，通过 git submodule 引入）
- **Hugo 版本**: v0.158.0 extended（必须使用 extended 版本，主题依赖 SCSS 编译）
- **部署**: push 到 `main` 分支后 GitHub Actions 自动构建并部署

## 内容结构

```
content/
  posts/      # 博文（主要内容区域）
  about/      # 关于页面（index.md）
  paper/      # 论文/学术内容区
```

## 配置要点

主配置文件为 `hugo.toml`：

- `defaultContentLanguage = "zh-cn"`，`hasCJKLanguage = true`（中文字数统计正确）
- `baseURL = "https://slayerwalt.github.io/"`
- 博文的 front matter 使用 YAML 格式（`---` 分隔符），字段：`title`、`date`、`tags`、`categories`

## 注意事项

- 主题 `themes/FixIt` 是 git submodule，克隆后需执行 `git submodule update --init --recursive`
- 不要修改 `themes/FixIt/` 下的文件（会导致 submodule 冲突），自定义应放在 `layouts/` 或 `assets/` 目录
