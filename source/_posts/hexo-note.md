---
title: Hexo 站点搭建笔记
date: 2022-01-01 18:53:30
categories: # 分类
 - Blog
tags: # 标签 -hexo 
 - Hexo
 - Blog
description: Hexo 站点安装 个性化 及文章发布
---

## 安装

### 仓库搭建

1. 在 github 账号中新建仓库 `${username}.github.io` 作为站点部署仓库

   如本站站点仓库[gochri site](https://github.com/gochri/gochri.github.io)

2. 在 github 新建任意仓库如[hexo](https://github.com/gochri/hexo)，作为代码存储仓库

### 本地部署

本地部署首先需要安装 nodejs@12.13 与 git , 并连接 ssh Key 到 git 上

1. 全局安装 npm 包

```bash
npm install hexo-cli -g
```

此时 hexo-cli 已全局安装，可查看其版本号确认

```bash
hexo: 6.0.0
hexo-cli: 4.3.0
```

2. 创建博客文件夹

```bash
mkdir hexo
cd hexo
npx hexo init
```

此时项目初始化已经完成，可在 vscode 中浏览项目结构

3. 本地启动

```bash
npx hexo clean # 在启动前清除上次缓存
npx hexo server --debug
```

此时在 localhost:4000 上即可看到页面预览

### 线上部署

线上部署需安装 hexo-deployer-git ，如不存在，先安装

首先在 \_config.yml 中指定远程代码仓库

```yml
# Deployment
## Docs: https://hexo.io/docs/one-command-deployment
deploy:
  type: git
  repository: git@github.com:gochri/gochri.github.io.git
  branch: master
```

然后执行

```bash
npx hexo generate --deploy
```

返回形如下

```bash
Branch 'master' set up to track remote branch 'master' from 'git@github.com:gochri/gochri.github.io.git'.
INFO  Deploy done: git
```

此时在 [Gochri's Site](gochri.github.io) 上即可看到部署的网页

## 个性化

### 主题

在 [themes](https://hexo.io/themes/) 中查找主题，并参考文档进行安装

这里以[next](https://theme-next.js.org/docs/getting-started/)为例

1. 引入并启用主题

```bash
npm install hexo-theme-next
```

修改 \_config.yml 如下：

```yml
## Themes: https://hexo.io/themes/
theme: next
```

此时重新启动，可看到主题已经被修改

2. 主题定制化

拷贝 node_modules\hexo-theme-next_config.yml 文件到项目根部并修改文件名为 \_config.next.yml ，此时 next 主题中配置项即可被项目文件覆盖

具体修改如下：

\_config.next.yml

```yml
# Schemes
# scheme: Muse
# scheme: Mist
scheme: Pisces
# scheme: Gemini

# Dark Mode
darkmode: false

# Usage: `Key: /link/ || icon`
menu:
  home: / || fa fa-home
  # about: /about/ || fa fa-user
  # tags: /tags/ || fa fa-tags
  # categories: /categories/ || fa fa-th
  archives: /archives/ || fa fa-archive
  # schedule: /schedule/ || fa fa-calendar
  # sitemap: /sitemap.xml || fa fa-sitemap
  # commonweal: /404/ || fa fa-heartbeat

social:
  GitHub: https://github.com/gochri || fab fa-github
  E-Mail: mailto:gochri@qq.com || fa fa-envelope
  #Weibo: https://weibo.com/yourname || fab fa-weibo

# Sidebar Avatar
avatar:
  # Replace the default image and set the url here.
  url: /images/avatar.webp
  # If true, the avatar will be displayed in circle.
  rounded: true
  # If true, the avatar will be rotated with the cursor.
  rotated: true
```

其中 url 下的地址是在 source/ 下的相对地址

### 项目定制化

最后修改下网站标题

修改 \_config.yml 形如下：

```yml
# Site
title: gochri site
subtitle: ""
description: ""
keywords:
author: gochri
language: zh-CN
```

## 发布文章

### 新建文章

执行如下语句，即可在 source\_posts\ 中新建对应的 md 文件

```bash
hexo new post hexo-note # post 为默认值 可不写
```

其头部 Front-matter 依赖于 scaffolds\post.md 中的声明，形如下

```markdown
---
title: { { title } }
date: { { date } }
categories: # 分类 非平行
tags: # 标签 平行
description: # 首页出现的描述 不写默认显示 <!--more--> 前所有内容
---
```

**标签与分类的详细操作 待下篇进行**

### 编辑文章

除在本地编辑 markdown 文件外，还可通过网页进行线上编辑

```bash
npm i hexo-admin –save
```

此时访问 localhost:4000/admin 即可进入文章后台

最后 重新在本地启动 并 线上部署 即可发布文章

## 参考链接

[本站建站指导](https://ringoer.com/others/MyWebsiteGuide/)
[「Hexo_1」 从零开始建站 - 环境搭建与云端配置](https://lyrikp.art/2020/08/25/Hexo1-%E4%BB%8E%E9%9B%B6%E5%BC%80%E5%A7%8B%E5%BB%BA%E7%AB%99/)