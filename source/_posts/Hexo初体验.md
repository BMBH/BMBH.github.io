---
title: Hexo初体验
date: 2018-05-10
tags: Hexo
categories: Hexo
---

### 多PC同步

 1. master分支

修改_config.yml
```
   type: git
   repo: https://github.com/username/username.github.io.git
   branch: master
```

 2. hexo分支简单的说就是把Hexo环境push到hexo分支

    流程：
    >  1. 创建仓库，http://yourname.github.io
    >  2. 创建两个分支：master 与 hexo
    >  3. 设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件），使用git clonegit@github.com:yourname/yourname.github.io.git拷贝仓库
    >  4. 修改_config.yml中的deploy参数，分支应为master。执行hexo g -d生成网站并部署到GitHub上。
    
### 更换主题

- _config.yml 修改 theme: next
- 主题目录下找到_config.yml也可以进行修改
