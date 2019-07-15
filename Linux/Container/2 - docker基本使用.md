# 1 docker的基本结构
docker的架构图如下：
![docker](../photo/docker.jpg)

- Client端  --> docker client
- Hosts(Docker_Host,server端)  --> docker daemon
- Registries  --> docker registries

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;docker是一个采用C/S结构的应用程序。当我们运行docker daemon是，这台主机变成了Server端，它默认情况下通过监听socket来接受client端的请求，当然服务器端还支持其他两种类型套接字(IPv4，IPv6)，默认是unix socket套接字文件

PS：docker client 到 docker Host 使用的是也是HTTPS（docker HOST 遵循Restful风格）

docker中的资源和对象：（restful中的资源是可以支持增删改查的操作)
- images：镜像
- containers：容器
- networks：网络
- volumes：存储卷
- plugins：插件

## 1.1 server端
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;运行docker daemon的主机就变成了DOCKER_HOST主机,也就是server端，它内部主要包含两部分：
- Containers:容器
- Images:镜像，来自于registry，镜像仓库，默认为docker hub，初始化是空的从远端下载(HTTP,HTTPS),默认为HTTPS，除非明确确认安全。

PS：镜像是静态的(文件)，容器是动态的，包含生命周期（类似进程）
## 1.2 仓库
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;docker提供的公共存储空间叫做docker hub,默认情况下我们的server都是从它那来下载镜像的，但是docker称它为 registry,那么为什么不不叫repository，而叫registry呢，其实它还包含了如下功能
- 镜像存储
- 用户认证
- 可用镜像的索引(搜索索引)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所以registry不仅仅是一个仓库，而是一个应用程序，包含多个仓库(repository)，一个仓库用于存放一个应用程序的镜像(一般仓库名就是应用程序名)，单个仓库加标签，唯一标示一个镜像(图)，如果没有执行标签，那么就是最新版,当然还有稳定版的标签(stable)指向对应的版本。

# 2 安装及使用docker
基础平台需求：
- 64bits CPU
- Linux Kernel 3.10+
- Linux Kernel cgroups and namespaces

这里使用Centos 7来安装，在centos 7的Extras repository源已经收录了docker-ce，所以只需要使用yum安装docker-ce即可
```bash
yum install -y docker-ce
```

## 2.1 配置文件
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;docker默认配置文件是json格式的，位置为：`/etc/docker/daemon.json`,第一次启动前时不存在的，可以手动创建

国内的主机访问dockerhub都很慢的，所以这里配置镜像站，来加速下载
```bash
vim /etc/docker/daemon.json
{
    "registry-mirrors:["https://xxx.com"]"
}
# 阿里云的需要自己注册账号来获取专属的加速链接
```
启动docker: `systemctl start docker.service`

## 2.2 主要命令
docker的命令主要分两类：
- 杂乱无章的(早期) 
- 分类管理(后期)

### 2.2.1 常用操作
docker image 系列包含如下命令：
- docker search: 在docker Hub中搜索镜像(docker image search)
- docker pull: 从仓库中拉取镜像(下载到本地)(docker image pill)
- docker images: 显示本地images列表(docker image ls)
- docker rmi: 删除一个或多个镜像(docker image rm)

docker container 系列命令包含如下命令
- docker create: 创建一个新的容器
- docker start: 启动一个容器
- docker run: 在一个新的容器中运行一个命令（创建并启动容器)
- docker attach: 
- docker ps: 列出容器
- docker logs: 列出容器日志(输出到控制台的日志)
- docker restart: 重启容器
- docker stop: 停止一个或多个运行的容器
- docker kill: 杀掉一个或多个运行的容器
- docker rm: 删除一个或多个容器


### 2.2.2 其他
在容器中执行命令：
- docker exec -it ContainerName /bin/bash: 在容器内执行一个shell

从其他仓库拉去镜像
- docker pull [仓库][端口(5000)]/[命名空间][镜像名]:[tag]

进入一个docker镜像
- docker exec -it 容器名称 /bin/bash

docker容器状态转换机制命令：
    图 docker event state

