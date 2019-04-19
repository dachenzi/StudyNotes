<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 Template](#1-template)
    - [1.1 模板的基础使用](#11-模板的基础使用)
        - [1.1.1 变量](#111-变量)
        - [1.1.2 注释标签](#112-注释标签)
        - [1.1.3 深度查询](#113-深度查询)
        - [1.1.4 内置变量过滤器filter](#114-内置变量过滤器filter)
        - [1.1.5 自定义过滤器之filter](#115-自定义过滤器之filter)
        - [1.1.6 自定义过滤器之simple_tag](#116-自定义过滤器之simple_tag)
        - [1.1.7 inclusion_tag](#117-inclusion_tag)
    - [1.2 逻辑控制语法](#12-逻辑控制语法)
        - [1.2.1 for标签](#121-for标签)
        - [1.2.2 if标签](#122-if标签)
        - [1.2.3 ifequal/ifnotequal 标签](#123-ifequalifnotequal-标签)
    - [1.3 特殊标签](#13-特殊标签)
    - [1.4 extends模板继承](#14-extends模板继承)
    - [1.5 include引入](#15-include引入)

<!-- /TOC -->
# 1 Template
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用 Django的 模板系统 (Template System)来实现将Python代码和HTML代码分开的目的。
__`python的模板包涵：HTML代码＋逻辑控制代码`__ , 将具体数据嵌入到前端模板的语法，在一个html文件中包含模板语法的文件，可以认为是模板文件

## 1.1 模板的基础使用
主要分为两部分：渲染变量使用双大括号`{{ }}`，渲染标签则使用双大括号双百分号`{% %}`

### 1.1.1 变量
在html页面中使用两个大括号包起来的字符串叫做变量：
```jinja
{{ Var_name }}
```
这里通过python django的shell环境来举例（在这个环境中可以直接引用 所属django模块中的变量等信息）
```python
# 使用 manage.py进入shell界面
终端中输入：python manage.py shell
 
>>> from django.template import Context,Template
>>> t = Template('My name is {{ name }}')
>>> c = Context({'name':'dachenzi'})
>>> t.render(c)        # render会把变量进行渲染
'My name is dachenzi'
>>>
```

### 1.1.2 注释标签
单行注释 `{# #}`。
多行注释 `{% comment %} ... {% endcomment %}`.
```jinja
{# 这是一个注释 #}
{% comment %}
这是多行注释
{% endcomment %}.
```

### 1.1.3 深度查询
如果传递的变量是属组或者字典，那么需要进行深度查询(通过使用.(点)来完成深度查询)：
```jinja
{{ Var_Name.username }}    # 获取Var_Name中的key为：username的值 --> 变量为字典类型
{{ Var_Name.2 }}      # 获取Var_Name中第2个值 --> 变量为list类型
```
使用.带进行深度查询，查询list类型的数据
```python
>>> from django.template import context,Template
>>> t = Template('hello {{ name.1}}')
>>> c = Context({'name':[1,2,3]})
>>> t.render(c)
'hello 2'
```
使用`items`循环字典类型(不加括号)
```python
{% for key,value in user_dict.items %}
<li>{{  key }}-{{ value }}</li>
{% endfor %}
```

### 1.1.4 内置变量过滤器filter
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;可以理解为python中的内置函数，过滤器是模板的特有语法，通过前端来过滤部分数据。注意filter只能传递一个参数(也可以说是两个参数，因为第一个个参数已经固定，就是被处理的那个)。
格式：
```jinja
{{ var|method:parameter}}
```
method表示过滤器部分过滤器如下：

过滤器|说明|举例
---|-----|----|
first|取列表第一个元素
last|取列表最后元素
capfirst|首字母大写|
cut|从字符串中移除指定的字符|{{ 'aLbLcL'\|cut:'L' }}答案是abc
yesno|变量可以是True、False、None，yesno的参数<br>给定逗号分隔的三个值，返回3个值中的一个。<br>True对应第一个<br>False对应第二个<br>None对应第三个<br>如果参数只有2个，None等效False处理| {{ value \| yesno:"yeah,no,maybe"}}
add|加法。参数是负数就是减法 |数字加{{ value \| add:"100"}}<br>列表合并{{mylist \| add:newlist}}
divisibleby|能否被整除 {{ value \| divisibleby:"3" }}能被3整除返回True
addslashes|在反斜杠、单引号或者双引号前面加上反斜杠 {{ value \| addslashes }}
length|返回变量的长度| {% if my_list \| length > 1 %}
default|变量等价False则使用缺省值|{{ value \| default:"nothing" }}
default_if_none|变量为None使用缺省值|{{ value \|default_if_none:"nothing" }}
date|格式化 date 或者 datetime 对象|实例：{{my_dict.date\|date:'Y n j'}}<br>Y 2000 年<br>n 1~12 月<br>j 1~31 日
切片相关|{{ value7\|filesizeformat }}文件的字节数<br>{{ value7\|first }}第一个字符<br> {{ value7\|length }}长度<br> {{ value7\|slice:":-1" }}切片<br> 

### 1.1.5 自定义过滤器之filter
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;内置的过滤器总有不能满足我们业务需求的时候，这时，我们可以自定义过滤器，官方文档：https://docs.djangoproject.com/zh-hans/2.0/howto/custom-template-tags/

具体步骤为：
1. 在`app目录下`创建templatetags包(`名称必须为templatetags`)
2. 在包下创建任意 .py 文件，如：my_tags.py ，用来存放自定义的filter或tag函数
3. 编写自定义过滤器
4. 在模板中加载templatetags并调用

下面是my_tags.py的文件内容
```python
from django import template # 需要先导入文件
 
register = template.Library() # register的名字是固定的,不可改变,因为下面会进行调用
 
@register.filter # 注册
def multi(num1,num2):
return num1 * num2
```
在引用的html模板文件中进行加载(可以放在模板文件头部，或者用之前加载也可以，当和extend同时使用时，放在entend下面)
```jinja
{% load my_tags %} # 这里的名字就是我们创建的my_tags.py的文件名
```
调用
```jinja
{{ hello|multi:2 }}    # hello 表示传递的第一个参数，2表示传递的是第二个参数，冒号后面必须要紧跟传递的参数，多一个空格都不行
```
PS：filter 的函数只能接受一个额外参数(两个参数，默认会把要处理的对象当作第一个参数，第二个参数是我们需要传递的参数),可以用于if/for等语句的条件，渲染时 使用 {{  参数1 | xxx:参数2  }} 渲染 。

### 1.1.6 自定义过滤器之simple_tag
simpletag与filter不同的是，可以接受多个参数，但不可以用if/for等语句的条件，使用 {% xxx %} 渲染，创建步骤与filter相同

编写simpletag
```python
# my_tags.py文件

from django import template # 需要先导入文件
 
register = template.Library() # register的名字是固定的,不可改变,因为下面会进行调用

@register.simple_tag   # 必须要装饰我们自定义的函数才行
def simple_tag_multi(num1,num2):
return num1 * num2　
```
加载完毕后调用方式使用 {% %}
```python
{% simple_tag_multi 10 20 %}  # 参数以空格隔开，多个空格效果相同
```

### 1.1.7 inclusion_tag 
存放于：django.template.Library.inclusion_tag()  

主要作用：通过渲染一个模板来显示一些数据。   

> 例如，Django的Admin界面使用自定义模板标签显示"添加/更改"表单页面底部的按钮。这些按钮看起来总是相同，但链接的目标却是根据正在编辑的对象而变化的。 这种类型的标签被称为"Inclusion 标签"，属于自定义标签的一种。

```python
from django.template import Library

register = Library()

@register.inclusion_tag('menu_tpl.html')
def menu_dict():
    menu = {
        'option1': '首页',
        'option2': '菜单一',
        'option3': '菜单二',
        'option4': '菜单三',
        'option5': '菜单四'
    }

    return {'menu_dict': menu}

# 将函数的返回值，使用menu_tpl.html进行渲染，然后将渲染的结果当作menu_dict的返回值，在页面上调用时，整体渲染。
```
menu_tpl.html文件如下：（存放于templates目录下)
```html
{% for key,value in menu_dict.items %}
    <a href="" class="{{ key }}">{{ value }}</a>
{% endfor %}
```
在templates模版中调用时
```html
{% load rbac %}   我templatestags下存放的名称为 rbac，所以这里写文件名即可。
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>index</h1>
<div>

    {% menu_dict %}   直接渲染即可
</div>
</body>
</html>

```



## 1.2 逻辑控制语法
循环等逻辑语句需要使用{% %} 来进行渲染

### 1.2.1 for标签
```jinja
{% for i in li %}  # li 为后端传递给模板的变量
<p>{{ i }}</p>
{% endfor %}       #使用endfor来表示循环结束
```
在for循环中存在一个forloop变量，该变量记录循环相关数据
- forloop.counter：表示当前迭代数（第几次循环）从1开始
- forloop.counter0：同上，但是从0开始
- forloop.first：判断此次循环是否是第一次循环，是则返回True
- forloop.revcounter：表示当前迭代数（第几次循环）- 倒序
- forloop.last：判断此次循环是否是最后一次循环
- forloop.parentloop：用于在嵌套循环时，获取父循环的上面的信息

reversed和empty：  
- `reversed`：对可迭代对象进行逆序
- `empty`：如果可迭代对象为空

```jinja
{% for athlete in athlete_list reversed %}
...
{% empty %}
... 如果被迭代的列表是空的或者不存在，执行empty
{% endfor %}
```

### 1.2.2 if标签
```jinja
{% if li.1 > 100 %}
<p> 大于 </p>
{% elif li.1 < 100 %}
<p> 小于 </p>
{% else %}
<p> 等于 </p>
{% endif %}         #使用endif来表示条件判断结束
```

### 1.2.3 ifequal/ifnotequal 标签
{% ifequal %} 标签比较两个值，当他们相等时，显示在 {% ifequal %} 和 {% endifequal %} 之中所有的值。
下面的例子比较两个模板变量 user 和 currentuser :
```jinja
{% ifequal user currentuser %}
 <h1>Welcome!</h1>
{% endifequal %}
```
和 {% if %} 类似， {% ifequal %} 支持可选的 {% else%} 标签
```jinja
{% ifequal section 'sitenews' %}
 <h1>Site News</h1>
{% else %}
 <h1>No News Here</h1>
{% endifequal %}
```

## 1.3 特殊标签
`{% url  'varname' %}`：引用路由配置的地址
```jinja
<form action="{% url "LOGIN"%}" method="POST">    # 路由引用
          <input type="text">
          <input type="submit" value="提交">
</form>
```
`{% verbatim %}`:禁止render解析。

既在有的时候我们仅仅只是想打印出 {{ hello }} 而已，使用verbatim来禁止render进行解析
```jinja
{% verbatim %}
    {{  hello }}
{% endverbatim %}
 
# 使用verbatim 包起来的代码 将不会进行解析
```

`{% csrf_token %}`:用于跨站请求伪造保护，防止跨站攻击的。
```jinja
<form>
{% csrf_token %}
</form>
```
一般用在form表单中。

## 1.4 extends模板继承
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;到目前为止，我们的模板范例都只是些零星的 HTML 片段，但在实际应用中，你将用 Django 模板系统来创建整个 HTML 页面。 这就带来一个常见的 Web 开发问题： 在整个网站中，如何减少共用页面区域（比如站点导航）所引起的重复和冗余代码？Django 解决此类问题的首选方法是使用一种优雅的策略 —— `模板继承`.  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;本质上来说，模板继承就是先构造一个基础框架模板，而后在其子模板中对它所包含站点公用部分和定义块进行重载.  

编写模板需要使用模板标签：  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;`{% block name%} {% endblock %}`：所有的 {% block %} 标签告诉模板引擎，子模板可以重载这些部分。 每个{% block %}标签所要做的是告诉模板引擎，该模板下的这一块内容将有可能被子模板覆盖。

下面是一个html模板文件：
```html
// index.html文件

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>

</head>
<body>

    <div class="title"></div>
    <div class="left">
        <div class="left_list">
            <a href="/booklist/">文章列表</a>
        </div>

    </div>
    <div class="right">
        {% block content %}   // 这里定义content，表示这里可以被继承模板进行替换
        {% endblock %}    // 包起来的表示代码段
    </div>

</body>
</html>
```
> HTML中包含 block 语句块的都可以称为模板文件  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;子模板进行继承，并定制自己要显示的内容(子模板不需要额外的html代码，其代码都来自于模板文件)，仅仅需要定义 block块内的信息即可，当然也可以不定义，或者在block块中调用父模板的内容。

```html
// test.html

{% extends 'index.html' %}   // 继承模板文件
 
{% block  content %}     // 针对模板中content的代码块进行定义

    <h1>hello world</h1>
    <h1>hello world</h1>
    <h1>hello world</h1>
    <h1>hello world</h1>
    <h1>hello world</h1>
    <h1>hello world</h1>
 
{% endblock %}
```

使用注意：
- 如果在模板中使用 {% extends %} ，必须保证其为模板中的`第一个`模板标记(`html首部`)。 否则，模板继承将不起作用。
- 一般来说，基础模板中的 {% block %} 标签越多越好。 记住，子模板不必定义父模板中所有的代码块，因此你可以用合理的缺省值对一些代码块进行填充，然后只对子模板所需的代码块进行（重）定义。 俗话说，钩子越多越好。
- 如果发觉自己在多个模板之间拷贝代码，你应该考虑将该代码段放置到父模板的某个 {% block %} 中。如果你需要访问父模板中的块的内容，使用 `{{ block.super }}` 这个标签吧，这一个魔法变量将会表现出父模板中的内容。 如果只想在上级代码块基础上添加内容，而不是全部重载，该变量就显得非常有用了。
- 不允许在同一个模板中定义多个同名的 {% block %} .存在这样的限制是因为block 标签的工作方式是双向的。也就是说，block 标签不仅挖了一个要填的坑，也定义了在父模板中这个坑所填充的内容。如果模板中出现了两个相同名称的 {% block %} 标签，父模板将无从得知要使用哪个块的内容。
- `一个html文件只能引入一个模版`

## 1.5 include引入
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在很多网站中，基本上的都会有一个开头和一个结尾，在每一个网页中都会显示。相对于这种的来说，在Django中，最好的方法就是使用include的标签，在每一个模板中都加入这个开头和结尾的标签。  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;简单来说，就是那些多个页面都需要的公共标签放在一个统一的html文件，供其他html文件引入。
```jinja
{% include 'tpl.html' %}  : 表示引入tpl.html文件。
```
看下面的例子：
```python
# -------------------- tpl.html --------------------
<div>
    <h1>hello world</h1>
</div>
 
 
 
 
# -------------------- xxx.html --------------------
 
...
 
    <div>
        {% include 'tpl.html' %}   
        {% include 'tpl.html' %}  // tpl.html的全部内容会在这里填充
        ...
    </div>   
 
...
```
在一个页面中可以通过include引入不同的公共组件。