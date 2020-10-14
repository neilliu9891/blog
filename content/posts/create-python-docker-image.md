---
title: "Create Python Docker Image"
date: 2020-10-14T07:07:11+08:00
tags: ["docker"]
categories: ["技术杂谈"]
draft: fasle
---

# Create Python Docker Image
记一次基于centos docker镜像创建python 3.7.9版本镜像的过程, 创建流程比较简单，目的记录使用了docker创建镜像的两种方式。

Q: 为什么不使用官方的python镜像
A: 官方python镜像基于debian操作系统实现，而项目中多使用centos操作系统，为了保持一致，所以更新centos操作系统;构建的程序需要后台运行，基于systemctl 编写启动脚本实现，python官方镜像中并没有systemctl

---

文档记录并解决两个问题！

1. 基于Centos7.8镜像如何安装生成python3.7.9的镜像文件
2. 如何实现基于systemctl 服务的docker镜像的启动，以及如何编写k8s yaml文件实现systemctl

## 基础资源

官方docker仓库 
> https://hub.docker.com/search?q=&type=image
> https://hub.docker.com/_/centos/?tab=description

## 生成python基础镜像
### 基于镜像commit方式生成新镜像

1. 根据docker hub 搜索并下载合适的centos版本

```
docker pull centos:7.8.2003
```

2. 通过run命令启动下载下来的image镜像并进入容器

```
docker run -it centos:7.8.2003 /bin/bash
```

3. 执行安装python所必要的基础工具包(容器内执行)
```
yum install wget

yum install yum-utils

yum-builddep python3 # 下载python依赖 

yum install make # 编包使用
```

4. 安装python
```
wget https://www.python.org/ftp/python/3.7.9/Python-3.7.9.tgz #下载指定版本python包

tar xf Python-3.7.9.tgz  # 解压包

cd Python-3.7.9

./configure && make && make install # 配置、编译、安装

```

5. 退出并执行如下命令生成新镜像
```
docker ps -a | grep centos:7.8.2003 # 查看刚才启动的docker 容器

cfe4d2255644        centos:7.8.2003                            "/usr/sbin/init"         18 hours ago        Up 18 hours 

docker diff cfe4d2255644 # 显示差异，由于安装内容太多没必要显示

docker commit -m "centos python3.7.9" cfe4d2255644 imagename:tag # -m提交注释 ,imagename:tag表示新生成的容器名称     

```

### 基于文件的方式生成新镜像

基于文件的方式生成镜像其实就是将之前的命令基于Dockerfile的形式执行一遍。

1. 生成Dockerfile，内容如下：

```
FROM centos:7.8.2003

RUN yum install wget && yum install yum-utils && yum-builddep python3 && yum install make
RUN wget https://www.python.org/ftp/python/3.7.9/Python-3.7.9.tgz && tar xf Python-3.7.9.tgz && cd Python-3.7.9 && ./configure && make && make install

CMD [/bin/bash]

```

2. 执行生成镜像命令

```
docker -f Dockerfile . -t imagesname:tag
```

### 推送镜像到远端仓库

1. 给镜像打远端仓库的tag

```
docker tag centos7.8-python3.7.9:latest 10.254.7.1:5000/imagesname:tag

```

2. 执行push命令推送镜像

```
docker push 10.254.7.1:5000/imagesname:tag

```

## 如何在容器内运行systemctl命令以及后台运行服务

systemctl作为后台服务管理的工具，能够通过编写/etc/systemd/system/*.service的方式实现对服务的运行重启等功能。

Q: python程序想要在不更换docker镜像的情况下，修改python代码并重新运行，只要docker不重启，服务就不会失效的功能，如何实现？
A: 通过编写service文件通过systemctl监控实现。

Q: systemctl 在docker中运行失败，错误提示
```
System has not been booted with systemd as init system (PID 1). Can't operate.
Failed to connect to bus: Host is down
```
```
FailedtogetD-Busconnection: Operation not permitted
```

1. 启动时添加--privileged=true
2. 使用/usr/sbin/init
3. 使用-itd（即-d后台运行否则一直会卡在那里，没有命令行终端可用）
4. example
```
docker run -itd --privileged=true images:tag /usr/sbin/init
docker ps -a | grep image:tag # 获取容器的id
docker run -it container_id /bin/bash # container_id:实际的容器id
```

