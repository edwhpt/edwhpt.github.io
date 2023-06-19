# Hexo Blog

使用 [Hexo](https://hexo.io/) + [GitHub Pages](https://pages.github.com/) 搭建一个博客，采用 [NexT](https://theme-next.js.org/) 主题。

Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 [Markdown](http://daringfireball.net/projects/markdown/)（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

GitHub Pages 允许每个账户创建一个名为 {username}.github.io 的仓库，另外它还会自动为这个仓库分配一个 github.io 的二级域名。


## Live Preview

[https://edwhpt.github.io/](https://edwhpt.github.io/)

## 目录

```
.
├── _config.yml     # hexo配置
├── package.json    # 项目配置
├── scaffolds       # 模版
├── source          # 开发目录
|   ├── _drafts		# 草稿
|   └── _posts		# 文章
└── themes          # hexo主题库
```

## Step

### 安装依赖

``` bash
$ npm install -g hexo-cli
$ cd <folder>
$ npm install
```

### 新建文章

``` bash
$ hexo new "post title with whitespace"
```

### 启动服务器，默认网址： `http://localhost:4000/`

``` bash
$ hexo server
```

### 生成静态文件

``` bash
$ hexo generate
```

### 部署网站

``` bash
$ hexo deploy
```

## 参考

### [Hexo + GitHub Pages 搭建个人博客](https://edwhpt.github.io/hexo-github-blog)

