# 1 什么是shell
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;shell是一个命令解释器，位于操作系统的最外层，它负责和用户直接对话，不用户输入的内容解释给操作系统，操作系统处理完毕后，输出结果，输出到屏幕上。分为交互式的和非交互式的。输入命令，得到结果。就是交互式的，执行脚本就是非交互的。
__总结：shell就像是一座桥梁，连通用户和操作系统。__
# 2 什么是shell脚本
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;把一个或多个linux命令组合起来放在一个程序文件中执行，这样的程序叫做shell脚本。严格来说把命令.变量和流程控制语句等结合在一起，就是shell脚本
## 2.1 如何写一个思路缜密的shell脚本
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;要多去模拟操作，多去想想为什么
## 2.2 shell的特点
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;linux中所有的配置文件.日志文件都是纯文本文件。而shell的特点就是善于处理纯文本的内容。
## 2.3 shell的分类
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bourne shell(包括 sh,ksh,bash) C shell(csh,tcsh)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;查看系统支持的sh类型：/etc/shells,常用的就是/bin/sh,/bin/bash,/sbin/nologin  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux主流的shell就是bash，它是bourne again shell的缩写，是由bonrne shell发展而来，它还包含了csh和ksh的特色。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__目前：通用的bourne shell 已经被bash shell取代__
## 2.4 其他脚本语言
1. php：主要用于web页面开发，适合wab前端展示
2. perl：比shell强大，语法灵活，实现方式很多，不易读，其中mysql的HA就是由perl写的
3. python：近几年很火的软件，不但可以开发前端，也可以开发后端，属于全能型语言
## 2.5 常用系统的默认shell
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux：bash  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;solaris：bonrne sh  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;AIX：ksh  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;HP-unix：posix shell（sh）  
## 2.6 修改用户默认的shell
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以修改/etc/default/useradd文件的shell字段，更改新添加的用户。也可以直接修改/etc/passwd中的最后一个字段，直接修改某个用户的shell。
## 2.7 shell脚本的建立
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__通过vim编辑器编写shell脚本__ ,注意脚本的第一行 __#!/bin/bash__ 表示来指定解释器
>注意：python 程序开头 #!/usr/bin/env python因为Linux默认就是使用bash，所以使用bash的话，可以不用加，为了规范还是要添加。如果不指定解释器，就需要用bash test.sh来执行了。  
>注意：写脚本要写注释
## 2.8 shell脚本的执行
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当shell运行时，它会先查找系统环境变量ENV，该变量指定了环境文件（通常是.bashrc,.bash_profile,/etc/bashrc,/etc/profile等），然后从该环境变量文件开始执行脚本，当读取了ENV的文件后，shell会开始执行shell脚本中的内容。  
注意：设置cron任务时，最好把系统变量在脚本中重新定义一遍
__shell脚本的执行通常采用以下几种方式__
1. bash scripts-name 或 sh scripts-names （推荐）
2. patch/scripts-name 或 ./scripts-names （需要脚本有执行权限）
3. source scripts-name 或　. scripts-names （执行脚本的同时，使父shell继承子shell的变量)
4. sh<scripts-name 或　cat scripts-name|sh
>注意：我们用户登录的时候会默认分配一个shell，这个shell就是父shell，而在这个shell中执行的shell脚本，就属于子shell，父shell是不能直接继承子shell的变量的，但子shell可以直接继承父shell的变量，如果想要父shell获取子shell中的变量，就需要使用source或者.来执行脚本了。（相当于php中的include）
# 3 shell脚本开发基本规范
1. 脚本第一行指定解释器
```bash
#!/bin/bash或者#!/bin/sh
```
2. 脚本开头加版权信息
```bash
#Date:
#Author: create by lixin
#Mail：
#Function:功能
```
3. 脚本中不用中文注释
4. 脚本使用.sh作为扩展名
5. 写代码优秀习惯
    + 成对的符号尽量一次打出来，防止遗漏
    * [ a ]中括号两端要有空格，书写的时候就街上，[[ a ]]双中括号也是如此。
    * 流程控制语句一次书写完，在添加内容
