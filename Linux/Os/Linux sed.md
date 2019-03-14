## 1 功能说明
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sed 是一种在线编辑器，它一次处理一行内容。处理时，把当前处理的行存储在临时缓冲区中，称为“模式空间”（pattern space），接着用sed命令处理缓冲区中的内容，处理完成后，把缓冲区的内容送往屏幕。接着处理下一行，这样不断重复，直到文件末尾。文件内容并没有改变，除非你使用重定向存储输出或者加入i参数。Sed主要用来自动编辑一个或多个文件；简化对文件的反复操作；编写转换程序等。      
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__小结：sed的功能是，对字符串进行增加、删除、改变、查找，即增删改查!__
# 2 语法格式
```bash
sed [optian] [sed-command] [input-file]

# 注意sed和后面的选项之间至少有一个空格。
# 为了避免混淆，称呼sed为sed软件。sed-commands(sed命令)是sed软件内置的一些命令选项，为了和前面的options(选项)区分，故称为sed命令。
# sed-commands既可以是单个sed命令，也可以是多个sed命令组合。
# input-file(输入文件)是可选项，sed还能够从标准输入如管道获取输入。
```
# 3 常用选项
测试文本内容为：
```
[root@lixin ~]# cat daxin.txt 
I am daxin teacher!
I teach linux.

I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.cnblogs.org
my qq num is 49000448.

not 4900000448.
my god ,i am not dachenzi,but daxin!
gd
good
gooood
[root@lixin ~]#
```
## 3.1 取消标准输出
-n：表示取消标准输出（默认情况下sed替换后会把文件所有内容包括更改过的内容一起输出到屏幕上，我们使用-n后，让它只输出匹配表达式的字符串，一般和p连用，p表示print，打印表达式结果）。
```bash
[root@lixin ~]# sed  '/old/p' daxin.txt    
I am daxin teacher!
I am daxin teacher!
I teach linux.

I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
my blog is http://daxin.blog.51cto.com
our site is http://www.cnblogs.org
my qq num is 49000448.

not 4900000448.
my god ,i am not dachenzi,but daxin!
my god ,i am not dachenzi,but daxin!
gd
good
gooood
[root@lixin ~]# sed  -n '/old/p' daxin.txt 
I am daxin teacher!
my blog is http://daxin.blog.51cto.com
my god ,i am not dachenzi,but daxin!
[root@lixin ~]#     //-n和p连用后表示只打印符合表达式规则的行
```
## 3.2 支持扩展正则表达式
-r：参数用来支持扩展正则表达式，比如{}符号，如果不加-r参数需要使用\来进行转意。
```bash
[root@lixin ~]# sed -n '/o\{1,2\}/p' daxin.txt 
I am daxin teacher!
I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.cnblogs.org
not 4900000448.
my god ,i am not dachenzi,but daxin!
good
gooood
[root@lixin ~]# sed -nr '/o{1,2}/p' daxin.txt    
I am daxin teacher!
I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.cnblogs.org
not 4900000448.
my god ,i am not dachenzi,but daxin!
good
gooood
[root@lixin ~]#  //表达式含义：o最少重复1次，最多重复2次
```
## 3.3 直接修改文件内容
-i：表示直接对目标文件进行操作（增删改查），加上这个参数之后，不会在屏幕上输出信息，会直接对源文件进行更改(慎用)。
```bash
[root@lixin ~]# sed -i '/^$/d' daxin.txt   //直接删除文件内的空行，并没有打印。
[root@lixin ~]# cat daxin.txt  
I am daxin teacher!
I teach linux.
I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.cnblogs.org
my qq num is 49000448.
not 4900000448.
my god ,i am not dachenzi,but daxin!
gd
good
gooood
[root@lixin ~]#
```
# 4 sed-command
测试文本内容为：
```bash
[root@lixin ~]# cat study.txt 
1,I am a Linux student
2,I am studying Linux
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]#
```
## 4.1 增加
-i：表示插入，插入到目标行之前。
```bash
[root@lixin ~]# sed '2i she is a girl' study.txt 
1,I am a Linux student
she is a girl
2,I am studying Linux
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]#   //2i表示在第二行插入，不加行号表示在每一行之前插入
```
-a：表示在目标行之后添加
```bash
[root@lixin ~]# sed  '3a he is a boy' study.txt  
1,I am a Linux student
2,I am studying Linux
3,I like movice
he is a boy
4,I am super man
5,I like computer games
[root@lixin ~]#
```
多行增加
```bash
# 方法一：利用\n,表示换行符
[root@lixin ~]# sed '2i 1\n2\n3' study.txt 
1,I am a Linux student
1
2
3
2,I am studying Linux
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]#

# 方法二：利用shell特性（\回车）表示接着下一行。
[root@lixin ~]# sed '2i 1\
> 2\
> 3 \' study.txt
1,I am a Linux student
1
2
3
2,I am studying Linux
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]#
```
### 4.1.1 限定区域添加
sed软件可以对单行或多行进行处理。如果在sed命令前面不指定地址范围，那么默认会匹配所有行。  
```bash
n1[,n2]{sed-commands}
# 注：地址用逗号分隔的，n1,n2可以用数字、正则表达式、或二者的组合表示。
```
例子：  
`n{sed-commands}` 对第n行操作。
```bash
[root@lixin ~]# sed '5c she is a girl ' study.txt 
1,I am a Linux student
2,I am studying Linux
3,I like movice
4,I am super man
she is a girl 
[root@lixin ~]#   //表示针对第武行进行替换，替换为she is a girl
```
`n,m{sed-commands}`对n到m行操作,包括第n,m行。
```bash
[root@lixin ~]# sed '2,4c she is a girl ' study.txt 
1,I am a Linux student
she is a girl 
5,I like computer games
[root@lixin ~]#   //表示把2到4行替换成she is a girl。
```　　　
`n，+m{sed-commands}`对n到n+m行操作,包括第n,m行
```bash
[root@lixin ~]# sed '2,3d' study.txt 
1,I am a Linux student
4,I am super man
5,I like computer games
[root@lixin ~]# //表示2到3行删除
```
`1~2{sed-commands}` 对1,3,5,7,……行操作
```bash
[root@lixin ~]# sed '2~2d' study.txt  
1,I am a Linux student
3,I like movice
5,I like computer games
[root@lixin ~]#   //表示对2,4,6,8等偶数行进行操作
```
`n，${sed-commands}` 对n到最后一行($代表最后一行)操作,包括第n行
```bash
[root@lixin ~]# sed '2,$d' study.txt 
1,I am a Linux student
[root@lixin ~]# //删除2行到$(尾行)
```
`/Linux/{sed-commands}` 对匹配daxin的行操作
```bash
[root@lixin ~]# sed '/Linux/d' study.txt 
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]#   //表示把包涵Linux的行删掉
```
`/Linux/,/movice/{sed-commands}`对匹配daxin的行到匹配Alex的行操作
```bash
[root@lixin ~]# sed '/Linux/,/movice/d' study.txt  
4,I am super man
5,I like computer games
[root@lixin ~]#   //表示把第一个包涵Linux到第一个包涵movice的行删掉
```
`/movice/,${sed-commands}`对匹配daxin的行到最后一行操作
```bash
[root@lixin ~]# sed '/movice/,$d' study.txt 
1,I am a Linux student
2,I am studying Linux
[root@lixin ~]# //表示从movice到最后一行删掉
```
`/games/,10{sed-commands}`对匹配daxin的行到第10行操作，注意：如果前10行没有匹配到daxin，sed软件会显示10行以后的匹配daxin的所有行，如果有。
```bash
[root@lixin ~]# sed '/games/,4d' study.txt    
1,I am a Linux student
2,I am studying Linux
3,I like movice
4,I am super man
[root@lixin ~]#
```
`1,/am/{sed-commands}` 对第1行到匹配Alex的行操作
```bash
[root@lixin ~]# sed '1,/am/d' study.txt 
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]#   //表示从1行到第一个匹配到am的行删除
```
`/student/,+2{sed-commands}` 对匹配daxin的行到其后的2行操作
```bash
[root@lixin ~]# sed '/student/,+2d' study.txt 
4,I am super man
5,I like computer games
[root@lixin ~]# //表示从匹配student的第一项到它的后两行进行删除
```
## 4.2 删除
-d：表示删除，可以直接删除行，可以删除包涵关键字的行
```bash
[root@lixin ~]# sed '2d' study.txt 
1,I am a Linux student
3,I like movice
4,I am super man
5,I like computer games
games       //删除第2行
[root@lixin ~]# sed '/am/d' study.txt 
3,I like movice
[root@lixin ~]#   //删除包涵am的行
```
## 4.3 查找
p表示打印，把符合我们表达式的项目打印出来，由于sed默认输出所有，再使用拍的话会打印两遍，所以一般和-n参数连用，表示只打印匹配到的行。
```bash
[root@lixin ~]# sed '/am/p' study.txt  //由于没有-n所以会把匹配到的行和默认输出一起打印
1,I am a Linux student
1,I am a Linux student
2,I am studying Linux
2,I am studying Linux
3,I like movice
4,I am super man
4,I am super man
5,I like computer games
5,I like computer games
games
games
[root@lixin ~]# sed -n '/am/p' study.txt  //-n取消默认输出，p打印，所以只显示匹配项
1,I am a Linux student
2,I am studying Linux
4,I am super man
5,I like computer games
games
[root@lixin ~]# sed –nr ‘/am|student/’ study.txt  //查找包含am或student的行
1,I am a Linux student
[root@lixin ~]# sed -n '2p' study.txt 
2,I am studying Linux
[root@lixin ~]#   //只显示第二行
```
## 4.4 改
-c：按行替换，表示一次替换一整行的内容。
```bash
[root@lixin ~]# sed '1c she is a girl' study.txt 
she is a girl
2,I am studying Linux
3,I like movice
4,I am super man
5,I like computer games
games
[root@lixin ~]# //表示把第1行，替换成she is a girl 
```
### 4.4.1 文本替换
s：表示全局查找某个关键字，并替换每行第一个匹配项。  
g：表示全局替换，不属于sed命令，一般搭配s使用。
```bash
# 格式：
sed  ‘s#1#2#g’
#:表示定界符，可以是/、@、或者其他成对符号。
# 1：表示要替换的源文件中的字符串（可以是正则表达式）。
# 2：表示要替换成的文件，必须是给定的字符串。
```
文本替换实例：
```bash
[root@lixin ~]# !ca
cat study.txt 
1,I am a Linux student
2,I am studying Linux
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]# sed  '1s#a#b#' study.txt    //把第一个a替换为b
1,I bm a Linux student
2,I am studying Linux
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]#
[root@lixin ~]# sed  '1s#a#b#g' study.txt  //把第一行所有a替换为b
1,I bm b Linux student
2,I am studying Linux
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]#
```
变量替换实例：
```bash
[root@lixin ~]# n=a       //设定变量n的值为a
[root@lixin ~]# m=c     //设定变量m的值为c
[root@lixin ~]# echo $n   //验证变量n的值
a
[root@lixin ~]# echo $m   //验证变量b的值
c
[root@lixin ~]# sed '1s#$n#$m#g' study.txt  //由于’’单引号，不解释内部变量值所以没有替换
1,I am a Linux student
2,I am studying Linux
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]# sed "1s#$n#$m#g" study.txt  // “” 双引号表示引用内部的变量，所以替换了 
1,I cm c Linux student
2,I am studying Linux
3,I like movice
4,I am super man
5,I like computer games
[root@lixin ~]#  //表示可以利用变量的值来对字符串进行过滤或替换（必须用””双引号）
```
### 4.4.2 后向引用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;sed软件的( )的功能可以记住正则表达式的一部分，其中，\1为第一个记住的模式即第一个小括号中的匹配内容，\2第二记住的模式，即第二个小括号中的匹配内容，sed最多可以记住9个。这种在后面调用的方式，叫做后向引用。
```bash
# 格式：
sed ‘s#()()#\1\2#g’
# 第一个（）表示第一项，第二个（）表示第二项，一次类推，最多九个，不想匹配的可以写在括号后面。
# \1表示引用前面第一个（）括号内的内容，\2表示引用前面第二个（）括号内的内容。
```
实例：  
```bash
# 获取ip地址信息
[root@lixin ~]# ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 00:0C:29:E4:83:06  
    inet addr:10.0.0.8  Bcast:10.0.0.255  Mask:255.255.255.0
    inet6 addr: fe80::20c:29ff:fee4:8306/64 Scope:Link
    UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
    RX packets:981 errors:0 dropped:0 overruns:0 frame:0
    TX packets:562 errors:0 dropped:0 overruns:0 carrier:0
    collisions:0 txqueuelen:1000 
    RX bytes:85039 (83.0 KiB)  TX bytes:62853 (61.3 KiB)
[root@lixin ~]# ifconfig eth0 | sed -nr '2s#^.*r:(.*).?B.*$#\1#gp'
10.0.0.8  
[root@lixin ~]#
# ^.*r: 表示以任意字符开头到r：为止。
# (.*) 表示任意字符，这里就是我们要取得ip信息。
# .?B.*$ ，.?表示匹配任意两个字符，B.*$表示从B到结尾。
```
特殊符号的使用:  
批量进行文件后缀名更改  
```bash
[root@lixin lixin]# ls 
LinuxAdministartor10.bak  LinuxAdministartor2.bak  LinuxAdministartor4.bak  LinuxAdministartor6.bak  LinuxAdministartor8.bak
LinuxAdministartor1.bak   LinuxAdministartor3.bak  LinuxAdministartor5.bak  LinuxAdministartor7.bak  LinuxAdministartor9.bak
[root@lixin lixin]# ls | sed -nr 's#(^L.*[0-9])(.*$)#mv & \1.txt#gp'
mv LinuxAdministartor10.bak LinuxAdministartor10.txt
mv LinuxAdministartor1.bak LinuxAdministartor1.txt
mv LinuxAdministartor2.bak LinuxAdministartor2.txt
mv LinuxAdministartor3.bak LinuxAdministartor3.txt
mv LinuxAdministartor4.bak LinuxAdministartor4.txt
mv LinuxAdministartor5.bak LinuxAdministartor5.txt
mv LinuxAdministartor6.bak LinuxAdministartor6.txt
mv LinuxAdministartor7.bak LinuxAdministartor7.txt
mv LinuxAdministartor8.bak LinuxAdministartor8.txt
mv LinuxAdministartor9.bak LinuxAdministartor9.txt
[root@lixin lixin]# ls | sed -nr 's#(^L.*[0-9])(.*$)#mv & \1.txt#gp' | bash
[root@lixin lixin]# ls
LinuxAdministartor10.txt  LinuxAdministartor2.txt  LinuxAdministartor4.txt  LinuxAdministartor6.txt  LinuxAdministartor8.txt
LinuxAdministartor1.txt   LinuxAdministartor3.txt  LinuxAdministartor5.txt  LinuxAdministartor7.txt  LinuxAdministartor9.txt
[root@lixin lixin]#
# &符号表示前面两个分隔符中间的所有内容
#shell的命令都是bash在执行，所以把命令拼出来后直接交给bash处理即可
```
# 5 特定含义的符号
## 5.1 =
在sed中=号表示显示行号，但是会换行显示，只是显示行号可以省略“”（引号）。
```bash
[root@lixin daxin]# head -n 3 /etc/passwd | sed =
1
root:x:0:0:root:/root:/bin/bash
2
bin:x:1:1:bin:/bin:/sbin/nologin
3
daemon:x:2:2:daemon:/sbin:/sbin/nologin
[root@lixin daxin]#
```
## 5.2 l
在sed中l表示显示隐藏字符，只是显示隐藏符号可以省略“”（引号）
```bash
[root@lixin daxin]# head -3 /etc/passwd | sed -n l 
root:x:0:0:root:/root:/bin/bash$
bin:x:1:1:bin:/bin:/sbin/nologin$
daemon:x:2:2:daemon:/sbin:/sbin/nologin$
[root@lixin daxin]#
```
## 5.3 {}
在sed中，{}符号表示要执行的sed内置command的集合，之间用；（分号）隔开。
```bash
[root@lixin daxin]# sed -n '10,20{=;p}' /etc/passwd   
10
uucp:x:10:14:uucp:/var/spool/uucp:/sbin/nologin
11
operator:x:11:0:operator:/root:/sbin/nologin
12
games:x:12:100:games:/usr/games:/sbin/nologin
13
gopher:x:13:30:gopher:/var/gopher:/sbin/nologin
14
ftp:x:14:50:FTP User:/var/ftp:/sbin/nologin
15
nobody:x:99:99:Nobody:/:/sbin/nologin
16
dbus:x:81:81:System message bus:/:/sbin/nologin
17
vcsa:x:69:69:virtual console memory owner:/dev:/sbin/nologin
18
abrt:x:173:173::/etc/abrt:/sbin/nologin
19
haldaemon:x:68:68:HAL daemon:/:/sbin/nologin
20
ntp:x:38:38::/etc/ntp:/sbin/nologin
[root@lixin daxin]#  //{=;p}表示，先打印行号，在输出
```
## 5.4 G
在sed中G表示在一行后面追加一行空行。
```bash
[root@lixin daxin]# head -n 3 /etc/passwd | sed G
root:x:0:0:root:/root:/bin/bash

bin:x:1:1:bin:/bin:/sbin/nologin

daemon:x:2:2:daemon:/sbin:/sbin/nologin

[root@lixin daxin]#
```
