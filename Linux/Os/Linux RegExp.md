# 1 什么是正则表达式
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;正则表达式RE（Regular Expression）是由一串字符和元字符构成的字符串。主要功能是文本查询和字符串操作，它可以匹配文本的一个字符或字符集合。实际上正则表达式完成了数据的过滤，将不满足正则表达式定义的数据拒绝掉，剩下与正则表达式匹配的数据。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__简单的说，正则表达式就是为处理大量的字符串而定义的一套规则和方法。__
## 1.1   正则表达式特点
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;正则表达式应用非常广泛，存在于各种语言中，例如：php、python、java等。Linux系统运维工作中的正则表达式，叫做Liunx正则表达式，它与其他语言中的正则表达式有一些不同，在Linux中最常应用正则表达式的是grep（egrep），sed，awk。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;__正则表达式和我们常用的通配符特殊字符是有本质区别的。注意设置字符集 LC_ALL=C其作用是为了去除所有本地化的设置，让命令能正确执行。__
# 2  什么是通配符
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通配符是一种特殊语句，主要有星号(*)和问号(?)，用来模糊搜索文件。当查找文件夹时，可以使用它来代替一个或多个真正字符。当不知道真正字符或者懒得输入完整名字时，常常使用通配符代替一个或多个真正的字符。
## 2.1   通配符和正则表达式的区别
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通配符一般用于命令行bash环境，用于匹配文件的名称而linux正则表达式一般用于awk、grep（egrep）、sed环境用于匹配字符串。
# 3  通配符的含义
## 3.1   *号
表示任意位任意字符，通常用来匹配任意文件名
```bash
[root@lixin ~]# ls
1   123.txt      1.txt  4           persion.txt
12  123.txt.bak  3      daxin.txt  sshd_config
[root@lixin ~]# ls *.txt
123.txt  1.txt  daxin.txt  persion.txt
[root@lixin ~]#
```
## 3.2   ?号
？号表示任意一个字符，多个？号可以叠加连用
```bash
[root@lixin ~]# ls ?.txt       # ？代表一个字符
1.txt  2.txt  3.txt  4.txt  5.txt
[root@lixin ~]# ls ??.txt     # ？？代表两个字符
10.txt  11.txt  12.txt  13.txt  14.txt  15.txt
[root@lixin ~]#
```
## 3.3   {}号
{}表示一个命令的区块组合。
```bash
[root@lixin ~]# ls {1,2}.txt  # 表示ls 1.txt 和2.txt的组合
1.txt  2.txt
[root@lixin ~]# ls {1..5}.txt   # 两个点表示1到5
1.txt  2.txt  3.txt  4.txt  5.txt
[root@lixin ~]#
```
## 3.4   特殊符号
__；号表示两个命令的分隔。__
```bash
[root@lixin ~]# ls 1.txt
1.txt
[root@lixin ~]# ls 2.txt
2.txt
[root@lixin ~]# ls 1.txt;ls 2.txt
1.txt
2.txt
[root@lixin ~]#

#号表示注释，常用在配置文件中注释掉一些内容，比如一些提示信息。
[root@lixin ~]# head -5 /etc/inittab
# inittab is only used by upstart for the default runlevel.
#
# ADDING OTHER CONFIGURATION HERE WILL HAVE NO EFFECT ON YOUR SYSTEM.
#
# System initialization is started by /etc/init/rcS.conf
[root@lixin ~]#
```
__|管道符，用于把管道符前面的数据传递给管道符后面的命令。__
```bash
[root@lixin ~]# ls -l
-rw-r--r-- 1 root root    0 Mar 21 12:57 10.txt
-rw-r--r-- 1 root root    0 Mar 21 12:57 11.txt
-rw-r--r-- 1 root root    0 Mar 21 12:57 12.txt
-rw-r--r-- 1 root root    0 Mar 21 12:56 3.txt
-rw-r--r-- 1 root root    0 Mar 21 12:56 4.txt
-rw-r--r-- 1 root root    0 Mar 21 12:56 5.txt
drwxr-xr-x 2 root root 4096 Mar 21 13:08 lixin
[root@lixin ~]# ls -l | grep '^d'
drwxr-xr-x 2 root root 4096 Mar 21 13:08 lixin
[root@lixin ~]#
```
__~号表示当前用户的家目录，不管在在任何路径下使用cd ~都可以回到用户的家目录（cd后面什么也不加默认就是回到用户的家目录），同时也可以指定进入某个用户的家目录下。__
```bash
[root@lixin sysconfig]# pwd
/etc/sysconfig
[root@lixin sysconfig]# cd     # 回到用户家目录
[root@lixin ~]# pwd
/root
[root@lixin ~]# ls /home
admins  lixin  daxin  dachenzi  tmpdir  tmpdir1
[root@lixin ~]# cd ~daxin     #进入到daxin用户的家目录下
[root@lixin daxin]# pwd
/home/daxin
[root@lixin daxin]#
```
__-号表示上一次切换前的路径，同时由OLDPWD这个变量记录。__
```bash
[root@lixin daxin]# pwd
/home/daxin
[root@lixin daxin]# cd /etc/sysconfig
[root@lixin sysconfig]# pwd
/etc/sysconfig
[root@lixin sysconfig]# cd -
/home/daxin
[root@lixin daxin]# echo $OLDPWD    # -调用的就是OLDPWD变量的值
/etc/sysconfig
[root@lixin daxin]#
```
__$号，表示引用一个变量的值。__
```bash
[root@lixin daxin]# echo $OLDPWD    # 记录上一次切换目录的变量
/etc/sysconfig
[root@lixin daxin]# echo $LANG             # 记录字符集的变量
en_US.UTF-8
[root@lixin daxin]#     
```
__/号，路径的分隔符，同时也表示根目录__
```bash
[root@lixin daxin]# cd /etc/sysconfig
[root@lixin sysconfig]# ls -l /
total 110
drwxr-xr-x   3 root root  4096 Mar 15 12:32 app
dr-xr-xr-x.  2 root root  4096 Mar 13 01:51 bin
dr-xr-xr-x.  5 root root  1024 Mar  3 23:29 boot
drwxr-xr-x  17 root root  3640 Mar 21 11:58 dev
drwxr-xr-x. 91 root root  4096 Mar 21 11:58 etc
drwxr-xr-x   2 root root  4096 Mar 19 01:57 etiantian
drwxr-xr-x.  8 root root  4096 Mar 19 20:02 home
dr-xr-xr-x. 12 root root  4096 Mar  4 17:17 lib
dr-xr-xr-x.  9 root root 12288 Mar  4 17:17 lib64
……
[root@lixin ~]#
```
__\>号表示输出重定向，在重定向到文件中的时候会覆盖文件的内容__
```bash
[root@lixin ~]# cat 1.txt
I am studying Linux
[root@lixin ~]# echo I love you > 1.txt
[root@lixin ~]# cat 1.txt
I love you                                      # 已经覆盖
[root@lixin ~]#
```
__\>>号表示追加输出重定向，在重定向到文件中的时候不会覆盖文件的内容，只会在末尾追加。__
```bash
[root@lixin ~]# cat 1.txt
I love you
[root@lixin ~]# echo I am studying Linux >> 1.txt
[root@lixin ~]# cat 1.txt
I love you
I am studying Linux                    # 没有覆盖
[root@lixin ~]#
```
__<号表示输入重定向，将原本应该由键盘输入的数据经由文件读入。__
```bash
[root@lixin ~]# cat < 1.txt
I love you
I am studying Linux
[root@lixin ~]#
```
__<<号表示追加输入重定向。__
```bash
[root@lixin ~]# cat >> 2.txt << EOF   //
> I am a good boy
> EOF
[root@lixin ~]# cat 2.txt
I am a good boy
[root@lixin ~]#
```
__''号（单引号）表示不解释引号中的内容，所见即所得。__
```bash
[root@lixin ~]# echo '$PATH'
$PATH
[root@lixin ~]# echo $PATH
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
[root@lixin ~]#
```
__""号（双引号）表示如果引号中有变量则解释变量的内容。__
```bash
[root@lixin ~]# echo '$PATH'
$PATH
[root@lixin ~]# echo "$PATH"
/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/root/bin
[root@lixin ~]#
```
__··号（反引号）一般用来解释引号中的命令的。__
```bash
[root@lixin ~]# touch xiaoming_`date +%F`.txt # 表示引用date +%F的值，可以用$()表示引用
[root@lixin ~]# ls
xiaoming_2016-03-21.txt
[root@lixin ~]#
``
__！号表示逻辑上的“非“。__
```bash
[root@lixin ~]# find . -type f               
./.lesshst
./.tcshrc
./4.txt
./.bash_profile
./.cshrc
./.bash_history
[root@lixin ~]# find . -type f ! -name '.*'
./1.txt
./5.txt
./3.txt
./2.txt
./4.txt
[root@lixin ~]#         # 显示出文件名是非.开头的目录
# && 号表示并且的意思，两边条件同时满足，当前一个命令执行成功时，执行后一个命令。（类似于and）
[root@lixin ~]# mkdir lixin && touch lixin/xiaoming
[root@lixin ~]# ls -l lixin/xiaoming
-rw-r--r-- 1 root root 0 Mar 21 13:39 lixin/xiaoming
[root@lixin ~]#         # 表示lixin这个目录创建成功，我才会去新建一个xiaoming文件
```
__||号表示或者的意思，当前一个命令执行失败时，执行后一个命令。类似于（or）__
```bash
[root@lixin ~]# ls
1.txt  2.txt
[root@lixin ~]# ls 1.txt || ls 2.txt
1.txt
[root@lixin ~]# rm 1.txt
rm: remove regular empty file `1.txt`? y
[root@lixin ~]# ls 1.txt || ls 2.txt
ls: cannot access 1.txt: No such file or directory
2.txt
[root@lixin ~]#     # 表示目录下有1.txt则查看，若没有1.txt则执行ls 2.txt
```
__.号表示当前目录，..号表示上一级目录。__
```bash
[root@lixin ~]# pwd
/root
[root@lixin ~]# cd .
[root@lixin ~]# pwd
/root
[root@lixin ~]# cd ..
[root@lixin /]# pwd
/
[root@lixin /]#
```
# 4  正则表达式含义
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Linux中正则表达式分为基础正则表达式（Basic Regular Expression）和扩展正则表达式(Extended Regular Experssion)。
## 4.1基础正则表达式（BRE）
下面以grep为实例，使用grep的—color=auto参数把匹配到的字符以红色显示便于观察。
测试文本为：
```js
I am daxin teacher!
I teach linux.

