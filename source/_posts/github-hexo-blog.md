---
title: GitHub + Hexo（NexT）搭建博客
date: 2023-03-25 23:49:26
categories: Tools
tags:
- GitHub
- Hexo
- hexo-theme-next
---

使用 GitHub Pages + [Hexo](https://hexo.io/zh-cn/) 搭建一个博客，采用 [NexT](https://theme-next.js.org/) 主题。

GitHub Pages 允许每个账户创建一个名为 {username}.github.io 的仓库，另外它还会自动为这个仓库分配一个 github.io 的二级域名。

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

<!-- more -->

## 安装

在本地安装项目依赖环境

### 安装前提

- [Node.js](https://nodejs.org/) (Node.js 版本需不低于 10.13，建议使用 Node.js 12.0 及以上版本)

- [Git](https://git-scm.com/)

### 安装 Hexo

```bash
$ npm install -g hexo-cli
```



## 初始化

使用 `hexo-cli` 创建项目

```bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```

目录如下：

```
.
├── _config.yml
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
```

生成静态文件

```bash
$ hexo generate
```

启动服务器。默认情况下，访问网址为： `http://localhost:4000/`

```bash
$ hexo server
```

新建文章

```bash
$ hexo new "My New Post"
```



## 配置主题

Hexo 安装主题的方式非常简单，只需要将主题文件拷贝至站点目录的 `themes` 目录下， 然后修改下配置文件即可。具体到 NexT 来说，安装步骤如下。

### 下载主题

如果你在使用 Hexo 5.0 或更新版本，最简单的安装方式是通过 npm：

```bash
$ cd hexo-site
$ npm install hexo-theme-next
```

你也可以直接克隆整个仓库：


```bash
$ cd hexo-site
$ git clone https://github.com/next-theme/hexo-theme-next themes/next
```

### 启用主题

安装完成后，在 Hexo 配置文件中将 `theme` 设置为 `next`

```yml
# hexo-site/_config.yml
theme: next
```

### 配置

在根目录下创建 `_config.next.yml`

```bash
# Installed through npm
cp node_modules/hexo-theme-next/_config.yml _config.next.yml
# Installed through Git
cp themes/next/_config.yml _config.next.yml
```



## 部署

本文将使用 [GitHub Actions](https://docs.github.com/zh/actions) 部署至 GitHub Pages，此方法适用于公开或私人储存库。若你不希望将源文件夹上传到 GitHub，请参阅[一键部署](#一键部署)。 

在 GitHub 上创建名称为 `<username>.github.io` 的储存库，若之前已将 Hexo 上传至其他储存库，将该储存库重命名即可

将 Hexo 文件夹中的文件 push 到储存库的默认分支，默认分支通常名为 `main`，旧一点的储存库可能名为 `master`

- 将 `main` 分支 push 到 GitHub：

  ```bash
  $ git push -u origin main
  ```

- 默认情况下 `public/` 不会被上传(也不该被上传)，确保 `.gitignore` 文件中包含一行 `public/`。整体文件夹结构应该与 [范例储存库](https://github.com/hexojs/hexo-starter) 大致相似。

使用 `node --version` 指令检查你电脑上的 Node.js 版本，并记下该版本 (例如：`v16.y.z`)

在储存库中建立 `.github/workflows/pages.yml`，并填入以下内容 (将 `16` 替换为上个步骤中记下的版本)：

```yml
# .github/workflows/pages.yml
name: Pages

on:
  push:
    branches:
      - main # default branch

jobs:
  pages:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v2
      - name: Use Node.js 16.x
        uses: actions/setup-node@v2
        with:
          node-version: "16"
      - name: Cache NPM dependencies
        uses: actions/cache@v2
        with:
          path: node_modules
          key: ${{ runner.OS }}-npm-cache
          restore-keys: |
            ${{ runner.OS }}-npm-cache
      - name: Install Dependencies
        run: npm install
      - name: Build
        run: npm run build
      - name: Deploy
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./public
```

当部署作业完成后，产生的页面会放在储存库中的 `gh-pages` 分支

在储存库中前往 `Settings > Pages > Source`，并将 branch 改为 `gh-pages`。

前往 `https://<你的 GitHub 用户名>.github.io` 查看网站。

### 一键部署

安装 [hexo-deployer-git](https://github.com/hexojs/hexo-deployer-git)

```bash
$ npm install hexo-deployer-git --save
```

配置 `_config.yml` 

```yml
deploy:
  type: git
  repo: https://github.com/<username>/<project>
  # example, https://github.com/hexojs/hexojs.github.io
  branch: gh-pages
```

执行部署命令

```bash
$ hexo clean && hexo deploy
```

浏览 `<GitHub 用户名>.github.io` 检查你的网站能否运作

