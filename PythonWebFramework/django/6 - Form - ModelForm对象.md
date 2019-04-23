<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 Form表单验证](#1-form表单验证)
    - [1.1 创建一个Form表单对象](#11-创建一个form表单对象)
    - [1.2 利用Form表单对象进行验证](#12-利用form表单对象进行验证)
    - [1.3 利用Form对象手动生成前端标签](#13-利用form对象手动生成前端标签)
    - [1.4 利用Form对象自动生成前端标签](#14-利用form对象自动生成前端标签)
    - [1.5 Form字段的widget属性](#15-form字段的widget属性)
    - [1.6 Form字段其他的属性](#16-form字段其他的属性)
    - [1.7 特别补充-ChoiceField](#17-特别补充-choicefield)
    - [1.8 自定义验证规则](#18-自定义验证规则)
        - [1.8.1 通过对象来自定义验证规则](#181-通过对象来自定义验证规则)
        - [1.8.2 通过函数来自定义验证规则](#182-通过函数来自定义验证规则)
        - [1.8.3 通过在当前类的中定义 clean_字段名 的方法 来实现正则验证](#183-通过在当前类的中定义-clean_字段名-的方法-来实现正则验证)
        - [例子：注册的时候，两次密码一致性的验证](#例子注册的时候两次密码一致性的验证)
        - [1.8.4 强制触发异常](#184-强制触发异常)
- [2 ModelForm表单验证](#2-modelform表单验证)
    - [2.1 基本使用](#21-基本使用)
    - [2.2 自定制字段名](#22-自定制字段名)
    - [2.3 展示指定的列](#23-展示指定的列)
    - [2.4 初始化数据](#24-初始化数据)
    - [2.5 更新数据](#25-更新数据)
    - [2.6 其他ModelForm配置项](#26-其他modelform配置项)

<!-- /TOC -->
# 1 Form表单验证
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;什么是form？django 中的  form 组件 专门用来做对用户通过form表单形式提交的数据的格式进行验证的,还可以提供诸如form表单生成等牛逼的功能。  

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用方式为：首先我们定义一个Form模版（可以理解为是匹配数据的模版），其中对字段进行了规范。接下来请求发过来后(request.POST)，我们把request.POST的数据交给form模版，进行验证，form模版验证完成后会产出三个信息。

1. 是否验证成功
2. 所有的正确信息
3. 所有的错误信息

## 1.1 创建一个Form表单对象
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用From模版和model一样，是一个类，使用时需要引入django的form组件，然后继承Form类来获得一些功能。(可以定义在models.py中)

```python
# 基本使用
from django.forms import Form
from django.forms import fields
 
class FM(Form):
    username = fields.CharField()    # 创建一个验证对象，用于验证用户提交的数据中的name属性为username的数据
    password = fields.CharField()
    email = forms.EmailField()
```
注意： 这里定义的Form字段名称 必须和 前端form表单中的 input 框的 name 属性的值一致，才能进行验证。

## 1.2 利用Form表单对象进行验证
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;当用户使用POST方法提交数据的时候，应该首先交给Form模版进行验证，这时Form模版会返回三个对象：
1. 验证的结果：formobj.as_valid()
2. 验证成功的数据：formobj.cleaned_data
3. 验证失败的数据：formobj.errors

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;使用form对象，那么需要实例化，在实例化时，传入需要验证的数据，这里传入request.POST

```python
def post(self,request):
　　obj = FM(data=request.POST)    # 创建一个FM与用户提交数据的一个对象
　　print(obj.is_valid())    　　　 # 对这个对象进行验证，匹配成功会返回True，否则会返回False
　　print(obj.cleaned_data)   　　  # 存放匹配成功的数据
　　print(obj.errors.as_json())    # 存放错误的数据(默认情况下会输出一个字符串，包含了HTML标签，我们还可以使用Form提供的as_json方法对输出的字符串进行格式化显示)
```

参数有：
- data参数：表示要验证的数据
- Initial参数(可选)：表示的是初始化数据(数据类型为字典嵌套元组)。

利用initial初始化Form数据
```python
class Login(View):

    def get(self,request):
        if request.GET.get('uid',None):    # 判断用户是否提交数据
            userid = request.GET.get('uid')   # 获取用户请求的数据
            userinfo = models.UserInfo.objects.filter(id=userid).first()   # 在数据库中找到对应的数据
            obj = FM(initial={'username':userinfo.username,'password':userinfo.password,'email':userinfo.email})  # 对Form表单进行初始化赋值
        else:   
            obj = FM()     # 否则不进行初始化
        return render(request,'login.html',{'data':obj})
```
可以得到的结果为：
```python
formobj.as_valid()的返回结果为：  
# True/False

formobj.cleaned_data的返回结果为：
# 验证成功会存放通过验证的数据
{'password': 'asasas', 'username': 'asas', 'email': 'asas@abc.com'}

formobj.errors的结果为：
# 默认的输出格式<br><ul class="errorlist"><li>pwd<ul class="errorlist"><li>This field is required.</li></ul></li><li>email<ul class="errorlist"><li>This field is required.</li></ul></li><li>user<ul class="errorlist"><li>This field is required.</li></ul></li></ul>
 
# 可以使用 as_json()格式化为字典
{"email": [{"message": "This field is required.", "code": "required"}],
  "pwd": [{"message": "This field is required.", "code": "required"}],
  "user": [{"message": "This field is required.", "code": "required"}]}
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于前端提交了空的数据，所以这里message表示错误提示信息， This field is required.(这个字段不能为空)，code 表示的错误代码，required 表示必须的(必填项)。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;有些同学会说，django返回的是英文的，我看不懂，那么其实我们可以自己进行定义，即针对code字段的类型进行自定义。（在error_message字典中，根据code的类型，定义不同的提示信息即可）
```python
class FM(Form):
    username = fields.CharField(error_messages={'required':'用户名不能为空'})  
    password = fields.CharField(max_length=12,min_length=6,error_messages={'required':'密码不能为空','min_length':'密码长度不能小于6位','max_length':'密码的长度不能超过12位'})
    email = fields.EmailField(error_messages={'required':'邮箱不能为空','invalid':'邮箱格式错误'})
```
这时，如果错误代码等于我们定义的，那么就会输出对应的错误信息。

扩展：由于formobj.erros中包含了所有的错误信息，那么我们可以把它传递到前端，然后再进行取值
```python
def post(self,request):
        print(request.POST)
        obj = FM(data=request.POST)   # 创建一个FM
        return render(request,'login.html',{'data':obj.errors})

# 前端页面
<form id = 'form' method="post">
    {% csrf_token %}
    <h1>Hello 用户登陆</h1>
    <p>用户名: <input type="text" name="username"> {{ data.username.0 }}</p>
    <p>
        密码:<input type="text" name="password">
        {{ data.password.0 }}
    </p>
    <p>
        邮箱:<input type="text" name="email">
        {{ data.email.0 }}
    </p>
    <input type="submit" value="提交">
</form>
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;注意：这里用.0 取数据是因为 错误信息 message，其实是个列表，会存放所有的错误信息，对于空的请求，只会返回 非空的 错误信息，但是数据类型也是列表，所以这里取第一个。 

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;点击提交后，用户输入正确的数据，会消失，这不是我们想要的，并且form标签看起来也很乱，有没有一种方式直接来生成呢，各位看官，请继续往下看。

## 1.3 利用Form对象手动生成前端标签
　　前面可以看到，我们创建FM类的时候，指定了三个Form字段，那么我们可以利用这三个Form字段来生成对应的input输入框。(Form字段其实本身不具有生成HTML标签的功能，它只有验证的功能，真正生成HTML标签的操作，是由Form字段类型(CharField)中的widgets属性定义了textinput插件完成的，详细信息，可以查看widgets的原码，继续了解)
```python
{{ Form对象.字段名 }}    # 前端渲染时会渲染出对应的HTML标签
```
对例子进行改写：
```python
# get方式要把FM对象传递给前端，否则无法生成HTML
def get(self,request):
    obj = FM()
    return render(request,'login.html',{'data':obj})
 
 
# 前端调用
<form method="post" novalidate>
    {% csrf_token %}
    <p>{{ data.username }} {{ data.errors.username.0 }}</p>
    <p>{{ data.password }} {{ data.errors.password.0 }}</p>
    <p>{{ data.email }} {{ data.errors.email.0 }}</p>
    <input type="submit" value="提交">
</form>
# 默认情况下高版本浏览器会对错误提示进行小窗提示，这里的为了避免浏览器这个行为，所以给form表单添加 novalidate 属性。
```

## 1.4 利用Form对象自动生成前端标签
Form提供了三种自动生成前端标签的方式：as_p,as_ul,as_table，由于是自动生成的，所以灵活性对比手动，要稍差点。
- as_p：生成的标签为p类型
- as_ul：生成的标签格ul类型
- as_table：生成的标签为tbody类型(注意，需要手动添加外层的<table></table>标签)

```html
<!-- 1 as_table  -->
<form method="post" novalidate>
    {% csrf_token %}
    <table>
        {{ data.as_table }}
    </table>
    <input type="submit" value="提交">
</form>
# 注意 as_table 的方式需要我们手动的添加table标签
 
<!-- 2 as_p  -->
<form method="post" novalidate>
    {% csrf_token %}
    {{ data.as_p }}
    <input type="submit" value="提交">
</form>
 
<!-- 2 as_ul  -->
<form method="post" novalidate>
    {% csrf_token %}
    {{ data.as_ul }}
    <input type="submit" value="提交">
</form>
```
注意：as_p,as_ul,as_table这种自动生成标签的方式来自于我们创建的FM对象，所以如果要让Form自动创建，那么需要在get请求模式下把Form对象，传递给Templates，才能引用
```python
def get(self,request):
    obj = FM()
    return render(request,'login.html',{'data':obj})
```
使用以上两种方式：用户的输入的正确数据都会被保留。是不是很屌。不屌，因为只能输出input标签，还不能定制HTML标签的属性，真是太low了。No、No、No 你想到的，开发人员早就已经想到了

## 1.5 Form字段的widget属性
由于前面已经说过，生成的标签类型是由widgets来控制的，那么我们就可以对其进行定制，生成我们想要的标签类型，并给标签添加属性信息。
```python
from django.forms import widgets    #导入插件库
 
class FM(Form):
    username = fields.CharField(
        error_messages={'required':'用户名不能为空'},
        widget=widgets.Textarea(attrs={'class':'c1'})     # 设置字段的类型为Textarea，并且添加class属性的值为cl
    )
    password = fields.CharField(
        max_length=12,
        min_length=6,
        error_messages={'required':'密码不能为空','min_length':'密码长度不能小于6位','max_length':'密码的长度不能超过12位'},
        widget=widgets.PasswordInput(attrs={'class':'c1'})  # 设置字段类型为Password类型，添加class属性为c1
    )
```
这里我们修改了username字段的 标签格式，输出的就是一个textarea标签，并且利用了attr参数给标签添加了 class 属性（多个属性可以在attr中继续添加），这个插件里面几乎包含了所有的输入框比如input、radio、select、等等。

## 1.6 Form字段其他的属性
一大波属性来袭：
```python
Field
    required=True,               是否允许为空
    widget=None,                 HTML插件
    label=None,                  用于生成Label标签或显示内容
    initial=None,                初始值
    help_text='',                帮助信息(在标签旁边显示)
    error_messages=None,         错误信息 {'required': '不能为空', 'invalid': '格式错误'}
    show_hidden_initial=False,   是否在当前插件后面再加一个隐藏的且具有默认值的插件（可用于检验两次输入是否一直）
    validators=[],               自定义验证规则
    localize=False,              是否支持本地化（比如时区，在存储时区数据时，会匹配本地时区，不常用）
    disabled=False,              是否可以编辑
    label_suffix=None            Label内容后缀(比如在前端使用formobj.as_table生成表单时，label和input中间的分隔符，默认是冒号，不常用)
  
  
CharField(Field)
    max_length=None,             最大长度
    min_length=None,             最小长度
    strip=True                   是否移除用户输入空白
  
IntegerField(Field)
    max_value=None,              最大值
    min_value=None,              最小值
  
FloatField(IntegerField)
    ...
  
DecimalField(IntegerField)
    max_value=None,              最大值
    min_value=None,              最小值
    max_digits=None,             总长度
    decimal_places=None,         小数位长度
  
BaseTemporalField(Field)
    input_formats=None          时间格式化  
  
DateField(BaseTemporalField)    格式：2015-09-01
TimeField(BaseTemporalField)    格式：11:12
DateTimeField(BaseTemporalField)格式：2015-09-01 11:12
  
DurationField(Field)            时间间隔：%d %H:%M:%S.%f
    ...
  
RegexField(CharField)
    regex,                      自定制正则表达式
    max_length=None,            最大长度
    min_length=None,            最小长度
    error_message=None,         忽略，错误信息使用 error_messages={'invalid': '...'}
  
EmailField(CharField)     
    ...
  
FileField(Field)
    allow_empty_file=False     是否允许空文件
  
ImageField(FileField)     
    ...
    注：需要PIL模块，pip3 install Pillow
    以上两个字典使用时，需要注意两点：
        - form表单中 enctype="multipart/form-data"
        - view函数中 obj = MyForm(request.POST, request.FILES)
  
URLField(Field)
    ...
  
  
BooleanField(Field) 
    ...
  
NullBooleanField(BooleanField)
    ...
  
ChoiceField(Field)
    ...
    choices=(),                选项，如：choices = ((0,'上海'),(1,'北京'),)
    required=True,             是否必填
    widget=None,               插件，默认select插件
    label=None,                Label内容
    initial=None,              初始值
    help_text='',              帮助提示
  
  
ModelChoiceField(ChoiceField)
    ...                        django.forms.models.ModelChoiceField
    queryset,                  # 查询数据库中的数据
    empty_label="---------",   # 默认空显示内容
    to_field_name=None,        # HTML中value的值对应的字段
    limit_choices_to=None      # ModelForm中对queryset二次筛选
      
ModelMultipleChoiceField(ModelChoiceField)
    ...                        django.forms.models.ModelMultipleChoiceField
  
  
      
TypedChoiceField(ChoiceField)
    coerce = lambda val: val   对选中的值进行一次转换
    empty_value= ''            空值的默认值
  
MultipleChoiceField(ChoiceField)
    ...
  
TypedMultipleChoiceField(MultipleChoiceField)
    coerce = lambda val: val   对选中的每一个值进行一次转换
    empty_value= ''            空值的默认值
  
ComboField(Field)
    fields=()                  使用多个验证，如下：即验证最大长度20，又验证邮箱格式
                               fields.ComboField(fields=[fields.CharField(max_length=20), fields.EmailField(),])
  
MultiValueField(Field)
    PS: 抽象类，子类中可以实现聚合多个字典去匹配一个值，要配合MultiWidget使用
  
SplitDateTimeField(MultiValueField)
    input_date_formats=None,   格式列表：['%Y--%m--%d', '%m%d/%Y', '%m/%d/%y']
    input_time_formats=None    格式列表：['%H:%M:%S', '%H:%M:%S.%f', '%H:%M']
  
FilePathField(ChoiceField)     文件选项，目录下文件显示在页面中(下拉框显示path指定目录下的文件)，提交时 提交的是 文件的路径
    path,                      文件夹路径
    match=None,                正则匹配
    recursive=False,           递归下面的文件夹
    allow_files=True,          允许文件
    allow_folders=False,       允许文件夹
    required=True,
    widget=None,
    label=None,
    initial=None,
    help_text=''
  
GenericIPAddressField
    protocol='both',           both,ipv4,ipv6支持的IP格式
    unpack_ipv4=False          解析ipv4地址，如果是::ffff:192.0.2.1时候，可解析为192.0.2.1， PS：protocol必须为both才能启用
  
SlugField(CharField)           数字，字母，下划线，减号（连字符）
    ...
  
UUIDField(CharField)           uuid类型
    ...
```
Django内置其他插件
```python
TextInput(Input)
NumberInput(TextInput)
EmailInput(TextInput)
URLInput(TextInput)
PasswordInput(TextInput)
HiddenInput(TextInput)
Textarea(Widget)
DateInput(DateTimeBaseInput)
DateTimeInput(DateTimeBaseInput)
TimeInput(DateTimeBaseInput)
CheckboxInput
Select
NullBooleanSelect
SelectMultiple
RadioSelect
CheckboxSelectMultiple
FileInput
ClearableFileInput
MultipleHiddenInput
SplitDateTimeWidget
SplitHiddenDateTimeWidget
SelectDateWidget
```

label 属性的用法举例
```python
<form action="/test/" method="post" novalidate>
    {% csrf_token %}
    <p>{{ obj.username.label }} {{ obj.username }} {{ obj.username.errors.0 }}</p>
    <p>{{ obj.password.label }} {{ obj.password }} {{ obj.password.errors.0 }}</p>
    <input type="submit" value="提交">
</form>
```

## 1.7 特别补充-ChoiceField
ChoiceField表示下拉选择类型，针对不同的标签，不同的效果有多种搭配，需要注意的是，当输入框为非ChoiceField类型时，可以使用widget来添加choices选项。如果是ChoiceField类型，那么就可以直接添加choices属性。
```python
# 单radio，值为字符串
user = fields.CharField(
 initial=2,　　# 初始数据数量为2
 widget=widgets.RadioSelect(choices=((1,'上海'),(2,'北京'),))
)
 
# 单radio，值为字符串
user = fields.ChoiceField(
 choices=((1, '上海'), (2, '北京'),),     # 定义要展示的数据信息，1表示ID
 initial=2,
 widget=widgets.RadioSelect
)
 
# 单select，值为字符串
user = fields.CharField(
 initial=2,
 widget=widgets.Select(choices=((1,'上海'),(2,'北京'),))
)
 
# 单select，值为字符串
user = fields.ChoiceField(
 choices=((1, '上海'), (2, '北京'),),
 initial=2,
 widget=widgets.Select
)
 
# 多选select，值为列表
user = fields.MultipleChoiceField(
 choices=((1,'上海'),(2,'北京'),),
 initial=[1,],
 widget=widgets.SelectMultiple
)
 
 
# 单checkbox
user = fields.CharField(
 widget=widgets.CheckboxInput()
)
 
 
# 多选checkbox,值为列表
user = fields.MultipleChoiceField(
 initial=[2, ],
 choices=((1, '上海'), (2, '北京'),),
 widget=widgets.CheckboxSelectMultiple
)
```
ChoiceField类型的字段表示的是下拉单选框，它的choice选项表示要提供给用户选择的选项。
```python
# choice 字段格式
city = fields.ChoiceField(
    choices=[(0,'上海'),(1,'北京')]
)
```
> 可以看到choices接收的数据类型为列表嵌套元组，是不是看起来很熟悉，没错，使用value_list获取到的数据，就是这个格式。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个时候需要来一波需求了，当choices的数据来源于数据库的时候，我新加了一个城市，ChoiceField 并不！会！动！态！的！重！新！读！取！，为什么呢？

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;这个和类的实例化有关，严格来说的话这些字段都属于类的属性，类属性不会在实例化的时候进行重新读取，那么想要类属性在实例化的时候重新读取该咋办，是的， 没错，定义在__init__ 函数中(每次实例化都会执行)。

```python
class UserForm(Form):
    username = fields.CharField(
        required=True,
        error_messages={'required':'用户名不能为空','invaild':'用户名格式不正确'}
    )
    password = fields.CharField(
        required=True,
        error_messages={'required':'密码不能为空','invalid':'密码格式不正确'}
    )
 
 
    ut_id = fields.ChoiceField(choices=[]) # 下面每次初始化都会执行，所以这里可以写空，避免二次查库
    # choices=[(1,'上海'),(2,'北京'),(3,'河南')] 这里的choices接受这种格式的数据，和values_list的结果很像
 
 
    def __init__(self,*args,**kwargs):
        super(UserForm, self).__init__(*args,**kwargs)
 
        self.fields['ut_id'].choices =models.UserType.objects.values_list('id','title')  # 由于每次实例化的时候都会执行__init__方法，就可以动态的实时更新了。
```
> 针对下拉列表这种可能会变得数据，让其每次实例化的时候都动态的生成，否则会造成数据不能实时显示的问题

## 1.8 自定义验证规则
我们知道Form表单对象内置了许多字段类型，根据这些字段类型我们可以知道，其内部是包含了正则表达式的。其中源码如下：
```python
# Email字段正则表达式
class EmailValidator(object):
    message = _('Enter a valid email address.')
    code = 'invalid'
    user_regex = _lazy_re_compile(
        r"(^[-!#$%&'*+/=?^_`{}|~0-9A-Z]+(\.[-!#$%&'*+/=?^_`{}|~0-9A-Z]+)*\Z"  # dot-atom
        r'|^"([\001-\010\013\014\016-\037!#-\[\]-\177]|\\[\001-\011\013\014\016-\177])*"\Z)',  # quoted-string
        re.IGNORECASE)
    domain_regex = _lazy_re_compile(
        # max length for domain name labels is 63 characters per RFC 1034
        r'((?:[A-Z0-9](?:[A-Z0-9-]{0,61}[A-Z0-9])?\.)+)(?:[A-Z0-9-]{2,63}(?<!-))\Z',
        re.IGNORECASE)
    literal_regex = _lazy_re_compile(
        # literal form, ipv4 or ipv6 address (SMTP 4.1.3)
        r'\[([A-f0-9:\.]+)\]\Z',
        re.IGNORECASE)
    domain_whitelist = ['localhost']
```
那么现在有一个需求，我要验证手机号，可是django Form没有内置这个正则表达式，我们该怎么办？ 能否自定义？答案是可以的。主要有以下三种方式：
1. 通过validators属性来定义(类型为列表)：主要分为下面两种方式
    - 对象
    - 函数
2. 通过在当前类的中定义 clean_字段名 的方法 来实现正则验证

### 1.8.1 通过对象来自定义验证规则
语法格式：
```python
validators=[RegexValidator(r'正则表达式','错误提示')]
```
匹配手机号的简单实例：
```python
from django.forms import fields
from django.forms import Form
from django.core.validators import RegexValidator     # 注意需要导入RegexValidator对象
 
 
class UserInfo(Form):
 
    username = fields.EmailField()
    phone = fields.CharField(
        validators=[RegexValidator(r'^[0-9]+$', '请输入数字'), RegexValidator(r'^159[0-9]+$', '数字必须以159开头')],
    )
```

注意：
1. 多个条件并存的时候，顺序为从左到右
2. 由于指定传递正则表达式对象，所以可定制性不高
3. 多用于简单的场合下

### 1.8.2 通过函数来自定义验证规则
语法格式：
```python
from django.core.exceptions import ValidationError  # 需要导入ValidationError
  
def 函数名(value):
  
    '''匹配规则若干'''
    如果不匹配:
        raise ValiadationError('error message')
  
validators=[函数名,]
  
# 在字段的validators参数中表示正则匹配使用函数
# 在函数中定义正则表达式进行返回
# 匹配成功可以不进行返回
# 匹配失败需要抛出ValidationError异常　
```
为什么要使用ValidationError来抛出异常呢？这里来自于源码，有人要问，为啥老看源码，小白看不懂啊，我想所，人家django都写好了，你不去学习，还好意思问。神经病啊你
```python
# RegexValidator内部信息
def __call__(self, value):
        """
        Validate that the input contains a match for the regular expression
        if inverse_match is False, otherwise raise ValidationError.
        """
        if not (self.inverse_match is not bool(self.regex.search(
                force_text(value)))):
            raise ValidationError(self.message, code=self.code)

# 可以看到 RegexValidator 正则表达式对象 匹配不成功的时候会使用raise 主动的抛出ValidationError 异常，并且ValidationError对象接受两个信息，但是code已经有默认值了，所以我们这里只返回错误message即可
```
匹配手机号示例：
```python
import re
from django.forms import fields
from django.forms import Form
from django.core.exceptions import ValidationError
 
def my_re(value):
 
    mobile_re = re.compile(r'^(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$')
 
    if not mobile_re.match(value):
 
        raise ValidationError('手机号码格式错误')
 
 
 
 
class User(Form):
 
    username = fields.CharField()
    phone = fields.CharField(
        validators=[my_re,]       # 调用我们定义的my_re函数进行验证
    )
```

### 1.8.3 通过在当前类的中定义 clean_字段名 的方法 来实现正则验证
为什么要定义 clean_字段名 来进行正则验证呢？这里通过form.is_valid() 入口查看form字段的验证规则。
```python
# is_valid
    def is_valid(self):
        """
        Returns True if the form has no errors. Otherwise, False. If errors are
        being ignored, returns False.
        """
        return self.is_bound and not self.errors
# 最后调用了 self.errors
 
 
# 查看self.errors函数
    def errors(self):
        "Returns an ErrorDict for the data provided for the form"
        if self._errors is None:
            self.full_clean()
        return self._errors
# 这里又返回了 full_clean()
 
 
# 查看full_clean()函数
    def full_clean(self):
        """
        Cleans all of self.data and populates self._errors and
        self.cleaned_data.
        """
        self._errors = ErrorDict()
        if not self.is_bound:  # Stop further processing.
            return
        self.cleaned_data = {}
        # If the form is permitted to be empty, and none of the form data has
        # changed from the initial data, short circuit any validation.
        if self.empty_permitted and not self.has_changed():
            return
 
        self._clean_fields()    # 这里执行了 clean_fields() 函数，只要定义了clean_fieldsname 就会自动被执行
        self._clean_form()
        self._post_clean()
```
所以这里方法名称有特殊要求，必须为：clean_当前字段,自动会去执行这个方法，定义字段的时候不用显示的调用这个方法。
```python
from django import forms
from django.forms import fields
from django.forms import widgets
from django.core.exceptions import ValidationError
from django.core.validators import RegexValidator
 
def clean_phone(self):
    # 去取用户提交的值：可能是错误是的，也可能是正确的
    value = self.cleaned_data['phone']
    mobile_re = re.compile(r'^(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$')
    if not mobile_re.match(value):
        raise ValidationError('手机号码格式错误')
    if models.UserInfo.objects.filter(phone=value).count():
        raise ValidationError('手机号码已存在')   # 可以利用数据库对数据唯一性进行验证
    else:
        return value   # 这里必须要返回正确的值，因为返回值会被重新赋值给cleaned_data中的Phone字段的值
     
    # 先通过form基础验证，才会继续匹配。 比如字段是必填的，如果没有填写，则不会执行clean_phone函数
```
**切记：
1. 不要瞎取值，因为不是按照顺序取值的，所以在你取得时候，可能还没有赋值。
2. 对象.__dict__：通过这种方法去取得值，因为每次顺序都是不一样的。所以可能你取的值，还没有被赋值。

小结：
1. 对象，只能添加正则表达式
2. 函数，不常用
3. 当前类方法中自定义，可以添加对数据库的操作，注意必须要有返回值。
4. 对象方法和类方法中定义，是可以同时存在的，先执行对象中定义的，然后再执行类方法中自定义的。

### 例子：注册的时候，两次密码一致性的验证
由于不能再一个验证规则里面取其他字段的值，所以前面的方法并不可行，这里django提供了其他的form表单的整体验证（细心的人会看到刚刚的full_clean后有一个clean_form函数）

定义clean函数，来对form进行整体验证(注意这里的数据，表示前面验证都有通过)
```python
from django.core.exceptions import ValidationError
 
def clean(self):
 
    pwd = self.cleaned_data['pwd']
    pwd_confirm = self.cleaned_data['pwd_confirm']
    if pwd == pwd_confirm:
        return self.cleaned_data   # 必须有返回值，事关再次取cleaned_data是否有值
    else:
        self.add_error('pwd',ValidationError('密码输入不一致'))   # 根据源码中的操作，把错误信息添加到 self.add_errors
        self.add_error('pwd_confirm',ValidationError('密码输入不一致'))  # 添加错误信息必须是ValidationError对象
        return self.cleaned_data
 
    # 如果有人添加错误，那么就不能通过
```
扩展：
1. 由于在full_clean()函数中会执行钩子函数(clean_fields，clean_form，_post_clean)，所以钩子函数存在，就执行，不存在就跳过。【clean_form，_post_clean功能类似，这里不在赘述】
2. 当对象和类方法 同时存在时，先匹配对象，然后再匹配类方法。　

### 1.8.4 强制触发异常
在某些场景下比如用户登录，如果用户名或者密码错误，我们需要手动触发一个异常，并提示：‘用户名或者密码错误’。这时可以利用form.add_error() 来进行触发。
```python
from django.core.exceptions import ValidationError
 
form.add_error('password',ValidationError('用户名或者密码错误'))
 
 
# 这样在前端的password字段后面会主动触发异常
```
当然，这里用了ValidationError对象，是因为django提供的 form.is_valid中最后也是用过ValidationError完成添加的，这里其实也可以直接写错误信息，但是推荐和源码保持一致。

# 2 ModelForm表单验证
ModelForm顾名思义就是model和form的结合体，所以它包含了form的功能，比如：
- 验证
- 表单生成

同时它还包含了model的功能，比如
- 数据库操作(比如数据的修改，保存等)

## 2.1 基本使用
使用起来比forms简单很多，不需要再编写字段，只需要传入要关联的model类即可。
```python
from django import forms
from rbac import model

class UserInfoModelForm(forms.ModelForm):
    class Meta:
        model = models.Userinfo  # 与Userinfo建立关系
        fields = "__all__"  # 要显示的Userinfo字段(__all__表示所有)
```
前端渲染方法和forms相同，这里不再赘述

## 2.2 自定制字段名
默认渲染出来的字段名称是model类中对应的字段的名称(英文),如果要显示中文名称，可以通过两种方式：
1. 在model类中添加verbose_name来定义显示的中文名称(djang admin也是读取这个名字的)
2. 在ModelForm类中定义显示的名称

ModelForm中：
```python
class UserInfoModelForm(forms.ModelForm):
    class Meta:
        model = models.Userinfo  # 与Userinfo建立关系
        fields = "__all__"  # 要显示的Userinfo字段(__all__表示所有)
        labels = {
            'username':'你好',
            'password':'世界',
            'roles':'谢谢'
        }
```
优先级高于字段的verbose_name，如果同时设置，会labels会覆盖verbose_name

## 2.3 展示指定的列
fields用于指定要在前端渲染的字段，主要有两类值
- \_\_all__: 显示所有字段
- \['x','xx']:显示指定字段 

```python
# 显示指定列
fields = ['username','email']  

# 排除指定列
exclude = ['role']      
```

## 2.4 初始化数据
ModelForm对象在实例化的同时，还可以指定两个参数，用于初始化form表单（比如在修改页面，提前把要修改的数据灌入form表单中）
- initial: 类型为字典
- instance: 类型为对象

```python
obj = models.Userinfo.objects.filter(pk=2).first()  
uf = UserInfoModelForm(instance=obj)

obj = models.Userinfo.objects.filter(pk=2).values('username','password').first()
uf = UserInfoModelForm(initial=obj)
```

## 2.5 更新数据
需要特别注意更新数据时的操作，如果不传入要更新的实例，就会变成新增数据
```python
if request.method == 'POST':
    obj = models.Userinfo.objects.filter(pk=2).first()
    # uf = UserInfoModelForm(request.POST)  # 直接调用是新增
    uf = UserInfoModelForm(data=request.POST,instance=obj)  # 这样才是更新数据
    if uf.is_valid():
        uf.save()
        return redirect('/arya/rbac/userinfo/')
```

## 2.6 其他ModelForm配置项
下面是ModelForm组件的相关配置项目：
```python
a.  class Meta:
        model,                           # 对应Model的
        fields=None,                     # 字段
        exclude=None,                    # 排除字段
        labels=None,                     # 提示信息
        help_texts=None,                 # 帮助提示信息
        widgets=None,                    # 自定义插件
        error_messages=None,             # 自定义错误信息（整体错误信息from django.core.exceptions import NON_FIELD_ERRORS）
        field_classes=None               # 自定义字段类 （也可以自定义字段）
        localized_fields=('birth_date',) # 本地化，如：根据不同时区显示数据
        如：
            数据库中
                2016-12-27 04:10:57
            setting中的配置
                TIME_ZONE = 'Asia/Shanghai'
                USE_TZ = True
            则显示：
                2016-12-27 12:10:57
b. 验证执行过程
    is_valid -> full_clean -> 钩子 -> 整体错误

c. 字典字段验证
    def clean_字段名(self):
        # 可以抛出异常
        # from django.core.exceptions import ValidationError
        return "新值"
d. 用于验证
    model_form_obj = ModelForm()
    model_form_obj.is_valid()
    model_form_obj.errors.as_json()
    model_form_obj.clean()
    model_form_obj.cleaned_data
e. 用于创建
    model_form_obj = ModelForm(data=request.POST)
    #### 页面显示，并提交 #####
    # 默认保存多对多
        obj = form.save(commit=True)
    # 不做任何操作，内部定义 save_m2m（用于保存多对多）
        obj = form.save(commit=False)
        obj.save()      # 保存单表信息
        obj.save_m2m()  # 保存关联多对多信息
```
下面是一个小例子：
```python
from django import forms
from django.forms import widgets as froms_widgets

class UserInfoModelForm(forms.ModelForm):

    class Meta:
        model = models.UserInfo
        fields = '__all__'
        # fields =  ['username','email']
        # exclude = ['username']
        labels = {
            'username': '用户名',
            'email': '邮箱',
        }
        help_texts = {
            'username': '...'
        }
        widgets = {
            'username': froms_widgets.Textarea(attrs={'class': 'c1'})
        }
        error_messages = {
            '__all__':{    # 整体错误信息

            },
            'email': {
                'required': '邮箱不能为空',
                'invalid': '邮箱格式错误..',
            }
        }
        field_classes = {  # 定义字段的类是什么
            # 'email': Ffields.URLField  # 这里只能填类，加上括号就是对象了。
        }

        # localized_fields=('ctime',)  # 哪些字段做本地化
```
