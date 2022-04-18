---
title: linux 配置docker，部署mysql
date: 2022-04-18 19:57:28
tags: 
    - 环境配置
    - 云计算
    - Ubuntu
    - 实验报告
    - wsl
    - docker
categories: 云计算
math: false
index_img: /img/article/docker.png
---
## 1 安装

- 使用脚本安装

```bash
curl -sSL https://get.daocloud.io/docker | sh
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204042241275.png" alt="image-20220404224134678" style="zoom: 67%;" />

- 更改权限

```shell
sudo chmod 666 /var/run/docker.soc
```

![image-20220411162329378](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111623506.png)

- 查看版本

![image-20220411162445405](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111624567.png)

- 更改镜像源

```
cd /etc/docker
sudo vim daemon.json	
```

​	写入以下内容

```json
{
  "registry-mirrors": [
    "https://hub-mirror.c.163.com",
    "https://ustc-edu-cn.mirror.aliyuncs.com",
    "https://ghcr.io",
    "https://mirror.baidubce.com"
  ]
}
```

​	重启

```bash
service docker restart
```

![image-20220411162942084](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111629166.png)

## 2 docker部署mysql

### 2.1 拉取镜像

- 在[dockerhub](https://registry.hub.docker.com/_/mysql?tab=tags)选合适版本

![image-20220411163511252](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111635301.png)

- 拉取镜像

```dockerfile
docker pull mysql:8.0.28-oracle
```

![image-20220411163721573](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111637821.png)

- 查看镜像

```bash
docker images
```

![image-20220411163851617](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111638702.png)

### 2.2 启动实例

- 启动mysql实例

```bash
docker run --name some-mysql -e MYSQL_ROOT_PASSWORD=my-secret-pw -d mysql:tag
```

![image-20220411164304227](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111643276.png)

- 进入docker环境

```bash
docker exec -ti xxx(容器id) /bin/bash
```

- 登录mysql

```bash
mysql -u root -p
```

​		输入密码，登陆成功

![image-20220411165314050](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111653219.png)

- 创建数据库，插入数据

```mysql
create database dockertest;
use dockertest;
create table dt(name varchar(20), ID varchar(20));
insert into dt values('mysql 8.0.28','0ca4d64ca878');
select * from dt;
```

![image-20220411170318569](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111703656.png)

## 3 打包新镜像

- 退出镜像

```bash
quit();
exit
```

![image-20220411170653064](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111706145.png)

- 获得容器ID

```bash
docker ps -a
```

![image-20220411170809761](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111708916.png)

- 打包

```bash
docker commit 容器id 要保存的镜像名称
```

![image-20220411171027162](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111710251.png)

- 查看

```bash
docker images
```

![image-20220411171108709](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204111711817.png)



## 4 wsl2部署docker

### 4.1 原生部署

即使用上述方法，原生部署linux docker

### 4.2 docker desktop for windows

**Windows 安装docker desktop ,wsl2可以使用**

- 下载应用

  [下载链接](https://www.docker.com/products/docker-desktop/)

- 默认安装，一路下一步

  ![image-20220418200802552](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204182008720.png)

- 在wsl2中启用

  - 启用基于`WSL2`的引擎
  
    ![image-20220418200924854](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204182009968.png)
  
  - 允许安装的linux发行版使用
  
    ![image-20220418201010181](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204182010263.png)
  
  