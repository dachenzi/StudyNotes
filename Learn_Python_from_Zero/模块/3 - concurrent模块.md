<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 conturrent包](#1-conturrent包)
- [2 future模块](#2-future模块)
- [3 ThreadPoolExecutor对象](#3-threadpoolexecutor对象)
- [4 Future对象](#4-future对象)
- [5 ProcessPoolExecutor对象](#5-processpoolexecutor对象)
- [6 支持上下文管理](#6-支持上下文管理)
- [7 异步爬网站的小例子](#7-异步爬网站的小例子)

<!-- /TOC -->

# 1 conturrent包
conturrent包内只包含了一个future模块，它为异步执行调用提供了高级的接口。

# 2 future模块
主要提供了两个用于异步执行的类：
- ThreadpPoolExecutor：异步调用的线程池的执行器
- ProcessPoolExecutor：异步调用的进程池的执行器
> 两者实现相同的接口，该接口由抽象Executor类定义。

# 3 ThreadPoolExecutor对象
提供了以下方法用于构建和执行多线程任务：

|方法|含义|
|-----|----|
|ThreadpPoolExecutor(max_workers=None, thread_name_prefix='')|池中至多创建max_workers个线程的池来同时异步执行，返回Executor实例，`如果max_workers没有指定，那么会开启cpu核数(或1) * 5 个线程`
|submit(fn, *args, **kwargs)|提交执行的函数及其参数，返回Future类的实例
|shutdown(wait=True)|清理池  

基本使用:
```python
import concurrent.futures

def worker(name, count=1000):
    total = 0
    for i in range(count):
        total += 1
    return name, total

fs = []
executor = concurrent.futures.ThreadPoolExecutor(max_workers=3)  # 线程池为3
for i in range(3):
    f = executor.submit(worker, 'worker-{}'.format(i), 10000)    # 提交即异步执行
    fs.append(f)

for f in fs:
    print(f.result())    # 获取结果
    
executor.shutdown()  # 等待所有进程执行完毕后，关闭线程池。

# ('worker-0', 10000)
# ('worker-1', 10000)
# ('worker-2', 10000)
```
submit返回的是一个Future对象。当执行的任务多余线程池的数量时，超出的任务会被阻塞，即等待执行状态，打印对象即可查看
```python
<Future at 0x2456e953cc0 state=running>
<Future at 0x2456ec65588 state=running>
<Future at 0x2456ec6d7b8 state=running>  # 运行
<Future at 0x2456ec6dac8 state=pending>  # 阻塞等待
<Future at 0x2456eeba0f0 state=pending>
```

# 4 Future对象
包含了许多针对多进程/线程执行的相关结果的方法。

|方法|含义|
|----|----|
|done()|如果调用被成功取消或者执行完成，返回True|
|cancelled()|如果调用被成功取消，返回True|
|running()|如果正在运行且不能被取消，返回True|
|cancel()|尝试取消调用。如果已经执行且不能取消返回False，否则成功取消返回True|
|result(timeout=None)|获取执行结果，timeout为None，表示阻塞等待。<br>timeout到期还未返回，抛出concurrent.futures._base.TimeoutError异常
|exception(timeout=None)|获取返回的异常，timeout为None，表示阻塞等待。<br>timeout到期还未返回，抛出concurrent.futures._base.TimeoutError异常

# 5 ProcessPoolExecutor对象
实现了和ThreadPoolExecutor一样的接口，所以只需要把上面例子的ThreadPoolExecutor换为ProcessPoolExecutor即可完成进程的创建。
```python
import concurrent.futures

def worker(name, count=1000):
    total = 0
    for i in range(count):
        total += 1
    return name, total

if __name__ == '__main__':

    fs = []
    executor = concurrent.futures.ProcessPoolExecutor(max_workers=3)
    for i in range(5):
        f = executor.submit(worker, 'worker-{}'.format(i), 10000)
        print(f)
        fs.append(f)

    for f in fs:
        print(f.result())

# ('worker-0', 10000)
# ('worker-1', 10000)
# ('worker-2', 10000)
```

# 6 支持上下文管理
当进程吃使用完毕时，我们需要调用shutdown方法来关闭它。ThreadPoolExecutor和ProcessPoolExecutor实现了上下文管理，当我们离开时，会自动调用shutdown方法，非常方便
```python
import concurrent.futures

def worker(name, count=1000):
    total = 0
    for i in range(count):
        total += 1
    return name, total

if __name__ == '__main__':

    fs = []
    executor = concurrent.futures.ProcessPoolExecutor(max_workers=3)
    with executor:
        for i in range(5):
            f = executor.submit(worker, 'worker-{}'.format(i), 10000)
            print(f)
            fs.append(f)

        for f in fs:
            print(f.result())
```
> 该库统一了线程池，进程池的调用，简化了编程。唯一的缺点是无法设置线程的名称，但这都无所谓了。

# 7 异步爬网站的小例子

```python
import concurrent.futures
import requests


def get_html(url):
    response = requests.get(url)
    length = len(response.text)
    return length, response.text


if __name__ == '__main__':
    url_list = [
        'http://www.baidu.com',
        'http://www.sina.com',
        'http://www.qq.com',
    ]
    with concurrent.futures.ProcessPoolExecutor(max_workers=3) as executor:
        fs = [executor.submit(get_html, url) for url in url_list]
        for f in fs:
            print(f.result())
```
