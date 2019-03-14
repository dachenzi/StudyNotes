# 1 Linux上安装软件包的方式
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux下常用的软件包安装方式有三种。
## 1.1 编译安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;编译:将源代码变为机器可执行的代码文件。 安装:将可执行文件安装到操作系统里,才可以使用。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;编译安装的优缺点如下:  
- 优点
    1. 可以定制化安装目录
    2. 按需开启功能
- 缺点
    1. 需要查找并试验出适合的编译参数
    2. 编译时间过长
    3. 依赖包需要单独安装
## 1.2 yum安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;yum安装顾名思义，就是使用yum工具进行程序的安装。    
- 优点
    1. 自动处理依赖关系
    2. 自动化帮我们直接安装在操作系统中
- 缺点
    1. 不能定制化选择我们需要的功能模块
    2. 不能自定义安装目录
    3. 依赖于网络，网络不通，则无法安装
## 1.3 rpm安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`rpm`，Redhat Packages Manager，红帽包管理工具，使用rpm工具（-i）进行软件程序的安装。    
- 优点
    1. 本地安装
    2. 强大的查询以及软件包验证的功能
    3. yum安装方式，实质上安装的就是一个个rpm包
- 缺点
    1. 安装软件时，需要首先获取软件包依赖的所有包
    2. 无法直接处理依赖关系（需要制定—aid）参数
## 1.4 rpm定制+yum安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;综合了rpm的优点和yum的优点，定制化rpm包，自定义yum仓库，启用我们自己的yum源，这样可以使用yum帮我们一键安装软件，并执行某些操作，这在批量安装多台服务器的时候是非常有用的。
# 2 RPM包定制
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在linux环境下时，总是需要进行大量的软件安装和软件测试，如果从源码编译，可能要花费大量的编译时间，在确保包依赖关系正常的情况下，将安装好的软件打包成rpm包，可以很快的安装部署。打成rpm包有两种方式
1. 使用rpmbuild
2. 使用fpm打包  

