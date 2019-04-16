# 1 subprocess模块
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;用于执行命令，它取代了比较老得os.system,commands等模块，它通过启动子进程来完成命令的执行，并且提供了更多灵活，高效的方法。

# 2 subprocess.call
功能类似于os.system 无法捕获结果
```python
subprocess.call('ls -l',shell=True)
```
注意当执行的cmd中有空格的时候，python解释器会认为后面的参数都为第一个命令的参数，这个时候方便起见，都会使用shell=True,它真正的含义是：
- shell=True参数会让subprocess.call接受字符串类型的变量作为命令，并调用shell去执行这个字符串
- 当shell=False是，subprocess.call只接受数组变量作为命令，并将数组的第一个元素作为命令，剩下的全部作为该命令的参数。

基于安全考虑，官方的推荐是尽量不要设置shell=True。

# 3 subprocess.check_call
功能和call相同，无法捕捉结果,与call不同的是，check_call默认会检查命令的返回值，如果不是0，则会返回异常subprocess.CalledProcessError

# 4 subprocess.Popen
用于执行命令，但具有更高的定制化，比如信息的输出，返回码的获取，其调用外部命令的时候，会像C那样fork一个子进程进行执行。
```python
# 格式：
Popen(args, bufsize=-1, executable=None, stdin=None, stdout=None, stderr=None, preexec_fn=None, close_fds=<object object at 0x1005ac1c0>, shell=False, cwd=None, env=None, universal_newlines=False, startupinfo=None, creationflags=0, restore_signals=True, start_new_session=False, pass_fds=(), *, encoding=None, errors=None)
```
- args: 执行的命令
- stdin: 输入
- stdout: 输出信息（默认到控制台）
- stderr: 错误输出信息（默认到控制台）
- cwd: 切换到某个目录后再执行命令

例子：
```python
subprocess.Popen('cmd',shell=True,stdout=subprocess.PIPE)
subprocess.Popen(['cmd'],stdout=subprocess.PIPE)    --> 一般用这种
```
上面两行代码功能相同，不写stdout的话，默认会把输出打印到当前的屏幕上,其中PIPE 表示管道符，会把命令的输出方式暂时缓存，然后通过对应的方式.进行读取p.stdout.read()
- stdout  表示标准输出
- stdin 表示标准输入，stdin=subprocess.PIPE,表示输入从PIPE读取（很少用）
- stderr 表示错误输出

## 4.1 Popen的方法
- p.pid  查看子进程的PID
- p.poll()  查看执行状态，完毕返回0
- p.returncode  获取进程的返回值
- p.kill()　kill掉相关进程
- p.terminate()  暂停进程
- p.wait()   等待进程执行完毕,并返回进程执行完毕的`返回码`
- p.communicate()  可以和子进程进行交互，可以给子进程的stdin传递参数（子进程的stdin为PIPE），会返回一个元组，（stdout,stderr）	,需要子进程也输出到PIPE，这样才能接收到  ，所以它会等待子进程执行完毕，相当于p.stdin.write()、p.stdin.close()，p.stdout.read()三个方法

例子：
```python
p = Popen(['ls'],stdout=PIPE,stderr=PIPE)
stdout,stderr = p.communicate() 获取p的stdout和stderr
```

注意：
Popen会fork一个子进程去执行命令，父脚本会继续向下执行，如果要等待Popen执行完毕，那么就需要使用p.wait()来等待子进程执行完毕，再继续执行

# 6 PIPE 管道
会把命令的结果进行储存
例子：
```python
p1 = Popen(['ls'],stdout = PIPE)
p2 = Popen(['grep','py'],stdin=p1.stdout,stdout=PIPE)
p2.stdout.read()
```

# 7 不建议使用PIPE
官方建议在某些场景下不要使用PIPE，因为可能会造成死锁。Popen对象的信息是可以直接输出到文件对象中去的，那么有两个解决办法
- 在内存中构建一个文件对象
- 在磁盘上构建一个文件对象

本质上都是构建一个文件对象，但内存是有限的，如果一个脚本的输出信息过多，可能会给内存造成压力，这里可以模仿操作系统，来一个临时文件，可以利用tempfile模块，来帮我们完成这个需求

# 7.1 tempfile模块
用于生成一个临时的文件
```python
from tempfile import TemporaryFile

temp = TemporaryFile()
print temp
print temp.name

'''
TemporaryFile类的构造方法，其返回的还是一个文件对象。但这个文件对象特殊的地方在于
1. 对应的文件没有文件名，对除了本程序之外的程序不可见
2. 在被关闭的同时被删除
所以上面的两句打印语句，输出分别是一个文件对象，以及一个<fdopen>（并不是文件名）
'''
# 向临时文件中写入内容
temp.write('hello\nworld')

# ...一些操作之后需要读取临时文件的内容了
temp.seek(0)     # 从头读取，和一般文件对象不同，seek方法的执行不能少
print temp.read()

temp.close()    # 关闭文件的同时删除文件
```

`TemporaryFile类`是tempfile中最常用的类之一，其目的就在于提供一个统一的临时文件调用接口，读写临时文件，并且保证临时文件的隐形性。这个类的构造方法和一般的文件对象很类似：tempfile.TemporaryFile([mode='w+b'[, bufsize=-1[, suffix=''[, prefix='tmp'[, dir=None]]]]])。可以看到默认的打开模式是w+b，但是和一般通过wb模式打开的文件对象不同之处在于，正如上面所示它也可以用于读取文件。
- 在TemporaryFile类的基础上又衍生出了两个更加精细化的类用来处理。NamedTemporaryFile类是在前者的基础上，初始化时加上了delete参数，默认值为True。当此参数为True时和TemporaryFile类完全一致。如果是False，那么临时文件对象在被关闭时不会删除。因此可以在下面的代码中通过同样的对象再次打开。
- 另一个是SpooledTemporaryFile，它在TemporaryFile的基础上增加了一个max_size参数默认值为0。当这个类的对象调用write方法向临时文件中写入内容时，这些内容暂时先存在于缓存中，只有当内容大小达到了max_size指定的大小(经试验应该不是这样的，存疑)