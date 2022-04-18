---
title: ubuntu 配置hadoop伪分布式
date: 2022-04-18 17:51:34
tags:
    - 环境配置
    - 云计算
    - Ubuntu
    - 实验报告
categories: 云计算
math: false
index_img: /img/article/hadoop.jpg
---
`这里选择安装伪分布式的形式`[原文链接](https://www.jianshu.com/p/f5a6c4d888e0)

## 1 安装前准备

下载hadoop和jdk安装包，因为伪分布式版本要求，我们选择hadoop2.7.3 与java 1.8

下载链接：[hadoop](http://archive.apache.org/dist/hadoop/core/hadoop-2.7.3/?C=S;O=A),[jdk](https://www.openlogic.com/openjdk-downloads?field_java_parent_version_target_id=416&field_operating_system_target_id=426&field_architecture_target_id=391&field_java_package_target_id=396)

![image-20220418162025511](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181620856.png)

## 2 安装 ssh server

```bash
sudo apt-get install openssh-server
```

```bash
ssh localhost
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181622284.png" alt="image-20220418162251454" style="zoom:67%;" />

```bash
## 注销
exit
```

![image-20220418162356263](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181623321.png)

## 3 生成密钥对

```bash
cd ~/.ssh
ssh-keygen -t rsa
cat ./id_rsa.pub >> ./authorized_keys
```

<img src="https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181625708.png" alt="image-20220418162526500" style="zoom:67%;" />



```bash
## 测试免密登录
ssh localhost
```

![image-20220418162640904](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181626156.png)

## 4 安装jdk

- 创建文件夹

  ```bash
  sudo mkdir -p /usr/lib/jvm
  ```

  ![image-20220418163001936](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181630012.png)

- 解压jdk到/usr/lib/jvm

  ```bash
  sudo tar zxvf jdk-8u101-linux-x64.tar.gz -C /usr/lib/jvm
  ```

  ![image-20220418163216346](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181632450.png)

- 配置环境变量

  ```bash
  sudo vim /etc/profile
  ```

  ```bash
  #set java environment
  export JAVA_HOME=/usr/lib/jvm/openjdk8
  export JRE_HOME=${JAVA_HOME}/jre
  export CLASSPATH=.:{JAVA_HOME}/lib:${JRE_HOME}/lib
  export PATH=${JAVA_HOME}/bin:$PATH
  ```

  ![image-20220418163326224](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181633362.png)

- 查看安装版本

  ```bash
  ## 使环境变量生效
  source /etc/profile
  ## 查看java版本
  java -verison
  ```

  ![image-20220418163641795](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181636936.png)

## 5 配置hadoop

### 5.1 解压重命名

```bash
## 解压
sudo tar zxvf hadoop-2.7.3.tar.gz -C /usr/local
## 重命名
sudo mv /usr/local/hadoop-2.7.3 /usr/local/hadoop
```

![image-20220418164137152](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181641233.png)

### 5.2 配置环境变量

```bash
sudo vim /etc/profile
```

```bash
#set hadoop path
export HADOOP_HOME=/usr/local/hadoop
export PATH=$PATH:$HADOOP_HOME/bin
```

![image-20220418164255774](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181642843.png)

```bash
## 使环境变量生效
source /etc/profile
```

### 5.3 更改配置文件

- 修改权限

  ```bash
  ## feng 是我的用户名
  sudo chown -R feng /usr/local/hadoop/
  ```

  ![image-20220418164607904](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181646959.png)

- 修改JAVA_HOME

  可以直接使用vscode 远程连接虚拟机，对vim命令不太熟悉的我们更方便

  ```bash
  cd /usr/local/hadoop/etc/hadoop/
  vim hadoop-env.sh
  ```

  ![image-20220418165328146](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181653183.png)

- core-site.xml

  添加内容

  ```xml
  <property>
      <name>hadoop.tmp.dir</name>
      <value>file:/usr/local/hadoop/tmp</value>
  </property>
  <property>
      <name>fs.defaultFS</name>
      <value>hdfs://localhost:9000</value>
  </property>
  ```

  ![image-20220418165512841](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181655882.png)

- hdfs-site.xml

  ```xml
      <property>
              <name>dfs.replication</name>
              <value>1</value>
      </property>
      <property>
              <name>dfs.namenode.name.dir</name>
              <value>file:/usr/local/hadoop/tmp/dfs/name</value>
      </property>
      <property>
              <name>dfs.datanode.data.dir</name>
              <value>file:/usr/local/hadoop/tmp/dfs/data</value>
      </property>
  ```

  ![image-20220418165702338](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181657389.png)
  
- mapred-site.xml

  ```bash
  mv mapred-site.xml.template mapred-site.xml
  ```

  ```bash
          <property>
               <name>mapreduce.framework.name</name>
               <value>yarn</value>
          </property>
  ```

  ![image-20220418165927102](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181659157.png)

- yarn-site.xml

  ```xml
          <property>
               <name>yarn.nodemanager.aux-services</name>
               <value>mapreduce_shuffle</value>
          </property>
  ```

  ![image-20220418170204186](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181702228.png)

## 6 格式化

```bash
hadoop namenode -format
```

成功的话，会看到 “successfully formatted” 和 “Exitting with status 0” 的提示，若为 “Exitting with status 1” 则是出错

![image-20220418170547835](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181705171.png)

## 7 启动

```bash
cd /usr/local/hadoop
sbin/start-all.sh
```

![image-20220418170911109](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181709333.png)

## 8 查看jps

```bash
jps
```

![image-20220418170952885](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181709994.png)

出现这个说明hadoop安装成功



## 9 上传文件

- 创建用户文件夹

  ```bash
   ./bin/hdfs dfs -mkdir /user
   ./bin/hdfs dfs -mkdir /user/feng
  ```

  ![image-20220418172719186](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181727300.png)

- 本地新建一个文件

  ```bash
  vim hadooptest1.txt
  ```

  ![image-20220418173843280](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181738321.png)

- 上传文件

  ```bash
   ./bin/hdfs dfs -put hadooptest1.txt
  ```

- 查看

  ```bash
  ./bin/hdfs dfs -ls
  ```

  ![image-20220418172816100](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181728211.png)

  ```bash
   ./bin/hdfs dfs -cat hadooptest1.txt
  ```

  ![image-20220418173024463](https://cdn.jsdelivr.net/gh/F7kyyy/picture@main/img/202204181730556.png)

## 总结

hadoop在一年前配置过环境，hadoop,hdfs,hbase,mapeduce,断断续续搞了一个月。

如今重新配置，也还是配置了一个多小时。

hadoop作为一个分布式系统基础架构，hdfs提供了安全可靠的分布式文件存储系统，mapreduce更是一个高效的超大规模数据处理方法。 

可惜因为效率等种种原因，hadoop逐渐被淘汰了，不过其推广的mapreduce思想永不过时。
