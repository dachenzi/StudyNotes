# 1 虚拟化介绍
主流的虚拟机化方式：
- `主机级虚拟化`：虚拟化的是整个完整的物理硬件平台(VMware workstation)
- `容器级虚拟化`：虚拟化的是一个个的用户空间

主机虚拟化两种类型的实现：
- `Type-I`:直接在硬件设备上安装虚拟机管理器一般叫hypervisor，然后再来创建虚拟机
- `Type-II`:在宿主机上安装一个宿主机操作系统，再装一个VMM(viurtul Machine Manage) 虚拟机管理器，然后再来创建虚拟机（WMware、virtualbox等) 


没有一个虚拟机是直接跑在硬件设备之上的。所以减少中间层，中间环境，就可以有效的提高效率。  容器技术

# 2 容器技术

每一个用户空间都应该存在：
- 主机名/域名（UTS）
- 文件系统（Mount） - 树形结构
- 进程间通讯（IPC） - 
- 进程(PID) - 属性结构（归属与init或者本身就是init)
- 用户(User)
- 网络(Network)

Linux在内合层面通过使用名称空间(namespace)对以上六种资源进行的隔离进行直接支持，并通过系统调用直接输出。 
下面六大资源支持的内核版本

|namespace|系统调用参数|隔离内容|内核版本|
|---|---|---|---|
|UTS|CLONE_NEWUTS|主机和域名|2.6.19|
|IPC|CLONE_NEWIPC|信号量、消息队列和共享内存|2.6.19|
|PID|CLONE_NEWPID|进程编号|2.6.24|
|Network|CLONE_NEWNET|网络设备、网络栈、端口等|2.6.29|
|Mount|CLONE_NEWNS|挂在点(文件系统)|2.4.19|
|User|CLONE_NEWUSER|用户和用户组|3.8|
