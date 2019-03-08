<font size=5 face='微软雅黑'>__文章目录__</font>
<!-- TOC -->

- [1 Rlock可重入锁](#1-rlock可重入锁)
- [2 Condition](#2-condition)

<!-- /TOC -->
# 1 Rlock可重入锁
就是可以重复进入的锁，也叫做'递归锁'。这种锁相对Lock来说，有其他三个特点：
1. 谁拿到谁释放。如果线程A拿到锁，线程B无法释放这个锁，只有A可以释放；
2. 同一线程可以多次拿到该锁，即可以acquire多次；
3. acquire多少次就必须release多少次，只有最后一次release才能改变RLock的状态为unlocked
> Rlock是线程相关的锁。

它具有和lock对象相同的方法：
名称|含义
----|----|
acquire(blocking=True)|获取锁并加锁<br>blocking:表示是否阻塞，默认为True表示阻塞。
release()| 释放锁，可以从任何线程上调用释放.<br>已上锁，调用时会释放锁，即重置为unlocked。<br>未上锁，调用时会抛出RuntimeError异常
> 由于在一个进程内可以获取多次，所以即便是block=True也不会阻塞，但是在一个线程没有释放完，其他线程在获取时就会阻塞了。
```python
import threading

def worker(l):
    print(threading.current_thread().ident)  # 37180
    print(l)    # <unlocked _thread.RLock object owner=0 count=0 at 0x000001CA920FA3C8>
    l.acquire()
    l.acquire()
    print(l)    # <locked _thread.RLock object owner=37180 count=2 at 0x000001CA920FA3C8>
    l.release()
    print(l)    # <locked _thread.RLock object owner=37180 count=1 at 0x000001CA920FA3C8>
    l.release()
    print(l)    # <unlocked _thread.RLock object owner=0 count=0 at 0x000001A35A7EA3C8>

rlock = threading.RLock()
t = threading.Thread(target=worker,args=(rlock,))
t.start()
```
通过打印我们看出来：
- owner: 表示当前拥有锁的线程ID
- count：计数器，当前线程获取锁的次数，释放时也要释放这么多次才可以被其他线程获取  

当没有线程获取该锁时，owner和count都为0.
```python
import threading
import logging
import time
FORMAT = '%(asctime)s %(threadName)s %(thread)s %(message)s'
logging.basicConfig(level=logging.INFO,format=FORMAT)

def worker(rlock):
    logging.info('子进程运行')
    rlock.acquire()
    logging.info('子进程获取锁成功！')

rlock = threading.RLock()
logging.info('主进程获取锁1')
rlock.acquire()
t = threading.Thread(target=worker,args=(rlock,))
t.start()
logging.info('主进程获取锁2')
rlock.acquire()
count = 2
while True:
    time.sleep(2)
    if count == 0:break
    logging.info('主进程释放锁')
    rlock.release()
    count -= 1

# 2019-03-04 21:29:41,609 MainThread 18164 主进程获取锁1
# 2019-03-04 21:29:41,610 Thread-1 33748 子进程运行
# 2019-03-04 21:29:41,610 MainThread 18164 主进程获取锁2
# 2019-03-04 21:29:43,611 MainThread 18164 主进程释放锁
# 2019-03-04 21:29:45,611 MainThread 18164 主进程释放锁
# 2019-03-04 21:29:45,611 Thread-1 33748 子进程获取锁成功！
```
等到主进程锁释放完毕后，子进程才可以正常获取锁运行。
# 2 Condition
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;Condition：条件，指的是应用程序状态的改变。这是另一种同步机制，其中某些线程在等待某一条件发生，其他的线程会在该条件发生的时候进行通知。一旦条件发生，线程会拿到共享资源的唯一权限。
> 内部还是通过锁机制实现的，默认情况下 Condition 使用的是Rlock，也可通过参数传递，指定为Lock

|名称|含义|
|---|---|
acquire(*args)|获取锁|
wait(timeout=None)|等待或者超时|
notify(n=1)|唤醒至多指定数目个数的等待的线程，没有等待的线程就没有任何操作|
notify_all|欢迎所有等待的线程|