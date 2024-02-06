---
title: 从零开始搭建Hexo + shokaX主题博客
date: 2024-02-05 11:14:52
tags:
  - Hexo
  - 前端
  - 个人博客
categories:
  - [一些日常开发积累]
---

# 前言

最近在掘金、知乎等论坛浏览技术文章的时候，我发现一些人都有维护着自己的一套 UI 精美、用户交互十分良好的个人小站，上面或是记录自己的技术积累、或是发布平时的一些琐碎日常。作为一个以前端为主要方向的技术人，我当然很好奇这么优美的样式是否都是站主一个人独自开发实现的，如果真的是，那也太厉害了吧！我必须得赶紧阅读源码，看看他们是怎么搭建的。

不过当我用浏览器的开发者工具检测了一下这些博客，结果令我非常震惊：这些优美的博客不是用 vue 与 react 搭建的。那总不会是三件套吧？或者可能是渲染模板？

更令我震惊的是，这些博客似乎都有相同的模板与套路可供复刻——因为我已经看过不下 10 个人在用相同 UI 的博客了，只是把界面的图片换了一下。

那么也就是说，这些博客都是由一套相同的工具生成，内部提供了一些配置文件以供用户自定义，从而生成出用户想要的界面。

于是我去网上找了一些资料——果不其然，我浏览过的这些个人博客都是由名为 **Hexo** 的快速、简单且功能强大的博客框架进行生成的，并且基于其基础配置采用了不同风格的 **主题（theme）** ，从而实现了这么美丽的博客。

除了 hexo 之外，也还有如 Astro 之类的快速博客框架，不过这篇文章主要介绍的是我自己搭建博客的历程，也就是使用 hexo+shokaX 主题搭建一款个人博客。shokaX 主题不仅样式精美，而且提供了很多非常人性化的配置，可以说是给我体验最好的一款主题。

而之所以要写这一篇文章，也正是因为我在基于这个主题搭建的过程中遇到了很多的坑。。。所以想着记录一下。

