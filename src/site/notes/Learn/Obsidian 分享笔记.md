---
{"dg-publish":true,"dg-permalink":"自定义链接(可选)","permalink":"/自定义链接(可选)/","title":"文章标题","tags":["flashcards","gardenEntry"],"noteIcon":"","created":"2026-03-09T23:37:03.000+08:00","updated":"2026-03-09T23:38:35.000+08:00"}
---

## 简介
Obsidian 不仅仅是一个强大的本地知识库，通过 **Digital Garden** 插件，它可以直接变成一个在线博客发布器。相比 Vercel 或 Netlify 等可能有费用或额度限制的平台，GitHub Pages 提供了一个稳定且免费的替代方案。
Digital Garden 插件的优势在于它能将 Obsidian 的特有语法（如双向链接、双链引用、Callouts 等）完美转换为 HTML，并实现“一键发布”。
## 第一步：安装 Digital Garden 插件
在 Obsidian 的 **社区插件 (Community Plugins)** 中搜索并安装 `Digital Garden`。
## 第二步：准备 GitHub 账号
你需要一个 GitHub 账号来托管你的网页。
* **建议**：如果想要主域名形式（如 `username.github.io`），可以专门为此创建一个新账号。
* **注意**：此时先不要手动创建仓库，请继续执行第三步。
## 第三步：克隆模版仓库
1. 访问 [Digital Garden 模版页面](https://github.com/oleeskild/digitalgarden)。
2. 点击绿色的 **"Use this template"** 按钮，选择 **"Create a new repository"**。
3. **关键点**：仓库名称必须填入 `你的用户名.github.io`（例如 `pizzablog.github.io`）。这会告知 GitHub 将其作为用户主页发布。
4. 保持仓库为 **Public (公开)**，然后点击创建。
## 第四步：配置工作流 (Workflows)
为了让 GitHub 知道如何构建你的站点，需要添加一个部署文件：
1. 进入你的仓库，导航至 `.github/workflows` 目录（如果不存在请手动创建）。
2. 在该目录下新建一个名为 `deploy.yml` 的文件。
3. 将以下代码粘贴并提交：
```yaml
name: Deploy Eleventy site to GitHub Pages

on:
  push:
    branches: [main]

permissions:
  contents: write
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install dependencies
        run: npm ci

      - name: Build site
        run: npm run build

      - name: Upload artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: dist

  deploy:
    runs-on: ubuntu-latest
    needs: build
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```
## 第五步：启用 GitHub Pages
1. 在仓库页面点击 **Settings (设置)**。
2. 在左侧边栏选择 **Pages**。
3. 在 **Build and deployment > Source** 下拉菜单中，将来源从 "Deploy from a branch" 改为 **"GitHub Actions"**。
## 第六步：连接 Obsidian 与 GitHub
1. **生成 Token**：前往 [GitHub Personal Access Tokens](https://github.com/settings/personal-access-tokens) 生成一个新令牌。
	* 建议有效期设为 3-6 个月。
	* **Repository access**：选择 "Only select repositories"，并选中你刚创建的博客仓库。
	* **Permissions**：为 `Contents` 和 `Pull requests` 开启 **Read and write** 权限。
2. **插件配置**：打开 Obsidian 的 Digital Garden 插件设置，填入：
	* **GitHub username**: 你的用户名
	* **GitHub repo**: `你的用户名.github.io`
	* **GitHub token**: 粘贴你刚生成的 Token
3. 看到绿色的勾选标记即表示连接成功。
## 第七步：属性 (Frontmatter) 设置
Digital Garden 通过 Frontmatter 控制发布逻辑。建议创建一个包含以下内容的模板：
```markdown
---
title: 文章标题
tags: 标签
dg-permalink: 自定义链接(可选)
dg-publish: false
dg-pinned: false
dg-home: false
---
```
* **dg-publish**: 必须设为 `true` 插件才会发布该笔记。
* **dg-home**: 如果是你的主页，请设为 `true`。
* **dg-permalink**: 可以自定义 URL 路径，如 `pizzablog.github.io/pepperoni`。
## 第八步：发布你的第一页
1. 创建新笔记并填入上述模板。
2. 将 `dg-publish` 设为 `true`。
3. 按下 `Cmd/Ctrl + P` 打开命令面板，选择 **"Digital Garden: Publish Single Note"**。
4. 稍等片刻（可以在 GitHub 的 Actions 选项卡查看进度），你的笔记就会出现在 `username.github.io` 上。