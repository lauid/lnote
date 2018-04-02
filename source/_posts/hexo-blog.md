---
title: hexo blog
date: 2017-01-04 00:11:54
tags: blog
---
- hexo安装
[hexo install](https://hexo.io/zh-cn/docs/setup.html)
- hexo next主题
[next theme](http://theme-next.iissnan.com/getting-started.html)
- npm install hexo-deployer-git --save
	安装之后可以使用hexo deploy一键部署到github
- hexo new 文章标题
- hexo generate
	生成静态文件
- hexo p == hexo publish
- hexo d == hexo deploy#部署
- hexo clean #清除缓存 网页正常情况下可以忽略此条命令
- hexo server
	本地服务可以调试预览
	hexo server #Hexo 会监视文件变动并自动更新，您无须重启服务器。
	hexo server -s #静态模式
	hexo server -p 5000 #更改端口
	hexo server -i 192.168.1.1 #自定义 IP

<!--more-->

```
	tags:
	  - tag1
	  - tag2
	categories: xxx
```
冒号后面要有空格
应该在 ---之上，---下面是页面内容
令：要添加tags和categories页面；且主题的配置文件和站点的配置文件tags和categories的注释要打开

- 添加标签
hexo new page tags
确认站点配置文件里有tag_dir: tags
确认主题配置文件里有tags: /tags
编辑站点的source/tags/index.md，添加

```
title: tags
date: 2015-10-20 06:49:50
type: "tags"
comments: false
```

- 添加分类
hexo new page categories
确认站点配置文件里有category_dir: categories
确认主题配置文件里有categories: /categories
编辑站点的source/categories/index.md，添加

```
title: categories
date: 2015-10-20 06:49:50
type: "categories"
comments: false
```
