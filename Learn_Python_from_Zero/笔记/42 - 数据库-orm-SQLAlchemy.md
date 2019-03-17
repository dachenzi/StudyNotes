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
        - [3.7.4 删除数据(不建议)](#374-删除数据不建议)
        - [3.7.5 状态](#375-状态)
        - [3.7.6 枚举字段](#376-枚举字段)
        - [3.7.7 复杂查询](#377-复杂查询)
            - [3.7.7.1 where条件查询](#3771-where条件查询)
            - [3.7.7.2 排序](#3772-排序)
            - [3.7.7.3 分页(偏移量)](#3773-分页偏移量)
            - [3.7.7.4 消费方法](#3774-消费方法)
            - [3.7.7.5 分组及聚合方法](#3775-分组及聚合方法)
            - [3.7.7.6 关联查询](#3776-关联查询)
                - [隐式连接](#隐式连接)
                - [join连接](#join连接)
            - [3.7.7.7 relationship](#3777-relationship)

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
修改的数据的流程分为两步：
1. 查找匹配的数据
2. 修改后，提交

```python
std = session.query(Student).get(30)
print(std)  # <30 daxin 200>
std.age = 1000
session.add(std)
session.commit()
std = session.query(Student).get(30)
print(std)  # <30 daxin 1000>
```
> 大部分ORM是都是这样，必须先查才能改。

### 3.7.4 删除数据(不建议)
使用session.delete来删除数据
```python
std = session.query(Student).get(30)
session.delete(std)
session.commit()
std = session.query(Student).get(30)
print(std)  # None
```

### 3.7.5 状态
当我们创建一条数据，或是从数据库中获取一条数据时，数据本身是存在一个状态属性的，用来标识当前数据是否持久化，或者其他状态，在sqlalchemy中，`inspect(obj)`可以用来窥探数据(obj)的状态
```python
daxin = Student(name='daxin')
daxin.age = 10000
state = sqlalchemy.inspect(daxin)
print(state)  # <sqlalchemy.orm.state.InstanceState object at 0x00000186EEC38EB8>
std = session.query(Student).get(28)
state = sqlalchemy.inspect(std)
print(state)  # <sqlalchemy.orm.state.InstanceState object at 0x00000186EFDCDA20>
```
我们看到，inspect返回的是一个InstanceState对象。这个对象有以下几个属性：
- session_id：获取数据时的session ID，如果是自建数据未持久化ID为None
- _attached: 是否是附加的(数据已经加载(已add)，就差提交的数据库了)
- transient：是否是临时数据(临时创建，还没有提交(还没有add))
- pending: 是否是待定的数据
- persistent: 是否是持久的
- deleted: 是否已删除
- detached：是否被分离
- key：key是多少
> InstanceState对象，可以通过sqlalchemy.orm.state导入

具体的状态信息与含义如下：

|状态|说明|
----|----|
`transient`| 实体类尚未加入到session中，同时并没有保存到数据库中
`pending`|transient的实体被加入到session中，状态切换到pending，但它还没有被flsh到数据库中
`persistent`|session中的实体对象对应着数据库中的真实记录。pending状态在提交成功后可以变成persistent状态，或者查询成功返回的实体也是persistent状态
`deleted`|实体被删除且已经flush但未commit完成。事物提交成功了，实体变成detached，事物失败返回persistent状态
`detached`| 删除成功的实体进入这个状态

所以数据的状态变化如下：
1. 新建一个实体，状态是transient临时的
2. add()以后，状态变为pending
3. commit()以后，状态变为persistent
4. 查询成功返回的实体状态也是persistent状态
5. delete()并flush()以后，状态变为deleted
6. commit()以后，变为detached，提交失败，回退到persistent状态
> flush()方法，主动把改变应用到数据库中去

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;删除、修改操作，需要对应一个真实存在的数据，也就是说数据的状态是persistent才行。当使用add语句新增信息时，如果这个对象已经添加过数据库了，那么它的状态会变为persistent，如果对persistent的数据进行修改继续提交的话，那么使用的将会是update语句而非insert。这也是前面为啥多次对一个数据进行add，提交了多次只会插入1次的原因。

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

def getstate(state: InstanceState):
    print("""
    session_id={} transient={} _attached={} pending={} persistent={} deleted={} detached={}
    """.format(state.session_id, state.transient, state._attached, state.pending, state.persistent, state.deleted,
               state.detached))

daxin = Student(name='daxin')
daxin.age = 10000
state = sqlalchemy.inspect(daxin)
getstate(state)  # session_id=None transient=True _attached=False pending=False persistent=False deleted=False detached=False

session.add(daxin)
getstate(state)  # session_id=1 transient=False _attached=True pending=True persistent=False deleted=False detached=False

session.commit()
getstate(state)  # session_id=1 transient=False _attached=True pending=False persistent=True deleted=False detached=False

std = session.query(Student).get(28)
state = sqlalchemy.inspect(std)
getstate(state)  # session_id=1 transient=False _attached=True pending=False persistent=True deleted=False detached=False

std = session.query(Student).get(31)
session.delete(std)
state = sqlalchemy.inspect(std)
getstate(state)  # session_id=1 transient=False _attached=True pending=False persistent=True deleted=False detached=False

session.flush()
getstate(state)  # session_id=1 transient=False _attached=True pending=False persistent=False deleted=True detached=False

session.commit()
getstate(state)  # session_id=None transient=False _attached=False pending=False persistent=False deleted=False detached=True
```

### 3.7.6 枚举字段
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在数据库的字段类型中，比如性别字段，我们可能会限制数据来源为M(male),F(Female),这个时候字段类型可以是枚举的，但是在sqlalchemy中，原生的字段类型没有枚举类型，那么就需要借助enum类了。
```python
import sqlalchemy
from sqlalchemy.ext import declarative
from sqlalchemy import Column, String, Integer
from sqlalchemy.orm import sessionmaker
import enum

db_url = 'mysql+pymysql://dahl:123456@10.0.0.13:3306/test'
engine = sqlalchemy.create_engine(db_url, echo=True)

Base = declarative.declarative_base()
DBSession = sessionmaker(bind=engine)
session = DBSession()

class GenderEnum(enum.Enum):
    M = 'M'
    F = 'F'

class Student(Base):
    __tablename__ = 'student'

    id = Column(Integer, primary_key=True, nullable=False, autoincrement=True)
    name = Column(String(24), default='Null')
    age = Column(Integer, default='Null')
    gender = Column(sqlalchemy.Enum(GenderEnum),nullable=False)

    def __repr__(self):
        return '<{} {} {}>'.format(
            self.id, self.name, self.age
        )
```
用起来很麻烦，所以建议性别使用1或者0来存储，显示的时候做对于转换即可

### 3.7.7 复杂查询

#### 3.7.7.1 where条件查询
使用filter方法进行条件过滤查询：
- `session.query(student).filter(student.id > 10)`：相当于select * from student where student.id > 10 

where条件中的关系：
- `AND`(与) 对应 `and_`
- `OR`(或) 对应 `or_`
- `not`(非) 对应 `not_`
- `in` 对应字段的 `in_`
- `not in` 对应字段的 `notin_`
- `like` 对应 字段的like方法
- `not like` 对应 字段的notlike方法

想要使用与或非，需要先行导入
```python
import sqlalchemy
from sqlalchemy.ext import declarative
from sqlalchemy import Column, String, Integer
from sqlalchemy.orm import sessionmaker
from sqlalchemy import and_, or_, not_

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

def getresulte(stds):
    for std in stds:
        print(std)

```
条件判断之`AND`(&,and_)
```python
# and
std_list = session.query(Student).filter(and_(Student.id < 30, Student.id > 27))
getresulte(std_list)

std_list = session.query(Student).filter(Student.id < 30).filter(Student.id > 27)   # filter返回的还是一个结果集，所以还可以继续使用filter进行过滤
getresulte(std_list)

std_list = session.query(Student).filter(Student.id < 30, Student.age > 100)  # 多个条件一起写，也是and的关系
getresulte(std_list)

std_list = session.query(Student).filter((Student.name == 'daxin') & (Student.age > 28))
getresulte(std_list)
```
条件判断之`OR`(or_，|)
```python
# or
std_list = session.query(Student).filter(or_(Student.id > 27, Student.age < 50))
getresulte(std_list)

std_list = session.query(Student).filter((Student.id > 27) | (Student.age < 50 ))
getresulte(std_list)
```
条件判断之`NOT`(not_,~)
```python
# not
std_list = session.query(Student).filter(not_(Student.id == 32))
getresulte(std_list)

std_list = session.query(Student).filter(~(Student.id == 32))
getresulte(std_list)
```
`like`和`in`及`not in`
```python
# like
std_list = session.query(Student).filter(Student.name.like('da%'))
getresulte(std_list)

# not like
std_list = session.query(Student).filter(Student.name.notlike('da%'))
getresulte(std_list)

# in
std_list = session.query(Student).filter(Student.age.in_([10,30,50]))
getresulte(std_list)

# not in
std_list = session.query(Student).filter(Student.age.notin_([10,30,50]))
getresulte(std_list)
```

#### 3.7.7.2 排序
- order_by：排序
- asc: 升序（默认）
- desc: 降序

```python
# 升序
std_list = session.query(Student).filter(Student.name.like('da%')).order_by(Student.age)
getresulte(std_list)

# 降序
std_list = session.query(Student).filter(Student.name.like('da%')).order_by(Student.age.desc())
getresulte(std_list)
```

#### 3.7.7.3 分页(偏移量)
- limit: 显示结果的条目数
- offset：偏移量

```python
# limit
std_list = session.query(Student).filter(Student.name.like('da%')).order_by(Student.age.desc()).limit(3).offset(2)
getresulte(std_list)
# select * from Student where name like 'da%' order_by age desc limit 3 offset 2
```

#### 3.7.7.4 消费方法
- count(): 获取总条数（仅仅针对结果集，不能all以后再count）
- all(): 取所有行（默认）（列表）
- first()：取首行
- one()：有且只有一行，多行则抛出异常
- delete()：对查询出来的数据直接进行删除
- scalar(): 第一条记录的第一个元素

```python
# count
std_list = session.query(Student)
print(std_list.count())

# first
std_list = session.query(Student).first()
print(std_list)
```

> first本质上就是limit。

#### 3.7.7.5 分组及聚合方法
sqlalchemy同样提供了聚合方法，使用sqlalchemy.func来调用
- group_by:分组显示

func提供的方法有
- max:求最大值
- min:求最小值
- avg:求平均值
- count:聚合（一般和分组连用）

```python
# max
std_list = session.query(Student.name,sqlalchemy.func.max(Student.age))  # select name,max(age) from Student;
getresulte(std_list)

# count
std_list = session.query(Student.name,sqlalchemy.func.count(Student.id)).group_by(Student.name)  # SELECT name, count(id)  FROM student GROUP BY name 
getresulte(std_list)
```

#### 3.7.7.6 关联查询
sqlalchemy提供ForeignKey用来进行外键关联，它的格式为：
```python
sqlalchemy.ForeignKey(表名.字段名,ondelete='更新规则')

# 如果填写映射后的class，那么可以直接写：类.字段
# 如果填写数据库中的表，那么需要使用引号：'数据库表名.字段名'
```

更新规则和删除规则，可选项如下：
- `CASCADE`:级联删除，删除被关联数据时，从表关联的数据全部删除。
- `SET NULL`:从父表删除或更新行，会设置子表中的外键列为NULL，但必须保证子表没有指定 NOT NULL，也就是说子表的字段可以为NULL才行。
- `RESTRICT`:如果从父表删除主键，如果子表引用了，则拒绝对父表的删除或更新操作。(保护数据)
- `NO ACTION`:表中SQL的关键字，在MySQL中与RESTRICT相同。拒绝对父表的删除或更新操作。  

现有如下关系表
```sql
CREATE TABLE `departments` (
  `dept_no` char(4) NOT NULL,
  `dept_name` varchar(40) NOT NULL,
  PRIMARY KEY (`dept_no`),
  UNIQUE KEY `dept_name` (`dept_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `employees` (
  `emp_no` int(11) NOT NULL,
  `birth_date` date NOT NULL,
  `first_name` varchar(14) NOT NULL,
  `last_name` varchar(16) NOT NULL,
  `gender` enum('M','F') NOT NULL,
  `hire_date` date NOT NULL,
  PRIMARY KEY (`emp_no`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

CREATE TABLE `dept_emp` (
  `emp_no` int(11) NOT NULL,
  `dept_no` char(4) NOT NULL,
  `from_date` date NOT NULL,
  `to_date` date NOT NULL,
  PRIMARY KEY (`emp_no`,`dept_no`),
  KEY `dept_no` (`dept_no`),
  CONSTRAINT `dept_emp_ibfk_1` FOREIGN KEY (`emp_no`) REFERENCES `employees` (`emp_no`) ON DELETE CASCADE,
  CONSTRAINT `dept_emp_ibfk_2` FOREIGN KEY (`dept_no`) REFERENCES `departments` (`dept_no`) ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```
创建对应的映射实体类
```python
import sqlalchemy
from sqlalchemy.ext import declarative
from sqlalchemy import Column, String, Integer, Date
from sqlalchemy.orm import sessionmaker
import enum

db_url = 'mysql+pymysql://dahl:123456@10.0.0.13:3306/test'
engine = sqlalchemy.create_engine(db_url, echo=True)

Base = declarative.declarative_base()
DBSession = sessionmaker(bind=engine)
session = DBSession()

class Departments(Base):
    __tablename__ = 'departments'
    dept_no = Column(String(4), nullable=False, primary_key=True)
    dept_name = Column(String(40), nullable=False, unique=True)

    def __repr__(self):
        return '<{} {} {}>'.format(self.__class__.__name__, self.dept_no, self.dept_name)

class GenderEnum(enum.Enum):
    M = 'M'
    F = 'F'

class Employees(Base):
    __tablename__ = 'employees'
    emp_no = Column(Integer, nullable=False, primary_key=True)
    birth_date = Column(Date, nullable=False)
    first_name = Column(String(14), nullable=False)
    last_name = Column(String(16), nullable=False)
    gender = Column(sqlalchemy.Enum(GenderEnum), nullable=False)
    hire_date = Column(Date, nullable=False)

    def __repr__(self):
        return '<{} {} {} {} {} {}>'.format(
            self.__class__.__name__, self.emp_no, self.birth_date, self.first_name, self.last_name, self.gender,
            self.hire_date
        )

class Dept_emp(Base):
    __tablename__ = 'dept_emp'
    emp_no = Column(Integer, sqlalchemy.ForeignKey(Employees.emp_no, ondelete='CASCADE'), primary_key=True, )
    dept_no = Column(String(4), sqlalchemy.ForeignKey(Departments.dept_no, ondelete='CASCADE'), nullable=False)
    from_date = Column(Date, nullable=False)
    to_date = Column(Date, nullable=False)

    def __repr__(self):
        return '<{} emp_no={} dept_no={}>'.format(
            self.__class__.__name__, self.emp_no, self.dept_no
        )

def getres(emps):
    for emp in emps:
        print(emp)
```
查询10010员工所在的部门编号及员工信息

##### 隐式连接
```python
emps = session.query(Employees, Dept_emp).filter(and_(Employees.emp_no == Dept_emp.emp_no, Employees.emp_no == '10010')).all()
getres(emps)

emps = session.query(Employees, Dept_emp).filter(Employees.emp_no == Dept_emp.emp_no).filter(Employees.emp_no == '10010').all()
getres(emps)


# 相当于:select * from employees,dept_emp where employees.emp_no = dept_emp.emp_no and employees.emp_no = '10010';
```

##### join连接
默认情况是是inner join，当然还可以选择outjeroin(left join)。
```python
# join连接
std_list = session.query(Employees).join(Dept_emp,Employees.emp_no == Dept_emp.emp_no).filter(Employees.emp_no == '10010')
getres(std_list)

std_list = session.query(Employees).join(Dept_emp).filter((Employees.emp_no == Dept_emp.emp_no) & (Employees.emp_no == '10010'))
getres(std_list)

# 如果query中，仅列出一个表明，相当于
# select employees.* from employees inner join dept_emp on employees.emp_no = dept_emp.emp_no where employees.emp_no = '10010';
```
为什么只显示了员工信息，部门信息呢？

#### 3.7.7.7 relationship
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果没有relationship，我们想要查询一个员工的信息以及他的部门信息，那么只能通过管理的方式将两个表连接起来，并且必须要显示的查询这两个表的所有信息才可以。relationship呢，就是通过在一个映射类中增加一个属性，该属性用于表示连接关系，可以在结果中，访问该属性来访问关联的表信息
```python
deparments = relationship('Dept_emp')
```
回到刚刚的问题，为employees添加属性后，查询结果如下
```python
std_list = session.query(Employees).join(Dept_emp,Employees.emp_no == Dept_emp.emp_no).filter(Employees.emp_no == '10010')
for std in str_list:
    print(std)
```
我们看到结果，依旧一样，但这时的std会存在一个deparments属性，它里面存放的就是关联的dept_emp的表信息
```python
std_list = session.query(Employees).join(Dept_emp,Employees.emp_no == Dept_emp.emp_no).filter(Employees.emp_no == '10010')
for std in str_list:
    print(std.deparments)    # [<Dept_emp emp_no=10010 dept_no=d004>, <Dept_emp emp_no=10010 dept_no=d006>]

# 或者
emp = session.query(Employees).join(Dept_emp,Employees.emp_no == Dept_emp.emp_no).filter(Employees.emp_no == '10010').first()
print(emp.departments[0].dept_no)  # d004
print(emp.departments[1].dept_no)  # d006
``` 
为什么呢？当我们执行：print(emp.departments) 来获取当前对象的departments时，
- 会再次执行SQL语句，通过结果集emp的emp_no,在dept表中查找信息
- 等于 select * from dept_emp where dept_emp.emp_no = 10011

relationship为子类添加了一个新的关联属性，可以跨表查询，它还有一个参数用于反向提供服务
- `backerf`：用于在被关联的表中插入新的字段属性，来反向来接本身

```python
class Employees:
    ... ...
    departments = relationship('Dept_emp',backref='employess')
```
动态的向Dept_emp表中，注入employess属性，用于提供相反的服务，即通过Dept_emp.employess来访问Employees表中的信息
```python
deps = session.query(Dept_emp).filter(Dept_emp.dept_no == 'd004')
for dep in deps:
    print(dep.employees.first_name)
```