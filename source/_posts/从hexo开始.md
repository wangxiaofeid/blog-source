---
title: 从hexo开始
date: 2018-06-28 09:34:37
tags:
---

### 是什么
Hexo 是一个快速、简洁且高效的博客框架。Hexo 使用 Markdown（或其他渲染引擎）解析文章，在几秒内，即可利用靓丽的主题生成静态网页。

### 怎么做
* 安装node
* 安装git
* 安装hexo
* 安装 Hexo 完成后，请执行下列命令，Hexo 将会在指定文件夹中新建所需要的文件
``` bash
$ hexo init <folder>
$ cd <folder>
$ npm install
```
* 新建页面
``` bash
$ hexo new [layout] <title>
```
* 生成静态文件
``` bash
$ hexo generate
```
* 部署网站（注：需要在_config.yml里面配置deploy的git地址）
``` bash
$ hexo deploy
or
$ hexo d
```

官网文档请移步: [文档](https://hexo.io/zh-cn/docs/index.html)
