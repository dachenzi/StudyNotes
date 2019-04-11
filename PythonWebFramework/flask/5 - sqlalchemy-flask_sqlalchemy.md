# 1 sqlalchemy
关于sqlalchemy的用法请参考：[sqlalchemy用法点这里](https://www.cnblogs.com/dachenzi/p/10569495.html)

# 2 flask_sqlalchemy
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Flask-SQLAlchemy 是一个为您的 Flask 应用增加 SQLAlchemy 支持的扩展。它需要 SQLAlchemy 0.6 或者更高的版本。它致力于简化在 Flask 中 SQLAlchemy 的使用，提供了有用的默认值和额外的助手来更简单地完成常见任务。
[官方文档点这里](http://www.pythondoc.com/flask-sqlalchemy/config.html#id2)

例子：
```bash
cmdb
├── cmdb
│   ├── __init__.py
│   ├── models.py
│   ├── static
│   ├── templates
│   └── views
│       ├── __init__.py
│       ├── account.py
│       └── home.py
├── create_table.py   # 用于创建数据库表
├── manage.py
└── settings.py
```
其内部封装简化了很多sqlalchemy的操作，比如创建session，线程安全的scope_session，链接池等，都被封装好了，直接用就好了。


## 2.1 创建对象
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;新版本需要安装flask_sqlalchemy，并且倒入方式也和老版本有些区别，上面的官方文档我本地无法进行倒入。使用 `pip3 install flask_sqlalchemy` 安装
```python
# cmdb.__init__.py
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()
```
注意，

## 2.2 初始化init_app
在一个函数中定义的应用，SQLAlchemy 对象是全局的，后者如何知道前者了？答案就是 init_app()函数
```python
# cmdb.__init__.py
from flask_sqlalchemy import SQLAlchemy
import Flask

db = SQLAlchemy()

def create_app():
    app = Flask(__name__)
    db.init_app(app)  # 需要把app传入，这样SQLAlchemy在内部就可以读取app中的相关配置，比如sql的driver，host等等
```
关于sqlalchemy的配置，这样就可以直接写在app的settings.py中了，可以配置的项目参考:[sqlalchemy配置](http://www.pythondoc.com/flask-sqlalchemy/config.html)

比如：
```python
# settings.py

class Base(object):
    SECRET_KEY = '121212asdsd121dafd'
    SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://dahl:123456@10.0.0.10:3306/test'
    SQLALCHEMY_POOL_SIZE = 5
```

## 2.3 上下文管理
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;我们写的model如何创建数据库表呢？在sqlalchemy中，需要使用base.metadata.create_all()，在flask_sqlalchey中，简化了这个过程。利用db.create_all就可以完成创建,注意需要在db所在的地方，引入编写的model的
```python
# cmdb.__init__

from flask import Flask
from flask_sqlalchemy import SQLAlchemy

# 实例化一个flash_sqlalchemy对象
db = SQLAlchemy()

# 只能放在下面，因为会用到db 对象
from .models import User    # 倒入编写的model文件中的类
from .views import account
from .views import home

def create_app():
    app = Flask(__name__)
    app.config.from_object('settings.Base')

    # 注册蓝图
    app.register_blueprint(account.ac)
    app.register_blueprint(home.hm)

    # 初始化
    db.init_app(app)

    return app
```
创建数据库表
```python
# create_table.py 

from cmdb import db
from manage import app

if __name__ == '__main__':
    with app.app_context():  # 需要上下文环境
        db.create_all()      # 初始化表结构
```

## 2.4 视图中使用
在views中的视图函数中，通过导入db对象，即可使用它来操作数据库
```python
# views.account.py

from flask import Blueprint
from cmdb import db
from cmdb import models

ac = Blueprint('ac', __name__)

@ac.route('/login')
def login():
    res = db.session.query(models.User).all()  # 直接使用
    print(res)
    return 'login'

@ac.route('/logout')
def logout():
    return 'logout'
```

## 2.5 关闭触发信号
启动的时候会发送一个信号警告
```
FSADeprecationWarning: SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and will be disabled by default in the future.  Set it to True or False to suppress this warning.
  'SQLALCHEMY_TRACK_MODIFICATIONS adds significant overhead and '
```
在配置文件中关闭出发的信号
```python
# settings.py
class Base(object):
    SECRET_KEY = '121212asdsd121dafd'
    SQLALCHEMY_DATABASE_URI = 'mysql+pymysql://dahl:123456@10.0.0.10:3306/test'
    SQLALCHEMY_POOL_SIZE = 5
    SQLALCHEMY_TRACK_MODIFICATIONS = False   # 添加这个配置即可。
```
