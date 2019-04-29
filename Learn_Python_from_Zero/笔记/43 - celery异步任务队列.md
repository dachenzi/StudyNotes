# 1 Celery概述
关于celery的定义，首先来看官方网站：
__Celery(芹菜) 是一个简单、灵活且可靠的，处理大量消息的分布式系统，并且提供维护这样一个系统的必需工具。__

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;简单来看，是一个基于python开发的分布式异步消息任务队列，持使用任务队列的方式在分布的机器、进程、线程上执行任务调度。通过它可以轻松的实现任务的异步处理， 如果你的业务场景中需要用到异步任务，就可以考虑使用celery， 举几个实例场景中可用的例子:

1. 你想对100台机器执行一条批量命令，可能会花很长时间 ，但你不想让你的程序等着结果返回，而是给你返回 一个任务ID,你过一段时间只需要拿着这个任务id就可以拿到任务执行结果，在任务执行ing进行时，你可以继续做其它的事情。 　　
2. 你想做一个定时任务，比如每天检测一下你们所有客户的资料，如果发现今天 是客户的生日，就给他发个短信祝福 。　　

Celery 在执行任务时需要通过一个中间人(消息中间件)来接收和发送任务消息，以及存储任务结果，完整的中间人列表请查阅官方网站

PS：任务队列是一种在线程或机器间分发任务的机制。
PS：消息队列的输入是工作的一个单元，称为任务，独立的工作（Worker）进程持续监视队列中是否有需要处理的新任务。 

# 2 Celery简介
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Celery 系统可包含多个职程和中间人，以此获得高可用性和横向扩展能力，其基本架构由三部分组成，`消息中间件`（message broker），`任务执行单元`（worker）和`任务执行结果存储`（task result store）组成。
- `消息中间件`，Celery本身不提供消息服务，但是可以方便的和第三方提供的消息中间件集成，一般使用rabbitMQ or Redis，当然其他的还有MySQL以及Mongodb。
- `任务执行单元`，Worker是Celery提供的任务执行的单元，worker并发的运行在分布式的系统节点中。
- `任务结果存储`，Task result store用来存储Worker执行的任务的结果，Celery支持以不同方式存储任务的结果，包括Redis，MongoDB，Django ORM，AMQP等。

Celery的主要特点：
1. 简单：一单熟悉了celery的工作流程后，配置和使用还是比较简单的
2. 高可用：当任务执行失败或执行过程中发生连接中断，celery 会自动尝试重新执行任务
3. 快速：一个单进程的celery每分钟可处理上百万个任务
4. 灵活： 几乎celery的各个组件都可以被扩展及自定制

![celery](photo/celery.png)

根据前面的介绍，我们可以得出如下流程图：
1. 用户应用程序将任务(已经在celery app中注册的)放入Broker中。
2. 多个worker通过Broker获取任务并执行。
3. worker执行完成后，会把任务的结果、状态等信息返回到Broker中存储，供用户程序读取。

PS：Celery 用消息通信，通常使用中间人（Broker）在客户端和职程(worker)间斡旋。这个过程从客户端向队列添加消息开始，之后中间人把消息派送给职程(worker)。

# 3 Celery模块的基本使用
要使用Celery需要先安装celery模块，下面的例子使用Python3进行举例,使用redis作为消息中间人的角色。

