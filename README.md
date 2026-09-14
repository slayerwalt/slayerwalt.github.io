## Slayer's Blog

[访问博客](https://slayerwalt.cn/)

博客使用 Hugo Extended 0.158.0 和 FixIt 主题。正式站点部署在阿里云 ECS，由 Nginx 提供静态文件服务，使用 Let's Encrypt 证书和 Certbot 自动续期，HTTP 请求自动跳转至 HTTPS。

更新正式站点时，在本地按 `https://slayerwalt.cn/` 构建并上传静态产物到 ECS。仓库同时保留 [GitHub Pages 站点](https://slayerwalt.github.io/)：推送到 `main` 后，GitHub Actions 会自动构建并部署到 Pages；该工作流不会更新 ECS。