I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.etiantian.org
my qq num is 49000448.

not 4900000448.
my god ,i am not dachenzi,but daxin!
gd
good
gooood
```
### 4.1.1 ^word
匹配以word开头的内容。vi/vim 里面^代表一行的开头。
```bash
[root@lixin ~]# grep -i '^i' daxin.txt 
I am daxin teacher!
I teach linux.
I like badminton ball ,billiard ball and chinese chess!
[root@lixin ~]#                   # grep –I 表示忽略大小写
```
### 4.1.2  word$
word$ 匹配以word结尾的内容，vim/vi里面$代表一行的末尾。
```bash
[root@lixin ~]# grep -i 'm$' daxin.txt
my blog is http://daxin.blog.51cto.com
[root@lixin ~]#
```
### 4.1.3  ^$
^$表示以结尾为开头，以开头为结尾，表示空行。
```bash
[root@lixin ~]# grep -n '^$' daxin.txt
3:
8:
[root@lixin ~]#         # -n表示匹配到的行添加行号，为了显示去匹配到了空行
```
### 4.1.4  .
.表示代表任意一个字符。
```bash
[root@lixin ~]# grep 'o.d' daxin.txt
I am daxin teacher!
my blog is http://daxin.blog.51cto.com
my god ,i am not dachenzi,but daxin!
good
[root@lixin ~]#
```
### 4.1.5 \
\表示转意符号，在正则里面加入我们真的要使用.这个符号，由于它有特别的意义，需要使用\号，让它脱掉马甲，还原真意。
```bash
[root@lixin ~]# grep '.$' daxin.txt 
I am daxin teacher!
I teach linux.
I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.etiantian.org
my qq num is 49000448.
not 4900000448.
my god ,i am not dachenzi,but daxin!
gd
good                  //不转意的话，由于.表示任意字符，所以都能匹配到
[root@lixin ~]# grep '\.$' daxin.txt
I teach linux.
my qq num is 49000448.
not 4900000448.
[root@lixin ~]#  //转意之后只能匹配.结尾
```
### 4.1.6 *
*号，表示前面的字符0次或多次出现。
```bash
[root@lixin ~]# grep 'go*d' daxin.txt
my god ,i am not dachenzi,but daxin!
gd
good
[root@lixin ~]#  //可以匹配到0次o，或者多次o
```
### 4.1.7 .*
.*号，为组合符号，.表示匹配任意字符，*表示前面的字符0次或多次出现，加在一起一般会匹配所有，Linux中一般成它们的组合为贪婪匹配。
```bash
[root@lixin ~]# grep -n '.*' daxin.txt
1:I am daxin teacher!
2:I teach linux.
3:
4:I like badminton ball ,billiard ball and chinese chess!
5:my blog is http://daxin.blog.51cto.com
6:our site is http://www.etiantian.org
7:my qq num is 49000448.
8:
9:not 4900000448.
10:my god ,i am not dachenzi,but daxin!
11:gd
12:good
[root@lixin ~]#         //贪婪匹配，所以连空行也能匹配到
```
### 4.1.8 [abc]
[abc]表示匹配括号内任意的一个内容，也可以写为[0-9]，表示任意的数字。[a-z]表示任意的小写字母，[A-Z]表示任意的大写字母。
```bash
[root@lixin ~]# grep '[abc]' daxin.txt
I am daxin teacher!
I teach linux.
I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.etiantian.org
my god ,i am not dachenzi,but daxin!            # 仅仅匹配到了abc
[root@lixin ~]# grep '[0-9]' daxin.txt     
my blog is http://daxin.blog.51cto.com
my qq num is 49000448.
not 4900000448.                        # 匹配到了数字
[root@lixin ~]# grep '[a-z]' daxin.txt   
I am daxin teacher!
I teach linux.
I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.etiantian.org
my qq num is 49000448.
not 4900000448.
my god ,i am not dachenzi,but daxin!
gd
good                                    # 匹配到所有小写字母
[root@lixin ~]#
```
### 4.1.9  [^abc]
[^abc]表示匹配非abc的字符，^放在[]里面表示非。
```bash
[root@lixin ~]# grep '[^0-9]' daxin.txt
I am daxin teacher!
I teach linux.
I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.etiantian.org
my qq num is 49000448.
not 4900000448.
gd
good
[root@lixin ~]#
```
### 4.1.10  a\{n,m\}
a\{n,m\}表示括号外的a重复至少n次，最多m次，因为使用了扩展正则表达式的{}的符号，所以要用转意符号。如果是grep的话可以使用grep –E（egrep）来支持扩展正则表达式。
```bash
[root@lixin ~]# grep 'b\{1,2\}' daxin.txt 
I am daxin teacher!
I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
my god ,i am not dachenzi,but daxin!
[root@lixin ~]#
[root@lixin ~]# grep -E 'b{1,2}' daxin.txt
I am daxin teacher!
I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
my god ,i am not dachenzi,but daxin!
[root@lixin ~]#    # grep –E==egrep，表示b匹配最少出现1次，最多出现2次。
```
### 4.1.11  a\{n,\}
a\{n,\}表示括号外的内容最少出现n次。
```bash
[root@lixin ~]# grep -E 'o{2,}' daxin.txt 
good
[root@lixin ~]#
```
4.1.12  a\{n\}
a\{n\}表示重复前面的字符a等于n次。
```bash
[root@lixin ~]# grep -E 'o{1}' daxin.txt
I am daxin teacher!
I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.etiantian.org
not 4900000448.
my god ,i am not dachenzi,but daxin!
good  //匹配了两次，不是一次匹配了2个。
[root@lixin ~]#
```
### 4.1.13  a\{,m\}
a\{,m\}表示匹配前面的字符a最多m次。
```bash
[root@lixin ~]# grep -E '0{,3}' daxin.txt  
I am daxin teacher!
I teach linux.

