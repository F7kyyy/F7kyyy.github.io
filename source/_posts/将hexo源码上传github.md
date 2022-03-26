---
title: 将hexo源码上传github
date: 2022-03-11 20:25:53
tags:
    - 环境配置
    - hexo
categories: 环境配置
math: false
index_img: /img/article/hexo_github.png
---



在将博客部署在github后，因为仓库的默认分支中存储的是静态html，而不是源码，所以在重新部署时会非常麻烦，需要重新配置。因为本身是一个喜欢折腾的人，为了防止某一天环境搞乱后追悔莫及，也为了方便查看配置文件更改了什么内容，所以使用版本控制是十分必要的。（额。。每个主题config不同，而我未来换主题的次数可能有辣么一点点多。)

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261340923.png" alt="202203112113312" style="zoom:50%;" />

网上的教程挺多的，但是有点过程十分繁琐而且没有必要。在尝试后，我总结了相对简单的步骤，使用vscode 进行图形化git操作，因为我身边的大部分人可能只会git clone 

### 1. 新建分支

一般来说，hexo生成的静态html文件都保存在master分支，我们需要新建一个分支用来存储源代码，我新建了source分支

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261341357.png" alt="202203112044392" style="zoom:50%;" />

然后把source设置为默认分支

**Settings->Branches->Default branches**

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261341110.png" alt="202203112047709" style="zoom:67%;" />

### 2. 本地操作

将仓库克隆到本地，使用vscode打开，左下角应该显示source分支

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261342795.png" alt="202203112051236" style="zoom:80%;" />

把里边的东西删掉，因为source是从master clone 过来的静态html我们并不需要这些东西，保留.git文件夹

然后将你的hexo文件夹下的所有内容复制到该文件下(执行hexo clean & g &s 的目录) 如果主题是clone下来的，删除主题文件夹下的.git文件夹

<img src="https://gitee.com/Fantastic-Feng/picgo/raw/master/202203112054538.png" alt="image-20220311205410479" style="zoom: 67%;" />

一般来说在初始化会自己有.gitignore文件，没有的话就自己新建

```xml
.DS_Store
Thumbs.db
db.json
*.log
node_modules/
public/
.deploy*/
_multiconfig.yml
```

这样的目的是把无关的内容去掉，因为我们只需要config以及博客.md，npm依赖包和生成的静态html我们并不需要

再次确定**博客是部署在非source分支**，一般在_config.yml文件下

```yml
deploy:
  type: git
  repository: https://github.com/username/username.github.io.git
  branch: master
```



### 3. 推送

将更改提交，并且推送到github

![202203112104237](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261345221.png)

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261346720.png" alt="202203112106897" style="zoom:67%;" />

这时我们就成功了，在本地做更改时，我们会将更改的config文件和md文件推送到source分支

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261346277.png" alt="202203112109311" style="zoom:67%;" />

而执行hexo d 会部署在master分支

![202203112110969](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261346698.png)![202203112110060](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261347781.png)

至此我们的目的就达到了，这样在弄些花里胡哨的东西时就不用担心出现错误无法回退了。