## 3.1 利用pip3命令安装celery模块
```python
pip3 install celery

# 测试是否成功安装
[root@namenode ~]# python3
Python 3.6.4 (default, Dec 21 2017, 17:26:43)
[GCC 4.4.7 20120313 (Red Hat 4.4.7-16)] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import celery      # 没有报错表示模块安装正常
>>>
```
PS：如果你是编译安装的Python3，执行以上步骤后不一定代表正确安装，还需要在命令行下执行celery命令，如果报错请参考这篇文章：[Python3安装Celery模块后执行Celery命令报错](http://www.cnblogs.com/dachenzi/p/8082129.html)


## 3.2 创建一个celery application
用来定义任务列表，这里任务文件的名称叫做task.py(注意后面会用到文件名)。

### 3.2.1 通过实例定制化celery
在实例化celery应用时，对celery进行配置
```python
from celery import Celery
 
app = Celery('task',　　# 是当前模块的名称，这个参数是必须的，这样的话名称可以自动生成
    broker="redis://10.0.0.3:6379/0",     # 中间人的地址
    backend="redis://10.0.0.3:6379/1"　　　# 结果数据存放地址
)
 
@app.task　　　　　# 使用celery标识一个任务，多个任务都需要使用该装饰器　
def add(x,y):　　　　
    return x+y
```

### 3.2.2 通过配置信息定制化celery
实例化celery应用后，通过conf对celery进行定制化配置（可以将配置项写在配置文件中，一般使用这种方式）
```python
from celery import Celery

app = Celery('mycelery')

# 全局配置(启用UTC时间，并配置时区)
app.conf.update(enable_utc=True, timezone='Asia/ShangHai')
# 派发任务要使用的队列(broker)
app.conf.broker_url = 'redis://:daxin@10.0.0.10:6379/0'
# 避免重复执行(执行超过visibility_timeout指定的时间，会重新派发任务，一般的解决办法是把这个时间调长
app.conf.broker_transport_options = {'visibility_timeout': 3600*12}
# 任务执行结果存储
app.conf.result_backend = 'redis://:daxin@10.0.0.10:6379/1'
```

其他中间人的配置
```python
# rabbitmq
broker = 'amqp://user:password@ip:5672//'

# redis
broker = 'redis://:passwordf@ip:6379/db'
```
## 3.3 运行一个worker
当然这里可以执行多个worker，在命令行下执行(在test.py文件所在目录下执行)
```bash
celery -A task worker --loglevel=debug
celery -A task worker -l debug  # 使用-l设置输出日志级别也可以
```
参数含义：
- -A参数表示的是Celery APP的名称，task就是APP的名称(`应用文件名`)
- worker表示是一个执行任务角色
- loglevel=info记录日志类型默认是info。

## 3.4 发布(调用)任务
这里在task.py所在的目录调用Python解释器执行，这样可以方便的引入这个app。
```python
>>> from task import add
>>> add.delay(4, 4)
```
- 使用 `delay()` 方法来调用任务，这是 `apply_async()` 方法的快捷方式，该方法允许你更好地控制任务执行(异步执行)。
- 这个任务已经由之前启动的职程(worker)执行，可以查看职程的控制台输出来验证。

PS：调用任务会返回一个 `AsyncResult` 实例，可用于检查任务的状态，等待任务完成或获取返回值（如果任务失败，则为异常和回溯）。 但这个功能默认是不开启的，需要设置一个 Celery 的结果后端(配置了backend才会生效)。

## 3.5 观察执行结果
下面是worker在终端输出的日志
```python 
[2017-12-22 15:34:36,819: INFO/MainProcess] Received task: task.add[b881bbea-e729-444e-9efd-434fa6e43f57]
[2017-12-22 15:34:46,828: INFO/ForkPoolWorker-1] Task task.add[b881bbea-e729-444e-9efd-434fa6e43f57] succeeded in 10.008738335978705s: 3
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;从上面日志可以看到，当worker从中间人那获取任务后，会生成一个task ID，这里我们和之前发送任务返回的AsyncResult对比我们发现，每个task都有一个唯一的ID，第二行说明了这个任务执行succeed,执行结果为3。

PS：通过检查redis的数据也可以查看结果信息
```python
root@namenode python]# redis-cli
127.0.0.1:6379[1]> keys *
1) "celery-task-meta-31f2f75d-20d9-4bb6-8b04-079ef65afbab"
2) "celery-task-meta-334f0e9b-87f6-461a-a068-571bd3773691"
3) "celery-task-meta-57bb002b-017d-43bc-a88a-6d470a9dfe95"
4) "celery-task-meta-05161a38-3fd9-4be1-bae9-dc674d30296c"
127.0.0.1:6379[1]> get celery-task-meta-cddbc105-ce52-4d30-bab0-2114749dcf99
"{\"status\": \"SUCCESS\", \"result\": 4, \"traceback\": null, \"children\": [], \"task_id\": \"cddbc105-ce52-4d30-bab0-2114749dcf99\"}"
127.0.0.1:6379[1]>
``` 
- 可以看到有很多celery任务执行的结果
- 使用get就可以查看存储的信息(类型为string)

## 3.6 AsyncResult 实例常用方法
通过获取异步执行对象的返回值来获取AsyncResult实例对象

|名称|含义|
|----|----|
|forget()|清除执行结果|
|revoke(terminate=True)|取消任务，terminate默认为False，正在执行的无法中断，为True时，正在执行的也会被中断|
|get(timeout=None,<br>propagate=False)|阻塞等待获取结果(可以重复拿)<br>1.timeout为超时时间,如果在指定时间内没有返回会报异常，否则永远等待<br>2.propagate为True时，可以美化输出信息<br>3.倘若任务抛出了一个异常， get() 会重新抛出异常。<br>4.如果celery没有配置backend，那么执行get方法将会异 |
|ready()|结果为 True/False 表示 完成/未完成|
|successful()|执行成功时返回True|
|failed()|执行失败时返回True|
|result|获取结果|
|state|任务当前的状态<br>1.SUCCESS:运行成功<br>2.PENDING:等待被执行<br>3.STARTED:已经开始执行<br>4.RETRY:正在重试<br>5.FAILURE:执行失败|
|task_id|任务的id|
|traceback|获取原始的回溯信息|

导入AsyncResult对象，来查看结果的方法
```python
from celery.result import AsyncResult
from task import app