I like badminton ball ,billiard ball and chinese chess!
my blog is http://daxin.blog.51cto.com
our site is http://www.etiantian.org
my qq num is 49000448.

not 4900000448.
my god ,i am not dachenzi,but daxin!
gd
good
[root@lixin ~]#
```
## 4.2 扩展正则表达式（ERE）
### 4.2.1 +
+号，表示前面的字符1次或多次出现，注意和*区分开，*表示的是0次或多次出现。
```bash
[root@lixin ~]# grep -E 'go+d' daxin.txt
my god ,i am not dachenzi,but daxin!
good
[root@lixin ~]#     //表示g和d之前的后出现1次以上
```
### 4.2.2 ？
?表示重复前面的字符0次或1次。
```bash
[root@lixin ~]# grep -E 'go?d' daxin.txt 
my god ,i am not dachenzi,but daxin!
gd
[root@lixin ~]#  //表示g和d之间的o出现o次或1次
```
### 4.2.3 |
|表示或，用于同时查找多个字符。
```bash
[root@lixin ~]# grep -E 'daxin|dachenzi' daxin.txt
I am daxin teacher!
my blog is http://daxin.blog.51cto.com
my god ,i am not dachenzi,but daxin!
[root@lixin ~]#         //表示匹配daxin或dachenzi的列
```
### 4.2.4 ()
()表示用户组，括号中的内用以组的形式出现。但是在sed中一般用来表示后巷引用。
```bash
[root@lixin ~]# grep -E 'g(oo)d' daxin.txt
good       
[root@lixin ~]#                 //只匹配g和d之间是oo的列
```
# 5 扩展实例
## 5.1取出IP地址信息
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;思路：一般用^.*表示目标项前面的部分，^表示开头，后面写上目标项前面的字母。目标项后面的字符用.*$表示，$表示结尾，前面写上目标项后面的字母即可。使用grep、awk、sed命令或搭配其他命令取出。
```bash
[root@lixin ~]# ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 00:0C:29:E4:83:06 
   inet addr:10.0.0.8  Bcast:10.0.0.255  Mask:255.255.255.0
   inet6 addr: fe80::20c:29ff:fee4:8306/64 Scope:Link
   UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
   RX packets:1993 errors:0 dropped:0 overruns:0 frame:0
   TX packets:960 errors:0 dropped:0 overruns:0 carrier:0
   collisions:0 txqueuelen:1000
   RX bytes:172487 (168.4 KiB)  TX bytes:107575 (105.0 KiB)
