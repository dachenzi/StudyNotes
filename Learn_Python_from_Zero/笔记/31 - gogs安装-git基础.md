# 1 Gogs安装
Gogs 是一款极易搭建的自助 Git 服务。Gogs 的目标是打造一个最简单、最快速和最轻松的方式搭建自助 Git 服务。使用 Go 语言开发使得 Gogs 能够通过独立的二进制分发，并且支持 Go 语言支持的 所有平台，包括 Linux、Mac OS X、Windows 以及 ARM 平台。
## 1.1 安装准备
> 以下步骤均在CentOS 7.6下测试成功
1. 更新yum源
```bash
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
```
2. 安装git
```bash
yum install -y git
```
3. 安装数据库
支持MySQL、Mariadb、PostgreSQL、SQLite等
```bash
yum install -y mariadb-server
```
## 1.2 启动数据库
1. 启动数据库
```bash
systemctl start mariadb-server
```
2. 基础配置
```bash
# 初始化数据库
mysql_secure_installation
```
按照提示删除test库，以及创建数据库的root等
