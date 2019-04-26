
<!-- TOC -->

- [1 路由系统(urls控制)](#1-路由系统urls控制)
    - [1.1 正则字符串参数](#11-正则字符串参数)
    - [1.2 url的分组](#12-url的分组)
        - [1.2.1 无名分组](#121-无名分组)
        - [1.2.2 有名分组](#122-有名分组)
    - [1.3 URLconf 在什么上查找](#13-urlconf-在什么上查找)
    - [1.4 include(路由分发)](#14-include路由分发)
    - [1.5 别名(name参数)](#15-别名name参数)
    - [1.6 反推URL](#16-反推url)
    - [1.7 命名空间](#17-命名空间)

<!-- /TOC -->
# 1 路由系统(urls控制)
url控制其实就是把不同的url对应到不同的views函数中去

格式：
```python
# 项目目录下的urls.py文件中

urlpatterns = [
    url(regex, view, kwargs=None, name=None)
    ... ...
]
```
url可以有多个，每个url都是一个独立的规则。
参数如下：
- `regex`(url正则表达式):与之匹配的 URL 会执行对应的第二个参数 view。
- `view`(views视图函数): 用于执行与正则表达式匹配的 URL 请求。
- `kwargs`(参数列表): 视图使用的字典类型的参数。  --> 很少使用
- `name`(别名): 用来反向获取 URL。

## 1.1 正则字符串参数
url的第一个参数为正则表达式，所以常用的正则表达式符号都可以进行匹配
```python
from user import views  # 导入我们的app的views，才可以调用
urlpatterns = [
    url(r'^articles/2003/$', views.special_case_2003),
    url(r'^articles/[0-9]{4}/$', views.year_archive), 
    url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.month_archive), # 传递两个位置参数
    url(r'^articles/([0-9]{4})/([0-9]{2})/([0-9]+)/$', views.article_detail),  # 传递三个位置参数
]
```
这里对url表达式使用括号，表示取出匹配到的路径字符串（分组）


注意：
- 一旦`匹配成功则不再继续匹配`
- `不需要添加一个前导的反斜杠`，因为每个URL 都有。例如，应该是^articles 而不是 ^/articles。
- 每个正则表达式前面的'r'，是可选的但是建议加上。

## 1.2 url的分组
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在url正则表达式上加上括号，就表示对括号内的元素进行取出，然后传给后面的views函数，根据传递参数的方式不同，分为`无名分组`和`有名分组`:
> 若要从URL中捕获一个值，只需要在它周围放置一对圆括号。

### 1.2.1 无名分组
只需要把要分组的正则字符串用括号括起来即可。这样，括号内的匹配的内容会当作位置参数传递给后面指定的views函数。
```python
urlpatterns = [
    url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.year_month),
]
 
# ([0-9]{4}) 第一个位置参数
# ([0-9]{2}) 第二个位置参数
```
views函数需要定义位置参数来一一对应,否则将会抛出TypeError异常

### 1.2.2 有名分组
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;即捕获url中的一个值时，并赋予其名称，使用关键字参数来进行传递。在Python 正则表达式中，命名正则表达式组的语法是(?P<name>pattern)，其中 name 是组的名称，pattern 是要匹配的模式。
```python
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^index/(?P<year>201[0-9])', user.views.index)
]

# 会把 {'year':201[0-9]} 当作关键字参数
```
这种方式可以在views中的处理函数内，直接定义关键字变量来接受，而不用在意参数的位置。
```python
def index(request,year):
    ... ...
```
如果year没有捕获到数据，那么views函数index将会报错，所以我们一般可以在index中为year，配置默认值来避免这种错误。当然也可以在urls内，指定默认参数
```python
url(r'^index/year(?P<year>[0-9]?)', user.views.index, {'year': 2019}) 
```
但是这样就把前面匹配到的year的值给覆盖了。请慎重选用

## 1.3 URLconf 在什么上查找
请求的URL被看做是一个普通的Python 字符串， URLconf在其上查找并匹配。进行匹配时将不包括GET或POST请求方式的参数以及域名。
- GET:把用户发送的参数放在URL中进行传递，在URL中一般用?隔开。
- POST:把用户发送的参数放在请求头中传递。  

例如：
- https://www.example.com/myapp/ 请求中，URLconf 将查找 myapp/
-  https://www.example.com/myapp/?page=3 请求中，URLconf 仍将查找 myapp/ 。  

URLconf 不检查使用了哪种请求方法。换句话讲，所有的请求方法,即对同一个URL的无论是 POST请求、GET请求、或是HEAD请求方法等等，都将路由到相同的函数。

## 1.4 include(路由分发)
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当项目中的应用变得越来越多的时候，如果所有的应用的URL都通过项目的urls统一进行分配，那么耦合度会很高，另外也不利于管理，所以这里通过include来交给应用的urls来处理。
```python
# 项目的urls.py
from django.conf.urls import include, url
 
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^user/', include('user.urls')),
]
 
# 通过include 来指定交给那个应用的urls来处理
# include 包涵在 django.conf.urls中
```
在应用user下创建urls.py文件，写入
```python
from django.conf.urls import url
from user import views

urlpatterns = [
    url('^login', views.login)
]
```
注意：路由分发后，子路径的起始位置就从分发的URL开始了。上面匹配到的路径为：`user/login`

## 1.5 别名(name参数)
当我们在做路径匹配然后配合form表单等需要提交的数据的元素的时候会遇到以下问题
```python
#urls.py

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^login/', views.login),      #  匹配路径
]
```
返回的页面文件：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
 
 
<h1>登录页面</h1>
<form action="/login/" method="post">     <!--   根据路径进行提交 -->
    <p><h2>姓名</h2></p>
    <p><input type="text" name="username"></p>
    <p><h2>密码</h2></p>
    <p><input type="text" name="password"></p>
    <p><input type="submit" value="登录"></p>
</form>
 
</body>
</html>
```
如果我们某一天改了url匹配的路径，那么,就绪要修改页面中所有的提交路径，否则将会提交失败.而url的name就是解决这个问题的。
```python
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^login/$', views.login,name='LOGIN'),  # 添加name关键字
]
```
返回的html文件如下
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
 
 
<h1>登录页面</h1>
<form action="{% url 'LOGIN' %}" method="post">      <!-- 这里使用name关键字来动态的传递url -->
    <p><h2>姓名</h2></p>
    <p><input type="text" name="username"></p>
    <p><h2>密码</h2></p>
    <p><input type="text" name="password"></p>
    <p><input type="submit" value="登录"></p>
</form>
 
</body>
</html>
```
这样就算URL后期更改，也会动态指向最新的URL。
- 个人觉得说白了就是一个变量的替换，只不过是用的是django特有的语法格式。
name='LOGIN' 就是把前面匹配的url保存到LOGIN中，然后Django 在返回html文件的时候再进行替换。
补充：
- 当url存在正则表达式的时候，只使用name参数是不行的。因为正则部分无法进行渲染。目前的解决方法是在url部分进行拼接
```python
# --------------------------- url ---------------------------
from django.conf.urls import url
from django.contrib import admin
from app01 import views
 
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^abc/(\d+)/', views.login,name="LOGIN"),
]
 
 
 
# -------------------------- Teamplates ---------------------------
 
<form action="{% url "LOGIN" 4 %}">     # 这里正则匹配了几个部分，那么就需要传递几个参数
    <p>Username:</p>
    <input type="text" name="username">
    <p>Password:</p>
    <input type="text" name="password">
    <input type="submit" value="提交">
</form>
```

## 1.6 反推URL
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;什么叫反推url？在views函数中，如果想要获取当前函数对应的url，该怎么办呢？还记得前面的name属性吗，反推url就是在views中根据name属性的值，获得对应的url
```python
from django.urls import reverse
 
url = reverse('name')
```
当然reverse还有两个参数(args,kwargs)
```python
args = ()
kwargs = {}
 
# 参数是配合urls中的正则表达式的
url('^detail/(\d+)' ,name='i1',views.detail)          -- >   reverse('i1',args=(10,))
# 反推的URL为： detail/10
 
# kwargs则表示在命名关键字的情况下
url('^detail/(?P<nid>\d+)' ,name='i2',views.detail)   -- >   reverse('i1',kwargs={'nid':10})
# 反推的url为： detail/10
```
还有一个方法是利用request对象的path_info，因为其中存放的是用户提交的url。

## 1.7 命名空间
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当在url(路由系统)中使用了include时，在views函数中，我们就无法单独的利用name参数来反推url了，因为在include时，是无法使用name关键字的，不过django提供了其他关键字提供类似功能：namespace，称作命名空间。
```python
# urls.py
 
url = [
    url(r'crm/',include('crm.urls'),namespace='crm'),
    url(r'cmdb/',include('cmdb.urls'),namespace='cmdb'),
]
 
# crm 中的 urls.py
 
app_name = crm
url = [
    url(r'index/',views.index,name='index'),
]
 
# crm 中的views.py
 
def test(request):
    url = reverse('crm:index')    # 这里通过namespace:name 来反向获取url
    print(url)
    return HttpResponse(200)
```
PS：在html中，通过{% url 'crm:index' %} 也是通过namespace：name来获取url的。

