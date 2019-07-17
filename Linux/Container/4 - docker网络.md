brctl show

docker 网络：
docker 四种网络类型：
- bridge(Bridged Container): NAT桥模式(默认)
- host(Open Container): 与当前物理机共享网络资源(可以直接管理物理机网卡配置),容器使用宿主机的网络资源
- none(Closed container): 以为这容器没有网络，只有lo接口(新买一台主机，没有网卡)
- 联盟网络(joined Container): 多个容器公用一组资源(NET、IPC、)

docker network inspect web 

ip命令管理网络名称空间
ip netns: 用于管理网络名称空间(模拟容器间通信)

- ip netns list:显示网络名称空间
- ip netns add NAME：添加一个网络名称空间
- ip netns set NAME NETNSID：设置一个网络名称名称空间的SID
- ip [-all] netns exec [NAME] cm d ...：在一个网络名称空间中执行命令
- ip link：挪动网络设备到容器中去

当我们使用ip去管理网络名称空间时，只有网络名称空间是被隔离的，其他名称空间都是共享的

例子：
创建网络名称空间：
- ip netns add r1
- ip netns add r2 

添加虚拟网卡：(一对)
- ip link add name veth1.1 type veth peer name veth1.2
- ip link show：会看到创建的一对儿的网卡

移动虚拟网卡到网络命名空间中去：
- ip link set dev veth1.2 netns r1 
- ip netns exec r1 ip a:验证

修改网络名称空间中的网卡名称：
- ip netns exec r1 ip link set dev veth1.2 name eth0

设置主机的veth的网卡的地址
- ip addr add 192.168.0.2/24 dev veth1.1
- ip link set veth1.1 up

设置网络名称空间中的veth的网卡地址
-  ip netns exec r1 ip addr add 192.168.0.3/24 dev eth0
- ip nets exec r1 ip link set dev eth0 up

把创建的veth1.1移动到r2中，并设置IP地址，那么两个网络名称空间就可以通信了
- ip link set dev veth1.1 netns r1
- ip netns exec r1 ip addr add 192.168.0.3/24 dev veth1.1
- ip netns exec r1 ping 192.168.0.2


启动容器时指定容器的网络模型
- docker run -it --rm --name t1 --network bridge/none/host busybox:latest 

指定容器的主机名，默认时，容器的主机名为它的容器ID
- docker run -it --rm --name t1 -h t1 --network none busybox:latest
- 自动生成hosts文件中的本地解析
- 自动将宿主机设置为nameserver(未浮现)

设置自定义的dns,指定dns的搜索域
- docker run -it --rm --hostname t1 --network bridge --dns 114.114.114.114 busybox:latest
- docker run -it --rm --hostname t1@daxin.com --network bridge --dns-search ilinux.io busybox:latest 
- docker run -it --rm --name t1 --hostname t1@daxin.com --network bridge --dns 114.114.114.114 --dns 223.5.5.5 --dns-search ilinux.io busybox:latest

hosts文件解析记录
- docker run -it --rm --name t1 --hostname t1@daxin.com --network bridge --dns 114.114.114.114 --dns 223.5.5.5 --dns-search ilinux.io --add-host www.baidu.com:1.1.1.1 busybox:latest