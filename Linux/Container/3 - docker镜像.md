# 1 docker image 镜像
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;docker host 从docker registries 中获取镜像并存入到本地，所以要求docker host本地能够存储这种镜像，这种存储空间要求是一种特殊而且专用的文件系统(1.18 overlay2存储驱动)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;镜像可以理解为应用程序的集装箱(docker(码头工人)负责转配这些集装箱)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Docker镜像含有启动容器所需要的文件系统及其内容，因此，其用于创建并启动Docker容器
- 采用分层构建机制，大体上分为两部分：最底层为bootfs，其之上为rootfs(图，rootfs层(/etc,/bin,/sbin等)，bootfs层)
    - bootfs：用于系统引导的文件系统，包括bootloader和kernel，容器启动完成后会被卸载以节约内存资源（从内存中移除)
        底层为aufs/btrfs/overlay文件系统，来确保能够引导并启动一个用户空间
    - rootfs：位于bootfs之上，表现为docker容器的根 文件系统
        - 传统模式中，系统启动之时，内核挂载rootfs时会首先将其挂载为“只读”模式，完整性自检完成后将其重新挂载为读写模式；
        - docker中，rootfs由内核挂载为“只读”模式，而后通过“联合挂载”技术额外挂载一个“可写”层。 
    - 位于下层的镜像称为父镜像(parent image)，最底层的称为基础镜像(base image)
    - 最上层为“可读写”层，其下的均为“只读”层。(图，可写层，http层，vim层，base image层，bootfs层)

## 专有文件系统
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;docker的这种分层构建，联合挂载，需要的是专有的文件系统的支持，比如Aufs、overlay2，brtfs等。

### Aufs文件系统
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Aufs：advanced multi-layered unification filesystem：高级多层统一文件系统
- 用于为Linux文件系统实现”联合挂载“
- aufs是之前的UnionFS的重新实现，2006年由Junjiro Okajima开发；
- Docker最初使用aufs作为容器文件系统层，它目前仍作为存储后端之一来支持；
- aufs的竞争产品是overlayfs，后者自从3.18版本开始被合并到Linux内核；
- docker的分层镜像，除了aufs、docker还支持btrfs，devicemapper和vfs等
    - 在Ubuntu系统下，docker默认Ubuntu的aufs；而在Centos7上，用的是devicemapper(DM)；

### devicemapper文件系统




### overlay2文件系统
overlay2是一个抽象的二级文件系统，它需要构建在一个本地文件系统之上，这个文件系统就是xfs(extfs)
## docker registry
启动容器时，docker daemon会视图从本地获取相关的镜像；本地镜像不存在时，其将从registry中下载该镜像并保存到本地；(图，docker Client ,http/https, Docker Daemon(stroage Driver, aufs,dm), Public Docker registry DockerHub, Private Docker Registry,Storage Driver)

PS: docker distribution：docker提供的一个镜像服务，但是不完成，默认不支持HTTPS服务，在安全情况下，docker daemon要求 docker registry必须是HTTPS服务的，如果不是，需要配置docker daemon来明确指示其使用不安全的镜像仓库(现在更流行使用vmware开源的harbor)

分类：
- Registry用于保存docker镜像，包括镜像的层次结构和元数据
- 用户可自建Registry，也可以使用官方的Docker Hub
- 分类：
    - Sponsor Registry：第三方的registry，供客户和Docker社区使用
    - Mirror Registry：第三方的registry，只让客户使用(加速器，比如阿里云)
    - Vendor Registry：由发布docker镜像的供应商提供的registry（比如红帽提供的供购买它服务的客户使用)
    - Privage Registry：通过设有防火墙和额外安全层的私有实体提供的registry

组成：
一般由两部分组成：Repository、Index
- Repository
    - 有特定的docker镜像的所以迭代版本组成的镜像仓库
    - 一个Registory中可以存在多个Repository
        - Repository可以分为”顶层仓库”和“用户仓库“
        - 顶层仓库名称格式为“仓库名”
        - 用户仓库名称格式为”用户名/仓库名“
    - 每个仓库可以包含多个Tag(标签),每个标签对应一个镜像，但一个镜像可以对应多个标签

- Index
    - 维护用户账户、镜像的校验以及公共命名空间的信息
    - 相当于为Registry提供了一个完成用户认证等功能的检索接口

docker registry中的镜像来自于何处？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通常由镜像开发人员制作，而后推送至“公共”或“私有”Registry上保存，供其他人员使用，例如“部署”到生产环境；

镜像相关的操作
1 镜像的生成途径
- dockerfile
- 基于容器制作
- Docker Hub Automated Builds


基于容器制作镜像：
- 正在运行的容器，不能是已经停止的
    docker commit -p  web    (-p表示暂停，否则如果容器内的某个文件正在读写，会造成文件不完整的情况)
    这种方式是没有tag和仓库名称的
- 为自定义容器加tag
    docker tag ca87c7dc2d2c dachenzi/nginx:latest
    一个镜像可以包含多个标签，一个标签只能针对一个镜像

    修改容器启动时运行的命令：
        docker commit -a "dachenzi <dachenzi@gmail.com>" -c 'CMD ["/bin/httpd","-f","-h","/data/html/"]' -p dachenzi/httpd:v0.2

- 将制作好的镜像上传至dockerhub（镜像的名称要和dockerhub仓库的名称一致，否则上传失败)
    docker login -u xxx -p 
    docker push dachenzi/nginx[tag] 不加tag就是上传所有的tag

- 将制作好的镜像上传至阿里云的镜像仓库内


镜像的导入和导出：
- 仅仅是本地的导出，再另一台主机上导入测试
- docker save ：打包
    docker save -o myimages.gz dachenzi/nginx:latest dachenzi/nginx:stable  可以写多个,一次打包多个镜像
- docker load ：导入
    docker load -i myimages.gz
