---
title: ubuntu 配置openstack
date: 2022-04-01 15:27:03
tags: 
    - 环境配置
    - 云计算
    - Ubuntu
    - 实验报告
categories: 云计算
math: false
index_img: /img/article/CloudComputing.png
---

## 实验题目

面向IaaS的OpenStack部署

##  实验目的

在Linux环境下，熟悉OpenStack环境。

具体包括：了解OpenStack编程环境的配置和部署，完成实验环境及实验工具的熟悉，撰写实验报告。

## 实验环境

- 硬件环境

  Intel Core I5-8300H 

  Nvidia Geforce gtx1060

- 软件环境

  VMware Workstation 16

  ubuntu 20.04

## 实验步骤与内容

> 选择安装MicroStack MicroStack是一个OpenStack上游发行版

### 安装

```bash
sudo snap install microstack --beta
```

按如下方式查看已安装信息

```bash
snap list microstack
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204011529741.png" alt="image-20220328232126221" style="zoom:50%;" />

在这里，我们看到OpenStack Ussuri已经部署！

### 初始化

初始化步骤会自动部署、配置和启动 OpenStack 服务。

```bash
sudo microstack init --auto --control
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204011529360.png" alt="image-20220328233234671" style="zoom:50%;" />

### 验证

验证步骤的目的是确认云处于工作状态。验证将包括以下操作：

- 执行各种 OpenStack 查询
- 创建实例
- 通过 SSH 连接到实例
- 访问云仪表板

#### 列出默认的image

```bash
microstack.openstack image list
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204011529907.png" alt="image-20220328233602010" style="zoom:67%;" />

#### 获取默认的实例列表

```bash
microstack.openstack flavor list
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204011529029.png" alt="image-20220328233752779" style="zoom:67%;" />

#### 创建实例

MicroStack附带了一个方便的实例创建命令`microstack launch`，它对其实例使用以下默认值：

- keypair `microstack`
- flavor `m1.tiny`
- floating IP address on subnet `10.20.20.0/24`

基于"cirros"映像创建名为"demo"的实例

```bash
microstack launch cirros -n demo
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204011529345.png" alt="image-20220328234855179" style="zoom: 50%;" />

使用与默认密钥对关联的私有 SSH 密钥访问实例：

```bash
ssh -i /home/feng/snap/microstack/common/.ssh/id_microstack cirros@10.20.20.222
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204011529537.png" alt="image-20220328235122956" style="zoom:60%;" />

#### 访问云仪表盘

- 访问: https://10.20.20.1:443

- 获得密码

```shell
sudo snap get microstack config.credentials.keystone-password
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204011529297.png" alt="image-20220328235644333" style="zoom:67%;" />

- 登录

  <img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204011529574.png" style="zoom: 50%;" />
  
#### 关闭

openstack 在启动时会占用大量cpu,启动后，会一直占据至少4g内存，让我们本来就性能羸弱的虚拟机不堪重负:crying_cat_face:。我们可以选择关闭它。

```bash
# 关闭
sudo snap stop microstack
# 取消开机自启
sudo snap disable microstack
```

当然我们也可以重新启动

```bash
sudo snap enable microstack
sudo snap start microstack
```



##  结论分析与体会

Openstack是一个非常流行的创建私有云的框架，但是对资源消耗过大，安装较为繁琐。

使用MicroStack安装过程较为简单，但是因为其版本还停留在beta阶段，可能存在问题，需要以后解决。

  

  

  

   
