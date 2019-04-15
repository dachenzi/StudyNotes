# 1 简介
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;ssh是一个协议，OpenSSH是其中一个开源实现，paramiko是Python的一个库，支持Python 2.6+ 和Python 3.3+ 版本，实现了SSHv2协议(底层使用cryptography)。有了Paramiko以后，我们就可以在Python代码中直接使用SSH协议对远程服务器执行操作，而不是通过ssh命令对远程服务器进行操作。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于paramiko属于第三方库，所以需要使用如下命令先行安装
```python
pip3 install paramiko
 
# 测试
python3 -c 'import paramiko'
```
# 2 Paramiko介绍
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;paramiko包含两个核心组件：SSHClient和SFTPClient。
- SSHClient的作用类似于Linux的ssh命令，是对SSH会话的封装，该类封装了传输(Transport)，通道(Channel)及SFTPClient建立的方法(open_sftp)，通常用于执行远程命令。
- SFTPClient的作用类似与Linux的sftp命令，是对SFTP客户端的封装，用以实现远程文件操作，如文件上传、下载、修改文件权限等操作。

## 2.1 Paramiko中的几个基础名词：
1. Channel：是一种类Socket，一种安全的SSH传输通道；
2. transport：是一种加密的会话，使用时会同步创建了一个加密的Tunnels(通道)，这个Tunnels叫做Channel；
3. Session：是client与Server保持连接的对象，用connect()/start_client()/start_server()开始会话。


# 3 Paramiko的基本使用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;基于常用的两个核心组件进行说明。

## 3.1 SSHClient介绍
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面是一个使用SSHClient连接服务器并执行命令的操作步骤：（支持密码认证和密钥认证）
```python
import paramiko
 
#实例化SSHClient
client = paramiko.SSHClient()
 
#自动添加策略，保存服务器的主机名和密钥信息，如果不添加，那么不再本地know_hosts文件中记录的主机将无法连接
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
 
#连接SSH服务端，以用户名和密码进行认证
client.connect(hostname='10.0.0.3',port=22,username='root',password='123456')

#打开一个Channel并执行命令
stdin,stdout,stderr = client.exec_command('df -h ')      # stdout 为正确输出，stderr为错误输出，同时只有1个变量有值
 
#打印执行结果
print(stdout.read().decode('utf-8'))
 
#关闭SSHClient
client.close()
```
SSHClient使用私钥连接:
```python
import paramiko

# 配置私人密钥文件位置
private = paramiko.RSAKey.from_private_key_file('/Users/DahlHin/.ssh/id_rsa')

#实例化SSHClient
client = paramiko.SSHClient()

#自动添加策略，保存服务器的主机名和密钥信息，如果不添加，那么不再本地know_hosts文件中记录的主机将无法连接
client.set_missing_host_key_policy(paramiko.AutoAddPolicy())

#连接SSH服务端，以用户名和密码进行认证
client.connect(hostname='10.0.0.3',port=22,username='root',pkey=private)

#打开一个Channel并执行命令
stdin,stdout,stderr = client.exec_command('df -h ')      # stdout 为正确输出，stderr为错误输出，同时是有1个变量有值

#打印执行结果
print(stdout.read().decode('utf-8'))

#关闭SSHClient
client.close()

# 在登陆的时候传递还可以利用key_filename直接来指定私钥的位置比如
client.connect(IP,Port,Username,key_filename='~/.ssh/id_rsa')
```

### 3.1.1 常用的方法
`connect()`：实现远程服务器的连接与认证，对于该方法只有hostname是必传参数。
```python
connect(self,hostname,port=SSH_PORT,username=None,password=None,
        pkey=None,key_filename=None,timeout=None,allow_agent=True,
        look_for_keys=True,compress=False,sock=None,gss_auth=False,gss_kex=False,
        gss_deleg_creds=True,gss_host=None,banner_timeout=None,auth_timeout=None,)
```
`set_missing_host_key_policy()`：设置远程服务器没有在know_hosts文件中记录时的应对策略。目前支持三种策略：
`AutoAddPolicy`：自动添加服务器到know_hosts文件中
`RejectPolicy(默认策略)`：拒绝本次连接
`WarningPolicy`：警告并将服务器添加到know_hosts文件中
`exec_command()`：在远程服务器执行Linux命令的方法。
`open_sftp()`：在当前ssh会话的基础上创建一个sftp会话。该方法会返回一个SFTPClient对象。