[root@lixin ~]#                   //测试文本

# 方法一：使用sed后项引用方式，先把IP用（）匹配到，然后再引用
[root@lixin ~]# ifconfig eth0 | sed -nr '2s#^.*r:(.*).?B.*$#\1#gp'
10.0.0.8 
[root@lixin ~]#

# 方法二：利用awk多分隔符的方式，最好取目标项左右两边的字符为分隔符 (最佳)
[root@lixin ~]# ifconfig eth0 | awk -F "[ :]+" 'NR==2 {print $4}'
10.0.0.8
[root@lixin ~]#

# 方法三：利用grep过滤出目标行，使用cut数字符（比较low的方法）
[root@lixin ~]# ifconfig eth0 | grep 'inet addr' | cut -c 21-28
10.0.0.8
[root@lixin ~]#

# 方法四：由于cut只能识别一个分隔符，所以我们利用tr把多分隔符转换成一个进行分割
[root@lixin ~]# ifconfig eth0 | sed -n 2p | tr ':' ' '|cut -d " " -f13
10.0.0.8                
[root@lixin ~]#

# 方法五：利用awk过滤关键行，利用sed把目标项之前的项，替换成空
[root@lixin ~]# ifconfig eth0 | awk '/inet addr/''{print $2}' | sed 's#^a.*:##g'
10.0.0.8
[root@lixin ~]#

