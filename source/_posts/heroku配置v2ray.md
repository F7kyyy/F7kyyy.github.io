---
title: heroku配置v2ray
date: 2022-04-19 18:49:38
tags:
    - 环境配置
    - 科学上网
categories: 环境配置
math: false
index_img: /img/article/heroku.png
---
**注意！本教程仅且只能用于研究与学习，只是做一个记录**

## 注意事项

1. *Heroku免费套餐每月只有**550小时**的免费时间，限量**2TB**，适合在非常时期当做备胎使用*
2. *Heroku对流量检测有些严格，请避免大流量消耗*
3. *引用项目原作者的话：Heroku提供给我们免费的服务，**我们不应该滥用它***
3. ***绝大部分的Heroku服务器节点都被twitter屏蔽了**，可能是我新建的暂时还没有*
4. *Heroku免费容器未使用超过一段时间（三十分钟左右）就会休眠*

## 必要的准备

1. 一个非国内邮箱地址，gmail 或者 outlook

2. Heroku与Cloudflare国内访问速度并不理想，可使用非国内网络环境

   (也就是说必须有一个机场，机场的免费时间一般来说就够用了)

3. 翻译工具，也可以使用Google Chrome浏览器

4. V2Ray软件，如果没有安装过请下载core版本，自带V2ray-Core[点击下载](https://github.com/2dust/v2rayN/releases)

## 一、服务端部署

###  1.1 注册Heroku账户

很简单，根据官网的提示操作就行:[点击注册](https://signup.heroku.com/)

注意：请使用外网IP注册，且请准备好翻译工具，并在收到邮件后激活账户；注册时不能使用QQ邮箱

### 1.2 在Heroku部署v2ray

点击下方按钮部署应用[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://dashboard.heroku.com/new?template=https://github.com/F7kyyy/heroku-v2ray)

ps: （2022.01.16）最近大佬们的仓库都被heroku给ban了，所以想要使用可以自建一个仓库，~~然后import大佬Fbclswl0827的heroku-v2ray项目自行部署~~

ps: 上面部署方式是我自己的仓库

####  Extra 创建github项目仓库

1、打开 [github](https://github.com/) 登录你自己的账号

2、点击左边的 `New` 创建一个仓库

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191901306.png" alt="img" style="zoom:67%;" />

3、在打开的页面中填写仓库名称，然后点击 `Create repository` 创建仓库

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191901796.png" alt="img" style="zoom:67%;" />

4、在新打开的页面下翻找到 `import`

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191905575.png" alt="img" style="zoom:67%;" />

5、在 `Your old repository’s clone URL` 中填入 `https://github.com/bclswl0827/v2ray-heroku.git`, 随后点击 `Begin import` 导入

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191901228.png" alt="img" style="zoom:67%;" />

**这位大佬的github仓库已经被disable了，可以选择其他人的，搜索一下应该很容易找到**

6、导入完成后，进入仓库，修改 `README.md` ，修改完成后点击下方 `Commit changes` 提交

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191901430.png" alt="img" style="zoom:67%;" />

7、最后进入仓库点击 `Deploy to Heroku` 图标开始部署

App name随便填写，可用就行；Choose a region就是你的服务器地区；UUID可自行修改（建议修改，使用默认UUID会使节点暴露在危险下）；然后点击`Deploy app`系统会自动部署

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191901573.png" alt="1.png" style="zoom:67%;" />

稍微等待一会儿，几秒的样子，直到全部打勾变绿

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191902144.png" alt="2.png" style="zoom:67%;" />

## 二、客户端使用

### 2.1 相关配置信息

点击Manage App进入你的项目；或者在https://dashboard.heroku.com/apps中找的到你的项目并进入

注意：你会看到一个项目，点击上方的`Settings`进入，查看你的V2Ray具体配置，如图

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191902864.png" alt="img" style="zoom:67%;" />

点击`Reveal Config Vars`显示V2RrayN相关配置信息，如图

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191902242.png" alt="img" style="zoom:67%;" /><img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191906266.png" alt="img" style="zoom:67%;" />

### 2.2 配置v2rayN

开始配置V2Ray，如果你记好了以上两个（那串字母UUID和二级域名xxxx.herokuapp.com）

ps: 二级域名不用加`https://`，直接填域名就好

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191902011.png" alt="img" style="zoom:67%;" />

最基础的配置完成了，可以发现节点的速度慢的令人发指，接下来我们利用cloudflare + 自选ip进行加速

## 三、Cloudflare Workers反代加速

对速度有要求的人群（强迫症患者）可以看一下；主要是使用Cloudflare Workers加速，虽然免费套餐有调用限制，但是一般个人使用不可能用完

### 3.1 创建Cloudflare Workers

在[Cloudflare Workers](https://dash.cloudflare.com/)中创建一个Workers

![img](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191902438.png)
![img](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191902747.png)

点击`快速编辑`进入项目编辑
![img](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191903865.png)
将原有的示例代码全部删除，复制如下代码，并将第四行的xxx.herokuapp.com 替换为你的V2Ray的地址 ps: 不需要`https://`

```js
VBSCRIPT
addEventListener(
	"fetch",event => {
		let url=new URL(event.request.url);
		url.hostname="xxxx.herokuapp.com";
		let request=new Request(url,event.request);
		event. respondWith(
			fetch(request)
		)
	}
)
```

点击右侧的`发送`按钮，看最后一行是否出现了`Bad Request`，出现则代表成功

![img](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191903829.png)

成功后，点击`保存并部署`，并记下你的Workers二级域名

### 3.2修改V2rayN中的配置

把V2RayN中原来的域名改为现在的Workers域名就行了，其实利用Cloudflare Workers进行反代以后速度已经可以了，但是优选ip以后速度会更加快
<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191903475.png" alt="img" style="zoom:67%;" />](https://gitee.co

**对速度有更高追求以及不怕折腾的人可以接着往下看，我个人测试，这个没什么太大的效果**

## 四、Cloudflare自选IP

点击下载[IP自选程序](https://github.com/badafans/better-cloudflare-ip/releases/lastest/download/batch.zip)，解压，在Windows系统下运行

其他的`使用说明.txt`文件中都有说，在此就不过多赘述

最后，配置V2Ray：

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191903472.png" alt="img" style="zoom:67%;" />

### 注意事项

由于各地的网络情况每天都不同，所以每天（甚至是每半天）的最优节点都不尽相同；但对于一个应急用的已经足够了

## 五、移动端使用

iOS端需要外区apple ID 平且基本移动端v2ray工具都需要付费下载，而这也需要信用卡较为麻烦，淘宝代充很多是盗刷信用卡，慎重考虑

### 5.1 下载apk

可以选择google play 直接下载v2rayNG,可能存在困难，所以推荐[github下载apk](https://github.com/2dust/v2rayNG) 直接选择release版本

如果想使用最新版，也可以自己使用Android Studio 编译安装

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204191903746.png" alt="image-20220322195545604" style="zoom:67%;" />

### 5.2 复制订阅

v2rayN 右键点击服务器->导出url到剪贴板->发送到手机

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203221958180.png" alt="image-20220322195844002" style="zoom:67%;" />

### 5.3 移动端使用

点击右上角加号，从剪贴板导入，即可使用

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203222004855.png" alt="image-20220322200417813" style="zoom:67%;" />

## 效果图

实际使用效果图：

- 4320P不要想了可以选择白嫖谷歌云3个月，但是需要国外信用卡，比较麻烦
- 2160P看情况
- 1440P，1080P没什么问题，Youtube码率比B站高得多 (当然你想看的视频可能还没有720P:smile:)

![image-20220321170725117](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202203211707537.png)

比某些大部分的机场免费套餐都好，甚至直逼少数机场的初级付费套餐；和我自己买的流量包10块30GB（他涨价了，之前50GB）差不太多，至少上个github，stackoverflow没什么太大问题。**在这个越来越严格的大环境下，在你的机场时不时抽风的时候，作为备选还是不错的。**

