---
title: 博客搭建指南
date: 2026-01-28 16:45:31
tags:
---

### 本地化准备

### 第一步：本地初始化 Hexo

在终端中执行以下命令来初始化博客：

```bash
# 1. 安装 Hexo CLI (如果未安装)
npm install -g hexo-cli

# 2. 初始化项目 (请替换 my-blog 为你想要的文件夹名)
hexo init my-blog
cd my-blog

# 3. 安装依赖
npm install

# 4. 本地预览 (检查是否成功)
npx hexo server
```

### 第二步：准备 GitHub 仓库

1. 在 GitHub 上创建一个新仓库（例如 `username.github.io` 或 `my-blog`）。
2. 将本地代码推送到 GitHub：

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
# 将下面的 URL 替换为你的仓库地址
git remote add origin https://github.com/你的用户名/仓库名.git
git push -u origin main
```

### 第三步：配置 GitHub Actions (核心步骤)

这是实现自动部署的关键。我们需要创建一个 Workflow 文件，当你推送到 `main` 分支时，它会自动构建并发布到 `gh-pages` 分支。

1. 在项目根目录下创建目录和文件：`.github/workflows/deploy.yml`
2. 填入以下内容：

```yaml
name: Deploy Hexo Site

on:
  push:
    branches:
      - main  # 监听的分支，如果你用 master 请修改这里

permissions:
  contents: write # 允许 Action 写入仓库 (用于推送到 gh-pages 分支)

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    concurrency:
      group: ${{ github.workflow }}-${{ github.ref }}
    steps:
      - name: Checkout
        uses: actions/checkout@v4
        with:
          submodules: true # 如果你的主题是作为 submodule 引入的，需要设为 true

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '20' # 推荐使用 LTS 版本
          cache: 'npm'

      - name: Install Dependencies
        run: npm ci # 使用 npm ci 安装依赖更稳定

      - name: Build Hexo
        run: npx hexo generate # 生成静态文件到 public 目录

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }} # GitHub 自动生成的令牌，无需手动设置
          publish_dir: ./public # Hexo 生成的静态文件目录
          user_name: 'github-actions[bot]'
          user_email: 'github-actions[bot]@users.noreply.github.com'
          commit_message: ${{ github.event.head_commit.message }}
```

### 第四步：配置 GitHub Pages

1. 将上述文件推送到 GitHub：`git add . && git commit -m "Add deploy workflow" && git push`
2. 等待 Actions 运行完成（可以在仓库的 **Actions** 标签页查看进度）。
3. Action 成功变绿后，它会自动创建一个 `gh-pages` 分支。
4. 进入仓库 **Settings** -> **Pages**。
5. 在Build and deployment下：
   - **Source**: 选择 `Deploy from a branch`
   - **Branch**: 选择 `gh-pages` 分支，文件夹选 `/ (root)`
6. 点击 Save。稍等片刻，你的博客就会上线了！





### 常见问题提示

- **主题配置**：如果你下载了第三方主题，记得把主题文件夹下的 `.git` 目录删除，或者在 `.github/workflows/deploy.yml` 中开启 `submodules: true`。
- **站点配置**：编辑 `_config.yml`，确保 `url` 字段填写为你最终的博客网址（例如 `https://yourname.github.io`），否则 CSS/JS 路径可能会出错。
- **不要手动修改 `gh-pages`**：因为每次 Actions 运行都会覆盖这个分支，你手动改的东西会丢失。
- **不要在本地运行 `hexo d`**：既然配置了 Actions，就不需要在本地部署了。你只需要在本地写文章、预览 (`hexo s`)，满意后推送到 `main` 即可。



### 自动化流程是怎样的？

当你执行 `git push origin main` 后，GitHub Actions 会充当“自动化工厂”的角色：

1. 它检测到 `main` 分支有更新。
2. 它启动一个虚拟环境，下载你的源码。
3. 它执行 `hexo generate`，把 Markdown 编译成 HTML。
4. 它把生成的 HTML 文件**强制推送**覆盖到 `gh-pages` 分支。
5. GitHub Pages 检测到 `gh-pages` 变动，更新网站。



### 修改博客访问地址：

**在不购买自定义域名的情况下**，GitHub Pages 的网址和仓库名是严格对应的。

例如，设置仓库名为`rhainlee-blog`

打开项目根目录下的 `_config.yml`，找到 `URL` 部分，修改如下：

```yaml
# _config.yml

# 将 url 改为你的完整博客地址
url: https://rhainlee.github.io/rhainlee-blog

# 关键：将 root 改为你的仓库名（前后都要有斜杠）
root: /rhainlee-blog/
```


