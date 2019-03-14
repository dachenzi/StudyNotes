# 1 nginx介绍
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HTTP协议包含很多功能。常用的www，world wide web是http功能之一。默认端口为80端口，也被称之为7层协议。  
常用web软件：apache、nginx（静态web服务软件）。  
流行的web组合：
- LAMP (Linux Apache Mysql Php) =====>经典
- LNMP(Linux Nginx Mysql Php)=====>流行（或者LEMP，E（engine X 这样读法）
- nginx由俄罗斯人开发，是一款开源的，支持高性能，高并发的www服务和代理服务软件，一共780K，C语言开发。nginx本身是一款静态（html。css，js，jpg等）www的软件，不能解析动态的PHP,JSP,DO。因具有高并发（特别是静态资源）占用系统资源少等有点，且功丰富，而流行起来。淘宝，在nginx做了更改（优化）生成 tengine。
## 1.1 nginx重要特性
1. 支持高并发：能支持几万并发连接（特别是静态小文件业务环境）
2. 资源消耗少：在3万并发连接下，开启10个Nginx线程消耗不到200M内存。
3. 可以做HTTP反向代理及加速缓存，即负载均衡功能，内置对RS节点服务器健康检查功能，这相当于专业的haproxy软件或lvs功能。
4. 具备squid等专业缓存软件等的缓存功能。
5. 支持异步网络IO事件模型epoll（Linux2.6+）
## 1.2 nginx主要功能
1. 作为web服务软件：  
    - 运行HTML、JS、CSS、小图片等静态数据（此功能类似lighttpd软件）  
    - 结合FastCGI运行PHP等动态程序（例如使用fastcgi_pass方式）。  
    - 结合tomcat/resin等支持Java动态程序（常用）
2. 作为反向代理或负载均衡服务：  
    - 为web服务、PHP服务等动态服务及Memcached缓存提供代理。  
    - 可以作为邮件代理服务软件。  
    - 在1.9.0版本以后支持tcp/IP代理，之前是不支持的。
3. 作为前端的数据缓存服务器：  
    - 通过自身的proxy_cache模块实现类squid等专业缓存软件的功能。
## 1.3 nginx与其他web软件对比
__`apache`__
1. 2.2版本非常稳定强大，据官方说，其2.4版本性能超强。
2. Prefork模式取消了进程创建开销，性能很高。
3. 处理动态业务数据时，因关联到后端的引擎和数据库，瓶颈不在apache本身。
4. 高并发时消耗系统资源相对多一些。
5. 基于传统的select模型。
6. 支持扩展库，DSO方法，apxs方法编译安装额外的插件功能，不需要重新编译
7. 功能多，更稳定，更安全，插件多
8. 市场份额在逐年降低  

__`Nginx`__
1. 基于异步IO模型(epoll,kqueue),性能强，能够支持上万并发。
2. 对小文件支持很好，性能很高（限静态小文件1M）。
3. 代码优美，扩展库必须编译进主程序。
4. 消耗系统资源比较低。
5. 支持web、proxy、cache三大重点功能，并且都很优秀。
6. 市场份额在逐年增加  

