---
title: Hexo初体验
date: 2018-05-10
tags: Hexo
categories: Hexo
---
### Hexo本地安装

- Node.js安装
- Hexo npm安装如下

> 1. npm install hexo -g //安装
> 2. hexo init //初始化
> 3. npm install //安装组件
> 4. hexo g //首次体验
> 5. hexo s //本地启动

- GitHub Pages新建
- Git安装并设置SSH
- Hexo _config.yml修改git

```
   type: git
   repo: https://github.com/username/username.github.io.git
   branch: master
```

**注：yml文件语言要求严格，注意冒号之后的半角英文空格**

- Hexo 新建博客

> hexo new post “博客名”

- Hexo 生成部署

> 1. npm install hexo-deployer-git --save //安装扩展
> 2. hexo d -g //生成部署


### Hexo多机同步

Hexo分支简单的说就是把Hexo环境push到Hexo分支

>  1. 在Hexo仓库下创建一个hexo分支，设置为默认分支
>  2. 克隆仓库的hexo分支
>  3. 将Hexo本地的原始文件全部拷贝进克隆仓库提交
>  4. 其他PC克隆仓库的hexo分支，npm install安装Hexo组件

**注：themes目录下的.git文件夹需要删除，否则不能上传相关主题**

### Hexo更换Next主题

- Git Bash 将Next主题克隆到themes文件夹
- _config.yml 修改 theme: next

### Hexo Next主题配置

- 主题风格

```
 # Schemes
 #scheme: Muse
 scheme: Mist
 #scheme: Pisces
 #scheme: Gemini
```

- 菜单设置

```
menu:
  home: / || home
  #about: /about/ || user
  tags: /tags/ || tags
  categories: /categories/ || th
  archives: /archives/ || archive
  #schedule: /schedule/ || calendar
  #sitemap: /sitemap.xml || sitemap
  #commonweal: /404/ || heartbeat
```

- 添加标签页面

```
hexo new page tags
```

index.md

```
	---
	title: tags
	date: 2018-05-09 16:41:16
	type: "tags"
	---
```

- 添加分类

```
hexo new page categories
```

index.md

```
	---
	title: categories
	date: 2018-05-09 16:42:53
	type: "categories"
	---
```
