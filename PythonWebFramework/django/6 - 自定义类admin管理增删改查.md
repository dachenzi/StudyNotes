<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 django admin分析](#1-django-admin分析)
- [2 django admin源码分析](#2-django-admin源码分析)
    - [2.1 注册部分](#21-注册部分)
    - [2.2 路由分发部分](#22-路由分发部分)
    - [2.3 流程总结](#23-流程总结)
    - [2.4 定制admin](#24-定制admin)
- [3 django admin的接口](#3-django-admin的接口)
- [4 自定义ayra进行CRUD](#4-自定义ayra进行crud)
    - [4.1 准备工作](#41-准备工作)
        - [4.1.1 文件加载顺序](#411-文件加载顺序)
        - [4.1.2 让我们的程序被优先加载](#412-让我们的程序被优先加载)
        - [4.1.3 编写调用文件](#413-编写调用文件)
        - [4.1.4 更换装载应用的方式](#414-更换装载应用的方式)
        - [4.1.5 include本质](#415-include本质)
    - [4.2 为每个model对象生成对应的url](#42-为每个model对象生成对应的url)
        - [4.2.1 注册model类](#421-注册model类)
        - [4.2.2 service/arya基本实现](#422-servicearya基本实现)
        - [4.2.3 添加urls及views函数](#423-添加urls及views函数)

<!-- /TOC -->

# 1 django admin分析
在每个app下都会存在一个admin.py文件用于注册当前app下的models文件，给django admin来管理，仔细观察Django admin针对不同models的管理页面、URL等信息，我们发现
1. 不同的类对象生成的url是类似的。
```python
# UserInfo
/admin/app01/userindo/
/admin/app01/userinfo/add
/admin/app01/userinfo/1/change
/admin/app01/userinfo/2/delete
# Role
/admin/role/
/admin/role/add
/admin/role/1/change
/admin/role/2/delete
```
2. 不同的类，页面的布局模版也是类似的。
所以，我们分析，admin内部应该是为我们的注册的类，生成了对应的url，以及视图函数，并且通过相同的模版渲染来返回页面的。下面来跟一下源码文件
# 2 django admin源码分析
## 2.1 注册部分
首先从注册开始，看下我们的admin.py文件
```python
from django.contrib import admin
from .models import *
admin.site.register(Userinfo)
```
注册是通过site的register方法完成的，那么从这个入口开始看它内部是怎么完成的：
```python
site = AdminSite() 
```
查看他的register方法
```python
 def register(self, model_or_iterable, admin_class=None, **options):
    ... ... 
    self._registry[model] = admin_class(model, self)
```
内部把我们的model类使用字典存储起来(self._registry = {}),而admin_class默认情况下的值为ModelAdmin
## 2.2 路由分发部分
查看项目的urls.py文件
```python
from django.contrib import admin
urlpatterns = [
    url(r'^admin/', admin.site.urls),
]
```
这里的site和admin.py里的site是同一个对象,而在urls代码内部调用了get_urls方法
```python
 def get_urls(self):
    ... ...
        valid_app_labels = []
        for model, model_admin in self._registry.items():
            urlpatterns += [
                url(r'^%s/%s/' % (model._meta.app_label, model._meta.model_name), include(model_admin.urls)),
            ]
```
内部为每一个model类封装了一个url(inclue)路由：
- model._meta.app_label:就是app的名称
- model._meta.model_name：就是model的名称
最后调用的是model_admin的urls方法，其实也就是封装的ModelAdmin对象的urls方法
```python
    def get_urls(self):
        from django.conf.urls import url
        urlpatterns = [
            url(r'^$', wrap(self.changelist_view), name='%s_%s_changelist' % info),
            url(r'^add/$', wrap(self.add_view), name='%s_%s_add' % info),
            url(r'^(.+)/history/$', wrap(self.history_view), name='%s_%s_history' % info),
            url(r'^(.+)/delete/$', wrap(self.delete_view), name='%s_%s_delete' % info),
            url(r'^(.+)/change/$', wrap(self.change_view), name='%s_%s_change' % info),
            # For backwards compatibility (was the change url before 1.9)
            url(r'^(.+)/$', wrap(RedirectView.as_view(
                pattern_name='%s:%s_%s_change' % ((self.admin_site.name,) + info)
            ))),
        ]
        return urlpatterns
```
这里才是，为每个model类生成的url列表，对应的就是处理方法。
## 2.3 流程总结
整个流程如下：
1. site 是一个 AdminSite()对象
2. 查看AdminSite()对象的register方法，使用字典封装了我们的model类{model类,ModelAdmin(model类，site对象)}
3. 访问urls时，admin为每个model类创建一个以'/app名称/model类名'为前缀的url
4. 映射到包装好的ModelAdmin对象的get_url方法内
5. 在内部为CRUD操作，定义了url列表以及对应的views方法。
## 2.4 定制admin
根据分析，我们知道，真正完成数据库的增删改查的任务，其实是ModelAdmin对象，而在我们注册我们的model时，django提供了一个额外的参数admin_class，用于让我们定制自己的admin。
所以，我们可以通过继承ModelAdmin来定制，比如
```python
from django.contrib.admin.options import ModelAdmin
class MyModelAdmin(ModelAdmin):
    def changelist_view(self, request, extra_context=None):
        return HTTPResponse('hello')
admin.site.register(Permission, admin_class=MyModelAdmin)
```
这样，Permission表的，list页面就会被我们修改。
# 3 django admin的接口
上面的方法比较麻烦，django admin提供了不少接口和参数用于控制admin页面的展示
定制Admin
在admin.py中只需要讲Mode中的某个类注册，即可在Admin中实现增删改查的功能，如：
admin.site.register(models.UserInfo)
但是，这种方式比较简单，如果想要进行更多的定制操作，需要利用ModelAdmin进行操作，如：
```python
方式一：
    class UserAdmin(admin.ModelAdmin):
        list_display = ('user', 'pwd',)
 
    admin.site.register(models.UserInfo, UserAdmin) # 第一个参数可以是列表
     
 
方式二：
    @admin.register(models.UserInfo)                # 第一个参数可以是列表
    class UserAdmin(admin.ModelAdmin):
        list_display = ('user', 'pwd',)
```

ModelAdmin中提供了大量的可定制功能，如
1. list_display，列表时，定制显示的列。
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    list_display = ('user', 'pwd', 'xxxxx')
 
    def xxxxx(self, obj):
        return "xxxxx"
```
2. list_display_links，列表时，定制列可以点击跳转。
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    list_display = ('user', 'pwd', 'xxxxx')
    list_display_links = ('pwd',)
```
3. list_filter，列表时，定制右侧快速筛选。
```python
from django.utils.translation import ugettext_lazy as _
 
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    list_display = ('user', 'pwd')
 
    class Ugg(admin.SimpleListFilter):
        title = _('decade born')
        parameter_name = 'xxxxxx'
 
        def lookups(self, request, model_admin):
            """
            显示筛选选项
            :param request:
            :param model_admin:
            :return:
            """
            return models.UserGroup.objects.values_list('id', 'title')
 
        def queryset(self, request, queryset):
            """
            点击查询时，进行筛选
            :param request:
            :param queryset:
            :return:
            """
            v = self.value()
            return queryset.filter(ug=v)
 
    list_filter = ('user',Ugg,)
```
4. list_select_related，列表时，连表查询是否自动select_related
5. 分页相关
```python
# 分页，每页显示条数
    list_per_page = 100
 
# 分页，显示全部（真实数据<该值时，才会有显示全部）
    list_max_show_all = 200
 
# 分页插件
    paginator = Paginator
```
6. list_editable，列表时，可以编辑的列
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    list_display = ('user', 'pwd','ug',)
    list_editable = ('ug',)
```
7. search_fields，列表时，模糊搜索的功能
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
     
    search_fields = ('user', 'pwd')
```
8. date_hierarchy，列表时，对Date和DateTime类型进行搜索
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
 
    date_hierarchy = 'ctime'
```
9. preserve_filters，详细页面，删除、修改，更新后跳转回列表后，是否保留原搜索条件
10. save_as = False，详细页面，按钮为“Sava as new” 或 “Sava and add another”
11. save_as_continue = True，点击保存并继续编辑
```python
save_as_continue = True
 
# 如果 save_as=True，save_as_continue = True， 点击Sava as new 按钮后继续编辑。
# 如果 save_as=True，save_as_continue = False，点击Sava as new 按钮后返回列表。
 
New in Django 1.10.
```
12. save_on_top = False，详细页面，在页面上方是否也显示保存删除等按钮
13. inlines，详细页面，如果有其他表和当前表做FK，那么详细页面可以进行动态增加和删除
```python
class UserInfoInline(admin.StackedInline): # TabularInline
    extra = 0
    model = models.UserInfo
 
 
class GroupAdminMode(admin.ModelAdmin):
    list_display = ('id', 'title',)
    inlines = [UserInfoInline, ]
```
14. action，列表时，定制action中的操作
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
 
    # 定制Action行为具体方法
    def func(self, request, queryset):
        print(self, request, queryset)
        print(request.POST.getlist('_selected_action'))
 
    func.short_description = "中文显示自定义Actions"
    actions = [func, ]
 
    # Action选项都是在页面上方显示
    actions_on_top = True
    # Action选项都是在页面下方显示
    actions_on_bottom = False
 
    # 是否显示选择个数
    actions_selection_counter = True
```
15. 定制HTML模板
```python
add_form_template = None
change_form_template = None
change_list_template = None
delete_confirmation_template = None
delete_selected_confirmation_template = None
object_history_template = None
```
16. raw_id_fields，详细页面，针对FK和M2M字段变成以Input框形式
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
 
    raw_id_fields = ('FK字段', 'M2M字段',)
```
17. fields，详细页面时，显示字段的字段
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    fields = ('user',)
```
18. exclude，详细页面时，排除的字段
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    exclude = ('user',)
```
19. readonly_fields，详细页面时，只读字段
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    readonly_fields = ('user',)
```
20. fieldsets，详细页面时，使用fieldsets标签对数据进行分割显示
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    fieldsets = (
        ('基本数据', {
            'fields': ('user', 'pwd', 'ctime',)
        }),
        ('其他', {
            'classes': ('collapse', 'wide', 'extrapretty'),  # 'collapse','wide', 'extrapretty'
            'fields': ('user', 'pwd'),
        }),
    )
```
21. 详细页面时，M2M显示时，数据移动选择（方向：上下和左右）
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    filter_vertical = ("m2m字段",) # 或filter_horizontal = ("m2m字段",)
```
22. ordering，列表时，数据排序规则
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    ordering = ('-id',)
    或
    def get_ordering(self, request):
        return ['-id', ]
```
23. view_on_site，编辑时，是否在页面上显示view on set
```python
view_on_site = False
或
def view_on_site(self, obj):
    return 'https://www.baidu.com'
```
24. radio_fields，详细页面时，使用radio显示选项（FK默认使用select）
```python
radio_fields = {"ug": admin.VERTICAL} # 或admin.HORIZONTAL
```
25. show_full_result_count = True，列表时，模糊搜索后面显示的数据个数样式
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    # show_full_result_count = True # 1 result (12 total)
    # show_full_result_count = False  # 1 result (Show all)
    search_fields = ('user',)
```
26. formfield_overrides = {}，详细页面时，指定现实插件
```python
from django.forms import widgets
from django.utils.html import format_html
 
class MyTextarea(widgets.Widget):
    def __init__(self, attrs=None):
        # Use slightly better defaults than HTML's 20x2 box
        default_attrs = {'cols': '40', 'rows': '10'}
        if attrs:
            default_attrs.update(attrs)
        super(MyTextarea, self).__init__(default_attrs)
 
    def render(self, name, value, attrs=None):
        if value is None:
            value = ''
        final_attrs = self.build_attrs(attrs, name=name)
        return format_html('<textarea {}>\r\n{}</textarea>',final_attrs, value)
 
 
 
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
 
    formfield_overrides = {
        models.models.CharField: {'widget': MyTextarea},
    }
```
27. prepopulated_fields = {}，添加页面，当在某字段填入值后，自动会将值填充到指定字段。
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
 
    prepopulated_fields = {"email": ("user","pwd",)}
PS: DjangoAdmin中使用js实现功能，页面email字段的值会在输入：user、pwd时自动填充
```
28. form = ModelForm，用于定制用户请求时候表单验证
```python
from app01 import models
from django.forms import ModelForm
from django.forms import fields
 
 
class MyForm(ModelForm):
    others = fields.CharField()
 
    class Meta:
        model = models = models.UserInfo
        fields = "__all__"
 
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
 
    form = MyForm
```
29. empty_value_display = "列数据为空时，显示默认值"
```python
@admin.register(models.UserInfo)
class UserAdmin(admin.ModelAdmin):
    empty_value_display = "列数据为空时，默认显示"
 
    list_display = ('user','pwd','up')
 
    def up(self,obj):
        return obj.user
    up.empty_value_display = "指定列数据为空时，默认显示"
```
# 4 自定义ayra进行CRUD
django admin的功能比较强大，比较重，我们可以自己写一个轻量级的类admin的管理工具。

## 4.1 准备工作

### 4.1.1 文件加载顺序
分析djando admin的代码我们知道，在请求进来之前，我们注册的model就已经被包装好了，所以admin.py这个文件是最先被加载的，然后才才可以创建url，完成url的映射

### 4.1.2 让我们的程序被优先加载
每个应用下的admin.py总是在应用启动的时候就被加载了，我们的程序如何加载呢。观察admin的源码我们发现其内部有一个：
```python
def autodiscover():
    autodiscover_modules('admin', register_to=site)
```
这个方法看名字是自动发现，并加载模块的。它是在哪里调用了呢,通过查看源码，我们发现在django.contrib.admin.apps下
```python

class AdminConfig(SimpleAdminConfig):
    """The default AppConfig for admin which does autodiscovery."""

    def ready(self):
        super(AdminConfig, self).ready()
        self.module.autodiscover()
```
ready方法调用了autodiscover.

### 4.1.3 编写调用文件
知道了怎么被调用的，那么我们先来在每个app下定义一个ayra文件.
```python
# arya.py
print('被加载了')
```
然后我们创建一个arya app，然后在他的apps.py文件中做如下修改
```python
class AryaConfig(AppConfig):
    name = 'arya'

    def ready(self):
        from django.utils.module_loading import autodiscover_modules
        autodiscover_modules('arya')
```

### 4.1.4 更换装载应用的方式
在settings.py重，更换下面的导入方式
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'arya.apps.AryaConfig',    # 通过app来导入（顺序无所谓）
    'rbac',
    'main',
]
```
这样，准备工作就做好了，启动的django的时候，同时也会加载每个项目目录下的arya.py 文件了

### 4.1.5 include本质
观察include源码，我们知道include是一个函数，最后返回的是`一个三元组`，通过观察源码，可以的指导它们代表的值是：
- urlconf_module：url列表
- app_name：None
- namespace：None

所以，不使用include时，我们可以有如下操作来分发路由
```python
# 测试代码
from django.shortcuts import HttpResponse

def test1(request):
    return HttpResponse('test1')

def test2(request):
    return HttpResponse('test2')

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^login/', main.views.login),
    url(r'^index/', main.views.index),
    # url(r'^rbac/', include('rbac.urls')) # 等于下面的
    url(r'rbac/', ([
                       url('user/info', test1),
                       url('user/add', ([
                                            url('list', test2),
                                        ], None, None)),
                   ], None, None))
]
```

看起来有些杂乱，分析一下产生的url
1. /rbac/user/info 对应test1
2. /rbac/user/add/list 对应test2

这就是include的本质

## 4.2 为每个model对象生成对应的url
根据django admin的源码的编写方式，我们来模仿一下，用于动态的生成对应的urls。创建arya app，然后创建service模块，用于存放arya.py文件，在该文件内部编写主要实现逻辑

### 4.2.1 注册model类
首先在每一个app下的arya.py中，模仿admin的方式，注册model类
```python
from arya.service import arya
from .models import *

arya.site.register(Userinfo, arya.AryaConfig)
arya.site.register(Permission, arya.AryaConfig)
```
不同的app中调用的是相同的site对象，一般称它为单例模式，并且它存在一个register方法，通过了解admin的源码， 我们知道它包含两个参数，一个是model类，还有一个是默认的一个类对象。

### 4.2.2 service/arya基本实现
根据上一部的分析，注册功能就实现了，我们的arya如下：
```python
from django.shortcuts import HttpResponse

# 根据admin site的源码实现
class AryaConfig(object):

    def __init__(self, model_class, site):
        self.model_class = model_class
        self.site = site

# 根据admin site的源码实现
class AryaSite(object):

    def __init__(self):
        self._register = {}

    def register(self, model_class, arya_class):
        self._register[model_class] = arya_class(model_class, self) # 接受两个参数

# 单例模式
site = AryaSite()
```

### 4.2.3 添加urls及views函数
下面从入口继续编写：
```python
# 项目的urls.py文件
url(r'^arya/',arya.site.url)
```
根据前面include的本质，和adjango admin的源码我们知道，这里的url返回的是一个三元组，即[],None,None。列表就是urlpatterns

```python
# service/arya.py
from django.shortcuts import HttpResponse

class AryaConfig(object):

    def __init__(self, model_class, site):
        self.model_class = model_class
        self.site = site

    @property
    def urls(self):
        from django.conf.urls import url

        # 为不同的model对象，生成不同的urls
        patterns = [
            url(r'^$', self.changelist_view),
            url(r'^add/', self.add_view),
            url(r'^(\d+)/change/', self.change_view),
            url(r'^(\d+)/delete/', self.delete_view),
        ]
        return patterns, None, None   # 必须返回三元组

    # 真正处理CRUD的 views函数
    def changelist_view(self,request):
        return HttpResponse('列表页面')

    def add_view(self,request):
        return HttpResponse('增加页面')

    def change_view(self,request, id):
        return HttpResponse('修改页面')

    def delete_view(self,request, id):
        return HttpResponse('删除页面')


class AryaSite(object):

    def __init__(self):
        print('执行了')
        self._register = {}

    def register(self, model_class, arya_class):
        self._register[model_class] = arya_class(model_class, self)

    @property
    def url(self):
        from django.conf.urls import url
        patterns = []

        # 为不同modellei 生成不同的include对象，包含CRUD的urls
        for model_class, arya_class in self._register.items():
            pt = url('^{}/{}/'.format(model_class._meta.app_label, model_class._meta.model_name), arya_class.urls)
            patterns.append(pt)

        return patterns, None, None

site = AryaSite()
```
> 注意：urls路径匹配结尾处，要添加/

为什么要保存model呢？ 观察上面的views函数，内部存在model类对象的话，是不是就可以进行CRUD了呢？配上一个templates模版，就可以完成页面渲染了。
```python
self.model_class.objects.all()
```

