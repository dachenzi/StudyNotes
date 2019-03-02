<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 线程同步](#1-线程同步)
    - [1.1 Event](#11-event)
        - [1.1.1 什么是Flag？](#111-什么是flag)
        - [1.1.2 Event原理](#112-event原理)
        - [1.1.3 吃包子](#113-吃包子)
    - [1.2 Lock](#12-lock)
        - [1.2.1 lock方法](#121-lock方法)
        - [1.2.2 计数器](#122-计数器)
        - [1.2.3 非阻塞锁](#123-非阻塞锁)
        - [1.2.4 锁应用场景](#124-锁应用场景)

<!-- /TOC -->
# 1 线程同步
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;线程同步，线程间协同，通过某种技术，让一个线程访问某些数据时，其他线程不能访问这些数据，直到该线程完成对数据的操作后。不同的操作系统有多种实现方式。比如临界区(Critical Section)、互斥锁(Mutex)、信号量(Semaphore)、事件(Event)等。
## 1.1 Event
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Event是线程间通讯机制最简单的实现，使用一个内部标记flag，主要提供了三个方法wait、clear、set，通过操作flag来控制线程的执行。
- clear()：将'Flag'设置为False。
- set()：将'Flag'设置为True。
- wait(timeout=None)：等待'Flag'为True后，继续执行(timeout为超时时间，否则永远等待)。
- is_set(): 判断'Flag'是否为
### 1.1.1 什么是Flag？
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Event对象在全局定义了一个'Flag'，如果'Flag'值为 False，那么当程序执行 Event对象的wait方法时就会阻塞，如果'Flag'值为True，那已经阻塞的wait方法会继续执行。
### 1.1.2 Event原理
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在使用threading.Event 实现线程间通信时：使用threading.Event可以使一个线程等待其他线程的通知，我们把这个Event传递到线程对象中，Event默认内置了一个标志，初始值为False。一旦该线程通过wait()方法进入等待状态，直到另一个线程调用该Event的set()方法将内置标志设置为True时，该Event会通知所有等待状态的线程恢复运行。
### 1.1.3 吃包子
有下面代码，大欣负责吃包子，厨师负责做包子，只有厨师做好了，大欣才能开始吃。
```python
import time
import random

event = threading.Event()

def consumer():
    current = threading.current_thread()
    print('{} 等着吃包子...'.format(current.name))
    event.wait()
    print('包子来了，我正在吃')
    for i in range(1,5):
        time.sleep(random.randrange(1,3))
        print('{} 吃 包子-{}。'.format(current.name,i))


def chef():
    current = threading.current_thread()
    print('{} 正在做包子'.format(current.name))
    time.sleep(3)
    print('{} 包子做好了，下班'.format(current.name))
    event.set()

chef = threading.Thread(target=chef,name='大厨师')
chef.start()

consumer = threading.Thread(target=consumer,name='大欣')
consumer.start()
```
运行consumer时，包子还没做，所以只能等着，等chef做完了以后，设置了event为True，这时consumer就开始吃了。wait还可以指定等待时间，比如chef做的太慢了，consumer不吃了。
```python
import time
import random

event = threading.Event()

def consumer():
    current = threading.current_thread()
    print('{} 等着吃包子...'.format(current.name))
    if not event.wait(2):
        print('太慢了，不吃了')
    else:
        print('包子来了，我正在吃')
        for i in range(1,5):
            time.sleep(random.randrange(1,3))
            print('{} 吃 包子-{}。'.format(current.name,i))

def chef():
    current = threading.current_thread()
    print('{} 正在做包子'.format(current.name))
    time.sleep(8)
    print('{} 包子做好了，下班'.format(current.name))
    event.set()

chef = threading.Thread(target=chef,name='大厨师')
chef.start()

consumer = threading.Thread(target=consumer,name='大欣')
consumer.start()
```
当Event被set后，wait的返回值就是True，如果wait(2)，在2秒内，Event没有被set，那么返回值是False。
## 1.2 Lock
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;由于线程间的数据是共享的，当我们多个线程操作一个相同的用户的数据时，有可能造成混乱，如下例子：
```python
import threading
import time

n = 10

def work():
    global n
    while n > 0:
        time.sleep(0.01)
        n -= 1

if __name__ == '__main__':
    t_l = []
    for i in range(10):
        t = threading.Thread(target=work)
        t_l.append(t)
        t.start()

    for t in t_l:
        t.join()

    print(n)
```
　　我们认为结果应该是0，但是结果可能不如人意，因为进程间共享数据的问题，多个进程同时修改共享数据时，由于GIL的存在同一时刻只有1个线程在运行。当n的值为1的时候，10个子线程很有可能同时判断成功，再要修改的时候被挂起(时间片用完)，等到真正回来修改的时候，n已经被其他线程改过来！所以如果要保持数据的正确性，那么就需要牺牲性能，即使用锁机制。  
1. 创建一个锁,由于进程内的线程共享进程数据，那么不需要传递，就可以直接调用
```python
import threading
mutex = threading.Lock()  
```
2. 加锁解锁
Lock内部实现了__enter__和__exit__的方法，所以我们可以使用两种方式来加锁或者解锁。
```python 
# 调用方式一
mutex.acquire()    # 加锁
'''code'''
mutex.release()    # 解锁

# 调用方式二
with mutex:
    '''code'''   # 离开wit代码段，自动解锁
```
### 1.2.1 lock方法
名称|含义
----|----|
acquire(blocking=True,timeout=-1)|获取锁并加锁<br>blocking:表示是否阻塞，默认为True表示阻塞。<br>timeout：表示阻塞超时时间。<br>当blocking为非阻塞时，timeout不能进行设置
release()| 释放锁，可以从任何线程上调用释放.<br>已上锁，调用时会释放锁，即重置为unlocked。<br>未上锁，调用时会抛出RuntimeError异常

所以，上面的例子可以有如下修改:
```python
import threading
import time

mutex = threading.Lock()
n = 10
def work():
    global n
    while True:
        mutex.acquire()
        if n > 0:
            time.sleep(1)
            n -= 1
            mutex.release()
        else:
            mutex.release()
            break

if __name__ == '__main__':
    t_l = []
    for i in range(10):
        t = threading.Thread(target=work)
        t_l.append(t)
        t.start()

    for t in t_l:
        t.join()

    print(n)
```
在判断的时候就开始加锁，在修改完毕的时候解锁。这样加锁的情况下我们发现运行时间变长了，那是因为只有抢到锁的线程才可以工作(穿行执行)，
### 1.2.2 计数器
有下面一个计数器类，来看如何加锁
```python
import threading

class Counter:
    def __init__(self):
        self._value = 0

    @property
    def value(self):
        return self._value

    def inc(self):
        self._value += 1

    def dec(self):
        self._value -= 1

def calc(c:Counter):
    for _ in range(1000):
        for i in range(-50,50):
            if i < 0:
                c.dec()
            else:
                c.inc()

c = Counter()
lst = []
for i in range(10):
    t = threading.Thread(target=calc, args=(c,))
    lst.append(t)
    t.start()

for t in lst:
    t.join()

print(c.value)
```
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;在需要调用和修改的地方加锁，修改完毕后解锁，是锁使用的基本原则，一般来说，加锁就要解锁，但是加锁和解锁之间会有一些代码要执行，如果出现异常，那么锁是无法释放的，但是当前线程已经终止了，这种情况一般称为死锁，可以添加异常处理，来确保锁一定被释放。
```python
import threading

mutex = threading.Lock()
class Counter:
    def __init__(self):
        self._value = 0

    @property
    def value(self):
        return self._value

    def inc(self):
        try:  # 添加异常处理，即便时崩溃也可以释放锁
            mutex.acquire()
            self._value += 1
        finally:
            mutex.release()

    def dec(self):
        with mutex:  # 上下文管理写法
            self._value -= 1

def calc(c:Counter):
    for _ in range(1000):
        for i in range(-50,50):
            if i < 0:
                c.dec()
            else:
                c.inc()

c = Counter()
lst = []
for i in range(10):
    t = threading.Thread(target=calc, args=(c,))
    lst.append(t)
    t.start()

for t in lst:
    t.join()
```
当然这里也可以为每一个计数器实例对象初始化一个自己的锁，如果用全局锁，那么不同的计数器实例，会相互影响(因为多个实例，共享一把锁)，因为不同实例的结果是不同的，所以建议为每个实例构建一个自己的锁。
```python
class Counter:
    def __init__(self):
        self._value = 0
        self._lock = threading.Lock()

    @property
    def value(self):
        return self._value

    def inc(self):
        try: 
            self._lock.acquire()
            self._value += 1
        finally:
            self._lock.release()

    def dec(self):
        with self._lock:  
            self._value -= 1
```
### 1.2.3 非阻塞锁
当lock.acquire(False)时，该锁就是非阻塞锁了，调用时，仅获取一次，如果获取到那么返回True，否则返回False。
```python
import time
import threading
import random
import logging

logging.basicConfig(level=logging.INFO, format='%(asctime)s %(threadName)s %(thread)s %(message)s')

def worker(tasks:list):
    for task in tasks:
        if task.lock.acquire(False):  # 抢到任务
            time.sleep(random.randrange(1,3))  # 模拟执行任务需要的时间
            logging.info('{} I am working for {}'.format(threading.current_thread().name, task))
        else:  # 没有抢到任务
            logging.info('{} not get {}'.format(threading.current_thread().name,task))

class Task:
    def __init__(self,name):
        self.task = name
        self.lock = threading.Lock()

    def __repr__(self):
        return self.task

    __str__ = __repr__

if __name__ == '__main__':
    task_list = [Task('task{}'.format(x)) for x in range(1,11)]   # 构建任务列表
    for _ in range(10):
        threading.Thread(target=worker,args=(task_list,)).start()  # 开启多线程执行任务
```
10个任务，交给10个线程运行，谁抢到哪个线程，谁就运行。
### 1.2.4 锁应用场景
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;锁适用于访问和修改同一个共享资源的时候，即读写同一个资源的时候。但如果都是读取的话，就不需要了。使用锁的时候有一下几点需要特别注意：
1. 少于锁，必要时用锁，使用了锁，多线程访问被被锁资源时，就成了穿行的了，要么排队，要么争抢执行。
2. 加锁时间越短越好，不需要时立即释放锁。
3. 一定要避免死锁。