# 4 shell变量基础及深入
## 4.1 什么是变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;比如x=1,y=x+1,那么y等于2。x和y就是变量名，x变量的值是1，而y变量的值是x+1，等号在这里叫做赋值。
简单的说，变量就是用一个固定的字符串（也可能是字符数字等的组合），代替更多更复杂的内容，这个内容里可能还会包含变量.路径.字符串等其他的内容。
__变量是暂时存数据的地方，这个存储的数据是存在于内存空间中的，通过调用变量的名字就可以取出变量对应的数据。__
## 4.2 shell的变量特性
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bash shell中默认的情况下是不会区分变量的内容的类型，例如：整数.字符串.小数等。这一点和其他强类型语言是有区别的。如果需要指定shell变量内容类型，可以使用declare显示指定定义变量的类型（没这个需求）。
### 4.2.1 变量的类型
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;变量可以分为：环境变量（全局变量）和普通变量（局部变量）。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;环境变量也成为全局变量，可以在创建他们的shell及其派生出来的任意子进程shell中使用，环境变量又可以分为自定义环境变量和bash内置的环境变量。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;普通变量也可成为局部变量，只能在创建他们的shll函数或shell脚本中使用。普通变量一般是由开发者在开发脚本程序时创建的。
### 4.2.2 环境变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;环境变量一般用于定义shell的运行环境，保证shell命令的正确执行，shell通过环境变量来确定登录用户名.命令路径.终端类型.登录目录的等，所有的环境变量都是系统全局变量，可用于所有子进程中，这包括编辑器.shell脚本和各类应用（cron任务比较特殊需要注意）。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;环境变量可以在命令行中设置创建，但用户退出命令是这些变量值就会丢失，因此，如果希望永久保存环境变量，可以在用户家目录下.bash_profile或.bashrc文件或全部配置文件/etc/profile或/etc/bashrc文件或者/etc/profile.d/下定义，因为每次用户登录时这些变量的值都会被初始化一次。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;所有环境变量名字格式均为大写，环境变量用于用户进程程序前，都应该用export命令导出定义，例如：export KEY=1.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;环境变量可以在创建他们的shell和从该shell派生的任意子shell或进程中，他们通常被称为全局变量以区别局部变量。通常，环境变量应该大写。环境变量是已经用export内置命令导出的变量。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有一些环境变量，比如HMOE.PATH.SHELL.UID.USER等，在用户登录之前就已经被/bin/login程序设置好了，通常环境变量定义并保存在用户家目录下的.bash_profile中。
#### 4.2.2.1 定义全局变量格式
1. export 变量名=value；
2. 变量名=value；export 变量名
3. declare -x 变量名=value  
__注意：如果是java环境，最后再脚本里面再定义一遍java环境变量。__
#### 4.2.2.2 显示与取消环境变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过echo或printf命令打印环境变量
例：
```bash  
打印用户登录的目录: echo $HOME
打印用户的UID: printf $UID
使用env和set命令查看所有的环境变量
通过unset命令取消环境变量: unset NAME
```
#### 4.2.2.3 环境变量小结
1. 变量名通常要大写
2. 可以在自身的shell以及子shell中使用
3. 通过export来定义
4. 查看使用echo $NAME
5. 删除使用unset NAME
6. 永久生效写入到用户环境变量文件，或者全局环境变量文件中。
7. 写定时任务的时候，注意在脚本中添加环境变量（java）。
#### 4.2.3 普通变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本地变量再用户当前的shell生存期的脚本中使用，例如，本地变量key的取值为123，这个值只在用户当前shell生存期中有意义。如果在shell中启动另一个进程或退出，本地变量key值将无效。
#### 4.2.3.1 定义普通变量
定义普通变量的方法：
1. 变量名=value
2. 变量名='value'
3. 变量名="value"  
>其中，变量名一般是有字符，数字，下划线组成。可以字符或下划线开头，但是不能以数字开头。  
变量的内容，可以用单引号或双引号引起来，或不加引号。三者的区别是；不加引号，不能输入不连续的值，只要变量连续，可以解释其中引用的其他变量。单引号和echo中的单引号相同，就是内部有变量则不会解释变量，双引号即如果定义的值中引用其他变量，则解释该变量，变量的值可以是不连续的。而反引号，一般用来解析命令，相当于$().
```bash
例子：
 a=192.168.1.2
 b='192.168.1.2-$a'
 c="192.168.1.2-$a"
 echo $a
 192.168.1.2
echo $b
       192.168.1.2-$a
echo $c
 192.168.1.2-192.168.1.2

a=192.168.1.2-$a
b='192.168.1.2-$a'
c="192.168.1.2-$a"
echo $a
       192.168.1.2-192.168.1.2
echo $b
       192.168.1.2-$a
echo $c
       192.168.1.2-192.168.1.2-192.168.1.2
注意：日常定义变量如果是纯数字的话可以不加引号，其他情况建议都使用双引号。
扩展：把命令的结果作为变量
变量名=`ls`或者 变量名=$(ls)
```
#### 4.2.3.2 普通变量小结
1. 连续的数字字符串可以不加引号
2. 变量内容是命令的结果，定义时可以使用反引号或$().
3. 若变量和其他字符连接的时候，就必须给变量加上大括号{}。${key}_keyname;
#### 4.2.3.3 变量的高级应用
定义变量后可以直接在各个地方调用
```bash
key=init
grep $key /etc/iniitab
grep "$key" /etc/inittab
# 上面两个会过滤出/etc/inittab中包含init字符串的行
grep '$key' /etc/inittab
# 单引号，则会过滤出/etc/inittab中包含$key的行,同样适用于sed。
```
__注意：awk中不同于grep和sed中__  
awk中的特殊用法1：
```awk
key=init
awk 'BEGIN{print "$key"}'
# awk中""是不解释变量的，和常规的相反，只有使用单引号才会解析。
awk 'BEGIN{print '$key'}'
# 不加引号，则不会输出
```
awk中的特殊用法2：
```awk
key='int'
awk 'BEGIN{print '$key'}'
# 这时也是不会输出内容的，需要在单引号外面再加一层双引号才可以
awk 'BEGIN{print "'$key'"}'
```
### 4.2.4 shell的特殊重要变量
#### 4.2.4.1 位置变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__$0__ 获取当前执行的shell脚本，如果执行脚本带路径那么就包括脚本路径。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__$n__ 获取当前执行的shell脚本的第n个参数的值，n=1..9，当n为0时表示脚本的文件名，如果n大于9，用大括号括起来${10}.例子：
```bash
# 创建测试文件：
[root@db01 ~]# echo -n 'echo ' > test1.sh && echo \${1..15} >> test.sh
[root@db01 ~]# cat test.sh
echo $1 $2 $3 $4 $5 $6 $7 $8 $9 $10 $11 $12 $13 $14 $15 $16
[root@db01 ~]# bash test.sh {a..z}
a b c d e f g h i a0 a1 a2 a3 a4 a5 a6
# 由于变量超过了9个，所以从第10个变量开始要加大括号，不然shell认为$10，是$1的值和0组合在一起，所以才会显示a0
[root@db01 ~]# cat test.sh
echo $1 $2 $3 $4 $5 $6 $7 $8 $9 ${10} ${11} ${12} ${13} ${14} ${15} ${16}
[root@db01 ~]# bash test.sh {a..z}
a b c d e f g h i j k l m n o p
# $# 表示脚本传参的个数，脚本名称后面的参数的个数.
# $* 将命令行脚本所有参数视为单个字符串等同于"$1$2$3",$*要用双引号。
# $@ 将命令行脚本每个参数视为单独的字符串，等同于"$1","$2","$3",这是将参数传递给其他程序的最佳方式，因为他会保留所有内嵌在每个参数里的任何空白。
# 注意：上述区别仅限于添加双引号的时候，即"$*"和$"@"
例：
[root@db01 scripts]# set -- a b c
[root@db01 scripts]# for i in "$@" ;do echo $i; done;
a
b
c
[root@db01 scripts]# for i in "$*" ;do echo $i; done;
a b c
[root@db01 scripts]#
```
#### 4.2.4.2 进程状态变量
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$? 查看上一个命令的执行结果。0表示成功，非0表示不成功。  
>错误代码返回值参考：  
2：权限不够  
1-125:表示运行失败（脚本命令.系统命令错误或参数传递错误）  
126：找到该命令了，但无法执行  
127：命令找不到  
128:被强制中断了    
>注意：在shell脚本中一般用exit 数字，来把返回值传递给$?
如果是函数，则通过return 数字，返回返回值给$?  
扩展：通过set命令进传参  
http://httpd.blog.51cto.com/2561410/1175971
```bash
set -- 'i am' handsome boy 传递3个参数到shell
set -- 1 2 3
[root@db01 ~]# echo $1 $2 $3
1 2 3
# $$ 显示当前进程的进程号
# $! 上一个进程的id号
cat pid.sh
echo $$ >/tmp/a.pid
sleep 300

sh pid.sh &
cat /tmp/a.pid
```
#### 4.2.4.3 bash内置命令
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;bash的内置命令，这些命令内置在bash中，没有绝对路径，需要时可以直接在shell脚本中调用 __alias, bg, bind, break, builtin, caller, cd,command, compgen, complete, compopt, continue, declare, dirs,disown,  echo,  enable,  eval, exec, exit, export, false, fc,fg, getopts, hash, help, history,  jobs,  kill,  let,  local,logout,  mapfile,  popd,  printf, pushd, pwd, read, readonly,return, set, shift,  shopt,  source,  suspend,  test,  times,trap,  true,  type,  typeset,  ulimit, umask, unalias, unset,wait__
### 4.2.4.4 变量子串
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过man bash找到Parameter Expansion来查看子串的全部用法
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;${#变量名}取变量的长度
```bash
[root@db01 ~]# httpd="I am httpd"
[root@db01 ~]# echo $httpd
I am httpd
[root@db01 ~]# echo $httpd|wc -L
11
[root@db01 ~]# echo ${#httpd}
11
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过使用expr命令计算变量的长度
```bash
[root@db01 ~]# expr length "$httpd"
11

# 计算I am good boy,welcome to myhome中字母小于5个的并打印
for i in I am good boy,welcome to myhome
do
[ ${#i} -lt 5 ] &&{
  echo "$i"
}
done
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;${变量：2}截取变量的第几个字符到最后
```bash
[root@db01 ~]# httpd="I am httpd"
[root@db01 ~]# echo ${httpd:2}
am httpd
[root@db01 ~]# echo ${httpd:3}
m httpd
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;${#变量:n:m}从第n个字符开始截取m个
```bash
[root@db01 ~]# echo ${httpd:2:2}
am
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;${变量#字符串}从开头开始匹配，删除最短匹配字符串的值
```bash
[root@db01 scripts]# echo $LIXIN
123123123123123123123123456
[root@db01 scripts]# echo ${LIXIN#1*3}
123123123123123123123456
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;${变量##字符串}从开头开始匹配，删除最长匹配字符串的值
```bash
[root@db01 scripts]# echo ${LIXIN##1*3}
456
[root@db01 scripts]# echo $LIXIN
i am good boy boy
[root@db01 scripts]# echo ${LIXIN%b*y}
i am good boy
[root@db01 scripts]# echo ${LIXIN%%b*y}
i am good
[root@db01 scripts]#
```
小结：
1. #是开头删除匹配最短
2. ##是开头删除匹配最长
3. %是结尾删除匹配最短
4. %%是结尾删除匹配最长  
***
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;${变量/字符串1/字符串2}，从开头开始，把变量中的字符串1替换为字符串2.
```bash
[root@db01 scripts]# LIXIN="i am good boy boy"
[root@db01 scripts]# echo $LIXIN
i am good boy boy
[root@db01 scripts]# echo ${LIXIN/boy/girl}
i am good girl boy
[root@db01 scripts]# echo ${LIXIN//boy/girl}
i am good girl girl
两个斜线表示替换所有符合的字符串
[root@db01 scripts]# echo ${LIXIN/#boy/girl}
i am good boy boy
#表示以boy开头的，替换为girl，由于变量LIXIN没有以boy开头，所以不会替换
[root@db01 scripts]# echo ${LIXIN/%boy/girl}
i am good boy girl
[root@db01 scripts]#
```
#### 4.2.4.5 变量替换
__${values:-word}__ 如果变量名称存在并且非null，则返回变量的值。否则返回word字符串，表示如果变量未定义，则返回默认值。
```bash
[root@db01 scripts]# echo ${lixin:-test}
test
[root@db01 scripts]# lixin=123
[root@db01 scripts]# echo ${lixin:-test}
123
[root@db01 scripts]# echo $lixin
空
```
__${value:=word}__ 如果变量名存在且非null，则返回变量值。否则，设置这个变量为word，并返回其值
```bash
[root@db01 scripts]# echo ${lixin:=test}
test
[root@db01 scripts]# echo $lixin
test
[root@db01 scripts]#
```
>注意：-和=的区别是，-表示变量为空的时候返回的默认值，这个默认值并不会被赋予变量，所以echo变量的时候依旧为空，=表示如果变量为空，则把word赋予该变量。  
>扩展：这个冒号：也可以不加，功能和加上是相同的
```bash
[root@db01 scripts]# unset LIXIN
[root@db01 scripts]# echo ${LIXIN-test}
test
[root@db01 scripts]# LIXIN=1
[root@db01 scripts]# echo ${LIXIN-test}
1
[root@db01 scripts]#
[root@db01 scripts]# unset LIXIN
[root@db01 scripts]# echo ${LIXIN=test}
test
[root@db01 scripts]# echo $LIXIN
test
[root@db01 scripts]#
```
#### 4.2.4.6 变量的数值(整数)计算
变量的数值计算常见的有如下几个命令：__(()),let,expr,bc(小数运算)，${}，其他都是整数。__  
常用运算符号：
* ++，--：增加或减少，可以前置也可以放在末尾
* +,-,*,/,%：分别对应加.减.乘，除，求余
* <,<=,>,>=：分别对应小于，小于等于，大于，大于等于
* ==，!=：等于，不等于
* &&：逻辑和，即command1 && comand2 ，command1成功则执行command2。
* ||：逻辑或，即command1 || comand2，command1不成功则执行command2。
* =，+=，-=，*=，/=，%=，赋值运算符，a+=1,表示a=a+1,a*=1,表示a=a*1。

__1.(())用法__  
如果要执行简单的整数就算，只需要将特定的算数表达式用$(())括起来即可。
```bash
[root@db01 scripts]# echo $((1+2))
3
[root@db01 scripts]# echo $(1+2)
-bash: 1+2: command not found

[root@db01 scripts]#
[root@db01 scripts]# ((a=1+2**3-4%3))
[root@db01 scripts]# echo $a
8
[root@db01 scripts]#
+=例子：
[root@db01 scripts]# echo $a
8
[root@db01 scripts]# echo $((a+=1))
9
[root@db01 scripts]# echo $a
9
[root@db01 scripts]# echo $((a-=1))
8
[root@db01 scripts]# echo $a
8
[root@db01 scripts]#
n++例子：
[root@db01 scripts]# echo $a
8
[root@db01 scripts]# echo $((a++))
8
[root@db01 scripts]# echo $a
9
[root@db01 scripts]# echo $((a--))
9
[root@db01 scripts]# echo $a
8
[root@db01 scripts]#
++n例子：
[root@db01 scripts]# echo $a
9
[root@db01 scripts]# echo $((a--))
9
[root@db01 scripts]# echo $a
8
[root@db01 scripts]# echo $((--a))
7
[root@db01 scripts]# echo $a
7
[root@db01 scripts]# echo $((++a))
8
[root@db01 scripts]# echo $a
8
[root@db01 scripts]#
```
小结: n++和++n的区别是，变量n在前，则表达式的值为n，然后n自增或自减，变量n在后，则表达式的值为自增自减后的n

__2.let命令（用于整数计算）__  
格式：let 赋值表达式
```bash
[root@db01 scripts]# i=2
[root@db01 scripts]# i=i+8
[root@db01 scripts]# echo $i
i+8
# 因为定义了两次i，所以第二次会覆盖之前对于i的定义
[root@db01 scripts]# i=2
[root@db01 scripts]# let i=i+8
[root@db01 scripts]# echo $i
10
[root@db01 scripts]#
# let的效果和(())相同，但效率不如(())高，所以建议用双括号
```
__3.expr__  
+ __计算(整数数字，并且要有空格)__
```bash
[root@db01 scripts]# expr 1 + 2
3
[root@db01 scripts]# expr 1+2
1+2
[root@db01 scripts]#
# 注意：除号.乘号需要用反斜线转意
[root@db01 scripts]# expr 2 \* 2
4
[root@db01 scripts]#
```
* __判断是否为整数__  
由于expr不能计算非整数的表达式，可以通过expr的返回值确定传递的变量是否为整数。
```bash
[root@db01 scripts]# expr 1 + a
expr: non-numeric argument
[root@db01 scripts]# echo $?
2
[root@db01 scripts]# expr 1 + 1
2
[root@db01 scripts]# echo $?
0
[root@db01 scripts]# cat ceshi.sh
#!/bin/bash
a=$1
expr $a + 1 &>/dev/null
[ $? -eq 0 ]||{
 echo "please input number"
}
# 或者
[root@db01 scripts]# cat ceshi1.sh
#!/bin/bash
a=$1
expr $a + 1 &>/dev/null
[ $? -ne 0 ] &&{
 echo "please input number"
}
```
__计算字符串的长度__
```bash
[root@db01 scripts]# expr length $LIXIN
4
[root@db01 scripts]# echo $LIXIN
test
[root@db01 scripts]#
其他计算变量长度的方法：
[root@db01 scripts]# echo ${#LIXIN}
4
[root@db01 scripts]# echo $LIXIN|wc -L
4
[root@db01 scripts]# echo $LIXIN | awk "{print length $0}"
4
```
__expr判断后缀名__   
通过expr对变量进行模式匹配,格式为:expr value : 关键字
```bash
[root@db01 scripts]# LIXIN=test.pub
[root@db01 scripts]# expr $LIXIN : ".*\.pub"
8
[root@db01 scripts]# LIXIN=test.puv
[root@db01 scripts]# expr $LIXIN : ".*\.pub"
0
# 匹配为0，不匹配则非0
```
__time命令__ 计算命令的运行时间（真实时间.内核时间.用户时间）
```bash
[root@db01 scripts]# time sh bianliang1.sh 1

real 0m0.003s
user       0m0.001s
sys  0m0.000s
[root@db01 scripts]#
# 该命令可以对相同结果的命令进行压力测试，看哪个执行速度更快
```
小结：内置功能速度更快（${#value}计算速度最快）
__bc命令__  
linux内置命令，同时也是一个计算器，直接输入bc，会进入计算器界面。并且bc支持小数(awk也支持小数)
```bash
# 命令行：
[root@db01 scripts]# echo 1+2|bc
3
[root@db01 scripts]# echo 2*2|bc
4
[root@db01 scripts]#
# 转换成二进制
[root@db01 scripts]# echo "obase=2;255"|bc
11111111
[root@db01 scripts]#
# 确定小数点后数字的个数
[root@db01 scripts]# echo "scale=2;10/3"|bc
3.33
[root@db01 scripts]# echo "scale=3;10/3"|bc
3.333
[root@db01 scripts]#
# awk方法计算小数
[root@db01 scripts]# echo "1.5 2.3"|awk '{print ($1*$2)}'
3.45
# 例题:
# 计算1+2+3+4+5+6+7+8+9+10的值，并且列出1+2+3+4+5+6+7+8+9+10=55的结果
[root@db01 scripts]# echo `seq -s '+' 10`=`seq -s '+' 10|bc`
1+2+3+4+5+6+7+8+9+10=55
[root@db01 scripts]# a=`seq -s "+" 10` && echo $a=`echo $a|bc`
1+2+3+4+5+6+7+8+9+10=55
[root@db01 scripts]#
```
小结：常用的shell的数值运算方法为(())和let
### 4.2.3 变量的输入  
shell变量除了可以直接复制或脚本传参外，还可以使用read命令从标准输入获得，read为bash内置命令，可以通过help read查看帮助  
__格式：read [参数] [变量名]__  
常用参数: -p,设置提示时间。-t，设置等待时间，默认单位为秒
```bash
[root@db01 ~]# read -t 10 -p "please input two number"
please input two number1 2
[root@db01 ~]# read -t 10 -p "please input two number" a b
please input two number1 2
[root@db01 ~]# echo $a $b
1 2
[root@db01 ~]#
```
在脚本中通过read命令进行传参
```bash
[root@db01 scripts]# vim read.sh
#!/bin/bash
read -t 10 -p "please input two number" a b
echo "$a+$b=$(($a+$b))"
echo "$a-$b=$(($a-$b))"
echo "$a/$b=$(($a/$b))"
echo "$a*$b=$(($a*$b))"
扩展：read的提示性信息，可以通过
echo -n 'please input two number'
read a b
来获取
```
扩展题目：使用read读入2个变量，计算他们的加减乘除结果，并且判断是否为2个变量，以及是否为整数，当第二个数为被除数的时候不能为0
```bash
[root@db01 scripts]# vim read.sh

#!/bin/bash
read -t 10 -p "please input two number" a b
[ -z $a  ]&&{
  echo "the First number is null,please input number"
  exit 1
}  -->可以通过${#value}，如果返回值为0，表示变量为空
[ -z $b  ]&&{
  echo "the Seconed number is null,please input number"
  exit 2
}
expr $a + $b + 1 &>/dev/null
result=$?
[ $result -ne 0 ] &&{
  echo "please input two number"
  exit 3
}
echo "$a+$b=$(($a+$b))"
echo "$a-$b=$(($a-$b))"
echo "$a*$b=$(($a*$b))"
[ $b -eq  0 ] && {
 echo "The Seconed is "$b",can not calc /"
 exit 4
}
echo "$a/$b=$(($a/$b))"
```
## 4.3 条件测试与比较
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在bash的各种流程控制结果中，通常要进行各种测试，然后根据测试结果珍惜ing不同的操作，有时也会通过与if等条件语句相结合，更方便的完成判断。
### 4.3.1 条件测试表达式
条件测试有三种语法形式：  
1. __test <测试表达式>__  
2. __[<测试表达式>]__  
3. __[[<测试表达式>]]__
>说明：  
>1.其中1和2是等价的，只是写法不同。  
>2.在[[]]中可以使用通配符进行模式匹配。&&.||.<.>等操作符可以直接应用于[[]]中，但不能应用于[]中。  
>3.对于整数的关系运算，也可以使用shell的算数运算符(())。
#### 4.3.1.1 test <测试表达式>
格式为：test [option] 目标
```bash
[root@db01 scripts]# test -f /etc/hosts && echo 0 || echo 1
0
[root@db01 scripts]# test -x /etc/hosts && echo 0 || echo 1
1
[root@db01 scripts]# test -x /server/scripts/pid.sh && echo 0 || echo 1
0
[root@db01 scripts]# test -d /etc/hosts && echo 0 || echo 1
1
[root@db01 scripts]# test -d /etc && echo 0 || echo 1
0
[root@db01 scripts]#
```
#### 4.3.1.2 [<表达式>]
__格式为：[ 选项 目标 ]__  
多个条件使用-a（并且），-o（或），来链接
```bash
[root@db01 scripts]# [ -f /etc/hosts ] && echo 0||echo 1
0
[root@db01 scripts]# [ -f /etc/hosts -a -f /etc ] && echo 0 ||echo 1
1
# 由于etc不是文件，所以表达式为假
[root@db01 scripts]# [ -d /etc/host ] && echo 0||echo 1
1
[root@db01 scripts]# [ -d /etc/hosts -a -d /etc ] && echo 0 ||echo 1
1
# 由于hosts不是目录，所以表达式为假
[root@db01 scripts]# [ -x /etc/host ] && echo 0||echo 1
1
[root@db01 scripts]#
```
#### 4.3.1.3 [[<表达式>]]
__格式为：[[ 选项 目标 ]]__  
多个条件使用&&（并且），||（或），来链接
```bash
[root@db01 scripts]# [[ -f /etc/hosts ]] && echo 0 ||echo 1
0
[root@db01 scripts]# [[ -f /etc/hosts -a /etc/services ]] && echo 0 ||echo 1
-bash: syntax error in conditional expression
-bash: syntax error near '-a'
[root@db01 scripts]# [[ -f /etc/hosts && /etc/services ]] && echo 0 ||echo 1
0
[root@db01 scripts]#
# [[]]与[]不同的是表示逻辑关系的时候，&&（-a），||（-o）。
```
### 4.3.2 文件测试表达式
部分选项如下：
* -f FILE:当测试文件存在，并且是一个普通文件，表达式的值为真
* -d FILE:当测试文件是目录，表达式为真
* -s FILE:当测试文件存在，并且非空，表达式的值为真
* -e FILE:当测试文件存在(只要存在)，表达式为真
* F1 -nt F2: F1比F2新
* F1 -ot F2：F1比F2旧
* -x FILE:当测试文件对当前用户来说可执行，表达式为真
* -a FILE:当测试文件存在，表达式为真
* -b FILE:当测试文件为块设备，表达式为真
* -c FILE:当测试文件为字符设备，表达式为真
* -g FILE:当测试文件拥有set GID，表达式为真
* -h FILE:当测试文件为硬链接文件，表达式为真
* -L FILE:当测试文件为软链接文件，表达式为真
* -k FILE:当测试文件拥有sticky biy，表达式为真
* -p FILE:当测试文件为管道文件，表达式为真
* -r FILE:当测试文件对当前用户来说可读，表达式为真
* -S FILE:当测试文件为socket文件，表达式的值为真
* -u FILE:当测试文件拥有set UID，表达式为真
* -w FILE:当测试文件对当前用户来说可写，表达式为真　　　　

__注：在选项前加！表示取反__  
```bash
# 例：高效的判断方法
[root@db01 scripts]# [ -f /etc/host ] || echo 123
123
[root@db01 scripts]# cat tiaojiao.sh
#!/bin/bash
[ -f $1 ] || {
  echo 1
  echo 2
  echo 3
}
# 判断$1是否是文件，不是得话输出1，2，3
[root@db01 scripts]# sh tiaojian.sh /etc/host
1
2
3
```
### 4.3.3 字符串测试表达式   
* -z "string":当字符串为空，表达式为真
* -n "string":当字符串不为空，表达式为真
* "string 1" = "string 2":当字符串1和字符串2相等，则表达式为真，=号可以用==代替
* "string 1" != "string 2":当字符串1不等于字符串2，表达式为真  

注意：
   1. 字符串用双引号"",引起来
   2. 等号两边要有空格
### 4.3.4 整数测试表达式
__在[]和test中 在[[]]和(())中 含义__
+ -eq =或== 等于
+ -ne != 不等于
+ -gt > 大于
+ -lt < 小于
+ -ge >= 大于等于
+ -le <= 小于等于
```bash
# 例：生产中一般使用[]和eq这样的搭配
[root@db01 scripts]# echo $LIXIN
2
[root@db01 scripts]# [ $LIXIN -ne 0 ] && echo 0 ||echo 1
0
[root@db01 scripts]# [ $LIXIN -eq 2 ] && echo 0 ||echo 1
0
[root@db01 scripts]# [ $LIXIN -eq 3 ] && echo 0 ||echo 1
1
[root@db01 scripts]#
```
### 4.3.5 逻辑操作符
__在[]和test中 在(())和[[]]中 含义__
+ -a && 并且
+ -o || 或
+ ！非

### 4.3.6 shell实例
#### 4.3.6.1 比大小
传参的方式比大小：
```bash
#!/bin/bash
[ $# -ne 2 ]&&{
  echo "USAGE:$0 avg1 avg2"
  exit 1
}
expr $1 + $2 + 1 &>/dev/null
[ $? -ne  0 ]&&{
  echo "please input num"
  exit 2
}
[ $1 -eq $2 ]&&{
  echo "$1=$2"
  exit 3
}
[ $1 -gt $2 ]&&{
  echo "$1>$2"
}||{
  echo "$1<$2"
}
```

read方式比大小：
```
#!/bin/bash
read -t 10 -p "please input num1" a
read -t 10 -p "please input num2" b
[ -z $a ]&&{
  echo "error,please input number1"
  exit 1
}
[ -z $b ]&&{
  echo "error,pleasr input number2"
  exit 2
}
expr $a + $b + 1 &>/dev/null
[ $? -ne  0 ]&&{
  echo "please input num"
  exit 2
}
[ $a -eq $b  ]&&{
  echo "$a=$b"
  exit 1
}
[ $a -gt $b ]&&{
  echo "$a>$b"
}||{
  echo "$a<$b"
}
```
__扩展：针对判断整数的问题，可以有如下思路：__
```bash
[root@db01 scripts]# echo $LIXIN
123a
[root@db01 scripts]# echo $LIXIN | sed s#[^0-9]##
123
[root@db01 scripts]# [ "`echo $LIXIN | sed s#[^0-9]##`" = "$LIXIN" ] && echo 1 || echo 0
0
[root@db01 scripts]# LIXIN=123
[root@db01 scripts]# [ "`echo $LIXIN | sed s#[^0-9]##`" = "$LIXIN" ] && echo 1 || echo 0
1
[root@db01 scripts]#
# 把输入的字符串中的非数字替换为空，如果和源字符串相等，则表示输入的是数字。
```
#### 4.5.6.2 打印菜单
要求：
1. 当用户输入1时，输出'start installing lamp'，然后执行/server/scripts/lamp.sh，脚本内容输入'lamp is installed'后退出脚本；
2. 当用户输入2时，输出'start installing lnmp'，然后执行/server/scripts/lnmp.sh，脚本内容输入'lnmp is installed'后退出脚本；
3. 当用户输入3时，退出当前菜单及脚本；
4. 当输入任何其他字符，给出提示'input error'后退出脚本
5. 要对执行的脚本进行相关条件判断，例如：脚本是否存在，是否可以执行等。
```bash
[root@db01 ~]# cat menu.sh
#!/bin/bash
man(){
cat <<EOF
    1.install lamp
    2.install lnmp
    3.exit
    please input number
    EOF
}
man
read num
if [ $num -eq 1 ];then
    echo "installing lamp......."
elif [ $num -eq 2 ];then
    echo "installing lnmp......."
elif [ $num -eq 3 ];then
    echo "good bye"
    exit 1
else
    echo "input error"
    exit 2
fi
[root@db01 ~]# 
```
## 4.4 分支与循环结构
### 4.4.1 if条件语句
__1. 单分支结构：__
```bash
if [ 条件 ]
    then
       指令
fi
# 或
if [ 条件 ];then
    指令
fi
```
__2. 双分支结构：__
```bash
if [ 条件 ]
    then
       指令1
else
    指令2
fi
# 类似：[ -f /etc/hosts ] && echo 0 || echo 1
```
__3. 多分支结构:__
```bash
if [ 条件1 ]
    then
       指令1
elif [ 条件2 ]
    then 指令2
# .......可以有多个elif
else
  指令3
fi
```
例子：开发shell脚本判断系统剩余内存的大小，如果低于100M，就邮件报警给管理员，并且加入系统定时任务每3分钟执行一次检查。
```bash
[root@db01 ~]# tail -1 /etc/mail.rc
set from=beyondlee2011@126.com smtp=smtp.126.com smtp-auth-user=beyondlee2011 smtp-auth-password=aini3845 smtp-auth=login
[root@db01 scripts]# cat memtest.sh
#!/bin/bash
mem=`free -m | awk 'NR==3{print $NF}'`
[ $mem -le 100 ] && {
   echo 'mail is sending to Administrator.......'
   echo "memory is $mem" | mail -s "Warning:memory is not enough" 287990400@qq.com
}
//定时任务
[root@db01 scripts]# crontab -l
*/3 * * * * /bin/bash /server/scripts/memtest.sh &>/dev/null
```
例子2：用if双分支实现对mysql服务是否正常进行判断，使用端口，进程数或者URL的方式拍段，如果进程没起动，把进行启动
```bash
[root@db01 scripts]# cat testmysql.sh
#!/bin/bash
mysqlinfor=`netstat -lntup | grep 3306 | grep -v grep | wc -l`
if [ $mysqlinfor -ge 1 ] ;then
  echo "mysql is started"
  else
    echo "mysql is starting"
    /data/3306/mysql start &>/dev/null
    sleep 3
    echo "ok"
fi
[root@db01 scripts]#
```
例子3：以read读入的方式比较两个数的带下，用if多分支来实现。
```bash
[root@db01 scripts]# cat testnum.sh
#!/bin/bash
read -t 10 -p "please input two number " a b
expr $a + $b + 1 &>/dev/null
[ $? -ne 0 ] && {
  echo "please input number ok?"
  exit 1
}
[ -z "$a" ] && {
  echo "The First number is empty"
  exit 2
}
[ -z "$b" ] && {
  echo "The Seconed number is empty"
  exit 3
}
if [ "$a" -gt "$b" ]
  then
    echo "$a>$b"
  elif [ $a -lt "$b" ]
   then
     echo "$a<$b"
  else
   echo "$a=$b"
fi
[root@db01 scripts]#
```
>扩展：  
>判断mysql或者web服务共同方法：  
>1. 端口：  
    本地：netstat/ss/lsof  
    远程：telnet/nmap/nc命令  
>2. 进程：  
    本地：ps aux(ps -ef)  
>3. wget/curl(http方式，判断根据返回值或者返回内容)  
>4. header(http)，（根据状态码判断）  
>5. 数据库特有，通过mysql客户端查看返回值

通过端口和进程判断：
```bash
if [ "`netstat -lnt|grep 3306|awk -F "[ :]+" '{print $5}'`" = "3306" ] 由于后面使用的是比较，所以如果使用-eq 进行比较的话，当mysql没有运行的时候，第一个变量就为空,所以[ "" -eq "3306" ]会报语法错误。所以使用=号，用字符串进行比较。
if [ `ps -ef | grep mysql | grep -v grep |wc -l` -gt 0 ] 用这种方式比较的话，注意脚本的名称不要包含mysql
if [ `nc -w 2 10.0.0.7 3306 &>/dev/null && echo ok | grep ok | wc -l` -gt 0 ]
if [ `nmap 10.0.0.7 -p 3306 2>/dev/null | grep open | wc -l ` -gt 0 ]
if [ `netstat -lntup | grep mysqld | wc -l ` -gt 0 ]
if [ `lsof -i tcp:3306 | wc -l`  -gt 0 ]
```
通过curl进行判断
```bash
if [ "`curl -I -s -o /dev/null -w "%{http_code}\n" http://web`" = '200' ] -I：取返回头。-s：静默模式。-o：输出信息到空。-w：只取返回值。
if [ `curl -I http://web 2>/dev/null | head -1 | egrep "200|302|301" | wc -l ` -eq 1 ] 默认情况下curl命令会带一些错误的头部信息，这个时候可以使用静默模式，或者使用2>把错误信息输出到空
```
范例：实现通过传参的方式往/etc/user.conf里添加用户，具体要求如下：
1. 脚本用法
USAGE：sh adduser {-add | -del | -search} username
2. 传参需求
-add 表示添加，-del 表示删除， -search 表示查找
3. 如果有同名的用户则不能添加，没有对应用户则无需删除，查找到用户以及没有用户时给出明确提示。
4. /etc/user.conf不能被所有外部用户直接删除及修改
```bash
[root@db01 scripts]# cat adduser.sh
#!/bin/bash
[ $UID -ne 0 ] && {
    echo "please use root"
    exit 0
}
[ $# -ne 2 ] && {
    echo "USAGE:sh /server/scripts/adduser { -add|-del|-search} username"
    exit 1
}
cmd=$1
user=$2
user_info=`grep $user /etc/user.conf | wc -l`
if [ "$user_info" = "0" -a "$cmd" = "-add" ]
    then
        echo "$user" >> /etc/user.conf
        echo "adduser $user successful!"
        exit 2
elif [ "$user_info" = "1" -a "$cmd" = "-add" ]
    then
        echo "$user is cunzai"
        exit 3
elif [ "$user_info" = "0" -a "$cmd" = "-del" ]
    then
        echo "$user is not create,please create it first"
        exit 4
elif [ "$user_info" = "1" -a  "$cmd" = "-del" ]
    then
        sed -i "/$user/d" /etc/user.conf
        echo "del user $user successsful!!!"
        exit 5
elif [ "$cmd" = "-search" -a "$user_info" = "0" ]
    then
        echo "$user is not create,please create it first"
        exit 6
elif [ "$cmd" = "-search" -a "$user_info" = "1" ]
    then
        echo "$user is a good boy!"
        exit 7
else
    echo "USAGE:sh /server/scripts/adduser { -add|-del|-search} username"
    exit 8
fi
```
面试及实战考试题：监控web站点目录(/var/html/www)下所有文件是否篡改（文件内容被改了），如果有就打印改动的文件名（发邮件），定时任务每3分钟执行一次。
```bash
[root@db01 scripts]# cat testweb.sh
#!/bin/bash
dir=/var/www/html
result=/tmp/web.log
ip=`ip addr ls eth0 | awk -F "[ /]+" 'NR==3{print $3}'`
error_file=/tmp/error_file.log
[ -f $result ] || {
    find $dir -type f -name "*.html" | xargs md5sum >$result
}

if [ ! "`md5sum -c $result 2>/dev/null | grep FAILED | tee $error_file | wc -l`" = "0" ]
    then
        echo "web is not safe"
        mail -s "$ip is not safe" daxin.li@foxmail.com < $error_file
        mv $result /root/$(date +%F)_web_error.log
fi
# 保存退出
[root@db01 scripts]#  crontab -l
*/3 * * * * /bin/bash /server/scripts/testweb.sh &>/dev/null
```
### 4.4.2 case结构条件句
case语句实际上就是规范的多分支if语句  
__基本语法：__
```bash
case "字符串变量" in
    值1) 指令1..
;;
    值2) 指令2..
;;
    值*) 指令3..
esac
```
例子：根据用户驶入判断是哪个数字，如果用户输入1或2或3，则输出对应输入的数字，如果是其他内容，返回不正确，退出
```bash
[root@db01 scripts]# cat case1.sh
#!/bin/bash
a=$1
case "$a" in
    1) echo "$a"
