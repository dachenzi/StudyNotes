<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 Python操作数据库](#1-python操作数据库)
- [2 安装模块](#2-安装模块)
- [3 基本使用](#3-基本使用)
    - [3.1 创建一个连接](#31-创建一个连接)
    - [3.2 连接数据库](#32-连接数据库)
    - [3.3 游标](#33-游标)
        - [3.3.1 利用游标操作数据库](#331-利用游标操作数据库)
        - [3.3.2 事务管理](#332-事务管理)
        - [3.3.3 执行SQL语句](#333-执行sql语句)
            - [3.3.3.1 批量执行](#3331-批量执行)
            - [3.3.3.2 SQL注入攻击](#3332-sql注入攻击)
            - [3.3.3.3 参数化查询](#3333-参数化查询)
    - [3.4 获取查询结果](#34-获取查询结果)
        - [3.4.1 带列明的查询](#341-带列明的查询)
    - [3.5 上下文支持](#35-上下文支持)

<!-- /TOC -->

# 1 Python操作数据库
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Python 提供了程序的DB-API，支持众多数据库的操作。由于目前使用最多的数据库为MySQL，所以我们这里以Python操作MySQL为例子，同时也因为有成熟的API,所以我们不必去关注使用什么数据，因为操作逻辑和方法是相同的。

# 2 安装模块
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Python 程序想要操作数据库，首先需要安装 模块 来进行操作，Python 2 中流行的模块为 MySQLdb，而该模块在Python 3 中将被废弃，而使用PyMySQL，这里以PyMySQL模块为例。下面使用pip命令安装PyMSQL模块
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
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;首先我们需要手动安装一个MySQL数据库，这里不再赘述，参考博文：http://www.cnblogs.com/dachenzi/articles/7159510.html   

连接数据库并执行sql语句的一般流程是：
1. 建立连接
2. 获取游标(创建)
3. 执行SQL语句
4. 提交事务
5. 释放资源

对应到代码上的逻辑为：
1. 导入相应的Python模块
2. 使用connect函数连接数据库，并返回一个Connection对象
3. 通过Connection对象的cursor方法，返回一个Cursor对象
4. 通过Cursor对象的execute方法执行SQL语句
5. 如果执行的是查询语句，通过Cursor对象的fetchall语句获取返回结果
6. 调用Cursor对象的close关闭Cursor
7. 调用Connection对象的close方法关闭数据库连接

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
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;调用connect函数，将创建一个数据库连接并得到一个Connection对象，Connection对象定义了很多的方法和异常。
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


&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在实际的编程过程中，一般不会直接调用begin、commit和rollback函数，而是通过`上下文管理器实现事务的提交与回滚操作`。

## 3.3 游标
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;游标是系统为用户开设的一个数据缓存区，存放SQL语句执行的结果，用户可以用SQL语句逐一从游标中获取记录，并赋值给变量，交由Python进一步处理。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在数据库中，游标是一个十分重要的概念。游标提供了一种对从表中检索出的数据进行操作的灵活手段，就本质而言，游标实际上是一种能从包括多条数据记录的结果集中每次提取一条记录的机制。游标总是与一条SQL 选择语句相关联因为游标由结果集（可以是零条、一条或由相关的选择语句检索出的多条记录）和结果集中指向特定记录的游标位置组成。当决定对结果集进行处理时，必须声明一个指向该结果集的游标。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;正如前面我们使用Python对文件进行处理，那么游标就像我们打开文件所得到的文件句柄一样，只要文件打开成功，该文件句柄就可代表该文件。对于游标而言，其道理是相同的。

### 3.3.1 利用游标操作数据库
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

### 3.3.2 事务管理
Connection对象包含如下三个方法用于事务管理：
- begin：开始事务
- commit：提交事务
- rollback：回滚事务

```python
import pymysql

def connect_mysql():
    db_config = {
        'host': '10.0.0.13',
        'port': 3306,
        'user': 'dahl',
        'password': '123456',
        'charset': 'utf8'
    }

    return pymysql.connect(**db_config)


if __name__ == '__main__':
    conn = connect_mysql()
    cursor = conn.cursor()  # 创建游标

    conn.begin()
    sql = r"insert into test.student (id,name,age) VALUES (5,'dahl',23)"  # 要执行的sql语句
    res = cursor.execute(sql)  # 交给 游标执行
    print(res)
    conn.commit()
```
这样使用是极其不安全的，因为sql语句有可能执行失败，当失败时，我们应该进行回滚
```python
if __name__ == '__main__':
    conn = None 
    cursor = None
    try:
        conn = connect_mysql()
        cursor = conn.cursor()  # 创建游标
        sql = r"insert into test.student (id,name,age) VALUES (6,'dahl',23)"  # 要执行的sql语句
        cursor.execute(sql)  # 交给 游标执行
        conn.commit()    # 提交事务
    except:
        conn.rollback()   # 当SQL语句执行失败时，回滚
    finally:
        if cursor:   
            cursor.close()    # 关闭游标
        if conn:
            conn.close()   # 关闭连接
```

### 3.3.3 执行SQL语句
用于执行SQL语句的两个方法为：
- cursor.execute(sql)：执行一条sql语句
- executemany(sql,parser)：执行多条语句

```python
sql = r"select * from test.student" 
res = cursor.execute(sql)  
```

#### 3.3.3.1 批量执行
executemany用于批量执行sql语句，
- sql 为模板，c风格的占位符。
- parser：为模板填充的数据（可迭代对象）

```python
sql = r"insert into test.student (id,name,age) values (%s,'daxin',20)"
cursor.executemany(sql,(7,8,9,))
```

#### 3.3.3.2 SQL注入攻击
我们一般在程序中使用sql，可能会有如下代码：
```sql
# 接受用户id，然后拼接查询用户信息的SQL语句，然后查询数据库。
userid = 10 # 来源于程序
sql = 'select * from user where user_id = {}'.format(userid)
```
userid是可变的，比如通过客户端发来的request请求，直接拼接到查询字符串中。如果客户端传递的userid是 '5 or 1='呢
```sql
sql = 'select * from test.student where id = {}'.format('5 or 1=1')
```
此时真正在数据库中查询的sql语句就变成了
```sql
select * from test.student where id = 5 or 1=1
```
这条语句的where条件的函数含义是：__`当id等于5，或者1等于1时，列出所有匹配的字段，这样就轻松的查到了标准所有的数据！！！`__
> 永远不要相信客户端传来的数据是规范及安全的。

#### 3.3.3.3 参数化查询
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;cursor.execute(sql)方法还额外提供了一个args参数，用于进行参数化查询，它的类型可以为元组、列表、字典。如果查询字符串使用命名关键字(%(name)s)，那么就必须使用字典进行传参。
```sql
sql = 'select * from test.student where id = %s'
cursor.execute(sql,args=(2,))

sql = 'select * from test.student where id = %(id)s'
cursor.execute(sql,args={'id':2})
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们说使用参数化查询，效率会高一点，为什么呢？因为SQL语句缓存。数据库一般会对SQL语句编译和缓存，编译只对SQL语句部分，所以参数中就算有SQL指令也不会被执行。编译过程，需要此法分析、语法分析、生成AST、优化、生成执行计划等过程，比较耗费资源。服务端会先查找是否是同一条语句进行了缓存，如果缓存未失效，则不需要再次编译，从而降低了编译的成本，降低了内存消耗。
> 可以认为SQL语句字符串就是一个key，如果使用拼接方案，每次发过去的SQL语句都不一样，都需要编译并缓存。大量查询的时候，首选使用参数化查询，以节省资源。

## 3.4 获取查询结果
Cursor类提供了三种查询结果集的方法：（返回值为元组，多个值为嵌套元组）
- `fetchone()`：获取一条
- `fetchmany(size=None)`：获取多条，当size为None时，返回一个包含一个元素的嵌套元组
- `fetchall()`：获取所有

```python
sql = 'select * from test.student'
cursor.execute(sql)
print(cursor.fetchall())
print(cursor.fetchone())  # None
print(cursor.fetchmany(2))  # None
```

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;fetch存在一个指针，fetchone一次，就读取结果集中的一个结果，如果fetchall，那么一次就会读取完所有的结果。可以通过调整这个指针来重复读取
- cursor.rownumber: 返回当前行号，可以修改，支持 负数
- cursor.rowcount: 返回总行数

```python
sql = 'select * from test.student'
cursor.execute(sql)
print(cursor.fetchall())  # ((2, 'dahl', 20), (3, 'dahl', 21), (4, 'daxin', 22), (5, 'dahl', 23), (6, 'dahl', 23), (7, 'daxin', 20), (8, 'daxin', 20), (9, 'daxin', 20))
cursor.rownumber = 0
print(cursor.fetchone())  # (2, 'dahl', 20)
print(cursor.fetchmany()) # ((3, 'dahl', 21),)
```

> fetch操作的是结果集，结果集是保存在客户端的，也就是说fetch的时候，查询已经结束了。

### 3.4.1 带列明的查询
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;结果中不包含字段名，除非我们记住字段的顺序，不然很麻烦，那么下面来解决这个问题。

观察cursor原码，我们发现，它接受一个参数cursor，有一个参数是：`DictCursor`，查看源码得知，它是Cursor的Mixin子类。
```python
def cursor(self, cursor=None):
    """
    Create a new cursor to execute queries with.

    :param cursor: The type of cursor to create; one of :py:class:`Cursor`,
        :py:class:`SSCursor`, :py:class:`DictCursor`, or :py:class:`SSDictCursor`.
        None means use Cursor.
    """
    if cursor:
        return cursor(self)
    return self.cursorclass(self)
```
观察DictCursor的原码，得知结果集会返回一个字典，一个字段名:值的结果，所以，只需要传入cursor参数即可
```python
    from pymysql import cursors  # DictCursor存在与cursors模块中
    ... ... 
    cursor = conn.cursor(cursors.DictCursor) 
    sql = 'select * from test.student'
    cursor.execute(sql)
    print(cursor.fetchone())  # {'id': 2, 'name': 'dahl', 'age': 20}
```

## 3.5 上下文支持
Connection和Cursor类实现了__enter__和__exit__方法，所以它支持上下文管理。
- Connection：进入时返回一个cursor，退出时如果有异常，则回滚，否则提交
- Cursor: 进入时返回cursor本身，退出时，关闭cursor连接

所以利用上下文的特性，我们可以这样使用
```python
import pymysql

def connect_mysql():
    db_config = {
        'host': '10.0.0.13',
        'port': 3306,
        'user': 'dahl',
        'password': '123456',
        'charset': 'utf8'
    }

    return pymysql.connect(**db_config)

if __name__ == '__main__':
conn = connect_mysql()
with conn as cursor:
    sql = 'select * from student'
    cursor.execute(sql)
    print(cursor.fetchmany(2))

# conn的exit只是提交了，并没有关闭curosr
cursor.close()
conn.close()
```
如果要自动关闭cursor,可以进行如下改写
```python
if __name__ == '__main__':
conn = connect_mysql()
with conn as cursor:
    with cursor   # curosr的exit会关闭cursor
        sql = 'select * from student'
        cursor.execute(sql)
        print(cursor.fetchmany(2))

conn.close()
```
> 多个 cursor共享一个 conn





















