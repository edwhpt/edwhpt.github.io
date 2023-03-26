---
title: GitHub + Hexo（NexT）搭建博客
date: 2023-03-25 23:49:26
tags:
- github
- hexo
- hexo-theme-next
categories:
- Tools
---

使用 GitHub Pages + [Hexo](https://hexo.io/zh-cn/) 搭建一个博客，采用 [NexT](https://theme-next.iissnan.com/) 主题。

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

```bash
$ cd your-hexo-site
$ git clone https://github.com/iissnan/hexo-theme-next themes/next
```

### 启用主题

在 `theme/next/_config.yml` 中找到 `theme` 字段，并将其值更改为 `next`

```yml
theme: next
```



## 部署

### 创建仓库

在 GitHub 上创建名称为 `<username>.github.io` 的储存库

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

当部署作业完成后，产生的页面会放在储存库中的 `gh-pages` 分支

在储存库中前往 `Settings > Pages > Source`，并将 branch 改为 `gh-pages`

前往 `https://<username>.github.io` 查看网站


