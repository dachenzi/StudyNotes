# 1 Python操作数据库
Python 提供了程序的DB-API，支持众多数据库的操作。由于目前使用最多的数据库为MySQL，所以我们这里以Python操作MySQL为例子，同时也因为有成熟的API,所以我们不必去关注使用什么数据，因为操作逻辑和方法是相同的。

# 2 安装模块
Python 程序想要操作数据库，首先需要安装 模块 来进行操作，Python 2 中流行的模块为 MySQLdb，而该模块在Python 3 中将被废弃，而使用PyMySQL，这里以PyMySQL模块为例。下面使用pip命令安装PyMSQL模块
```python
pip3 install pymysql
```
如果没有pip3命令那么需要确认环境变量是否有添加，安装完毕后测试是否安装完毕。
```python
lidaxindeMacBook-Pro:~ DahlHin$ python3
Python 3.6.1 (v3.6.1:69c0db5050, Mar 21 2017, 01:21:04)
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import pymysql
>>>
# 如果没有报错，则表示安装成功
```
# 3 基本使用
首先我们需要手动安装一个MySQL数据库，这里不再赘述，参考博文：http://www.cnblogs.com/dachenzi/articles/7159510.html 

## 3.1 创建一个连接
使用`pymysql.connect`方法来连接数据库
```python
import pymysql
 
pymysql.connect(host=None, user=None, password="",
                 database=None, port=0, unix_socket=None,
                 charset=''......)
```
主要的参数有:
- host：表示连接的数据库的地址
- user：表示连接使用的用户
- password：表示用户对应的密码
- database：表示连接哪个库
- port：表示数据库的端口
- unix_socket：表示使用socket连接时，socket文件的路径
- charset：表示连接使用的字符集　
- read_default_file：读取mysql的配置文件中的配置进行连接

## 3.2 连接数据库
调用connect函数，将创建一个数据库连接并得到一个Connection对象，Connection对象定义了很多的方法和异常。
```python
host = '10.0.0.13'
port = 3306
user = 'dahl'
password = '123456'
database = 'test'

conn = pymysql.connect(host, user, password, database, port)
print(conn)  # <pymysql.connections.Connection object at 0x000001ABD3063550>
conn.ping()  # 没有返回值，无法连接会提示异常
```
这里conn就是一个Connection对象，它具有一下属性和方法：
- `begin`：开始事务
- `commit`：提交事务
- `rollback`：回滚事务
- `cursor`：返回一个Cursor对象
- `autocommit`：设置事务是否自动提交
- `set_character_set`：设置字符集编码
- `get_server_info`：获取数据库版本信息
- `ping(reconnect=True)`: 测试数据库是否活着，reconnect表示断开与服务器连接后是否重连，连接关闭时抛出异常（一般用来测通断）


在实际的编程过程中，一般不会直接调用begin、commit和rollback函数，而是通过`上下文管理器实现事务的提交与回滚操作`。

## 3.3 游标
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;游标是系统为用户开设的一个数据缓存区，存放SQL语句执行的结果，用户可以用SQL语句逐一从游标中获取记录，并赋值给变量，交由Python进一步处理。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在数据库中，游标是一个十分重要的概念。游标提供了一种对从表中检索出的数据进行操作的灵活手段，就本质而言，游标实际上是一种能从包括多条数据记录的结果集中每次提取一条记录的机制。游标总是与一条SQL 选择语句相关联因为游标由结果集（可以是零条、一条或由相关的选择语句检索出的多条记录）和结果集中指向特定记录的游标位置组成。当决定对结果集进行处理时，必须声明一个指向该结果集的游标。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;正如前面我们使用Python对文件进行处理，那么游标就像我们打开文件所得到的文件句柄一样，只要文件打开成功，该文件句柄就可代表该文件。对于游标而言，其道理是相同的。

### 3.3.1 创建游标
在进行数据库的操作之前需要创建一个游标对象，来执行sql语句。
```python
cursor = conn.cursor()
```
下面利用游标来执行sql语句：
```python
import pymysql
 
def connect_mysql():
 
    db_config = {
        'host':'127.0.0.1',
        'port':3306,
        'user':'root',
        'password':'abc.123',
        'charset':'utf8'
    }
 
    return  pymysql.connect(**db_config)
 
if __name__ == '__main__':
    conn = connect_mysql()
    cursor = conn.cursor()    # 创建游标
    sql = r" select user,host from mysql.user "  # 要执行的sql语句
    cursor.execute(sql)  # 交给 游标执行
    result = cursor.fetchall()  # 获取游标执行结果
    print(result)
```









































小结
　　在Python中操作数据库，基本步骤如下：
导入相应的Python模块
使用connect函数连接数据库，并返回一个Connection对象
通过Connection对象的cursor方法，返回一个Cursor对象
通过Cursor对象的execute方法执行SQL语句
如果执行的是查询语句，通过Cursor对象的fetchall语句获取返回结果
调用Cursor对象的close关闭Cursor
调用Connection对象的close方法关闭数据库连接