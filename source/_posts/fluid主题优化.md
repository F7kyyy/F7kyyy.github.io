---
title: fluid主题优化
date: 2022-03-17 23:33:12
tags: 
 - hexo
 - 花里胡哨
categories: 环境配置
math: false
index_img: /img/article/hexo_fluid.png
---

虽然fluid 主题已经非常好看了，但我依旧可以通过一些简单的操作，让你的博客更加易用（更花里胡哨和装逼）:smile:

## 1. Fluid 页脚增加网站运行时长

主题预览网站中有了很详细的介绍，这里就不在重复了

[配置链接](https://hexo.fluid-dev.com/posts/fluid-footer-custom/)，效果如下：

![202203172342237](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261337892.png)

## 2. markdown 使用emoji

默认不支持输入emoji表情，我们可以通过hexo-filter-emoji插件实现

```bash
npm install hexo-filter-emoji
```

之后在hexo配置文件，_config.html中添加

```yml
## hexo-filter-emoji
emoji:
  enable: true
  className: github-emoji
  styles:
  customEmojis:
```

效果如下:thumbsup::

![202203172346789](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261338900.png)

## 3. 使用Tidio进行实时交流

 Tidio是一个客服服务平台，可以在blog中实现Live Chat,存在网页端和移动端，可以及时回复消息，同时免费功能足够我们使用，效果如下：

![202203191703130](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261337746.png)

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203261338806.png" alt="202203191703265" style="zoom:50%;" />

- 首先进入 [Tidio 官网](https://www.tidio.com/) 进行注册，可能需要魔法上网

- 选择live chat 在settings 中调整自己的语句和外观

- 在setting->developver中复制public key

  <img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202207031414402.png" alt="202203191711294" style="zoom:50%;" />

- 在主题的配置文件中添加

  ````yml
  # Tidio online chat
  tidio:
    enable: true
    key: your public key
  ````

- 更改主题文件 

  /hexo-theme-fluid/layout\_partials/footer.ejs中添加

  ```ejs
  <!-- 在线通讯Tidio -->
  <% if (theme.tidio.enable){ %>
  	<script src="//code.tidio.co/your_public_key.js"></script>
  <% } %>
  ```

  <img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202207031416121.png" alt="image-20220703141600054" style="zoom:50%;" />

- 重新部署

  ```bash
  hexo clean
  hexo g
  hexo s
  ```

  