result = AsyncResult(id='d01289e4-9034-4a46-8c63-7bca45946efa', app=app)
print(result.task_id)  # d01289e4-9034-4a46-8c63-7bca45946efa
print(result.successful())  # True
```
注意：AsyncResult对象，需要一个id，和它对应的app，才可以解析到结果。

## 3.7 定时执行一次
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;celery内置定时任务和crontab的效果一样，现在我的需求是在某一时刻来执行(比如at命令)，那么可以使用apply_async的eta参数

```python
from task import func
import datetime

ctime = datetime.datetime.now()
print(ctime)
utc_time = datetime.datetime.utcfromtimestamp(ctime.timestamp())
print(utc_time)

addten = datetime.timedelta(seconds=20)
utc_time += addten

# utc_time = datetime.datetime(year=2019, month=4, day=11, hour=9, minute=41)  # 直接构建，也需要是utc时间

result = func.apply_async(args=[1, 2], eta=utc_time)
```

# 4 多目录结构
上面只是基础的用法，在生产中我们往往不这样使用，一般的目录结构如下
```python
├── celery_task
│   ├── account.py
│   ├── celery.py  # 配置文件
│   └── tasks.py
└── givetask.py  # 分发任务
```
说明：
- 一般会在项目目录下直接创建一个以celery开头的文件夹用于存放celery的配置文件和任务
- 必须存在一个celery.py文件，连接等相关的配置需要在这里创建
- task函数所在的文件名称可以自定义,只需要在celery.py中，列出即可

下面是一个典型的celery.py的内容
```python
# 必须是celery名字，celery连接相关的配置(配置文件)
from celery import Celery

cel = Celery('cel',
             broker='redis://10.0.0.10:6379',
             backend='redis://10.0.0.10:6379',
             include=['celery_task.tasks', 'celery_task.account'])

# 时区
cel.conf.timezone = 'Asia/Shanghai'

# 使用使用UTC
cel.conf.enable_utc = False
```
- include列表中，包含所有task的文件名称即可，注意要从celery_task目录开始写起。
- 调用时，可以在celery_task目录同级目录，来导入task函数，来传递任务。

## 4.1 运行
需要注意的是必须在celery_task目录同级目录下，执行命令运行worker服务
```python
celery -A celery_task worker -l info
```

## 4.2 后台启动多进程celery work
前面的启动方法一次只能启动一个，会在前台显示并打印日志，终端退出，那么celery进程也会退出，当然我们可以使用&来进行后台启动，日志可以使用-q来禁止输出，但是这毕竟不是最佳方法，官方提供了多进程后台的启动方式利用 celery  multi 启动

```python
# 后台启动 celery worker进程
 
celery multi start work_1 -A appcelery 
 
# work_1 为woker的名称，可以用来进行对该进程进行管理
```
## 4.3 常用其他命令
多进程相关
```python
# 停止worker进程,有的时候这样无法停止进程，就需要加上-A 项目名，才可以删掉
celery multi stop WOERNAME      

# 重启worker进程
celery multi restart WORKNAME       

