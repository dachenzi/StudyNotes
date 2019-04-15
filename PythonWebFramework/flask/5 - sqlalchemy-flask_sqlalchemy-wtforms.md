# 1 sqlalchemy
关于sqlalchemy的用法请参考：[sqlalchemy用法点这里](https://www.cnblogs.com/dachenzi/p/10569495.html)

# 2 flask_sqlalchemy
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Flask-SQLAlchemy 是一个为您的 Flask 应用增加 SQLAlchemy 支持的扩展。它需要 SQLAlchemy 0.6 或者更高的版本。它致力于简化在 Flask 中 SQLAlchemy 的使用，提供了有用的默认值和额外的助手来更简单地完成常见任务。
[官方文档点这里](http://www.pythondoc.com/flask-sqlalchemy/config.html#id2)

下面是一个标准的falsk程序的目录结构：
```bash
cmdb
├── cmdb
│   ├── __init__.py   # 创建db，create_app函数
│   ├── models.py   # orm映射文件
│   ├── static
│   ├── templates
│   └── views   # views视图函数
│       ├── __init__.py
│       ├── account.py
│       └── home.py
├── create_table.py   # 用于创建数据库表
├── manage.py    # 管理，启动app
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


# 3 wtforms
WTForms是一个支持多个web框架的form组件，主要用于对用户请求数据进行验证。和django的froms相同。

## 3.1 用户登陆
```python
from flask import Flask, render_template, request, redirect
from wtforms import Form
from wtforms.fields import core
from wtforms.fields import html5
from wtforms.fields import simple
from wtforms import validators
from wtforms import widgets

class LoginForm(Form):
    name = simple.StringField(
        label='用户名',   # 显示的标签名称
        validators=[     # 验证规则
            validators.DataRequired(message='用户名不能为空.'),   # 必填项
            validators.Length(min=6, max=18, message='用户名长度必须大于%(min)d且小于%(max)d')
        ],
        widget=widgets.TextInput(),  # 输入框类型
        render_kw={'class': 'form-control'}   # 添加属性

    )
    pwd = simple.PasswordField(
        label='密码',
        validators=[
            validators.DataRequired(message='密码不能为空.'),
            validators.Length(min=8, message='用户名长度必须大于%(min)d'),
            validators.Regexp(regex="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[$@$!%*?&])[A-Za-z\d$@$!%*?&]{8,}",
                              message='密码至少8个字符，至少1个大写字母，1个小写字母，1个数字和1个特殊字符')  # 正则匹配

        ],
        widget=widgets.PasswordInput(),    # 密码框类型
        render_kw={'class': 'form-control'}  
    )
# 建议字段名称和数据库字段相同，因为可以直接将用户提交的数据更新到数据库中
```
对应的视图函数：
```python
@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        form = LoginForm()   # 将form对象传递到前端渲染
        return render_template('login.html', form=form)
    else:
        form = LoginForm(formdata=request.form)  # 把用户的数据交给form验证
        if form.validate():   # 如果可用
            print('用户提交数据通过格式验证，提交的值为：', form.data)
        else:
            print(form.errors)   # 不可用的提示信息
        return render_template('login.html', form=form) 
```
前端渲染：
```html
<form method="post">
    <!--<input type="text" name="name">-->
    <p>{{form.name.label}} {{form.name}} {{form.name.errors[0] }}</p>

    <!--<input type="password" name="pwd">-->
    <p>{{form.pwd.label}} {{form.pwd}} {{form.pwd.errors[0] }}</p>
    <input type="submit" value="提交">
</form>
```

## 3.2 用户注册
```python
from flask import Flask, render_template, request, redirect
from wtforms import Form
from wtforms.fields import core
from wtforms.fields import html5
from wtforms.fields import simple
from wtforms import validators
from wtforms import widgets

app = Flask(__name__, template_folder='templates')
app.debug = True



class RegisterForm(Form):
    name = simple.StringField(
        label='用户名',
        validators=[
            validators.DataRequired()
        ],
        widget=widgets.TextInput(),
        render_kw={'class': 'form-control'},
        default='daxin'  # 默认值
    )

    pwd = simple.PasswordField(
        label='密码',
        validators=[
            validators.DataRequired(message='密码不能为空.')
        ],
        widget=widgets.PasswordInput(),
        render_kw={'class': 'form-control'}
    )

    pwd_confirm = simple.PasswordField(
        label='重复密码',
        validators=[
            validators.DataRequired(message='重复密码不能为空.'),
            validators.EqualTo('pwd', message="两次密码输入不一致")
        ],
        widget=widgets.PasswordInput(),
        render_kw={'class': 'form-control'}
    )

    email = html5.EmailField(   # 包含邮箱的验证规则
        label='邮箱',
        validators=[
            validators.DataRequired(message='邮箱不能为空.'),
            validators.Email(message='邮箱格式错误')
        ],
        widget=widgets.TextInput(input_type='email'),
        render_kw={'class': 'form-control'}
    )

    gender = core.RadioField(
        label='性别',
        choices=(
            (1, '男'),
            (2, '女'),
        ),
        coerce=int   # 数字类型的转换方式
    )
    city = core.SelectField(
        label='城市',
        choices=(
            ('bj', '北京'),
            ('sh', '上海'),
        )
    )

    hobby = core.SelectMultipleField(
        label='爱好',
        choices=(
            (1, '篮球'),
            (2, '足球'),
        ),
        coerce=int
    )

    favor = core.SelectMultipleField(
        label='喜好',
        choices=(
            (1, '篮球'),
            (2, '足球'),
        ),
        widget=widgets.ListWidget(prefix_label=False),
        option_widget=widgets.CheckboxInput(),
        coerce=int,
        default=[1, 2]
    )

    def __init__(self, *args, **kwargs):
        super(RegisterForm, self).__init__(*args, **kwargs)
        self.favor.choices = ((1, '篮球'), (2, '足球'), (3, '羽毛球'))  

    def validate_pwd_confirm(self, field):
        """
        自定义pwd_confirm字段规则，例：与pwd字段是否一致
        :param field: 
        :return: 
        """
        # 最开始初始化时，self.data中已经有所有的值

        if field.data != self.data['pwd']:
            # raise validators.ValidationError("密码不一致") # 继续后续验证
            raise validators.StopValidation("密码不一致")  # 不再继续后续验证


@app.route('/register', methods=['GET', 'POST'])
def register():
    if request.method == 'GET':
        form = RegisterForm(data={'gender': 1})
        return render_template('register.html', form=form)
    else:
        form = RegisterForm(formdata=request.form)
        if form.validate():
            print('用户提交数据通过格式验证，提交的值为：', form.data)
        else:
            print(form.errors)
        return render_template('register.html', form=form)



if __name__ == '__main__':
    app.run()
```
注意：choices字段和django中的form是一样的，如果数据来自于数据库，那么只能在第一次实例化的时候加载，无法做的实时更新，所以处理方法也是相同的，放在__init__中，每次实例化的时候更新。


for循环渲染多个不同的标签：
```python
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>用户注册</h1>
<form method="post" novalidate style="padding:0  50px">
    {% for item in form %}
    <p>{{item.label}}: {{item}} {{item.errors[0] }}</p>
    {% endfor %}
    <input type="submit" value="提交">
</form>
</body>
</html>

```