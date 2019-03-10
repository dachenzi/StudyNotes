<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 socket编程弊端](#1-socket编程弊端)
- [2 SocketServer模块](#2-socketserver模块)
    - [2.1 服务器类](#21-服务器类)
    - [2.2 Mixin类](#22-mixin类)
    - [2.3 RequestHandlerClass是啥](#23-requesthandlerclass是啥)
    - [2.4 编程接口](#24-编程接口)
- [3 实现EchoServer](#3-实现echoserver)
- [4 聊天室](#4-聊天室)

<!-- /TOC -->

# 1 socket编程弊端
socket编程过于底层，编程虽然有套路，但是要写出健壮的代码还是比较困难的，所以很多语言都会socket底层API进行封装，Python的封装就是SocketServer模块。它是网络服务编程框架，便于企业级快速开发。

# 2 SocketServer模块
SocketServer，网络通信服务器，是Python标准库中的一个高级模块，其作用是创建网络服务器。该模块简化了编写网络服务器的任务。

## 2.1 服务器类
SocketServer模块中定义了五种服务器类。
- BaseServer(server_address, RequestHandlerClass)：服务器的基类，定义了API。
- TCPServer(server_address, RequestHandlerClass, bind_and_activate=True)：使用TCP/IP套接字。
- UDPServer：使用数据报套接字
- UnixStreamServer：使用UNIX套接字，只适用UNIX平台
- UnixDatagramServer：使用UNIX套接字，只适用UNIX平台

它们的继承关系：
```bash
        +------------+
        | BaseServer |
        +------------+
              |
              v
        +-----------+        +------------------+
        | TCPServer |------->| UnixStreamServer |
        +-----------+        +------------------+
              |
              v
        +-----------+        +--------------------+
        | UDPServer |------->| UnixDatagramServer |
        +-----------+        +--------------------+
```
> 除了基类为抽象基类意外，其他四个类都是同步阻塞的。

## 2.2 Mixin类
SocketServer模块中还提供了一些Mixin类，用于和同步类组成多线程或者多进程的异步方式。
- class ForkingUDPServer(ForkingMixIn, UDPServer)
- class ForkingTCPServer(ForkingMixIn, TCPServer)
- class ThreadingUDPServer(ThreadingMixIn, UDPServer)
- class ThreadingTCPServer(ThreadingMixIn, TCPServer)
- class ThreadingUnixStreamServer(ThreadingMixIn, UnixStreamServer)
- class ThreadingUnixDatagramServer(ThreadingMixIn, UnixDatagramServer)

> fork是创建多进程，thread是创建多线程。fork需要操作系统支持，Windows不支持。

## 2.3 RequestHandlerClass是啥
要想使用服务器类，需要传入一个RequestHandlerClass类，为什么要传入这个RequestHandlerClass类，它是干什么的？下面一起来看看源码：
1. TCPServer入口
```python
# 446 行 
def __init__(self, server_address, RequestHandlerClass, bind_and_activate=True):
    """Constructor.  May be extended, do not override."""
    BaseServer.__init__(self, server_address, RequestHandlerClass)    # 把RequestHandlerClass交给了BaseServer
    self.socket = socket.socket(self.address_family,
                                self.socket_type)
```
2. BaseServer
```python
# 204 行
def __init__(self, server_address, RequestHandlerClass):
    """Constructor.  May be extended, do not override."""
    self.server_address = server_address
    self.RequestHandlerClass = RequestHandlerClass  # 将RequestHandlerClass当作类属性付给了实例本身
    self.__is_shut_down = threading.Event()
    self.__shutdown_request = False
```
3. 开启的服务serve_forever方法
```python
# 219 行
def serve_forever(self, poll_interval=0.5):
    self.__is_shut_down.clear()
    try:
        ... ...
                if ready:
                    self._handle_request_noblock()   # 这里执行了_handle_request_noblock()方法

                self.service_actions()
    finally:
        self.__shutdown_request = False
        self.__is_shut_down.set()
```
4. _handle_request_noblock()方法
```python
# 304行
    def _handle_request_noblock(self):
        ... ... 
            try:
                self.process_request(request, client_address)    # 这里又调用了process_request方法
            except Exception:
                self.handle_error(request, client_address)
                self.shutdown_request(request)
            except:
                self.shutdown_request(request)
                raise
        else:
            self.shutdown_request(request)
```
5. process_request方法
```python
# 342 行
def process_request(self, request, client_address):
    self.finish_request(request, client_address)   # 这里又调用了finish_request方法
    self.shutdown_request(request)
```

6. finish_request方法
```python
# 359 行
def finish_request(self, request, client_address):
    """Finish one request by instantiating RequestHandlerClass."""
    self.RequestHandlerClass(request, client_address, self)   # 实例化对象
```
通过观察源码，我们知道RequestHandlerClass接受了两个参数，根据上下文代码我们可知：
1. request：客户端的socket连接
2. client_address: 客户端的地址二元组信息

那么这个RequestHandlerClass到底是啥？
## 2.4 编程接口
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;通过源码，我们可以知道，最后传入RequestHandlerClass的是socket和client的ip地址，是不是和我们前面写的TCP多线程版本的recv函数很想？没错，RequestHandlerClass在这里被称为编程接口，由于处理业务逻辑。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;但是RequestHandlerClass我们没有参照，该如何写呢？ socketserver提供了抽象基类BaseRequestHandler，共我们继承来实现。
> `BaseRequestHandler 是和用户连接的用户请求处理类的基类，所以 RequestHandlerClass 必须是 BaseRequestHandler 的子类，`。
```python
class BaseRequestHandler:

    # 在初始化时就会调用这些方法
    def __init__(self, request, client_address, server):
        self.request = request
        self.client_address = client_address
        self.server = server
        self.setup()
        try:
            self.handle()
        finally:
            self.finish()

    def setup(self):    # 每一个连接初始化
        pass

    def handle(self):   # 每一次请求处理
        pass 

    def finish(self):   # 每一个连接清理
        pass
```
观察源码我们知道，setup、handler、finish在类初始化时就会被执行：
1. setup()：执行处理请求之前的初始化相关的各种工作。默认不会做任何事。
2. handler()：执行那些所有与处理请求相关的工作。默认也不会做任何事。
3. finish()：执行当处理完请求后的清理工作，默认不会做任何事。

这些参数到底是什么？
```python
import socketserver

class MyRequestHandler(socketserver.BaseRequestHandler):
    def handle(self):
        print(self.server)  # <socketserver.TCPServer object at 0x00000153270DA320>
        print(self.client_address)  # ('127.0.0.1', 62124)
        print(self.request)  # <socket.socket fd=296, family=AddressFamily.AF_INET, type=SocketKind.SOCK_STREAM, proto=0, laddr=('127.0.0.1', 999), raddr=('127.0.0.1', 62124)>

s = socketserver.TCPServer(('127.0.0.1', 999), MyRequestHandler)
s.serve_forever()
```
根据上面的输出信息我们知道：
- self.server:指代当前server本身
- self.client_address:客户端地址
- self.request: 和客户端连接的socket对象

我们总结一下，`使用socketserver创建一个服务器`需要以下几个步骤：
1. __从BaseRequestHandler类派生出子类，并覆盖其handler()方法来创建请求处理程序类，此方法将处理传入请求。__
2. __实例化一个服务器类，传参服务器的地址和请求处理类__
3. __调用服务器实例的handler_request()或serve_forever()方法(启动服务)__
4. __调用server_close()方法(关闭服务)__

# 3 实现EchoServer
顾名思义：Echo，即来什么消息就回显什么消息，即客户端发来什么消息，就返回什么消息。
```python
import socketserver
import logging
import threading

FORMAT = '%(asctime)s %(message)s'
logging.basicConfig(level=logging.INFO, format=FORMAT)

class MyRequestHandler(socketserver.BaseRequestHandler):

    def setup(self):
        
        # 优先执行父类同名方法是一个很好的习惯，即便是父类不作任何处理
        super(MyRequestHandler, self).setup() 
        self.event = threading.Event()

    def handle(self):
        super(MyRequestHandler, self).handle()
        while not self.event.is_set():
            data = self.request.recv(1024)
            if data == b'quit' or data == b'':
                self.event.set()
                break

            msg = '{}:{} {}'.format(*self.client_address, data.decode())
            logging.info(msg)
            self.request.send(msg.encode())

    def finish(self):
        super(MyRequestHandler, self).finish()
        self.event.set()

if __name__ == '__main__':
    with socketserver.ThreadingTCPServer(('127.0.0.1', 9999), MyRequestHandler) as server:

        # 让所有启动的线程都为daemon进程
        server.daemon_threads = True

        # serve_foreve会阻塞，所以交给子线程运行
        threading.Thread(target=server.serve_forever, daemon=True).start()
        while True:
            cmd = input('>>>').strip()
            if not cmd: continue
            if cmd == 'quit':
                server.server_close()
                break
            print(threading.enumerate())

```
ThreadingTCPServer是TCPServer的子类，它混合了ThreadingMixIn类使用多线程来接受客户端的连接。

# 4 聊天室
代码如下：
```python
import socketserver
import logging
import threading

FORMAT = '%(asctime)s %(message)s'
logging.basicConfig(level=logging.INFO, format=FORMAT)


class ChatRequestHandler(socketserver.BaseRequestHandler):
    clients = set()

    # 每个连接进来时的操作
    def setup(self):
        super(ChatRequestHandler, self).setup()
        self.event = threading.Event()
        self.lock = threading.Lock()
        self.clients.add(self.request)

    # 每个连接的业务处理
    def handle(self):
        super(ChatRequestHandler, self).handle()
        while not self.event.is_set():
            
            # use Client code except
            try:
                data = self.request.recv(1024)
            except ConnectionResetError:
                self.clients.remove(self.request)
                self.event.set()
                break
                
            # use TCP tools except
            if data.rstrip() == b'quit' or data == b'':
                with self.lock:
                    self.clients.remove(self.request)
                logging.info("{}:{} is down ~~~~~~".format(*self.client_address))
                self.event.set()
                break

            msg = "{}:{} {}".format(*self.client_address, data.decode()).encode()
            logging.info('{}:{} {}'.format(*self.client_address, msg))
            with self.lock:
                for client in self.clients:
                    client.send(msg)

    # 每个连接关闭的时候，会执行
    def finish(self):

        super(ChatRequestHandler, self).finish()
        self.event.set()
        self.request.close()


class ChatTCPServer:
    def __init__(self, ip, port):
        self.ip = ip
        self.port = port
        self.sock = socketserver.ThreadingTCPServer((self.ip, self.port), ChatRequestHandler)

    def start(self):
        self.sock.daemon_threads = True
        threading.Thread(target=self.sock.serve_forever, name='server', daemon=True).start()

    def stop(self):
        self.sock.server_close()


if __name__ == '__main__':
    cts = ChatTCPServer('127.0.0.1', 9999)
    cts.start()

    while True:
        cmd = input('>>>>:').strip()
        if cmd == 'quit':
            cts.stop()
        if not cmd: continue
        print(threading.enumerate())
```
使用TCP工具强制退出时会发送空串，而使用我们自己写的tcp client，则不会发送，所以这里所了两种异常的处理。socketserver只是用在服务端，客户端使用上一篇TCP client即可。