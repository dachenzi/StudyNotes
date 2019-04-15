<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 简介](#1-简介)
    - [1.1 基本组成](#11-基本组成)
    - [1.2 命令介绍](#12-命令介绍)
    - [1.3 Inventory文件介绍](#13-inventory文件介绍)
- [2 Ansible配置文件介绍](#2-ansible配置文件介绍)
    - [2.1 常用配置](#21-常用配置)
    - [2.2 ssh相关](#22-ssh相关)
    - [2.3 权限提升相关](#23-权限提升相关)
- [3 常用模块](#3-常用模块)
    - [3.1 setup](#31-setup)
    - [3.2 file](#32-file)
    - [3.3 copy](#33-copy)
    - [3.4 command](#34-command)
    - [3.5 shell](#35-shell)
    - [3.6 service](#36-service)
    - [3.7 cron](#37-cron)
    - [3.8 filesystem](#38-filesystem)
    - [3.9 yum](#39-yum)
    - [3.10 user](#310-user)
    - [3.11 synchronize](#311-synchronize)
    - [3.12 mount](#312-mount)
    - [3.12 template](#312-template)
    - [3.13 get_url](#313-get_url)
    - [3.14 unarchive](#314-unarchive)
    - [3.15 git](#315-git)
    - [3.16 stat](#316-stat)
    - [3.17 sysctl](#317-sysctl)
- [4 模块的返回值](#4-模块的返回值)
- [5 YAML简介](#5-yaml简介)
    - [5.1 变量](#51-变量)
        - [5.1.1.facts](#511facts)
        - [5.1.2 register(注册器)](#512-register注册器)
        - [5.1.3 通过命令行传递变量](#513-通过命令行传递变量)
    - [5.2 Inventory](#52-inventory)
        - [5.2.1 文件格式](#521-文件格式)
        - [5.2.2 主机变量](#522-主机变量)
        - [5.2.3 组变量](#523-组变量)
        - [5.2.4 组嵌套](#524-组嵌套)
        - [5.2.5 inenvtory 参数](#525-inenvtory-参数)
    - [5.3 条件测试](#53-条件测试)
    - [5.4 迭代](#54-迭代)
- [6 playbook简介](#6-playbook简介)
    - [6.1 playbook的一般结构](#61-playbook的一般结构)
    - [6.2 target 配置项目](#62-target-配置项目)
    - [6.3 variable 可用的配置项目](#63-variable-可用的配置项目)
    - [6.4 task 配置项目](#64-task-配置项目)
    - [6.5 handler 配置项目](#65-handler-配置项目)
    - [6.6 ansible-playbook命令](#66-ansible-playbook命令)
    - [6.7 其他配置](#67-其他配置)
- [7 Roles](#7-roles)
    - [7.1 role内各目录中的文件说明](#71-role内各目录中的文件说明)
    - [7.2 ansible-galaxy 命令](#72-ansible-galaxy-命令)

<!-- /TOC -->

# 1 简介
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;高度模块化，调用特定的模块，完成特定的任务，基于Yaml，来完成批量任务的模板化，来支持playbook。基于Python语言实现，主要使用Paramiko、PyYAML和JinJa2三个关键模块，部署简单(agentless)，主从模式，支持自定义模块，支持playbook，幂等性：允许重复执行N次，没有变化时，只会执行第一次。  
特点：
1. Configuration(cfengine,chef,puppet) 配置管理
2. Deployment(Capistrano,Fabric) 部署发布
3. Ad-Host Task(func) 命令行批量执行
4. Multi-Ter Orchestration(Juju,sort of) 多层次任务编排

PS：集成了上面括号中的工具的主要功能

## 1.1 基本组成
主要由模块、插件、主机群、以及剧本的组成，各部分含义如下：
1. 核心模块(core modules):Ansible 自带的模块。
2. 自定义模块(custom Modules)：如果核心模块不足以完成某种功能，可以自行添加自定义模块(支持市面上大部分的编程语言)。
3. 插件(Plugins)：支持使用插件的方式对ansible本身的功能进行扩展。模块是用来实现任务的，增强ansible平台自己的功能就需要使用插件（loggin插件记录日志，email插件发送邮件）
    - 其中最常用是：连接插件(Connectior Plugins) ：ansibile基于连接插件连接到各个主机上，虽然默认情况下ansible使用ssh连接到各个主机上，但它还支持其他的连接方法(mq)。
4. 主机群(Host Inventory): 主机清单，定义ansible管理的主机，还可以存放一下针对不同主机的变量，也可以写入主机的用户名和密码
5. 剧本(playbooks)：ansible的任务配置文件，将多个任务定义在剧本中，由ansible自动执行
```bash
yum install -y ansible
pip3 install ansible
```
运行原理：把命令翻译成shell命令，拷贝到目标主机(/root/.ansible/tmp/下)，再执行，执行完毕后删除tmp文件。

## 1.2 命令介绍
ansible 命令主要参数信息
- -u : remote user，默认使用root用户登陆
- -i : Inventory，指定主机，默认是/etc/ansible/hosts
- -m ：指定模块的名称（不指定-m，那么默认是command模块）
- -a : 模块的参数(比如使用command模块，那么-a参数就是要执行的命令)
- -k : 用来提示输入远程主机的密码(基于用户密码登录)
- -f : 一次执行几个进程(并发数量)，默认为5个
- --sudo : 执行命令时使用 sudo 权限(需要用户具有sudo权限)
- --key-file: 建立SSH链接的私钥文件
- --list-hosts: 列出匹配到的服务器列表

基于主机清单：
```python
ansible -i /etc/ansible/hosts test -u root -m command -a 'ls /home' -k （ansible 依赖 sshpass）
|
--> ansible test -a 'ls /home' -k 

# 上面两条命令默认状态下效果相同

# 我们知道ansible通过ssh的方式来远程管理多台主机，所以我们需要使用ssh key的方式来进行ssh认证，当然你也可以使用ansible的时候加上-k，来通过交互式输入密码。当有了ssh key以后，那么我们就可以直接使用ansbile来执行任务了,比如：ansible all -m ping 

# 基于主机： 
ansible 127.0.0.1 -m ping
```
## 1.3 Inventory文件介绍
/etc/ansible/hosts:(默认的Inventory)
```python
[test]                    # 用于定义主机组,all表示所有主机，所以尽量避免使用all作为组名
127.0.0.1                 # 该组的主机列表，可以是主机名(dns解析)或者IP地址，或者用符号来表示连续的主机
192.168.1.[80:88]         # 表示 192.168.1.80 - 192.168.1.88

[web:children]            # 表示子组
nginx                     # nginx组
test                      # test组
配置文件中的变量及参数：

[nginx]
127.0.0.1 ansible_ssh_user=root ansible_ssh_host=web1 # 更细致的配置

    - ansible_ssh_user : 用于指定管理远程主机的帐号
    - ansible_ssh_host : 用于指定被管理的主机
    - ansible_ssh_port ：用于指定ssh的端口
    - ansible_ssh_private_key_file ：指定key文件
    - host_key_checking=False ：当第一次连接主机时，会提示yes/no，跳过此次环节
```
# 2 Ansible配置文件介绍
ansible 查找 Ansible.cfg 文件遵循以下顺序：
1. ANSIBLE_CONFIG环境变量指定的配置文件
2. 当前目录下的ansible.cfg文件
3. 当前用户home目录下的.ansible.cfg文件
4. Ansible默认的/etc/ansible/ansible.cfg文件

## 2.1 常用配置
- inventory：指定inventory文件的路径
- remote_user：SSH连接时使用的用户名
- remote_port：SSH连接时使用的端口号
- private_key_file：SSH连接时使用的私钥文件
- roles_path：查找roles的路径，可以指定多个查找路径，多个路径之间用冒号分隔
- log_path：Ansible的日志文件路径
- host_key_checking：类似于ssh命令中的StrictHostKeyChecking选项，当等于False时，不检查远程主机是否存在于Konw_hosts文件中
- forks：并行进程的数量
- gathering：控制收集Facts变量的策略

## 2.2 ssh相关
- ssh_args：可以通过这个参数控制Ansible的ssh连接
- pipelining: 多个task之间共享SSH连接，开启pipelining能够有效提升Ansible的执行速度
- control_path：保存ControlPath socket的路径

## 2.3 权限提升相关
- become：是否进行权限提升
- become_method：权限提升的方式，默认为sudo
- become_user：提升为哪个用户的权限，默认为root
- become_ask_pass：默认为False，表示权限提升时不需要密码(设置为true时，手动输入密码，或者配置ansible_become_pass变量)

# 3 常用模块
查看模块的帮助：
```python
ansible-doc -l # 查看核心的模块(包含模块的大致说明,比较慢)
ansible-doc -s 'command' # 执行模块名，可以列出模块的用法
```
## 3.1 setup
用于统计系统的相关信息
```python
# 例子
ansible test -m setup 
# 参数
  - filter： 过滤要显示的内容
  - ansible test -m setup -a "filter=ansible_python_version" -k # 只获取主机ansible_python_version的值
  - 更多的帮助 ansible-doc -s setup
```

## 3.2 file
设置文件属性,常用参数如下：
  - force: 在两种情况下会强制创建软连接，默认值为 no
  - 源文件不存在但之后会建立的情况下
  - 目标软连接已存在，需要先取消之前的软连，然后创建新的软链
  - group：定义文件的属组
  - mode：定义文件的权限
  - owner：定义文件的属主
  - path：必选项，定义文件的路径
  - recurse：递归设置文件的属性，只对目录生效（-R）
  - src：要被链接的源文件的属性，只应用于state=link的情况
  - dest：被链接到的路径，只应用与state=link的情况
  - state：
  - directory：如果目录不存在创建目录
  - file：即使文件不存在，也不会被创建
  - link：创建软连接
  - hard：创建硬链接
  - touch：如果文件不存在，则会创建一个新的文件，如果文件存在，则会更新其最后修改时间
  - absent：删除目录，文件或者取消链接文件

例子:
```python
ansible test -m file -a 'src=/etc/fstab dest=/tmp/fstab state=link' -k                              # 等于 ln -s /etc/fstab /tmp/fstab
ansible test -m file -a 'path=/tmp/fstab state=absent' -k                                           # 删除文件
ansible test -m file -a 'path=/tmp/lixin.txt state=touch owner=nobody group=nobody mode=666' -k     # 创建文件指定其属主，属组，权限
```
## 3.3 copy
复制文件到远程主机，每次备份会产生一个md5sum，如果两次赋值文件的md5sum相同，那么就不会再次执行复制动作。

参数
  - backup：在覆盖之前将原文件备份，备份文件名包含时间信息。有两个选项：yes | no 
  - src：源文件，如果是目录的话，会递归赋值，包括目录本身，如果目录为/，那么只会复制目录下的文件
  - dest：目标文件路径
  - others：所有file模块里的选项都可以在这里使用
  - content：用于替代'src'，可以直接设定文件的值
  - directory_mode：递归设定目录的权限，默认为系统默认权限
  - force：如果目标主机包含该文件，但内容不同，如果设置为yes，则强制覆盖，如果为no，则只有当前目标主机的目标位置不存在该文件时，才复制，默认为yes。

例子
```python
ansible test -m copy -a "content='hello world' dest=/tmp/tmp.txt" -k    # 在目录主机上创建tmp.txt文件，内容为hello world
```

## 3.4 command
在远程主机上执行命令。（默认模块）,参数
  - creates：文件名，当该文件存在时，则不执行后面的命令
  - chdir：执行命令前，先修改当前路径
  - removes：文件名，当文件不存在时，则不执行后面的命令
  - executable：切换shell来执行命令，该执行路径必须是一个绝对路径

## 3.5 shell
和command模块相同，参数也相同。command，shell，raw，script模块同属于commands类，都是用来执行命令，不同的是：
  - command模块执行的命令中不能包含：|，<,>,& 符号。
  - shell模块:可以执行远程服务器上的shell脚本文件，也支持 管道符等，脚本需要使用绝对路径。
  - raw模块和shell模块相同，可以执行任意命令就像在本机一样，相当于ssh之直接执行Linux命令，不会进入到ansible的模块子系统中。
  - script模块，用来执行一个shell脚本，其实将管理端的shell脚本copy到被管理端上然后执行，相当于 scp + shell 的组合。(需要在脚本所在目录下执行)

PS:官方建议使用command模块，如果需求不满足，可以使用raw模块

## 3.6 service
用于管理服务,参数
  - arguments: 给命令行提供一些选项
  - enable：是否开机启动 yes | no
  - name：服务名称
  - pattern：定义一个模式，如果通过status命令来查看服务的状态时，没有响应，就会通过ps指令在进程中根据该模式进行查找，如果匹配到，则认为该服务已经在运行，否则会认为未启动( ps aux | grep `pattern` )
  - runlevel：运行级别
  - sleep：如果执行了，restarted，则在 stop 和 start 之间沉睡几秒钟
  - state：started、stoped、restarted和reloaded，其中started和stoped是幂等的，也就是说如果服务已经停止，那么运行stoped不会执行任何操作

例子
```python
ansible test -m service -a 'name=nginx state=started enabled=yes' -k                   # 启动nginx进程，并设置为开机启动
ansible test -m service -a 'name=nginx state=stopped' -k                               # 关闭nginx进程
ansible test -m service -a 'name=network state=restarted arguments=eth0' -k            # 重启network进程，并传递eth0 作为参数，即：重启eth0网卡
ansible test -m service -a 'name=nginx pattern=/usr/local/nginx state=started' -k      # 如果无法使用 service nginx status查寻到nginx的状态，那么会使用 ps 来过滤 pattern指定的关键字，如果存在，则表示程序已经正常启动　
```

## 3.7 cron
用于管理定时任务(crontab 来操作),参数
  - backup: 对远程主机上的原计划内容修改之前做备份
  - cront_file: 如果指定该选项，则用该文件替换远程主机上的cron.d目录下的用户计划任务
  - day: 日(1-31,*,*/2,......)
  - hour: 小时(0-23,*,*/2,......)
  - minute: 分钟(0-59,*,*/2,......)
  - mouth: 月(1-12,*,*/2,......)
  - weekday: 周(0-7,*,......)
  - job: 要执行的任务，依赖于state=present
  - name: 该任务的描述，必选项。
  - special_time：指定什么时候执行（被触发），参数：reboot，yearly，annually，monthly，weekly，daily，hourly。
  - state：确认该任务计划是创建还是删除 Present(启用) | Absent(停用)
  - user：以哪个用户的身份执行

例子
```python
ansible test -m cron -a "hour=3 month=2 day=1 job='echo hello' name='test' state=present " -k  # 添加一条定时任务，* 3 1 2 * echo hello
ansible test -m cron -a 'name=test state=absent' -k                                            # 删除name为test的计划任务
```

## 3.8 filesystem
用于在块设备上创建文件系统。参数
  - dev：目标块设备
  - force：在一个已有的文件系统的设备上强制创建
  - fstype：文件系统类型
  - opts：传递给mkfs命令的选项

## 3.9 yum
使用yum包管理软件包,参数
  - config_file：yum的配置文件
  - disable_gpg_check: 关闭gpg_check
  - disablerepo: 不启用某个源
  - enablerepo：启用某个源
  - list：列出repo源
  - name：软件包的名称，可以为一个url路径，或者本地一个rpm包的路径
  - state：状态
    - 安装：
        - present
        - installed * 
        - latest
    - 删除：
        - absent 
        - removed *

例子
```python
ansible test -m yum -a 'name=httpd state=installed' -k     # yum 安装 httpd　　
```

## 3.10 user
用于管理用户（useradd,userdel） 也可以用command模块来执行shell命令创建和管理用户,参数
  - home：指定用户的家目录
  - createhome: 是否创建用户家目录 yes | no
  - groups：用户的属组
  - uid：用户的uid
  - password：用户的密码
  - name：用户的名称
  - system：是否是系统用户
  - remove：是否删除用户家目录
  - state：具体的操作， 删除/添加， present | absent
  - shell： 指定用户的shell　

## 3.11 synchronize
使用rsync同步文件,参数
  - archive：是否进行归档，默认为yes，相当于同时开启recursive,links,perms,times,owner,group -D等选项
  - checksum：是否校验和
  - copy_links：是否复制链接文件
  - delete：删除源中没有而目标存在的文件
  - dest_port: 对方用于接收的端口
  - dirs: 非递归传输目录
  - mode：模式，推和拉模式， push | pull ,默认为push(推)
  - src：同步的数据源的位置
  - rsync_opts: 指定rsync的选项，多个选项可以用逗号分隔
  - dest：目标文件
  - compress: 默认为yes，表示在文件同步过程中是否启用压缩

例子
```python
ansible test -m synchronize -a 'src=/etc/yum.repos.d/epel.repo dest=/tmp/epel.repo' -k                  # rsync 传输文件
ansible test -m synchronize -a 'src=/tmp/123/ dest=/tmp/456/ delete=yes' -k                             # 类似于 rsync --delete 
ansible test -m synchronize -a 'src=/tmp/123/ dest=/tmp/test/ rsync_opts="-avz,--exclude=passwd"' -k    # 同步文件，添加rsync的参数-avz，并且排除passwd文件
ansible test -m synchronize -a 'src=/tmp/test/abc.txt dest=/tmp/123/ mode=pull' -k                      # 把远程的文件，拉到本地的/tmp/123/目录下　　
```

## 3.12 mount
用于磁盘挂载相关操作,参数
  - fstype：挂载文件的类型
  - name：挂载点
  - opts：传递给mount命令的参数
  - src：要挂载的文件
  - state： 
    - present：只会在 fstab 中添加，不会操作磁盘
    - mounted：自动创建挂载点并挂载
    - unmounted：卸载
    - absent：只会在 fstab 中删除，不会操作磁盘

例子
```python
ansible test -m mount -a 'fstype=ext4 src=/dev/loop0 name=/aaa state=mounted' -k   # 挂在/dev/loop0 设备
```

## 3.12 template
模版，用于将模版文件渲染后，输出到远程主机上，模版文件一般以.j2为结尾，标识其是一个jinja2模版文件,参数
  - src：模版文件的路径
  - dest：拷贝到远程主机上的位置
  - mode：权限
  - attributes: 特殊权限 类似于 chattr
  - force：存在覆盖
  - group：属组
  - owner：属主

## 3.13 get_url
从互联网下载数据到本地，作用类似于Linux下的curl命令。参数
  - url：必传参数，文件的下载地址
  - dest：必传参数，文件保存的绝对路径
  - mode：文件的权限mode
  - othes：所有file模块里的选项都可以在这里使用
  - checksum：文件的校验码
  - headers：传递给下载服务器的 HTTP Headers
  - backup：如果本地已经存在同名的配置文件，先行备份
  - timeout：下载的超时时间

## 3.14 unarchive
用于解压文件，其作用类似于Linux下的tar命令，默认情况下unarchive的作用是将控制节点的压缩包拷贝到远程服务器上，然后进行解压,参数
  - remote_src: 用来表示需要解压的文件存在远程服务器中还是本地服务器中，默认为no，表示解压前先将控制节点上的文件复制到远程主机上中，然后再进行解压。
  - src：指定压缩文件的路径，该选项取决于remote_src的取值，如果remote_src取值为yes,则src指定的是远程服务器中压缩包的地址，如果remote_src取值为no，则src指向的是控制节点中的路径
  - dest：该选项指定的是远程服务器上的绝对路径，表示压缩文件解压的路径
  - list_files: 默认情况下该选项的取值为no，如果该选项取值为yes，也会解压缩文件，并且在ansible的返回值中列出压缩包里的文件；
  - exclude：解压文件时排出exclude选项指定的文件或目录列表
  - keep_newer: 默认值为False，如果该选项为True，那么当目标地址中存在同名文件，并且文件比压缩包中的文件更新时，不进行覆盖。
  - owner：文件或目录解压以后的所有者
  - group: 文件或目录解压以后所属的群组
  - mode: 文件或目录解压以后的权限

## 3.15 git
在远程服务器上执行git相关操作。依赖于git命令，需要在远程主机上先进性安装,参数
  - repo：远程git库的地址，可以是一个git协议，ssh协议或者http协议的git库地址
  - dest：必选参数，git库clone到本地服务器以后保存的绝对路径
  - version：克隆远程git库的版本，取值可以为HEAD，分支名称，tag的名称，也可以是一个commit的hash值
  - foces：默认为no，当该选项为yes时，如果本地git库有修改，将会抛弃本地的修改(强制覆盖)
  - accept_hostkey: 当该选项取值为yes时，如果git库服务器不再know_hosts中，则添加到know_hosts中，key_file指定克隆远程git库地址时使用的私钥。　

## 3.16 stat
用于获取远程服务器上的文件信息，类似于Linux下的stat命令,参数
  - path：用于指定文件或目录的路径　

## 3.17 sysctl
与Linux下的sysctl命令相似，用来控制Linux内核参数,参数：
  - name：需要设置的参数
  - value：需要设置的值
  - sysctl_file: sysctl.conf文件的绝对路径，默认路径为/etc/sysctl.conf
  - reload：该选项可以取值为 yes 或 no，默认为yes，用于表示设置完以后是否需要实行sysctl -p的操作

例子
```python
ansible test -m sysctl -a 'name=vm.overcommit_memory value=1'  # 修改vm.overcommit_memory的值为1
```

# 4 模块的返回值
Ansible通过模块来执行具体的操作，由于模块功能千差万别，所以执行模块操作后，Ansible会根据不同的需要返回不同的结果，Ansible中一些常见的返回值如下：
- changed：几乎所有的Ansible模块都会返回该变量，表示模块是否对远程主机进行了操作
- failed：如果模块未能执行完成，将返回failed和true
- msg：模块执行失败的原因，常见的错误如ssh连接失败，没有执行权限等
- rc：与命令行相关的模块会返会rc，表示执行Linux名的返回码
- stdout：与rc相似，返回的是标准输出的结果
- stderr：与rc详细，返回的是标准差错误输出
- backup_file：所有存在backup选项的模块，用来返回备份文件的路径
- results：应用在Playbook中存在循环的情况，返回多个结果

# 5 YAML简介
ansible中使用YAML基础元素：
- 变量
- Inventory
- 条件测试
- 迭代

## 5.1 变量
变量仅能由字母、数字、下划线组成，且只能以字母开头

### 5.1.1.facts
facts是由正在通信的远程目标主机返回的信息，这些信息被保存在ansible变量中，要获取指定的远程主机所所支持的facts，可以使用setup模块。

### 5.1.2 register(注册器)
把任务的输入定义为变量，然后用于其它任务
```python
task：
- shell：/usr/bin/foo
  register: foo_result
  ignore_errors: True

- name: test
  shell: echo ‘hello world'
  when: foo_result.success    # wen条件判断foo_result
```
### 5.1.3 通过命令行传递变量
在运行playbook的时候也可以传递一些变量提供playbook使用：
```python
ansible-playbook test.yaml --extra-vars 'hosts=www user=www'
4.通过roles传递变量
当给一个主机英语角色的时候，可以传递变量，然后在角色内使用这些变量

- host: webservers
  roles:
    - common                                                   # common 角色
    - {roles:foo_app_instance,dir:'/var/lib/html',port:8080}   # 传递变量参数
```

## 5.2 Inventory
ansible的主要功能在于批量主机操作，为了便捷地使用其中的部分主机，可以在Inventory file中讲其分组命名，默认的Inventory file为/etc/ansible/hosts。

### 5.2.1 文件格式
Inventory文件遵循INI文件风格，中括号中的字符表示组名。可以将同一个主机同时归并到多个不同的组中，此外，当如果目标主机使用了非默认的SSH端口，还可以在主机名称之后使用冒号加端口号来标明。
```python
192.168.1.1         # 直接列出主机

[web]
192.168.1.2
www.daxin.com

[web]
db[1:3].daxin.com   # 表示 db1.daxin.com db2.daxin.com db3.daxin.com

# 字母或者数字是连续的那么，可以使用列表的方式进行标识
```
### 5.2.2 主机变量
可以在 Inventory 中自定义主机时为其添加主机变量以便于在playbook中使用
```python
[nginx]
127.0.0.1 http_port=8080 user=daxin
```
PS：定义主机时传递的专有的变量，在playbook中可以直接调用

### 5.2.3 组变量
组变量是指赋予给指定组内所有主机上的在playbook中可用的变量。
```python
[nginx]
192.168.1.1
192.168.1.2

[nginx:vars]      # 固定用法                   
ntp_server = ntp1.daxin.com
nfs_server = nfs1.daxin.com　
```
### 5.2.4 组嵌套
inventory 中，组还可以包含其他组，并且也可以向组中的主机指定变量，不过，这些变量只能在ansible-playbook中使用，而ansible不支持。
```python
[nginx]
192.168.1.1

[apache]
192.168.1.2

[web:children]        # 引用其他组作为自己组内的主机(子组)
nginx                 # nginx组
apache                # apache组　
```
### 5.2.5 inenvtory 参数
ansible基于ssh连接inventory中指定的远程主机时，还可以通过参数指定其交互方式：
- ansible_ssh_user : 用于指定管理远程主机的帐号
- ansible_ssh_host : 用于指定被管理的主机
- ansible_ssh_port ：用于指定ssh的端口
- ansible_ssh_private_key_file ：指定key文件
- host_key_checking=False ：当第一次连接主机时，会提示yes/no，跳过此次环节
- ansible_ssh_pass : 用于ssh登录的密码
- ansible_sudo_pass : 用于用户sudo时的密码
- ansible_connection : 连接客户端的类型(local,ssh,paramiko), 1.2版本以前模式为paramiko，后面版本为ssh

## 5.3 条件测试
如果需要根据变量，facts或此前任务的执行结果来做为某task执行与否的前提时要用到条件测试。

when语句：在task后添加when子句即可使用条件测试：when语句支持Jinja2表达式语法
```python
tasks:
- name: 'shutdown redhat system'
  command: /sbin/shutdown -h now
  when: ansible_os_family == 'redhat'     # ansible_os_family 是 setup中收集到的信息，这里可以直接进行调用
```

## 5.4 迭代
当有需要重复性执行的任务时，可以使用迭代机制，其使用格式为将需要迭代的内容定义为item变量引用，并通过with_items语句来指明迭代的元素列表即可。
```python
tasks:
  - name: add user
    user: name={{ item }} state=present groups=wheel # 必须要用 {{item}} 来表示迭代。
    with_items:
      - daxin1
      - daxin2

# 等同于：

tasks:
  - name: add user
    user: name=daxin1 state=present groups=wheel
  - name: add user
    user: name=daxin2 state=present groups=wheel
```
事实上,with_items中还可以使用元素为hashes，例如：
```python
tasks:
  - name: add users
    user: name={{item.name}} state=parsent groups={{items.group}}
    with_items:
      - {name:'daxin',group='daxin'}
      - {name:'daxin2',group='daxin2'}
```
注意：with_items中的列表值也可以是字典，但引用时要使用item.key

# 6 playbook简介
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;playbook是由一个或多个play组成的列表。play的主要功能在于将事先归并为一组的主机装扮成通过ansible中的task定义好的角色。从根本上讲，所谓task无非是调用ansible的一个module。将多个play组织在一个playbook中，即可让他们联合起来按事先编排的机制完成某一任务。

>有一个形象的比如，Ansible中的模块类似于Linux下的命令，而Ansible的Playbook则类似于Linux下的shell脚本。Shell脚本将各个Linux命令组合起来，以此实现复杂的功能。

在Ansible中，一个play必须包含两部分内容，hosts，tasks，play配置文件例如：
```python
[root@linux-node2 ~]# cat httpd.yaml 
--- #yaml的默认开头格式
- hosts: test 
  user: root
  vars: 
    - info: 'hello world'
  tasks:
    - name: setup a infomation # task名称
      copy: dest=/tmp/info content="{{info}}"
      notify: say something 
  handlers:
    - name: say something # handlers名称
      command: echo 'copy ok'
```

## 6.1 playbook的一般结构
一个playbook针对配置区域划分，主要分为4类：
- target section：定义将要执行playbook的远程主机组
- variable section：定义playbook运行时需要使用的变量
- task section：定义将要在远程主机上执行的任务列表
- handler section：定义task执行完以后需要调用的任务

细化一点的话playbook的结构为：
```python
Inventory：操作的主机 *
Modules：使用的模块 *
Ad Hoc Commands：在主机上指定的命令
Playbooks 
  Tasks：任务，即调用模块完成的某操作 *
  Variables：变量 
  Templates：模板
  Handlers：处理器，由某事件触发完成的操作
  Roles：角色
```
## 6.2 target 配置项目
- hosts：定义远程的主机组
- user：执行该任务的用户
- remote_user: 远端运行该play的用户，可以细分到每一个task
- *sudo：如果设置为yes，执行该任务组的用户在执行任务的时候，获取root权限
- *sudo_user: 如果你设置user为tom，速度设置为yes，sudo_user为jerry，则tom用户则会获取jerry用户的权限
- connection：通过什么方式连接到远程主机，默认为ssh
- become：是否提权
- become_method： 提权得方式(sudo)
- notify：当任务执行完毕后，通知handler section 中的任务模块(出发执行)
- set_fact: 模块可以自定义facts，这些自定义的facts可以通过template或者变量的方式在playbook中使用。如果你想要获取一个进程使用的内存的百分比，则必须通过set_fact来进行计算之后得出其值，并将其值在playbook中引用。set_fact: innodb_buffer_pool_size_mb="{{ ansible_memtotal_mb / 2 }}"
- gather_facts：除非你明确说明不需要在远程主机上执行setup模块，否则会默认自动执行。如果你的确不需要setup模块所传递过来的变量，你可以启用该选项

注意：
- 可以使用setup获取到的字典中的key，来渲染对应的值
- 当值的类型为字典嵌套时，可以使用点来进行深入查找 {{ansible_eth0.ipv4.address}}
- 当值的类型为列表时，{{ansible[0].xxx}}　　

## 6.3 variable 可用的配置项目
- vars：定义一个变量。key：value，多个变量，可以使用{key:value,key2:value2}
- vars_files：定义一个变量文件。这里列出变量文件的绝对路径即可。 变量文件中的直接使用key:value定义即可。
- vars_prompt: 通过交互的方式输入的值
- name：变量的值
- prompt：提示信息
- private：是否为私有变量，yes | no， yes时输入的变量不在屏幕上显示，否则会显示在屏幕上

## 6.4 task 配置项目
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;play的主体部分是task list,task list中的各任务被按次序诸葛在hosts中 指定的所有主机上运行,即在所有主机上完成第一个任务后再开始第二个.在运行自上而下某个playbook时,如果中途发生错误,所有已执行任务都可以能回滚,因此,在更正playbook后重新执行一次即可.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;task的目的是使用指定的参数运行模块,而在模块参数中可以使用变量,模块执行是幂等的,这意味着多次执行是安全的,因为其结果一致.

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;每个task都应该有其name,用于playbook的执行结果输出，建议其内容尽可能清晰的描述任务执行步骤。如果未提供name，则action的结果将用于输出。

定义任务的三种方式：

1. 方式1(action)
```python
- name: task的名称
  action：要做的操作
# 例如： action: yum name= httpd state=installed
```
2. 方式2(直接写模块名)
```python
- name: task的名称
  copy: src=files/httpd.conf dest=/etc/httpd/conf/httpd.conf
```
3. 方式3(模块名时，模块的参数也可以用下列方式使用)
```python
- name: task的名称
  service:
    name:httpd
    state:restarted
```
在众多模块中只有command和shel模块仅需要给定一个列表而无需使用key=value格式
```python
tasks:
- name: disable selinux
  command: /sbin/setenforce 0
```
如果命令或脚本的输出码不以0，可以使用如下方式替代
```python
tasks:
- name: run this command and ignore the result
  shell: /usr/bin/somecommand || /bin/true     # 强行成功
```
或者使用ignore_errors来忽略错误信息
```python
tasks:
  - name: run this command and ignore the result
    shell: /usr/bin/somecommand
    ignote_errors:True
```
常用参数：
- ignote_errors： 是否忽略错误
- register： 注册变量，即把当前任务的结果注册到register指定的变量后，在后续的task中可以进行调用，或者判断
- when：当when后面的条件为真时才会执行当前task，一般配合register来做,当然，when也可以读取变量

>when支持多条件
```python
# 1、当多个条件是列表时，它们之间是并且的关系
when:
- ansible_distribution == "Centos"
- ansible_distribution == "ubuntu"

# 2、如果要表达更复杂的关系，可以用and/or, 
when: (ansible_distribution == "Centos" and ansible_distribution_major_version == "6" ) or (ansible_distribution == "ubuntu")
```
## 6.5 handler 配置项目
用于当关注的医院发生变化时采取一定的操作
- Handlers 也是一些 task 的列表,通过名字来引用,它们和一般的 task 并没有什么区别。
- Handlers 是由通知者进行 notify, 如果没有被 notify,handlers 不会执行。
- 不管有多少个通知者进行了 notify,等到 play 中的所有 task 执行完成之后,handlers 也只会被执行一次。
- Handlers 最佳的应用场景是用来重启服务,或者触发系统重启操作.除此以外很少用到了。

例子：
```python
tasks:
  - name: template configuration file
    template: src=template.j2 dest=/etc/configuration.conf
    notify:
      - restart memcached

handlers:
  - name: restart memcached
    service: name=memcached state=restart
```

## 6.6 ansible-playbook命令
playbook需要使用ansible-playbook来运行，它具有ansible的部分参数，以及自己独有的参数：
- --list-tasks：列出playbook的任务列表。
- --step：每执行一个任务后停止，等待用户确认。(yes/no/continue,yes会继续下一个task，continue会执行完当前play，下一个play再次等待用户输入)
- --syntax-check：检查playbook的语法
- -C --check：检查当前这个playbook是否会修改远程服务器，相当于预测playbook的执行结果

## 6.7 其他配置
`tags`: 在playbook中，可以为某个任务或某些任务定义一个“标签”，在执行此playbook时，通过为ansible-playbook 使用 --tags选项能实现仅指定的tasks，而非所有的。
```python
tasks:
  - name: user add
    user: name='daxin' state=parsent
  - name: config file
    file: src=/tmp/daxin.txt dest=/tmp/daxin.txt
    tags:
    - copyfile

# 运行时
ansible-playbook test.yaml --tags='copyfile'
```
`include`：在playbook中，一般情况下，一个yaml文件包含一个play，比如我要同时安装db.yaml 和 web.yaml，那么我们可以用include的方式来进行聚合
```python
cat all.yaml
---
- include:db.yaml
- include:web.yaml　　
```
这时只需要执行all.yaml即可。 ansible all all.yaml

`strategy`: free,任务策略
默认情况下，ansible是按照顺序执行task的，当匹配到的所有主机执行完task1后，再继续指定task2。
从Ansible 2.0开始，ansible支持free的任务执行策略，允许较快的远程服务器提前完成play的部署，不用等待其他远程服务器一起执行task。

更多知识： https://github.com/lorin/ansible-quickref

# 7 Roles
ansible自1.2版本引入的新特性，用于层次性、结构化地组织playbook，roles能够根据层次型结构自动装载变量文件、tasks以及handles等。要使用roles，只需要在playbook中使用include指令即可。简单来讲，roles就是通过分别将变量、文件、任务、模块及处理器放置于单独的目录中，并可以便捷的通过include它们的一种机制。角色一般用于基于主机构建服务的场景中，但也可以是用于构建守护进程等场景中。

创建roles的步骤
- 创建以roles命名的目录
- 在roles目录中分别创建以各角色名称命名的目录
- 在每个角色命名的目录中，分别创建files，handlers，meta，tasks，templates和vars目录；用不到的可以创建为空目录，或者不创建
- 在playbook文件中，调用各角色

## 7.1 role内各目录中的文件说明
1. tasks目录：至少应该包含一个main.yml的文件，其定义了此角色的任务列表；此文件可以使用include包含其他的位于此目录的task文件
2. files目录：存放由copy或script等模块调用的文件
3. templates目录：templates模块会自动在此目录中寻找Jinja2模版文件
4. handlers目录：此目录中应该包含一个main.yml
5. yml文件，用于定义此角色用到的各handler，在handler中使用include包含的其他handler文件也应该位于此目录中
6. vars目录：应当包含一个main.yml文件，用于定义此角色用到的变量
7. meta目录：应当包含一个main.yml文件,用于定于此角色的特殊设定及依赖关系,ansible1.3 以后才支持
8. default目录: 为当前角色设定默认变量时使用此目录，应当包含一个main.yml文件

例子：
```python
# 目录结构：
├── db.yaml
└── roles
└── db
├── files
├── handler
├── tasks
│   └── main.yml
├── templates
│   └── my.cnf.j2
└── vars
└── main.yml

# db.yaml 文件
- hosts: test
  roles:
    - db

# tasks.main.yml 文件
---
- name: Install MySQL
  yum:
    name: mariadb-server
    state: present

- name: copy File
  template:
    src: "my.cnf.j2"
    dest: "/etc/my.cnf"

- name: Start MySQL
  service:
    name: mariadb
    state: started

# vars.main.yml 文件
---
mysql_port: 3307

# templates.my.cnf.j2 文件
[mysqld]
datadir=/var/lib/mysql
socket=/var/lib/mysql/mysql.sock
user=mysql
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
port = {{ mysql_port }}
bind = {{ ansible_all_ipv4_addresses[0] }}

[mysqld_safe]
log-error=/var/log/mariadb/mariadb.log
pid-file=/var/run/mariadb/mariadb.pid

# 运行
ansible-playbook -i hosts db.yaml
```

## 7.2 ansible-galaxy 命令
我们可以利用ansible-galaxy来快速创建并初始化一个roles
```python
ansible-galaxy init roles/mysql
```
我们在galaxy.ansible.com 上找到适合自己的roles时，只需要运行 ansible-galaxy install 作者id.角色包名称即可完成安装
```python
# 安装别人写好的roles：(galaxy.ansible.com)
ansible-galaxy install -p /etc/ansible/roles bennojoy.mysql
# 安装到指定目录
ansible-galaxy -p ./roles install bennojoy.ntp
```
默认情况下role存放在/etc/ansible/roles目录，也可以在ansible.cfg 中 使用 roles_path ，来制定roles的目录

列出已经安装的roles：
```python
ansible-galaxy list
```
查看已安装的roles信息：
```python
ansible-galaxy info bennojoy.mysql
```
卸载roles:
```python
ansible-galaxy remove bennojoy.mysql
```