# 查看该项目运行的进程数
celery status -A celery_task      
```

# 5 Celery的定时任务
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;celery支持定时任务，设定好任务的执行时间，celery就会定时自动帮你执行， 这个定时任务模块叫celery beat，类似于Linux中的crontab。主要有两种类型：每隔多久执行一次或者定期执行。

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于是定期执行，所以celery的定时任务主要有两类进程，即完成任务的worker进程和分发任务的beat进程。

下面是一个定时任务程序
```python
from celery import Celery
from celery.schedules import crontab
 
app = Celery('celeryperiodoc',
    broker="redis://10.0.0.10:6379/0",
    backend="redis://10.0.0.10:6379/1"
)
 
@app.on_after_configure.connect        # 定时任务必须要用的装饰器
def setup_periodic_tasks(sender, **kwargs):      # sender是必须传递的参数，类似于django的requests一样
    # Calls test('hello') every 10 seconds.
    sender.add_periodic_task(10.0, test.s('hello'), name='add every 10')    # 添加一个定时任务，10.0表示每隔10秒，test.s表示给test函数传递的参数，name表示任务名称
 
    # Calls test('world') every 30 seconds
    sender.add_periodic_task(30.0, test.s('world'), expires=10)
 
    # Executes every Thureday at 19:52 p.m.
    sender.add_periodic_task(          
        crontab(hour=19, minute=52,day_of_week='thu'),      # 利用crontab设定定时执行
        test.s('Happy Mondays!'),
    )
 
@app.task
def test(arg):
    print(arg)
```
启动一个beat进程：celery -A celeryFIleName beat


```python
 [root@namenode celerymodule]# celery -A celeryperiodoc beat
celery beat v4.1.0 (latentcall) is starting.
__    -    ... __   -        _
LocalTime -> 2017-12-26 20:13:54
Configuration ->
    . broker -> redis://10.0.0.10:6379/0
    . loader -> celery.loaders.app.AppLoader
    . scheduler -> celery.beat.PersistentScheduler
    . db -> celerybeat-schedule
    . logfile -> [stderr]@%WARNING
    . maxinterval -> 5.00 minutes (300s)
```
启动一个worker进程： celery -A celeryFIleName worker

日志结果
```python
[2017-12-26 20:24:14,267: INFO/MainProcess] Received task: celeryperiodoc.test[641d0e17-ff26-4791-acca-8c0cce8d6709] 
[2017-12-26 20:24:14,269: WARNING/ForkPoolWorker-1] hello
[2017-12-26 20:24:14,276: INFO/ForkPoolWorker-1] Task celeryperiodoc.test[641d0e17-ff26-4791-acca-8c0cce8d6709] succeeded in 0.007056772010400891s: None
 
[2017-12-26 20:24:24,269: INFO/MainProcess] Received task: celeryperiodoc.test[25b3ea6a-0405-4431-a1e1-840f5dbdc57e]   expires:[2017-12-26 12:24:34.266983+00:00]
[2017-12-26 20:24:24,270: WARNING/ForkPoolWorker-1] world
[2017-12-26 20:24:24,271: INFO/ForkPoolWorker-1] Task celeryperiodoc.test[25b3ea6a-0405-4431-a1e1-840f5dbdc57e] succeeded in 0.0009060979355126619s: None
 
[2017-12-26 20:24:24,271: INFO/MainProcess] Received task: celeryperiodoc.test[b5dce374-4596-4282-987a-494e2ede365c] 
[2017-12-26 20:24:24,273: WARNING/ForkPoolWorker-1] hello
[2017-12-26 20:24:24,273: INFO/ForkPoolWorker-1] Task celeryperiodoc.test[b5dce374-4596-4282-987a-494e2ede365c] succeeded in 0.0006978920428082347s: None
 
[2017-12-26 20:24:34,270: INFO/MainProcess] Received task: celeryperiodoc.test[1a67a24a-0348-416d-a4c0-fc033108cda1] 
[2017-12-26 20:24:34,271: WARNING/ForkPoolWorker-1] hello
[2017-12-26 20:24:34,272: INFO/ForkPoolWorker-1] Task celeryperiodoc.test[1a67a24a-0348-416d-a4c0-fc033108cda1] succeeded in 0.0008945289300754666s: None
 
