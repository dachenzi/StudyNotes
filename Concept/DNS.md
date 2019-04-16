
<!-- TOC -->

- [1 什么是DNS](#1-什么是dns)
- [2 DNS域名系统](#2-dns域名系统)
- [3 DNS域名结构](#3-dns域名结构)
- [4 域名服务器](#4-域名服务器)
- [5 域名解析过程](#5-域名解析过程)
- [6 特殊记录说明](#6-特殊记录说明)
    - [6.1 什么是A记录？](#61-什么是a记录)
    - [6.2 什么是NS记录？](#62-什么是ns记录)
    - [6.3 什么是CNAME记录？](#63-什么是cname记录)
    - [6.4 什么是MX记录](#64-什么是mx记录)
    - [6.5 什么是PTR记录](#65-什么是ptr记录)
- [7 DNS相关命令](#7-dns相关命令)

<!-- /TOC -->
# 1 什么是DNS
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`Domain Name System`，域名系统。因特网上作为域名和IP地址相互映射的一个分布式数据库，能够使用户更方便的访问互联网，而不用去记住能够被机器直接读取的IP数串。通过主机名，最终得到该主机名对应的IP地址的过程叫做域名解析（或主机名解析）。DNS协议运行在`UDP`协议之上，使用端口号`53`。
# 2 DNS域名系统
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;域名系统DNS(Domain Name System)是因特网使用的命名系统，用来把便于人们使用的机器名字转换成为IP地址。域名系统其实就是名字系统。为什么不叫"名字"而叫"域名"呢？这是因为在这种因特网的命名系统中使用了许多的"域(domain)"，因此就出现了"域名"这个名词。"域名系统"明确地指明这种系统是应用在因特网中。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们都知道，`IP`地址是由`32`位的二进制数字组成的。用户与因特网上某台主机通信时，显然不愿意使用很难记忆的长达32位的二进制主机地址。即使是点分十进制IP地址也并不太容易记忆。相反，大家愿意使用比较容易记忆的主机名字。但是，机器在处理IP数据报时，并不是使用域名而是使用IP地址。这是因为IP地址长度固定，而域名的长度不固定，机器处理起来比较困难。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;因为因特网规模很大，所以整个因特网只使用一个域名服务器是不可行的。因此，早在1983年因特网开始采用层次树状结构的命名方法，并使用分布式的域名系统DNS。并采用客户服务器方式。DNS使大多数名字都在`本地解析(resolve)`，仅有少量解析需要在因特网上通信，因此DNS系统的效率很高。由于DNS是分布式系统，即使单个计算机除了故障，也不会妨碍整个DNS系统的正常运行。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;域名到IP地址的解析是由分布在因特网上的许多域名服务器程序共同完成的。域名服务器程序在专设的结点上运行，而人们也常把运行域名服务器程序的机器称为域名服务器。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;域名到IP地址的解析过程的要点如下：当某一个应用需要把主机名解析为IP地址时，该应用进程就调用解析程序，并称为DNS的一个客户，把待解析的域名放在DNS请求报文中，以UDP用户数据报方式发给本地域名服务器。本地域名服务器在查找域名后，把对应的IP地址放在回答报文中返回。应用程序获得目的主机的IP地址后即可进行通信。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;若本地域名服务器不能回答该请求，则此域名服务器就暂时称为DNS的另一个客户，并向其他域名服务器发出查询请求。这种过程直至找到能够回答该请求的域名服务器为止。
# 3 DNS域名结构
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于因特网的用户数量较多，所以因特网在命名时采用的是层次树状结构的命名方法。任何一个连接在因特网上的主机或路由器，都有一个唯一的层次结构的名字，即域名(domain name)。这里，"域"(domain)是名字空间中一个可被管理的划分。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从语法上讲，每一个域名都是有标号(label)序列组成，而各标号之间用点(小数点)隔开。
如下例子所示：  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;![dns](photo/dns.png)  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这是中央电视台用于手法电子邮件的计算机的域名，它由三个标号组成，其中标号`com`是顶级域名，标号`cctv`是二级域名，标号`mail`是三级域名。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DNS规定，域名中的标号都有英文和数字组成，每一个标号不超过63个字符(为了记忆方便，一般不会超过`12`个字符)，也不区分大小写字母。标号中除连字符(-)外不能使用其他的标点符号。级别最低的域名写在最左边，而级别最高的字符写在最右边。由多个标号组成的完整域名总共不超过`255`个字符。DNS既不规定一个域名需要包含多少个下级域名，也不规定每一级域名代表什么意思。各级域名由其上一级的域名管理机构管理，而最高的顶级域名则由`ICANN`进行管理。用这种方法可使每一个域名在整个互联网范围内是唯一的，并且也容易设计出一种查找域名的机制。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;域名只是逻辑概念，并不代表计算机所在的物理地点。据2006年12月统计，现在顶级域名TLD(Top Level Domain)已有`265`个，分为三大类：
1. 国家顶级域名`nTLD`：采用ISO3166的规定。如：`cn`代表中国，`us`代表美国，`uk`代表英国，等等。国家域名又常记为`ccTLD`(cc表示国家代码contry-code)。
2. 通用顶级域名`gTLD`：最常见的通用顶级域名有7个，即：`com`(公司企业)，`net`(网络服务机构)，`org`(非营利组织)，`int`(国际组织)，`gov`(美国的政府部门)，`mil`(美国的军事部门)。
3. 基础结构域名(infrastructure domain)：这种顶级域名只有一个，即`arpa`，用于反向域名解析，因此称为反向域名。  
__注：目前全球有13台根服务器。__
# 4 域名服务器
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果采用上述的树状结构，每一个节点都采用一个域名服务器，这样会使得域名服务器的数量太多，使域名服务器系统的运行效率降低。所以在DNS中，采用`划分区`的方法来解决。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个服务器所负责管辖(或有权限)的范围叫做区(`zone`)。各单位根据具体情况来划分自己管辖范围的区。但在一个区中的所有节点必须是能够连通的。每一个区设置相应的权限域名服务器，用来保存该区中的所有主机到域名IP地址的映射。
- `顶级域名服务器`：负责管理在该顶级域名服务器注册的一级域名。
- `权限域名服务器`：负责一个"区"的域名服务器。
- `本地域名服务器`：本地服务器不属于下图的域名服务器的层次结构，但是它对域名系统非常重要。当一个主机发出DNS查询请求时，这个查询请求报文就发送给本地域名服务器。
# 5 域名解析过程
1. 主机向本地域名服务器的查询一般都是采用递归查询。所谓递归查询就是：如果主机所询问的本地域名服务器不知道被查询的域名的IP地址，那么本地域名服务器就以DNS客户的身份，向其它根域名服务器继续发出查询请求报文(即替主机继续查询)，而不是让主机自己进行下一步查询。因此，递归查询返回的查询结果或者是所要查询的IP地址，或者是报错，表示无法查询到所需的IP地址。
2. 本地域名服务器向根域名服务器的查询叫做迭代查询。迭代查询的特点：当根域名服务器收到本地域名服务器发出的迭代查询请求报文时，要么给出所要查询的IP地址，要么告诉本地服务器："你下一步应当向哪一个域名服务器进行查询"。然后让本地服务器进行后续的查询。根域名服务器通常是把自己知道的顶级域名服务器的IP地址告诉本地域名服务器，让本地域名服务器再向顶级域名服务器查询。顶级域名服务器在收到本地域名服务器的查询请求后，要么给出所要查询的IP地址，要么告诉本地服务器下一步应当向哪一个权限域名服务器进行查询。最后，知道了所要解析的IP地址或报错，然后把这个结果返回给发起查询的主机。  
小结：
3. （1）`递归查询`: 一般客户机和服务器之间属递归查询，即当客户机向DNS服务器发出请求后,若DNS服务器本身不能解析,则会向另外的DNS服务器发出查询请求，得到结果后转交给客户机；
3. （2）`迭代查询`(反复查询): 一般DNS服务器之间属迭代查询，如：若DNS2不能响应DNS1的请求，则它会将DNS3的IP给DNS2，以便其再向DNS3发出请求；
4. 用户访问www.baidu.com，向本地DNS请求百度的服务器地址。
5. 本地域名服务器没有查到本地缓存的记录，请求根DNS服务器响应，根DNS服务器把请求的顶级域名服务器地址（com）告诉它，让它去找com解析。
6. 本地域名服务器得到根域名服务器回应后，就去访问com，请求com进行解析，com服务器会把请求的一级域名服务器地址（baidu.com）告诉它，r让它去找baidu.com解析。
7. 本地域名服务器得到一级域名服务响应后，就去访问baidu.com，请求baidu.com进行解析，baidu.com服务器会把自己的www主机地址，反馈给本地域名服务器，本地域名缓存之后，会把www.baidu.com及对应的ip地址反馈给客户机。
8. 客户机根据得到的IP地址，直接访问www.baidu.com服务器。
# 6 特殊记录说明
## 6.1 什么是A记录？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`A (Address) 记录`是用来指定主机名（或域名）对应的IP地址记录。用户可以将该域名下的网站服务器指向到自己的web server上。同时也可以设置域名的二级域名。
## 6.2 什么是NS记录？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`NS（Name Server）记录`是域名服务器记录，用来指定该域名由哪个DNS服务器来进行解析。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;DNS记录在不同的环境下需要特定的域名设置，相应地就会有不同的记录来表示和描述。比如说A (Address) 记录是用来指定主机名(或域名)对应的IP地址记录。用户可以将该域名下的网站服务器指向到自己的web server上。`记录(CNAME)`也被称为规范名字。这种记录允许您将多个名字映射到同一台计算机。通过`MX(Mail Exchange)记录`，用户可以将该域名下的邮件服务器指向到自己的mail server上，然后即可自行操作控制所有的邮箱设置。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;而NS(Name Server)记录是域名服务器记录，用来指定该域名由哪个DNS服务器来进行解析。我们在使用域名解析过程中，后台操作都会有域名管理这一项，尤其是我们选择了智能DNS解析的时候，在域名管理过程中经常会遇到NS相关的一些问题。如域名转入，以及记录解释里的添加、修改、删除等相关操作都会涉及到NS问题。
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们知道DNS在网络中扮演的角色相当重要，其地位完全可以用举足轻重来形容。所以DNS的正常与否，直接影响到整个网络的正常运作，同时还涉及到相关安全问题。在这种情况下，安全起见，一般都会有多台服务器备用可供选择，在机器故障发生应急时使用。这些服务器组合在一起又叫服务器组，一个服务器组里至少包含两个服务器。
## 6.3 什么是CNAME记录？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;别名记录也被称为规范名字。这种记录允许将多个名字映射到同一台计算机。通常用于同时提供WWW和MAIL服务的计算机。例如，有一台计算机名为"host.domain.com"（A记录）。它同时提供WWW和MAIL服务，为了便于用户访问服务。可以为该计算机设置两个别名（CNAME）：WWW和MAIL。这两个别名的全称就是"www.domain.com"和"mail.domain.com"。实际上他们都指向"host.domain.com"。
## 6.4 什么是MX记录
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过MX(Mail Exchange)记录，用户可以将该域名下的邮件服务器指向到自己的mail server上，然后即可自行操作控制所有的邮箱设置。
## 6.5 什么是PTR记录
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`PTR (Pointer Record)`，指针记录，是电子邮件系统中的一种数据类型，被互联网标准文件RFC1035所定义。与其相对应的是A记录、地址记录。二者组成邮件交换记录。
- A记录解析名字到地址，而PTR记录解析地址到名字。
- 地址是指一个客户端的IP地址，名字是指一个客户的完全合格域名。
# 7 DNS相关命令
__`dig`__
```bash
[root@lixin ~]# dig www.baidu.com
 
; <<>> DiG 9.8.2rc1-RedHat-9.8.2-0.37.rc1.el6 <<>> www.baidu.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 22543
;; flags: qr rd ra; QUERY: 1, ANSWER: 2, AUTHORITY: 0, ADDITIONAL: 0
 
;; QUESTION SECTION:
;www.baidu.com.                 IN      A
 
;; ANSWER SECTION:
www.baidu.com.          5       IN      A       119.75.218.70
www.baidu.com.          5       IN      A       119.75.217.109
 
;; Query time: 7 msec
;; SERVER: 10.0.0.2#53(10.0.0.2)
;; WHEN: Thu Apr 14 17:44:17 2016
;; MSG SIZE  rcvd: 63
 
[root@lixin ~]#
nslookup
[root@lixin ~]# nslookup www.baidu.com
Server:         10.0.0.2
Address:        10.0.0.2#53
 
Non-authoritative answer:
Name:   www.baidu.com
Address: 119.75.217.109
Name:   www.baidu.com
Address: 119.75.218.70
 
[root@lixin ~]#
```
__`host`__
```bash
[root@lixin ~]# host www.baidu.com
www.baidu.com has address 119.75.218.70
www.baidu.com has address 119.75.217.109
www.baidu.com is an alias for www.a.shifen.com.
www.baidu.com is an alias for www.a.shifen.com.
[root@lixin ~]#
```