# 方法六：使用ip命令查看网卡IP地址，然后awk多分隔符取出IP地址
ip addr show eth0| awk -F '[ /]+' 'NR==3{print $3}'
```
### 5.2取出文件或目录的权限
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当命令结果包含我们需要的内容的时候，我们要想到命令的参数是否有具有的参数能够一步达到我们需要的结果呢？通常在manual中会记录，这时可以使用man command查寻。
```bash
[root@lixin ~]# stat daxin.txt      
File: `daxin.txt'
Size: 261             Blocks: 8          IO Block: 4096   regular file
Device: 803h/2051d      Inode: 25401       Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Access: 2016-03-21 14:47:16.710194978 +0800
Modify: 2016-03-21 14:47:15.268186883 +0800
Change: 2016-03-21 14:47:15.281186631 +0800
[root@lixin ~]#                   //测试文本

# 方法一：使用stat内置参数-c，配合%a来用数字显示文件的权限信息（最佳）
[root@lixin ~]# stat -c %a daxin.txt
644
[root@lixin ~]#

# 方法二：利用awk指定多分隔符，去目标项左右两边的字符为分隔符
[root@lixin ~]# stat daxin.txt | awk -F '[0/]' 'NR==4 {print $2}'
644
[root@lixin ~]#

# 方法三：利用sed的后向引用
[root@lixin ~]# stat daxin.txt | sed -nr '4s#^A.*\(0(.*)/-.*$#\1#gp'
644
[root@lixin ~]#

# 方法四：使用ls –l列出字母方式权限，替换成数字，相加完成（比较low，但思路特别）
[root@lixin ~]# ls -l daxin.txt | awk '{print $1}'|cut -c 2- | tr "rwx-" "4210" | awk -F "" '{print $1+$2+$3$4+$5+$6$7+$8+$9}'
644
[root@lixin ~]#
```