;;
    2) echo "$a"
;;
    3) echo "$a"
;;
    *) echo "input error"
esac
```
例子：执行脚本打印一个水果菜单如下：
1. apple
2. pear
3. banana
4. cherry  

当用户选择水果的时候，打印告诉它选择的水果是什么，并给水果单词加上一种颜色。要求用case语句实现。
```bash
[root@db01 scripts]# cat case2.sh
#!/bin/bash
RED="\033[31m"
GREEN="\033[32m"
YELLO="\033[33m"
BLUE="\033[34m"
PINK="\033[35m"
END="\033[0m"

cat << EOF
    1.apple
    2.pear
    3.banana
    4.cherry
    please input your choice
EOF

read -t 10 Fruit
case $Fruit in
    1) echo -e "$RED apple $END"
;;
    2) echo -e "$GREEN pear $END"
;;
    3) echo -e "$BLUE banana $END"
;;
    4) echo -e "$PINK cherry $END"
;;
    *) echo "please input {1|2|3|4}"
    exit 1
esac
```
范例3：开发一个给指定内容加指定颜色的脚本要求：1.使用read或传参实现：2.case实现3.以传参为例：在脚本命令行传2个参参数，给指定内容(是第一个参数)加指定颜色（是第二个参数）
```bash
[root@db01 scripts]# cat color_case.sh
#!/bin/bash
RED="\033[31m"
GREEN="\033[32m"
YELLOW="\033[33m"
BLUE="\033[34m"
PINK="\033[35m"
END="\033[0m"

