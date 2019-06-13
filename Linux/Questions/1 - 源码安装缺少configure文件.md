## 源代码中没有configure的软件安装方法
今天下载了一个旧版的GeoIP软件包，解压以后发现代码包中没有configure文件，现在这这里记录一下安装遇到的问题：

## 生成configure文件的步骤
在软件包内执行如下命令：
- aclocal
- autoconf
- autoheader(出现什么AC_CONFIG_HEADERS not found in configure.ac 可以忽略) 
- automake --add-missing(出现ltmain.sh not found，需要执行autoreconf  -ivf)  --> 会生成Makefile.in 文件

然后就会生成configure文件，继续按照软件的INSTALL/README文件开始安装即可
