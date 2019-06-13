## 源代码中没有configure的软件安装方法
今天下载了一个旧版的GeoIP软件包，解压以后发现代码包中没有configure文件，现在这这里记录一下安装遇到的问题
网上大部分GeoIP下载地址已经失效，因为GeoIP新版本GeoIP2，所以这里附旧版Geoip的下载地址：[GeoIP下载地址](https://github.com/maxmind/geoip-api-c/releases)

## 生成configure文件的步骤
在软件包内执行如下命令：
- `aclocal`
- `autoconf` --> 生成configure文件
- `autoheader`(出现什么AC_CONFIG_HEADERS not found in configure.ac 可以忽略)
- `automake --add-missing`(出现ltmain.sh not found，需要执行`autoreconf  -ivf`)  --> 会生成Makefile
.in 文件

然后就会生成configure文件，继续按照软件的INSTALL/README文件开始安装即可

参考： [生成configure](https://blog.csdn.net/BabyBirdToFly/article/details/69941756)
