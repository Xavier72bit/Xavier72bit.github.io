---
title: 使用Hexo框架搭建GitHub Page
date: 2024-09-14 14:33:08
tags:
---

# 先决条件
* 拥有GitHub账号
* 创建了USERNAME.github.io的仓库，USERNAME为你的GitHub用户名
* 系统中安装了Git
* 安装了Node.js

# 初始化

## 安装hexo

```bash
npm install -g hexo-cli
```

如果安装失败，可以尝试使用cnpm

```bash
npm install -g cnpm --registry=https://registry.npm.taobao.org
cnpm install -g hexo-cli
```

## 初始化hexo

```bash
mkdir hexo_blog && cd hexo_blog
hexo init
```

## 编写博客文章

```bash
# 进入hexo_blog目录后
hexo new "some-title"
```

然后会在hexo_blog/source/_posts目录下生成一个some-title.md文件，编辑该文件即可。

## 生成博客页面

```bash
# 进入hexo_blog目录后
hexo clean
hexo generate
hexo server  # 本地预览
```

## 部署到github

reference: https://hexo.io/zh-cn/docs/github-pages

### 配置GitHub Actions

1. 创建.github/workflows/pages.yml文件，内容参考如下，主要是注意修改node-version，确保它与你本地的node版本一致：
```yaml
name: Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          # If your repository depends on submodule, please see: https://github.com/actions/checkout
          submodules: recursive
      - name: Use Node.js 20
        uses: actions/setup-node@v4
        with:
          # Examples: 20, 18.19, >=16.20.2, lts/Iron, lts/Hydrogen, *, latest, current, node
          # Ref: https://github.com/actions/setup-node#supported-version-syntax
          node-version: "20"
      - name: Cache NPM dependencies
        uses: actions/cache@v4
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v3
        with:
          path: ./public
  deploy:
    needs: build
    permissions:
      pages: write
      id-token: write
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

2. 推送hexo_blog目录下的所有文件到main分支

### 使用命令行工具

1. 先安装hexo-deployer-git

```bash
# 进入hexo_blog目录后
npm install --save hexo-deployer-git
```

2. 配置部署仓库

配置文件为hexo_blog/_config.yml，找到deploy部分，修改为：

```yaml
deploy:
  type: 'git'
  repo: 'https://github.com/USERNAME/USERNAME.github.io.git'
  branch: 'main'
```

将USERNMAE替换为你的GitHub用户名。

3. 部署

```bash
# 进入hexo_blog目录后
git config --global user.name "USERNAME"        # 将USERNAME替换为你的GitHub用户名
git config --global user.email "EMAIL_ADDRESS"  # 将EMAIL_ADDRESS替换为你的GitHub注册邮箱
hexo deploy
```

# 后续的日常工作

维护博客内容

```bash
hexo clean             # 清除缓存
hexo new "some-title"  # 写博客
hexo generate          # 生成博客页面
hexo s                 # 这一步不执行也可以，就是预览一下效果
```

部署博客

* 如果是使用命令行工具，执行`hexo deploy`即可
* 如果是使用GitHub Actions，执行`git commit && git push`即可