Usage(){
    echo "Please input color and character!"
    exit 1
}

Color(){
case $1 in
    RED)
        echo -e "$RED $2 $END"
        exit 0
    ;;
    GREEN)
        echo -e "$GREEN $2 $END"
        exit 0
    ;;
    YELLOW)
        echo -e "$YELLO $2 $END"
        exit 0
    ;;
    BLUE)
        echo -e "$BLUE $2 $END"
        exit 0  
    ;;
    PINK)
        echo -e "$PINK $2 $END"
        exit 0
    ;;
    *)
        echo "color is {RED|GREEN|YELLO|BLUE|PINK}"
        exit 2
    esac
}

main(){
    if [ $# -ne 2 ]
        then
            Usage
    fi
    Color $1 "$2"
}
main "$@"
```
小结：
1. case语句就相当于多分支的if语句。case语句优势更规范.易读。
2. case语句适合变量的值少，且为固定对的数字或字符串集合
3. 系统服务启动脚本传参的判断多用case语句（值是固定的start|stop|status）

# 5 循环
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;while循环工作中使用的不多，一般是守护进程或始终循环执行的场景会用，其他循环计算，都会用for替换while。
## 5.1 当型循环和直到循环
__1. while 条件句__  
```bash
# 语法：
while 条件
    do
        指令
done
```
__2. until 条件句 （很少用）__
```bash
# 语法
until 条件
    do
        指令
done
```
## 5.2 sleep休息命令
语法：sleep 时间（单位是秒）  
扩展：usleep 微秒（1秒等于1,000,000微秒）  
注意：超过1分钟，要使用定时任务。  
例子：每隔两秒输出一个负载值
```bash
#!/bin/bash
while true
    do
        cat << EOF
-----------------------------------------------------------------------------
EOF
    uptime
    sleep 2
done
```
注意：while true表示条件永远为真，即永远执行下去。
## 5.3 前后台运行
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在执行脚本的时候加上&，表示后台执行  
防止客户端执行脚本中的方法：
```bash
1. sh scripts.sh &
2. nohup /server/scripts/scripts.sh &
3. screen 命令
```
其他操作：  
* &：放在后台执行
* jobs：查看当前窗口后台或暂停的任务
* fg ID：把某一个任务调整到前台
* bg ID：把某一个任务重新调成到后台
* kill %ID：杀掉一个任务
```bash
[root@db01 ~]# sh while3.sh &
[1] 67583
[root@db01 ~]#
[root@db01 ~]# jobs
[1]+  Running                 sh while3.sh &
[root@db01 ~]# kill %1
[root@db01 ~]# jobs
[1]+  Terminated              sh while3.sh
[root@db01 ~]# jobs
[root@db01 ~]#
```
## 5.4 while循环
例子：计算1+...+100之和
```bash
# 方法1：
[root@db01 scripts]# vim while_sum.sh
#!/bin/bash
i=1
while  (( $i <= 100 ))
    do
        ((sum+=i))
        ((i++))
done
echo $sum

# 方法2：
[root@db01 scripts]# vim while_sum.sh
#!/bin/bash
i=1
while [ $i -le 100 ]
    do
        ((sum+=i))
        ((i++))
done
echo $sum
```
例2：列出10到1的所有数字
```bash
[root@db01 scripts]# cat while_num.sh
#!/bin/bash
i=10
while [ $i -ge 1 ]
    do
        echo $i
        ((i--))
done
[root@db01 scripts]#
```
练习题  
1. 猜数字游戏：首先让系统随机生成一个数字，给这个数字定一个范围（数字前50及后50），让用户输入猜的数字，对输入判断，如果不符合数字就给予高与低的提示，猜对后给下猜对用的次数，请用while语句实现。

2. 手机充值10元，每发一次短信（输出当前余额）花费1角5分钱，当余额低于1角5分钱不能发短信，提示余额不足，请充值(可以允许用户充值继续发短信)，请用while语句实现。
解答：单位换算。统一单位，统一成整数，10元=1000分，1角5分=15分

### 5.4.1 while循环读文件
```bash
# 方式1：
exec < file
while read line
    do
        cmd
done

# 方式2：
cat $file_path|while read line
    do
        cmd
done

#方式3：
while read line
    do
        cmd
done < FILE
```
例子：记录日志中访问的文件的大小，并做加法
```bash
[root@db01 scripts]# cat access.sh
#!/bin/bash
i=0
sum=0
exec < /root/access.log
while read line
    do
        i=`echo $line | awk '{print $10}' | grep -E '[0-9]+'`
        ((sum+=i))
done
    echo $sum
# 注意：由于有些访问客户端本地会缓存所以它不会下载，一般这种访问的大小为-，我这里做了grep过滤，即只把数字过滤出来，另一种方法为：
[root@db01 scripts]# vim access_new.sh
#!/bin/bash
while read line
    do
        i=`echo $line | awk '{print $10}'`
        expr $i + 1 &>/dev/null
        if [ $? -ne 0 ]
            then
                continue
        fi
        ((sum+=i))
done < /root/access.log
echo $sum
# 解答：如果$i,不是数字，利用expr的方法判断是否是数字，不是的话，利用continue跳出本次循环，继续下一次。即不是数字的行不加
```
例子：一个httpd.log日志10个IP记录，每十秒钟一个导出到nginx.log里，倒腾到nginx.log里和httpd.log内容一样。
```bash
[root@db01 ~]# cat quip.sh
#!/bin/bash
while read line
    do
        apple2=`grep $line /root/nginx.log | wc -l `
        if [ $apple2 -eq 0 ]
            then
                echo $line >> /root/nginx.log
                sed -i "/$line/d" /root/httpd.log
                sleep 10
                continue
        else
            echo "the ip $line are ready in /root/nginx.log"
            continue
        fi
done < /root/httpd.log
[root@db01 ~]# 
```
### 5.4.2 while循环小结
1. while循环的特长是执行守护进程以及我们系统循环不退出持续执行的情况，用于频率小于1分钟循环处理(crond)，其他的while循环几乎都可以被for替代。
2. if.for最常用.其次（守护进程），case（服务启动脚本）。

## 5.5 for循环
1. 常规语法：
```bash
for 变量 in 变量取值列表
do
       指令
done
# 提示：在此结构中“in 变量取值列表” 可省略，使用for i 就相当于 for i in "$@"
```
2. C语言型语法结构
```bash
for((exp1;exp2;exp3))
    do
        指令
done
```
例子：for和while对比
```bash
for ((i=0;i<5;i++))
    do
        echo $i
done
# 对应的while循环实现方式
i=0
while ((i<5))
    do
        echo $i
        ((i++))
done
```
例子：
```bash
[root@db01 ~]# cat test3.sh
#!/bin.bash
for i in 1 2 3 4 5 6 7 8 9 10
    do
        echo $i
done

# 升级方法1
[root@db01 ~]# cat test4.sh
#!/bin/bash
for i in {1..10}
    do
        echo $i
done
[root@db01 ~]#

#升级方法2
[root@db01 ~]# cat test4.sh
#!/bin/bash
for i in `seq 10`
    do
        echo $i
done
[root@db01 ~]#
```
例子：批量生成10个名称任意的文件
```bash
[root@db01 ~]# cat test6.sh
#!/bin/bash
mkdir ./test
for ((i=1;i<10;i++))
    do
        touch ./test/`echo $RANDOM|md5sum|cut -c 1-8`.log
done
[root@db01 ~]#
```
例子，批量改文件后缀名
```bash
[root@db01 test]# ls
01e2c72d.log  3a14455d.log  3be873ce.log  44902678.log  783e7809.log  7f1fcb3c.log  c4c2399d.log  d55801a9.log  f8e1527b.log
0d339dcf.log  3bc67de9.log  3d90e27a.log  77d45895.log  7a93b22e.log  a29613b1.log  c6909d86.log  ea69f47e.log  fa601ba4.log 

# 方法1:
[root@db01 ~]# cat test7.sh
#!/bin/bash
cd /root/test
for i in `find /root/test/ -name "*.log"`
    do
        name=` echo $i | awk -F '.' '{print $1}'`
        mv $i ${name}.html
done
[root@db01 ~]#

# 方法2：（变量的替换功能）
[root@db01 ~]# vim test8.sh
#!/bin/bash
cd /root/test
for i in `ls`
    do
        mv $i ${i/.html/.jpg}
done
[root@db01 ~]#

# 方法3：拼接功能
[root@db01 ~]# cat test9.sh
#!/bin/bash
cd /root/test
for i in `ls`
    do
        mv $i `echo $i | sed 's#.jpg#.log#g'`
done
[root@db01 ~]#

# 方法4:通过rename改名
[root@db01 ~]# vim test10.sh
#!/bin/bash
cd /root/test
for i in `ls`
    do
        rename ".log" ".jpg" $i
done
[root@db01 ~]#
rename "把什么" "改成什么"  对谁修改
```
例子：批量修改chkconfig管理
```bash
[root@db01 ~]# vim test11.sh
#!/bin/bash
for i in `chkconfig --list | grep 3:on |awk '{print $1}'| grep -Ev "network|crond|rsyslog|sshd|sysstat"`
    do
        chkconfig $i off
done
[root@db01 ~]#
```
例子：利用for命令计算1..100的和
```bash
# C语言型：
[root@db01 ~]# cat test12.sh
#!/bin/bash
sum=0
for ((i=1;i<=100;i++))
    do
        ((sum+=i))
done
echo $sum
[root@db01 ~]#

# 普通型：
[root@db01 ~]# cat test13.sh
#!/bin/bash
sum=0
for i in `seq 100`
    do
        let sum+=i
done
echo $sum
[root@db01 ~]#
```
练习题：等差数列,计算n-m的累加之和可以用等差公式m(n+m)/2
## 5.6 循环控制
$shell循环控制主要有四种方式：break、continue、exit、return。
1. break、continue、exit一般用于循环结构中控制循环（for，while，if）的走向
2. break n：n表示跳出循环的层数，如果省略n，表示跳出整个循环
3. continue n：n表示退出到第n层继续循环，如果省略n表示跳过本次循环，忽略本次循环的声誉代码，进入循环的下一次循环
4. exit n：退出当前shell程序，n为返回值，n也可以省略，再下一个shell里通过$?接收这个n的值。
5. return n：由于在函数里，作为函数的返回值，用于判断函数执行是否正确。

例子1：
```bash
[root@db01 ~]# cat test15.sh
#!/bin/bash
for ((i=1;i<=5;i++))
    do
        if [ $i -eq 3 ]
            then
                #break
                exit
                #continue
        fi
        echo $i
done
echo 'OK'
[root@db01 ~]# sh test15.sh
1
2
[root@db01 ~]#
# 注意：由于使用了exit，所以在循环到第三次的时候，就直接退出了脚本
```
例子2：
```bash
[root@db01 ~]# cat test15.sh
#!/bin/bash
for ((i=1;i<=5;i++))
    do
        if [ $i -eq 3 ]
            then
                #break
                #exit
                continue
        fi
        echo $i
done
echo 'OK'
[root@db01 ~]# sh test15.sh
1
2
4
5
OK
[root@db01 ~]#
# 注意：由于使用了continue，所以在执行到第三次的时候，结束本次循环，继续执行下一次循环。
```
例子3：
```bash
[root@db01 ~]# cat test15.sh
#!/bin/bash
for ((i=1;i<=5;i++))
    do
        if [ $i -eq 3 ]
            then
                break
                #exit
                #continue
        fi
        echo $i
done
echo 'OK'
[root@db01 ~]# sh test15.sh
1
2
OK
[root@db01 ~]#
# 注意：由于使用了break，当执行到第三次的时候，跳出循环，继续执行其他命令。
```
练习题：开发shell脚本实现给服务器临时配置多个别名IP，并可以随时撤销配置的所以IP，IP地址为10.0.2.1-16，其中10.0.2.10不能配置。

# 6 shell函数
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;$简单的说，函数的作用就是把程序里多次调用相同的代码部分定义成一份，然后为这一份代码起个名字，其他所有的重复调用这部分代码就都可以只调用这个名字就OK了。当需要修改在这部分重复代码时，只需要改变函数体内的一份代码即可实现 所有调用修改。
## 6.1 为什么要使用shell函数
使用函数的优势：
1. 把相同的程序段定义成函数，可以减少整个程序的代码量。
2. 可以让程序代码结构更清晰。
3. 增加程序的可读、易读性，以及可管理性。
4. 可以实现程序功能模块化，不同的程序使用函数模块化。     
__注意：对应shell来说，linux的2000多个命令都可以说是shell的函数。__
## 6.2 shell函数语法
简单的语法格式:
```bash
函数名称(){
   指令...
   return n
}
```
规范的语法格式:
```bash
function 函数名(){
   指令...
   return n
}
```
提示：shell的返回值是exit输出返回值，函数里用return输出返回值。
## 6.3 shell的函数执行
1. 直接执行函数名即可（不带括号）  
    * 执行函数的时候，不要带小括号。
    * 函数的定义必须在函数调用前定义，shell的执行是从上到下按行执行的。  
2. 带参数的函数执行方法：
    * 函数名 参数1 参数2 
    * 函数的传参和脚本的传参类似，只是脚本名换成函数名即可。  
    
注意：
+ shell的位置参数($1、$2、$#、$*、$?、$@)都可以是函数的参数。
+ 此时父脚本的参数临时地被函数参数所掩盖或隐藏。
+ $0比较特使，它仍是父脚本的名称。
+ 当函数完成时，原来的命令行脚本的参数即恢复。
+ 在shell函数里面，return命令功能与shell里的exit类似，作用是跳出函数。
+ 在shell函数里使用exit会退出整个shell脚本，而不是退出shell函数。
+ return语句会返回一个退出值（返回值）给调用函数的程序。
+ 函数的参数变量是在函数体里面定义，如果是普通变量一般会使用local i定义（函数里面的变量，只在本函数中生效）。

实例：
```bash
[root@db01 scripts]# cat hanshu.sh
#!/bin/bash
function main(){
    echo "hello,i am handsome man"
}
main
[root@db01 scripts]# sh hanshu.sh
hello,i am handsome man
[root@db01 scripts]#
```
注意:引用其他脚本的函数的时候，可以在本脚本中通过source或者.来调用
```bash
[root@db01 scripts]# cat hanshu.sh
#!/bin/bash
. /etc/init.d/functions
function main(){
    echo "hello,i am handsome man"
    action "is is ok" /bin/true
}
main
[root@db01 scripts]# sh hanshu.sh
hello,i am handsome man
is is ok                                                   [  OK  ]
[root@db01 scripts]#
```
例子：函数传参转成脚本命令传参，对任意指定URL判断是否异常。
1. 脚本传参检查url是否正常。
2. 脚本的功能写成函数
3. 函数传参转成脚本命令行传参，对任意指定的URL进行判断。
```bash
[root@db01 scripts]# cat testurl.sh
#!/bin/bash
[ -f /etc/init.d/functions ] && . /etc/init.d/functions
Usage(){
    echo "USAGE:$0 url"
    exit 1
}

testurl(){
    local code
    code=`curl -I -s $1 -w '%{http_code}' --connect-timeout 3 -o /dev/null`
    if [ $code -eq 200 ]
        then
            action "$1 url" /bin/true
            exit 0
    fi
    action "$1 url" /bin/false
    exit 1
}

main(){
    if [ $# -ne 1 ]
        then
            Usage
    fi
        testurl $1
}
main $*
# 注意：$*加上引号，才表示是一个变量，$*会把命令行的参数都传给shell脚本，比如我输入sh testurl.sh www.baidu.com www.qq.com,$*会直接把www.baidu.com www.qq.com传给shell，由于中间有空格，所以shell函数认为是两个变量，所以会报错。
```
### 6.3.1 shell脚本字体颜色
字体颜色：30-37  
>echo -e "\033[30m 黑色字 \033[0m"  
echo -e "\033[31m 红色字 \033[0m"  
echo -e "\033[32m 绿色字 \033[0m"   
echo -e "\033[33m 黄色字 \033[0m"    
echo -e "\033[34m 蓝色字 \033[0m"  
echo -e "\033[35m 紫色字 \033[0m"  
echo -e "\033[36m 天蓝字 \033[0m"  
echo -e "\033[37m 白色字 \033[0m"  

字背景颜色范围：40-47
>echo -e  "\033[40;37m 黑底白字 \033[0m"  
echo -e "\033[41;37m 红底白字 \033[0m"  
echo -e "\033[42;37m 绿底白字 \033[0m"  
echo -e "\033[43;37m 黄底白字 \033[0m"  
echo -e "\033[44;37m 蓝底白字 \033[0m"  
echo -e "\033[45;37m 紫底白字 \033[0m"  
echo -e "\033[46;37m 天蓝底白字 \033[0m"  
echo -e "\033[47;30m 白底黑字 \033[0m"  
echo -e "\033[41;33m 红底黄字 \033[0m"  
echo -e "\033[41;37m 红底白字 \033[0m"  

__可以通过man console_code来查询__  
实例：传递两个参数给脚本，并打印对应颜色的字体
```bash
[root@db01 scripts]# cat color.sh
#!/bin/bash
RED="\033[31m"
GREEN="\033[32m"
YELLO="\033[33m"
BLUE="\033[34m"
PINK="\033[35m"
END="\033[0m"
Usage(){
    echo "Please input color and character!"
    exit 1
}

Color(){
    if [ "$1" = "RED" ]
        then
            echo -e "$RED $2 $END"
            exit 0
    elif [ "$1" = "GREEN" ]
        then
            echo -e "$GREEN $2 $END"
            exit 0
    elif [ "$1" = "YELLO" ]
        then
            echo -e "$YELLO $2 $END"
            exit 0
    elif [ "$1" = "BLUE" ]
        then
            echo -e "$BLUE $2 $END"
            exit 0
   elif [ "$1" = "PINK" ]
        then
            echo -e "$PINK $2 $END"
            exit 0
    fi
        echo "color is {RED|GREEN|YELLO|BLUE|PINK}"
        exit 2
}

main(){
    if [ $# -ne 2 ]
        then
            Usage
    fi
    Color $1 $2
}
main "$@"
```
# 7 shell数组
平时定义a1,b=2,c=3,变量如果多了，再一个一个定义很费劲，并且去变量也很费劲。  
简单的说，数组就是各种数据类型的元素按一定顺序排列的集合。  
数组就是把有限个元素变量或数据用一个名字命名，然后用编号区分他们的变量的集合。这个名字成为数组名，编号成为数组下标  
组成数组的各个变量成为数组的分量，也成为数组的元素，有时也成为下标变量。
## 7.1 数组的定义
__array=(1 2 3)__   
数组是一个特殊的变量。其中数组名为array，里面有3个元素，可以使用 echo ${#array[*]},来查看数组的长度。这里*和@都可以。  
注意：数组的下标从0开始，即0代表取数组内第一个数据  
例子：循环打印出数组中的IP地址
```bash
# 方法1：
[root@db01 ~]# cat array.sh
#!/bin/bash
array=(
    10.0.0.1
    10.0.0.2
    10.0.0.3
)
for ((i=0;i<${#array[*]};i++))
    do
        echo ${array[i]}
done
[root@db01 ~]#

# 方法2：
[root@db01 ~]# cat array1.sh
#!/bin/bash
array=(
    10.0.0.1
    10.0.0.2
    10.0.0.3
)
for i in ${array[*]}
    do
        echo $i
done
 
[root@db01 ~]#
```
## 7.2 向数组的添加值
```bash
[root@db01 ~]# array=(1 2 3)
[root@db01 ~]# echo ${array[*]}
1 2 3
[root@db01 ~]# array[3]=4
[root@db01 ~]# echo ${array[*]}
1 2 3 4
[root@db01 ~]#
```
7.3 删除数组中的值
```bash
[root@db01 ~]# echo ${array[*]}
1 2 3 4
[root@db01 ~]# unset array[0]
[root@db01 ~]# echo ${array[*]}
2 3 4
[root@db01 ~]#
```
7.4 把命令的结果定义成array
```bash
# 方法1，通过$()取命令的结果
[root@db01 ~]# array=($(ls))
[root@db01 ~]# echo ${array[*]}
2016-07-01_web_error.log access.log array1.sh array.sh eluosi.sh for_sum.sh fruit.sh kuang2 menu.sh oldboy.log oldgirl.log quip.sh test test10.sh test11.sh test12.sh test13.sh test14.sh test15.sh test1.sh test2.sh test3.sh test4.sh test5.sh test6.sh test7.sh test8.sh test9.sh test.sh while1.sh while2.sh while3.sh while.sh
[root@db01 ~]#
# 方法2：通过``取命令的结果
[root@db01 ~]# array=(`ip a l | grep eth | grep inet | awk '{print $NF}'`)
[root@db01 ~]# echo ${array[@]}
eth0 eth1
[root@db01 ~]#
```
## 7.5 小结
1. 定义
    * 静态数组:array=(1 2 3)
    * 动态数组：array=($(ls))
    * 给数组赋值：array[1]=2
2. 打印
    * 打印所有元素：${array[*]}或${array[@]}
    * 打印数组长度：${#array[*]}或${#array[@]}
    * 打印单个元素：${array[i]},i是数组的下标
