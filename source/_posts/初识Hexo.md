---
title: 初识Hexo
date: 2018-05-09 14:30:46
type: "GitHub"
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
    >  1. 创建仓库，http://CrazyMilk.github.io
    >  2. 创建两个分支：master 与 hexo
    >  3. 设置hexo为默认分支（因为我们只需要手动管理这个分支上的Hexo网站文件）
    >  4. 使用git clonegit@github.com:CrazyMilk/CrazyMilk.github.io.git拷贝仓库
    >  5. 在本地http://CrazyMilk.github.io文件夹下通过Git bash依次执行npm install hexo、hexo init、npm install 和 npm install hexo-deployer-git（此时当前分支应显示为hexo）
    >  6. 修改_config.yml中的deploy参数，分支应为maste
    >  7. 依次执行git add .、git commit -m "..."、git push origin hexo提交网站相关的文件
    >  8. 执行hexo g -d生成网站并部署到GitHub上。
    
### 更换主题

- _config.yml 修改 theme: next
- 主题目录下找到_config.yml也可以进行修改
- 新生成page
```
 hexo new page tags 生成标签页 路径
 hexo new page categories  生成分类页面
```

参考：[使用Hexo+Github一步步搭建属于自己的博客（基础）](https://www.cnblogs.com/fengxiongZz/p/7707219.html)

参考：[hexo 修改主题](https://blog.csdn.net/zhy421202048/article/details/77877580)