__`Lighttpd`__
1. 基于异步I/O模型，性能和Nginx相近。
2. 扩展库是SO模式，比Nginx要灵活。
3. 目前国内的使用率比较低，安全性没有apache和nginx好。
4. 通过插件（mod_secdownload）可实现文件URL地址加密。
5. 社区不活跃，市场份额较低。
## 1.4 select和epoll的区别
select和epoll的区别就相当于同步和异步的区别，apache使用select模型，nginx使用epoll模型。
Apache select和Nginx epoll区别技术对比：
|指标|select|epoll|
|:---:|:---:|:----:|
|性能|随着连接数增加，急剧下降。处理成千上万并发连接数时，性能很差|随着连接数增加，性能基本没有下降。处理成千上万并发连接时，性能很好。|
|连接数|连接数有限制，处理的最大连接数不超过1024.如果要处理超过1024个连接数，则需要修改FD_SETSIZE宏，并重新编译。|连接数无限制。|  
|内在处理机制|线性轮训|回调callback|
|开发复杂性|低|中|
## 1.5 web服务器选择
- 静态业务：高并发，采用nginx或lighttpd，根据自己的掌握程度或公司的要求，首选nginx。
- 动态业务：理论上采用nginx和apache均可，建议选择Nginx，要避免相同业务服务软件多样化，额外增加维护成本，动态业务可以由Nginx兼作前端代理，再根据页面元素的类型或者目录，向后转发到后端相应的服务器进行处理。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;既有静态业务又有动态业务：nginx。动态业务可以由前端代理（haproxy），根据页面元素的类型，向后转发相应的服务器进行处理。如果并发不是很大，又对apache很熟悉，采用apache也是可以的，apache2.4版本也很强大，并发连接数也有所增加。总的来说，在满足需求的前提下，先选择自己最擅长的软件，若看上了更好的软件，可在掌握新软件之后逐步替换。虽然动态和静态业务都倾向于选择Nginx，但是大前提是自己要熟悉掌握Nginx。切记企业工作中不要盲从，这可能最终会导致自己无法控制给企业带来灾难的恶果。
# 2 安装nginx
## 2.1 安装前的准备工作
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先要安装`PCRE`包，PCRE(Perl Compatible Regular Expressions)是一个Perl库，包括 `perl` 兼容的正则表达式库。安装pcre库是为了使`nginx`支持`HTTP rewrite`（域名跳转）模块。其次还要安装`openssl`包，因为nginx在使用HTTPS服务的时候要用到此模块（http over ssl ）需要用到ssl进行加密。
```bash
[root@web01 conf]# yum install -y pcre pcre-devel
[root@web01 conf]# yum install -y openssl openssl-devel
```
## 2.2 安装nginx
### 2.2.1 使用yum安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用yum安装nginx，首先需要有配置epel源（因为默认源里面没有nginx包）  
__安装epel源的方法：__
```bash
yum install -y epel-release
wget -O /etc/yum.repos.d/epel.repo http:#mirrors.aliyun.com/repo/epel-6.repo
# 安装完epel源后，就可以使用yum安装nginx了
yum install -y nginx
# 由于yum安装无法自定义编译参数，所以一般不用这种方法。
```
### 2.2.2 使用rpm安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用rpm安装nginx，首先需要自行下载rpm类型的nginx安装包，然后通过 `rpm –ivh `进行安装，但是rpm无法解决软件间的依赖关系，所以一般也不用这种方法。
### 2.2.3 编译安装
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;编译安装的好处在于，我们在编译的时候可以自行制定需要以及不需要的功能模块。在nginx官网下载nginx源码包，解压之后进入到目录中进行编译。
```bash
[root@web01 nginx-1.6.3]# ls
auto     CHANGES.ru  configure  html     Makefile  objs    sbin
CHANGES  conf        contrib    LICENSE  man       README  src
[root@web01 nginx-1.6.3]#
# 使用./confiruare命令进行配置编译参数
[root@web01 nginx-1.6.3]#./configure --user=nginx --group=nginx --prefix=/application/nginx1.6.3 --with-http_stub_status_module --with-http_ssl_module
[root@web01 nginx-1.6.3]#make && make install 编译完毕后直接安装
[root@web01 nginx-1.6.3]#ln –s /application/nginx-1.6.3 /application/nginx
# 注意：
# 由于我们指定了软件的用户，所以我们首先要创建这个用户
# 创建链接目录是为了让其他人员调用时方便（比如我升级了nginx版本，我只需要把新的版本链接到nginx目录，这样上层程序不用# 更改。
# 在每一步之后，都可以使用echo $?来测试结果，0表示成功。
```
__编译参数说明：__
- --prefix=：表示软件的安装路径。
- --user=：启动进程时的用户。
- --group=：启动进程时的用户组。
- --with-http_stub_status_module：启动状态信息模块。
- --with-http_ssl_module：启动ssl加密模块。
## 2.3 启动nginx服务
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;安装完毕后，并不能直接提供服务，需要我们手动的开启服务。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开启服务之前，首先要测试配置文件的正确性（如果修改了配置文件之后）。
```bash
[root@web01 nginx-1.6.3]# /application/nginx/sbin/nginx -t
nginx: the configuration file /application/nginx-1.6.3/conf/nginx.conf syntax is ok
nginx: configuration file /application/nginx-1.6.3/conf/nginx.conf test is successful
[root@web01 nginx-1.6.3]#    #显示OK，表示正确
[root@web01 nginx-1.6.3]# /application/nginx/sbin/nginx  #开启服务
# 如果修改配置文件，不想停服，可以用 -s reload 来进行优雅重启
```
## 2.4 检查启动结果
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在启动完毕后，我们要进行验证。
1. 检查进程是否开启
```bash
[root@web01 nginx-1.6.3]# ps aux | grep nginx
root       1514  0.0  0.4  45256  1784 ?        Ss   15:02   0:00 nginx: master process /application/nginx/sbin/nginx
www        1546  0.0  0.4  45256  1696 ?        S    15:06   0:00 nginx: worker process       
root       2096  0.0  0.2 103308   844 pts/1    S+   17:29   0:00 grep nginx
[root@web01 nginx-1.6.3]#
```
2. 检查端口是否监听
```bash
[root@web01 nginx-1.6.3]# lsof -i :80
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
nginx   1514 root    6u  IPv4  11354      0t0  TCP *:http (LISTEN)
nginx   1546  www    6u  IPv4  11354      0t0  TCP *:http (LISTEN)
[root@web01 nginx-1.6.3]#
[root@web01 nginx-1.6.3]# netstat -lntup | grep nginx
tcp        0      0 0.0.0.0:80                  0.0.0.0:*                   LISTEN      1514/nginx         
[root@web01 nginx-1.6.3]#
```
这时，我们就可以使用crul命令来测试默认页面了。
```bash
[root@web01 www]# !curl
curl 127.0.0.1
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
body {
width: 35em;
margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif;
}
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
[root@web01 www]#
```
## 2.5 客户端验证
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;开启了服务，由于nginx有默认页面，这时，我们就可使用浏览器进行访问了。打开浏览器直接访问我们web服务器监听的IP地址，就可以获得请求页面。  
__如果无法获取：__
1. ping 服务器地址 物理通不通
2. telnet 服务器地址 80 浏览器到web通不通（防火墙）
3. 服务器本地curl 127.0.0.1
4. 更换浏览器进行尝试
# 3 nginx基本信息
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nginx之所以强大，是因为它有各种更能区块。
1. Nginx core modules 必需的  
包括Main、Events
2. Standard HTTP modules 缺省都会安装，不建议改动  
常用的包括`core`（核心的http参数），`access`（访问控制模块），`fastCGI`（动态应用相关，PHP），`proxy`（代理模块），`upstream`（负载均衡模块），`rewrite`（URL重写模块）,`stub_status`（静态信息模块），`ssl`（ssl模块）
## 3.1 nginx目录结构
```bash
[root@web01 /]# tree /application/nginx
/application/nginx
├── client_body_temp
├── conf                      #配置文件
│   ├── fastcgi.conf                  #fastCGI配置文件
│   ├── fastcgi.conf.default
│   ├── fastcgi_params           #fastCGI参数文件
│   ├── fastcgi_params.default
│   ├── koi-utf
│   ├── koi-win
│   ├── mime.types                  #支持的mine类型
│   ├── mime.types.default
│   ├── nginx.conf           #主配置文件
│   ├── nginx.conf.bak
│   ├── nginx.conf.basename
│   ├── nginx.conf.default
│   ├── scgi_params                #scgi支持的参数
│   ├── scgi_params.default
│   ├── uwsgi_params            #uwsgi支持的参数
│   ├── uwsgi_params.default
│   └── win-utf
├── fastcgi_temp
├── html                      #网站的根目录
│   ├── 1.html
│   ├── 1.jpg
│   ├── 50x.html
│   ├── bbs
│   │   └── index.html
│   ├── blog
│   │   └── index.html
│   ├── index7.html
│   ├── index.html
│   ├── index.html.28
│   ├── index.html.bak
│   └── www
│       └── index.html
├── logs                                #日志
│   ├── access.log                    #默认的访问日志
│   ├── error.log                       #默认的错误日志
│   └── nginx.pid                       #nginx的进程文件
├── proxy_temp
├── sbin                       #命令目录
│   └── nginx
├── scgi_temp
└── uwsgi_temp

12 directories, 31 files
[root@web01 /]#
```
## 3.2 主配置文件
```bash
[root@web01 conf]# cat nginx.conf
worker_processes  1;    #nginx进程数，建议设置为等于CPU总核心数。
error_log /tmp/error.log info;          #全局错误日志定义类型
events {                                #事件区块开始
    worker_connections  1024;            #单个进程最大连接数
}
http {                  #http区段开始
    include       mime.types;            #包含类型文件
    default_type  application/octet-stream;       #默认媒体类型
    sendfile        on;                                   #开启高效传输模式
    keepalive_timeout  65;                             #连接超时时间
#############www.beyondlee.org#############
server {                                          #server模块开始
    listen       80;                       #监听端口80
    server_name www.beyondlee.org;                    #提供服务的域名
    location / {                                    #location区块开始（可以有多个location区块）
        root   html/www;          #该server站点根目录
        index  index.html index.htm;         #该server站点主页
        }
    }
}
[root@web01 conf]#
```
# 4 nginx虚拟主机
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;虚拟主机，在web服务里就是一个独立的web站点，这个站点对应独立的域名，独立的IP，以及独立的资源，可以独立的对外提供服务。这个站点使用server{}来进行标记的，而在apache中用`<virtualHost></virtualHost>`来标记  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nginx虚拟主机分为三个类型：
- 基于域名的虚拟主机。通过域名来区分虚拟主机，应用：外部网站
- 基于端口的虚拟主机。通过端口来区分虚拟主机，应用：公司内部网站，外部网站的后台
- 基于ip的虚拟主机。不常用。
## 4.1 配置基于域名的虚拟主机
```bash
[root@web01 ~]# cat /application/nginx/conf/nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.beyondlee.org;
    location / {
        root   html/www;
        index  index.html index.htm;    
        }
    }
    server {
    listen       80;
    server_name  bbs.beyondlee.org;
    location / {
        root   html/bbs;
        index  index.html index.htm;
        }
    }
    server {
        listen       80;
        server_name  blog.beyondlee.org;
        location / {
            root   html/blog;
            index  index.html index.htm;
            }
        }
    }
```
注意：
1. 复制一个完整的server标签段，到结尾，注意：要放在http的结束大括号前，也就是server标签段放入http标签
2. 更改server_name及对应网页的root根目录。
3. 检查配置文件语法，平滑重启服务reload。
4. 创建server_name对应网页的根目录，并且建立测试文件，如果没有index首页会出现403错误。
5. 在客户端对server_name的主机名做host解析或DNS配置，并检查（ping域名看返回的IP对不对）。
6. windows浏览器访问，或者在linux客户端做host解析，用wget或curl访问。
## 4.2 配置基于端口的虚拟主机
```bash
[root@web01 conf]# cat nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       80;
        server_name  www.beyondlee.org;
        location / {
            root   html/www;
            index  index.html index.htm;
            }
        }
    server {
        listen       81;
        server_name  bbs.beyondlee.org;
        location / {
            root   html/bbs;
            index  index.html index.htm;
        }   
    }
    server {
        listen       82;
        server_name  blog.beyondlee.org;
        location / {
            root   html/blog;
            index  index.html index.htm;
        }
    }
}
```
## 4.3 配置基于ip的虚拟主机
```bash
[root@web01 conf]# cat nginx.conf
worker_processes  1;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    server {
        listen       10.0.0.8:80;
        server_name  www.beyondlee.org;
        location / {
            root   html/www;
            index  index.html index.htm;
        }
    }
    server {
        listen       10.0.0.101:80;
        server_name  www.beyondlee.org;
        location / {
            root   html/bbs;
            index  index.html index.htm;
        }
    }
    server {
        listen       10.0.0.102:80;
        server_name  www.beyondlee.org;
        location / {
            root   html/blog;
            index  index.html index.htm;
            }
        }
    }
```
注意：
- 不同的虚拟主机监听不同的IP地址。
- 客户端请求不同的IP地址获取不同的界面。
- 可以使用ip或ifconfig命令给一个网卡配置多个IP地址（临时）
    - `ip addr add 192.168.1.1/24 dev eth0 label eth0:0`
    - `ifconfig eth0:1 192.168.1.2/24 up`
