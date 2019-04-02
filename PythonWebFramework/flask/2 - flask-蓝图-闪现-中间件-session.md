# 1 flask蓝图
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;为了在一个或多个应用中，使应用模块化并且支持常用方案，Flask引入了`蓝图`概念。蓝图可以极大地简化大型应用并为扩展提供集中的注册入口。 `Blueprint对象`与 Flask 应用对象的工作方式类似，但不是一 个真正的应用。它更像一个用于构建和扩展应用的蓝图 。

蓝图的用途：
1. 把一个应用分解为一套蓝图。这是针对大型应用的理想方案：一个项目可以实例化几个应用，初始化多个扩展，并注册许多蓝图。
2. 在一个应用的 URL 前缀和（或）子域上注册一个蓝图。 URL 前缀和（或）子域的参数成为蓝图中所有视图的通用视图参数（缺省情况下）。
3. 使用不同的 URL 规则在应用中多次注册蓝图。
4. 通过蓝图提供模板过滤器、静态文件、模板和其他工具。蓝图不必执行应用或视图 函数。
5. 当初始化一个 Flask 扩展时，为以上任意一种用途注册一个蓝图。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在蓝图被注册到应用之后，当分配请求时，Flask 会把蓝图和视图函数关联起来，并生成两个端点之前的URL 。

## 1.1 第一个蓝图
蓝图的标准目录结构如下：
```python
flask_app
├─manage.py
└─flask_app
	├─__init__.py
    ├─static
    ├─templates
    └─views   # 视图函数存放的位置
    	├─account.py
    	└─content.py
```
要求在__init__.py文件中去实例化flask对象
```python
from flask import Flask

app = Flask(__name__)

from .views.acount import account
from .views.content import content

app.register_blueprint(account)  # 注册蓝图
app.register_blueprint(content)  # 注册蓝图
```
> 注册蓝图时，也可以添加`url_prefix='/pages'`来指定当前蓝图的前缀。  

在manage.py来做启动管理
```python
from flask_app import app

if __name__ == '__main__':
    app.run()
```
在我们的views的模块中，我们需要通过blueprint对象来注册路由
```python
# user.py
from flask import Blueprint

account = Blueprint('account', __name__)  # 创建蓝图对象

@account.route('/login')   # 添加蓝图管理的视图函数
def login():
    return 'login'

@account.route('/logout')  # 添加蓝图管理的视图函数
def logout():
    return 'logout'
```
可以把blueprint对象理解为代理（也可以理解为django中的app的概念)，使用蓝图以后的执行逻辑为：
1. 创建蓝图对象
2. 使用不同的蓝图对象来聚合不同类型的视图函数
3. 将蓝图注册给app管理

## 1.2 before_request/after_request
在使用蓝图的情况下，before_request/after_request装饰器，就可以选择针对不同的蓝图来调用，还是针对全局来调用。以上面的结构为例：
1. 定义在__init__.py中就是全局执行
2. 在某个蓝图对象中，调用before_request/after_request就是针对这个蓝图对象来执行的。

针对全局(在__init__.py文件中):
```python
@app.before_request
def auth():
    print('auth')

```

针对蓝图对象(在对应的views函数中):
```python
# 以content.py的content蓝图对象为例
@account.before_request
def check():
    print('check')
```
执行顺序为：先执行全局before_request，然后执行蓝图before_request，先执行蓝图after_request，然后执行全局after_request

## 1.3 蓝图配置
`url_prefix`: 为蓝图添加前缀
```python
from flask import Blueprint

account = Blueprint('account', __name__,url_prefix='/account')  # 创建蓝图对象

@account.route('/login')   # 添加蓝图管理的视图函数
def login():
    return 'login'

@account.route('/logout')  # 添加蓝图管理的视图函数
def logout():
    return 'logout'
```
访问当前蓝图接管的视图函数时，需要添加前缀/account才可以，比如/login,就是/account/login

`static_folder`: 蓝图的静态文件存放路径
`static_url_path`: 蓝图静态文件的访问路径
`template_folder`: 蓝图的模板文件存放路径

