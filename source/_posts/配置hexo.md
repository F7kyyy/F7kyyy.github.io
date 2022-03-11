---
title: 使用github,hexo搭建个人博客
date: 2022-03-08 15:36:48
tags: 
  - 环境配置
  - 云计算
categories: 实验报告
index_img: https://gitee.com/Fantastic-Feng/picgo/raw/master/202203111633391.png
---

# 山东大学  计算机科学与技术  学院 云计算技术  课程实验报告

## 1. 姓名学号

201900130128 冯子恺 数据19

## 2. 实验题目

Github + Hexo搭建个人博客系统

## 3. 实验目的

熟悉个人博客系统的搭建

## 4. 实验环境

- 硬件环境

  Intel Core I5-8300H 

  Nvidia Geforce gtx1060

- 软件环境

  Windows10 21H2


## 5. 实验步骤与内容

### 5.1注册github账号，下载git，node

之前已经做过，而且较为简单，这里不再重复

### 5.2 新建一个仓库

新建一个公共仓库，作为博客的部署的位置

### 5.3 配置ssh key

由于椭圆加密相同密钥长度下，安全性能更高同时计算量小，处理速度快，在私钥的处理速度上比RSA快的多，所以在生成密钥对时，使用ECC加密

```powershell
ssh-keygen -t ecdsa
```

将公钥上传到github,测试连接

```powershell
ssh -T git@github.com
```

连接成功

![](https://gitee.com/Fantastic-Feng/picgo/raw/master/202203072016135.png)

### 5.4 配置hexo

- 安装hexo

```bash
npm install -g hexo
```

- 新建一个文件夹,初始化博客

```bash
hexo init
```

-  安装依赖包，确保git部署

```bash
npm install hexo-deployer-git --save
```

此时本地已经配置好博客

使用命令启动

```bash
hexo g
hexo s
```

### 5.5 更换主题

依据个人喜好更换主题，这里使用Aurora

### 5.6 博客部署在github.io

- 复制SSH链接

  <img src="https://gitee.com/Fantastic-Feng/picgo/raw/master/202203072023291.png" alt="" style="zoom: 80%;" />

- 编辑 config_yml

  ```yaml
  deploy:
    type: git
    repository: git@github.com:Fantastic-Feng/Fantastic-Feng.github.io.git
    branch: main
  ```

- 配置Deploy keys 

  与SSH配置方法相同，存在bug，添加后不显示，不过可以使用

- 部署

  ```bash
  hexo g
  hexo d
  ```

  <img src="https://gitee.com/Fantastic-Feng/picgo/raw/master/202203072028743.png" alt="" style="zoom:50%;" />

### 5.7 访问

https://f7kyyy.github.io/

<img src="https://gitee.com/Fantastic-Feng/picgo/raw/master/202203072031185.png" alt="" style="zoom: 50%;" />

### 5.7 添加一篇博客

生成一个新的md文件，在 ./source/_posts文件夹下

```bash
hexo new "new article"
```

```bash
hexo g     //生成静态页面
hexo s    //启动本地服务器进行查看
hexo d   //查看后没有问题即可部署到github上
```

<img src="https://gitee.com/Fantastic-Feng/picgo/raw/master/202203072055102.png" alt="" style="zoom:50%;" />

##   6. 结论分析与体会

网络上程序员好像人手一个博客，之前想过用java 和 vue自己实现一个博客项目，太麻烦了就一直没弄

hexo还是比较方便的，同时有各种主题可以选择，还是很不错的