先附一下我自己的个人博客地址：[hana's blog](https://nonhana.github.io)，如果有兴趣的可以访问一下哦！

![首页1](https://common-1319721118.cos.ap-shanghai.myqcloud.com/picgo/image-20240205092016357.png)

# 教程

## Hexo 初始化

### 安装前提

由于是从零开始，hexo 的安装需要用到两个对于前端开发来说最基本的工具：

1. Node.js
2. Git

Node.js Windows 版本的安装教程可以参考 csdn 文章：[node.js 安装及环境配置超详细教程【Windows 系统安装包方式】](http://t.csdnimg.cn/xv4sj)

Git Windows 版本的安装教程可以参考 csdn 文章：[git 的安装与配置教程-超详细版](http://t.csdnimg.cn/W3qon)

当上述两个工具安装完成之后，可以开始进行下面的步骤。

### Hexo 安装

Hexo 的安装直接采用 npm 提供的第三方包的安装形式即可，而 npm 是随着 Node.js 的安装一起被安装好的。

```bash
npm install hexo-cli -g
```

此处的 `-g` 指的是全局安装 `hexo-cli` ，意味着你从任何的目录打开中断都可以执行 hexo 相关的命令。

当然，如果有特殊的需求，也可以直接局部安装，不过推荐全局进行安装，方便无基础的人进行博客搭建。

### 站点生成

切进你想要建立个人博客项目的目录， `Shift` + `鼠标右键` ，选择 `在此处打开powershell窗口` 后执行：

```bash
hexo init <你的博客名称>
```

后，hexo 会自动在这个目录下建立你输入的博客名称的目录，内部就是 hexo 项目本身。

![创建完成](https://common-1319721118.cos.ap-shanghai.myqcloud.com/picgo/image-20240205093900782.png)

之后打开这个目录，可以观察一下目录结构：

```
demo-blog
├─ .github
│  └─ dependabot.yml
├─ .gitignore
├─ package.json
├─ pnpm-lock.yaml
├─ scaffolds
│  ├─ draft.md
│  ├─ page.md
│  └─ post.md
├─ source
│  └─ _posts
│     └─ hello-world.md
├─ themes
│  └─ .gitkeep
├─ _config.landscape.yml
└─ _config.yml
```

各个文件的详细含义可以参考[hexo 的官方文档](https://hexo.io/zh-cn/docs/setup)。 **最重要的文件是\_config.yml，这是配置博客各种渲染方式以及插件的文件。**

完成了上述配置之后，可以在终端执行 `hexo s` 来生成静态访问连接来进行访问，默认的端口是 4000。

![默认的landscape主题](https://common-1319721118.cos.ap-shanghai.myqcloud.com/picgo/image-20240205094659347.png)

最初 hexo 内置的 theme 是 landscape，之后我们就要更改这个主题为 shokaX。

## shokaX 主题配置

ShokaX 是基于 Shoka 开发的主题，在 Shoka 的基础上提供了更高的性能、更多的功能以及更现代化的代码。

官方文档：[Hexo-theme-ShokaX](https://docs.kaitaku.xyz)，下述步骤有参考了官方文档。

### 初始化 shokaX 主题

在上述博客项目的终端中，执行以下命令：

1. 全局安装 `shokax-cli` ：

   ```bash
   npm install shokax-cli --location=global
   ```

2. 使用安装好的 cli 进行初始化：

   ```bash
   SXC install shokaX
   ```

### 主题配置

更改根目录 `/_config.yml` 中的 `theme` 为 `shokaX` (SXC 默认 origin 或 npm 安装修改为 `shokax`)
请注意，本主题仅在 npm 上使用的是`shokax`,其他情况下均为`shokaX`
对于 linux 等大小写敏感的系统，npm 源的 theme 必须使用`shokax`同时修改自定义配置文件为`_config.shokax.yml`。

另外，使用 hexo init 进行博客初始化的时候，是根据你的计算机中 yarn、pnpm、npm 的是否安装来决定采用哪种包管理器进行安装的。如果你安装了 yarn，那么就会固定以 yarn 进行安装；如果 pnpm 和 npm 都有安装，那么就会从上到下查找，先找到了 pnpm 包管理器，那么就会采用 pnpm 进行安装。 **而 SXC 进行 npm 包的安装采用哪种包管理器，取决于你在 hexo init 的时候采用了哪种包管理器。** 我此处由于安装了 pnpm，因此会采用 pnpm 进行安装。

**shokaX 官方推荐使用 pnpm 包管理器进行安装！！**

markdown 配置如下，直接粘贴至 `_config.yml` 文件的末尾即可：

```yaml
markdown:
  render: # 渲染器设置
    html: false # 过滤 HTML 标签
    xhtmlOut: true # 使用 '/' 来闭合单标签 （比如 <br />）。
    breaks: true # 转换段落里的 '\n' 到 <br>。
    linkify: true # 将类似 URL 的文本自动转换为链接。
    typographer:
    quotes: "“”‘’"
  plugins: # markdown-it 插件设置
    - plugin:
        name: markdown-it-toc-and-anchor
        enable: true
        options: # 文章目录以及锚点应用的 class 名称，shoka 系主题必须设置成这样
          tocClassName: "toc"
          anchorClassName: "anchor"
    - plugin:
        name: markdown-it-multimd-table
        enable: true
        options:
          multiline: true
          rowspan: true
          headerless: true
    - plugin:
        name: ./markdown-it-furigana
        enable: true
        options:
          fallbackParens: "()"
    - plugin:
        name: ./markdown-it-spoiler
        enable: true
        options:
          title: "你知道得太多了"

autoprefixer:
  exclude:
    - "*.min.css"
```

为了使上述配置生效，需要停掉默认的代码高亮：

停用默认代码高亮（<=`6.3.0`）：

```yaml
highlight:
  enable: false

prismjs:
  enable: false
```

停用默认代码高亮（>=`7.0.0-rc1`）：

```yaml
syntax_highlighter: false
```

之后，采用 `hexo-lightning-minify` 来进行文件压缩。先进行安装：

```bash
pnpm add hexo-lightning-minify
```

配置如下，直接粘贴至 `_config.yaml` 文档的末尾即可：

```yaml
minify:
  js:
    enable: true # ShokaX 自带 esbuild 优化，不建议开启，其他主题建议开启
    exclude: # 排除文件，接受 string[]，需符合 micromatch 格式
  css:
    enable: true # 开启 CSS 优化
    options:
      targets: ">= 0.5%" # browserslist 格式的 target
    exclude: # 排除文件，接受 string[]，需符合 micromatch 格式
  html:
    enable: true # 开启 HTML 优化
    options:
      comments: false # 是否保留注释内容
    exclude: # 排除文件，接受 string[]，需符合 micromatch 格式
  image:
    enable: true # 开启图片预处理和自动 WebP 化
    options:
      avif: false
      webp: true # 预留配置项，现版本无作用
      quality: 80 # 质量，支持1-100的整数、lossless或nearLossless
      effort: 2 # CPU 工作量，0-6之间的整数(越低越快)
      replaceSrc: true # 自动替换生成html中的本地图片链接为webp链接
      # 我们更建议使用 Service Worker 来在用户侧实现 replaceSrc 的功能，这将能够以一种侵入式更小的方式实现链接替换
    exclude:
```

自动 WebP 化功能在初次 `hexo g` 或 `hexo cl` 后不可用，需要再运行一次 `hexo g` 。 **如果之后需要部署到 Github Pages 上面的，那么最好把 replaceSrc 这个选项关掉，否则会导致无法渲染。**

完成后，配置 feed 生成功能以生成 `rss`、`atom`、`feed.json` 等文件：

```yaml
feed:
  limit: 20
  order_by: "-date"
  tag_dir: false
  category_dir: false
  rss:
    enable: true
    template: "node_modules/hexo-theme-shokax/layout/_alternate/rss.ejs"
    output: "rss.xml"
  atom:
    enable: true
    template: "node_modules/hexo-theme-shokax/layout/_alternate/atom.ejs"
    output: "atom.xml"
  jsonFeed:
    enable: true
    template: "node_modules/hexo-theme-shokax/layout/_alternate/json.ejs"
    output: "feed.json"
```

完成上述基础配置之后， **必须在 `_config.shokaX.yml` 文件中添加一条你的社交媒体的记录** ：

```yaml
social:
  github: https://github.com/name || github || "#191717"
  # google: https://plus.google.com/yourname || google
  # twitter: https://twitter.com/yourname || twitter || "#00aff0"
  # zhihu: https://www.zhihu.com/people/yourname || zhihu || "#1e88e5"
  # music: https://music.163.com/#/user/home?id=yourid || cloud-music || "#e60026"
  # weibo: https://weibo.com/yourname || weibo || "#ea716e"
  # about: https://about.me/yourname || address-card || "#3b5998"
  # email: mailto:foo@xxx.com || envelope || "#55acd5"
  # facebook: https://www.facebook.com/yourname || facebook
  # stackoverflow: https://stackoverflow.com/ || stack-overflow
  # youtube: https://youtube.com/yourname || youtube
  # instagram: https://instagram.com/yourname || instagram
  # skype: skype:yourname?call|chat || skype
  # douban: https://www.douban.com/people/yourname/ || douban
```

**否则就会导致项目崩溃，无法渲染！！**

之后在终端执行 `hexo s` 命令，观察是否能够正常生成站点：

![初次运行报错](https://common-1319721118.cos.ap-shanghai.myqcloud.com/picgo/image-20240205100720949.png)

可见，事情不是一帆风顺的，出现了很多报错。

仔细观察后，可见很多的错误都是 **无法找到模块** 类型的错误，这些错误基本都是 pnpm 这个包管理器的模块管理模式设置错误导致的，因此我们只需要依次进行这些报错包的安装即可：

```bash
pnpm install hexo-fs
pnpm install js-yaml
pnpm install hexo-util
pnpm install hexo-pagination
```

上述四个包安装完成后，再次执行 `hexo s` ，已经可以无报错成功生成站点了。我们进行访问：

![初始站点访问](https://common-1319721118.cos.ap-shanghai.myqcloud.com/picgo/image.png)

可见，界面十分美观！！

### 更多配置

此外的其他自定义配置，在此处不过多的介绍了，可以参考官方的配置文档：[主题配置](https://docs.kaitaku.xyz/guide/theme.html)

- algolia 搜索配置：[ algolia 搜索](https://docs.kaitaku.xyz/guide/config.html#algolia-%E6%90%9C%E7%B4%A2)
- 评论系统配置：[评论系统](https://docs.kaitaku.xyz/guide/comment.html)

### 图标映射、导航栏名称映射

有关于 shokaX 中集成的 iconfont 图标以及对应导航栏的中文映射，在其官方文档中并未给出详细的映射关系，而图标和导航栏名称又是该主题用于配置导航栏中必不可少的一个要素，我在此处给出源码中对应的映射文件位置：

- [iconfont 名称映射](https://github.com/theme-shoka-x/hexo-theme-shokaX/blob/main/source/css/_iconfont.styl)
- [menu 名称映射](https://github.com/theme-shoka-x/hexo-theme-shokaX/blob/main/languages/zh-CN.yml)

有需要配置的同学可以前往源码处自行查看。

## Github Pages 部署

有关于 Github Pages 部署的部分，建议直接查看相关的教程文档：[部署到 GitHub](https://easyhexo.com/1-Hexo-install-and-config/1-4-deploy-hexo.html#%E9%83%A8%E7%BD%B2%E5%88%B0-github)。当然不仅限于 Github 的部署，还有其他的部署方式可供选择。