```python
# 利用SSHClient对象的open_sftp()方法，可以直接返回一个基于当前连接的sftp对象，可以进行文件的上传等操作.
sftp = client.open_sftp()
sftp.put('test.txt','text.txt')
```

## 3.2 SFTPClient介绍
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;下面是一个使用SFTPClient连接服务器并传输文件的操作步骤：（支持密码认证和密钥认证）
```python
import paramiko
 
# 获取Transport实例
tran = paramiko.Transport(('10.0.0.3',22))
 
# 连接SSH服务端，使用pkey指定私钥
tran.connect(username = "root", password='123456)
 
# 获取SFTP实例
sftp = paramiko.SFTPClient.from_transport(tran)
 
# 设置上传的本地/远程文件路径
localpath="/Users/DahlHin/Downloads/daxin.txt"
remotepath="/tmp/daxin.txt"
 
# 执行上传动作
sftp.put(localpath,remotepath)
 
tran.close()
```
配置SFTPClient使用密钥
```python
import paramiko

# 配置私人密钥文件位置
private = paramiko.RSAKey.from_private_key_file('/Users/DahlHin/.ssh/id_rsa')

# 获取Transport实例
tran = paramiko.Transport(('10.0.0.3',22))

# 连接SSH服务端，使用pkey指定私钥
tran.connect(username = "root", pkey=private)

# 获取SFTP实例
sftp = paramiko.SFTPClient.from_transport(tran)

# 设置上传的本地/远程文件路径
localpath="/Users/DahlHin/Downloads/daxin.txt"
remotepath="/tmp/daxin.txt"

# 执行上传动作
sftp.put(localpath,remotepath)

tran.close()
```
### 3.2.1 常用的方法
`put()`：上传本地文件到远程服务器
`get()`：从远程服务器下载文件到本地
`mkdir()`：在远程服务器上创建目录
`remove()`：删除远程服务器中的文件
`rmdir()`：删除远程服务器中的目录
`rename()`：重命名远程服务器中的文件或目录
`stat()`：获取远程服务器中文件的详细信息
`listdir()`：列出远程服务器中指定目录下的内容

### 3.2.2 使用paramiko部署监控程序
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;现有如下需求：在所有的主机上部署monitor.py监控程序。
```python
import paramiko
from paramiko.ssh_exception import AuthenticationException
 
def deploy_monitor(ip):
 
    with paramiko.SSHClient() as client:     # 利用上下文管理的方式，可以不用显示的进行链接的关闭
        try:
            client.set_missing_host_key_policy(paramiko.AutoAddPolicy())
            client.connect(ip,22,'root','......')
 
            stdin,stdout,stderr = client.exec_command('ls -l')
            print(stdout.read())
 
            with client.open_sftp() as sftp:
                sftp.put('sendmail.py','sendmail.py')
                sftp.chmod('sendmail.py',0o755)      # 注意这里的权限是八进制的，八进制需要使用0o作为前缀
        except AuthenticationException as e:
            print('用户名或者密码不正确')
 
 
def main():
    ip = 'x.x.x.x'   # 多IP的情况可以读取文件中的IP列表,利用for循环或者配合多线程就可以完成分发。
    deploy_monitor(ip)
 
 
if __name__ == '__main__':
    main()
```

# 4 新版ssh-keygent生成的key的问题
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;现在使用命令 ssh-keygen -t rsa  生成ssh，默认是以新的格式生成，id_rsa的第一行变成了“BEGIN OPENSSH PRIVATE KEY” 而不在是“BEGIN RSA PRIVATE KEY”，此时用来msyql、MongoDB，配置ssh登陆的话，可能会报 “Resource temporarily unavailable. Authentication by key (/Users/youname/.ssh/id_rsa) failed (Error -16). (Error #35)” 提示资源不可用，这就是id_rsa 格式不对造成的。`Paramiko也会认为这个key，不是一个合法的RSA密钥`。

## 4.1 解决方法
使用 ssh-keygen -m PEM -t rsa -b 4096 来生成
- -m 参数指定密钥的格式，PEM（也就是RSA格式）是之前使用的旧格式
- -b：指定密钥长度；
- -e：读取openssh的私钥或者公钥文件；
- -C：添加注释；
- -f：指定用来保存密钥的文件名；
- -i：读取未加密的ssh-v2兼容的私钥/公钥文件，然后在标准输出设备上显示openssh兼容的私钥/公钥；
- -l：显示公钥文件的指纹数据；
- -N：提供一个新密语；
- -P：提供（旧）密语；
- -q：静默模式；
- -t：指定要创建的密钥类型