__第一种方式我没有尝试过，太繁琐，一点一点写spec文件的参数，各种出错，一不留神就花费大半个小时；软件的产生就是为了方便系统管理员管理，减少不必要的时间浪费，，学会软件使用需要花费太多的时间掌握，实在是有点惨不忍睹。__
# 3 FPM方式打包
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;FPM功能简单说就是将一种类型的包转换成另一种类型。
## 3.1 支持的源类型的包
|名称|含义|
|:---:|:---|
|dir|将目录打包成需要的类型，可以用于源码编译安装的软件包|
|rpm|对rpm进行转换|
|gem|对rubygem包进行转换|
|python|对python模块打包成响应的类型|
## 3.2 支持的目标类型的包
|名称|含义|
|:--:|:---|
|rpm|转换为rpm包|
|deb|转换成deb包|
|solaris|转换为solaris包|
|puppet|转换为puppet模块|
## 3.3 安装FPM工具
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于fpm工具是ruby写的，因此系统环境需要安装ruby，且ruby版本号大于1.8.5。  
1. 安装ruby模块  
```bash
[root@centos7 ~]# yum install –y ruby rubygems ruby-devel
```
2. 添加ruby源（默认是国外的的，速度比较慢)  
```bash
[root@centos7 ~]#gem sources -a http://mirrors.aliyun.com/rubygems/
```
3. 移出原生的Ruby仓库
```bash
[root@centos7 ~]#gem sources –remove http://rubygems.org/
```
4. 安装fpm
```bash
[root@centos7 ~]#gem install fpm -v 1.3.3
```
## 3.4 FPM参数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;部分常用参数如下：
- -s 指定源*****
- -t 指定目标类型*****
- -n 指定包的名字*****
- -v 指定包的版本号*****
- -C 指定打包的路径*****
- -d 指定依赖于哪些包*****
- -f 第二次打包时目录下如果有同名文件，则覆盖它
- -p 输出的安装包的目录，不想放在当前目录下就需要指定
- --post-install 软件包安装完之后要运行的脚本；同--after-install*****
- --pre-install 软件安装完成之前要运行的脚本，同--before-install
- --post-uninstall 软件包卸载完成后要运行的脚本；同--after-remove
- --pre-uninstall 软件包卸载完成之前要运行的脚本；同--before-remove
# 4 实例
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们打包前需要首先安装先安装该软件（保存依赖包）
## 4.1 打开yum保存yum包参数
```bash
[root@centos7 ~]#sed -i ‘s#keepcache=0#keepcache=1#g’ /etc/yum.conf
```
因为yum每次安装完毕后，会自动清理安装包，所以开启yum的cache功能后，下载的包会保存并存放（默认）在`/var/cache/yum/x86_64/6/base/packages。`
## 4.2 编译安装nginx
下载安装依赖包
```bash
[root@centos7 ~]#yum install -y pcre pcre-devel openssl openssl-devel
```
创建运行nginx的用户
```bash
[root@centos7 ~]#useradd -M -s /sbin/nologin nginx
```
配置nginx（上传nginx安装包，并解压）
```bash
[root@centos7 ~]#cd nginx-1.6.2
[root@centos7 ~]#./configure --prefix=/application/nginx-1.6.2 --user=nginx --group=nginx --with-http_ssl_module --with-http_stub_status_module
```
编译安装
```bash
[root@centos7 ~]#make && make install
[root@centos7 ~]#ln -s /application/nginx-1.6.2/ /application/nginx
```
## 4.2 使用fpm工具进行打包
### 4.2.1 创建rpm安装完毕后要执行的脚本
```bash
[root@centos7 ~]#cd /server/scripts/
[root@centos7 ~]#vim nginx_rpm.sh
#!/bin/bash
useradd nginx -M -s /sbin/nologin
ln -s /application/nginx-1.6.2/ /application/nginx
# 这个脚本就是安装完rpm包要执行的脚本
```
### 4.2.2 打包
```bash
[root@beyond ~]# fpm -s dir -t rpm -n nginx -v 1.6.2 -d 'pcre-devel,openssl-devel' --post-install /server/scripts/nginx_rpm.sh -f /application/nginx-1.6.2/ 
no value for epoch is set, defaulting to nil {:level=>:warn}
no value for epoch is set, defaulting to nil {:level=>:warn}
Created package {:path=>"nginx-1.6.2-1.x86_64.rpm"}
[root@beyond ~]# ll -h nginx-1.6.2-1.x86_64.rpm
-rw-r--r-- 1 root root 6.7M Nov  1 10:02 nginx-1.6.2-1.x86_64.rpm
```
附录：(php打包参数)
```bash
fpm -s dir -t rpm -n php -v 5.5.32 -d 'zlib-devel libxml2-devel libjpeg-devel libjpeg-turbo-devel libiconv-devel freetype-devel libpng-devel gd-devel libcurl-devel libxslt-devel libiconv libmcrypt-devel mhash mcrypt ' --post-install /server/scripts/php_rpm.sh -f /application/php5.5.32/
[root@centos7 ~]#vim /server/scripts/php_rpm.sh
#!/bin/bash
useradd -M -s /sbin/nologin www
ln -s /application/php-5.5.32/ /application/php
/application/php/sbin/php-fpm &
```
### 4.2.3 安装定制好的包
使用rpm安装打好的包
```bash
[root@web02 tmp]# rpm -ivh php-5.5.32-1.x86_64.rpm
error: Failed dependencies:
        libiconv-devel is needed by php-5.5.32-1.x86_64
        fr is needed by php-5.5.32-1.x86_64
        eetype-devel is needed by php-5.5.32-1.x86_64
        libiconv is needed by php-5.5.32-1.x86_64
        mhash is needed by php-5.5.32-1.x86_64
        mcrypt is needed by php-5.5.32-1.x86_64
[root@web02 tmp]#
# 提示缺少依赖关系包
```
__使用yum安装,它会检测本地rpm包依赖的包，然后通过yum源下载安装。__
# 5 YUM仓库搭建
## 5.1 安装createrepo 软件
```bash
[root@centos7 ~]#yum -y install createrepo
[root@centos7 ~]#cd  /application/tools/
[root@centos7 ~]#rz  之前打包好的两个包
[root@centos7 ~]#tar xf nginx_yum.tar.gz                    # nginx的依赖包（keepcache=1，缓存的包）
[root@centos7 ~]#mkdir -p /application/yum/centos6/x86_64/       # 创建yum目录
[root@centos7 ~]#cd /application/yum/centos6/x86_64/
[root@centos7 ~]#cp /application/tools/*rpm .
```
## 5.2 初始化repodata目录并提供yum仓库服务
初始化
```bash
[root@centos7 ~]#createrepo -pdo /application/yum/centos6/x86_64/ /application/yum/centos6/x86_64/
# 会在/application/yum/centos6/x86_64/目录下生成repodata文件夹，里面存放的是索引文件
```
每加入一个rpm包就要更新一下
```bash
[root@centos7 ~]#createrepo --update /application/yum/centos6/x86_64/ 
```
启动一个简单的http服务，测试yum仓库（也可以用httpd服务或者nginx服务,这里用自带的模块启动）
```bash
[root@centos7 ~]#python -m SimpleHTTPServer 80 &>/dev/null &  
# 在yum仓库的路径下执行，例如（/application/yum/centos6/x86_64/）
```
## 5.3 客户端配置
```bash
[root@centos7 ~]#cd /etc/yum.repos.d
[root@centos7 ~]#mkdir yum_bak && mv *repo yum_bak           # 移出其他yum源
[root@centos7 ~]#vim beyondedu.repo                 # 创建yum源
[beyondedu]
name=Server            #源名称
baseurl=http://10.0.0.61   # 自行指定yum源地址（服务端定义的/application/yum/centos6/x86_64/）
enable=1           # 启用源
gpgcheck=0               # 是否进行安全性检查，0表示不启动（一般用来效验包）
[root@centos7 ~]#yum clean all            # 清空本机已有yum缓存
[root@centos7 ~]#yum list  # 列表显示yum仓库（在列表末尾会显示创建的源里面的包，@anaconda-ks表示的是本机已经安装的包）
```
## 5.4 测试
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;客户端直接下载nginx，由于我们之前已经打包了nginx所需的依赖包，所以所有包都是在本地指定的yum中下载，速度非常快~！