[2017-12-26 20:24:44,272: INFO/MainProcess] Received task: celeryperiodoc.test[e6853e92-9c42-4c49-8148-5d575268f3a7] 
[2017-12-26 20:24:44,273: WARNING/ForkPoolWorker-1] hello
[2017-12-26 20:24:44,274: INFO/ForkPoolWorker-1] Task celeryperiodoc.test[e6853e92-9c42-4c49-8148-5d575268f3a7] succeeded in 0.0009920489974319935s: None
```
- 观察hello的输出间隔，发现是每隔10秒输出一次
- 观察world的输出间隔，可以看出是每隔30秒输出一次

其他:执行完毕后会在当前目录下产生一个二进制文件celerybeat-schedule,该文件用于存放上次执行结果：
- 如果存在celerybeat-schedule文件，那么读取后根据上一次执行的时间，继续执行。
- 如果不存在celerybeat-schedule文件，那么会立即执行一次。
- 如果存在celerybeat-schedule文件，读取后，发现间隔时间已过，那么会立即执行。

# 6 在django中集成celery
最常用的场景，应该就是用户注册时的发送邮件了，下面以完成邮件发送为目的来，编写celery任务

## 6.1 编写celery.py配置文件
在项目的目录下(与settings.py文件同级)，创建celery文件
```python
from __future__ import absolute_import, unicode_literals
import os
from celery import Celery

# set the default Django settings module for the 'celery' program.
os.environ.setdefault('DJANGO_SETTINGS_MODULE', 'stu_crm.settings')

app = Celery('stu_crm')

# Using a string here means the worker doesn't have to serialize
# the configuration object to child processes.
# - namespace='CELERY' means all celery-related configuration keys
#   should have a `CELERY_` prefix.
app.config_from_object('django.conf:settings', namespace='CELERY')

# Load task modules from all registered Django app configs.
app.autodiscover_tasks()

# ------------------------------------
## 后面的是celery的配置信息，可以放在setting.py中，通过配置来调用
app.conf.update(utc_time=True, timezone='Asia/Shanghai')
app.conf.broker_url = 'redis://:daxin@10.0.0.10:6379/0'
app.conf.result_backend = 'redis://:daxin@10.0.0.10:6379/1'
app.conf.broker_transport_option = {'visibility_timeout': 3600 * 12}
```

## 6.2 编写__init__.py文件
需要修改项目的__init__.py文件，添加如下内容
```python
from __future__ import absolute_import, unicode_literals

# This will make sure the app is always imported when
# Django starts so that shared_task will use this app.
from .celery import app as celery_app

__all__ = ('celery_app',)
```

## 6.3 编写tasks.py文件(发送邮件)
在各个项目内，创建tasks.py文件，针对不同的项目编写不同的task任务
```python
# 需要引入
from __future__ import absolute_import, unicode_literals
# 通过app.task来标识celery任务函数，也可以使用from celery import shared_task，shared_task来装饰
from stu_crm.celery import app
from django.conf import settings
from django.core.mail import send_mail

# @shared_task(name='send')
@app.task(name='send')
def sendmail():
    send_mail(
        subject='Test测试邮件',
        message='这是一封测试邮件',
        from_email=settings.EMAIL_HOST_USER,
        recipient_list=['287990400@qq.com'],
        fail_silently=False
    )
```
这里利用了django的提供的邮件发送功能，所以需要添加如下关于邮箱的配置，在settings.py中
```python
EMAIL_HOST_USER = 'beyondlee2011@126.com'
EMAIL_PORT = 465
EMAIL_HOST_PASSWORD = 'xxxxxxxxxxx'
EMAIL_HOST = 'smtp.126.com'
EMAIL_USE_SSL = True   # 是否使用SSL
EMAIL_USE_TLS = False
```
当然也可以使用yagmail来发送邮件：[yagmail发送邮件](http://www.cnblogs.com/dachenzi/p/8655022.html)


## 6.4 编写views视图函数
通过视图函数来模拟调用过程。
```python
from .tasks import sendmail

def send(request):
    try:
        sendmail.delay()
        # add.delay(10, 20)
    except Exception as e:
        return HttpResponse(e)
    return HttpResponse('ok')
```

## 6.5 启动worker
发布任务的流程已经创建完毕，接下来，就需要启动worker来监控broker获取任务并执行了。
在manage.py同级目录下：
```python
celery -A crm worker -l info
```
这样就完成了启动。接下来只需要访问定义好的url，那么就可以发布任务了。