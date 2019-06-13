## nginx 安装第三方echo模块
openresty是基于nginx的一个分支版本，echo模块用于打印出内置变量的值，这样可以方便我们知晓整个过程，便于排错，对于nginx来说，可以使用echo-nginx-module来添加echo模块。

## 添加echo-nginx-module模块
1. 下载echo-nginx-module模块 
[echo-nginx-module下载地址](https://github.com/openresty/echo-nginx-module/releases)

2. 解压后重新编译nginx
只需要在原有的nginx编译命令后添加如下命令
```bash
$ ./configure ... ...--add-module=/opt/software/echo-nginx-module-0.61/
$ make && make install 
```

## 基本使用
在nginx.conf中使用 echo 即可输出打印的变量
[nginx内置变量](https://nginx.org/en/docs/http/ngx_http_core_module.html#variables)

比如nginx配置文件
```bash
location /hello {
    echo $http_user_agent
	echo $host
}
```