# 2 消息闪现
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;一个好的应用和用户界面都需要良好的反馈。如果用户得不到足够的反馈，那么应用 最终会被用户唾弃。 Flask 的闪现系统提供了一个良好的反馈方式。闪现系统的基 本工作方式是：在且只在下一个请求中访问上一个请求结束时记录的消息。一般我们 结合布局模板来使用闪现系统。
> 注意，浏览器会限制 cookie 的大小，有时候网络服 务器也会。这样如果消息比会话 cookie 大的话，那么会导致消息闪现静默失败。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;比如：我们在上一个函数中执行了错误的操作，那么直接在当前页面报错可能不是特别友好，更多的用法是跳转到特定的错误页面，然后将异常信息输出。通常我们使用的方式是通过session传递，产生错误时，将信息设置到session中，跳转到错误页面时，通过session将信息取出显示完毕后，删除session中的数据(如果只用一次的话)

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;消息闪现就是基于session的一次性消息的注入与提取，主要使用两个函数实现
- `flash(message,category='message')`:设置消息,category可以为消息设置分类(可以不分类)
- `get_flashed_messages(category_filter=[])`：可以通过指定类别来获取消息(可以不指定过滤)

```python
@account.route('/logout')
def logout():
    flash('用户退出')  # 等同于session['message'] = '用户退出'
    return redirect('/info')

@account.route('/info')
def info():
    data = get_flashed_messages(category_filter='message')  # 等同于session.pop('message')
    return Markup('<h1>{}</h1>'.format(data[0]))
```

# 3 中间件
django中的中间件可以用于登录验证，IP限制的问题，flask的中间件实现原理不同，看下面的列子：
```python
if __name__ == '__main__':
    app.run()  # 这里的run，最终执行的是wsgi_app
```
那么我们如何使用中间件呢，需要在接受请求之前来执行我们的代码，run函数实际上调用的是对象的wsgi_app，那么我们改写wsgi_app即可。
```python
class Middleware(object):
	def __init__(applciation):
		self.app = application

	def __call__(self,*args,**kwargs):
		... ...  # 可以定制的代码
		... ...
		return self.app(*arge, **kwargs)


if __name__ == '__main__':
	app.wsgi_app = Middleware(app.wsgi_app)
    app.run()  # 这里的run，最终执行的是wsgi_app
```
需要注意的是，__request对象是在wsgi_app执行的时候才会产生的，所以我们这里是没办法直接使用request对象的。__

# 4 flask-session插件
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;flask-session插件用于处理session敏感度的问题，如果加密存放在客户端，那么有一定几率被反解，一般情况下，我们会把session放在redis/memcache等nosql中去，使用flask-session可以帮我们简化这个过程。

## 4.1 安装
首先需要安装如下模块
- flask-session:扩展插件
- redis:Python的redis驱动程序

```python
pip install flask-session
pip install redis
```

## 4.2 将session放在redis中
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;flask比较厉害的一点，就是想要更换redis存储的位置，那么代码根本不需要修改，只需要添加下面redis的配置即可
```python
# 在创建app的地方进行配置

from flask.ext.session import Session 
from redis import Redis  

app.config['SESSION_TYPE'] = 'redis'
app.config['SESSION_REDIS'] = Redis(host='10.0.0.13',port='6379')
Session(app) 
```
问题:
1. 新版本已经无法通过flask.ext.session来导入Session对象了，需要使用flask_session来导入Session
2. 如果redis存在密码，那么只需要给Redis对象添加password属性即可

redis中存储的session格式：（存放的是session的id,利用uuid生成)
```bash
10.0.0.13:6379> keys *
1) "session:857c79dc-3871-4a89-9737-ce6b4aeea3b8"
2) "session:48c714d0-9c54-434b-b43a-70a53c1e0056" 
```

## 4.3 为什么可以直接使用
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;还记得的我们session生成的过程吗？当用户第一次访问时，如果session为空，那么会使用SecureCookieSessionInterface()为我们生成一个session(空字典)，然后调用它的open_session方法来处理生成新的session。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在使用前我们使用Session(app)包装了一下我们的app，它做了什么？
- 利用init_app初始化app
- 将app的session_interface替换为Session对象的_get_interface(app)的返回值
- 内部将system_interface替换成了RedisSessionInterface对象的实例
- 所以再执行open_session方法的时候，其实已经被换成了RedisSessionInterface的open_session方法
- 在它们内部处理redis相关的读取操作

> 利用uuid生成的随机唯一ID，通过cookie写到客户端，真正保存的信息，存放在redis中