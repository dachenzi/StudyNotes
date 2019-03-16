<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 ORM](#1-orm)
- [2 sqlalchemy](#2-sqlalchemy)
- [3 基本使用](#3-基本使用)
    - [3.1 创建连接](#31-创建连接)
    - [3.2 创建基类](#32-创建基类)
    - [3.3 创建实体类](#33-创建实体类)
    - [3.4 实例化](#34-实例化)
    - [3.5 创建表](#35-创建表)
    - [3.6 创建会话Session](#36-创建会话session)
    - [3.7 数据操作](#37-数据操作)
        - [3.7.1 增加数据](#371-增加数据)
        - [3.7.2 简单查询](#372-简单查询)
        - [3.7.3 修改数据](#373-修改数据)

<!-- /TOC -->
# 1 ORM
Object-Relational Mapping，把关系数据库的表结构映射到对象上。使用面向对象的方式来操作数据库。 
  
![orm](photo/orm.png)  

下面是一个关系模型与Python对象之间的映射关系：
- table   -->  class    : 表映射为类
- row     -->  object   : 行映射为实例
- column  -->  property : 字段映射为属性

# 2 sqlalchemy
SQLAlchemy是一个ORM框架。内部是使用了连接池来管理数据库连接。要使用sqlalchemy，那么需要先进行安装：
```python
pip3 install sqlalchemy
```
查看版本
```python
In [1]: import sqlalchemy
In [2]: print(sqlalchemy.__version__)
1.3.1
```

# 3 基本使用
先来总结一下使用sqlalchemy框架操作数据库的一般流程：
1. `创建连接`(不同类型数据库使用不同的连接方式)
2. `创建基类`(类对象要继承，因为基类会利用元编程为我们的子类绑定关于表的其他属性信息)
3. `创建实体类`(用来对应数据库中的表)
4. `编写实体类属性`(用来对应表中的字段/属性)
5. `创建表`(如果表不存在,则需要执行语句在数据库中创建出对应的表)
6. `实例化`(具体的一条record记录)
7. `创建会话session`(用于执行sql语句的连接) 
8. `使用会话执行SQL语句`
9. `关闭会话`

## 3.1 创建连接
sqlalchemy 使用引擎管理数据库连接(DATABASE URLS)，连接的一般格式为：
```python
dialect+driver://username:password@host:port/database
```
- `dialect`：表示什么数据库(比如,mysql,sqlite,oracle等)
- `driver`：用于连接数据库的模块(比如pymysql,mysqldb等)
- `username`：连接数据库的用户名
- `password`：连接数据库的密码
- `host`: 数据库的主机地址
- `port`: 数据库的端口
- `database`: 要连接的数据库名称

pymysql模块是较长用于连接mysql的模块，使用pymysql的连接的语句为：
- `mysql+pymysql://dahl:123456@10.0.0.13:3306/test`

创建引擎用于进行数据库的连接：`create_engine(urls)`
```python
import sqlalchemy

db_url = 'mysql+pymysql://dahl:123456@10.0.0.13:3306/test'
engine = sqlalchemy.create_engine(db_url,echo=True)
```
- echo：引擎是否打印执行的sql语句，等于True时，表示打印，便于调试

> `特别注意`：创建引擎并不会马上连接数据库，直到让数据库执行任务是才连接。

## 3.2 创建基类
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用: `sqlalchemy.ext.declarative.declarative_base` 来构造声明性类定义的基类。因为sqlalchemy内部大量使用了元编程，为实例化的子类注入映射所需的属性，所以我们定义的映射要继承自它（必须继承）
> 一般只需要一个这样的基类
```python
Base = sqlalchemy.ext.declarative.declarative_base()
# 或者
from sqlalchemy.ext import declarative
Base = declarative.declarative_base()
```

## 3.3 创建实体类
现数据库存在如下表
```sql
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(30) DEFAULT NULL,
  `age` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=10 DEFAULT CHARSET=utf8mb4;
```
创建对应的实体类:
```python
import sqlalchemy
from sqlalchemy.ext import declarative
from sqlalchemy import Column, String, Integer

db_url = 'mysql+pymysql://dahl:123456@10.0.0.13:3306/test'
engine = sqlalchemy.create_engine(db_url, echo=True)

Base = declarative.declarative_base()

class Student(Base):

    __tablename__ = 'student'

    id = Column(Integer, primary_key=True, nullable=False, autoincrement=True)
    name = Column(String, default='Null')
    age = Column(Integer, default='Null')

print(Student.__dict__)

# Table('student', MetaData(bind=None),   这里没有绑定engine，所以是None
# Column('id', Integer(), table=<student>, primary_key=True, nullable=False), 
# Column('name', String(), table=<student>, default=ColumnDefault('Null')), 
# Column('age', Integer(), table=<student>, default=ColumnDefault('Null')), schema=None),
```
- \_\_tablename__: 表明，如果表已存在，则必须指定正确的表名
- Column: 用于构建一个列对象，它的参数一般都和数据库中的列属性是对应的，主要有：
    - name: 数据库中表示的此列的名称
    - `type`：字段类型(比如String、Integer，这里是sqlalchemy包装的类型，对应的是数据库的varchar、int等)，来自TypeEngine的子类
    - `autoincrement`：是否自增
    - default：默认值，可以是值，可调用对象或者类，当写入数据该字段没有指定时，调用。
    - doc：字段说明信息
    - key: 一个可选的字符串标识符，用于标识表上的此Column对象。
    - index: 是否启用索引
    - `nullable`: 是否可以为空
    - onupdate: 如果在更新语句的SET子句中不存在此列，则将在更新时调用该值
    - `primary_key`: 主键
    - server_default:它的值是 FetchedValue实例、字符串、Unicode 或者 text()实例，用作DDL语句中该列的default值

注意：Column和String、Integer等都来自于sqlalchemy下的方法，要么直接导入，要么就使用sqlalchemy.String来引用。

## 3.4 实例化
通过我们构建的类，来实例化的对象，在将来就是数据库中的一条条记录。
```python
student = Student()
student.name = 'daxin'
student.id = 1
student.age = 20
print(student)   # <1 daxin 20>
```

## 3.5 创建表
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们自己写的类都是继承自Base，每继承一次Base类，在Base类的metadata属性中就会记录当前子类，metadata提供了方法用于删除/创建表。如果数据库中已经存在对应的表，那么将不会继续创建
- drop_all(bind=None, tables=None, checkfirst=True):删除metadata中记录的所有表
- create_all(bind=None, tables=None, checkfirst=True):创建metadata中记录的所有表

```python
Base.metadata.create_all(bind=engine)  # 需要通过引擎去执行

# 下面是engine的echo为true时的输出信息
# 2019-03-16 16:53:45,922 INFO sqlalchemy.engine.base.Engine 
# CREATE TABLE hello (
# 	id INTEGER NOT NULL AUTO_INCREMENT, 
# 	name VARCHAR(24), 
# 	age INTEGER, 
# 	PRIMARY KEY (id)
# )
# 
# 
# 2019-03-16 16:53:45,922 INFO sqlalchemy.engine.base.Engine {}
# 2019-03-16 16:53:45,926 INFO sqlalchemy.engine.base.Engine COMMIT
# 2019-03-16 16:53:45,927 INFO sqlalchemy.engine.base.Engine 
# CREATE TABLE world (
# 	id INTEGER NOT NULL AUTO_INCREMENT, 
# 	name VARCHAR(24), 
# 	age INTEGER, 
# 	PRIMARY KEY (id)
# )
# 
# 
# 2019-03-16 16:53:45,927 INFO sqlalchemy.engine.base.Engine {}
# 2019-03-16 16:53:45,928 INFO sqlalchemy.engine.base.Engine COMMIT
```
- 生产环境很少这样创建表，都是系统上线的时候由脚本生成。
- 生产环境很少删除表，宁可废除都不能删除。

## 3.6 创建会话Session
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在一个会话中操作数据库，绘画建立在连接上，连接被引擎管理，当第一次使用数据库时，从引擎维护的连接池中取出一个连接使用。

```python
from sqlalchemy.orm import sessionmaker
Session = sessionmaker(bind=engine) 
session = Session()  # 实例化一个session对象
```
- session对象线程不安全。所以不同线程应该使用不同的session对象
- Session类和engine有一个就行了。

`scoped_session`是sqlalchemy提供的线程安全的session，利用的是ThreadLocal实现的。
## 3.7 数据操作

### 3.7.1 增加数据

|方法|含义
----|---|
add()| 增加一个对象
add_all()| 增加多个对象，类型为可迭代

```python
import sqlalchemy
from sqlalchemy.ext import declarative
from sqlalchemy import Column, String, Integer
from sqlalchemy.orm import sessionmaker

db_url = 'mysql+pymysql://dahl:123456@10.0.0.13:3306/test'
engine = sqlalchemy.create_engine(db_url, echo=True)

Base = declarative.declarative_base()
DBSession = sessionmaker(bind=engine)
session = DBSession()

class Student(Base):
    __tablename__ = 'student'

    id = Column(Integer, primary_key=True, nullable=False, autoincrement=True)
    name = Column(String(24), default='Null')
    age = Column(Integer, default='Null')

    def __repr__(self):
        return '<{} {} {}>'.format(
            self.id, self.name, self.age
        )

daxin = Student(id=12, name='daxin', age=20)
session.add(daxin)
dachenzi = Student(id=13,name='dachenzi', age=21)
xiaobai = Student(id=14,name='xiaobai', age=22)
session.add_all((dachenzi,xiaobai))   # 新增三条数据
session.commit()
```
需要注意的是下面的情况：
```python
daxin = Student(name='daxin')
daxin.age = 40
session.add(daxin)  # 1
session.commit()   
daxin.age = 20  
session.add(daxin)  # 2
session.commit()
daxin.age = 10
session.add_all([daxin, daxin, daxin, daxin]) # 3
session.commit()  
# <30 daxin 10>
```
结果生成个1条数据，<30 daxin 10>，为什么呢？由于id属于自增列，我们在执行#1时，id是没有固定下来的。
1. #1执行后，commit，则daxin的id就固定下来了。
2. #2执行时，由于id没变，所以并不会新增数据，而是使用update语句更新了age字段
3. #3执行时，由于都是daxin的，id，name，age都没有改变，所以只会执行1条语句
> 当engine的echo等于true时，看到具体的sql语句，一切就很明白了。

### 3.7.2 简单查询
使用session的query方法进行简单查询,格式为：
- `session.query(student)`: 等同于select * from student;
- `session.query(student).get(2)`: 等同于select * from student where id = 2,这里的get方法只能主键查询

```python
std_list = session.query(Student)
print(std_list)        # SELECT student.id AS student_id, student.name AS student_name, student.age AS student_age FROM student
print(type(std_list))  # <class 'sqlalchemy.orm.query.Query'>
```
这里直接打印并不会结果，因为它太懒了，你不迭代它，它就不会真的去数据库查询。
```python
std_list = session.query(Student)
for std in std_list:
    print(std)

# 通过get来过滤主键，是可以直接执行返回结果的。
std_list = session.query(Student).get(30)
print(std_list)
```

### 3.7.3 修改数据
