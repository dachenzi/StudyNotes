## docker daemon启用tcp端口异常

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;默认情况下docker daemon监听的是本地的sock的，它的位置一般为/var/run/docker.sock，其实docker也是支持监听tcp端口的，今天在修改监听方式时，遇到如下问题：

>OS:Centos 7.6 Docker:18.04

我的修改方式为，在/etc/docker/daemon.json中添加如下配置
```json
{
    "hosts":["tcp://0.0.0.0:2375","unix:///var/run/docker.sock"]
}
```
启动docker时，一直启动失败，提示如下错误
```java

7月 19 22:52:34 localhost.localdomain dockerd[28536]: unable to configure the Docker daemon with file /etc/docker/daemon.json: the following dir
ectives are specified both as a flag and in the configuration file: hosts: (from flag: [fd://], from file: [unix:///var/run/docker.sock tcp://12
7.0.0.1:2375])
7月 19 22:52:34 localhost.localdomain systemd[1]: docker.service: main process exited, code=exited, status=1/FAILURE
7月 19 22:52:34 localhost.localdomain systemd[1]: Failed to start Docker Application Container Engine.
-- Subject: Unit docker.service has failed
-- Defined-By: systemd
-- Support: http://lists.freedesktop.org/mailman/listinfo/systemd-devel
--
-- Unit docker.service has failed.
```

看日志提示hosts和fd://的配置冲突，但是我本身没有进行fd相关的配置，仔细阅读了官方文档，发现有说明：https://docs.docker.com/config/daemon/ 从 `USE THE HOSTS KEY IN DAEMON.JSON WITH SYSTEM` 部分得知
- 使用systemd管理方式，启动docker时，会默认添加-H参数
- 这个参数与配置文件中的hosts关键字冲突


解决办法：

创建/修改 `/etc/systemd/system/docker.service.d/docker.conf` 文件，添加如下内容
```sh
[Service]
ExecStart=
ExecStart=/usr/bin/dockerd
```

或者直接执行`systemctl edit docker`，插入上述内容也可，重启docker服务，正常监听