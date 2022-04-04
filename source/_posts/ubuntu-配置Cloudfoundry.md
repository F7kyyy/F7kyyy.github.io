---
title: ubuntu 配置Cloudfoundry
date: 2022-04-04 23:43:13
tags: 
    - 环境配置
    - 云计算
    - Ubuntu
    - 实验报告
categories: 云计算
math: false
index_img: /img/article/CloudComputing.png
---

没错，又到了每周一度痛苦的配环境环节，更痛苦的是配完的环境可能再也不会打开。

![image-20220404235500586](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042355639.png)

[官方文档](https://docs.cloudfoundry.org/cf-cli/install-go-cli.html)

## 1 安装

- 添加仓库

  ```bash
  wget -q -O - https://packages.cloudfoundry.org/debian/cli.cloudfoundry.org.key | sudo apt-key add -
  echo "deb https://packages.cloudfoundry.org/debian stable main" | sudo tee /etc/apt/sources.list.d/cloudfoundry-cli.list
  ```

  ![image-20220404171709580](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204041717625.png)

- 更新系统

  ```bash
  sudo apt-get update
  ```

- 安装cli-v8版本

  ```bash
  sudo apt-get install cf8-cli
  ```

  ![image-20220404172248144](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204041722245.png)

- 测试安装

  ![image-20220404213210295](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042132354.png)

  

- 设置语言

  ![image-20220404213645622](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042136698.png)

## 2 设置环境并登录

- 注册 Cloudfoundry试用账户

  [链接](https://account.hanatrial.ondemand.com/cockpit#/home/trialhome)

![image-20220404220724578](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042207657.png)



- 设置API

![image-20220404221356694](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042213764.png)

- 登录

因为某种原因登陆失败

![image-20220404221443087](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042214245.png)

使用短期验证码登录

![image-20220404222010633](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042220777.png)

## 3 测试命令

- 列出组织名称

![image-20220404222807113](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042228192.png)

- 重命名

![image-20220404222929544](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042229618.png)

- 查看信息

![image-20220404223008663](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042230817.png)

- 显示用户

![image-20220404223144864](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042231986.png)



## 4 部署Python 版的hello world

###   4.1 安装docker

```bash
curl -sSL https://get.daocloud.io/docker | sh
```

![image-20220404224134678](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042241275.png)

docker可能并不是必备的，只是安装文档中说需要装，就装了:laughing:

### 4.2 使用Cloud Foundry CLI 安装 Cloud Foundry 本地插件

```bash
cf install-plugin cflocal
```

![image-20220404224439713](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042244912.png)

###   4.3 部署python flask版的hello world

#### 4.3.1 克隆项目

```bash
git clone https://github.com/YoheiFukuhara/cloudfoundry-python-flask-sample.git
```

![image-20220404224917681](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042249781.png)

#### 4.3.2 部署

进入文件夹

```bash
cf push cf-helloworld
```

![image-20220404231803170](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042318346.png)

更改文件:

- runtime.txt

  ```txt
  python-3.6.14
  ```

删除失败的

```bash
cf delete py-helloworld
```

![image-20220404232259210](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042322290.png)

重新push,成功

![image-20220404232447652](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042324836.png)

![image-20220404232719291](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042327333.png)