# 5 nignx常用功能
## 5.1 虚拟主机规范化
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;生产场景中会针对不同的虚拟主机建立不同的配置文件以及虚拟主机根目录、日志等，这个时候如果再把server写在主配置文件中会非常的乱，这样我们就可以把虚拟主机写在某个目录下，然后在主配置文件中定义它们的位置就可以了。
```bash
[root@web01 conf]# cat nginx.conf
worker_processes  1;
error_log /tmp/error.log info;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    include      vhost/*.conf;          #包含vhost下的*.conf配置文件
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
}
[root@web01 conf]#
[root@web01 conf]# cd vhost/
[root@web01 vhost]# ls
bbs.conf  blog.conf  www.conf
[root@web01 vhost]# cat www.conf              #只定义虚拟主机配置文件
server {
    listen       80;
    server_name www.beyondlee.org;
    location / {
        root   html/www;
        index  index.html index.htm;
    }
}
[root@web01 vhost]#
```
## 5.2 nginx主机别名
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所谓主机别名就是为虚拟主机配置多个名称，这样就能实现用户访问的多个域名对应统一虚拟主机网站的功能。
```bash
[root@web01 vhost]# cat www.conf
server {
    listen       80;
    server_name www.beyondlee.org lixin.beyondlee.org;
    location / {
        root   html/www;
        index  index.html index.htm;
    }
}
[root@web01 vhost]#
```
## 5.3 nginx状态信息
在配置文件里配置如下内容
```bash
[root@web01 vhost]# cat www.conf
server {
    listen       80;
    server_name www.beyondlee.org lixin.beyondlee.org;
    location / {
        root   html/www;
        index  index.html index.htm;
        stub_status on;
        access_log off;
    }
}
[root@web01 vhost]#
```
打开：lixin.beyondlee.com有如下显示
```bash
Active connections: 145
server accepts handled requests
1749    1749         3198
Reading: 0 Writing: 3 Waiting: 142
```
__各字段含义：__
1. `Active connections`: nginx正处理的活动连接数145个。
2. `Server`: nginx启动到现在共处理了 1749个连接
3. `accepts`: nginx启动到现在共成功创建 1749次握手 请求丢失数=(握手-连接),可以看出，我们没丢请求;
4. `handled requests`: 总共处理了3198 次请求。
5. `Reading` ：nginx读取到客户端的Header信息数。
6. `Writing` ： nginx返回给客户端的Header信息数。
7. `Waiting` ： Nginx已经处理完正在等候下一次请求指令的驻留连接.开启keep-alive的情况下,这个值等于active–(reading+writing)。
## 5.4 nginx记录错误日志
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nginx软件会把自身运行的故障信息（main区域的时候）及用户访问的日志信息（server区域的时候）记录到指定的日志文件里。nginx错误日志，属于核心功能模块的参数，该参数名称为error_log，可以放在main区块中的全局配置，也可以放在不同的虚拟主机中单独记录。  
error_log的语法格式如下：
```bash
# error_log       logs/error.log       error;
# 关键字              日志文件         日志级别
[root@web01 conf]# cat nginx.conf
worker_processes  1;
error_log /tmp/error.log info;
events {
    worker_connections  1024;
}
http {
    include       mime.types;
    include      vhost/*.conf;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
}
[root@web01 conf]#
```
__error_log的日志级别有：[ `debug` | `info` | `notice` | `warn` | `error` | `crit` | `alert`| `emesg` ]，常用的有warn,error,crit。注意不要使用info，会使磁盘I/O增加。__
## 5.5 nginx访问日志
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;nginx会把每个访问信息记录到访问日志中，提供给分析者进行分析。  
主要有两个参数负责：
1. log_formart：用来定义记录信息的格式。
2. access_log：用来指定记录日志文件的路径以及格式。  
默认的日志格式如下：
```bash
log_format access '$remote_addr - $remote_user [$time_local] "$request" '
'$status $body_bytes_sent "$http_referer" '
'”$http_user_agent" “$http_x_forwarded_for”';
```
__各字段含义如下：__
```bash
$remote_addr 记录访问客户端的IP地址
$remote_user 远程客户端用户名称
$time_local 记录访问时间与时区
$request 用户的HTTP请求信息
$status http状态码
$body_bytes_sent 服务发给客户端的响应body字节数
$http_referer 记录客户端是从哪个连接访问过来的（可以根据它来进行防盗链的设置）
$http_user_agent 用户浏览器标识
$http_x_forwarded_for 当前端有代理服务器时，设置web节点记录客户端的地址的配置，参数生效的前提是代理服务器也进行了相关X_forwarded_for的设置
```
注意：先定义格式，然后包含的主机配置文件才能使用，所以要先定义log_format定义格式， 再 include  vhost/*.conf
```bash
[root@web01 vhost]# cat www.conf
server {
listen       80;
server_name www.beyondlee.org;
log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
          '$status $body_bytes_sent "$http_referer" '
          '"$http_user_agent" "$http_x_forwarded_for"';
location / {
    root   html/www;
    index  index.html index.htm;
    #stub_status on;
    }

    access_log logs/www_access.log main;
}
[root@web01 vhost]#    #定义完毕后，就可以直接调用了
```
注意：在高并发场合，添加buffer和flush选项，可以提升网站性能 `access_log logs/www_access.log main gzip buffer=32K flush=5s;`
## 5.6 访问日志轮训切割
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于nginx没有自带的日志切割工具，那么只能我们手动的进行切割了，要用到cron+mv.  
写一个脚本：
```bash
#!/bin/bash
d=`date -d "-1 day" +%Y%m%d`
[ -d /usr/local/nginx/access-log] || mkdir -p /usr/local/nginx/access-log
mv /tmp/access.log /usr/local/nginx/access-log/$d.log
/etc/init.d/nginx reload 2>/dev/null

# 然后每天晚上0点运行
crontab –l
#######cut for nginx by lixin######
00 00 * * * /bin/bash  /server/scripts/nginx_log_cut.sh  &>/dev/null
```